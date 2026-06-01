---
title: STM32 MAN
date: 2026-5-31
categories: [UNIX,MCUs,MAN]
tags: [UNIX,MCUs,MAN]
---



# STM32 Unix MAN

[TOC]

#### 0. 背景知识

##### 软件及插件

| 项目               | 用途                                                         |
| :----------------- | ------------------------------------------------------------ |
| VSCode Cmake Tools | 替代build.sh，自动输入创建目录、调用Cmake、Ninja、objcopy以及Openocd等的代码 |
| CMake              | Meta-Build System: CMakeList.txt，其中定义项目结构、建立引用与依赖、选择编译工具链，最后自动生成build.ninja |
| Ninja/Make         | Build Tool: 调用编译器根据build.ninja处理c文件               |
| arm-none-eabi-gcc  | Compiler：交叉编译执行器，交叉指编译执行与运行架构不同，如AArch64编译Arm32固件 |
| 重要：STM32CLT**** | **包含CMake、Ninja、arm-none-eabi-gcc等重要toolchain**       |
| Openocd            | Flash：将编译生成文件写入单片机                              |

**重要：**使用Homebrew安装的arm-none-eabi-gcc似乎会出现问题，Homebrew似乎没有安装Newlib

项目内必须包含：

```file
CMakeLists.txt, toolchain.cmake, xxx.ld, xxx.cfg
```

> CMakeList文件规定CMake编译的具体方法，语言及源文件位置
>
> toolchain文件规定CMake链接的GCC库，保证链接到arm-none-eabi-gcc库
>
> ld文件规定控制器的ROM/FLASH与RAM起始地址
>
> cfg文件用于调试，配合OpenOCD仿真等等

说明性文件：

```file
settings.json, extensions.json, c_cpp_properties.json
```

> settings.json规定了CMake的文件位置
>
> ```json
> {
>     "files.autoSave": "onFocusChange",
>     "python.defaultInterpreterPath": "/Users/admin/venv/bin/python",
>     "python.analysis.typeCheckingMode": "standard",
>     "window.confirmSaveUntitledWorkspace": false,
>     "cmake.cmakePath": "/opt/ST/STM32CubeCLT_1.21.0/CMake/bin/cmake",
>     "cmake.additionalBuildProblemMatchers": [
> 
>     ],
>     "cmake.exportCompileCommandsFile": true,
>     "git.openRepositoryInParentFolders": "never",
>     "explorer.confirmDelete": false,
>     "C_Cpp.default.compileCommands": [
> 
>     ],
>     "cmake.additionalCompilerSearchDirs": [
>         "/opt/ST/"
>     ],
>     "cmake.cacheInit": null,
>     "cmake.configureSettings": {
>         
>     }
> }
> ```
>
> 

##### STM32资源介绍

STM32的硬件资源主要由时钟树及中断、ADC、WD

**时钟树：**

![](/Users/admin/Documents/fuck_csdn.github.io/assets/images/2026-05-31_stm32f103c8clock_tree.png)







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

##### 基本语法

**父文件**：

```cmake
cmake_minium_required(VERSION x.xx)
project(xxx)#1 工程名
add_executable(${PROJECT_NAME} main.c)#2 或add_library，至少一个作为编译对象
add_subdirectory(src)#3 定义子文件路径名
```

> Note
>
> 1. Cmake会自动将xxx赋给变量${PROJECT_NAME}，因此无需显式定义。
> 2. add_executable指定生成文件名称，同时可以指定要编译的文件，**但很多时候更倾向于在子文件中使用target_sources来添加文件**（如下）。

**子文件**：

```cmake
cmake_minimum_required(VERSION 3.22)
enable_language(C ASM)
target_sources(${PROJECT_NAME} PRIVATE xxx.c)#0
```



##### CubeMX生成语法

**父文件**：

