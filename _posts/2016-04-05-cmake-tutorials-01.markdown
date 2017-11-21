---
layout: post
title: cmake 学习笔记（一）
category: Document
tags: cmake c++
date: 2016-04-05 15:04:57
published: true
summary: cmake 和 autotools 都得学啊，有人说 c 没落了？呵呵……
mathjax: false
highlight: true
---

cmake 编译 C 和 C++ 工程，只需要编写一个 CMakeLists.txt 文件即可，过程很简单，所有的工作只要编写 CMakeLists.txt 文件上。本例列举最简单的一个可执行文件的编译过程。


```cpp
/**
 * @File Name: main.cc
 * @Description:
 *      最基础的往往是个可执行的 exe 工程，其 cmake 的编写参见 CMakeLists.txt 文件
 */

#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char* argv[]) {
    if (argc < 2) {
        fprintf(stdout, "Usage: %s number\n", argv[0]);
        return 1;
    }

    double inputValue = atof(argv[1]);
    double outputValue = sqrt(inputValue);
    fprintf(stdout, "%g 的平方根是 %g\n", inputValue, outputValue);
    return 0;
}
```

编写 CMakeLists.txt 如下：

```makefile
cmake_minimum_required (VERSION 2.6)
project (tutorial)
add_executable(tutorial main.cc)
```

把这两个文件放在同一目录下，然后：

```bash
$ cmake .
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joshua/Documents/cmake_tutorials
$ make
Scanning dependencies of target tutorial
[100%] Building CXX object CMakeFiles/tutorial.dir/main.cc.o
Linking CXX executable tutorial
[100%] Built target tutorial
$ ./tutorial 4.8
4.8 的平方根是 2.19089
```

好了，整个过程完成！接下来

## Step 1：添加版本号和配置头文件

版本号添加在 CMakeFiles 中可以灵活修改，当然你也可以在代码中添加。

```makefile
cmake_minimum_required(VERSION 2.6)
project(Tutorial)

# 版本号
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# 配置一个头文件来传递 CMake 设置
configure_file (
    "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
    "${PROJECT_BINARY_DIR}/TutorialConfig.h"
    )

# 添加二进制树到搜索路径以搜索代码文件
include_directories("${PROJECT_BINARY_DIR}")

add_executable(tutorial main.cc)
```

配置文件将会写进二进制树中，所以我们得把那个目录添加到搜索路径上去。然后创建 TutorialConfig.h.in 文件并写入如下内容：

```makefile
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当 cmake 配置这个头文件时 `@Tutorial_VERSION_MAJOR@` 和 `@Tutorial_VERSION_MINOR@` 的值将会被 CMakeLists 中的值替代。接下来可以修改 main.cc 来验证：


```cpp
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "config.h"

int main(int argc, char* argv[]) {
    if (argc < 2) {
        fprintf(stdout, "%s 版本号 %d.%d\n",
                argv[0],
                Tutorial_VERSION_MAJOR,
                Tutorial_VERSION_MINOR);
        fprintf(stdout, "用法： %s 数字\n", argv[0]);
        return 1;
    }

    double inputValue = atof(argv[1]);
    double outputValue = sqrt(inputValue);
    fprintf(stdout, "%g 的平方根是 %g\n", inputValue, outputValue);
    return 0;
}
```

## Step 2：添加 lib

假如要添加一个自定义的求平方根的 lib libmymath.a，其文件目录如下：

```bash
|-- mymath
|   |-- CMakeLists.txt
|   `-- mymath.h
|-- mysqrt.cc
|-- main.cc
|-- CMakeLists.txt 
```

首先，需要编译出 lib 文件，在 CMakeLists.txt 中添加：

```makefile
add_library(mymath mysqrt.cc)
add_subdirectory(mymath)
```

其次需要在 main.cc 中引用 mysqrt 需要把 mymath.h 引入编译搜索路径

```makefile
include_directories("${PROJECT_SOURCE_DIR}/mymath")
```

最后，链接 lib 文件到 可执行文件

```makefile
target_link_libraries(tutorial mymath)
```

OK, lib 添加完毕并能正常运行。但我想在编译时选择使用自定义的数学函数或者仍然使用库里的函数怎么办呢？可以使用 option 定义编译宏

```makefile
# 宏 ON 表示开启
option (USE_MYMATH "使用自定义数学函数" ON)

if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/mymath")
    add_subdirectory(mymath)
    set (EXTRA_LIBS ${EXTRA_LIBS} mymath}
endif (USE_MYMATH)

add_executable(tutorial main.cc)
target_link_libraries(tutorial, ${EXTRA_LIBS})
```

