---
title: Android逆向之旅---Android中分析抖音和火山小视频的数据请求加密协议(IDA动态调试SO)
date: 2018-06-04 10:21:01
tags: [Android, 逆向, 抖音]
---

一、前言
最近萌发了一个做app的念头，大致什么样的app先暂时不说，后面会详细介绍这个app的开发流程和架构，不过先要解决一些技术前提问题，技术问题就是需要分析解密当前短视频四小龙：抖音，火山，秒拍，快手这四家短视频的数据请求加密协议，只有破解了加密协议，才可以自定义数据请求拉回数据。不多说了，本文先来第一站搞定抖音和火山这两个数据请求的加密协议，为什么说是这两个呢？因为我们在后面分析会发现这两个app其实用的是一套加密协议，毕竟这两个app都是今日头条内部孵化的项目。所以只要搞定一个即可。我们决定搞定抖音吧，毕竟我比较看好和喜爱看抖音。

<!---more--->

二、逆向分析
不多说了，赶紧来分析吧，不过本文不在利用粗暴的静态方式去破解了，应广大同学要求，就介绍IDA动态调试so来进行破解，这样也能给大家带来IDA的使用技巧。毕竟我写文章技术都是为了你们。这种分析请求数据的突破口一般都是抓包，这不用多说了。不过都是用了https请求，所以需要手动在设备中安装Fiddler证书，才能抓到正确的数据信息：
![640?wx_fmt=png&wxfrom=5&wx_lazy=1](/assets/third/5e7dede7-18c8-4d70-acff-20cdfec4232a.jpg)

打开抖音之后，看到数据刷的很快，发现一个feed的接口是返回的首页的数据，在分析它的请求参数中有三个字段是minCursor,maxCursor,count其实这三个字段就是后面他进行数据分页请求的关键，到后面再详细说。不过这里看到还没有什么问题，不过问题往下看他的更多请求参数，会发现两个字段：
![0?wx_fmt=png](/assets/third/a7e96151-5a87-40ce-bb7a-f970def6b15d.png)

这时候会发现其他参数都和本地设备有关或者直接写死的，唯独这两个参数信息是始终变化的。所以猜想这两个字段是用于请求协议数据加密和服务端进行校验的。那么如果我们想单独构造信息去请求，这两个字段的信息一定要正确，不然请求不到正确的数据的。好了，这里简单了，使用Jadx打开抖音app即可，直接全局搜索字段的信息"as="：
![0?wx_fmt=png](/assets/third/fb8fcae1-4f71-436f-bc8b-aa727b722f80.jpg)
看到这里是构造了as和cp两个字段的值，直接点击进入即可：
![0?wx_fmt=png](/assets/third/463cb80d-6cc1-4328-9f6c-2b0264a56af3.png)
这里大致的逻辑也比较简单，利用UserInfo.getUserInfo函数获取字符串，然后对半开给as和cp两个字段，右移操作就是除以2的意思。这里不分析代码来看看参数怎么来的，直接用Xposed进行hook这个函数打印参数看结果，粗暴快捷，以后其实这都是快速解决问题的一种方式，hook大法是最无敌的
![0?wx_fmt=png](/assets/third/b335015e-adcd-45d2-9756-fa87053eca12.png)
先来看一下这个加密方法的参数信息，看到是native的，也就是说加密操作是在so中做的，后面需要调试的也是这个so了。也不管了，hook这个方法然后打印信息看参数构造：
![0?wx_fmt=png](/assets/third/b2ad399d-85f1-41cc-8e5f-29d670bf0411.jpg)
直接运行模块，打印日志看信息：
![0?wx_fmt=png](/assets/third/b4710987-b286-455b-94e8-b0946b58f5d9.jpg)
看到三个参数打印的值，发现大致三个参数的意义是：当前时间戳，请求url，请求参数的数组信息。好了，我们现在可以新建一个Android工程，然后构造这三个参数信息，然后调用它的so函数，为了能够快速找到这个so名称可以用万能大法：全局搜索字符串信息"System.loadLibrary" 即可：
![0?wx_fmt=png](/assets/third/f7c9830d-b437-4990-b91d-cc640868da60.jpg)
这样就找到了这个so名称了，到libs目录下把这个so拷贝出来到我们自己构建的demo工程中，这里先不着急调试去分析native的加密函数功能，还是老规矩拿来主义，直接上层构造一个和他一样的包名的native函数，调用它的so获取结果即可：
![0?wx_fmt=png](/assets/third/3d00106e-5612-4018-968a-7c9e1d8273b0.jpg)
然后就开始构造参数信息了，这里为了简单，先把那些公共的参数信息写死比如设备的sid，aid，版本号等信息：
![0?wx_fmt=png](/assets/third/4f99f9a2-d2d2-49b7-b1e7-f5eef3588596.jpg)
这里我们利用公众参数信息，构造请求字段数组信息，然后还有单独的几个字段值不能参与操作，可以通过之前的打印日志分析出来。当然时间戳也是服务器格式，和本地是少三位的，直接除以1000即可。下面为了简单直接把公众参数写死即可，当然后续优化就是把这些参数值进行动态获取填充即可：
![0?wx_fmt=png](/assets/third/b3c81f4a-84eb-48af-ae2a-2352e7b1e23d.jpg)
构造好参数之后，然后就开始调用native函数，然后获取返回结果即可：
![0?wx_fmt=png](/assets/third/a84d1e81-55da-419c-b227-7c410b0799e7.png)
到这里，我们构造的工作就完成了，下面就开始直接运行吧，不过可惜的是，没有这么顺利，直接运行闪退了：
![0?wx_fmt=png](/assets/third/d6a9bd2e-f08b-40cd-8864-f9f5c3311733.png)
这里应该进行加载so出现问题了，那么我们在回过头看看我们是不是有些环境没初始化，看看UserInfo类中是不是还有一些初始化方法没调用：
![0?wx_fmt=png](/assets/third/078009f8-3009-4e40-a125-b7965475c3d1.png)
这里看到的确有两个方法有点可疑，全局查看这两个方法的调用地方：
![0?wx_fmt=png](/assets/third/23ad2286-a3d7-42ff-b095-b09b32078683.png)
看到全局中就一个地方调用了setAppId方法，而且值就是写死的为2，另一个方法initUser就有点麻烦了，不过还是万能的hook大法，直接hook这个方法打印他的参数信息即可
![0?wx_fmt=png](/assets/third/f0f73572-4c67-418e-8ee7-fcb9aecc2abf.png)
运行模块，查看打印的日志信息：
![0?wx_fmt=png](/assets/third/177e3a7e-2f55-42ec-837c-c69b618229d6.png)
看到了，参数信息一致都是这个字符串信息，直接拷贝出来赋值调用即可：
![0?wx_fmt=png](/assets/third/e420b2d9-6e9c-4bc1-b6e3-dcecfb680280.png)

