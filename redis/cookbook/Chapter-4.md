# 【Chapter-4】Redis开发




## redis 常见应用场景


1. 回话存储-session共享，redis中对键的过去的支持可以天然地用于回话的超时管理
2. 分析，统计计数器
3. 排行榜，使用sorted set，将榜分作为元素的权重，使用zrevrange命令返回元素列表
4. 队列，list的push、pop命令
5. 最新的n个记录，lpush key和ltrim key 0 10，key将始终包含最新增加的10家条记录
6. 缓冲，在数据库面前使用redis作为缓冲增加数据库的查询过程







- 在关系数据库中运行缓慢或难以完成的任务，考虑使用redis可以快速和轻松的完成
- redis默认将所有的数据存储在内存中，所以数据大小超过内存大小时redis无法容纳所有的数据
- redis事物不完全符合ACID规范，如果需要完全符合ACID规范的事物，就不能使用redis


















