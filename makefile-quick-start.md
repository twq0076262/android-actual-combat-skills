# [Android 实战技巧之三十六：Makefile 快速入门](http://blog.csdn.net/lincyang/article/details/46482079)

<table class="table table-bordered table-striped table-condensed"> <tr> <td>目录(?)[+]</td> </tr> </table>

## 目标

通过一篇文章的介绍达到能够编写简单 Makefile 以及能够看懂普通的 Makefile 之目的。

## make 简介

make 是一个老牌的构建（build）工具，1970年问世以来已经度过了45年的时光而魅力不减，这在技术发展日新月异的今天是不可思议的。make 在大型的软件项目中发挥着巨大作用。我是在学习 Linux kernel 时才第一次接触它，Android 系统也是用 make 和 python 等脚本一起构建系统，所以掌握 make 知识是你迈进这些系统的第一道坎。你一定要给予 make 足够的重视，不要以为掌握了 C/C++、Java 这些主力编程语言就可以掌控世界，当你迷失在海量的 Makefile 以及各种 *.py，*.sh 时，没有构建系统的知识是令人抓狂的。

make 的优点是你可以把程序中各元素之间的关系告诉 make，之后 make 会根据这些关系和时间戳去判断应该重新编译哪些步骤以产生你需要的程序。这就是我们常说的增量编译。

## 一个最简单的 Makefile

通常 make 是根据 Makefile/makefile 执行工作的，Makefile 中包含了一组用来编译程序的规则。一项规则可分为三个部分：目标（target）、必要条件（prereq）以及所要执行的命令（command）。

```
    target: prereq1 prereq2
        commands
```

C 文件只有一个 main.c

```
    #include <stdio.h>

    int main(int argc, char** argv) {
            printf("hello, makefile!\n");
    }
```

**第一个 Makefile**

```
    #first makefile
    hello-makefile: main.c
           gcc -o hello-makefile main.c
```

解读如下： 

这个 Makefile 是由一项任务（规则）组成，目标是一个叫作 hello-makefile 的可执行文件，必要条件是 main.c 文件，而命令是一个 gcc 的编译命令。 

执行结果如下：

```
    $ make
    gcc -o hello-makefile main.c
```

**第二个 Makefile**

```
    #final target
    hello-makefile: main.o
            gcc -o hello-makefile main.o
    #main.o
    main.o: main.c
            gcc -c main.c
```

解读如下： 

将上述步骤分解成两步，为了生成 hello-makefile 需要依赖一个条件是 main.o，而接着将 main.o 作为目标再写一条规则。通常一个 Makefile 就是由这样多个不同的规则组合而成。 

执行结果如下：

```
    $ make
    gcc -c main.c
    gcc -o hello-makefile main.o
```

## Makefile 基本语法

**从上而下的结构**

就像上面的 Makefile 那样，默认从最上层的工作目标（通常称为 all）开始工作，把有些工作目标如 clean 工作放在文件的最底部。 

**特殊符号**
 
井号（#）用来表示注释 

反斜杠（\）当作续行符 

**通配符**
 
与常用 shell 通配符一致。 

星号（*）代表任意数量的任意字符，问号（?）代表一个任意字符。
 
**.PHONY** 

假想工作目标，并且可以避免名字冲突。出现在如下场合：

```
    .PHONY: clean
    clean:
            rm -f *.o
```

常用的 .PHONY如下：
    
```
    all        执行编译应用程序的所有工作
    install    从已编译的二进制文件进行程序的安装
    clean      清楚生成的二进制文件
    distclean  清楚所有生成的文件
    TAGS       建立可供编辑器使用的标记表
    check      执行与程序相关的任何测试
```

**变量**

```
    $(variable)
    ${}
```

变量名称是单一字符的就不用括号了。

**VPATH** 

告诉 make 如果在当前目录没有找到，就去指定的目录找。比如：

```
    VPATH = src include
```

**C++ 标示**

有时需要输入一些参数告诉 gcc 做一些事，比如加入 -I 选项告知启动隐含编译规则： 

```
    CPPFLAGS = -I include
```

## include 关键字

有时需要调用其他的 makefile，只需要用 include 将其加入就可以了：

```
    include /home/linc/workspace/lab/OpenCV-android-sdk-2.4.11/sdk/native/jni/OpenCV.mk
```

## 接下来

读读《GNU Make 项目管理》吧，更深入的知识和经验都可以在书中找到。