三、IDA调试so代码
在次运行，发现可惜了，还是报错，而且诡异的是日志没打印，也就说setAppId和initUser函数没有调用就退出了，那么就会想到？是不是so中的JNI_OnLoad函数中做了一些判断逻辑呢？直接用IDA打开这个so文件即可：
![0?wx_fmt=png](/assets/third/a9e9e1fd-71df-412d-acec-d4d92aade6eb.png)
打开之后找到JNI_OnLoad函数，F5查看他的C代码，发现果然内部有很多判断，然后直接调用exit函数退出了。所以如果想成功的调用它的so方法，得先过了他的这些判断了，这里就要开始进行调试操作了，其实这里可以直接静态分析有一个快捷的方法，但是为了给大家介绍动态调试so技巧，就多走点弯路吧，后面再说一下粗暴简单的技巧。

下面就来开始进行调试so文件了，关于IDA调试so文件，我之前已经写过一篇非常详细的文章了：Android中IDA动态调试so文件详解；这里不会在详细介绍具体步骤了，直接上手干：
第一步：拷贝IDA安装目录下的android_server文件到设备目录下

第二步：运行android_server命令
![0?wx_fmt=png](/assets/third/7f871928-fca5-49ff-a1de-c981815717f8.png)

第三步：转发端口和用debug模式打开应用
![0?wx_fmt=png](/assets/third/de494af2-3ff5-47a2-b143-b1b6a9b30e99.png)
这里有同学好奇为什么不需要修改xml中的debug属性呢？因为我调试的是我自己的demo工程，而Eclipse默认签名打包出来的apk这个属性值就是true的，所以不需要进行修改了。
![0?wx_fmt=png](/assets/third/38231241-8685-408a-9cd0-5061be72e867.png)

第四步：启动IDA附件进程
![0?wx_fmt=png](/assets/third/ce19b487-cfa7-42d0-8d43-3d3ea06664cc.png)
设置本地地址和一些选项：
![0?wx_fmt=png](/assets/third/3f62f0b6-f086-48a8-8902-2d937b106395.jpg)
因为现在已经知道需要调试JNI_OnLoad函数，所以需要设置挂起load函数处，然后选择调试的应用进程：
![0?wx_fmt=png](/assets/third/8b133391-b0eb-4792-9728-2f2f971467c0.png)