父文件个人按照其用途等划分为三部分：基本设置（语言和支持的版本控制，编译类型）；工程定义（工程名称，支持语言）；链接文件(定义文件路径变量，添加文件）。

其中各部分之间**应该**不能交换位置，同时部分内的部分命令也**应该**不能随意调换位置，但第三部分**应该**没有限制。

```cmake
cmake_minimum_required(VERSION 3.22)
# Setup compiler settings
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)
# Define the build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")#5
endif()
# Enable compile command to ease indexing with e.g. clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)#6
```

> Note
>
> 该部分包含基本设置，包括CMake版本控制、C版本控制、编译类型等。
>
> 5. 编译类型设置：
>
> | Build Type      | Machine Code Size    | Debug Info | Suitable Applications           |
> | --------------- | -------------------- | ---------- | ------------------------------- |
> | Debug           | 最大                 | 完整       | 调试与开发阶段，配合仿真器Debug |
> | Release         | 最小                 | 无         | 最终发布、烧录版本              |
> | RealWithDebInfo | 较小                 | 部分       | 半实物仿真（没用过）            |
> | MinsizeRel      | 极限小（但速度稍慢） | 无         | 性能受限设备ROM太小（没用过）   |
>
> 6. 设置“生成期间启用/禁用编译命令输出”，编译命令指的是/build/Debug/compile_commands.json文件，该文件与IntelliSence红线有关。

```cmake
# Set the project name
set(CMAKE_PROJECT_NAME pwm-gen)
# Core project settings
project(${CMAKE_PROJECT_NAME}) # must prior to enable_language()
message("Build type: " ${CMAKE_BUILD_TYPE})# not necessarily a must
# Enable CMake support for ASM and C languages
enable_language(C ASM)#3
```

>Note
>
>该部分较为简短，但是是不可或缺的。
>
>3. 必须放在project定义之后

```cmake
# Create an executable object type
add_executable(${CMAKE_PROJECT_NAME})#0

# Add STM32CubeMX generated sources
add_subdirectory(cmake/stm32cubemx)#1


target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE
    xxxx
)#2

target_sources(${CMAKE_PROJECT_NAME} PRIVATE

)#3


target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE

)#4

# Add project symbols (macros)
target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined symbols
)#5

# Remove wrong libob.a library dependency when using cpp files
list(REMOVE_ITEM CMAKE_C_IMPLICIT_LINK_LIBRARIES ob)#6

# Add linked libraries
target_link_libraries(${CMAKE_PROJECT_NAME}
    stm32cubemx
)#7
```

> Note
>
> 0. 同基本语法一节中的说明，很多时候不在这里指定要编译的文件，而是利用target_sources添加。
> 1. 子文件所在文件夹的位置
> 2. d
>
> 4. d
> 5. d
> 6. d
> 7. d

**子文件**

子文件的主要作用是添加自动生成的源文件，因此如果使用CubeMX自动生成初始化代码，自行开发的源必须加到target_sources里，否则不会被编译。

其余添加library等语法，一般嵌入式开发**可能**用不上，了解即可。

```cmake
cmake_minimum_required(VERSION 3.22)
# Enable CMake support for ASM and C languages
enable_language(C ASM)
# STM32CubeMX generated symbols (macros)
set(MX_Defines_Syms 
	USE_HAL_DRIVER 
	STM32F103xB
    $<$<CONFIG:Debug>:DEBUG>
)
```



```cmake
# STM32CubeMX generated include paths
set(MX_Include_Dirs
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Inc)
# STM32CubeMX generated application sources
set(MX_Application_Src
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/main.c)
# STM32 HAL/LL Drivers
set(STM32_Drivers_Src
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/system_stm32f1xx.c)
# Drivers Midllewares

# Link directories setup
set(MX_LINK_DIRS

)
# Project static libraries
set(MX_LINK_LIBS STM32_Drivers ${TOOLCHAIN_LINK_LIBRARIES})


```



```cmake
# Interface library for includes and symbols
add_library(stm32cubemx INTERFACE)
target_include_directories(stm32cubemx INTERFACE ${MX_Include_Dirs})
target_compile_definitions(stm32cubemx INTERFACE ${MX_Defines_Syms})

# Create STM32_Drivers static library
add_library(STM32_Drivers OBJECT)
target_sources(STM32_Drivers PRIVATE ${STM32_Drivers_Src})
target_link_libraries(STM32_Drivers PUBLIC stm32cubemx)

# Add STM32CubeMX generated application sources to the project
target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${MX_Application_Src})

# Link directories setup
target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE ${MX_LINK_DIRS})

# Add libraries to the project
target_link_libraries(${CMAKE_PROJECT_NAME} ${MX_LINK_LIBS})

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES ADDITIONAL_CLEAN_FILES ${CMAKE_PROJECT_NAME}.map)

# Validate code is compatible C standard
if((CMAKE_C_STANDARD EQUAL 90) OR (CMAKE_C_STANDARD EQUAL 99))
    message(ERROR "Generated code requires C11 or higher")
endif()

```

