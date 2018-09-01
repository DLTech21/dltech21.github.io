---
title: Mac下编译FFmpeg，输出只有一个libffmpeg.so
date: 2018-08-15 22:33:52
tags: [FFmpeg, Android, so]
---

这两天在研究mac下编译FFmpeg为一个so，查阅网上的资料大多数都是输出多个so，这样对于直接使用这些类库或者用FFmpeg的so再输出其他类库真的不方便，因此可以将多个合并为一个类库 libffmpeg.so。
<!-- more -->
思路就是先编译成.a文件再合并so，这里不再需要修改Configure的内容。

核心脚本build_android.sh

```bash
#!/bin/bash
PLATFORM=/Users/donal/Library/Android/sdk/ndk-bundle/platforms/android-19/arch-arm/
TOOLCHAIN=/Users/donal/Library/Android/sdk/ndk-bundle/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
PREFIX=./android
function build_one
{
./configure \
--prefix=$PREFIX \
--target-os=android \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--arch=arm \
--sysroot=$PLATFORM \
--extra-cflags="-I$PLATFORM/usr/include" \
--cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
--nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
--disable-shared \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--enable-small \
--enable-gpl \
--enable-asm \
--enable-jni \
--enable-mediacodec \
--enable-decoder=h264_mediacodec \
--enable-hwaccel=h264_mediacodec \
--enable-decoder=hevc_mediacodec \
--enable-decoder=mpeg4_mediacodec \
--enable-decoder=vp8_mediacodec \
--enable-decoder=vp9_mediacodec \
--enable-nonfree \
--enable-version3 \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make j8
make install
$TOOLCHAIN/bin/arm-linux-androideabi-ld \
-rpath-link=$PLATFORM/usr/lib \
-L$PLATFORM/usr/lib \
-L$PREFIX/lib \
-soname libffmpeg.so -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o \
$PREFIX/libffmpeg.so \
libavcodec/libavcodec.a \
libavfilter/libavfilter.a \
libswresample/libswresample.a \
libavformat/libavformat.a \
libavutil/libavutil.a \
libswscale/libswscale.a \
libavdevice/libavdevice.a \
libpostproc/libpostproc.a \
-lc -lm -lz -ldl -llog --dynamic-linker=/system/bin/linker \
$TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9.x/libgcc.a


	cp $PREFIX/libffmpeg.so $PREFIX/libffmpeg-debug.so
    arm-linux-androideabi-strip --strip-unneeded $PREFIX/libffmpeg.so

}
# arm v7vfp
CPU=arm
OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU "
ADDI_CFLAGS="-marm"
build_one
```
