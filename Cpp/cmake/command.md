# 常用指令

## ADD_DEFINITIONS

```cmake
ADD_DEFINITIONS(-DENABLE_DEBUG -DABC)
```

向 C/C++ 编译器添加 -D 定义

## ADD_DEPENDENCIES

```cmake
ADD_DEPENDENCIES(target-name depend-target1 depend-target2 ...)
```

定义 target 依赖的其他 target，确保在编译本 target 之前，其他的 target 已经被构建

## ADD_EXECUTABLE

```cmake
ADD_EXECUTABLE(<name> [source1] [source2 ...])
```

利用源码文件生成目标可执行程序

## ADD_LIBRARY

```cmake
ADD_LIBRARY(<name> [STATIC | SHARED | MODULE] [source1] [source2 ...])
```

根据源码文件生成目标库

## ADD_SUBDIRECTORY

```cmake
ADD_SUBDIRECTORY(NAME)
```

添加一个文件夹进行编译，该文件夹下的 CMakeLists.txt 负责编译该文件夹下的源码

NAME 是想对于调用 add_subdirectory 的 CMakeListst.txt 的相对路径

## AUX_SOURCE_DIRECTORY

```cmake
AUX_SOURCE_DIRECTORY(dir VARIABLE)
```

发现一个目录下所有的源代码文件并将列表存储在一个变量中

## CMAKE_MINIMUM_REQUIRED

```cmake
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
```

定义 cmake 的最低兼容版本

## SET

```cmake
SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

# 定义 SRC_LST 代替后面的字符串
SET (SRC_LST main.c other.c)
```

用来显式的定义变量，使用 `${NAME}` 来获取变量的名称