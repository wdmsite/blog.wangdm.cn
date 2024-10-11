---
title: "FFmpeg编译：Windows平台MSYS2环境编译"
date: 2024-03-04T18:57:00+08:00
slug: ffmpeg-windows-compile
categories:
- FFmpeg
tags:
- FFmpeg
draft: false
---

### 编译环境
- 操作系统： Windows 11
- 编译环境： MSYS2 + UCTR64
- 编译软件：

  ```shell
  pacman -S mingw-w64-ucrt-x86_64-toolchain
  pacman -S git cmake
  ```


### 源码下载
- ffmpeg 

  [ffmpeg-5.1.6.tar.xz](https://ffmpeg.org/releases/ffmpeg-5.1.6.tar.xz)

- fdk-aac 

  [fdk-aac-2.0.3.tar.gz](https://github.com/mstorsjo/fdk-aac/archive/refs/tags/v2.0.3.tar.gz)
  
- x264

  git clone https://gitee.com/MediaSource/x264.git
  
- x265

  git clone https://gitee.com/MediaSource/x265.git

### 源码编译

#### x264编译

```shell
mkdir x264/build;cd x264/build
../configure --prefix=$PWD/../../library --enable-static --disable-cli --enable-strip
make -j8
make install
```

#### x265编译

```shell
mkdir x265/source/build;cd x265/source/build
cmake .. \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$PWD/../../../library \
-DENABLE_SHARED=FALSE \
-DENABLE_CLI=FALSE
make -j8
make install
```

#### fdk-aac编译

```shell
tar -xf fdk-aac-2.0.3.tar.gz
mkdir fdk-aac-2.0.3/build;cd fdk-aac-2.0.3/build

cmake .. \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=$PWD/../../library \
-DBUILD_SHARED_LIBS=FALSE
make -j8
make install
```

#### ffmpeg编译

```shell
export PKG_CONFIG_PATH=$PWD/library/lib/pkgconfig:$PKG_CONFIG_PATH

# pkg-config fdk-aac --libs --cflags
# pkg-config x264 --libs --cflags
# pkg-config x265 --libs --cflags

tar -xf ffmpeg-5.1.6.tar.xz
mkdir ffmpeg-5.1.6/build;cd ffmpeg-5.1.6/build
../configure --prefix=$PWD/../../library --disable-shared --enable-static \
--enable-gpl --enable-nonfree --disable-doc \
--enable-libfdk-aac --enable-libx264 --enable-libx265 \
--disable-lzma --disable-bzlib --disable-iconv
make -j8
make install
```

### 编译问题

- **x265在make install时报错**

  ```shell
  $ make install
  common/CMakeFiles/common.dir/compiler_depend.make:4: *** 多个目标匹配。 停止。
  make[1]: *** [CMakeFiles/Makefile2:253：common/CMakeFiles/common.dir/all] 错误 2
  make: *** [Makefile:136：all] 错误 2
  ```

  *使用 cmake --install . 替换 make install*

  

  
- **ffmpeg在configure时报错**

  ```shell
  ERROR: x265 not found using pkg-config
  ```

  *需要修改x265.pc文件，把-lstdc++放在Libs下*
  ```shell
  ...
  Name: x265
  Description: H.265/HEVC video encoder
  Version: 4.0
  Libs: -L${libdir} -lx265 -lstdc++
  Libs.private: -lstdc++ -lgcc_s -lgcc -lgcc_s -lgcc
  Cflags: -I${includedir}
  ```