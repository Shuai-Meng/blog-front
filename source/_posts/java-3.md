---
title: Java漫谈（三）
abbrlink: 4514
date: 2018-11-05 10:29:00
tags: java
categories: 技术
---
#### 问题
&ensp;&ensp;&ensp;&ensp;接上一篇文章中提到的问题：为什么java要求每个.java文件中最多只能有一个public类，并且文件名称还要和这个public类的名字保持一致呢？这个问题其实可以分为三个子问题：
&ensp;&ensp;&ensp;&ensp;1. 为什么要以类名来命名.java文件？
&ensp;&ensp;&ensp;&ensp;2. 为什么要以public类来命名.java文件？
&ensp;&ensp;&ensp;&ensp;3. 为什么一个.java文件只能存在一个public类？
&ensp;&ensp;&ensp;&ensp;第三个问题略显多余，因为一旦确认必须要以public类名来命名一个.java文件，那么显然这个文件只能存在一个public类，不然以哪个类来命名呢？可以肯定的是，这与java编译器的设计是有关系的。那是不是现在去扒一下javac的源代码？逻辑上讲，这是最彻底的办法。然而时间有限，这里采用取巧的办法——从javac的行为去推测可能的内部机理。
<!-- more -->
#### 编译依赖
先看个例子：
```
class Demo1 {
    public static void main(String[] args) {
        System.out.println(Demo2.a);
    }
}

class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
&ensp;&ensp;&ensp;&ensp;在类Demo1中访问了类Demo2的属性a，两个类分别存储在文件test1.java、test2.java中编译：
```
➜  show ls
test1.java  test2.java
➜  show javac test1.java 
test1.java:3: 错误: 找不到符号
        System.out.println(Demo2.a);
                           ^
  符号:   类 Demo2
  位置: 类 Demo1
1 个错误
```
&ensp;&ensp;&ensp;&ensp;编译器报错了，说找不到类Demo2，但Demo2的源码文件就在同一目录下，这说明编译器需要的是Demo2的class文件而非源码文件。猜测需要先把Demo2编译出来，Demo1的编译才能通过，验证如下：
```
➜  show ls
test1.java  test2.java
➜  show javac test2.java 
➜  show ls
Demo2.class  test1.java  test2.java
➜  show javac test1.java 
➜  show ls
Demo1.class  Demo2.class  test1.java  test2.java
```
&ensp;&ensp;&ensp;&ensp;果然如此，现在Demo1的源码可以正常编译出来。这说明了一件事：若类A依赖于类B，即类A的代码会访问到类B的代码，那么在编译类A的时候会需要类B的class文件，从而类B必须先编译出来。
#### 自动编译
&ensp;&ensp;&ensp;&ensp;如果类A依赖的类有很多，那就需要人工将这些被依赖的类一个个提前编译出来——很累，而且容易出错，这种活儿显然更适合让机器去做。能不能让javac自动先去把这些依赖类先去编译了呢？可以，***但是要编译依赖类，首先得找到类的源代码放在哪里***。javac虽然知道依赖的类名称，然而源码文件的名称可能跟类的名称不一致，所以无法推断这个类的源码文件的名称。
&ensp;&ensp;&ensp;&ensp;说到这里思路就已经很清晰了，为了能让javac自动编译依赖类，需要强制添加一种约束：源码文件名称要和它保存的类的名字保持一致。javac是不是按照这种思路设计的呢？把源码文件的名字改为它保存的类名，编译如下：
```
➜  show ls
Demo1.java  Demo2.java
➜  show javac Demo1.java 
➜  show ls
Demo1.class  Demo1.java  Demo2.class  Demo2.java
```
&ensp;&ensp;&ensp;&ensp;虽然没有手动编译类Demo2，但是在编译类Demo1的时候，javac“顺便”自动把Demo2也给编译了。于是证实了我的推测：javac确实是根据类名去寻找依赖类的源码文件的。
&ensp;&ensp;&ensp;&ensp;所以，如果一个类依赖了其他的类，为了使这个类能顺利通过编译，有两个选择：手动把所有的依赖类编译出来；或者让javac自动编译。*如果选择后者，需要把依赖类保存在以它的名字命名的源码文件里。*
&ensp;&ensp;&ensp;&ensp;回到开篇的问题，第一个子问题有了答案：以类名来命名源码文件可以提高编译的效率——自动编译依赖类。要回答第二个子问题，还要牵扯权限，放在下篇。
#### 重要参数
&ensp;&ensp;&ensp;&ensp;前面的例子说明了javac的编译大致过程：若不存在依赖类，直接编译；否则，寻找依赖类的class文件；若没有找到class文件，根据类名去寻找源代码文件自动编译出class文件。
&ensp;&ensp;&ensp;&ensp;那么问题来了：javac去哪里寻找class文件？又去哪里寻找源代码文件？javac自动编译出的class文件存放在哪里？上例中，所有的编译步骤都是在同一个目录下完成 的，这是一般还是特殊的情况？
##### -classpath参数
&ensp;&ensp;&ensp;&ensp;在上例中，javac之所以能够找到Demo2的class文件，是因为它“碰巧”放在了当前目录下，即执行javac命令时所在的目录——javac在寻找class文件的时候，**默认是从当前目录开始搜索的**。然而，更一般的，javac编译的时候，并非每次依赖类都这么碰巧出现在当前目录下，这时候就需要使用-classpath参数指定所需class文件的路径。
&ensp;&ensp;&ensp;&ensp;-classpath可以简写为-cp，它的使用有几条规则：
&ensp;&ensp;&ensp;&ensp;1. 若没有使用这个参数，默认只搜索当前目录；否则，以指定路径为准（覆盖默认值）；
&ensp;&ensp;&ensp;&ensp;2. 可以是绝对路径也可以是javac执行时所在目录的相对路径；
&ensp;&ensp;&ensp;&ensp;3. 可以指定多个路径，使用冒号分隔（针对Linux系统）；
&ensp;&ensp;&ensp;&ensp;4. 存在多个路径时，按照**从后往前**的顺序搜索，找到为止；
&ensp;&ensp;&ensp;&ensp;5. 不可以指定具体的class文件名，只能指定到其所在的目录（目录后面的斜线可加可不加）；
&ensp;&ensp;&ensp;&ensp;6. javac不会递归搜索指定目录的子目录
&ensp;&ensp;&ensp;&ensp;7. 可以具体到某个jar包（将一组class文件打包）名称；
&ensp;&ensp;&ensp;&ensp;8. 可以使用通配符“\*”来匹配目录下所有jar包（不可以使用“\*.jar”的形式，单独一个星号即可），但是不能匹配任何具体的class文件。这意味着class文件只能使用目录；
```
➜  show tree
.
├── Demo1.java
└── test
    ├── Demo2.class
    └──Demo2.java

