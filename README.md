CMake-Demo
=====

### 单个源文件.编译项目

当前目录执行`cmake .`，得到Makefile后再使用`make`命令编译得到Demo1可执行文件

即可根据代码情况进行输入，**注意输入路径**

```
daina@daina-KLV-WX9:~/实验室/cmake-demo/Demo2$ ./Demo 2 7
2 ^ 7 is 128
```

### 多个源文件

#### 同一目录

```cmake
# CMake 最低版本号要求

cmake_minimum_required (VERSION 2.8)

# 项目信息

project (Demo2)

# 指定生成目标

add_executable(Demo main.cc MathFunctions.cc)
```

唯一的改动只是在 `add_executable` 命令中增加了一个 `MathFunctions.cc` 源文件。这样写当然没什么问题，但是如果源文件很多，把所有源文件的名字都加进去将是一件烦人的工作。更省事的方法是使用 `aux_source_directory` 命令，该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名。其语法如下：

```cmake
aux_source_directory(<dir> <variable>)
```

因此，可以修改 CMakeLists.txt 如下：

 ```cmake
# CMake 最低版本号要求

cmake_minimum_required (VERSION 2.8)

# 项目信息

project (Demo2)

# 查找当前目录下的所有源文件

# 并将名称保存到 DIR_SRCS 变量

aux_source_directory(. DIR_SRCS)

# 指定生成目标

add_executable(Demo ${DIR_SRCS})
 ```

这样，CMake 会将当前目录所有源文件的文件名赋值给变量 `DIR_SRCS` ，再指示变量 `DIR_SRCS` 中的源文件需要编译成一个名称为 Demo 的可执行文件。

#### 多个目录

进一步将 MathFunctions.h 和 [MathFunctions.cc](http://mathfunctions.cc/) 文件移动到 math 目录下

```
./Demo3
|
+--- main.cc
|
+--- math/
|
+--- MathFunctions.cc
|
+--- MathFunctions.h
```

对于这种情况，需要分别在项目根目录 Demo3 和 math 目录里各编写一个 CMakeLists.txt 文件。为了方便，可以先将 math 目录里的文件编译成静态库再由 main 函数调用

**根目录中的 CMakeLists.txt ：**

```cmake
# CMake最低版本号要求
cmake_minimum_required( VERSION 2.8 )
# 项目信息
project( Demo3 )
# 查找当前目录下的所有源文件
# 并将名称保存到DIR_SRCS变量
aux_source_directory( .DIR_SRCS )
# 添加math目录 , 指明本项目包含一个子目录 math , 这样 math 目录下的 CMakeLists.txt 文件和源代码也会被处理 
add_subdirectory( math )
# 指定生成目标
add_executable( Demo main.cc )
# 添加链接库 , 指明可执行文件 main 需要连接一个名为 MathFunctions 的链接库
target_link_libraries( Demo MathFunctions )
```

**子目录中的 CMakeLists.txt：**

```cmake
# 查找当前目录下的所有源文件
# 并将名称保存到DIR_LIB_SRCS变量
aux_source_directory( .DIR_LIB_SRCS )
# 生成链接  使用命令 add_library 将 src 目录中的源文件编译为静态链接库
add_library( MathFunctions ${DIR_LIB_SRCS} )
```

### 自定义编译选项

CMake 允许为项目增加编译选项，从而可以根据用户的环境和需求选择最合适的编译方案。

例如，可以将` MathFunctions` 库设为一个可选的库，如果该选项为 `ON` ，就使用该库定义的数学函数来进行运算。否则就调用标准库中的数学函数库。

**修改 CMakeLists 文件**

我们要做的第一步是在顶层的 CMakeLists.txt 文件中添加该选项：

```cmake
# CMake 最低版本号要求
cmake_minimum_required( VERSION 2.8 )
# 项目信息
project(Demo4)
# 加入一个配置头文件，用于处理CMake对源码的设置
#  configure_file 命令用于加入一个配置头文件 config.h ，这个文件由 CMake 从 config.h.in 生成，通过这样的机制，将可以通过预定义一些参数和变量来控制代码的生成
configure_file(
"${PROJECT_SOURCE_DIR}/config.h.in"
"${PROJECT_BINARY_DIR}/config.h"
)
# 是否使用自己的MathFunctions库  option 命令添加了一个 USE_MYMATH 选项，并且默认值为 ON 
option( USE_MYMATH "Use provided math implementation" ON )
# 是否加入MathFunctions库, 根据 USE_MYMATH 变量的值来决定是否使用我们自己编写的 MathFunctions 库
if( USE_MYMATH )
include_directories("${PROJECT_SOURCE_DIR}/math")
add_subdirectory(math)
set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif(USE_MYMATH)
# 查找当前目录下的所有源文件
# 并将名称保存到DIR_SRCS变量
aux_source_directory(.DIR_SRCS)
# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries(Demo ${EXTRA_LIBS})
```

**修改 [main.cc](http://main.cc/) 文件**

之后修改 [main.cc](http://main.cc/) 文件，让其根据 `USE_MYMATH` 的预定义值来决定是否调用标准库还是 MathFunctions 库：

```c
#include <stdio.h>
#include <stdlib.h>
#include <config.h>		// 这里引用了一个 config.h 文件，这个文件预定义了 USE_MYMATH 的值
						// 但并不直接编写这个文件
						// 为了方便从 CMakeLists.txt 中导入配置 , 编写一个 config.h.in 文件

#ifdef USE_MYMATH
  #include <MathFunctions.h>
#else
  #include <math.h>
#endif


int main(int argc, char *argv[])
{
    if (argc < 3){
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }

    double base = atof(argv[1]);
    int exponent = atoi(argv[2]);

#ifdef USE_MYMATH
    printf("Now we use our own Math library. \n");
    double result = power(base, exponent);
#else
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#endif
    
    printf("%g ^ %d is %g\n", base, exponent, result);
    return 0;
}
```

**编写 [config.h.in](http://config.h.in/) 文件**

```cmake
#cmakedefine USE_MYMATH
```

这样 CMake 会自动根据 CMakeLists 配置文件中的设置自动生成 config.h 文件