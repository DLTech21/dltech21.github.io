---
title: springboot微服务打包部署
date: 2018-05-04 16:28:53
tags: [springboot, 部署, ssh]
---

部署比较简单一般分为两种；一种是打包成jar包直接执行，另一种是打包成war包放到tomcat服务器下。

### 打jar包
使用的是maven来管理项目，执行以下命令

```bash
cd 项目根目录（与pom.xml同级）
mvn clean package
```

打包完成后jar包会生成到target目录下
<!--more-->

启动jar包命令

```bash
java -jar  xxx.jar
```
上面的方式，只要控制台关闭，服务也就不能访问了，不建议采用。下面我们使用在后台运行的方式来启动:

```bash
nohup java -jar target/spring-boot-scheduler-1.0.0.jar &
```

### 如何重启

### 简单粗暴
直接kill掉进程再次启动jar包

```bash
ps -ef|grep java 
##拿到对于Java程序的pid
kill -9 pid
## 再次重启
Java -jar  xxxx.jar
```
当然这种方式比较传统和暴力，所以建议大家使用下面的方式来管理

### 脚本执行
使用的是maven,需要包含以下的配置

```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

启动方式


```bash
./yourapp.jar
```

ps:
mac传输jar到linux的命令（带端口）

```bash
scp -P 端口 target/xxx.jar root@ip:/root/
```
