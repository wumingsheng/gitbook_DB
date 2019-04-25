# 【网易云课堂】MongoDB基本使用


## 1、 数据增加

语法：db.集合.insert()

范例：

1、增加一个简单操作

```
> use test
switched to db test
> db.infos.insert({"url":"www.baidu.com"})
WriteResult({ "nInserted" : 1 })
> db.infos.find()
{ "_id" : ObjectId("5a8f8654e9efc2d31c1c357b"), "url" : "www.baidu.com" }
```

2、保存多条数据，数组

```
db.infos.insert([{"url":"www.kuaibo.com"},{"url":"www.baidu.com"},{"url":"www.mongodb.org"}])
```
3、保存10000个数据？

语法类似JavaScript

```
for(var x=0;x<10000;x++){

    db.infos.insert({"url":"mongodb - " + x})
}

```

## 2、数据查询

mongo查询语法强大，包括关系运算，逻辑运算，数组运算，正则运算

语法：db.集合.find({查询条件}[,{设置显示的字段}])

范例：

1、最简单的就是使用find()函数

2、希望查询出url为“mongodb-888”的数据

```
> db.infos.find({"url":"mongodb - 888"})
{ "_id" : ObjectId("5a8f887be9efc2d31c1c38f7"), "url" : "mongodb - 888" }
> 
```

发现在进行数据查询的时候，也是按照json的形式设置的相等的关系，在整个开发中都不可能离开json数据

对于设置的显示字段严格来讲就称为数据的投影操作
如果不需要显示的字段设置为0，需要显示的字段设置为1

3、不想显示"_id"

```
> db.infos.find({"url":"mongodb - 888"},{"_id":0})
{ "url" : "mongodb - 888" }
> db.infos.find({"url":"mongodb - 888"},{"_id":0,"url":1})
{ "url" : "mongodb - 888" }
```

大部分情况下，这种投影操作的意义不打。同事对于数据的查询也可以使用`pretty()`函数

4、漂亮显示pretty函数格式化显示

```
> db.infos.find({"url":"mongodb - 888"},{"_id":0,"url":1}).pretty()
```

## 3、关系查询

在MongoDB中，关系运算：
大于（$gt）、大于等于（$gte）、小于（$lt）、小于等于（$lte）、不等于（$ne）、等于（key:value、$eq）

范例：定义学生信息集合

```
db.student.insert({"name":"zhansan-2","sex":"female","score":85,"addr":"朝阳区"});
db.student.insert({"name":"zhansan-3","sex":"female","score":88,"addr":"海定区"});
db.student.insert({"name":"zhansan-4","sex":"male","score":95,"addr":"大兴区"});
db.student.insert({"name":"zhansan-5","sex":"male","score":92,"addr":"东城区"});
db.student.insert({"name":"zhansan-6","sex":"female","score":90,"addr":"西城区"});
db.student.insert({"name":"zhansan-7","sex":"male","score":99,"addr":"通州"});
db.student.insert({"name":"zhansan-8","sex":"female","score":81,"addr":"房山"});
db.student.insert({"name":"zhansan-9","sex":"female","score":75,"addr":"天津"});
```
1、年龄等于zhansan
```
> db.student.find({"name":"zhansan"}).pretty()
{
        "_id" : ObjectId("5a8f8da2e9efc2d31c1c5c8f"),
        "name" : "zhansan",
        "sex" : "male",
        "score" : 89,
        "addr" : "chaoyang"
}
```

2、成绩大于90的

MongoDB自己的BSON结构语法

```
db.student.find({"score":{"$gt":90}}).pretty()

```
成绩大于90并且地址在通州
```
> db.student.find({"score":{"$gt":90},"addr":"通州"}).pretty()
{
        "_id" : ObjectId("5a8f8ebce9efc2d31c1c5c96"),
        "name" : "zhansan-7",
        "sex" : "male",
        "score" : 99,
        "addr" : "通州"
}
```


