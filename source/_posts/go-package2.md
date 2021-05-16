---
title: Golang基础：源码与包（2）
tags: golang
categories: 技术
abbrlink: 35349
date: 2021-04-15 07:36:44
---

根据用途的不同，Golang源代码分为三类，分别是：命令源码文件、库源码文件和测试源码文件。

#### 命令源码文件与“main”包

命令源码文件是保存程序执行入口代码的文件，所谓“入口代码”就是main函数，类似于C语言或Java，Golang也采用了main函数作为程序执行的起点和入口。

1. 一个源码代码文件能够成为命令源码文件的条件有两个：package声明为“main”且包含“main”函数，代码如下：

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

   因为这个程序只有一个空的main函数，什么都没有做，所以执行之后什么都没发生。

   <!-- more -->

2. 声明为“main”包的一个或多个源文件中，至少要有一个文件包含main函数，否则编译整个包会报错，如下：

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

3. 但是如果源文件包含了main函数却没有声明为main包，那么只会被当作是普通的库源码文件，如下：

   ```shell
   ➜   cat main.go 
   package other
   
   func main() {}
   ➜   go build main.go
   ➜   ls
   main.go
   ```

   对于普通的库源码文件，“go build”只是做语法检查，而不会输出任何结果，所以没有生成可执行文件。

4. 当然，源文件本身不需要命名为“main.go”，也可以叫“cmd.go ”之类的；而最终生成的可执行文件默认与（第一个）源文件名字或目录名字相同，但也可以指定名称。如下：

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

#### 库源码文件

从语法上讲，所有包声明不是“main”的源文件都是库源码文件。库源码文件不包含程序的执行入口代码，它们实现了某些功能，等待被其他模块调用。入口代码经过一系列调用最终会执行到这里。

库源码文件需要被提前编译成二进制文件（简称库文件），这样程序执行的时候才可以直接拿来用，而不是现场编译，否则太没效率了。但是“go build”编译库源码文件不会输出任何结果，这时候就需要一个新的工具——“go install”，它在“go build”的基础上添加了更多功能：编译命令源码文件并安装；编译库源码文件并保存。

“go install”的正常运作依赖于下面两个概念。

1. Golang环境变量

   环境变量是整个操作系统的设置，全局生效，可以使各种命令运行起来更加方便。Golang的环境变量会在Golang安装完成后自动设置一部分，后续也可以根据需求自行调整，其具体值可以使用“go env”命令来查看，如下：

   ```shell
   ➜   go env
   GOARCH="amd64"
   GOOS="darwin"
   GOROOT="/usr/local/opt/go@1.12/libexec"
   GOPATH="/Users/mengshuai/programs/go"
   GOBIN="/Users/mengshuai/programs/go/bin"
   ```

   这里只打印出了比较重要的几个。

   GOARCH代表当前机器的处理器架构，暂时不用关心。

   GOOS代表操作系统类型，我的电脑是mac，所以值为darwin；其他的值包括freebsd、linux和windows。

   GOROOT代表Golang的安装位置，Golang内置的工具命令如“go”都放在这里面，此外Golang的标准库源码和中间编译结果也在这里。

   GOPATH代表Golang的工作目录，Golang编译的时候，会去GOPATH中查询依赖包。工作目录分为两种：一种是全局的，存放所有项目都可能会用的公共包；另一种是项目私有的，存放项目自己会用到的包。上面只打印了全局GOPATH，有新的项目，可以将其绝对路径追加到GOPATH中，用“：”分隔。

   GOBIN代表的是操作系统中任意Golang项目编译后生成的可执行文件的存放位置，有了它，这些可执行文件在执行的时候不必指明其路径。

2. Golang工程标准目录结构

   标准目录结构是为了更好的对工程中的各类元素分工和隔离。Golang推荐的目录结构是：工程根目录下有src，pkg和bin三个字目录，其中src存放源代码；pkg存放编译后的中间文件；bin存放可执行文件。如上面的GOPATH全局目录，其结构如下：

   ```shell
   ➜   ls /Users/mengshuai/programs/go
   bin pkg src
   ```

   虽然这个目录结构知识某种约定而不是强制的，但是有些go命令确实依赖这种约定，目的就是为了执行更有效率，省去了每次都要指定目录的麻烦。有了标准路径，只要将项目根目录加到GOPATH中，编译的时候会自动地：去src目录寻找源码；将编译生成的可执行文件存放到bin下；将编译生成的库文件存放到pkg下。


