# 【网易云课堂】MongoDB索引


索引属性

1. 名称name:
    `db.collection.ensureIndex({},{"name":"indexName"})`
    `db.collection.dropIndex("indexName")`不使用自动生成的名字，可以自定义名称

2. 唯一性unique：
    `db.collection.ensureIndex({},{"unique":true/false})`
    不允许在一个集合中插入两条具有唯一索引的同一个字段值

3. 稀疏性sparse：
    `db.collection.ensureIndex({},{"sparse":true/false})`
    MongoDB默认会为不存在的字段创建索引，为了提高查询速度，避免这类事情发生，可以将sparse设置为true
    查看字段存在的记录` db.article.find({"title":{"$exists":true}})`

4. 过期索引expireAfterSeconds
    创建过期索引的数据过期后数据自动删除

强制使用索引`db.collection.find().hint("indexName")`





## １、简介

MongoDB中索引有两种，一种是自动创建的、一种是手动创建的、
索引不易过多，也不适合在频繁更新的数据上创建索引，影响性能

准备数据
```
db.student.insert({"name":"zhansan-1","sex":"女","age":12,"score":85,"addr":"朝阳区"});
db.student.insert({"name":"zhansan-2","sex":"女","age":11,"score":85,"addr":"朝阳区"});
db.student.insert({"name":"zhansan-3","sex":"女","age":12,"score":88,"addr":"海定区"});
db.student.insert({"name":"zhansan-4","sex":"男","age":13,"score":95,"addr":"大兴区"});
db.student.insert({"name":"zhansan-5","sex":"男","age":12,"score":92,"addr":"东城区"});
db.student.insert({"name":"zhansan-6","sex":"女","age":15,"score":90,"addr":"西城区"});
db.student.insert({"name":"zhansan-7","sex":"男","age":12,"score":99,"addr":"通州"});
db.student.insert({"name":"zhansan-8","sex":"女","age":10,"score":81,"addr":"房山"});
db.student.insert({"name":"zhansan-9","sex":"女","age":19,"score":75,"addr":"天津"});
```

此时在student的文档集合中并没有创建任何索引

### 1-1、查询索引

通过`db.student.getIndexes()`　
```
> db.student.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "test.student"
        }
]
> 

```

### 1-2、创建单键索引

`db.collection.ensureIndex({列：1})`

"1"表示创建的索引将安装升序的方式进行排列，如果使用降序设置"-1"

```
> db.student.ensureIndex({"age":1})
```
如果文档较多，负载较重，创建索引需要消耗很多时间甚至不能创建索引，所以需要在使用数据库之前就将索引创建完毕，否则严重影响数据库的新能

索引名称是自动命名的，命名规范：字段名称_排序方式
### 1-3、复合索引

```
db.student.ensureIndex({"name":1,"age":1})
```

hint("name":1,"age":1)函数强制使用索引


### 1-4、删除一个索引
```
db.student.dropIndex({"name":1,"age":1})
```
### 1-5、清楚全部索引
```
db.student.dropIndexes();
```

## 2、唯一索引

唯一索引的主要目的是用在某一个字段上，使该字段的内容不重复

### 2-1、创建一个唯一索引

```
> db.student.ensureIndex({"name":1},{"unique":true})
```

在name字段上的内容不允许重复


## 3、过期索引

在一些程序的站点上，会有若干秒之后信息被删除的情况，例如手机验证码，在MongoDB中，很容易实现过期索引，但是这个时间
往往不怎么准确


### 3-1、在一个phone的集合里面设置过期索引

过期索引必须设置在时间字段上，例如time字段就是一个时间，设置10秒后过期

```
db.phone.ensureIndex({"time":1},{expireAfterSeconds:10})
```

创建数据


```
db.phone.insert({"tel":"110","code":"110","time":new Date()})
db.phone.insert({"tel":"111","code":"111","time":new Date()})
db.phone.insert({"tel":"112","code":"112","time":new Date()})
db.phone.insert({"tel":"113","code":"113","time":new Date()})
db.phone.insert({"tel":"114","code":"114","time":new Date()})
db.phone.insert({"tel":"115","code":"115","time":new Date()})
```

