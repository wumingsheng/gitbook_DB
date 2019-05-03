# mysql本地安装

## ubuntu安装mysql


```
sudo apt-get install mysql-server -y
sudo apt-get install mysql-client -y 
sudo apt-get install libmysqlclient-dev
systemctl status mysql
```

初始用户名和密码

进入到`etc/mysql`目录下，查看`debian.cnf`文件

```
$ sudo cat debian.cnf 
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = q47yDLlQ7ssi4T5I
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = q47yDLlQ7ssi4T5I
socket   = /var/run/mysqld/mysqld.sock

```
找到用户名和密码，用此账号登录mysql

```
mysql -u debian-sys-maint -p

```

![](/assets/20190503152935.png)

## centos安装mysql

```bash
# 获取yum源
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
# 安装yum源
yum -y install mysql57-community-release-el7-10.noarch.rpm
# 安装mysql
yum -y install mysql-community-server

systemctl start  mysqld.service
systemctl status mysqld.service

# 查看root密码
grep "password" /var/log/mysqld.log
# 进入数据库
mysql -uroot -p     # 回车后会提示输入密码
# 修改root密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# 将密码规则设置的简单一些
mysql> set global validate_password_policy=0;
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global validate_password_length=1;
Query OK, 0 rows affected (0.00 sec)
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# 因为安装了Yum Repository，以后每次yum操作都会自动更新，需要把这个卸载掉：
yum -y remove mysql57-community-release-el7-10.noarch
# 可视化工具的登录授权：(如果授权不成功，请查看防火墙)
grant all on *.* to root@'%' identified by '123456';
```

## 安装mysql

```bash
# 方式一
sudo apt-get install mycli -y


# 方式二

sudo apt-get install python-pip
pip install mycli


```



