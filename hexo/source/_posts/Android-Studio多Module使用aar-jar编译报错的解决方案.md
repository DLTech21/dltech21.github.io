---
title: Android Studio多Module使用aar/jar编译报错的解决方案
date: 2018-04-24 16:12:57
tags: [Android, aar, jar]
---
## 多Module中使用aar编译报错

在开发Android App的时候，如果项目比较复杂，都可能会有很多个 <code class="highlighter-rouge">library</code>  工程，考虑如下场景：

Module A：Library

Module B：Library

Module C：App

<!--more-->

<p>其中 <code class="highlighter-rouge">Module B</code> 依赖 <code class="highlighter-rouge">Module A</code>，<code class="highlighter-rouge">Module C</code> 依赖 <code class="highlighter-rouge">Module A</code>，<code class="highlighter-rouge">Module B</code>。</p>

如果在 Module A 中添加了aar库的话，则编译的时候，可能会报错，例如在 Module A 中使用了gif库， 对应的引用代码是：

```groovy
compile(name: 'face', ext: 'aar')
```

在编译的时候，错误信息可能如下所示：

```groovy
Failed to resolve: face
```

Module A 是可以正常编译通过的，但是编译 Module C 的时候，就会发现找不到对应的 gif 库，因为在 Module A 中指定的 libs 目录是相对的，在 Module C 中的 **build.gradle** 只会从自己所在目录下面的 libs 中查找，结果是找不到 gif 相关的 aar 库，所以就会编译出错。

## 解决方案
### 方案一
所有依赖 Module A 的 Module 都添加：

```groovy
repositories {
    flatDir {
        dirs 'xxx/libs' // Module A的libs的目录地址
    }
}
```

该方案需要每一个依赖Module A的Module都需要这样操作，较为麻烦。

### 方案二
在根目录的**build.gradle** 的**repositories**添加相应的引用

```groovy
allprojects {
    repositories {
        jcenter()
			//添加代码
        flatDir {
            dirs project(':ModuleA').file('libs')  
        }
    }
}
```

方案二比较完美地解决了多 module 引用 aar 库的问题，推荐使用这种方法

## 多Module中使用jar编译报错

解决方法: 将任意一个Module中的jar依赖为compile files('your jar name')，其他需要依赖的地方改为provided files('your jar name')。即可  下面详细介绍为什么这样做以及案例

## 案例介绍

如 Module A和自己Module B都要用到定位sdk

1、在Module A的gradle中以compile引入如：

```groovy
compile files('libs/AMap_Location_V2.4.1_20160414.jar')
```

2、在Module B的gradle中以provided的方式引入如：

```groovy
provided files('libs/AMap_Location_V2.4.1_20160414.jar')
```


3、而且Module B的gradle中不能存在

```groovy
compile fileTree(include: ['*.jar'], dir: 'libs')
```