在之前的例子中，工程根目录没有被加进GOPATH中，但是“go build”依然可以正常运作，这说明“go build”真的只是单纯的编译，是Golang的基本和底层功能。但是，将工程根目录加入到GOPATH之前，“go install”是不会起作用的，如下：

```shell
➜   pwd
/Users/mengshuai/projects/tmp
➜   ls
bin pkg src
➜   go env GOPATH
/Users/mengshuai/programs/go
➜   cat src/other/other.go 
package other

func other() {}
➜   go install src/other/other.go
➜   ls pkg 
➜  
```

工程根目录是“/Users/mengshuai/projects/tmp”，根目录下是标准的Go工程目录结构，但是根目录不在GOPATH中，于是go install之后完全没有任何输出。

把根目录加入到GOPATH后，如下：

```shell
➜   go env GOPATH
/Users/mengshuai/programs/go:/Users/mengshuai/projects/tmp
➜   go install src/other/other.go 
➜   ls pkg 
➜   go install other
➜   ls pkg 
darwin_amd64
➜   ls pkg/darwin_amd64 
other.a
```

可以看到，此时pkg目录下正常生成了所谓的“库文件”，以“.a”结尾。这里需要注意两点：

1. 对于库源码文件，go install的编译单位是包，而不能是单个文件，上述代码已经证明单个文件不会有任何输出
2. 编译后的库文件也没有直接放在pkg目录下，而是放在一个叫“darwin_amd64”的子目录中，子目录的命名其实就是“\$GOOS_$GOARCH”

有了标准目录结构以后，编译工具会自动去GOPATH的src目录下找寻源文件。对于go install而言，除了可以指定包所在目录的绝对路径或者相对路径以外，还可以指定其相对于src目录的路径，编译工具会自动把“GOPATH/src”拼接在前面；但是“go install src/package-name”是不行的。



对于命令源码文件，go install同样会生成可执行文件，同时还会把可执行文件移动到bin目录中，如下：

```shell
➜   cat src/pack/cmd.go 
package main

func main() {}
➜   go install src/pack/cmd.go
go install: no install location for .go files listed on command line (GOBIN not set)
➜   go install pack
➜   ls bin 
pack
```

这里同样有几点需要注意：

1. 命令源码文件所在的目录名pack和声明的包名main不一致，对于go install而言，使用的其实是目录名（这里体现出了包名和目录名一致的好处——不容易混）；

2. go install后面的目录名，既没有绝对路径也没有相对路径的标识，这是因为go install会自动去GOPATH环境变量所代表的路径下的src子目录中查询；可以理解为“go install”自动在参数目录前加上了“GOPATH/src”；

3. go install单独编译某个命令源码文件似乎也行不通，报错信息是“GOBIN not set”，下面对此进行探究

   ```shell
   ➜   export GOBIN="/Users/mengshuai/programs/go/bin"
   ➜   go env GOBIN
   /Users/mengshuai/programs/go/bin
   ➜   rm -f bin/pack 
   ➜   go install src/pack/cmd.go
   ➜   ls bin 
   ➜   ls $GOBIN
   cmd
   ➜   rm -f $GOBIN/cmd 
   ➜   go install pack           
   ➜   ls bin 
   ➜   ls $GOBIN
   pack
   ```

   首先将GOBIN环境变量指向全局GOPATH的bin子目录；然后删除原来的可执行文件；再次执行“go install”，发现可以成功，但是可执行文件没有被移动到工程根目录的bin下，而是直接放在了GOBIN中。

   这说明go install可以安装单独某个命令源码文件，前提是设置了GOBIN环境变量，而且可执行文件会置于GOBIN中；当然main包的文件一般就是集中在一个文件中，如果分散在多个文件的话，也是要以包为单位进行“go install”的；

   另外一点，当设置了GOBIN后，会覆盖工程根目录下的bin目录，可执行文件总会放到GOBIN中；

   最后，可执行文件的名字是以go install的参数命名的，可以通过“-o”参数定制。

#### 测试源码文件

测试源码文件是用来测试以上两种源文件的，似乎应该被划分到库源码文件，因为它们也不是程序执行的入口，“go build”也不会产生任何输出。

虽然在语法层面没有特殊之处，但是有一点细微的区别：测试源码文件的命名一般以"_test.go"结尾，go build命令会忽略这类文件；测试源码文件可以被“go test”命令直接执行，这是Golang做的非常好的一个地方，单元测试直接内置到了原生工具箱里。

