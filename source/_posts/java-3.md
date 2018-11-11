---
title: Java漫谈（三）
abbrlink: 4514
date: 2018-11-05 10:29:00
tags: java
categories: 技术
---
#### 问题
&ensp;&ensp;&ensp;&ensp;接上一篇文章中提到的问题：为什么java要求每个.java文件中最多只能有一个public类，并且文件名称还要和这个public类的名字保持一致呢？
&ensp;&ensp;&ensp;&ensp;可以肯定的是，这与java编译器的设计是有关系的。那是不是现在去扒一下javac的源代码？逻辑上讲，这是最彻底的办法。然而时间有限，这里采用取巧的办法——从javac的行为去推测可能的内部机理。

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
<!-- more -->
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

#### -classpath参数

在编译的时候，javac从哪里去寻找需要的class文件呢？在本例中，javac之所以能够找到Demo2的class文件，是因为它“碰巧”放在了当前目录下，即执行javac命令时所在的目录——javac在寻找class文件的时候，**默认是从当前目录开始搜索的**。然而，更一般的，使用javac编译某个源码文件的时候，并非每次依赖类都这么碰巧出现在当前目录下，这时候就需要使用-classpath参数指定所需class文件的路径。

-classpath可以简写为-cp，它的值有几条规则：

1. 可以是绝对路径也可以是javac执行时所在目录的相对路径；
2. 不可以指定具体的class文件名，只能指定到其所在的目录（目录后面的斜线可加可不加）；
3. 可以具体到某个jar包（将一组class文件打包）名称；
4. 可以使用通配符“*”来匹配目录下所有jar包，但是不会匹配到任何具体的class文件（class文件只能使用目录形式）；
5. 使用通配符时，不可以使用“*.jar”的形式，单独一个星号即可。

示例如下：

#### -d参数



#### -sourcepath参数



#### 自动编译
&ensp;&ensp;&ensp;&ensp;那么问题来了：如果类A依赖的类有很多，那就需要人工将这些被依赖的类一个个提前编译出来——很累，而且容易出错，这种活儿显然更适合让机器去做。能不能让javac自动先去把这些依赖类先去编译了呢？可以，***但是要编译依赖类，首先得找到类的源代码放在哪里***。javac只知道依赖的类叫啥，但是存储这个类的源码文件叫啥可就无能为力，毕竟源码文件的名称和类的名称可是没关系的啊。
&ensp;&ensp;&ensp;&ensp;说到这里思路就已经很清晰了，为了能让javac自动编译前置类，需要强制添加一种约束：源码文件名称要和它保存的类的名字保持一致。javac是不是按照这种思路设计的呢？把源码文件的名字改为它保存的类名，编译如下：
```
➜  show ls
Demo1.java  Demo2.java
➜  show javac Demo1.java 
➜  show ls
Demo1.class  Demo1.java  Demo2.class  Demo2.java
```
&ensp;&ensp;&ensp;&ensp;我们发现，虽然没有手动编译类Demo2，但是在编译类Demo1的时候，javac“顺便”自动把Demo2也给编译了。于是证实了我们的推测：javac确实是根据类名去寻找前置类的源码文件的。
&ensp;&ensp;&ensp;&ensp;所以，如果一个类依赖了其他的类，为了使这个类能顺利通过编译，有两个选择：手动把所有的依赖类编译出来；或者让javac自动编译。*如果选择后者，你需要把依赖类保存在以它的名字命名的源码文件里。*
&ensp;&ensp;&ensp;&ensp;回到开篇的问题，终于迎来了一丝曙光，虽然仍然没能彻底回答，但至少找到了方向：以类名来命名源码文件可以提高编译的效率——自动编译依赖类。要彻底回答开篇的问题，还要牵扯权限的问题，下篇继续。