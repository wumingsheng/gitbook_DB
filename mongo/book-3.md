# 【第三章】MongoDB的基本使用


标签（空格分隔）： mongoDB

---
mongoDB没有表结构，没有字段类型，不支持事物
｛｝代表一行，逗号分割各个字段，冒号前是字段名，冒号后是字段值

### 数据插入insert,查询find

show dbs 列出数据库
use test　切换数据库（如果没有数据库自动创建一个数据库）
db.dropDatabase()  删除数据库
show collections 列出当前数据库中表show tables
db.imoonc.insert({x:1}) 向表中插入一条数据（如果没有表自动创建）
db.imooc_collection.find()　查询，find查询条件可以为null，默认查询出所有文档
db.imooc_collection.insert({x:3,_id:3})　插入_id默认生成全局唯一，也可以指定，分布式中要指定全局唯一
db.imooc_collection.find({x:3})　　带条件查询

查询find支持scape,limit,sort
插入多条数据for(i=8;i<100;i++)db.imooc_collection.insert({x:i,_id:i})
查询条数db.imooc_collection.find().count()
db.imooc_collection.find().skip(3).limit(5).sort({x:1})

### 数据更新update

db.imooc_collection.update({x:3},{x:333})　全部更新
db.imooc_collection.update({z:300},{$set:{z:301}})　部分更新
db.imooc_collection.update({dji:111},{fjdjio:444},true)　更新数据，如果数据不存在就创建
db.imooc_collection.update({c:1},{$set:{c:2}},false,true)　mongoDB默认只更新满足条件的第一条数据，如果全部更新，需要使用set命令，第四个true代表更新全部满足条件的记录

### 数据删除remove

db.imooc_collection.remove({c:2}) 部分删除
db.imooc_collection.drop()　　　整张表删除








