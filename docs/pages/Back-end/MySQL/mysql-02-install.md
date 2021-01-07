## MySQL 解压安装  <!-- {docsify-ignore} -->

卸载系统自带的Mariadb

```bash
[root@zhangxiaocai ~]# rpm -qa|grep mariadb
mariadb-libs-5.5.44-2.el7.centos.x86_64
[root@zhangxiaocai ~]# rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64
```

删除etc目录下的my.cnf文件

```bash
[root@zhangxiaocai ~]# rm /etc/my.cnf
rm: cannot remove ?etc/my.cnf? No such file or directory
```
检查mysql是否存在

```bash
[root@zhangxiaocai ~]# rpm -qa | grep mysql
[root@zhangxiaocai ~]# 
```
检查mysql组和用户是否存在，如无创建

```bash
[root@zhangxiaocai ~]# cat /etc/group | grep mysql 

[root@zhangxiaocai ~]#  cat /etc/passwd | grep mysql
```

创建mysql用户组，并设置密码

```bash
#创建mysql用户组
[root@zhangxiaocai ~]# groupadd mysql

#创建一个用户名为mysql的用户并加入mysql用户组
[root@zhangxiaocai ~]# useradd -g mysql mysql

#制定password 这里设置为111111
[root@zhangxiaocai ~]# passwd mysql
Changing password for user mysql.
New password: 
BAD PASSWORD: The password is a palindrome
Retype new password: 
passwd: all authentication tokens updated successfully.
```

一般安装到 `/usr/local` 目录，这里记录安装到`/var`

```bash
# 解压
[root@zhangxiaocai var]# tar -zxvf mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz 

# 重命名文件夹
[root@zhangxiaocai var]# mv mysql-5.7.19-linux-glibc2.12-x86_64/ mysql57

```bash
#更改所属的组和用户
[root@zhangxiaocai var]# chown -R mysql mysql57/

[root@zhangxiaocai var]# chgrp -R mysql mysql57/

[root@zhangxiaocai var]# cd mysql57/
# 创建数据目录
[root@zhangxiaocai mysql57]# mkdir data
# 授与用户权限权
[root@zhangxiaocai mysql57]# chown -R mysql:mysql data
```

在etc下新建配置文件my.cnf，并在该文件内添加以下配置

```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
skip-name-resolve
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=/var/mysql57
# 设置mysql数据库的数据的存放目录
datadir=/var/mysql57/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
lower_case_table_names=1
max_allowed_packet=16M
```

安装和初始化


```bash
[root@zhangxiaocai mysql57]# bin/mysql_install_db --user=mysql --basedir=/var/mysql57/ --datadir=/var/mysql57/data/
2017-04-17 17:40:02 [WARNING] mysql_install_db is deprecated. Please consider switching to mysqld --initialize
2017-04-17 17:40:05 [WARNING] The bootstrap log isn't empty:
2017-04-17 17:40:05 [WARNING] 2017-04-17T09:40:02.728710Z 0 [Warning] --bootstrap is deprecated. Please consider using --initialize instead
2017-04-17T09:40:02.729161Z 0 [Warning] Changed limits: max_open_files: 1024 (requested 5000)
2017-04-17T09:40:02.729167Z 0 [Warning] Changed limits: table_open_cache: 407 (requested 2000)
```

赋权限

```bash
# 复制进程服务
[root@zhangxiaocai mysql57]# cp ./support-files/mysql.server /etc/init.d/mysqld

# 赋权限
[root@zhangxiaocai mysql57]# chown 777 /etc/my.cnf 

# 允许执行
[root@zhangxiaocai mysql57]# chmod +x /etc/init.d/mysqld
```

重启mysql服务

```bash
[root@zhangxiaocai mysql57]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```

#设置开机启动

```bash
[root@zhangxiaocai mysql57]# chkconfig --level 35 mysqld on
[root@zhangxiaocai mysql57]# chkconfig --list mysqld

[root@zhangxiaocai mysql57]# chmod +x /etc/rc.d/init.d/mysqld

[root@zhangxiaocai mysql57]# chkconfig --add mysqld
[root@zhangxiaocai mysql57]# chkconfig --list mysqld

[root@zhangxiaocai mysql57]# service mysqld status
 SUCCESS! MySQL running (4475)
```

```bash
vi etc/profile/
```

```
export PATH=$PATH:/var/mysql57/bin
```

让配置生效

```bash
[root@zhangxiaocai mysql57]# source /etc/profile
```

获得初始密码

```bash
[root@zhangxiaocai bin]# cat /root/.mysql_secret  
# Password set for user 'root@localhost' at 2020-04-17 17:40:02 
p8Uhdk*1j
```
 

修改密码

```bash
[root@zhangxiaocai bin]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.18

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set PASSWORD = PASSWORD('111111');
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```

添加远程访问权限


```bash
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select host,user from user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | root      |
| localhost | mysql.sys |
+-----------+-----------+
2 rows in set (0.00 sec)
```

```
create user 'xxx'@'%' identified by '123';  
这里 @‘%’ 表示在任何主机都可以登录
```
 
重启生效

```bash
/bin/systemctl restart  mysql.service

[root@zhangxiaocai bin]# /etc/init.d/mysqld restart 
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```