临时数据保存非常有帮组

## 4、全文索引

传统的只能在某个字段上进行模糊查询，在MongoDB里面实现了非常简单的全文检索

```
db.news.insert({"title":"MongoDB索引","content":"详细介绍MongoDB的全文索引"})
db.news.insert({"title":"springboot简介","content":"详细介绍springboot简介，走向新世界"})
db.news.insert({"title":"springboot详解","content":"详细介绍springboot详解，走进新时代"})
db.news.insert({"title":"springcloud入门","content":"详细介绍springcloud的未来"})
db.news.insert({"title":"springcloud详解","content":"详细介绍springcloud的天下"})
```

### 4-1、创建全文索引
```
db.news.ensureIndex({"title":"text","content":"text"})
```

### 4-2、全文模糊查询

全文查询时，不需要指定具体的字段，使用$text操作用进行全文检索
$text操作符标示就是根据全文索引进行搜索，所以一个集合只允许有一个全文索引，不然的话就不知道根据那个全文索引进行搜索了

如果想表示出全文检索，使用"$text"操作符
如果想进行数据查询，使用"$search"操作符

- 查询指定的关键字：{"$search":"查询关键字"}
- 查询多个关键字（或关系）：{"$search":"查询关键字    查询关键字   ..."}     中间使用空格间隔
- 查询多个关键字（与关系）：{"$search":"\"查询关键字\"  \"查询关键字\"  ..."}     加上双引号，当然要使用转义字符转义双引号
- 查询多个关键字（排除某一个）：{"$search":"查询关键字    查询关键字   ... - 排除关键字"}     用减号"-"标记排除关键字

全文检索，同时检索title和content,好像用空格为匹配单元，springboot匹配不到，springboot简介可以匹配到
```
db.news.find({"$text":{"$search":"springboot"}})

```
包含"springboot简介" 不包含"MongoDB索引"
```
> db.news.find({"$text":{"$search":"springboot简介 -MongoDB索引"}})
```


包含"springboot详解" 或者包含"springboot简介"
```
> db.news.find({"$text":{"$search":"springboot详解 springboot简介"}})
```

但是在全文检索的时候还可以通过相识度打分来判断全文检索成果

```
> db.news.find({"$text":{"$search":"springboot简介"}},{"score":{"$meta":"textScore"}})
```

按照打分的成绩进行排列，分数越高匹配度越高

```
db.news.find({"$text":{"$search":"springboot简介"}},{"score":{"$meta":"textScore"}}).sort({"score":{"$meta":"textScore"}})
```


如果字段特别多，为每一个字段设置全文检索似乎就麻烦了点，我们可以为所有字段设置全文检索


```
> db.student.ensureIndex({"$**":"text"})
```
这个是最简单设置所有字段全文索引的方式，但是最好别用，一个子：慢



## 5、地理位置索引

地理信息索引分为两类：2D平面索引、2DSphere球面索引

mongodb为了计算平面（2d）和球面(2dsphere)距离，使用GeoJSON表示坐标位置的方式

`<field>: { type: <GeoJSON type> , coordinates: <coordinates> }`

```
location: {
      type: "Point",
      coordinates: [-73.856077, 40.848447]
}
```
如果是经纬度，经度在前，纬度在后，经度范围是-180~180；纬度的范围是-90~90

传统的写法有两种：
第一种：通过一个数组
```
<field>: [ <x>, <y> ]    平面坐标
<field>: [<longitude>, <latitude> ]  球面经纬度
```
第二种：
```
<field>: { <field1>: <x>, <field2>: <y> }
<field>: { <field1>: <longitude>, <field2>: <latitude> }
```

准备数据，店铺地址位置信息
```
> db.shop.insert({loc:[10,10]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[11,10]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[10,11]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[12,15]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[16,17]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[90,90]})
WriteResult({ "nInserted" : 1 })
> db.shop.insert({loc:[120,130]})
WriteResult({ "nInserted" : 1 })
> 
```

