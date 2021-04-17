---
title: Golang基础：源码与包（1）
abbrlink: 43491
date: 2021-04-11 17:51:11
tags: golang
categories: 技术
---

#### 为什么要引入“包”的概念？

1. 任何一门设计精良的编程语言，其编写出来的程序必然是可以模块化的。模块化的好处有两个：一是分工明确，互相隔离，容易看懂及排查问题；二是方便复用，减少重复建设，提高开发效率。

   “包”就是Golang用来模块化的工具，一个“包”就是一个独立的、可复用的模块，一个Golang程序是由多个互相配合的包组成的。尽管Golang中最小的逻辑单元是函数，但以函数作为模块是不合理的：一个复杂的模块会包含几个不同的功能接口，不可能放在同一个函数里面实现；即使是模块只有一个接口，如果接口功能复杂，只用一个函数实现，可读性也不会好。总而言之，函数无法承载起模块的角色，只有用“包”将许多函数封装到一起，才能起到模块的作用。

   对于只有一个函数的模块，是不是就可以不使用“包”了呢？请看第二点。

2. 不同模块之间的函数可能会重名，但是有了包做前缀，就不会相互影响。“包”在这里起到了命名空间的作用，可以隔离同名函数。就像同一个目录下不能出现名字完全一样的文件，但是不同目录下的文件重名却没有问题。当然，同一个包下面，也不允许出现同名的编程元素，比如变量或函数。

<!-- more -->

#### Golang“包”的语法

“包”的概念反映到Golang语法中就是关键字“package”，用来声明包的名字；而且还规定每个Golang源代码文件的第一行必须是包声明，否则无法通过编译；Golang的源代码文件是以“.go”为命名后缀的文本文件（不是这个后缀则无法被Golang的编译工具识别）。示例如下：

```shell
➜  cat test.go 
package test

func Test() {}
```

语法上，package关键字和包名之间隔了一个空格；包名不可以用双引号包裹；包名建议小写。

#### Golang编译工具之go build

Golang提供了一系列的工具，来保证程序的构建和运行，其中最基本的是编译工具“go build”。“go”是一个shell命令，在Golang安装完成后就可以在命令行使用；“build”是“go”命令的众多功能（或参数）之一，用来编译Golang源代码。

##### 编译单个文件

“go build”最基本的用法就是后面直接跟一个源代码文件，如下：

```shell
➜ ~ go build test.go
```

这是在源代码所在的目录中执行的；当然也可以在任意目录中编译，只要提供源代码文件的绝对路径或相对路径就可以了。类Unix平台下，绝对路径以“/”开头，相对路径以“.”或“..”开头，如果不是三者之一，那么默认在当前目录中寻找匹配文件。

##### 编译多个文件

为了提高效率，可以传入文件列表进行编译，即空格隔开的多个文件。

##### 编译目录

也可以将某个目录传递给“go build”，此时将对整个目录下的源文件进行编译；如果不给“go build”提供任何参数，那么默认将对当前目录进行编译。

#### Golang的包、目录和源代码文件之间的关系

1. 每个源文件必然对应一个包；但是每个包可以对应多个源文件。

   同一个包的源代码，可以集中在同一个文件中，也可以分散在不同的文件中，当然这些文件声明的得是**同一个包名**。

   虽然写代码的时候，会将一个包的代码分散到不同的源文件中；但是在编译的时候，我理解编译器会将同一个声明为同一个包的源文件在内存中进行合并，把这些代码做为一个整体编译。

2. 同一个包的源文件必须位于同一个目录下，否则编译器也会认为它们不属于同一个包。

   ```shell
➜  ~ cat other.go 
   package test

   func other() {}
   ➜  ~ cat test.go 
   package test
   
   func Test() {
   	other()
   }
   ➜  ~ go build
   ➜  ~ mv other.go tmp/ 
   ➜  ~ go build
   ./test.go:4:2: undefined: other
   ➜  ~ go build test.go ./tmp/other.go
   named files must all be in one directory; have ./ and ./tmp/
   ```
   
   test.go调用了other.go中的代码，第一次编译当前目录的时候，它们在同一个目录下，一起编译是没有问题的，同一个包内的代码可以通畅无阻的相互调用。
   
   然后把other.go移动到了子目录tmp中，第二次编译当前目录就出错了，没找到文件，这说明“go build”不会递归的去编译目录，只编译一级目录。

   第三次编译采用文件列表的形式，还是报错，说文件列表中的文件不在同一个目录下无法编译，说明“go build”一次只能编译一个目录内的文件（后面会说明有import的情况）。

   以上足以说明，同一个包的源文件必须位于同一个目录，不然无法完成整体编译；编译一个目录，等同于编译一个包；若文件间有相互引用，比如A引用了B，单独编译A也不行。

3. 同一个目录下所有的源代码文件要属于同一个包，否则以目录为单位进行编译会报错。

   ```shell
   ➜ ~ cat other.go 
   package other
   
   func other() {}
   ➜ ~ cat test.go 
   package test
   
   func Test() {}
   ➜ ~ go build other.go 
   ➜ ~ go build test.go 
   ➜ ~ go build
   can't load package: package .: found packages test (test.go) and other (other.go) in ~
   ```

   other.go和test.go声明了不同的包，以目录为单位进行整体编译就报错了；虽然它们单独编译都是通过的，但是在大型项目中几乎都是以目录为单位进行编译的，单独编译文件只适合练手的玩具项目，或者包的代码集中在一个源文件里（此时也不会有“不同的包”一说，一个文件不能同时声明两个包名）。

4. 目录名字可以和包名不一致，但是强烈建议保持一致。

   验证如下：

   ```shell
   ➜  ~ cat tmp/test.go 
   package test
   
   func Test() {}
   ➜  ~ cat tmp/other.go 
   package test
   
   func other() {}
   ➜  ~ go build ./tmp  
   ➜  ~ 
   ```

​       other.go和test.go的包声明都是“test”，但是所在目录是“tmp”，但是还是编译通过了。这说明包名可以和目录不一致，后面说到“import”关键字的时候会说吗为什么建议保持一致。

#### 多说一点

再看下另一种形式的整体——文件列表：

```shell
➜ ~ cat other.go 
package other

func other() {}
➜ ~ cat test.go 
package test

func Test() {}
➜ ~ go build test.go other.go 
can't load package: package main: found packages test (test.go) and other (other.go) in ~
➜ ~ go build other.go test.go 
can't load package: package main: found packages other (other.go) and test (test.go) in ~
```

编译结果说明：

首先，文件列表中文件的顺序无关紧要；

其次，文件列表内的所有文件也必须声明为同一个包，否则编译不通过；

再次，报错信息有点区别，这里的package说的是main，上面直接编译目录是“.”（代指当前目录），说明文件列表默认的包名是“main”。















