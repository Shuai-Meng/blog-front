---
title: Java 漫谈（二）
tags: java
categories: 技术
abbrlink: 18610
date: 2018-11-05 00:43:04
---
#### 类名
&ensp;&ensp;&ensp;&ensp;Java编译器的设计与java源代码的语法结构是相辅相成的，先有语法，再有编译器。大家都知道，java是以类为单位进行组织的，类是java对世界的抽象描述，Java程序的运行其实就是多个类之间的相互调用。表现在源码上，java的语法规定，用class关键字声明一个类，如下所示：
```
class Demo {
    public static void main(String[] args) {
        System.out.println(9);
    }
}
```
&ensp;&ensp;&ensp;&ensp;class关键字紧跟一个空格后面就是类的名称，再隔一个空格就是类的内容，用一对花括号包裹，类中的内容主要分为字段（也称为域或属性）和方法两部分。上面这个类没有属性，只有一个名为main的方法，它将会在屏幕上打印数字9。把这个类的代码保存在文件test.java中。
很多教科书上都说，类名需要首字母大写，但我自己实验了一下，其实小写也可以编译并且运行。然而，还是建议遵循规范，类名首字母大写。

<!-- more -->

#### javac与java命令
&ensp;&ensp;&ensp;&ensp;编译并执行上面的test.java，如下：
```
➜  show ls
test.java
➜  show javac test.java
➜  show ls
Demo.class test.java
➜  show java Demo
9
```
&ensp;&ensp;&ensp;&ensp;上面的操作是在Linux系统的命令行中，show是当前所在目录的名字，编译使用的javac命令以及执行使用的java命令都是java自身提供的原生命令，这两个命令可以说是java最基本的两个工具了。高级开发工具（IDE）如IDEA，底层其实也是调用的这两个命令，IDE虽然使开发更方便，但代价就是隐藏了很多底层细节，让你不知道在操作系统层面到底发生了什么。我个人其实更喜欢更底层打交道，这样才会对计算机技术了解的更深。现在，从上面的展示中可以发现以下信息：
&ensp;1. 编译之后的class文件名与类名保持一致；
&ensp;2. 源代码文件名可以与类名不一致，但是后缀必须是.java，否则编译器无法识别；
&ensp;3. 执行时，直接在java命令后面写上类名即可，不必加上.class后缀，加上反而会报错。
&ensp;&ensp;&ensp;&ensp;第二点需要展开说一下，一个java的源代码文件中，可以保存多个类，但是源码中声明了几个class，在编译后就会生成几个对应的class文件。这意味着，一个class文件只能保存一个类的信息，但是多个类的源代码可以共存在一个.java文件中。习惯上，我们倾向于一个java文件只保存一个类的源码，也就是只声明一个class，很少出现多个class的情形，这样做的目的是使工程结构更清晰，更方便管理。共存情况示例如下：
```
class Demo1 {
    public static void main(String[] args) {
        System.out.println(9);
    }
}

class Demo2 {
    public static void main(String[] args) {
        System.out.println(8);
    }
}
```
```
➜  show javac test.java
➜  show ls
Demo1.class Demo2.class test.java
```
#### 访问权限对源码文件的影响
##### 访问权限
&ensp;&ensp;&ensp;&ensp;下面引入访问权限的概念。说到访问权限，不得不提面试经常会被问到的java作为面向对象语言的三大特性：封装、继承和多态，而访问权限正是特性之一“封装”的体现。所谓封装，指的是隐藏实现细节，这也是设计的哲学之一：客户只需要知道功能怎么用即可，而不需要知道这个功能是如何实现的。
&ensp;&ensp;&ensp;&ensp;Java中的访问权限分为四个级别：私有权限、默认权限、继承权限和公共权限，权限依次增大。具体到语法上：
&ensp;1. private关键字代表私有权限，只能修饰类的字段或方法，不能修饰类本身，私有权限的意思是只能在该类内部可访问，其他类访问不到；
&ensp;2. protected关键字代表继承权限，同样无法修饰类本身，被protected修饰的字段或方法，只能被继承该类的子类所访问；
&ensp;3. public关键字代表公共权限，可以修饰类本身以及类的字段或方法，其含义是其他任何类都可以访问；
&ensp;4. 若类、类的字段或方法没有被以上任何一个关键字修饰，则意味着default（默认）权限，默认权限意味着只有和该类在同一个包下的类才可以访问。
&ensp;&ensp;&ensp;&ensp;注意，*默认权限并没有对应的关键字*，default虽然是java的一个关键字，但是跟权限无关；若想要类或类的字段及方法拥有默认权限，只要不在其前面添加任何权限修饰符即可。关于继承、包以及更多访问权限相关内容，以后会慢慢揭开它们的神秘面纱。
##### 对源码文件的影响
&ensp;&ensp;&ensp;&ensp;由上可知，类本身的访问权限只有两种：公共权限或默认权限。具体到语法上，class的权限修饰符，要么是public，要么啥都没有。这会产生什么影响呢？这会影响.java文件的命名。上面提到，同一份java源码文件中可以声明多个类，而且文件的命名与类的命名毫无关系。现在，引入访问权限机制以后，增加了以下约束：
&ensp;1. 在一份源码文件声明的所有类当中，最多只能有一个被public修饰；
&ensp;2. 若一份源码文件中存在一个类被public修饰，那么该源码文件的命名必须和该类名一致。
##### 验证
&ensp;&ensp;&ensp;&ensp;test.java文件修改如下：
```
public class Demo1 {
    public static void main(String[] args) {
        System.out.println(9);
    }
}

class Demo2 {
    public static void main(String[] args) {
        System.out.println(8);
    }
}
```
&ensp;&ensp;&ensp;&ensp;编译：
```
➜  show javac test.java
test.java:1: 错误: 类Demo1是公共的, 应在名为 Demo1.java 的文件中声明
public class Demo1 {
       ^
1 个错误
```
&ensp;&ensp;&ensp;&ensp;这说明，一旦有一个类被声明为public，那么源码文件的名称必须以该类的名称命名。将文件改名为Demo1.java后编译通过。如果两个类都是public的会怎样？

```
public class Demo1 { 
    public static void main(String[] args) {
        System.out.println(9);
    }
}

public class Demo2 {
    public static void main(String[] args) {
        System.out.println(8);
    }
}
```
```
➜  show javac Demo1.java
Demo1.java:7: 错误: 类Demo2是公共的, 应在名为 Demo2.java 的文件中声明
public class Demo2 {
       ^
1 个错误
```
&ensp;&ensp;&ensp;&ensp;上面的测试表明，当在一个.java文件中声明一个以上的public class时，会出现一山不容二虎的情况，编译器会提示每个public类要保存在一个单独的以自己名字命名的.java文件中。
&ensp;&ensp;&ensp;&ensp;那么问题来了：java为什么要这么设计？网上查了很多资料，并没有特别使我信服的，不过确实开阔了一下脑洞。自己做了些实验，应该解开了这个谜题，欲知后事如何，且听下回分解。
