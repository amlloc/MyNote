---
title: smali注入代码库
date: 2018-09-16 22:36:24
categories: codenote
tags: [Android, Reverse, 填坑记录]
---

<!--more-->

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
如果还有其他需要，详见这里提供一个链接github:
https://github.com/bingghost/SmaliLibrary
把里面的代码抠出来用即可。