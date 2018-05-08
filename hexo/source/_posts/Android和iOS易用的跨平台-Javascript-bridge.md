---
title: Android和iOS易用的跨平台 Javascript bridge
date: 2018-05-08 10:10:15
tags: [每日小技巧, Android, iOS, Javascript]
---

现在很多时候我们搞移动开发都需要嵌入h5界面，那么就得需要js交互，这时候就需要一套通用的js代码来支持Android及iOS的调用，经过采坑采用DSBridge作为解决方案。

Android端使用，我个人建议还是使用2.0版本，因为3.0版本在某些机子还是有点问题
使用方法详细请看:[DSBridge-Android](https://github.com/wendux/DSBridge-Android/blob/master/readme-chs.md)

iOS端使用方法详细请看:[DSBridge-IOS](https://github.com/wendux/DSBridge-IOS/blob/master/readme-chs.md)

<!--more-->
JavaScript端一定要添加才有效果

```java
<script src="https://unpkg.com/dsbridge/dist/dsbridge.js"> </script>
```

由于Android采用2.0版本，那么js代码就得稍微做一下适配，这个代码是服务器端返回html到前端，js不同之处就是传参数到Android端采用json的格式，而传给iOS则直接采用字符串即可。
	
```
if (!StringUtils.isEmpty(request.getHeader("platform"))) {
    if (request.getHeader("platform").equals("iOS")) {
        contentStr += "\n<script>\n" +
                "    function showImage(name) {\n" +
                "        dsBridge.call(\"showImage\", name)\n" +
                "    }\n" +
                "</script>";
    } else {
        contentStr += "\n<script>\n" +
                "    function showImage(name) {\n" +
                "        dsBridge.call(\"showImage\", {msg: name})\n" +
                "    }\n" +
                "</script>";
    }
}
```
