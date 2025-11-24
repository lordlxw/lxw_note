# CMake 学习

链接：https://www.bilibili.com/video/BV14s4y1g7Zj/?spm_id_from=333.337.search-card.all.click&vd_source=f93c2f87ac116d0ab5a7d34711f601c7

> **CMake 是一个“项目生成器”（或“构建系统生成器”），而 Makefile 是“构建系统”本身使用的具体指令集。**
>
> CMake 比 MakeFile 高级一些
>
> CMake本身不能直接生成可执行程序，它只是生成构建文件的工具。



> ToolChain
>
> 预处理——>编译——>汇编（.obj/.o）——>链接（打包）——>exe
>
> toolchain:
>
> 1.预处理：处理代码中的宏等
>
> 2.编译：使用编译器生成汇编文件
>
> 3.汇编：处理汇编文件，生成二进制文件，.o/.obj
>
> 4.链接：得到可执行程序



> ### Makefile
>
> - 直接构建工具 - 直接描述如何编译
>
> - 平台特定 - 需要为每个平台写不同的Makefile
>
> - 语法简单 - 但功能有限
>
> ### CMake
>
> - 构建系统生成器 - 生成其他构建系统的文件
>
> - 跨平台 - 一套CMakeLists.txt适用于多个平台
>
> - 功能强大 - 支持复杂项目结构





## CMakeLists.txt 

> CMakeLists.txt 是CMake的固定文件名，不能随意更改。



### 命令

> 1、cmake_minimum_required(VERSION 3.13)
>
> - 作用: 设置CMake的最低版本要求
>
> - 参数: VERSION 3.13 - 要求CMake版本至少3.13
>
> - 位置: 必须在CMakeLists.txt的第一行

> 2、project(gtest VERSION ${GOOGLETEST_VERSION} LANGUAGES CXX C)
>
> - 作用: 定义项目基本信息
>
> - 参数:
>
> - gtest - 项目名称
>
> - VERSION ${GOOGLETEST_VERSION} - 项目版本（使用变量）
>
> - LANGUAGES CXX C - 支持的语言（C++和C）

> 3、add_executable：定义工程会生成一个可执行程序
>
> 例子：add_executable(app add.c div.c main.c mult.c sub.c)
>
>
> 注意：编译的时候只会编译cpp/c文件，不会编译.h文件

> 4、执行CMake 命令
>
> ```cmake
> $ cmake CMakeLists.txt文件所在路径
> //在本地就cmake .
> //在上一级目录就cmake ..
> ```





> 5、set
>
> 类似宏 定义变量
>
> ```cmake
> # SET 指令的语法是：
> # [] 中的参数为可选项, 如不需要可以不写
> SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])
> ```
>
> ```set example
> # 方式1: 各个源文件之间使用空格间隔
> # set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)
> 
> # 方式2: 各个源文件之间使用分号 ; 间隔
> set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
> add_executable(app  ${SRC_LIST})
> ```
>
> 





> 6、制定使用的C++标准
>
> ==CMAKE_CXX_STANDARD==关键字
>
> ```cmake
> #增加-std=c++11
> set(CMAKE_CXX_STANDARD 11)
> #增加-std=c++14
> set(CMAKE_CXX_STANDARD 14)
> #增加-std=c++17
> set(CMAKE_CXX_STANDARD 17)
> ```
>
> 



> 7、指定输出路径 
>
> EXECUTABLE_OUTPUT_PATH
>
> ```cmake
> set(HOME /home/robin/Linux/Sort)
> set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
> ```



> 8、add_subdirectory——用于添加子目录到构建系统中。
>
> ```cmake
> add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
> ```
>
> ### 参数说明
>
> - source_dir - 包含CMakeLists.txt的子目录
>
> - binary_dir - 构建输出目录（可选）
>
> - EXCLUDE_FROM_ALL - 排除默认构建（可选）
>
> ### add_subdirectory的作用
>
> - ✅ 添加子目录 - 将子目录纳入构建系统
>
> - ✅ 执行子CMakeLists.txt - 自动执行子目录的CMakeLists.txt
>
> - ✅ 模块化构建 - 支持大型项目的模块化构建
>
> - ✅ 条件构建 - 根据条件选择性地添加子目录
>
> ### 使用场景
>
> - 大型项目 - 将项目分解为多个模块
>
> - 条件构建 - 根据平台或选项选择组件
>
> - 测试和示例 - 可选的测试和示例程序



> 9、add_library
>
> add_library (tlmc_xa_native SHARED) 的目的
>
> 告诉 CMake 要生成一个名为 tlmc_xa_native.dll 的动态链接库！
>
> 静态库就是STATIC



> 10、target_compile_definitions
>
> ```cmake
> target_compile_definitions (tlmc_xa_native PRIVATE TLMC_XA_NATIVE_API_DYNAMIC)
> ```
>
> 







==2025/11/6==

换了个链接学习：还是油管好一些？

https://www.youtube.com/watch?v=1Idu-RNA74w&list=PLalVdRk2RC6o5GHu618ARWh0VO0bFlif4

> 
