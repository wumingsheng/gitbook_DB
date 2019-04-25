# 【传智播客】MongoDB数据备份


数据备份与恢复使用的是`mongodump`和`mongorestore`命令

## 备份数据库


```
./mongodump -h 127.0.0.1:27017 -d test -o /opt/mongodb/data -u test -p 123456
```

    -h --host --port
    -d --db=<database-name>
    -o --out=<directory-path>
    -u --username
    -p --password
    

用户名和密码只有对这个数据库有读写权限即可


## 恢复数据库

```
[root@master bin]# ./mongorestore -h 127.0.0.1:27017 -d test -u test -p 123456 /opt/mongodb/data/test
```
    -h 指定ip和port
    -d 指定数据库
    -u 指定用户名
    -p 指定密码
    