---
title: mac sed命令学习
date: 2018-04-13 14:20:07
tags: [mac,sed,Jenkins]
---

最近在研究利用Jenkins打包Android的SDK以及Demo，由于SDK是以aar（版本会根据每次打包改变）为输出方式，那么demo就需要修改app模块的gradle脚本，随之而来的问题就是如何写shell脚本去修改工作区里的build.gradle。

在网上搜索一圈后，暂时采用下面的方式来修改

```bash
#! 替换demo里面app的build.gradle的aar引用
#! mac 在关键字一行后添加新的一行,gdcasdk为目标aar的前缀
sed -i '' '/'"compile(name: 'gdcasdk"'/a\
	tempdfds34\
' build.gradle
#! mac delete关键字一行
sed -i '' '/'"compile(name: 'gdcasdk"'/d' build.gradle

sed -i '' 's/tempdfds34/'"compile(name: '${aarNameforCompile}', ext: 'aar')"'/g' build.gradle
```

上面的思路就是通过替换的方式来修改，如有更简短的shell命令请告知，谢谢

ps：可以采用[python](https://dltech21.github.io/2018/04/26/Jenkins-python%E6%AD%A3%E5%88%99%E4%BF%AE%E6%94%B9%E5%86%85%E5%AE%B9/)