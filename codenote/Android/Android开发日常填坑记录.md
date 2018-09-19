---
title: Android开发日常填坑记录
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录]
---

Android开发零零碎碎
<!--more-->

## 一、给Android Studio项目添加依赖

```
android {
    ...
    repositories{
        flatDir{
            dirs 'libs'
        }
    }
    ...
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation (name:'aar-name', ext:'aar')
    implementation (name:'aar-name', ext:'aar')
}
```
## 二、Android Studio运行app时提示Please select Android SDK '.

报错`Error:Please select Android SDK`

> <img src="Android开发日常填坑记录/1.png">

有两个方法：

*1. 在File->Project Structure中将Build tools version修改，问题解决.* 

*2. 注释掉build.gradle中“buildToolsVersion '26.0.0'”。较新的AS创建的工程，已经默认去掉了该行*

## 三、在Android Studio项目中使用build.gradle进行签名

- 将`.jks`签名文件放在项目模块文件夹下(签名文件可在`Build->Generte signed apk->Create new...`新建`)

![2](Android开发日常填坑记录/2.png)

- 在模块下的`build.gradle`添加如下配置

  ```groovy
  
  android {
      
      ...
  
      signingConfigs {
          release {
              storeFile file("sign_amlloc.jks")
              storePassword 'xxxxxxx'
              keyAlias 'yyyyyyyyy'
              keyPassword 'xxxxxxxxx'
          }
  
          debug {
              storeFile file("sign_amlloc.jks")
              storePassword 'xxxxxxx'
              keyAlias 'yyyyyyy'
              keyPassword 'xxxxxxxxx'
          }
      }
  
       buildTypes {
          release {
              minifyEnabled false
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
              signingConfig signingConfigs.release
          }
  
          debug {
              signingConfig signingConfigs.debug
          }
      }
      ...
  }
  ```

- 此后编译出来的`release`和`debug`版本的app便会使用对应的签名文件进行签名

## 四、使用命令行的方式对AndroidStudio进行打包与清除

有时候启动Android Studio的时候会嫌弃它很重，懒得打开，所以特意整理了一下在命令行里面的命令

- 打包所有渠道的release

  `cd ${project_root}`

  `./gradlew assembleRelease`

- 清除项目

  `cd ${projece_root}`

  `./gradlew clean`

- 打包某个特定渠道的release

  `cd ${project_root} `

  `./gradlew assembleBaiduRelease`或`./gradlew aR`

- 打包debug版本的包

  `./gradlew assembleDebug`或`./gradkew aD`

- 打包特定module

  `./gradlew :${modulename}:assembleRelease`

## 五、/proc/{pid}/maps解读

总共6列，如

`76093000-76096000 r-xp 00000000 b3:19 941        /system/lib/libmemalloc.so`

1. 所处虚拟内存地址(VMA)范围：``76093000-76096000`

   - 在Linux中将进程虚拟空间中的一个段叫做虚拟内存区域VMA（Virtual Memory Area)。
   - VMA对应ELF文件中的segment。
   - ELF文件有section和segment的概念。从链接的角度看，ELF是按照section存储的，事实也的确如此；从装载的角度看，ELF文件又按照segment进行划分，这是为了防止按照section装载时造成的内部碎片。segment相当与是将多个属性（读写执行）相同的section合并在一起进行。program headers 存放segment的信息;section table存放section的信息.

2. VMA权限：`r-xp`

   `r=读，w=写,x=,s=共享,p=私有`

3. 偏移量：`00000000`

   表示VMA对应的segment在映像文件中的偏移。

4. 主设备号和次设备号（大雾）：`b3:19`

5. 映像文件的节点号inode：`941`

6. 映像文件的路径：`/system/lib/libmemalloc.so`

## 六、adb命令

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