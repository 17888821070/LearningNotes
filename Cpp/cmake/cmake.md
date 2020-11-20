# cmake

## 单一文件编译

CMakeLists.txt
```
# 最低版本限制
cmake_minimum_required(VERSION 2.6)

# 项目名称
project(demo)

# 指定编译可执行文件
# demo 为生成的可执行文件名称 
add_executable(demo main.cpp)
```

## 同一目录下多个源文件

CMakeLists.txt
```
# 最低版本限制
cmake_minimum_required(VERSION 2.6)

# 项目名称
project(demo)

# 把指定目录下的所有源文件放进变量中
# . 为CMakeLists.txt所在目录
aux_source_directory(. SRC_LIST)

# 指定文件加入变量
set( SRC_LIST
     ./main.cpp
     ./func1.cpp
     ./func2.cpp)

# 指定编译可执行文件 
add_executable(demo ${SRC_LIST})
```

## 不同目录下多个源文件

![](https://img-blog.csdn.net/20180901105703600?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doYWh1MTk4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

CMakeLists.txt
```
# 最低版本限制
cmake_minimum_required(VERSION 2.6)

# 项目名称
project(demo)

# 添加头文件搜索路径
include_directories(test_func test_func1)

# 把指定目录下的所有源文件放进变量中
aux_source_directory (test_func SRC_LIST)
aux_source_directory (test_func1 SRC_LIST1)

# 指定编译可执行文件 
add_executable(demo main.cpp ${SRC_LIST} ${SRC_LIST1})
```

## 正规一点的组织结构

为了项目结构更加清晰，将源文件放在 src 目录下，把头文件放在 include 目录下，生成的对象文件放在 build 目录下，最终输出文件放在 bin 目录下

![](https://img-blog.csdn.net/20180826221632533?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doYWh1MTk4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


添加两个 CMakeLists.txt

![](https://img-blog.csdn.net/20180826222905565?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doYWh1MTk4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

最外层 CMakeLists.txt

```
cmake_minimum_required (VERSION 2.8)

project (demo)

# 添加子目录，将外部目录添加进 build 任务列表中
# 执行时会去 src 目录下找 CMakeLists.txt
add_subdirectory (src)
```

在 src 目录下添加 CMakeLists.txt
这种写法是为了处理需要生成多个最终文件

```
aux_source_directory (. SRC_LIST)

include_directories (../include)

add_executable (main ${SRC_LIST})

# EXECUTABLE_OUT_PATH 和 PROJECT_SOURCE_DIR 是 CMake 自带的预定义变量
# EXECUTABLE_OUTPUT_PATH ：目标二进制可执行文件的存放位置
# PROJECT_SOURCE_DIR：工程的根目录
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```

或者只在最外层保留一个 CMakeLists.txt

```
cmake_minimum_required(VERSION 2.6)

project(demo)

aux_source_directory(src SRC_LIST)

include_directories(include)

add_executable(main ${SRC_LIST})

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

```

## 编译动态库和静态库

![](https://img-blog.csdn.net/20180827211429487?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doYWh1MTk4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

外层 CMakeLists.txt

```
cmake_minimum_required (VERSION 2.8)

project (demo)

add_subdirectory (lib_testFunc)
```

内部 CMakeLists.txt

```
aux_source_directory (. SRC_LIST)

# 生成动态库或静态库
# 第一个参数：库的名称
# 第二个参数：库的类型（动态/静态），默认静态
# 第三个参数：生成库的源文件
add_library (testFunc_shared SHARED ${SRC_LIST})
add_library (testFunc_static STATIC ${SRC_LIST})

# 设置输出属性
set_target_properties (testFunc_shared PROPERTIES OUTPUT_NAME "testFunc")
set_target_properties (testFunc_static PROPERTIES OUTPUT_NAME "testFunc")

# LIBRARY_OUTPUT_PATH：库的默认输出路径
set (LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
```

## 链接 pthread

```
find_package (Threads)

target_link_libraries (myapp ${CMAKE_THREAD_LIBS_INIT})
```

## 添加编译选项

```
add_compile_options(-std=c++11 -Wall)

add_definitions(-std=c++11)
```