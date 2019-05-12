# redis的LUA脚本编程


![](http://www.lua.org/images/luaa.gif)


## 1. 资源

* [官方](http://www.lua.org/)
    * [官方下载地址](http://luabinaries.sourceforge.net/download.html)
* [redis中文网](http://redis.cn)
    * [redis中文网lua脚本演示教程](http://redis.cn/commands/eval.html)
* [IDE-studio.zerobrane](https://studio.zerobrane.com/)
    * [官方下载页面](https://studio.zerobrane.com/download?not-this-time)

## 2. 安装

```bash
LUA_HOME=/home/user/Downloads/lua-5.3.5_Linux319_64_bin
PATH=$PATH:$LUA_HOME
export PATH
```

## 3. 为什么使用lua

1. 可以将命令缓存到redis服务端，减少网络开销
2. lua脚本命令的执行具有原子性，可以将命令一起发送到服务端一起执行

> 可以免去事物，避免出现多线程读取数据不正确







