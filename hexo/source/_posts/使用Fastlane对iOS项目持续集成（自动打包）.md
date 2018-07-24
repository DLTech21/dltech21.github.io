---
title: 使用Fastlane对iOS项目持续集成（自动打包）
date: 2018-07-09 16:42:15
tags: [Fastlane, Jenkins, iOS]
---

文章转载[https://www.jianshu.com/p/9cdfafc5f7b9](https://www.jianshu.com/p/9cdfafc5f7b9)

使用Fastlane对iOS项目持续集成（自动打包）
==========================


前言
==

  作为一名iOS app开发者，在我的工作过程中，基本遵循如下的一个流程：**分析需求、UI设计——>设计功能架构——>着手开发——>打测试包——>修复bug、优化功能**。  
　　在所有这些工作中，项目打测试包对于一个开发人员来说，可以说是一项无脑又浪费时间的工作，很荣幸的是，我在公司负责iOS项目的打包。  
　　那么来看看打包的时间都浪费在哪了。来看下打包的流程：**Archive项目——>勾选一堆选项及下一步，打包用途、app瘦身、证书——>导出ipa包——>打开蒲公英——>上传ipa包——>填写项目描述、安装密码——>发布测试包**。如此这般下来，真的是被恶心吐了，手动的操作是一方面，另一方面，在进行编译项目、导出ipa、上传ipa这些操作时，则需要等待很长时间，只有等待上一步耗时操作完成，才能进行下一步，无疑很浪费时间。  
　　那么当然会有一种办法，能为我们解决这个问题，因为懒才是科技进步的第一生产力。

<!-- more -->
Fastlane
========

  这里就要引入一个概念了，叫持续集成，引用下百度百科的介绍：

> 持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。

  而今天文章的主角就是**Fastlane**，一套ruby编写的持续集成工具集。通过Fastlane可以实现自动打包、发布、截取app图片等工作，而Fastlane可以执行通过ruby代码或者Fastlane提供的一些工具编写的脚本来实现这些工作。以下皆以打包发布到蒲公英为例，因为蒲公英为开发者提供了Fastlane的蒲公英插件，允许开发者通过Fastlane上传ipa包到蒲公英，fir好像也提供了Fastlane的插件，具体没去了解，如果有使用fir的，可以在回复中补充。  

![](https://upload-images.jianshu.io/upload_images/1682338-2be75e344eedd459.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

Fastlane

集成流程及使用方法
=========

### 1、ruby

  因为Fastlane是ruby编写的，所以我们首先保证电脑的ruby环境有正确安装，打开终端，输入如下命令来查看ruby版本。

     ruby -v
    

![](http://upload-images.jianshu.io/upload_images/1682338-7ed82e907d636f7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

查看ruby版本

  

　　这里保证ruby版本在2.0以上就好了。如果低于2.0，就需要升级ruby了，这里不提了，百度谷歌都有教程。

### 2、安装Fastlane

  首先安装Xcode命令行工具，因为编译、打包等操作，虽然是Fastlane帮我们做的，但本质上还是通过Xcode中的构建工具来完成的。在终端中运行如下命令，则会安装Xcode命令行工具：

    xcode-select --install
    

  然后安装Fastlane：

    sudo gem install fastlane --verbose
    

  如果发现最后报了这样的错误：

    ERROR:  While executing gem ... (TypeError)
        no implicit conversion of nil into String
    

  则更新gem版本，然后再次安装Fastlane：

    sudo gem update --system
    

  如果报错：

    ERROR:  While executing gem ... (Gem::FilePermissionError)
        You don't have write permissions for the /usr/bin directory.
    

  则尝试使用如下命令进行安装：

    sudo gem install -n /usr/local/bin fastlane
    

  如果还安装失败。。。去官网看看别的安装方法吧：[Getting started with fastlane for iOS](https://docs.fastlane.tools/getting-started/ios/setup/)  
  为了检查Fastlane是否成功安装，可以通过下面的命令来查看Fastlane版本号：

    fastlane --version
    

### 3、为项目初始化Fastlane

  如果Fastlane正确安装了，就可以为我们的项目初始化Fastlane了，首先通过终端，CD到项目目录，也就是项目的.xcodeproj文件所在位置。然后执行Fastlane初始化命令：

    fastlane init
    

  这里如果一直卡在bundle update，那应该就是被墙了，这时候来到项目目录下，找到Gemfile，打开Gemfile将里面的内容修改为如下：

    #source "https://rubygems.org"
    source "https://ruby.taobao.org"
    
    gem "fastlane"
    

  重开终端，运行`bundle update`就好了。  
  然后安装蒲公英插件，安装后有个y/n的选择，选择y：

    fastlane add_plugin pgyer
    

  初始化结束后，会提示选择Fastlane的用途，一共是四个选项，我选了最后一个，自定义，然后打开项目目录，会发现多了一个fastlane文件夹：

  

![](http://upload-images.jianshu.io/upload_images/1682338-ef9ecba794bd2664.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

fastlane文件夹

  

　　打开文件夹中的Fastfile，里面则是执行自动化打包任务的代码，这里我是用sublime打开的，在sublime的菜单中，找到View——>Syntax中选择ruby，即可高亮代码：

  

![](http://upload-images.jianshu.io/upload_images/1682338-4b01919d5ec865d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

Fastfile里的内容

### 4、创建一个lane

  Fastlane以lane为单位，去执行一个自动化任务，Fastfile中的代码如下：

    default_platform(:ios)
    
    platform :ios do
      desc "Description of what the lane does"
      lane :custom_lane do
        # add actions here: https://docs.fastlane.tools/actions
      end
    end
    

  **lane：custom_lane**，代表了一个叫**custom_lane**的任务，后面的**do**，则表示需要执行的操作。这里就不讲怎么写代码了（因为我也不会ruby啊！现用现找就好了），我把我项目中的Fastfile贴上来，讲解下Fastfile做了哪些事（见代码中的注释）：

    platform :ios do     #指定持续集成对象的平台名称
    lane :dev do|options|      #给lane命名
    branch = options[:branch]
    
    
    #这里我们项目为了区分线上环境和测试环境，而做了两个target
    #关于target区分环境的方法，可以参考我同事的简书文章https://www.jianshu.com/p/23cc84d40423
    #下面代码通过在终端输入1或者其他数字来选择要打包的target
    #puts是ruby中的输出，gets为获取终端中输入的文字，gets需要指定STDIN包中的gets方法，否则会识别为其他包中的gets方法，具体为什么我也不知道
    puts "请选择要打的scheme：（1：项目Target1，else: 项目Target2）"      
    scheme = STDIN.gets
    #  通过判断输入内容，来区分一些打包信息，1后面加\n是因为在终端输入1再敲回车的时候scheme就包含了回车的内容，所以scheme == "1\n"
    if scheme == "1\n" 
    #项目中target的名称，以QQ为例，如果我的target叫QQ，则下面填写QQ，如果是wechat，就填wechat
      schemeName = "项目Target1"
    #打包的用途，也就是app-store, package, ad-hoc, enterprise, development这几个中的一个，这里我们项目的target1用的是公司帐号，打的是开发包
      export_method = "development"      
     else
      schemeName = "项目Target2"
    #这里我们项目的target2用的是企业帐号，打的是企业包
      export_method = "enterprise"      
    end
    
    #从蒲公英平台拿到的api_key和user_key，下面我会讲怎么拿到这两个key，存在下面两个变量中
    api_key = "xxxxxxxxxxxxxxxxxxx"
    user_key = "xxxxxxxxxxxxxxxxxxx"
    
    
    
    #输入蒲公英上传ipa包后输入的版本描述信息
    puts "请输入版本描述："
    desc = STDIN.gets
    
    
    
    puts "开始打包 #{schemeName}"
    # 开始打包
    gym(
    #指定scheme的名字
    scheme: "#{schemeName}",
    #输出的ipa名称
    output_name:"#{schemeName}",
    # 是否清空以前的编译信息 true：是
    clean:true,
    # 指定打包方式，Release 或者 Debug
    configuration:"Release",
    # 指定打包所使用的输出方式，目前支持app-store, package, ad-hoc, enterprise, development
    export_method:"#{export_method}",
    # 指定输出文件夹，这里会保存我们最后生成的ipa文件，也就是存到了我们上面提到的fastlane文件夹中的build文件夹中
    output_directory:"./fastlane/build",
    )
    
    puts "开始上传到蒲公英"
    #开始上传ipa到蒲公英，这里用的是蒲公英提供的插件
    #update_description代表更新信息，password代表安装密码
    pgyer(update_description: "#{desc}", api_key: "#{api_key}", user_key: "#{user_key}", password: "1111", install_type: "2")
    
    
    
    #在上传完ipa后，打开ipa的存放文件夹，起到提示上传完成的作用
    system "open ../fastlane/build"
    
    end
    end
    

  如上代码，从头到尾读下来，其实就是一个很简单的**输入变量——>定义几个变量——>调用一个名为gym的打包方法，将定义的变量作为参数传进去——>调用一个名为pgyer的方法上传ipa，将前面定义好的变量作为参数传进去**。  
  至于蒲公英的api\_key和user\_key，可以在蒲公英官网的**我的应用——>之前发布的应用——>API**中找到：  

api\_key和user\_key

### 4、执行脚本

  cd到项目目录，输入如下命令来执行我们自己定义的lane，格式如下**fastlane+脚本第一行中的platform名+脚本第二行中的lane名**：

    fastlane ios dev
    

  可以发现Fastlane开始执行任务了，并且符合我们的代码逻辑：

  

符合我们的代码逻辑

  

  同时可以看到Fastlane在编译我们的文件，这里执行的是gym方法：

  

编译文件

  
  最后会生成dSYM文件以及ipa：  

打包结果

  

  接下来会继续上传ipa到蒲公英：

  

上传到蒲公英

  
　　可以看出，gym打包用时275秒，pgyer上传到蒲公英是58秒，用时一共是5分半，速度是非常快的，而且不需要人为的去进行各种操作。最后打开蒲公英就会发现我们的app已经上传上去了，而且设置好了下载密码、版本信息等。

一些问题汇总
======

如果在打包的时候提示如下错误，找不到Scheme：

  

![](http://upload-images.jianshu.io/upload_images/1682338-9b670c5252bda00f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

  

　　那么找到xcode的Manage Schemes，然后将你要打包的target的shared勾选上，再打包就可以了：

  

![](http://upload-images.jianshu.io/upload_images/1682338-241e361b6753030c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

总结与思考
=====

  以上就实现了我们的自动化打包并上传到蒲公英的功能，可以说是非常方便省时的了，虽然还是需要五分半的时间，但相比之前的各种复杂操作、各种等待，已经是质的飞跃了。  
　　而Fastlane能做的并不仅限于这些工作，参考Fastlane官方文档，可以发现很多其他功能：[Fastlane文档](https://docs.fastlane.tools/getting-started/ios/setup/)，比如自动发布到AppStore，自动上传app截图到itunes connect等等，这些功能我暂时还用不上（上线的时候还是要自己把关下，手动好一些，并且上线也不是经常性的），大家可以根据自己的需求找一些资料来尝试下，本篇文章的主旨还是满足大家的最基本的需求，也建立在我个人的真实使用场景下。  
　　另外，由于Fastlane采用ruby编写，则Fastlane的一个lane任务可以做的定制化，就不止我编写的那些简单内容了，当然我也只是为了满足我个人的需求，所以做了很简单定制，如果有对ruby感兴趣的同学或精通ruby的大神，完全可以对lane任务做更好的定制，比如上传完后弹出系统提示框，通知我们上传完成等更便利的功能（上面脚本中已经补充了这一功能）。  
　　参考文章：  
　　[使用 Fastlane 上传 App 到蒲公英](https://www.pgyer.com/doc/view/fastlane)  
　　[Getting started with fastlane for iOS](https://docs.fastlane.tools/getting-started/ios/setup/)  
　　[使用 fastlane 实现对 iOS Multi-Target 的一键打包部署](https://www.jianshu.com/p/77e7fc2cb3c2)  
　　[iOS自动化打包发布（Jenkins + Fastlane + GitLab + 蒲公英）](https://www.jianshu.com/p/0a113f754c09)  
　　[Ruby 教程](http://www.runoob.com/ruby/ruby-tutorial.html)