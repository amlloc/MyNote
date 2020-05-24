---
title: IDE折腾笔记
date: 2018-12-10 21:20:15
categories: others
tags: IDE
---

各种IDE折腾集合

目前主要使用：

- Android Studio
- vscode
- sublime
- Typora

<!--more-->

### vscode的配置

#### C/C++调试及运行相关配置

- launch.json

  这个文件表示vscode的配置运行文件

  ```json
  // https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
  {
      "version": "0.2.0",
      "configurations": [
          {
              "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
              "type": "cppdbg", // 配置类型，cppdbg对应cpptools提供的调试功能；可以认为此处只能是cppdbg
              "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
              "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
              "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
              "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
              "cwd": "${fileDirname}", // 调试程序时的工作目录，此为工作区文件夹；改成${fileDirname}可变为文件所在目录
              "environment": [], // 环境变量
              "externalConsole": true, // 为true时使用单独的cmd窗口，与其它IDE一致；18年10月后设为false可调用VSC内置终端
              "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
              "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb。但我没试过lldb
              "miDebuggerPath": "gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要
              "setupCommands": [
                  { // 模板自带，好像可以更好地显示STL容器的内容，具体作用自行Google
                      "description": "Enable pretty-printing for gdb",
                      "text": "-enable-pretty-printing",
                      "ignoreFailures": false
                  }
              ],
              "preLaunchTask": "Compile" // 调试会话开始前执行的任务，一般为编译程序。与tasks.json的label相对应
          }
      ]
  }
  ```


- tasks.json

  此文件可用于配置编译文件命令，一般在`launch.json`文件运行前运行。

  ```json
  // https://code.visualstudio.com/docs/editor/tasks
  {
      "version": "2.0.0",
      "tasks": [
          {
              "label": "Compile", // 任务名称，与launch.json的preLaunchTask相对应
              "command": "gcc", // 要使用的编译器，C++用clang++；如果编译失败，改成gcc或g++试试，还有问题那就是你自己的代码有错误
              "args": [
                  "${file}",
                  "-o", // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
                  "${fileDirname}/${fileBasenameNoExtension}.exe",
                  "-g", // 生成和调试有关的信息
                  "-Wall", // 开启额外警告
                  "-static-libgcc", // 静态链接libgcc，一般都会加上
                  // "--target=x86_64-w64-mingw", // clang的默认target为msvc，不加这一条就会找不到头文件；用gcc或者Linux则掉这一条
                  // "-std=c11", // C++最新标准为c++17，或根据自己的需要进行修改
              ], // 编译命令参数
              "type": "process", // process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
              "group": {
                  "kind": "build",
                  "isDefault": true // 不为true时ctrl shift B就要手动选择了
              },
              "presentation": {
                  "echo": true,
                  "reveal": "always", // 执行任务时是否跳转到终端面板，可以为always，silent，never。具体参见VSC的文档
                  "focus": false, // 设为true后可以使执行task时焦点聚集在终端，但对编译C/C++来说，设为true没有意义
                  "panel": "shared" // 不同的文件的编译信息共享一个终端面板
              },
              // "problemMatcher":"$gcc" // 此选项可以捕捉编译时终端里的报错信息；本文用的是clang，开了可能会出现双重报错信息；只用cpptools可以考虑启用
          }
      ]
  }
  ```

