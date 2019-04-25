# 【传智播客】MongoDB固定集合


## 概念

在创建集合（表table）的时候，指定集合的大小，如果空间不足，最早的文档会被删除，为新文档腾出空间

## 应用场景

 聊天记录，通话记录，session超时，


## 特性

固定集合适用于任何想要自动淘汰过期属性的场景，没有太多的操作限制

## 范例

```
db.createCollection("collectionName",{“capped”：true,size:100000,max:100})
```
capped:true 固定集合
size:集合大小，单位是kb
max：指定文档的数量

集合大小和文档数量同事约束













