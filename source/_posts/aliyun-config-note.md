---
title: 从零配置阿里云服务器的Java运行环境
date: 2018-04-10 16:50:32
tags:
- aliyun
categories:
- knowledge repo
---

1. 安装PCRE
```
sudo apt-get update 
sudo apt-get install libpcre3 libpcre3-dev 
```

2. 安装zlib
```
./configure
make test
make install
```

3. 安装nginx，[http://www.cnblogs.com/kunhu/p/3633002.html](http://www.cnblogs.com/kunhu/p/3633002.html)
```
./configure --prefix=/usr/local/nginx
make
make install
```

4. 安装mysql， [下载地址](https://dev.mysql.com/downloads/mysql/) 
```
# https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html
shell> groupadd mysql
shell> useradd -r -g mysql -s /bin/false mysql
shell> cd /usr/local
shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
shell> ln -s full-path-to-mysql-VERSION-OS mysql
shell> cd mysql
shell> mkdir mysql-files
shell> chown mysql:mysql mysql-files
shell> chmod 750 mysql-files
shell> bin/mysqld --initialize --user=mysql 
shell> bin/mysql_ssl_rsa_setup              
shell> bin/mysqld_safe --user=mysql &
# Next command is optional
shell> cp support-files/mysql.server /etc/init.d/mysql.server

# http://www.cnblogs.com/rainisic/archive/2012/05/18/mysql_5_5_install.html
```

5. mysql修改密码
```
killall -TERM MySQLd
./bin/mysqld_safe --skip-grant-tables &
MySQL -u root 

update MySQL.user set authentication_string=PASSWORD('新密码') where User='root';
flush privileges;
quit;
```

6. ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```
set password = password('root');
```

7. mysql远程连接报错1130, 无法给远程连接的用户权限问题。
```
# 修改user表的host字段为%后，重启服务
update user set host = '%' where user ='root';
flush privileges;
```

8. 启动jar包
```
# 后台启动
nohup java -jar xxx.jar &
# 查看后台job
jobs
# 切换到某个进程
fg 1
```

9. java.lang.UnsatisfiedLinkError:no net in java.library.path报错原因有可能是因为jar包损坏，拷贝、解压时有错误。

10. mybatis调用两次user = mapper.selectOne(user)，如果user之后赋值过新值，则user再次查询为null，但是实际执行语句和参数都没有任何问题，完全可以查出结果集。
这个的原因竟然是因为jdbc连接时的编码问题，只要在配置文件中的url里增加 `&characterEncoding=utf8`即可解决，具体原因未明。

11. nginx 出现413 Request Entity Too Large问题的解决方法
在`http{}`段中加入 `client_max_body_size 20m;` 20m为允许最大上传的大小。

12. 查看进程和杀死进程
```
ps -ef | grep java
kill -9 [pid]
```