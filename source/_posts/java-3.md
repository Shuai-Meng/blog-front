---
title: Java漫谈（三）
abbrlink: 4514
date: 2018-11-05 10:29:00
tags: java
categories: 技术
---

接上一篇文章中提到的问题：为什么java要求每个.java文件中最多只能有一个public类，并且文件名称还要和这个public类的名字保持一致呢？
可以肯定的是，这与java编译器的设计是有关系的。那是不是现在去扒一下javac的源代码？逻辑上讲，这是最彻底的办法。然而时间有限，这里采用取巧的办法——从javac的行为去推测可能的内部机理。先看个例子：
<!-- more -->
```
class demo1 {
    public static void main(String[] args) {
        System.out.println(demo2.a);
    }
}

class demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
在类demo1中访问了类demo2的属性a，两个类分别存储在文件test1.java、test2.java中。
```
➜  show ls
test1.java  test2.java
➜  show javac test1.java 
test1.java:3: 错误: 找不到符号
        System.out.println(demo2.a);
                           ^
  符号:   类 demo2
  位置: 类 demo1
1 个错误
```
编译器报错了，说找不到类demo2，但demo2的源码文件就在同一目录下，这说明编译器需要的是demo2的class文件而非源码文件。
```
➜  show ls
test1.java  test2.java
➜  show javac test2.java 
➜  show ls
demo2.class  test1.java  test2.java
➜  show javac test1.java 
➜  show ls
demo1.class  demo2.class  test1.java  test2.java
```
果然如此，现在demo1的源码可以正常编译出来。这说明了一件事：若类A依赖于类B，即类A的代码会访问到类B的代码，那么在编译类A的时候，类B必须先编译出来。那么问题来了：如果类A依赖的类有很多，那就需要人工将这些被依赖的类一个个提前编译出来。很累，而且容易出错，这种活儿显然更适合让机器去做。所以，如果在编译类A的时候发现A有依赖类，能不能让javac自动先去把这些依赖类先去编译了呢？可以，但是要编译依赖类，首先得找到类的源代码放在哪里。javac只知道依赖的类叫啥，但是存储这个类的源码文件叫啥可就无能为力，毕竟源码文件的名称和类的名称可是没关系的啊。说到这里思路就已经很清晰了，为了能让javac自动编译前置类，需要强制添加一种约束：源码文件名称要和它存储的类的名字保持一致。javac是不是按照这种思路设计的呢？做个实验：
```
➜  show ls
demo1.java  demo2.java
➜  show javac demo1.java 
➜  show ls
demo1.class  demo1.java  demo2.class  demo2.java
```
这里源码文件都改成了类名，类本身的代码同上。我们发现，虽然没有手动编译类demo2，但是在编译类demo1的时候，javac“顺便”自动把demo2也给编译了。于是证实了我们的推测：javac确实是根据类名去寻找前置类的源码文件的。那么，当你在写java代码的时候，如果一个类依赖了其他的类，为了使这个类能顺利通过编译，你有两个选择：手动把所有的依赖类编译出来；或者让javac自动帮你编译。如果选择后者，你需要把依赖类保存在以它的名字命名的源码文件里。
回到开篇的问题，终于迎来了一丝曙光，虽然仍然没能彻底回答，但本文至少证明了一件事：以类名来命名源码文件对编译来说好处大大的有。要彻底回答开篇的问题，还要牵扯权限的问题，下篇继续。
