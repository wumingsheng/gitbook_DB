# 【第四章】MongoDB的索引介绍

# db第四章：mongoDB的索引介绍

标签（空格分隔）： mongoDB

---

 db.user.ensureIndex({age:1})　创建索引
 db.user.getIndexes()　查询索引

## 索引的种类

1. _id索引
2. 单键索引
3. 多建索引
4. 复合索引
5. 过期索引
6. 全文索引
7. 地理位置索引

## 索引的属性

创建索引的格式：`db.table.ensureIndex({param},{param})`,其中第二个参数便是索引的属性

比较重要的属性有：

* name(名字)：mongoDB会自动给索引创建一个名字，可以自定义索引的名字
    * `db.imooc.ensureIndex({ss:1,dd:-1},{name:"myindex"})`
    * 可以通过索引的名字删除索引`db.imooc.dropIndex("myindex")`
* unique(唯一性):
    * `db.imooc.ensureIndex({},{unique:true/false})`
* sparse(稀疏性):默认false
    * true:只为存在的字段创建索引，false:索引字段不存在，任然保留索引
    * `db.table.ensureIndex({},{sparse:true/false})`
* expireAfterSeconds(是否定时删除)
    * TTL,过期索引

## _id索引

* _id索引是绝大多数集合（表）默认建立的索引
* 对于美插入的数据，mongoDB都会自动生成一条唯一的_id索引

## 单键索引

* 单键索引是最普通的索引
* 与_id索引不同，单键索引不会自动创建`db.imooc.ensureIndex({x:1})`

## 多建索引

* 多建索引与单键索引创建的形式相同，区别在于字段的值
    * 单键索引：值为单一的值，例如字符串，数字或者日期
    * 多建索引：值具有多个记录，例如数组
    * db.imooc.insert({x:[1,2,3,4,5]})


## 复合索引

 - db.imooc.insert({x:1,y:2,z:3}) 
 - db.imooc.ensureIndex({x:1,y:2})
 - db.imooc.find({x:1,y:2})

## 过期索引

* 在一段时间后，会过期的索引
* 在索引过期后，相应的数据也会被删除
* 这适合存储一些在一段时间后会失效的数据，比如用户的登录信息，存储日志
* 建立方法，db.collection.ensureIndex({time:1},{expireAfterSeconds:10})单位秒

1. 存储在过期索引字段的值必须是指定的时间类型（必须是ISODate或者ISODate数组，不能是时间戳，否则不能被自动删除）
2. 如果指定了ISODate数组，则按照最小的时间进行删除
3. 过期索引不能是复合索引（不能有两个过期时间）
4. 删除时间不是精确的（删除程序在后台每60s跑一次，而且删除需要时间，所以存在误差）

## 全文索引

一张表中只允许创建一个全文索引

* 全文索引：对字符串与字符串数组创建全文可搜索的索引
* 使用情况：｛author:"",title:"",article:""｝
* 创建方法：
    * `db.table.ensureIndex({"$**":"text"})`
    * `db.table.ensureIndex({key:"text"})`
    * `db.table.ensureIndex({key_1:"text",key_2:"text"})`
* 查询方法
    *  `db.table.find({$text:{$search:"coffee"}})`
    *  `db.table.find({$text:{$search:"aa bb cc"}})`包含aa或bb或cc
    *  `db.table.find({$text:{$search:"aa bb -cc"}})`包含aa或bb不包含cc的
    *  `db.table.find({$text:{$search:"\"aa\" \"bb\" \"cc\""}})`用引号引起来，既包含aa又包含bb又包含cc

* 全文索引相似度
    * 命令：` db.imooc.find({$text:{$search:"aa cc"}},{score:{$meta:"textScore"}})`
    * 根据相似度排序：`db.imooc.find({$text:{$search:"aa cc"}},{score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}})`

* 全文索引限制
    * 每次查询只能制定一个$text查询
    * $text查询不能出现在$nor查询
    * 查询中如果包含了$text,hint不再起作用（hint强制指定索引）
    * 很可惜，mongoDB全文索引不支持中文




## 地理位置索引

### 概念：
将一些点的位置存储在mongoDB中，创建索引后，可以按照位置来查找其他点

### 子分类：

* 2d索引，用于存储和查找平面上的点
    * 创建方式：`db.table.ensureIndex({fieldName:"2d"})`
    * 插入地理位置数据`db.location.insert({w:[50,10]})`
    * 查询距离某个点最近的点：`$near`,`db.location.find({w:{$near:[1,1],$maxDistance:10}})`
    * 查询某个形状内的点：`$geoWithin`
    
* 2dsphere索引，用于存储和查找球面上的点
* 创建方式：`db.table.ensureIndex({fieldName:"2dsphere"})`
    * 位置标示方式：geoJSON：描述一个点，一条直线，多边形形状
        * 格式：{type:"",coordinates:[<coordinates>]}
        
## 索引构建情况分析

1. mongostat工具介绍
2. profile集合介绍
3. 日志介绍
4. explain分析