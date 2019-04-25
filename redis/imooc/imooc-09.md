# zset数据结构

## redis的数据结构－sorted set

sorted set和set区别？

在set的基础上，为每一个元素关联一个分数，元素是不可重复的，唯一的，但是分数是可以重复的

sorted set和list比较

相同点：
1. 二者都是有序的
2. 二者都可以获取某一个范围的元素
不同点：
1. list是列表，sorted set是散列表，sorted set不仅向两端数据增删快，向中间数据增删也快，所以sorted set更消耗内存
2. 列表中不能简单的调整某个元素的位置，但是有序集合可以（通过更改分数实现）
3. 有序集合要比列表类型更耗内存。 向有序集合中加入一个元素和该元素的分数，如果该元素已经存在则会用新的分数替换原有的分数。返回值是新加入到集合中的元素个数，不包含之前已经存在的元素。

sorted set（有序唯一） = list（有序不唯一） + set(无序唯一)

### 应用场景

- 积分排行榜
- 构建索引数据


### 常用命令

```java
//添加单个元素
jedis.zadd("user", 1, "a");
//添加多个元素
Map<String,Double> map = new HashMap<>();
map.put("b", 2d);
map.put("c", 3d);
map.put("d", 4d);
jedis.zadd("user", map);
//获取某个元素的分数
Double zscore = jedis.zscore("user", "a");
//元素的数量
Long zcard = jedis.zcard("user");
//删除某一个元素
jedis.zrem("user", "a");
//按照分数从小到大排序　-1　代表最后一个元素　-2代表倒数第二个元素．．．依次类推
Set<String> zrange = jedis.zrange("user", 0, -1);
//按照分数从大到小反向排序
Set<String> zrange = jedis.zrevrange("user", 0, -1);
//按照分数顺序排列带分数[[[97],1.0], [[98],2.0], [[99],3.0], [[100],4.0]]
Set<Tuple> zrangeWithScores = jedis.zrangeWithScores("user", 0, -1);
//带分页排序（类似mysql中limit offset count）
jedis.zrangeByScore("user", 1, 2, 0, 2);
//反向排序带分数
Set<Tuple> zrevrangeWithScores = jedis.zrevrangeWithScores("user", 0, -1);
//按照索引的范围删除
Long zremrangeByRank = jedis.zremrangeByRank("user", 0 ,1);
//按照分数的范围删除
Long zremrangeByRank = jedis.zremrangeByScore("user", 0, 10);
//获取元素的排名
Long zrank = jedis.zrank("user", "c");
//设置分数增加
Double zincrby = jedis.zincrby("user", 20, "a");
//分数在某一个区间元素的个数
Long zcount = jedis.zcount("user", 1, 20);
```