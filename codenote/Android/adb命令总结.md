---
title: adb命令总结
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录]

---
adb的命令总结
<!--more-->

## adb命令总结

- 启动apk

  `adb shell am start -n 包名 /. 类名`

- 关闭APK

  `adb shell am force-stop 包名`

- 对某一台设备进行操作

  - 获取连接电脑的手机设备号:`adb devices`

  - 对特定设备进行操作`adb -s <device id> <other>`

    如：`adb -s <device id> shell am start -n 包名 /.类名`

- 查看log

  - 清空log

    `adb logcat -c`

  - 以二进制形式输出日志

    `adb logcat -B`

  - 过滤项解析

    - ` <tag>[:priority] `, 标签:日志等级, 默认的日志过滤项是 " `*:I `"

      `adb logcat WifiHW:D *:S`

    - 可以同时设置多个过滤器

      `adb logcat WifiHW:D dalvikvm:I *:S`
      
    - 输出指定包名的日志
    ```shell
    adb logcat | grep -F "`adb shell ps | grep com.amlloc.test | cut -c10-15`"
    ```