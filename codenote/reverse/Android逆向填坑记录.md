---
title: Android逆向填坑记录
date: 2018-09-16 22:36:24
categories: codenote
tags: [Android, Reverse, 填坑记录]
---

Android逆向零零碎碎
<!--more-->

## 动态调试步骤

- 打开monitor

  `cd ${SDK_PATH}/tools/`

  `./monitor`

  备注：没有开没法附加，具体原因暂时不明

- 将android_server push进可执行文件夹

  `adb push /home/IDA/dbgsrv/android_server ` 

  `/data/local/tmp/andser`

  `adb shell`

  `su`

  `cd /data/local/tmp`

  `chmod 777 andser`

  `./andser -p12345`

- open terminal

  `ctrl + t`

- 转发端口

  `adb forward tcp:12345 tcp:12345`

- 以启动模式启动,停在加载so文件之前

  `adb shell am start -D -n packagename/clasdx	sname`

- IDA附加进程

  `Debuger`->`Process option`->`Hostname:localhost Port:1234`->`attach...`

- jdb附加
  `jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=8700`

## IDA dump脚本

```c
auto fp, begin, end, dexbyte;
  fp = fopen("/home/amlloc/dump_shell.dex", "wb");
  begin = 0x761a1000;
  end = 0x763b6000;
  for ( dexbyte = begin; dexbyte < end; dexbyte ++ )
      fputc(Byte(dexbyte), fp);
```



## smali注入代码库
利用Android Killer的插入代码管理器可以很方便地组织管理自己的代码库，这里我列出几个用处较大的代码

```
#LoadLibrary
const-string v0, "so name"

invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```

```
#Log
const-string v0, "you message"

invoke-static {v0}, Lcom/android/killer/Log;->LogStr(Ljava/lang/String;)V

```

```
#LogUtils
#打印int，String，Intent ,HashMap ,List ,null
invoke-static {vx}, Lcom/mtools/logutils/LogUtils;->d(Ljava/lang/Object;)V


#打印float

invoke-static {vx}, Ljava/lang/Float;->valueOf(F)Ljava/lang/Float;

move-result-object v1

invoke-static {v1}, Lcom/mtools/logutils/LogUtils;->d(Ljava/lang/Object;)V


#打印double,如传入v0,则需要写成invoke-static {v0, v1}

invoke-static {vx, vx+1}, Ljava/lang/Double;->valueOf(D)Ljava/lang/Double;

 move-result-object v1

 invoke-static {v1}, Lcom/mtools/logutils/LogUtils;->d(Ljava/lang/Object;)V


#打印long,如传入v0,则需要写成invoke-static {v0, v1}
invoke-static {vx, vx+1}, Ljava/lang/Long;->valueOf(J)Ljava/lang/Long;

move-result-object v1

invoke-static {v1}, Lcom/mtools/logutils/LogUtils;->d(Ljava/lang/Object;)V


#打印json
invoke-static {vx}, Lcom/mtools/logutils/LogUtils;->json(Ljava/lang/Object;)V


#打印xml
invoke-static {vx}, Lcom/mtools/logutils/LogUtils;->xml(Ljava/lang/Object;)V
```

```
#mothodTrace 用于追踪函数也可用来性能测试，需要读写权限
#mothodTraceStart
invoke-static {}, Landroid/os/Debug;->startMethodTracing()V
#mothodTraceStop
invoke-static {}, Landroid/os/Debug;->stopMethodTracing()V
```

```
#dump出函数堆栈， Thread.dumpStack();
invoke-static{}, Ljava/lang/Thread;->dumpStack()V
```

```
#Toast
const-string v0, "you message"

const/4 v1, 0x1

invoke-static {p0, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;

move-result-object v0

invoke-virtual {v0}, Landroid/widget/Toast;->show()V
```

```
#等待调试
invoke-static {}, Landroid/os/Debug;->waitForDebugger()V
```

```
#写权限auto fp, begin, end, dexbyte;
  fp = fopen("/home/amlloc/dump_shell.dex", "wb");
  begin = 0x761a1000;
  end = 0x763b6000;
  for ( dexbyte = begin; dexbyte < end; dexbyte ++ )
      fputc(Byte(dexbyte), fp);
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
#读权限
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```
如果还有其他需要，详见github:
https://github.com/bingghost/SmaliLibrary
把里面的代码抠出来用即可。

## Arm nop
`00 00 A0 E1`

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