第五步：查看调试端口连接调试器
![0?wx_fmt=png](/assets/third/bbaf7d6a-1bb0-4601-bcfc-098a4b466e15.png)
在Eclipse中的DDMS中查看有红色小蜘蛛的调试应用端口号是8647，然后连接调试器：
![0?wx_fmt=png](/assets/third/549a6944-182b-497e-a129-c2d385bd3505.png)
这里一定注意端口正确，不然链接操作的。

第六步：点击IDA中的运行按钮，或者F9快捷键
![0?wx_fmt=png](/assets/third/a175673f-6ff1-40b1-ab7c-95261d19df48.png)
这时候发现红色蜘蛛变成绿色了，而且调试对话框也没有了。这时候就进入调试页面了：
![0?wx_fmt=png](/assets/third/e4b7d63c-2254-4c31-8b16-3cf04c8c978d.png)

为了安全起见，再一次查看debug选项有没有挂起load函数：
![0?wx_fmt=png](/assets/third/7d901268-8440-4d32-ab22-e7c270bb6314.jpg)
如果发现没有，还需要进行手动勾选上：
![0?wx_fmt=png](/assets/third/42c0153a-2625-453e-9b2d-9aaa6ffea5ab.png)
因为我们给JNI_OnLoad函数挂起了，而一个应用会加载很多系统的so文件，所以这里一直点击运行或者F9：
![0?wx_fmt=png](/assets/third/5df63798-b88d-46fa-87bc-c3e73dc6c1d0.png)
过了系统so加载步骤：
![0?wx_fmt=png](/assets/third/322cb50b-f41c-4991-af8a-3ca9590b650e.png)
这些都是系统的so文件加载，所以一路往下都直接运行越过调试即可：
![0?wx_fmt=png](/assets/third/4c914a53-0e71-49b5-b2db-e23948347900.png)
可以看到这里会加载很多系统的so文件，当我们点击应用的按钮，加载自己的libuserinfo.so文件的时候才开始进行调试工作：
![0?wx_fmt=png](/assets/third/36d53955-b1d7-4915-8c0b-5618599464e8.png)
点击OK加载进来，然后就停留在了挂起状态了，这时候，我们在右侧栏查找这个so文件：
![0?wx_fmt=png](/assets/third/e87318e2-ec2e-435c-b42c-557a8a35ca91.png)
然后点击，继续查找他的JNI_OnLoad函数：
![0?wx_fmt=png](/assets/third/37fc8ba0-966e-447a-9598-342349428c83.png)
点击进入JNI_OnLoad函数处，下个断点：
![0?wx_fmt=png](/assets/third/9b07453d-ac45-4e26-8f6c-a0e6875dc144.png)
因为我们在之前静态分析了这个函数内部有很多个exit函数，为了好定位是哪个地方exit了，所以在每个exit函数之前下个断点，来判定退出逻辑，这里下断点有技巧，因为是if语句，所以在arm指令中肯定就是CMP指令之后的BEQ跳转，所以在每个CMP指令下个断点即可，这里发现了9个地方，所以下了断点也很多，慢慢分析：
![0?wx_fmt=png](/assets/third/5033fe22-6c45-4d23-9650-2ca04e1acffb.png)
第一处的CMP指令下个断点，然后运行到此处，查看R3寄存器值是否为0：
![0?wx_fmt=png](/assets/third/60f0ce8f-2336-4f09-a065-2596fa6becb0.png)
是0，那么第一处exit就没问题，接着往下走，直接按F9到下一个CMP判断指令断点处：
![0?wx_fmt=png](/assets/third/5b972f66-764a-4342-b67b-fb3b988e29f5.png)
发现第二处的CMP中的R3寄存器值也是0，所以也没问题，直接F9到下一个断点：
![0?wx_fmt=png](/assets/third/34c26243-1a6e-4c51-965e-d00aee69bb70.png)
到了第三处判断发现R3寄出去你的值不是0了，而是1，所以为了继续能够往下走，就修改R3寄存器值即可，在右侧栏的寄存器中右击R3寄存器，然后点击修改值：
![0?wx_fmt=png](/assets/third/d9673bca-1fb4-4771-bcde-620353e95f23.jpg)
把1改成0，保存即可：
![0?wx_fmt=png](/assets/third/46a86147-919a-47c0-9c63-78192b29a140.png)
修改成功了，运行发现就过了判断：
![0?wx_fmt=png](/assets/third/ef1351f7-835d-4b30-ad75-a14096e17c81.png)就继续往下走，到下一个断点：
![0?wx_fmt=png](/assets/third/abc19ad6-03ea-43f6-9a0f-a216856b0eae.png)
这里有问题了，我们如果直接F9到第四处CMP的话，发现直接退出调试了，说明在3和4处判断中间有问题了，最终发现是这个跳转指令出现的问题，这里需要多次单步调试F7键，定位出现问题的地方了，我们进入这个跳转地址：
![0?wx_fmt=png](/assets/third/42eb578d-ec7f-4642-a4c0-cca380bbef75.png)
然后发现内部还有BL指令，出现问题了，继续深入查看：
![0?wx_fmt=png](/assets/third/f6979af6-3df7-4006-8291-ee094ab3d9cf.jpg)
看到了，这里发现问题的原因了，有一个GlobalContext类，而这个类应该是Java层的，native层应该用反射机制获取全局的Context变量，我们直接去Jadx中搜索这个类：
![0?wx_fmt=png](/assets/third/a153fd51-2540-42e4-b6cd-e0247743bec3.png)
那么问题就清楚了，因为我们的demo工程中压根没这个类，所以native中获取全局context变量就失败了，所以就exit失败退出了，解决办法也简单，直接在demo工程中构造这个类，然后在Application中初始化context即可：
![0?wx_fmt=png](/assets/third/cec63aba-10ab-49b8-94d4-1279cf39733c.jpg)
构造的时候一定要注意包名一致，然后在demo中的Application类进行设置context即可：
![0?wx_fmt=png](/assets/third/715e38bb-6093-4357-9e41-86ed73c58e58.png)
然后再次运行项目，可惜还是不行，所以还得进行调试JNI_OnLoad函数了，方法步骤和上面类似，不多说了，不过这里应该是过了上面的Context获取失败的问题了，继续往下走，发现了重要信息了：
![0?wx_fmt=png](/assets/third/5b50c86c-71a7-4263-9c60-770ca71c1886.png)
这里有一个类似于MD5的值，继续往下走BL处：
![0?wx_fmt=png](/assets/third/360b92bd-468a-44bc-8920-acff88eafbfe.png)
看到R0寄存器中的值，发现是demo应用的签名信息，继续往下走BLX看看：
![0?wx_fmt=png](/assets/third/7c9b5791-3fd3-4e98-8af0-023bf1d5250c.png)
这里看到大致清楚了，是获取应用签名信息，也就是签名校验了：
![0?wx_fmt=png](/assets/third/3f36a4c1-f35c-4698-802d-caa0e04a393c.png)
好吧，在这一处判断exit函数中，应该是通过反射获取Java层的context值，然后在获取应用的签名信息和已经保存的原始抖音签名信息做对比，如果不对就退出了。那么过签名校验也很方便，直接利用我之前的kstools原理，把hook代码代码拷贝到工程中，拦截获取签名信息，然后返回正确的签名信息即可：
![0?wx_fmt=png](/assets/third/6d4937fc-0cf1-455a-a2d5-166d67c1bac5.png)
具体的hook代码去看我的kstools实现原理即可：Android中自动爆破签名信息工具kstools；然后hook代码一定要在Application的第一行代码调用，不然没有效果的。这样操作完成之后，其实已经出数据了，不过为了能够进入加密函数进行调试分析具体的加密算法，我们继续调试JNI_OnLoad函数：
![0?wx_fmt=png](/assets/third/2f5127ee-d024-4316-b369-7bbefa167040.png)
继续往下走，会发现在第五个exit判断之后，有一个函数出问题了，就是这个BL，进入内部查看：
![0?wx_fmt=png](/assets/third/d08e2c7f-645f-45e6-ac4e-d7ff56b275ea.png)
哈哈，发现了这个字符串信息，弄过调试的同学大致都猜到了，这个是反调试操作，继续往下走：
![0?wx_fmt=png](/assets/third/089c069b-ba76-45b8-b475-b3d2082c89c5.png)
这里可以百分百确定是读取status文件中的TracerPid字段值来进行反调试检测了，给下面的两个CMP指令下个断点：
![0?wx_fmt=png](/assets/third/f7752553-25ef-4deb-9a2e-5bd048cd371f.jpg)
发现R3寄存器中的确不是0，所以为了过了反调试，直接修改R3寄存器值为0即可：
![0?wx_fmt=png](/assets/third/310e6bb8-bc13-4dcf-9b29-5844256c1974.png)
这样就过了反调试检测了，单步往下走到下一个exit的判断断点，最终还有一个地方需要处理就是第七个CMP指令：
![0?wx_fmt=png](/assets/third/ff9cf30e-29ee-4223-992f-cf8596734e34.png)
处理方法直接修改R3寄存器中的值为0即可，就这样我们成功的过了JNI_OnLoad中的所有判断exit的地方了：
![0?wx_fmt=png](/assets/third/69aa15a8-9236-4377-b018-4ad1121a85b9.png)
然后给加密函数getUserInfo下个断点：
![0?wx_fmt=png](/assets/third/46f95d7b-b229-4c2c-a201-2e7e2074837b.png)
直接点击进入即可：
![0?wx_fmt=png](/assets/third/cf2c2ce0-8db5-4f59-9476-b9f0f7219e1c.png)

