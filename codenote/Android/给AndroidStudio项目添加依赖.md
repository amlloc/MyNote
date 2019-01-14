---
title: 给AndroidStudio项目添加依赖
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录]

---
依赖脚本
<!--more-->

## 给Android Studio项目添加依赖

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