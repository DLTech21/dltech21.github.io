---
title: RecycleView实现抖音首页播放视频效果
date: 2018-06-14 15:08:00
tags: [Android, RecycleView, 抖音]
---

前端时间花了几天时间仿做了抖音的Android端，[APP体验地址](https://github.com/DLTech21/douyin)

在做首页的时候，考虑过viewpager和recycleview这两种方案，最后采用了recycleview实现。

<!--more--->
单依靠recycleview是不足以实现的，我采用了第三方库[RecyclerViewSnap](https://github.com/rubensousa/RecyclerViewSnap)实现卡片式滑动的效果

卡片式滑动代码

```java
new GravityPagerSnapHelper(Gravity.BOTTOM, true, new GravitySnapHelper.SnapListener() {
    @Override
    public void onSnap(final int position) {
        if (currentPosition == position) {
            return;
        }
        currentPosition = position;

        viewPager.postDelayed(new Runnable() {
            @Override
            public void run() {
                startPlay(position);
            }
        }, 500);

        if (currentPosition == datas.size() - 1) {
            fetch(-1);
        }

    }
}).attachToRecyclerView(recycleview);
```

### 视频播放
视频播放库选用[GSYVideoPlayer](https://github.com/CarGuo/GSYVideoPlayer)，使用方法简单不做介绍。

由于播放的是抖音的视频，考虑到版权问题，代码就不开源，有需要源码的可以pm我。