## 4、逻辑查询

逻辑查询主要有三种：与（$and）、或（$or）、非（$not、$nor）

查询分数在85~90

```
> db.student.find({"score":{"$gt":85,"$lt":90}}).pretty()
```


查询成绩大于90或者地址在通州
```
> db.student.find({"$or":[{"score":{"$gt":90}},{"addr":{"$eq":"通州"}}]})

```

或的求反$nor
```
> db.student.find({"$nor":[{"score":{"$gt":90}},{"addr":{"$eq":"通州"}}]})

```
## 5、模查询

模的查询使用`"$mod"`来完成
语法：{"$mod":[数字，余数]}
范例：

```
db.student.find({"score:{"$mod":[20,0]}"}).pretty()
```
## 6、范围查询

只要是数据库，必须存在"$in"（在范围之中）,"$nin"（不在范围之中）

查询成绩在范围内
```
> db.student.find({"score":{"$in":[92,95,99,85]}})
```

## 7、数组查询

在MongoDB中支持数组保存，一旦支持了数组保存，就需要针对数组的数据进行匹配


```
db.student.insert({"name":"woms-1","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"]});
db.student.insert({"name":"woms-2","sex":"female","score":90,"addr":"西城区","course":["语文","英语","政治","地理"]});
db.student.insert({"name":"woms-3","sex":"female","score":90,"addr":"西城区","course":["语文","数学","政治","地理"]});
db.student.insert({"name":"woms-4","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","地理"]});
db.student.insert({"name":"woms-5","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治"]});
```


可以使用MongoDB的BSON的几个操作符：$all、$size、$slice、$elemMathc

范例：

查询同时参加语文和数学的学生{"$all",[内容1,内容2,...]}
```
> db.student.find({"course":{"$all":["英语","地理"]}}).pretty()
```

查询数组中第二个内容为数学的信息（既然是数组就可以使用索引）
```
db.student.find({"course.1":"数学"})
```

查询数组个数，要求只参加两门课程的学生
```
db.student.find({"course":{"$size":2}})
```
控制数组返回的长度，截取数组，只查询出前两门课程
```
#取前两门的信息
 db.student.find({"name":"woms-1"},{"course":{"$slice":2}}).pretty()
 # -2表示后两门的信息
  db.student.find({"name":"woms-1"},{"course":{"$slice":-2}}).pretty()
  #取中间两门的信息，第一个数据表示跳过的数据量，第二个数据表示返回的数据个数
   db.student.find({"name":"woms-1"},{"course":{"$slice":[1,2]}}).pretty()
```
## 8、嵌套集合查询

mongo数据库里面每一个集合数据可以继续保存其他的集合数据

```
db.student.insert({"name":"wms-1","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"],"parent":[{"name":"wms-1-m","age":50},{"name":"wms-1-f","age":48}]});
db.student.insert({"name":"wms-2","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"],"parent":[{"name":"wms-2-m","age":51},{"name":"wms-2-f","age":47}]});
db.student.insert({"name":"wms-3","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"],"parent":[{"name":"wms-3-m","age":52},{"name":"wms-3-f","age":46}]});
db.student.insert({"name":"wms-4","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"],"parent":[{"name":"wms-4-m","age":53},{"name":"wms-4-f","age":45}]});
db.student.insert({"name":"wms-5","sex":"female","score":90,"addr":"西城区","course":["语文","数学","英语","政治","地理"],"parent":[{"name":"wms-5-m","age":55},{"name":"wms-5-f","age":44}]});
```

嵌套集合的数据判断只能通过`$elemMatch`去嵌套查询条件

```
db.student.find({"parent":{"$elemMatch":{"name":"wms-5-m"}}}).pretty()
```

## 9、判断某个字段是否存在

BSON=json+mongod操作符$

使用`$exists`可以判断某个字段是否存在，如果设置为true表示存在

范例：查询具有parent成员的数据
```
> db.student.find({"parent":{"$exists":true}})
```

