---
title: apk无法调试的问题
date: 2018-09-16 22:36:24
categories: codenote
tags: [Android, Reverse, 填坑记录]
---
<!--more-->

## apk无法调试的问题

参考链接：

[雪一梦：Android So动态调试之反jdb附加的解决大法](https://blog.csdn.net/feibabeibei_beibei/article/details/52744231)

如果需要判断一个apk是否为可调试的，则必须满足以下两个要求中任意一个

- `AndroidManifest.xml`文件`Application`包含`android:debuggable="true"`
- 文件`/default.prop`中的`ro.debuggable`的值为1

目前有四种方式可以解决这个问题

- 在`root`手机中安装`Xposed`与`xinstaller`插件
  使用方法：
  - 下载 Xposed 框架,并激活,再下载 xinstaller 插件安装;
  - 开启模块,点击 xinstall 插件设置专家模式,进入其他设置,开启调试应用,最后在 xposed 中激活重启

​        短处：apk中很有可能会有反`xposed`代码的存在，因此需要先过掉这个障碍才行。

- 直接通过反编译改包的方法

  缺点多多，不解释

- 如果我们在真机,则可以修改根目录下的 default.prop 文件,将里
  面的 ro.debuggable =1。

  - 从 Google 官方网站下载到 boot.img;
  - 使用工具(abootimg,gunzip, cpio)把 boot.img 完全解开,获取到 default.prop;
  - 修改 default.prop;
  - 把修改后的文件重新打包成 boot_new.img;
  - 使用 fastboot 工具把 boot_new.img 刷入设备(fastboot
    flash boot boot_new.img);

- ptrace 注入到 init 进程,然后修改内存中的`default.prop `文件属性值

  - 拷贝拷贝` mprop `到`/data/`目录下;
  - `./mprop ro.debuggable 1;`
  - `getprop ro.debuggable;`(查看此时 ro.debuggable 在内存
    中的值)
  - stop;start(重启 adbd 进程);

  我们在上面的第三步的时候查看好像没有改过来,分析原因为:该工具是通过 ptrace 修改 init 进程中的内存,然而 4.X 系统强制开启了 selinux;因此这个时候我们需要设置 Selinux 的状态.