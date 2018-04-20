---
title: Jenkins持续集成Android项目
date: 2018-04-20 10:04:59
tags: [Jenkins,Android,Mac]
---

[Jenkins](http://jenkins-ci.org/)是一个基于Java的开源的CI项目。它包括持续的软件版本测试/发布，监控外部调用执行的工作等...

在开发Andorid项目时，常常需要build新的APK,给内部人员或者外部人员测试使用,还有就是运行单元测试等，另外执行gradle assembleRelease本身又比较费时。所以借助Jenkines完成自动打包，发布的工作是一个不错的选择。

## 安装Jenkins

通常有两种：

1. tomcat+Jenkins.war
2. 下载安装文件进行安装

我个人推荐第一种简单快速，并且替换版本方便

开启tomcat后打开链接[http://localhost:8080/jenkins/](http://localhost:8080/jenkins/)进行开始引导安装和基本配置，这些默认就可以了。
<!-- more -->
## 集成android持续打包

### 配置Android SDK 

打开[系统配置](http://localhost:8080/jenkins/configure)，添加环境变量Android_Home

![](/assets/img/jenkinsconfig.png)

ps：其他教程一般都把gradle也配置，这个我觉得不怎么友好，因为gradle版本一旦发生变化还要再配置

### 创建任务并配置
创建一个新任务，选择构建一个自由风格的软件项目
![](/assets/img/jenkinstask.png)

配置git:
![](/assets/img/jenkinsgit.png)

增加构建步骤选择Execute shell
![](/assets/img/jenkinsshell.png)

这样编写shell脚本进行命令打包
```
/Users/gdca/Documents/nn/gradle-3.3/bin/gradle assembleDebug
```
or
```
/Users/gdca/Documents/nn/gradle-3.3/bin/gradle assembleRelease
```

构建后操作：选择archive the artifacts：
输入:example/build/outputs/apk/*.apk用来输出编译的Apk文件。

保存配置，这样一个简单的构建任务就配置完成

###  开始构建

点击build并等待构建成功:
![](/assets/img/jenkinsbuild.png)
成功了构建出了Apk文件。

至于构建成功如何生成二维码，下一次再详说