- settings.json

  ```json
  {
      "files.defaultLanguage": "c", // ctrl+N新建文件后默认的语言
      "editor.formatOnType": true, // （对于C/C++）输入分号后自动格式化当前这一行的代码
      "editor.suggest.snippetsPreventQuickSuggestions": false, // clangd的snippets有很多的跳转点，不用这个就必须手动触发Intellisense了
      "editor.acceptSuggestionOnEnter": "off", // 我个人的习惯，按回车时一定是真正的换行，只有tab才会接受Intellisense
      // "editor.snippetSuggestions": "top", // （可选）snippets显示在补全列表顶端，默认是inline
  
      "code-runner.runInTerminal": true, // 设置成false会在“输出”中输出，无法输入
      "code-runner.executorMap": {
          // "c": "cd $dir && clang '$fileName' -o '$fileNameWithoutExt.exe' -Wall -g -O2 -static-libgcc --target=x86_64-w64-mingw -std=c11 && &'$dir$fileNameWithoutExt'",
          // "cpp": "cd $dir && clang++ '$fileName' -o '$fileNameWithoutExt.exe' -Wall -g -O2 -static-libgcc --target=x86_64-w64-mingw -std=c++17 && &'$dir$fileNameWithoutExt'"
          "c": "cd $dir && clang $fileName -o $fileNameWithoutExt.exe -Wall -g -O2 -static-libgcc --target=x86_64-w64-mingw && $dir$fileNameWithoutExt",
          "cpp": "cd $dir && clang++ $fileName -o $fileNameWithoutExt.exe -Wall -g -O2 -static-libgcc --target=x86_64-w64-mingw  && $dir$fileNameWithoutExt"
      }, // 控制Code Runner命令；未注释的仅适用于PowerShell（Win10默认），文件名中有空格也可以编译运行；注释掉的适用于cmd（win7默认），也适用于PS，文件名中有空格时无法运行
      "code-runner.saveFileBeforeRun": true, // run code前保存
      "code-runner.preserveFocus": true, // 若为false，run code后光标会聚焦到终端上。如果需要频繁输入数据可设为false
      "code-runner.clearPreviousOutput": false, // 每次run code前清空属于code runner的终端消息，默认false
      "code-runner.ignoreSelection": true, // 默认为false，效果是鼠标选中一块代码后可以单独执行，但C是编译型语言，不适合这样用
  
      "C_Cpp.clang_format_sortIncludes": true, // 格式化时调整include的顺序（按字母排序）
      "C_Cpp.errorSquiggles": "Disabled", // 因为有clang的lint，所以关掉；如果你看的是别的答主用的不是vscode-clangd，就不要加这个了
      "C_Cpp.autocomplete": "Disabled", // 同上；这几条也可以考虑放到全局里，否则很多错误会报两遍，cpptools一遍clangd一遍
      "C_Cpp.suggestSnippets": false, // 同上
  }
  ```

  

### Windows下的配置简易版本

- launch.json

  这个文件表示vscode的配置运行文件

  ```xml
  {
      // Use IntelliSense to learn about possible attributes.
      // Hover to view descriptions of existing attributes.
      // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
      "version": "0.2.0",
      "configurations": [
          {
      
              "name": "(gdb) Launch",
              "type": "cppdbg",
                  "request": "launch",
              "program": "${workspaceFolder}/a.out",  //此处修改，运行目录下编译后生成的a.out文件
              "args": [],
              "stopAtEntry": false,
              "cwd": "${workspaceFolder}",
              "environment": [],
              "externalConsole": true,
              "MIMode": "gdb",
              "preLaunchTask": "c_pre_build",    //添加此条，配置在运行前进行编译，引号中的名字随便取
               "setupCommands": [{
                  "description": "Enable pretty-printing for gdb",
                  "text": "-enable-pretty-printing",
                  "ignoreFailures": true
              }]
          }
      ]
  }
  ```


- tasks.json

  此文件可用于配置编译文件命令，一般在`launch.json`文件运行前运行。

  ```xml
  {
      // See https://go.microsoft.com/fwlink/?LinkId=733558
      // for the documentation about the tasks.json format
      "version": "2.0.0",
      "tasks": [
          {
              "taskName":"c_pre_build",//这里是launch.json中配置的preLaunchTask字段
              "command": "g++",
              "args": ["-g", "${file}", "-o", "a.out"]
          }
      ]
  }
  ```

### 安装sublime text3 + OmniMarkupPreviewer遇到的坑

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

>  快捷键:
>
> Command +Option +O: 浏览器中预览
> Command+Option+X: 导出HTML
> Ctrl+Alt+C: 拷贝HTML标记至剪贴板

### IDEA安装吐血笔记

- 环境：Ubuntu14.04

不要下载不带SDK的压缩包！不要下载不带SDK的压缩包！不要下载不带SDK的压缩包！重要的事情说三遍！

