---
title: 利用bintray-release上传自己的library
date: 2018-04-13 10:27:20
tags: Android
---

前提条件：申请一个bintray的账号、一个library库

直接进入主题

###根目录下的build.gradle中加入上传开源库的依赖：
```
classpath 'com.aaron.gradle:bintray-release:1.3.1'

```

###library的build.gradle中加入 apply
```
apply plugin: 'com.aaron.gradle.bintray-release'
```

<!-- more -->

###library的build.gradle添加
```
publish {
    userOrg = 'dltech21' //bintray的username
    groupId = 'io.github.dltech21' 
    artifactId = 'pdfgo'
    publishVersion = '1.0.0'
    desc = 'Pdf multiple render page for Android'
    website = 'https://github.com/DLTech21/PdfRender-PdfGo'
}
```

###最后，利用 task bintrayUpload 完成上传
```
./gradlew bintrayUpload -PbintrayUser=你的username -PbintrayKey=你的Apikey -PdryRun=false
```

###见证奇迹的时候
![](/assets/img/bintrayupload.png)

最后在bintray上面看到自己的library,例如我的[pdfgo](https://bintray.com/dltech21/maven/pdfgo)

至此library已经可以引用自己的库了，只要在根目录下的build.gradle加上自己maven地址

```
maven { url 'https://dl.bintray.com/dltech21/maven'}
```

然后在app的build中加上引用即可

```
compile 'io.github.dltech21:pdfgo:1.0.0'
```



当然如果能够提交到JCenter就更好了，不再需要定义自己maven仓库地址，直接compile即可。
进入项目页，点击Add to JCenter，然后坐等jcenter的审核通过
![](/assets/img/bintraysync.png)




