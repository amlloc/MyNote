---
title: clang + llvm + vscode 配置记录
date: 2019-04-01 20:00:23
categories: others
tags: Blog
---

Ubuntu 14.04下 clang安装，vscode针对clang的配置

<!--more-->

环境： Ubuntu 14.04，已安装cmake

clang版本：7.0.1

安装方式：源码编译安装

## clang  + llvm安装

- 在[clang下载链接](http://releases.llvm.org/download.html)下载对应的clang源码版本和llvm版本

- 解压下载到的`llvm-7.0.1.src.tar.xz`文件并重命名为`llvm-7.0.1`，解压`cfe-7.0.1.src.tar.xz`并重命名为`clang`,完成之后目录树为：

  ```
  ./------当前目录
  ....llvm-7.0.1
       ...
  ........tools
  ............clang
  ```

- 在当前目录下`mkdir build`

  > 在高版本的llvm中，编译目录不能在llvm源码的子目录下进行

- `cd build`

- `cmake -DLLVM_ENABLE_PROJECTS=clang -G "Unix Makefiles" ../llvm-7.0.1`

- `make -j4`（j4代表使用4个cpu进行编译）,这一步比较消耗时间

- 进入`build/bin`下发现clang，clang++，clang-7等编译好的文件

- 在`build`目录下`sudo make install `进行安装

- `clang --version`

