---
title: Bugly热更新及Walle使用总结
date: 2018-04-14 14:35:08
tags: [Bugly,hotfix,Walle]
---

由于一次的紧急hotfix体验：采用gradle的多渠道打包+bugly的热更新，在打基准包耗时10mins+，修改代码2mins内，打补丁包15mins+，这样一个槽糕的hotfix让我再也无法忍受gradle的多渠道打包的低效率，从而使用Walle。

Walle是美团开源的多渠道打包方案，之前一直都有留意，这次终于下定决心进行改造。[bugly集成方法](https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20180119105842)(官方的很详细了,这里就不啰嗦了)

## 1. 集成Walle

### 配置build.gradle

在位于项目的根目录 build.gradle 文件中的dependencies添加Walle Gradle插件的依赖

```
classpath 'com.meituan.android.walle:plugin:1.1.6'
```

在当前App的 build.gradle 文件中apply这个插件，

```
apply plugin: 'walle'

dependencies {
    compile 'com.meituan.android.walle:library:1.1.6'
}
```
<!-- more -->
### 配置插件
```
walle {
	 // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels")
    // 定制渠道包的APK的文件名称
    apkFileNameFormat = '${appName}-${channel}-v${versionName}-${versionCode}-${buildTime}-${buildType}.apk'
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```

其中需要channelFile可以参考：[渠道配置文件示例](https://github.com/Meituan-Dianping/walle/blob/master/app/channel)

ps：如需要在debug和release分别使用对应的channelFile，可以添加

```
afterEvaluate {
    project.android.applicationVariants.all { BaseVariant variant ->
        variant.assemble.doFirst {
            if (variant.buildType.name == "debug") {
                project.walle.channelFile = new File("${project.getProjectDir()}/channel_debug")
            }
        }
    }
}
```

### 如何获取渠道信息

在需要渠道等信息时可以通过下面代码进行获取

```
String channel = WalleChannelReader.getChannel(this.getApplicationContext());
```

### 生成渠道包

终端执行

```
./gradlew clean assembleReleaseChannels
```

## 2.hotfix的步骤

* 使用命令行 **./gradlew clean assembleReleaseChannels**，此时会在app/build/barapk 目录下生成基准包

* 并且在 app/build/outputs/channels目录下生成所有渠道包

* 接下来修改tinker-support.gradlew文件，修改baseApkDir的值为基准包的值

* 构建基准包跟补丁包都要修改tinkerId，主要用于区分 基准包tinkerId必须小于补丁包的Id号 tinkerId = "XXX" //其中tinkerId命名一般是跟随versionname或者git提交编号，我采用的是git提交编号; 

* 使用命令行**./gradlew buildTinkerPatchRelease**，会在 app/build/outputs/patch下生成release的文件夹，里面有补丁包。我们会选择patch_signed_7zip.apk这个提交到Bugly的管理后台

ps:至于采用命令行**./gradlew buildTinkerPatchRelease**而不是跟Bugly的教程点击android studio右侧 Gradle 选择tasks/tinker-support,这个大家可以进行对比操作。

到这里整个hotfix结束，在我的项目实行这个hotfix用时：生成基准包4mins左右，修改代码2mins，生成patch包2mins左右，不到十分钟就可以搞定，这又可以有时间喝茶去。

下一步在构思如何采用Jenkins实现整个hotfix操作，只需要提交修改的代码就可以喝茶生成patch，这才是我想要的。









