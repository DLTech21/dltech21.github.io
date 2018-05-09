---
title: React Native问题备忘
date: 2018-05-09 13:43:33
tags: [React Native, 技巧]
---


本篇用于记录React Native踩过的坑，不定期更新

[学习文章](https://juejin.im/post/5898388b128fe1006cb943e3)

### unable to load script from assets index.android.bundle
解决方案

in project directory

```
1、mkdir android/app/src/main/assets
2、react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
3、react-native run-android
```