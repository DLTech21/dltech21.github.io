---
title: 替换aar里面的资源文件
date: 2018-05-03 18:29:00
tags: [Android, aar]
---
在开发过程中会用到一些第三方提供的aar，里面有资源跟项目的资源冲突或者需要修改替换，更有的需要hack里面的代码，这些需要先把aar解压，修改后再重新打包成新的aar。
具体命令

```
$ unzip myLib.aar -d tempFolder # or other extracting tool
# Change whatever you need

$ jar cvf myNewLib.aar -C tempFolder/ .
```


至于hack里面的代码，简单的可以参考替换class的方法
[https://juejin.im/entry/5805dc4f570c35006b7af21d](https://juejin.im/entry/5805dc4f570c35006b7af21d)

至于通过修改二进制来修改可以通过ida+python，这里以后再详说

