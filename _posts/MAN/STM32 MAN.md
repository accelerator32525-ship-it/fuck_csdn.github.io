---
title: STM32 MAN
date: 2026-5-31
categories: [UNIX,MCUs,MAN]
tags: [UNIX,MCUs,MAN]
---



# STM32 Unix MAN

[TOC]



#### 0. 背景知识

| 项目               | 用途                                                         |
| :----------------- | ------------------------------------------------------------ |
| VSCode Cmake Tools | 替代build.sh，创建目录、调用Cmake、Ninja、objcopy以及Openocd等 |
| Cmake              | Meta-Build System: CMakeList.txt，其中定义项目结构、建立引用与依赖、选择编译工具链，最后自动生成build.ninja |
| Ninja/Make         | Build Tool: 调用编译器根据build.ninja处理c文件               |
| arm-none-eabi-gcc  | Compiler：交叉编译执行器，交叉指编译执行与运行架构不同，如AArch64编译Arm32固件 |
| Openocd            | Flash：将编译生成文件写入单片机                              |

VSCode Cmake Tools负责Cmake

项目内必须包含：

```file
CMakeLists.txt, toolchain.cmake, xxx.ld, xxx.cfg
```

说明性文件：

```file
settings.json, extensions.json, c_cpp_properties.json
```



#### 1. VSCode插件安装

- C/C++ Extension Pack：C Language高亮等
- Cortex-Debug：
- Cmake Tools：CmakeList创建与代码跳转
- OpenOCD Debug：调试与烧录

可选：

- Linker Script Support：ld文件编辑辅助

#### 2. 文件结构

```file
---Project/
		  |---Core/
		  |     |---Src/
		  |     |    |---main.c/tim.c
		  |     |---Inc/
		  |            |---main.h/stmxxx_hal.h
		  |---Drives/
		  |     |---CMSIS/
		  |     |---stm32xxx_HAL_Driver/
		  |---CMakeList.txt
		  |---CMakeProsets.json
		  |---stm32xxx.ld
		  |---startup_xxx.s
		  |---xxx.ioc # 如果是用CubeMX生成代码
		  / # cmake第一次build之后生成
		  |---cmake/
		  |---Build/
```

#### 3. CMakeList语法

##### 父文件

```cmake
cmake_minium_required(VERSION x.xx)
project(xxx)
add_executable(${PROJECT_NAME} main.c)# 或add library，至少一个生成目标
add_subdirectory(src)# 路径名
```

> Note
>
> 1. Cmake会自动将xxx赋给变量${PROJECT_NAME}，因此无需显式定义。
> 2. add_executable指定生成文件名称，同时可以指定要编译的文件，**但很多时候更倾向于在子文件中使用target_sources来添加文件**（如下）。

##### 子文件

```cmake
target_sources(${PROJECT_NAME} PRIVATE xxx.c)
```



##### CubeMX生成的父文件：

```cmake
set(CMAKE_PROJECT_NAME pwm-gen)# 变量定义方法

project(${CMAKE_PROJECT_NAME})# 必须
add_executable(${CMAKE_PROJECT_NAME})# 必须
add_subdirectory(cmake/stm32cubemx)# 必须（如有），路径

target_link_libraries(库文件名称/可执行文件名称 链接的库文件名称)# 必须

```

> 1. 
>
>    ```cmake
>    ```
>
> 2. 

1. 子文件

```cmake
set(MX_Include_Dirs ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Inc)

# STM32CubeMX generated application sources
set(MX_Application_Src ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/main.c)
set(STM32_Drivers_Src ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/system_stm32f1xx.c

# Interface library for includes and symbols
add_library(stm32cubemx INTERFACE)
target_include_directories(stm32cubemx INTERFACE ${MX_Include_Dirs})
target_compile_definitions(stm32cubemx INTERFACE ${MX_Defines_Syms})
target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${MX_Application_Src})

# Create STM32_Drivers static library
add_library(STM32_Drivers OBJECT)
target_sources(STM32_Drivers PRIVATE ${STM32_Drivers_Src})
target_link_libraries(STM32_Drivers PUBLIC stm32cubemx)

target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${MX_Application_Src})
target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE ${MX_LINK_DIRS})
target_link_libraries(${CMAKE_PROJECT_NAME} ${MX_LINK_LIBS})

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES ADDITIONAL_CLEAN_FILES ${CMAKE_PROJECT_NAME}.map)

```



