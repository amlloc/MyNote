### vscode的配置

#### C/C++调试及运行相关配置

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

> 参考链接:https://www.jianshu.com/p/609345dfc9bd 