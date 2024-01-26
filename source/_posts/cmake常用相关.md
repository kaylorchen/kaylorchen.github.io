---
title: cmake常用相关 
comments: true
date: 2023-05-06 10:41:15
tags:
  - Linux
categories:
  - Linux
  - cmake
---

# find_package
## 检查需要的包的包名
```bash
╰─ dpkg -L libopencv-dev | grep 'cmake'                
/usr/lib/x86_64-linux-gnu/cmake
/usr/lib/x86_64-linux-gnu/cmake/opencv4
/usr/lib/x86_64-linux-gnu/cmake/opencv4/OpenCVConfig-version.cmake
/usr/lib/x86_64-linux-gnu/cmake/opencv4/OpenCVConfig.cmake
/usr/lib/x86_64-linux-gnu/cmake/opencv4/OpenCVModules-release.cmake
/usr/lib/x86_64-linux-gnu/cmake/opencv4/OpenCVModules.cmake
```
从上面的可以看到这个库的cmake文件是OpenCVConfig.cmake， 所以它的库名字是OpenCV, 所以CMakeLists.txt可以这么写：
```cmake
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS}) # Not needed for CMake >= 2.8.11
target_link_libraries(MY_TARGET_NAME ${OpenCV_LIBS})
```
find_package 会对应设置以下几类变量：

- [PackageName]_FOUND: 如果找到了包，则会设置为TRUE，否则为FALSE。 

- [PackageName]_INCLUDE_DIRS 或 [PackageName]_INCLUDES: 包含目录列表，你可以用这个变量来将包含目录添加到你的项目中。

- [PackageName]_LIBRARIES 或 [PackageName]_LIBS: 库文件列表，用来链接你的项目和依赖库。

- [PackageName]_DEFINITIONS: 编译器定义列表，需要添加到你的编译器命令行中去，以便使用库。

# Pkg-config
pkgconfig需求的是一个pc文件，我们可以查找库的pc文件，同上面的方法
```bash
╰─ dpkg -L libopencv-dev | grep 'pc'
/usr/lib/x86_64-linux-gnu/pkgconfig/opencv4.pc
```
这里找打了opencv4这个名字，需要写到pkg_check_modules的第三个参数

下面是一个CMakeLists.txt例子
```cmake
# CMakeLists.txt
# A sample CMake project that uses pkg-config to find a library (let's say, 'libexample')

cmake_minimum_required(VERSION 3.0)
project(ExampleProject)

# It's a good practice to set the minimum required version of pkg-config
find_package(PkgConfig REQUIRED)

# Use pkg-config to check if 'libexample' is installed on the system
pkg_check_modules(LIBEXAMPLE REQUIRED libexample)

# Include directories for 'libexample'
include_directories(${LIBEXAMPLE_INCLUDE_DIRS})

# Link directories for 'libexample'
link_directories(${LIBEXAMPLE_LIBRARY_DIRS})

# Add executable target with source files
add_executable(example_executable main.cpp)

# Link the executable to the required library
target_link_libraries(example_executable ${LIBEXAMPLE_LIBRARIES})

# Add any compiler definitions or compile options if necessary
add_definitions(${LIBEXAMPLE_CFLAGS_OTHER})
```