四、加密数据结果输出
也成功到达了加密函数断点处，这里已经没有任何判断逻辑了，就是一个单纯的加密函数了，不过这里不在进行调试分析了，感兴趣的同学可以自己操作了，因为我们的目的达到了，就是成功的获取到了加密之后的数据了：
![0?wx_fmt=png](/assets/third/1ecace59-d5b4-4442-8794-fdbc27771218.jpg)

看到了，我们成功的获取到了as和cp值，然后构造到请求url中，也成功的拿到了返回数据。我们这里多了一步解析json数据而已，原始的json数据是这样的：
![0?wx_fmt=png](/assets/third/d3e895dc-8648-4487-a596-6fd8cf43d273.jpg)
之所以要解析，也是为了后面的项目准备的，到时候我会公开项目的开发进程。不管怎么样，到这里我们就成功的获取抖音的加密信息了。主要通过动态调试JNI_OnLoad函数来解决so调用闪退问题，在文章开头的时候说到了，其实本文可以直接用简单粗暴的方式解决，就是万能大法：全局搜索字符串信息，包括Jadx中查找和IDA中查看，因为字符串信息给我们带来的信息非常多，有时候靠猜一下就可以定位到破解口了，比如这里，我们在IDA中使用快捷键Shift+F12打开字符串窗口：
![0?wx_fmt=png](/assets/third/c4b2b80d-01f5-47a7-9a88-6ed0c90bc953.jpg)
凭着这些关键字符串信息就能断定so中做了哪些操作。记住这些敏感的字符串信息，对日后的逆向非常关键。

