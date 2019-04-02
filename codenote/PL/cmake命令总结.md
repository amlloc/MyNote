---
title: cmake命令总结
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录, cmake]

---
常用cmake命令记录

<!--more-->

## cmake一些窍门

cmake可以指定具体的Generator进行编译，如果没有指定，则会尽可能猜测合理的Generator。一旦确定了Generator，cmake会使用符合的生成器为你的Generator去生成合适的文件。

如`cmake -G "Unix Makefiles" path/to/llvm/source/root`

使用`cmake -help`去查询合适的选项：

```
The following generators are available on this platform:
  Unix Makefiles               = Generates standard UNIX makefiles.
  Ninja                        = Generates build.ninja files.
  Watcom WMake                 = Generates Watcom WMake makefiles.
  CodeBlocks - Ninja           = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles  = Generates CodeBlocks project files.
  CodeLite - Ninja             = Generates CodeLite project files.
  CodeLite - Unix Makefiles    = Generates CodeLite project files.
  Sublime Text 2 - Ninja       = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                               = Generates Sublime Text 2 project files.
  Kate - Ninja                 = Generates Kate project files.
  Kate - Unix Makefiles        = Generates Kate project files.
  Eclipse CDT4 - Ninja         = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles= Generates Eclipse CDT 4.0 project files.	
```



## 常用的一些cmake变量

可以直接查询cmake手册或者`cmake --help-variable VARIABLE_NAME`进行查询

- CMAKE_BUILD_TYPE:STRING

  Sets the build type for make-based generators. Possible values are Release, Debug, RelWithDebInfo and MinSizeRel. If you are using an IDE such as Visual Studio, you should use the IDE settings to set the build type. Be aware that Release and RelWithDebInfo use different optimization levels on most platforms.

- CMAKE_INSTALL_PREFIX:PATH

  Path where LLVM will be installed if “make install” is invoked or the “install” target is built.

- LLVM_LIBDIR_SUFFIX:STRING

  Extra suffix to append to the directory where libraries are to be installed. On a 64-bit architecture, one could use -DLLVM_LIBDIR_SUFFIX=64 to install libraries to /usr/lib64.

- CMAKE_C_FLAGS:STRING

  Extra flags to use when compiling C source files.

- CMAKE_CXX_FLAGS:STRING

  Extra flags to use when compiling C++ source files.

## cmake命令总结

指令总结方面可以直接查看这个官方链接：https://cmake.org/cmake/help/v3.3/manual/cmake-commands.7.html

这里总结一些常用的命令及使用示例：

- set

  - `set (<variable> <value>... [PARENT_SCOPE])`

    设置某个变量的值，作用域为当前函数或者当前文件夹，`[PARENT_SCOPE]`这个参数没用过，参照官方文档可知似乎是将此变量设置在上一级作用域中（也就是父目录或者调用函数）

    > 使用变量的方法：${<variable>}

- file

  - `file(WRITE <filename> <content>...)`

    将content写入指定文件，如果文件不存在则创建文件；如果文件存在则覆盖该文件

    示例：`file(WRITE src/hello.txt "hello world")`

  - `file(APPEND <filename> <content>...)`

    示例：`file(APPEND src/hello.txt "hello world 2")`

    将content以追加的形式写入指定文件

  - `file(READ <filename> <variable> [OFFSET <offset>][LIMIT <max-in>] [HEX])`

    将`<filename>`文件里面的内容从偏移为`<offset>`的地方最多读取`<max-in>`个内容，并放入`<variable>`进行保存，HEX选项为16进制读取。

    示例：`file(READ src/hello.txt context LIMIT 5)`

  - `file(GLOB <variable> [LIST_DIRECTORIES true|false] [RELATIVE <path>] [<globbing-expressions>...])`
    生成`<globbing-expressions>`与之匹配的文件列表并将其存储到`<variable>`。Globbing表达式与正则表达式类似，但更简单。如果RELATIVE指定了flag，则结果将作为给定路径的相对路径返回。

    默认情况下列出GLOB目录 - 如果LIST_DIRECTORIES设置为false，则在结果中省略目录 。

    示例：`file(GLOB utils_src_cc src/utils/*.cc)`

- find_library()

  - `find_library (<VAR> name1 [path1 path2 ...])`

    此命令用于查找库。创建名为`<VAR>`的缓存条目以存储此命令的结果。如果找到库，则结果存储在变量中，除非清除变量，否则不会重复搜索。如果找不到任何结果，结果将是` <VAR>-NOTFOUND`，并在下次使用相同变量调用`find_library`时再次尝试搜索。搜索的库的名称由name1指定(值为库的名称，如`libz.a`，则`name`值为`z`)。可以在PATHS参数之后指定其他搜索位置。

    示例：`find_library( log-lib log)`

