# redis中lua脚本演示

![](/assets/20190512102144.png)

## `call` 和 `pcall` 的区别

1. call不会捕获异常直接把异常声明出去了，如果有异常，直接抛出，后面的代码不会执行
2. pcall会捕获异常，我们可以处理也可以不处理，但是不影响后面代码命令的执行


正常执行
```
127.0.0.1:6379> EVAL "redis.call('set', 'name1', 'value1'); redis.call('set', 'name2', 'value2');
redis.call('set', 'name3', 'value3')" 0
(nil)
127.0.0.1:6379> keys *
1) "name3"
2) "name2"
3) "name1"
127.0.0.1:6379> flushdb
OK
```

使用call,如果中间有异常，异常之前的命令正常执行，异常之后的命令不再执行了

可以看到执行代码后直接跑出了异常，但是异常之前的命令还是被执行了

```
127.0.0.1:6379> EVAL "redis.call('set', 'name1', 'value1'); redis.call('rset', 'name2', 'value2');
redis.call('set', 'name3', 'value3')" 0
(error) ERR Error running script (call to f_020cd072720d080c788502f85dd4841a30650a43):
@user_script:1: @user_script: 1: Unknown Redis command called from Lua script 
127.0.0.1:6379> keys *
1) "name1"
127.0.0.1:6379> get name1
"value1"


```

使用pcall，可以看到并没有抛异常出来，并且异常之后的代码还会继续执行

第二行代码有问题，但是第一行和第三行代码还是正常执行了

```

127.0.0.1:6379> EVAL "redis.pcall('set', 'name1', 'value1'); redis.pcall('rset', 'name2', 'value2');
redis.pcall('set', 'name3', 'value3')" 0
(nil)
127.0.0.1:6379> keys *
1) "name3"
2) "name1"

```










