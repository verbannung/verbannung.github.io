---
title: "Cmake理解"
date: 2025-12-19T21:46:39+08:00
draft: false
summary: "cmake理解"
tags: ["cmake"]
---

# cmake理解

## cmake
统一的跨平台的构架工具,用于生成makefile(makefile用于生成二进制文件)

## 核心理念

>Targets represent executables, libraries, and utilities built by CMake. Every [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library "(in CMake v3.30.3)"), [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable "(in CMake v3.30.3)"), and [`add_custom_target`](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target "(in CMake v3.30.3)") command creates a target.

### 核心命令
构建可执行文件
```cmake
add_executable(main main.cpp)

```

新增模块
```cmake
add_subdirectory(module)
```

模块连接
```cmake
target_link_libraries(main module)
```

新增模块
```cmake
add_library(module module.cpp)
```


头文件搜索模块
``` cmake
target_include_directories()
```


## 生成流程

模块独立编译(add_library)

模块连接(target_link_libraries)

可执行文件(add_executable)


## 常见问题
### target_include_directories()和add_subdirectory()区别

target_include_directories() 告诉在这个文件里找头文件
add_subdirectory()则是子项目在哪,会进行编译

``` text
# module_a/CMakeLists.txt
add_library(module_a a.cpp)
target_include_directories(module_a PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

---

## 对比示例
```
project/
├── CMakeLists.txt
├── include/
│   └── utils.h
├── src/
│   └── main.cpp
└── libs/
    └── math/
        ├── CMakeLists.txt
        ├── include/
        │   └── math.h
        └── src/
            └── math.cpp
```
	include可以被多个文件使用对外暴露

作为cpp,暴露头文件,但是在最终的连接阶段,由于定义在cpp文件中,那么这个时候就可以共享头文件
## Input

``` dataview
 LIST FROM [[note-cmake理解]]
```


## Reference

[知乎](https://zhuanlan.zhihu.com/p/661284197)
[cmake官网](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Key%20Concepts.html)