然后在 config.h.in 中定义这个宏

```makefile
#cmakedefine USE_MYMATH
```

然后在 main.cc 中就可以使用这个宏了

```cpp
#include <stdio.h>
#include <stdlib.h>
#include "config.h"
#ifdef USE_MYMATH
#include "mymath.h"
#else
#include <math.h>
#endif

int main(int argc, char* argv[]) {
    if (argc < 2) {
        fprintf(stdout, "%s 版本 %d.%d\n", argv[0],
                Tutorial_VERSION_MAJOR,
                Tutorial_VERSION_MINOR);
        fprintf(stdout, "用法：%s 数\n", argv[0]);
        return 1;
    }
    double inValue = atof(argv[1]);
#ifdef USE_MYMATH
    double outValue = mysqrt(inValue);
#else
    double outValue = sqrt(inValue);
#endif
    fprintf(stdout, "%g 的平方根是 %g\n", inValue, outValue);
    return 0;
}
```

## Step 3: 安装和测试

接下来，将进行安装规则和测试支持的配置，安装规则很直接，对于自定义数学库的安装规则，可以在 CMakeLists.txt 这么写：

```makefile
install (TARGETS mymath DESTINATION bin)
install (FILES mymath.h DESTINATION include)
```

对于本项目的可执行文件的安装，需要这么写：


```makefile
install (TARGETS tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h" DESTINATION include)
```

这就是安装所需要做的，编译后直接 `make install` 就安装了，普通用户默认安装在 `/usr/local/bin` 下，修改 `CMAKE_INSTALL_PREFIX` 可修改安装目录了，安装到此为止。

**测试**

CMake 内置了一个测试工具叫 CTest，可以在 Top level 的 CMakeLists 中引入它，并进行简单测试：

```makefile
# 引入 CTest
include(CTest)

# 能否正常执行
add_test(TutorialRuns tutorial 25)

# 求 25 的平方根
add_test(TutorialComp25 tutorial 25)
set_tests_properties(TutorialComp25
    PROPERTIES PASS_REGULAR_EXPRESSION "25 的平方根是 5")

# 求 -25 的平方根
add_test(TutorialNegative tutorial -25)
set_tests_properties(TutorialNegative
    PROPERTIES PASS_REGULAR_EXPRESSION "-25 的平方根是 0")


# 求小数的平方根
add_test(TutorialSmall tutorial 0.0001)
set_tests_properties(TutorialSmall
    PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 的平方根是 0.01")

# 帮助是否正常
add_test(TutorialUsage tutorial)
set_tests_properties(TutorialUsage
    PROPERTIES PASS_REGULAR_EXPRESSION "用法：.*数")
```

编译后，直接运行 `ctest` 或者 `make test` 就可以进行测试了，`add_test` 添加测试用例，`set_test` 设置测试成功条件，如果有很多相同的测试用例，可以使用宏来定义测试函数：

```makefile
# 定义测试宏
macro(do_test arg result)
    add_test(TutorialComp${arg} tutorial ${arg})
    set_tests_properties(TutorialComp${arg}
        PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro(do_test)

do_test(25 "25 的平方根是 5")
do_test(-25 "-25 的平方根是 0")
do_test(100 "100 的平方根是 10")
do_test(0.0001 "0.0001 的平方根是 0.01")
```

## Step 4：添加系统自检（System Introspection）

许多时候应用程序需要检查系统是否满足自己的运行条件，这时就需要用到系统自检。比如应用程序用到了 `log` 和 `exp` 函数，CMake 使用 CheckFunctionExists.cmake 宏来检查函数可见性：

```makefile
# 引入函数检查
include(CheckFunctionExists)
check_function_exists(log HAVE_LOG)
check_function_exists(exp HAVE_EXP)
```

然后修改 config.h.in 来定义宏如果 CMake 在系统上找到的话：

```makefile
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```

很重要的一点是，检测函数必须得在 configure_file 命令之前完成，否则就不起作用了，然后就可以在代码中使用这两个宏了：

```cpp
#if defined(HAVE_LOG) && defined(HAVE_EXP)
    result = exp(log(x) * 0.5);
#endif
```


## Step 5：添加中间生成文件和生成器

本章将展示如何添加文件生成器代码并把生成的头文件配置在工程中，加入我们需要0～9的开方表，那么新建一个名为 maketable.cc 的文件：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

