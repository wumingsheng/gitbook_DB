# 【网易云课堂】MongoDB聚合


MongoDB的产生背景是在大数据的环境下，所谓的聚合就是统计操作

## 1、获取集合个数

count()函数

```
> db.student.count()
9
```

模糊查询
```
> db.student.count({"name":/zhan/i})
9
```

## 2、消除重复数据


传统的sql数据库使用`distinct`，在MongoDB中依然支持

本次操作没有直接的函数支持，只能通过runCommand()指令

```
> db.runCommand({"distinct":"student","key":"name"})
{
        "values" : [
                "zhansan-1",
                "zhansan-2",
                "zhansan-3",
                "zhansan-4",
                "zhansan-5",
                "zhansan-6",
                "zhansan-7",
                "zhansan-8",
                "zhansan-9"
        ],
        "ok" : 1
}
```

进过官方查看，发现有函数支持
```
db.collection.distinct(field, query, options)
```
上面可以用下面的代替
```
> db.student.distinct("name")
> db.student.distinct("name",{})
```





## 3、聚合框架

aggregate()函数

### 3-1、$group



在MongoDB中会将集合依据指定的key的不同进行分组操作，并且每一个组都会产生一个处理的文档结果
```
{ $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }
```

_id是分组条件，如果_id设置为null，就是将整个文档作为一个整体组，不分组统计

```
db.sales.aggregate(
   [
      {
        $group : {
           _id : null,     设置为null，不分组，整个文档就是一个整体
           totalPrice: { $sum: { $multiply: [ "$price", "$quantity" ] } },
           averageQuantity: { $avg: "$quantity" },
           count: { $sum: 1 }
        }
      }
   ]
)
```

```
 db.student.aggregate({
 "$group":{
        _id:"$name",        name字段作为分组条件
         totalAge:{$sum:"$age"}   对age字段求和
         }
  })
  
  {
            "$group":
                    {
                        "_id":"name",
                        totalAge:{$sum:"$age"},
                        count:{$sum:1}    $sum 1就是求记录个数
                     }
  }
```
数组显示$push

```
db.student.aggregate({"$group":{"_id":"$name",first_score:{$first:"$score"},name:{"$push":"$name"},count:{$sum:1}}})
{ "_id" : "zhansan-9", "first_score" : 75, "name" : [ "zhansan-9" ], "count" : 1 }
{ "_id" : "zhansan-6", "first_score" : 90, "name" : [ "zhansan-6" ], "count" : 1 }
{ "_id" : "zhansan-5", "first_score" : 92, "name" : [ "zhansan-5" ], "count" : 1 }
{ "_id" : "zhansan-8", "first_score" : 81, "name" : [ "zhansan-8" ], "count" : 1 }
{ "_id" : "zhansan-1", "first_score" : 85, "name" : [ "zhansan-1", "zhansan-1" ], "count" : 2 }
{ "_id" : "zhansan-3", "first_score" : 88, "name" : [ "zhansan-3" ], "count" : 1 }
{ "_id" : "zhansan-2", "first_score" : 85, "name" : [ "zhansan-2" ], "count" : 1 }
{ "_id" : "zhansan-7", "first_score" : 99, "name" : [ "zhansan-7" ], "count" : 1 }
{ "_id" : "zhansan-4", "first_score" : 95, "name" : [ "zhansan-4" ], "count" : 1 }
```

数组去重$addToSet

```
> db.student.aggregate({"$group":{"_id":"$name",first_score:{$first:"$score"},name:{"$addToSet":"$name"},count:{$sum:1}}})
{ "_id" : "zhansan-9", "first_score" : 75, "name" : [ "zhansan-9" ], "count" : 1 }
{ "_id" : "zhansan-6", "first_score" : 90, "name" : [ "zhansan-6" ], "count" : 1 }
{ "_id" : "zhansan-5", "first_score" : 92, "name" : [ "zhansan-5" ], "count" : 1 }
{ "_id" : "zhansan-8", "first_score" : 81, "name" : [ "zhansan-8" ], "count" : 1 }
{ "_id" : "zhansan-1", "first_score" : 85, "name" : [ "zhansan-1" ], "count" : 2 }
{ "_id" : "zhansan-3", "first_score" : 88, "name" : [ "zhansan-3" ], "count" : 1 }
{ "_id" : "zhansan-2", "first_score" : 85, "name" : [ "zhansan-2" ], "count" : 1 }
{ "_id" : "zhansan-7", "first_score" : 99, "name" : [ "zhansan-7" ], "count" : 1 }
{ "_id" : "zhansan-4", "first_score" : 95, "name" : [ "zhansan-4" ], "count" : 1 }
```

注意，所有的分组都是无序的

### 3-2、$project

可以控制数据列的显示规则，可以执行的规则如下

1、普通列（{"成员"：1|true}）表示要显示的内容
2、_id列（{"_id": 0|false}）表示“_id”列是否显示
3、条件过滤列（{"成员"：表达式}）满足表达式之后的数据可以进行显示


只显示name列，只有被设置进去的列才被显示出来，_id默认显示
```
> db.student.aggregate([{"$project":{_id:0,name:1}}])
{ "name" : "zhansan-1" }
```
起始就是数据库的投影机制，在这个机制中也支持四则运算：加法（`$add`）、减法（`$substract`）、乘法（`$multiply`）、除法（`$dividi`）、求摸（`$mod`）

除了四则运算还支持

关系运算符：大小比较（$cmp）、等于（$eq）、不等于（$ne）、大于（$gt）、判断null（$ifNull）这些返回值都是boolean类型
逻辑运算符：与（$and）、或（$or）、非（$not）
字符串操作：连接（$concat）、截取（$substr）、转小写（$toLower）、转大写（$toUpper）、不区分大小写比较（$strcasecmp）

显示name不显示_id,判断score是否大于等于90
```
> db.student.aggregate({"$project":{"name":1,_id:0,score:{$gte:["$score",90]}}})
```

### 3-3、$sort

```
> db.student.aggregate({"$sort":{"score":1}})
```


### 3-4、分页处理

$limit和$skip

要先夸再取

```
> db.student.aggregate([{"$project":{"_id":0,"name":1}},{"$skip":2},{"$limit":2}])
```

### 3-5、$unwind

在查询数据时，经常会返回数组信息，但是数组并不是方便信息的浏览，所以提供有"$unwind",将数组信息转换为字符串

```
{ "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] }

db.inventory.aggregate( [ { $unwind : "$sizes" } ] )

{ "_id" : 1, "item" : "ABC1", "sizes" : "S" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "M" }
{ "_id" : 1, "item" : "ABC1", "sizes" : "L" }
```
### 3-6、$geoNear

计算距离，必须要有索引的支持

```
db.places.aggregate([
   {
     $geoNear: {
        near: { type: "Point", coordinates: [ -73.99279 , 40.719296 ] },
        distanceField: "dist.calculated",
        maxDistance: 2,
        query: { type: "public" },
        includeLocs: "dist.location",
        num: 5,
        spherical: true
     }
   }
])

db.places.aggregate([
   {
     $geoNear: {
        near: { type: "Point", coordinates: [ -73.99279 , 40.719296 ] },
        distanceField: "dist.calculated",
        minDistance: 2,
        query: { type: "public" },
        includeLocs: "dist.location",
        num: 5,
        spherical: true
     }
   }
])
```

### 3-7、$out

将查询结果输出到指定的集合里面（表的复制操作，生成一张新表）
```
db.books.aggregate( [
                      { $group : { _id : "$author", books: { $push: "$title" } } },
                      { $out : "authors" }
                  ] )

```


