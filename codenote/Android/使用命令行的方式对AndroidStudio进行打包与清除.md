---
title: 使用命令行的方式对AndroidStudio进行打包与清除
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录]


typora-copy-images-to: Android开发日常填坑记录

---

<!--more-->

## 使用命令行的方式对AndroidStudio进行打包与清除

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