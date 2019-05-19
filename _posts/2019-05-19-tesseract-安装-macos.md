---
layout: post
title: 【笔记】tesserract-ocr编译
categories: 编程
tags: 后端
author: adai
---

* content
{:toc}

#macos mojava 编译tesseract-ocr

官网：<https://github.com/tesseract-ocr/tesseract/wiki/Compiling#macos>
##1、安装brew 依赖
```shell
brew install automake autoconf libtool
brew install pkgconfig
brew install icu4c
brew install leptonica
brew install gcc
```
##2、安装 pango
```shell
brew install pango
```

##3、icu4c连接配置
```shell
brew info icu4c
## 由于macos 提供了libicucore.dylib这个库，所以icu4c未连接到 /usr/local目录
icu4c is keg-only, which means it was not symlinked into /usr/local,
because macOS provides libicucore.dylib (but nothing else).

## 需要设置环境变量
If you need to have icu4c first in your PATH run:
  echo 'export PATH="/usr/local/opt/icu4c/bin:$PATH"' >> ~/.bash_profile
  echo 'export PATH="/usr/local/opt/icu4c/sbin:$PATH"' >> ~/.bash_profile

#编译参数需要连接icu4c
For compilers to find icu4c you may need to set:
  export LDFLAGS="-L/usr/local/opt/icu4c/lib"
  export CPPFLAGS="-I/usr/local/opt/icu4c/include"
#pkg-config需要加入icu4c
For pkg-config to find icu4c you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/icu4c/lib/pkgconfig"
```

##4、编译运行
```shell
./configure cairo_LIBS=-L/usr/local/Cellar/cairo/1.16.0/lib cairo_CFLAGS=-I/usr/local/Cellar/cairo/1.16.0/include pango_LIBS=-L/usr/local/Cellar/pango/1.42.4_1/lib pango_CFLAGS=-I/usr/local/Cellar/pango/1.42.4_1/include/pango-1.0 CC=gcc-9 CXX=g++-9 CPPFLAGS='-I/usr/local/opt/icu4c/include -I/usr/local/Cellar/glib/2.60.2/include/glib-2.0 -I/usr/local/Cellar/glib/2.60.2/lib/glib-2.0/include -I/usr/local/Cellar/cairo/1.16.0/include/cairo -I/usr/local/Cellar/freetype/2.10.0/include/freetype2 -I/usr/local/Cellar/fontconfig/2.13.1/include' LDFLAGS='-L/usr/local/opt/icu4c/lib -L/usr/local/Cellar/giflib/5.1.4_1/lib -L/usr/local/Cellar/pango/1.42.4_1/lib -L/usr/local/Cellar/glib/2.60.2/lib -L/usr/local/Cellar/cairo/1.16.0/lib -L/usr/local/Cellar/fontconfig/2.13.1/lib'
```