## 10、条件过滤

在MongoDB中也提供有`"$where"`操作符


范例：使用where进行条件查询

```
> db.student.find({"$where":"this.score>90"})
#可以简写成
> db.student.find("this.score>90")
```

这类的操作是属于对于每一行信息判断，实际上对于数据量大的情况不方便使用
实际上相当于编写了一个操作的函数
```
db.student.find({"$where":function(){
    return this.score > 90;
}})
db.student.find(function(){
    return this.score > 90;
})
```
多条件链接，$and、$or最高级别操作符
```
 db.student.find({"$and":[{"$where":"this.score>90"},{"$where":"this.score<95"}]})
```

这个方式不方便使用数据库索引机制，不推荐使用

## 11、模糊查询-正则运算

如果要实现模糊查询，必须使用正则表达式，而正则表达式使用的是语言perl兼容的正则表达式的形式

基础语法：`{key:正则标记}`
完整语法：`{key:{"$regex":正则标记,"$options":选项}}`

对于options主要是设置正则的信息查询的标记：

- "i"     :忽略字母大小写;
- "m"  :多行查找;
- "x"    :空白字符除了被转义的或者在字符类中以外的完全被忽略;
- "s"    :匹配所有的字符（圆点、"."），包括换行内容

需要注意的是，如果是直接使用（基础语法）那么只能够使用i和m，

```
> db.student.find({"name":/woms/})
```

正则表达式没有双引号，有双引号的是字符串


不区分大小写
```
> db.student.find({"name":/woms/i})
```

如果要执行模糊查询的操作，严格来讲，值需要编写一个操作符就够了

正则除了可以查询出单个字段的内容外，还可以进行数组数据的查询

```
> db.student.find({"course":/woms?/})
```

## 12、数据排序

使用"sort()"函数，升序（1）、降序（-1）

```
> db.student.find().sort({"score":1}).pretty()
```

但是在排序里面，有一种方式称为自然排序，按照数据保存的先后顺序排序，使用`$natural`表示

```
> db.student.find().sort({"$natural":1}).pretty()
```


## 13、分页显示

- skip(n): 表示跨过多少数据行
- limit(n):表示取出数据行的个数限制

范例
第1页，skip(0).limit(5).sort({"score":1}).pretty()
第2页，skip(5).limit(5).sort({"score":1}).pretty()
```
> db.student.find().skip(0).limit(3).sort({"score":1}).pretty()
```

## 14、数据跟新操作

在MongoDB里面提供了两个函数save()、update()

update()函数语法：db.table.update(更新条件，新的对象数据（跟新操作符），upsert，multi)

- upsert ： 如果更新的数据不存在，就增加一条新的内容（true为增加，false不增加），默认false
- multi    ：表示是否只更新满足条件的第一条记录（false：只更新第一条记录，ture全部更新）默认false

老语法：

```
> db.student.update({"name":"zhangsan"},{"$set":{"score":100}},false,false)
```

upsert标记付在前，multi标记付在后，如果只有一个标记付，表示upsert，在前的优先，默认multi采用的是默认值
```
> db.student.update({"name":"zhangsan"},{"$set":{"score":100}},false)
```

