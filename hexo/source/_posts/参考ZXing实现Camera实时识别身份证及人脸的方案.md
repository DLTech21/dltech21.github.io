---
title: 参考ZXing实现Camera实时识别身份证及人脸的方案
date: 2018-07-10 15:03:57
tags: [Camera, ocr, ZXing]
---

由于ocr及活体检测的功能涉及公司保密性，本文主要写的是如何实现实时分析camera返回的每一帧。

在开始做这个的时候，联想到类似功能就是二维码识别，因此直接采用zxing的实现方式，通过previewcallback回调的data利用thread+handler

也就是zxing源码中的三个比较关键的类
CaptureActivityHandler, DecodeThread, DecodeHandler，其中DecodeHandler实现检测每一帧的类。

具体代码可以前往[github](https://github.com/DLTech21/ZXingCamera)