1 directory, 3 files
➜  show javac Demo1.java 
Demo1.java:3: 错误: 找不到符号
        System.out.println(Demo2.a);
                           ^
  符号:   变量 Demo2
  位置: 类 Demo1
1 个错误
➜  show javac -cp test Demo1.java
➜  show ls 
Demo1.class  Demo1.java  test
```
&ensp;&ensp;&ensp;&ensp;将Demo2.class转移到子目录test后，编译test1.java报错；指定classpath后，重新编译成功。
##### -d参数
&ensp;&ensp;&ensp;&ensp;上例中，test子目录里的Demo2.class是手工转移过去的。实际上，javac提供了-d参数，用来指定编译好的class文件的存放目录，前提是这个目录已经存在了，否则会报错——javac不会主动去建立指定目录。
```
➜  show ls
Demo1.java  Demo2.java
➜  show javac Demo2.java -d test
javac: 找不到目录: test
用法: javac <options> <source files>
-help 用于列出可能的选项
➜  show mkdir test
➜  show javac Demo2.java -d test
➜  show tree
.
├── Demo1.java
├── Demo2.java
└── test
    └── Demo2.class

1 directory, 3 files
```
使用-d参数可以使源码文件和class文件分开保存在不同的文件夹，方便管理，避免混乱。
##### -sourcepath参数
&ensp;&ensp;&ensp;&ensp;同上，javac之所以能够找到Demo2的源代码文件，是因为它“碰巧”放在了当前目录下，而当前目录也的确是javac默认搜索源代码的目录。当源代码不在当前目录的时候，需要使用 -sourcepath参数指定源码文件的路径。需要注意的是，这个参数指定的是依赖类的源码文件路径，对于将要被编译的目标类，仍然要直接写明其路径，告诉javac到底要编译的是哪个文件。
&ensp;&ensp;&ensp;&ensp;如下，show目录下新建src和target子目录，分别用来存放源代码和class文件。然后在show目录下直接编译Demo1.java：
```
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target

2 directories, 2 files
➜  show javac -sourcepath src -d target -cp target src/Demo1.java 
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target
    ├── Demo1.class
    └── Demo2.class

2 directories, 4 files
```
&ensp;&ensp;&ensp;&ensp;有了这三个参数，javac命令就可以在任何目录下执行，不再局限于被编译文件所在的目录。