- add_library()

  使用指定的源文件将库添加到项目中。

  - `add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL] source1 [source2 ...])`

    添加一个根据命令调用中列出的源文件,构建名为`<name>`的库目标。它`<name>`对应于逻辑目标名称，并且在项目中必须是全局唯一的。构建的库的实际文件名是基于本机平台（例如lib<name>.a or <name>.lib）的约定构造的。

    示例：

    ```cmake
    add_library(
            ${PROJECT_NAME}
    
            SHARED
    
            ${src1} //可以配合file命令，但官方不建议这么做，理由是cmake无法知晓有新的文件加入
            ${src2}
            ${src3}
            )
    ```

  - `add_library(<name> <SHARED|STATIC|MODULE|UNKNOWN> IMPORTED [GLOBAL])`

    导入名字为`<name>`的库

    示例：

    ```cmake
    add_library(myz SHARED IMPORTED)
    add_library(myz SHARED IMPORTED)
    #设置库文件所在路径
    set_target_properties(myz PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI}/libnative-static.a)
    ```

- target_link_libraries()

  - target_link_libraries(<target> ... <item>... ...)

    默认情况下，使用此签名，库依赖项是可传递的。当此目标链接到另一个目标时，链接到此目标的库也将出现在另一个目标的链接行上。这个传递性的“链接接口”存储在`INTERFACE_LINK_LIBRARIES`目标属性，可以通过直接设置属性来覆盖。

    示例：`target_link_libraries(${PROJECT_NAME}  ${log-lib} ${z-lib})`

- include_directories(）

  - `include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])`

    将给定目录添加到编译器用于搜索包含文件的目录中。相对路径被解释为相对于当前源目录。include目录被添加到当前CMakeLists文件的目录属性 INCLUDE_DIRECTORIES 之中。他们对当前CMakeLists文件中INCLUDE_DIRECTORIES属性中也加入了每个目标的目标属性。目标属性值是生成器使用的值。

  示例：

  ```cmake
  include_directories(
          src/dir1
          src/dir2
          src/dir3
          src/dir4
          )
  ```

- 两个Cmakelists.txt的例子：

  - 第一个

    ```cmake
    cmake_minimum_required(VERSION 3.4.1)
    
    set(PROJECT_NAME AMLLOC)
    
    file(WRITE amlloc/hello.txt "asdasda")
    file(READ amlloc/hello.txt context LIMIT 5)
    file(APPEND amlloc/hello.txt ${context})
    
    file(GLOB amlloc_src amlloc/src/*.cpp)
    file(GLOB utils1_src_cc amlloc/src/utils/*.cc)
    file(GLOB utils2_src amlloc/src/utils2/*.cpp)
    file(GLOB utils3_src_cc amlloc/src/utils3/*/*.cc)
    file(GLOB utils3_src_c amlloc/src/utils4/*/*.c)
    file(GLOB utils4_src amlloc/src/utils5/*.c)
    
    
    
    find_library( log-lib log)
    find_library( z-lib z)
    
    include_directories(
            amlloc/src/utils/include
            amlloc/src/include
            amlloc/src/utils1
            amlloc/src/utils2/md5
            amlloc/src/utils2
            amlloc/src/utils3
            )
    
    
    
    add_library(
            ${PROJECT_NAME}
    
            SHARED
    
            ${amlloc_src}
            ${utils1_src_cc}
            ${utils2_src}
            ${utils3_src_cc}
            ${utils3_src_c}
            ${utils4_src}
            )
    target_link_libraries(${PROJECT_NAME}  ${log-lib} ${z-lib})
    ```

  - 第二个

    首先看Project的目录结构

    ```
    ├── CMakeLists.txt
    ├── main.cpp
    └── module
        ├── CMakeLists.txt
        ├── GameSave.h
        ├── s_utils.cc
        ├── s_utils.h
        ├── log.cc
        ├── log.h
        ├── utils.cc
        └── utils.h

    ```
    

    子目录：

    ```cmake
    set(PROJECT_NAME module_demo)
    file(GLOB module_src ./*.cc)
    
    add_library(${PROJECT_NAME} SHARED ${module_src})
    find_library(log-lib log)
    target_link_libraries(${PROJECT_NAME}  ${log-lib})
    ```

    父目录：

    ```cmake
    cmake_minimum_required(VERSION 3.4.1)
    
    #find library log and the result is stored
    #in the var log-lib
    find_library(log-lib log)
    
    #get the .so directory
    set(DISTRIBUTION_DIR ${CMAKE_SOURCE_DIR}/../../../libs)
    
    add_library(gamesavesaty_module SHARED IMPORTED)
    
    set_target_properties(gamesavesaty_module
                          PROPERTIES IMPORTED_LOCATION
                          ${DISTRIBUTION_DIR}/armeabi/libmodule.so
                          )
    
    add_library(gamesavety SHARED IMPORTED)
    
    set_target_properties(gamesavety
                          PROPERTIES IMPORTED_LOCATION
                          ${DISTRIBUTION_DIR}/armeabi/libgamesavety.so
                          )
    
    
    
    #set c++ compiler
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    
    set(PROJECT_NAME demo)
    
    
    #include the module header file
    include_directories(./module)
    
    #add source file in module
    add_subdirectory(module)
    
    #Adds a library target called PROJECT_NAME to be built
    #from the source file main.cpp and files in module.
    add_library(${PROJECT_NAME} SHARED main.cpp)
    
    include_directories(${DISTRIBUTION_DIR}/armeabi)
    
    #Specify libraries module_demo and log-lib to
    #use when linking project demo.
    target_link_libraries(${PROJECT_NAME} module_demo gamesavety gamesavesaty_module ${log-lib})
    ```