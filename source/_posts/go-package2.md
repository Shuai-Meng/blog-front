---
title: Golang基础：源码与包（2）
tags: golang
categories: 技术
abbrlink: 35349
date: 2021-04-15 07:36:44
---

#### Golang环境变量

环境变量其实是整个操作系统的设置，全局生效。环境变量可以使各种命令运行起来更加方便。Golang的环境变量会在Golang安装完成后自动设置一部分，后续也可以根据需求自行调整。Golang的环境变量可以在命令行使用“go env”查看，如下：

```shell
➜   go env
GOARCH="amd64"
GOOS="darwin"
GOROOT="/usr/local/opt/go@1.12/libexec"
GOPATH="/Users/mengshuai/programs/go"
GOBIN="/Users/mengshuai/programs/go/bin"
```

这里只打印出了比较重要的几个。

<!-- more -->

GOARCH代表当前机器的处理器架构，暂时不用关心；

GOOS代表操作系统类型，我的电脑是mac，所以值为darwin；其他的值包括freebsd、linux和windows；

GOROOT代表Golang的安装位置，Golang内置的工具命令如“go”都放在这里面，还有GOlang的标准库；

GOPATH是Golang的工作目录。它分为两种，一种是全局GOPATH，存放所有项目都可能会用的公共包；另一种是项目对应的GOPATH，存放项目自己会用到的包；

GOBIN代表的是项目编译后生成的可执行文件的存放位置。

#### Golang工程标准目录结构

标准目录结构是为了更好的对工程中的各类元素分工和隔离。Golang推荐的目录结构是：工程根目录下有src，pkg和bin三个字目录，其中src存放源代码；pkg存放编译后的中间文件；bin存放可执行文件。如上面的GOPATH全局目录，其结构如下：

```shell
➜   ls /Users/mengshuai/programs/go
bin pkg src
```

#### Golang源代码的分类

根据用途的不同，Golang源代码分为三类，分别是：命令源码文件、库源码文件和测试源码文件。

1. 命令源码文件与“main”包

   保存程序执行入口代码的文件，所谓“入口代码”就是main函数，类似于C语言或Java，Golang也采用了main函数作为程序执行的起点和入口。

   一个源码代码文件能够成为命令源码文件的条件有两个：package声明为“main”且包含“main”函数，代码如下：

   ```shell
   ➜   cat main.go 
   package main
   
   func main() {}
   ```

   对于命令源码文件，”go build“在编译之后还会进行链接等动作，最终生成一个可执行文件或者命令，运行这个命令或可执行文件，就代表运行写好的程序。代码如下：

   ```sh
   ➜   go build main.go 
   ➜   ls
   main    main.go
   ➜   ./main 
   ➜   
   ```

   因为我们的程序只有一个空的main函数，什么都没有做，所以执行之后什么都没发生。

   

   声明为“main”包的一个或多个源文件中，至少要有一个文件包含main函数，否则编译整个包会报错，如下：

   ```shell
   ➜   cat main.go 
   package main
   
   func test() {}
   ➜   cat other.go 
   package main
   
   func other() {}
   ➜   go build
   # _/Users/mengshuai/projects/tmp
   runtime.main_main·f: function main is undeclared in the main package
   ```

   

   但是如果源文件包含了main函数却没有声明为main包，那么只会被当作是普通的库源码文件，如下：

   ```shell
   ➜   cat main.go 
   package other
   
   func main() {}
   ➜   go build main.go
   ➜   ls
   main.go
   ```

   对于普通的库源码文件，“go build”只是做语法检查，而不会输出任何结果，所以没有生成可执行文件。

   

   当然，源文件本身不需要命名为“main.go”，也可以叫“cmd.go ”之类的；而最终生成的可执行文件默认与（第一个）源文件名字或目录名字相同，但也可以指定名称。如下：

   ```shell
   ➜   cat cmd.go 
   package main
   
   func main() {}
   ➜   cat other.go 
   package main
   
   func other() {}
   ➜   go build cmd.go other.go 
   ➜   ls
   cmd      cmd.go   other.go
   ➜   go build other.go cmd.go 
   ➜   ls
   cmd      cmd.go   other    other.go
   ➜   pwd
   /Users/mengshuai/projects/tmp
   ➜   go build
   ➜   ls
   cmd      cmd.go   other    other.go  tmp
   ➜   go build -o test
   ➜   ls
   cmd      cmd.go   other    other.go test  tmp
   ```

   可以看到，cmd.go和other.go睡在前面，可执行文件的名字就跟谁相同；编译目录的话，就是目录的名字；可以使用“-o”参数定制可执行文件的名称。

2. 库源码文件

   从语法上讲，所有包声明不是“main”的源文件都是库源码文件。库源码文件不包含程序的执行入口代码，它们实现了某些功能，等待被其他模块调用。入口代码经过一系列调用最终会执行到这里。

   如上所说，使用“go build”编译库源码文件不会有任何输出，只做纯语法检查。

   但是，“go install”命令却会输出编译后的中间文件到pkg目录。

3. 测试源码文件

   测试源码文件是用来测试以上两种源文件的，似乎应该被划分到库源码文件，因为它们也不是程序执行的入口。出了功能上的不同，在语法层面没有特殊之处，“go build”也不会产生任何输出。但是有一点细微的区别：测试源码文件可以被“go test”命令直接执行。这是Golang做的非常好的一个地方，单元测试直接内置到了原生工具箱里。

