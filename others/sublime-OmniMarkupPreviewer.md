---
title: OmniMarkupPreviewer+chrome
categories: others
tags: others
---
安装sublime text3 + OmniMarkupPreviewer遇到的坑
<!--more-->
# 0xend
安装就不说了，百度一大把
打开Preferences->Package Settings->OmniMarkupPreviewer->Settings-Defult 
此文件可对该插件进行最基本的功能设置。

找到如下代码，该代码是快捷键后执行的命令

```
//   Launching preview using Google Chrome in OS X:
//      ["open", "-a", "Google Chrome", "{url}"]
"browser_command": [],
```
之所以打开的是空页面，是因为打开chrome并指定网页的命令已经改为:
```
google-chrome <url>
```
于是可以修改文件:
```
"browser_command":["google-chrome", "{url}"]
```
问题得以解决
# 0x01
快捷键:
```
Command +Option +O: 浏览器中预览
Command+Option+X: 导出HTML
Ctrl+Alt+C: 拷贝HTML标记至剪贴板
```
