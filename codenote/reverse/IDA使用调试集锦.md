---
title: IDA使用调试集锦
date: 2018-09-16 22:36:24
categories: codenote
tags: [Android, Reverse, 填坑记录]
---

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

## Arm nop
`00 00 A0 E1`
