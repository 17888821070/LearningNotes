# 常用变量

## 环境变量

变量名|描述|
-|-|
CMAKE_BINARY_DIR, PROJECT_BINARY_DIR|如果是 in source 编译，指得就是工程顶层目录；如果是 out-of-source 编译，指的是工程编译发生的目录
CMAKE_SOURCE_DIR, PROJECT_SOURCE_DIR|工程顶层目录
CMAKE_CURRENT_SOURCE_DIR|当前处理的 CMakeLists.txt 所在的路径
CMAKE_CURRRENT_BINARY_DIR|如果是 in-source 编译，它跟 CMAKE_CURRENT_SOURCE_DIR 一致，如果是 out-of-source 编译，指的是 target 编译目录
EXECUTABLE_OUTPUT_PATH, LIBRARY_OUTPUT_PATH|最终目标文件存放的路径
PROJECT_NAME|通过 PROJECT 指令定义的项目名称

## 编译选项

变量名|描述|
-|-|
BUILD_SHARED_LIBS|使用 ADD_LIBRARY 时生成动态库
BUILD_STATIC_LIBS|使用 ADD_LIBRARY 时生成静态库
CMAKE_C_FLAGS|设置 C 编译选项,也可以通过指令 ADD_DEFINITIONS() 添加
CMAKE_CXX_FLAGS|设置 C++ 编译选项,也可以通过指令 ADD_DEFINITIONS() 添加