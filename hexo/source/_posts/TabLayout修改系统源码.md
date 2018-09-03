---
title: TabLayout修改系统源码
date: 2018-08-01 10:59:54
tags: [Android, Tablayout]
---

转载自[github](https://github.com/zjw-swun/AppOrder)

# 0. 老规矩，先放效果图


![图片描述](https://github.com/zjw-swun/AppOrder/blob/master/images/1.gif?raw=true)

<!-- more -->

# 1. 骚操作之一：copy support包TabLayout 一份当做自己的自定义view
本次骚操作是基于support '27.1.0'版本，从support '27.1.0'拷出文件到我的项目目录如下图，蓝色部分，四个文件，当然不是一帆风顺的，需要改点包名，取消掉一下注解警告，总之后面会放出源码

![](https://user-gold-cdn.xitu.io/2018/4/5/16291cbabfc62690?w=396&h=215&f=png&s=15889)
不同的版本可能需要拷贝出来的文件不一样哟，于support '27.1.0'版本需要拷出上图蓝色的4个文件

# 2. 骚操作之二： fork源代码，读懂之后开始改造
首先指示器的线是画出来的，关键代码如下 （以下改动代码均为tabLayout类）

![](https://user-gold-cdn.xitu.io/2018/4/5/16291d279361e0c7?w=1396&h=521&f=png&s=116077)

```
   canvas.drawRect(mIndicatorLeft, getHeight() - mSelectedIndicatorHeight,
                            mIndicatorRight, getHeight(), mSelectedIndicatorPaint);
```
OK, mIndicatorLeft是滚动或者点击切tab时候通过偏移量计算出来的，总之不重要，完成第一个目标。修改指示器线宽，思路呢，就是给mIndicatorLeft和mIndicatorRight做一个偏移量就行了，
看看我怎么改的吧

![](https://user-gold-cdn.xitu.io/2018/4/5/16291d715ee6ddc4?w=1868&h=616&f=png&s=127003)
其中2个成员变量是我在SlidingTabStrip类中新增的
```
 private int mSelectedIndicatorWidth =  dpToPx(27);;
        private int mMinTabWidth = Integer.MAX_VALUE;
```

我这里偷懒一下就不做方法暴露了，直接写死了线宽为27dp了

![](https://user-gold-cdn.xitu.io/2018/4/5/16291d89e7648856?w=760&h=412&f=png&s=40677)


![](https://user-gold-cdn.xitu.io/2018/4/5/16291d9b7fb06f96?w=1691&h=970&f=png&s=196796)

好了，已经完成修改线宽目标了。(扩展一下：这里你也可以修改draw方法，画个图，或者画个小圆圈什么的)

接下来增加指示线滑动切tab的过渡动画

很简单我就放代码吧，关键就是在onPageScrolled方法里面做点手脚

![](https://user-gold-cdn.xitu.io/2018/4/5/16291e086784ff9c?w=1882&h=953&f=png&s=210378)

总共改动就50来行吧，就达成效果了。是不是很简单。（简单才怪，总之做出来之后觉得确实蛮简单的）
这样改好处多多，为什么呢？xml基本不需要改变，tablayout名字改一下，代码也是导包改一下，替换官方tablayout的时候代码几乎不需要变化，是不是很爽？

# 3. github下载，喜欢就给个star吧，如果对你有帮助的话
[https://github.com/zjw-swun/AppOrder](https://github.com/zjw-swun/AppOrder)

# 4. 总结
官方support包就是可以这么任性的拷贝出来，有时候一个拷出一个类根本没涉及到别的类，善假于物也。


