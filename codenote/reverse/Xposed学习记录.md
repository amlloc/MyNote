---
title: Xposed记录
date: 2019-01-15 22:36:24
categories: codenote
tags: [Android, Reverse, Xposed]
---

记录重温高版本Xposed的学习，体验上相比于低版本Xposed，高版本的Xposed构建项目更加方便。在手机上安装Xposed需要root，网上有很多资料，这里不再赘述。
<!--more-->

## 创建Xposed工程

- 在项目的`build.gradle`添加依赖

  `compileOnly 'de.robv.android.xposed:api:82`

- 在`AndroidManifest.xml`中的`application`标签下添加如下数据元

  ```xml
  <meta-data
              android:name="xposedmodule"
              android:value="true"/>
          <meta-data
              android:name="xposeddescription"
              android:value="xposed hookdemo"/>
          <meta-data
              android:name="xposedminversion"
              android:value="82"/>
  ```

- 安装后手机即可发现新的Xposed模块