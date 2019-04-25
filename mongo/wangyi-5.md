# 【网易云课堂】MongoDB深入操作


## 1、固定集合

所谓固定集合值的是规定集合的大小，如果要保存的内容已近超过了集合的长度，那么会采用最近最少使用的原则，将最早的数据移除保存新的数据

默认情况下，一个集合可以使用createCollection()函数创建，或者使用添加数据自动创建，如果使用固定集合
必须明确的创建一个空集合

创建一个空集合
```
db.createCollection("user",{"capped":true,"size":1024,"max":5})
```
"capped":true 表示是一个固定集合，"size":1024表的最大容量是1024个字节，"max":5表示最大的数据量是5条

普通集合可以通过 convertToCapped将普通集合转为固定集合

`db.runCommand({convertToCapped:”test_2”,size:10,max:3})`




## 2、GridFS

在MongoDB中支持大数据的存储，例如图片，音乐等二进制数据，但是这个做法需要用户自己进行处理，使用mogofiles命令完成

1、 利用命令行进入到所在的路径下
2、 mongofiles put photo.jpg
3、 查看保存的文件 mongofiles list
4、 show dbs数据库没有太大的变化
5、 db.fs.files.find()在数据库中查看
6、 删除文件mongofiles delete photo.jpg

## 3、用户管理

在MongoDB中默认情况下，只要进行链接可以不使用用户名和密码，因为要想让其起作用，必须有一下两个条件：

1、 服务器启动的时候，打开授权认证

#security
auth=true

2、 需要配置用户名和密码

但是需要明确的是，如果要想配置用户名和密码一定要针对一个数据库，必须首先切换到数据库上，
`use test`
执行用户的创建（u:hello p:java）
任何一个用户必须要有自己的角色，对于角色最基础的是read，readwrite

```
db.createUser({

"user":"hello",
"pwd":"java",
"roles":[{"role":"readWrite","db":"test"}]

})

```
如果想让此用户名起作用，必须以授权的方式启动服务

登录数据库使用用户名和密码
`mongo -u hello -p java`

如果要修改密码，就关闭授权登录，重启服务，修改密码

## 4、常用到的角色介绍：

* Read：允许用户读取指定数据库
* readWrite：允许用户读写指定数据库
* dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
* userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
* clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
* readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
* readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
* userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
* dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
* root：只在admin数据库中可用。超级账号，超级权限



1.在数据库安装成功的基础上连接上客户端输入如下指令：

     use admin

     db.createUser(
       {
         user: "root",
         pwd: "root",
         roles: [ { role: "root", db: "admin" } ]
       }
     )

注意：创建超级管理员成功

2.创建读写用户指令如下：

    use test
    db.createUser(
      {
        user: "test",
        pwd: "123456",
        roles: [ { role: "readWrite", db: "test" } ]
      }
    )