五、技术总结
上面就解决了抖音的请求数据加密信息问题了，下面来总结一下本次逆向学习到的技术：
第一、看到在native层用反射去调用Java层的方法获取信息也是一种防护so被恶意调用的方式。比如本文的context变量获取。
第二、签名校验永远都不过时，其实本文当时没想到他有签名校验，因为看到native函数中都没有传递context变量，谁知道他是用反射调用Java层方法获取的，长知识和经验了。
第三、反调试也是永远不过时的，不过他这里的反调试检测有点简单了，就一处而且就一个进程，如果高级点应该启多个进程，循环检查tracepid值进行校验，会增大难度。
第四、在逆向中有时候在Java层没必要去花时间分析一个方法的参数和返回值构造情况，直接利用Xposed进行hook大法打印方法的参数信息靠猜也就出来了。
第五、静态方式分析永远都不会过时，全局搜索字符串也是最基本法则，靠猜就可以快速获取结果。

六、火山小视频加密分析
上面解决了抖音的加密问题，下面再来看一下火山小视频的加密信息，突破口依然是使用Fiddler进行抓包查看数据：
![0?wx_fmt=png](/assets/third/9c8a1004-4ad3-4624-aee4-d3ed85a7a180.jpg)
通过抓包会很神奇的发现，和抖音的数据结构字段几乎异曲同工，果然是自家兄弟，继续查看他的加密字段：
![0?wx_fmt=png](/assets/third/deaf6a5c-3d59-4e10-a136-6fa7da531b8d.png)
这里不在多说了，既然加密的字段都是一样的。那么我们直接不多说用Jadx打开火山小视频，看看他有没有那个native类UserInfo信息：
![0?wx_fmt=png](/assets/third/90178856-be72-4897-9cee-8f9f0523dbbb.jpg)
看来不用多一次分析了，完全一样，那么直接用同一个so，同一个加密即可，在demo工程中运行查看日志：
![0?wx_fmt=png](/assets/third/7fb1d461-5a6f-4b66-9252-2d536580fd9c.png)
初始化都是一样的，直接运行：
![0?wx_fmt=png](/assets/third/01bc7b21-0c9c-4e18-8a49-498898869635.jpg)
看到了，数据也回来了，看看他的原始json数据：
![0?wx_fmt=png](/assets/third/50d8eeaf-963b-4da8-a61c-9799f935eb37.jpg)


本文转载自[https://blog.csdn.net/F0ED9cZN4Ly992G/article/details/78780254](https://blog.csdn.net/F0ED9cZN4Ly992G/article/details/78780254)