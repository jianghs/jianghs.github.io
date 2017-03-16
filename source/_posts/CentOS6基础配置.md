---
title: CentOS基础配置
date: 2017-03-16 13:52:45
tags: Linux
categories:
---
### 下载、安装及配置jdk
1、在线下载 jdk
```
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u91-b14/jdk-8u91-linux-x64.rpm
```

2、rpm 安装 jdk
```
rpm -ivh jdk-8u91-linux-x64.rpm
```

3、环境变量配置

编辑 profile 文件，在文件末尾添加以下：
```
export JAVA_HOME=/usr/java/jdk1.8.0_121
export CLASSPATH=,:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH

```

4、使 profile 生效
```
source /etc/profile
```

5、验证
```
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
  ```
  chmod 755 ***
  ```
* Exec format error  
Java 环境变量配置  
Linux 系统分隔符为英文冒号 ：，而 Windows 系统分隔符为分号 ; ，此处需区分明确。

### 下载、安装及配置 TomCat
1、下载 TomCat7
```
[root@iZ0xi0lndyippx6f13iwviZ software]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-7/v7.0.75/bin/apache-tomcat-7.0.75.tar.gz
```

2、解压 TomCat7
```
[root@iZ0xi0lndyippx6f13iwviZ software]# tar -zxv -f apache-tomcat-7.0.75.tar.gz
```

3、运行 TomCat

进入 TomCat 安装目录bin文件夹
```
sh startup.sh
```

4、设置开机自启动

编辑rc.local
```
[root@iZ0xi0lndyippx6f13iwviZ rc.d]# vim /etc/rc.d/rc.local
```
在末尾加入以下
```
JAVA_HOME = /usr/java/jdk1.8.0_121
export JAVA_HOME  
/usr/software/apache-tomcat-7.0.75/bin/startup.sh
```

### 部署
1、
