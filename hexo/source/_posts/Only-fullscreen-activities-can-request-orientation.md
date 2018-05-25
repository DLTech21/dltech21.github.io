---
title: Only fullscreen activities can request orientation?
date: 2018-05-24 18:06:38
tags: [Android]
---

前两天某个使用我开发的SDK，告诉我SDK有bug，我说怎么可能会有bug，然后甩给我错误日志

```
java.lang.RuntimeException: Unable to start activity 
ComponentInfo{com.linkedin.android.XXXX.XXXX/
com.linkedin.android.XXXX.XXXX.activity.LoginActivity}: 
java.lang.IllegalStateException: Only fullscreen activities can 
request orientation
```

然后经过我的深入了解，发现这个时Android官方的bug：在API26的设备跑在targetSdkVersion27且当在一个“translucent”的Activity里，试图执行setRequestedOrientation或者在设置android:screenOrientation="portrait"，
这样就会出现上述的bug。

简单的总结就是官方想限制不是全屏的activity就不要设置方向了，但是自己写的代码不完善然后就出现了这个坑。

由于我给别人的Demo是targetSdkVersion26，所以就不会留意到这个”坑”~~~

<!--more-->
### 埋坑方法：
1、改变targetSdkVersion，低于27

2、去掉android:screenOrientation="portrait"，这个方法相当简单粗暴，视乎activity是否需要强制竖屏。

3、增加每个activity的theme，让activity是全屏


