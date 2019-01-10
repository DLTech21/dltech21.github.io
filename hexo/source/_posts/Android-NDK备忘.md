---
title: Android NDK备忘
date: 2018-09-26 16:19:04
tags: [NDK,jni]
---

###NDK支持C++ 11/14方法
在 Android.mk 中加入

```
LOCAL_CPPFLAGS += -std=c++11

```
or

```
LOCAL_CPPFLAGS += -std=c++1y
```

<!-- more -->


###jstring to string 

```c++
const char *pInData = env->GetStringUTFChars(jstr, NULL);
std::string mapStr = std::string(pInData);
env->ReleaseStringUTFChars(jstr, pInData);
```

###string to jbyte
```c++
std::string after = signStringM(mapStr);
jbyteArray carr = env->NewByteArray(after.length());
env->SetByteArrayRegion(carr,0,after.length(),(jbyte*)after.c_str());
```


###JNI学习笔记
[RegisterNatives注册原生方法](http://jellypaul.github.io/java/2016/11/15/JNI-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-%E9%80%9A%E8%BF%87RegisterNatives%E6%B3%A8%E5%86%8C%E5%8E%9F%E7%94%9F%E6%96%B9%E6%B3%95.html)

[RegisterNatives注册原生方法2](https://blog.csdn.net/bigapple88/article/details/6756204)

[JNI学习笔记](http://jellypaul.github.io/java/2016/08/08/JNI%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.html)

[JNI学习笔记1](https://blog.csdn.net/shaosunrise/article/details/79838297)