1. $near 查询，查询距离某个点最近的坐标点
2. $geoWithin 查询，查询某个形状内的点

### 5-1、2D索引

创建索引
```
db.shop.ensureIndex({"loc":"2d"})
```
这个时候shop集合就可以实现坐标位置的查询了，而要进行查询，有两种查询方式

1. $near 查询，查询距离某个点最近的坐标点
2. $geoWithin 查询，查询某个形状内的点





假设当前坐标是[11,11]

```
db.shop.find({"loc":{"$near":[11,11]}})
```
如果执行以上查询，实际上将集合里面前100个点的信息全部返回来了，可是太远了，设置一个范围--3个点以内的
返回距离最近的前三个点
```
db.shop.find({"loc":{"$near":[11,11],"$maxDistance":3}})
```
需要注意的是，在2D索引里面虽然支持最大距离，但是不支持最小距离，在球面计算中，支持最小距离
但是也可以设置一个查询范围，使用 $geoWithin 查询，而可以设置的范围有一下几种

1、矩形范围`（$box）{"$box":[[x1,y1],[x2,y2]]}`
2、圆形范围`（$center）{"$center":[[x1,y1],r]]}`
3、多边形范围`（$polygon）{"$polygon":[[x1,y1],[x2,y2],[x3,y3],...]}`

查询在矩形内的所有点
```
> db.shop.find({"loc":{"$geoWithin":{"$box":[[9,9],[11,11]]}}})
```

在MongoDB内，除了一些支持的操作函数外，还有一个重要的命名：runCommand()
这个函数可以支持所有的特定的MongoDB命令

利用runCommand()命令执行查询

```
db.runCommand({"geoNear":"shop",near:[10,10],maxDistance:5,num:2})
```




### 5-2、2dsphere索引

创建索引

```
db.collection.createIndex( { <location field> : "2dsphere" } )
```

范例
```
db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },
      name: "Central Park",
      category : "Parks"
   }
)

db.places.createIndex( { loc : "2dsphere" } )
```
查询操作符和2d索引类似

1. $near 查询，查询距离某个点最近的坐标点
2. $geoWithin 查询，查询某个形状内的点

查询距离最近的点并且排序
```
db.<collection>.find( { <location field> :
                         { $near :
                           { $geometry :
                              { type : "Point" ,
                                coordinates : [ <longitude> , <latitude> ] } ,
                             $maxDistance : <distance in meters>
                      } } } )
```


查询球面上指定圈内的所有点
```
db.<collection>.find( { <location field> :
                         { $geoWithin :
                           { $centerSphere :
                              [ [ <x>, <y> ] , <radius> ] }
                      } } )
```
## 6、索引构建情况分析

索引好处：加快查询速度
索引不好处：增加磁盘空间消耗，降低写入性能

如何评判当前索引的构建情况？

1. mongostat工具
2. profile集合
3. 日志
4. explain分析

### 6-1、mongostat工具

mongostat：查看MongoDB运行状态的程序

使用说明：mongostat -h ip:port -u username -p password

### 6-2、profile集合

```
> db.getProfilingLevel()
2
> db.getProfilingStatus()
{"was":2,"slowms":1}
```

profile分为3个级别：0、1、 2

0级别：profile是关闭的，MongoDB不会记录任何操作
1级别：会配置slowms预设值，MongoDB会记录所有操作超过slowms的操作
2级别：MongoDB会记录任何操作，默认级别

设置级别
```
> db.setProfilingLevel(2)
> db.system.profile.find().sort({"$natural":-1})  $natural是根据数据写入先后的时间顺序进行排序
```

### 6-3、日志

通过verbose选项配置日志显示的级别

配置文件
```
#可以使用相对路径也可以使用绝对路径，这里使用相对路径
dbpath=db
#启动日志
logpath=log/mongod.log
#后台启动
fork=true
#security
auth=true
bind_ip_all=true
#v越多，显示的日志越详细
verbose=vvvvv
```

### 6-3、explain()

```
> db.demo.find({"x":1}).explain()
```











