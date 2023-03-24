# CMake 学习

**参考文档**

[An Introduction to Modern CMake · Modern CMake](https://cliutils.gitlab.io/modern-cmake/)

[It’s Time To Do CMake Right](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)

[CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

---

## 一个通俗的理解

* 你开了一家包子铺，打算做包子生意。（程序员）

* 随着你的客流量越来越大，你逐渐感觉有些力不从心。每次做包子都要手动活面，拌馅，千篇一律。（手动输指令编译链接）

* 于是你就想弄一个机器，自动帮你进行这些千篇一律的操作，你只需要打开开关按下按钮就可以了。（机器：Makefile，按钮：make 指令）

* 随着你的效率提升了，你想在其他地方开连锁店（不同的操作系统）。但是机器不好使了，某个地方没有小龙虾，只有小凤虾（比如 Windows 下面部分 API 在 Linux 中长另一个样），而另一个地方没有玉米棒子，但用豌豆也能获得差不多的口感（某个库没有对应的平台版本，但有些替代品提供了相同的接口）。由此一来你就得根据不同的地域条件制作不同的机器（编写不同的 Makefile），才能采摘原料，做出包子成品（正确运行 make 指令）。

* 这个时候你想给机器外挂一个自动控制系统，只要输入包子配方以及各种原料在不同地方的替代品，控制系统就能自动设置好机器并开始生产。这样，你只需要做出一份控制系统就可以了。（CMake 的用途：根据不同平台生成对应的 Makefile，然后你就可以使用 make 指令快速便捷的编译生成你需要的程序）。

&emsp;

## CMake 定义

CMake 是一个**跨平台**的编译 (Build) 工具，可以用简单的语句来描述所有平台的编译过程。即使是原作者给出了相关的结构文档，对新手来说建立工程的过程依旧是漫长而艰辛的，因此 CMake 的作用就凸显出来了。原作者只需要生成一份 `CMakeLists.txt` 文档，框架的使用者们只需要在下载源码的同时下载作者提供的 `CMakeLists.txt`，就可以利用 CMake，在“原作者的帮助下“进行工程的搭建。

&emsp;

## 基础语法介绍

在当前文件夹下编写一个 `CMakeLists.txt`，然后输入 `cmake .` 运行。

```cmake
cmake_minimum_required(VERSION 3.24)

project(hello VERSION 1.0)

set(SRC_LIST main.cpp)
set(CMAKE_CXX_STANDARD 17)

message(STATUS "This is BINARY dir " ${hello_BINARY_DIR})
message(STATUS "This is SOURCE dir " ${hello_SOURCE_DIR})

add_compile_options(-Wall -std=c++11 -o2)
add_executable(hello ${SRC_LIST})
```

* `project` 关键词：用来指定工程的名字和支持的语言，默认支持所有语言。例如 `project(hello CXX)` 就既指定了工程名同时代表支持的语言是 C++。该指定形式隐式的定义了两个变量：`<project_name>_BINARY_DIR` 和 `<project_name>_SOURCE_DIR`，在本例中都代表当前目录。如果还像本例中定义了版本类型，同样会隐式定义两个变量：`<project_name>_VERSION_MAJOR` 和 `<project_name>_VERSION_MINOR`。

* `set` 关键词：用来显式的指定变量。`set(SRC_LIST main.cpp)` 就代表 `SRC_LIST` 中包含 `main.cpp`，也可以是 `set(SRC_LIST main.cpp t1.cpp t2.cpp)`

* `message` 关键词：向终端输出⽤户⾃定义的信息，主要包含三种信息：
  
  * `SEND_ERROR`：产⽣错误，⽣成过程被跳过。
  
  * `SATUS`：输出前缀为 -- 的信息。
  
  * `FATAL_ERROR` ⽴即终⽌所有 cmake 过程。

* `add_executable` 关键词：⽣成可执⾏⽂件。`add_executable(hello ${SRC_LIST})` 表示⽣成的可执⾏⽂件名是 hello，源⽂件读取变量 `SRC_LIST` 中的内容。注意⼯程名的 hello 和⽣成的可执⾏⽂件 hello 是没有任何关系的。它的语法格式为 `add_executable (exe_name source1 source2 ... sourceN)`

* `add_compile_options` 关键词用来添加编译参数。

&emsp;

## CMake 常见变量

* `CMAKE_C_FLAGS` gcc 编译选项

* `CMAKE_CXX_FLAGS` g++ 编译选项
  
  ```cmake
  # 在 CMAKE_CXX_FLAGS 编译选项后追加 -std=c++11
  set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  ```

* `CMAKE_BUILD_TYPE` 编译类型(Debug, Release)
  
  ```cmake
  # 设定编译类型为 debug，调试时需要选择 debug
  set(CMAKE_BUILD_TYPE Debug)
  # 设定编译类型为 release，发布时需要选择 release
  set(CMAKE_BUILD_TYPE Release)
  ```

* `CMAKE_BINARY_DIR`, `PROJECT_BINARY_DIR`, `<project_name>_BINARY_DIR`
  
  三个变量指代的内容基本一致，如果是内部构建编译，指的就是工程顶层目录；如果是 外部构建编译，指的就是工程编译发生的目录。

* `CMAKE_SOURCE_DIR`, `PROJECT_SOURCE_DIR`, `<project_name>_SOURCE_DIR`
  
  这三个变量指代的内容是一致的，不论采用何种编译方式，都是工程顶层目录。

* `CMAKE_C_COMPILER`：指定 C 编译器
  `CMAKE_CXX_COMPILER` ：指定 C++ 编译器

* `EXECUTABLE_OUTPUT_PATH` ：可执行文件输出的存放路径。

* `LIBRARY_OUTPUT_PATH`：库文件输出的存放路径。

&emsp;

## 内部构建和外部构建

上述例⼦就是内部构建，他⽣产的临时⽂件特别多，不⽅便清理。外部构建，就会把⽣成的临时⽂件放在 `build` ⽬录下，不会对源⽂件有任何影响，强烈使⽤外部构建⽅式。

1. 建⽴⼀个 `build` ⽬录，可以在任何地⽅，建议在当前⽬录下，`mkdir build`。

2. 进⼊ build 目录，运⾏ `cmake ..`，表示上⼀级⽬录，⽣产的⽂件都在 `build` ⽬录下了。

3. 在 `build` ⽬录下，运⾏ make 来构建⼯程。注意外部构建的两个变量，`<project_name>_SOURCE_DIR` 还是⼯程路径，`<project_name>_BINARY_DIR` 则是编译路径。

&emsp;

## 工程化构建

在实际的项目开发中，我们并不是把所有的源文件都写在同一个目录下，而是分别包含在多个子目录下。

```bash
MacBook-Air temp % tree
.
├── CMakeLists.txt
├── build
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

外层 `CMakeLists.txt` 中：

```cmake
cmake_minimum_required(VERSION 3.24)
project(hello)
add_subdirectory(src bin)
```

`src` 下`CMakeLists.txt` 中：

```cmake
add_executable(hello main.cpp)
```

这里用到了 `add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])` ，这个指令⽤于向当前⼯程添加存放源⽂件的⼦⽬录，并可以指定中间⼆进制和⽬标⼆进制存放的位置。注意 src 目录下必须要有一个 `CMakeLists.txt` 文件。`EXCLUDE_FROM_ALL` 函数是将写的⽬录从编译中排除，如程序中的 example。

&emsp;

## 使用库

```bash
MacBook-Air Example % tree
.
├── CMakeLists.txt
├── MathFunctions
│   ├── CMakeLists.txt
│   ├── MathFunctions.h
│   └── mysqrt.cxx
├── TutorialConfig.h.in
└── tutorial.cxx
```

根据上述文件结构，首先我们需要在 `MathFunctions/CMakeLists.txt` 中声明这个源文件会被当作一个库来使用，即生成库文件。除了说明库的名称，这里还可以通过 `SHARED` 或者 `STATIC` 来指明是动态库还是静态库。

```cmake
# add_library(lib_name [SHARED|STATIC] source1 source2 ... sourceN)
add_library(MathFunctions mysqrt.cxx)
```

在外层的 `CMakeLists.txt` 中，还需要如下声明：

```cmake
cmake_minimum_required(VERSION 3.10)

project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Add MathFunctions to this project
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

# Use target_link_libraries to link the library to our executable
target_link_libraries(Tutorial PUBLIC MathFunctions)

# Add MathFunctions to Tutorial's target_include_directories()
# ${PROJECT_SOURCE_DIR} is a path to the project source.
target_include_directories(Tutorial PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          "${PROJECT_SOURCE_DIR}/MathFunctions"
                          )
```

`target_link_libraries()` 就是将之前打包的库，链接到生成的目标上，不然会出现光声明，没定义的错误，类似于 Makefile 中指定 g++ 编译器的 `-l` 参数。

`target_include_directories()` 用来将子目录中的头文件包含到目标中，类似于 Makefile 中指定 g++ 编译器的 `-I` 参数。
