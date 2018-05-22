---
title: 开发Android SDK使用fat-aar踩坑记
date: 2018-05-18 17:55:05
tags: [Android, SDK, fat-aar, aar]
---

### 出现No static field xxx of type I in class /R$layout的错误

检查是否存在资源重复，如果是v7包的，尽量考虑不引用v7包；如果是v4包，把compile改为provided；如果是重复embedded同一个子aar，那么就需要修改保证输出的aar只embedded了一次子aar
