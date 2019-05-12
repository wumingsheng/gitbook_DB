# redis中lua常用命令


## 1. 缓存脚本

生成一个SHA1值，这个值就是脚本字符串被加密后的值

1. 防止被偷窥
2. 如果脚本很长的话，用起来不是很方便，我们可以将这个脚本缓冲在服务器中，客户端只需要通过加密字符串调用即可


>（ 不管脚本有多长，经过SHA1加密后就是那么长）




```

127.0.0.1:6379> script load "return redis.call('get', 'n1')"
"70d11e3536c936ce819daa2f84d2ee2b814eb475"
127.0.0.1:6379> 

```
调用缓冲的脚本


```

127.0.0.1:6379> set n1 v1
OK
127.0.0.1:6379> get n1
"v1"
127.0.0.1:6379> evalsha 70d11e3536c936ce819daa2f84d2ee2b814eb475 0
"v1"

```


* SCRIPT EXISTS 查看脚本是否存在`SCRIPT EXISTS SHA1`
* SCRIPT FLUSH 删除所有的缓存脚本
* SCRIPT KILL 终止正在执行的脚本


https://github.com/pkulchenko/ZeroBranePackage/blob/master/redis.lua



![](/assets/20190512131508.png)

-------

![](/assets/20190512131649.png)




































