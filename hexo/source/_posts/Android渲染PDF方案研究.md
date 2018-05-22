---
title: Android多线程渲染PDF
date: 2018-05-10 14:46:28
tags: [Android, pdf, pdfium, mupdf]
---
由于业务需要用到PDF的解析，通过在github上搜索相关的开源库，实现渲染PDF的开源库有下面两个

[PdfiumAndroid](https://github.com/barteksc/PdfiumAndroid)

[MuPDF](https://mupdf.com/)

<!--more-->

这两个开源库我都使用过，如果业务是简单渲染PDF使用PdfiumAndroid就足够了，如果需要更多功能例如验签，看签名信息等就使用MuPDF进行二次开发即可。本文只是写渲染方面的，那么就采用PdfiumAndroid，同时也在PdfiumAndroid基础上封装了一个较为简单的PDF渲染库[PdfRender-PdfGo](https://github.com/DLTech21/PdfRender-PdfGo)

PdfiumAndroid渲染的核心方法是renderPageBitmap，也就是将PDF的某一页渲染到Bitmap，这个方法也有一定的耗时，如果我们需要用到viewpager来看PDF或者是打开一个PDF的缩略图目录，这时候就需要多线程渲染PDF，实现也不是难事，可以阅读源码。