---
title: CentOS基础配置
date: 2017-03-16 13:52:45
tags: Linux
categories:
---
# CentOS基础配置

## 下载、安装及配置jdk

1、在线下载 jdk

```bash
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.rpm
```

2、rpm 安装 jdk

```bash
rpm -ivh jdk-8u91-linux-x64.rpm
```

3、环境变量配置

编辑 profile 文件，在文件末尾添加以下：

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_121
export CLASSPATH=,:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH

```

4、使 profile 生效

```bash
source /etc/profile
```

5、验证

```bash
[root@iZ0xi0lndyippx6f13iwviZ software]# java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)

```

6、Q&A

* *** permission denied

对应文件没有执行权限,修改成可以执行。
7：用户可读，可写，可运行
5：用户组可读，可写，不可运行

  ```bash
  chmod 755 ***
  ```
* Exec format error

Java 环境变量配置
Linux 系统分隔符为英文冒号 ：，而 Windows 系统分隔符为分号 ; ，此处需区分明确。

### 下载、安装及配置 TomCat

1、下载 TomCat7

```bash
[root@iZ0xi0lndyippx6f13iwviZ software]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-7/v7.0.75/bin/apache-tomcat-7.0.75.tar.gz
```

2、解压 TomCat7

```bash
[root@iZ0xi0lndyippx6f13iwviZ software]# tar -zxv -f apache-tomcat-7.0.75.tar.gz
```

3、运行 TomCat

进入 TomCat 安装目录bin文件夹

```bash
sh startup.sh
```

4、设置开机自启动

编辑rc.local

```bash
[root@iZ0xi0lndyippx6f13iwviZ rc.d]# vim /etc/rc.d/rc.local
```

在末尾加入以下

```bash
JAVA_HOME = /usr/java/jdk1.8.0_121
export JAVA_HOME
/usr/software/apache-tomcat-7.0.75/bin/startup.sh
```

5、 部署

将项目打包成 war 包。
将 war 包放在 tomcat webapps 目录下即可。

### 下载、安装及配置 MySql

1、下载及安装

由于 CentOS 中集成了 MySQL ,我们可以用如下命令查看是否安装了 MySQL 数据库：

```bash
[root@li1600-30 /]# rpm -qa | grep mysql
```

如果能查到结果，可以删除后自行安装。
如果不删除，启动服务时，提示未识别的服务。
经确认需安装 mysql server ,命令如下：

```bash
yum install mysql-server
```

后续再次执行：

```bash
[root@li1600-30 /]# service mysqld start
Initializing MySQL database:  Installing MySQL system tables...
OK
Filling help tables...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/bin/mysqladmin -u root password 'new-password'
/usr/bin/mysqladmin -u root -h li1600-30.members.linode.com password 'new-password'

Alternatively you can run:
/usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems with the /usr/bin/mysqlbug script!

                                                           [  OK  ]
Starting mysqld:                                           [  OK  ]
```

2、开机启动设置

通过如下命令确认mysqld服务是不是开机自动启动：

```bash
[root@li1600-30 /]# chkconfig --list | grep mysqld
mysqld         	0:off	1:off	2:off	3:off	4:off	5:off	6:off
```

我们发现 mysqld 并没有开机启动，可以将其设置为开机启动：

```bash
[root@li1600-30 /]# chkconfig mysqld on
```

3、设置
数据库安装完后只会有一个 root 账号，并且未设置密码。
可以通过如下命令设置密码：

```bash
[root@li1600-30 /]# mysqladmin -u root password 'root'
```

现在就可以通过 root 用户登录数据库了。

```bash
[root@li1600-30 /]# mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

开启远程登录

查询 user 表，发现 root 用户对应的 host 只允许本地。

```bash
mysql> SELECT User, Password, Host FROM user;
+------+-------------------------------------------+------------------------------+
| User | Password                                  | Host                         |
+------+-------------------------------------------+------------------------------+
| root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B | localhost                    |
| root |                                           | li1600-30.members.linode.com |
| root |                                           | 127.0.0.1                    |
|      |                                           | localhost                    |
|      |                                           | li1600-30.members.linode.com |
+------+-------------------------------------------+------------------------------+
5 rows in set (0.00 sec)

```

下面我们添加一个新的 root 用户，密码为空，允许任意 IP 连接，后续刷新缓存，后续即可远程连接。

```bash
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT User, Password, Host FROM user;
+------+-------------------------------------------+------------------------------+
| User | Password                                  | Host                         |
+------+-------------------------------------------+------------------------------+
| root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B | localhost                    |
| root |                                           | li1600-30.members.linode.com |
| root |                                           | 127.0.0.1                    |
|      |                                           | localhost                    |
|      |                                           | li1600-30.members.linode.com |
| root |                                           | %                            |
+------+-------------------------------------------+------------------------------+
6 rows in set (0.00 sec)

mysql> flush privileges;
```

### 结语

至此，完成 jdk 安装， tomcat 配置及工程部署， MySQL 数据库配置。