新语法：

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```


范例：

默认值修改第一条被查询到的

```
> db.demo.find()
{ "_id" : ObjectId("5a935eaccad1423cb6ed5c6f"), "x" : 1 }
{ "_id" : ObjectId("5a935eaecad1423cb6ed5c70"), "x" : 1 }
{ "_id" : ObjectId("5a935eafcad1423cb6ed5c71"), "x" : 1 }
> db.demo.update({"x":1},{"x":2})       
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.demo.find()
{ "_id" : ObjectId("5a935eaccad1423cb6ed5c6f"), "x" : 2 }
{ "_id" : ObjectId("5a935eaecad1423cb6ed5c70"), "x" : 1 }
{ "_id" : ObjectId("5a935eafcad1423cb6ed5c71"), "x" : 1 }
```
如果修改多条，老语法
```
> db.demo.update({"x":1},{"$set":{"x":2}},false,true)
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
> db.demo.find()
{ "_id" : ObjectId("5a93603ecad1423cb6ed5c72"), "x" : 2 }
{ "_id" : ObjectId("5a93603fcad1423cb6ed5c73"), "x" : 2 }
{ "_id" : ObjectId("5a93603fcad1423cb6ed5c74"), "x" : 2 }
```
如果修改多条，新语法

```
> db.demo.update({"x":2},{"$set":{"x":3}},{"multi":true})
WriteResult({ "nMatched" : 3, "nUpserted" : 0, "nModified" : 3 })
> db.demo.find()
{ "_id" : ObjectId("5a93603ecad1423cb6ed5c72"), "x" : 3 }
{ "_id" : ObjectId("5a93603fcad1423cb6ed5c73"), "x" : 3 }
{ "_id" : ObjectId("5a93603fcad1423cb6ed5c74"), "x" : 3 }
```

multi也可以使用updateMany()和updateOne()代替

```
db.demo.updateMany({"x":3},{"$set":{"x":4}})
```






save()和更新不存在相似


## 15、修改器

MongoDB数据的修改会牵扯到内容的变更，结构的变更（包含有数组），所以在MongoDB设计的时候就提供了一些列修改器的应用

1、$inc    主要针对一个数字字段，增加某个数字字段的内容

        - 语法：{"$inc":{"成员":"内容"}}
        - db.student.update({"name":"zhansan"},{"$inc":{"score":-20}})  将姓名是zhansan的成绩减去20
        
        
2、$set     部分更新操作付

    不适用$set,所有的内容都会被覆盖，使用$set操作符，被更新的字段如果存在就更新，如果不存在就不更新

        - 语法：{"$set":{"成员":"新内容"}}
        - db.student.update({"name":"zhangsan"},{"$set":{"score":100}},false,false)   将姓名是zhansan的成绩重新设置成100
        
如果不用$set,整条记录被覆盖

```
> db.imooc.insert({"x":1,"y":2,"z":3})
WriteResult({ "nInserted" : 1 })
> db.imooc.find()
{ "_id" : ObjectId("5a935b4bcad1423cb6ed5c6d"), "x" : 1, "y" : 2, "z" : 3 }
> db.imooc.update({"x":1},{x:111})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.imooc.find()
{ "_id" : ObjectId("5a935b4bcad1423cb6ed5c6d"), "x" : 111 }
```
如果使用$set,部分修改，存在的字段修改，不存在就不修改

```
> db.imooc.insert({"x":1,"y":2,"z":3})
WriteResult({ "nInserted" : 1 })
> db.imooc.find()
{ "_id" : ObjectId("5a935c3ecad1423cb6ed5c6e"), "x" : 1, "y" : 2, "z" : 3 }
> db.imooc.update({"x":1},{"$set":{"x":999}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.imooc.find()
{ "_id" : ObjectId("5a935c3ecad1423cb6ed5c6e"), "x" : 999, "y" : 2, "z" : 3 }
> 
```
        
3、$unset   删除某个成员的内容

        - 语法：{"$unset":{"成员"：1}}
        - db.student.update({"name":"zhansan"},{"$unset":{"addr":1}}) 将姓名是zhansan的地址字段删除掉
        
        
4、$push   相当与将内容追加到指定的成员之中（基本上是数组）


    如果没有数组创建一个新的数组，如有有数组，内容追加

        - 语法：{"$push":{"成员"：value}}
        -  db.student.update({"name":"zhansan"},{"$push":{"course":"shuxue"}})

5、$pushAll   与push类似，可以一次追加多个内容到数组中

    

        - 语法：{"$pushAll":{"成员":数组内容}}
        -  db.student.update({"name":"zhansan"},{"$pushAll":{"course":["c","java","python"]}})


新版本取消了$pushAll,使用$each结合$push
```
db.students.update(
   { name: "joe" },
   { $push: { scores: { $each: [ 90, 92, 85 ] } } }
)
```
6、$addToSet 向数组里添加新内容，只有数组内容不存在被添加的内容的时候才增加新的内容

也就是说，如果值存在的情况下，是不添加的

        - 语法：{"$addToSet":内容}
        - > db.student.update({"name":"zhansan"},{"$addToSet":{"course":"php"}})


7、$pop   删除数组内的第一个数据或者最后一个数据

https://docs.mongodb.com/manual/reference/operator/update/pop/

    如果字段不是一个数组，操作失败，如果移除了最后一个元素，则字段保留一个空数组

        - 语法：{ "$pop" : { <field>: <-1 | 1>, ... } }
        -db.students.update( { _id: 1 }, { $pop: { scores: -1 } } )  -1移除第一个元素;1移除最后一个元素
        
8、$pull 从数组内删除一个指定内容的数据（如果有多个匹配，全部删除）

进行数据比对，如果是此数据，进行删除

        - 语法：{ $pull: { <field1>: <value|condition>, <field2>: <value|condition>, ... } }
        - db.stores.update(
                { },
                { $pull: { fruits: { $in: [ "apples", "oranges" ] }, vegetables: "carrots" } },
                { multi: true }
            )

        
9、$pullAll  从一个数组中一次性删除多个指定值的多个内容

        - 语法：{ $pullAll: { <field1>: [ <value1>, <value2> ... ], ... } }
        - db.survey.update( { _id: 1 }, { $pullAll: { scores: [ 0, 5 ] } } )


10、$rename 字段重新命名

        - 语法：{$rename: { <field1>: <newName1>, <field2>: <newName2>, ... } }
        - db.students.update( { _id: 1 }, { $rename: { 'nickname': 'alias', 'cell': 'mobile' } } )


## 16、删除数据

使用`remove()`函数


但是这个函数有两个可选项的

- 删除条件，满足条件的数据被删除
- 是否只删除一个数据，如果true或者1表示只删除一个，默认是false，全部删除

清空集合中的内容

- 老语法 
```
db.collection.remove(
   <query>,
   <justOne>
)
```

- 新语法
```
    db.collection.remove(
       <query>,
       {
         justOne: <boolean>,
         writeConcern: <document>,
         collation: <document>
       }
    )
```

清空数据：db.collection.remove({})
不添加条件，就是删除所有的数据，清空数据

db.collection.remove({})和db.collection.dtop()区别：
db.collection.remove({})：值删除集合内的数据，不删除集合的其他常用数据，集合仍然存在
db.collection.dtop()：删除集合，集合内的数据、索引、用户等全部删除，这个集合就没有了




`db.products.remove( { qty: { $gt: 20 } }, true ) `  这个是老语法
`db.products.remove( { qty: { $gt: 20 } }, {justOne: true} ) ` 这个是新语法


## 17、游标

游标是指数据可以一行一行的进行操作，非常类似与ResultSet，游标的控制非常简单，就是使用find()函数

对于已返回的游标，如果想要进行操作，用两个函数

- 判断是否有下一行数据：hasNext()
- 取出当前数据：next()


```
> var cursor = db.student.find()
> cursor.hasNext()
true
> cursor.next()
{
        "_id" : ObjectId("5a8f8da2e9efc2d31c1c5c8f"),
        "sex" : "male",
        "score" : 80,
        "course" : [
                "yuwen",
                "shuxue",
                "c",
                "java",
                "python",
                "php"
        ],
        "coursee" : [
                "php"
        ],
        "alias" : "zhansan"
}
> 
```


以上是游标的操作形式，但是实际上不可能这么用，因为我们必须用循环才能输出内容

```
var cursor = db.student.find()
while(cursor.hasNext()){

var doc = cursor.next();
printjson(doc)
}


```