int main(int argc, char* argv[]) {
    // 确保有文件名传入
    if (argc < 2)
        return 1;

    // 创建文件
    FILE* fo = fopen(argv[1], "w");
    if (!fo)
        return 1;
    fprintf(fo, "double sqrtTable[] = \n");
    for (int i = 0; i < 10; ++i) {
        double result = sqrt(static_cast<double>(i));
        fprintf(fo, "%g,\n", result);
    }

    // 关闭文件
    fprintf(fo, "0};\n");
    fclose(fo);
    return 0;
}
```

可以看出，这个生成器产生了一段合法的 C++ 代码并根据输入参数保存成了一个文件，接下来将在 mymath 目录下的 CMakeLists.txt 中使用合适的命令来编译 maketable 并执行它来产生目标文件，命令如下：

```makefile
# 定义添加可执行文件
add_executable(maketable maketable.cc)

# 产生源代码的命令行
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/table.h
    COMMAND maketable ${CMAKE_CURRENT_BINARY_DIR}/table.h
    DEPENDS maketable
    )

# 添加至搜索路径
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# 添加至 main library
add_library(mymath mysqrt.cc ${CMAKE_CURRENT_BINARY_DIR}/table.h)

# 安装
install(TARGETS mymath DESTINATION bin)
install(FILES mymath.h DESTINATION include)
```

首先添加 maketable 为可执行工程，然后添加了自定义命令来指定如何使用 maketable 生成 table.h，接下来让 CMake 知道 mysqrt.cc 依赖于生成的 table.h，把table.h 添加到 mymath 库的 source 下就可以了。同样，也需要把当前的二进制目录添加到 include 目录下去，这样 table.h 才能被 mysqrt.cc 看到并引用。

这样，当工程被编译时将先编译 maketable，然后产生 table.h，然后编译 mymath 库，然后编译 tutorial。

这时，root 下的 CMakeLists.txt 调整如下：

```makefile
# 指定 cmake 最低版本
cmake_minimum_required(VERSION 2.6)

# 指定工程名称
project(tutorial)

# 引入 CTest
include(CTest)

# 版本号
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# 引入函数检查
include(${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
check_function_exists(log HAVE_LOG)
check_function_exists(exp HAVE_EXP)

# 配置一个头文件来传递 CMake 的设置到源代码中
configure_file (
    "${PROJECT_SOURCE_DIR}/config.h.in"
    "${PROJECT_BINARY_DIR}/config.h"
)

# 把二进制目录树添加到引入路径的搜索路径上，
# 这样源代码中就可以找到上面配置的 config.h 了
include_directories("${PROJECT_BINARY_DIR}")

# 定义宏
option (USE_MYMATH "使用自定义数学函数" ON)

#
# 添加自定义 lib
if (USE_MYMATH)
    # 添加头文件（被源码引用）
    include_directories("${PROJECT_SOURCE_DIR}/mymath")
    # 添加子目录（好让 lib 被编译）
    add_subdirectory(mymath)
    set (EXTRA_LIBS ${EXTRA_LIBS} mymath)
endif (USE_MYMATH)


# 指定工程输出为可执行文件 tutotial
add_executable(tutorial main.cc)
target_link_libraries(tutorial ${EXTRA_LIBS})

#
# 安装配置
install (TARGETS tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h" DESTINATION include)

#
# 测试

# 能否正常执行
add_test(TutorialRuns tutorial 25)

# 定义测试宏
macro(do_test arg result)
    add_test(TutorialComp${arg} tutorial ${arg})
    set_tests_properties(TutorialComp${arg}
        PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro(do_test)

do_test(25 "25 的平方根是 5")
do_test(-25 "-25 的平方根是 0")
do_test(100 "100 的平方根是 10")
do_test(0.0001 "0.0001 的平方根是 0.01")

# 帮助是否正常
add_test(TutorialUsage tutorial)
set_tests_properties(TutorialUsage
    PROPERTIES PASS_REGULAR_EXPRESSION "用法：.*数")
```

## Step 6： 创建安装包

使用 cmake 创建安装包是如此的简单， CMake 使用 CPack 制作安装包，支持二进制安装和安装包管理工具（cygwin，debian，RPMs等等）的安装，只需要在顶层目录下的 CMakeLists 中写入如下命令即可：

```makefile
# 创建由 CPack 驱动的安装包
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/license.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```

这就完事了，正常执行 cmake 命令，会发现多个几个文件：`CPackConfig.cmake` 和 `CPackSourceConfig.cmake`，然后使用 CPack 命令就可以做包了：


制作二进制包：

```bash
$ cpack --config CPackConfig.cmake
```

制作源发布包：

```bash
$ cpack --config CPackSourceConfig.cmake
```

恭喜！
