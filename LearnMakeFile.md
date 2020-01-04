# LearnMakeFile

记录一些MakeFile的用法。

## Makefile Contents

一个Makefile文件包括五个部分：

1. 显示规则
2. 隐式规则
3. 变量定义
4. 指令（include）
5. 注释

### 分割长行

用'\'来连接多行。关于命令行和描述行相关的一大堆没看懂。

## Makefile文件名

GNUmakefile、makefile、Makefile。推荐Makefile。

使用"make -f name"、"make --file=name"来显示指明其他makefile文件名。

## 包含其他makefile文件

include filenames*** 来包含其他makefile文件。filenames可包含多个文件名、makefile变量、makefile函数。

include之前的空格会被忽略，但是不能出现tab。

include末尾可以出现‘#’的注释。

如果没有找到include的文件名，会依次到：

1. make的-I或者--include-dir指定的目录寻找
2. prefix/include (normally /usr/local/include 1) /usr/gnu/include, /usr/local/include, /usr/include.

找不到文件只会报错，并不会终止make的执行。

## MAKEFILES变量

如果定义了MAKEFILES环境变量，make会include环境变量MAKEWFILES指明的文件。
MAKEFILES指明文件中的Default Goal（要编译生成的目标）不会被引入。读MAKEIFLES指明文件失败也不被认做错误。

## 重新生成Makefile

> 放弃了，gg

