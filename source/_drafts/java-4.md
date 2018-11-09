---
title: Java漫谈（四）
tags: java 编译 权限
categories: 技术
abbrlink: 34891
date: 2018-11-08 17:05:54
---
####命名空间
#####包名
Java代码里面的类写多了，自然会遇到重名的问题，尤其是涉及到多人协作的时候。比如，两个人都想给自己的类命名为“Apple”，那么第三个人调用“Apple”这个类的时候，到底调的是哪个呢？
这就是命名冲突，Java给出的解决方案是引入包名，也就给类名加个前缀，如：com.Demo2。Demo2类的包名就是com，类名和包名之间用点号隔开。这时候“类名”就有了两个不同版本的叫法：全限定类名和非限定类名。前者指的是加上前缀之后的全名，后者就是平时所说的类名，也就是class关键字后面的名字。语法上，使用package关键字来单独声明包名，而不是将全限定类名放在class关键后面。如下所示：
```
package com;

class Demo {}
```
那么，类Demo2的全限定类名就是com.Demo2。有了前缀，就不怕冲突了，两个重名的类，只要他们的前缀不一样，也就是全限定类名不一样，就不会引起冲突。当然，引用的时候，需要写入全限定类名，而不是之前的“简称”。前缀，也就是包名，给了类一个安全的命名空间，只要在这个空间内没有重名就好，其余的事情让包名去考虑。在java的规范中，**包名需要全部小写**。
#####包的层次
如果包名也重名了怎么半？当然，理论上讲，包名可以做到不重名。26个英文字母随意排列组合，而且长度也可以无限扩大，怎么着都能组装出一个不同的包名。然而可以想象的是，随着类的无限增加，这个包名也会跟着不断增长，而且随意排列的名称也越来越没有意义。这显然不是我们想要的，于是设计者给包名添加了层级结构，类似于文件系统的目录树，**包下面还可以有子包**，各级之间的包名也用点号隔开。这样以来，问题也就解决了：当给一个类起了一个喜欢的包名，发现被别人占用了，那么给它加一个“父包名”；如果“父包名”也冲突，重复刚才的步骤，直到没有冲突。实际上，设计者建议使用公司层级倒序的域名作为包名，域名在世界范围内几乎是唯一的。
#####默认包
如果源码文件中没有使用package关键字来声明该类的包名（语法上这是合法的），那么这个类就属于默认包。

####默认权限
默认权限其实指的就是包权限。当声明一个类时，若class关键字前没有任何访问权限修饰符，那么这个类默认的可访问权限就是包权限，即它*只能被同一个包下的类访问*，不是同一个包的类无法访问到它。
下面将类Demo1和Demo2的包名分别声明为com1和com2,如下：
```
package com1;

class Demo1 {
    public static void main(String[] args) {
        System.out.println(com2.Demo2.a);
    }
}
```
```
package com2;

class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
Demo1依赖了com2.Demo2，注意这里必须写Demo2的全限定名，编译如下：
```
➜  show javac Demo2.java 
➜  show ls
Demo1.java  Demo2.class  Demo2.java
➜  show javac Demo1.java 
Demo1.java:5: 错误: 找不到符号
        System.out.println(com2.Demo2.a);
                           ^
  符号:   变量 demo2
  位置: 类 Demo1
1 个错误
```
由于Demo2没有和Demo1在同一个包下，所以Demo1无法访问到Demo2。同时可以看到，编译之后得到的class文件，其名称仍然只是类的“简称”，而非类的全限定名。
想要让Demo1编译成功，有两个办法：将二者的包名改为同一个；或者将类Demo2的访问权限改为public。如果Demo2是一个“私有类”，即开发者不希望这个类被别人依赖或访问，那么将其访问权限限制在包内是个好办法，这也就意味着该类只提供“包内服务”，不提供“公共服务”。反之，如果是开发者对外发布的接口，那么这个类必须设置为public，这样才会被别人访问到。
####包结构与目录树
接上面，采用第二种办法，将类Demo2的访问权限修改为public：
```
package com2;

public class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
继续编译：
```
➜  show ls
Demo1.java  Demo2.java
➜  show javac Demo2.java 
➜  show ls
Demo1.java  Demo2.class  Demo2.java
➜  show javac Demo1.java 
Demo1.java:5: 错误: 程序包com2不存在
        System.out.println(com2.Demo2.a);
                               ^
1 个错误
```
新的问题出现了。猜测需要在当前目录下新建一个com2目录，然后将Demo2.class放在该目录下才可以，验证如下：
```
➜  show mkdir com2
➜  show mv Demo2.class com2 
➜  show javac Demo1.java 
➜  show ls
com2  Demo1.class  Demo1.java  Demo2.java
```
果然如此，现在Demo1可以正常编译。这说明，当在类A的源码中以全限定名称引用类B的时候，javac会按照类B的包名层级关系，以当前目录为根目录，
去下面对应的目录中寻找类B的class文件。那么，“当前目录”指的是类A的源文件所在的目录还是执行javac命令时所在的目录呢？现在切换到show目录的上级目录去编译Demo1.java：
```
➜  show cd ..
➜  ~ javac show/Demo1.java 
show/Demo1.java:5: 错误: 程序包com2不存在
        System.out.println(com2.Demo2.a);
                               ^
1 个错误
```
这说明，“当前目录”指的是执行javac时所在的目录，所以在哪个目录下执行javac命令很重要。