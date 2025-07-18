# CMake

## 1. CMake简介
跨平台构建工具，通过`CMakeLists.txt`管理编译流程，支持多编译器和依赖管理。

### 特点
- 生成标准构建文件（Makefile/MSVC项目等）
- 支持C/C++/Python等多种语言
- 集成测试/安装/打包功能

---

## 2. 基本语法
### 核心命令
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyApp LANGUAGES CXX)

add_executable(app main.cpp)
add_library(mylib STATIC lib.cpp)

target_include_directories(mylib PUBLIC include)
target_link_libraries(app PRIVATE mylib)
```

### 命令速查表
|                            命令                            |          作用          |
| --------------------------------------------------------- | ---------------------- |
| `include_directories(${PROJECT_SOURCE_DIR}/include)`      | 增加头文件目录          |
| `aux_source_directory(<dir_path> <src_sets>)`             | 收集指定目录源文件       |
| `file(GLOB_RECURSE <src_sets> "src/*.cpp")`               | 递归收集源文件          |
| `set(<variable_name> <variable_value>)`                   | 设置变量，默认String    |
| `add_library(<lib_name> SHARED/STATIC <src_path>)`        | 生成动态或静态库         |
| `set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/Build)` | 设置生成可执行文件的路径 |
| `set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/Lib)`      | 设置生成库的路径         |
| `link_libraries(<lib> ...)`                               | 链接静态库              |
| `link_directories(<path> ...)`                            | 静态库的路径            |




---

## 3. 常用变量
### 内置变量
```cmake
${CMAKE_BINARY_DIR}    # 构建目录
${CMAKE_SOURCE_DIR}    # 源码根目录
${PROJECT_NAME}        # 项目名称
${CMAKE_CURRENT_LIST_DIR}  # 当前CMake文件所在目录
```

### 自定义变量
```cmake
set(MY_FLAGS "-Wall -O2")          # 定义编译选项
option(BUILD_TESTS "Enable tests" ON)  # 创建选项
```

---

## 4. 流程控制
### 条件分支
```cmake
if(MSVC)
  set(COMPILER_FLAGS "/W4")
elseif(UNIX)
  set(COMPILER_FLAGS "-Wall -pedantic")
endif()
```

### 循环操作
```cmake
foreach(src IN LISTS SOURCE_FILES)
  message("Processing: ${src}")
endforeach()
```

---

## 5. 模块管理
### 查找依赖
```cmake
find_package(Boost 1.70 REQUIRED COMPONENTS filesystem)
target_link_libraries(app PRIVATE Boost::filesystem)
```

### 第三方项目集成
```cmake
include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/v1.13.0.zip
)
FetchContent_MakeAvailable(googletest)
```

---

## 6. 安装与打包
### 安装配置
```cmake
install(TARGETS app
        RUNTIME DESTINATION bin
        ARCHIVE DESTINATION lib)

install(DIRECTORY docs/ DESTINATION share/doc)
```

### CPack打包
```cmake
set(CPACK_PACKAGE_VENDOR "MyCompany")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libboost-dev")
include(CPack)
```

---

## 7. 示例项目
### 目录结构
```
project/
├── CMakeLists.txt
├── include/
├── src/
└── tests/
```

### 完整配置
```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject VERSION 1.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_subdirectory(src)
add_subdirectory(tests)  # 包含测试目录

if(BUILD_TESTS)
  enable_testing()
  add_test(NAME tests COMMAND test_runner)
endif()
```

---

## 8. 高级技巧
### 交叉编译配置
```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)
```

### 预编译头文件
```cmake
target_precompile_headers(mylib PUBLIC include/common.h)
```

---

## 9. 参考资料
> [官方文档](https://cmake.org/cmake/help/latest/)
> [CMake应用：CMakeLists.txt完全指南](https://zhuanlan.zhihu.com/p/371257515)