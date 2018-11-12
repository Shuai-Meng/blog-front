---
title: Java漫谈（四）
tags: java
categories: 技术
abbrlink: 34891
date: 2018-11-08 17:05:54
---

### 命名空间
#### 包名
&ensp;&ensp;&ensp;&ensp;Java代码里面的类写多了，自然会遇到重名的问题，尤其是涉及到多人协作的时候。比如，两个人都想给自己的类命名为“Apple”，那么第三个人调用“Apple”这个类的时候，到底调的是哪个呢？
&ensp;&ensp;&ensp;&ensp;这就是命名冲突，Java给出的解决方案是引入包名，也就给类名加个前缀，如：com.Demo2。Demo2类的包名就是com，类名和包名之间用点号隔开。这时候“类名”就有了两个不同版本的叫法：全限定类名和非限定类名。前者指的是加上前缀之后的全名，后者就是平时所说的类名，也就是class关键字后面的名字。语法上，使用package关键字来单独声明包名，而不是将全限定类名放在class关键后面。如下所示：

```java
package com;

class Demo {}
```
<!-- more -->
&ensp;&ensp;&ensp;&ensp;那么，类Demo2的全限定类名就是com.Demo2。有了前缀，就不怕冲突了，两个重名的类，只要他们的前缀不一样，也就是全限定类名不一样，就不会引起冲突。当然，引用的时候，需要写入全限定类名，而不是之前的“简称”。前缀，也就是包名，给了类一个安全的命名空间，只要在这个空间内没有重名就好，其余的事情让包名去考虑。在java的规范中，**包名需要全部小写**。
#### 包的层次
&ensp;&ensp;&ensp;&ensp;如果包名也重名了怎么半？当然，理论上讲，包名可以做到不重名。26个英文字母随意排列组合，而且长度也可以无限扩大，怎么着都能组装出一个不同的包名。然而可以想象的是，随着类的无限增加，这个包名也会跟着不断增长，而且随意排列的名称也越来越没有意义。这显然不是我们想要的，于是设计者给包名添加了层级结构，类似于文件系统的目录树，**包下面还可以有子包**，各级之间的包名也用点号隔开。这样以来，问题也就解决了：当给一个类起了一个喜欢的包名，发现被别人占用了，那么给它加一个“父包名”；如果“父包名”也冲突，重复刚才的步骤，直到没有冲突。实际上，设计者建议使用公司层级倒序的域名作为包名，域名在世界范围内几乎是唯一的。
### 默认权限
&ensp;&ensp;&ensp;&ensp;默认权限其实指的就是包权限。当声明一个类时，若class关键字前没有任何访问权限修饰符，那么这个类默认的可访问权限就是包权限，即它***只能被同一个包下的类访问***，不是同一个包的类无法访问到它。
将类Demo2的包名声明为com2，如下：
```java
package com2;

class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
&ensp;&ensp;&ensp;&ensp;参照上篇文章中的目录树结构，src子目录专门存放源代码，而target目录作为所有class文件的根目录。先编译Demo2：
```shell
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target

2 directories, 2 files
➜  show javac -d target src/Demo2.java 
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target
    └── com2
        └── Demo2.class

3 directories, 3 files
➜  show 
```
&ensp;&ensp;&ensp;&ensp;可以看到，编译之后得到的class文件，其名称只是类的“简称”，**而非类的全限定名**；同时javac在target目录下自动生成了com2子目录，对应类Demo2的包名。可以想象，如果Demo2的包名具有多级结构，那么在根目录（即使用-d参数指定的class文件根目录，或者默认为当前目录）下也将生成与包名对应的目录树。另外一种方案是，直接在target目录下生成以类的全限定名称命名的class文件，而非建立与包名对应的目录树——但是javac并没有采用这种方案。
&ensp;&ensp;&ensp;&ensp;接下来将类Demo1的包名设为com1，并且Demo1访问了com2.Demo2的a属性，注意*Demo1引用Demo2的时候，写的是其全限定名*。
```java
package com1;

class Demo1 {
    public static void main(String[] args) {
        System.out.println(com2.Demo2.a);
    }
}
```
编译过程如下：
```shell
➜  show javac -cp target/com2 -d target src/Demo1.java
src/Demo1.java:5: 错误: 程序包com2不存在
        System.out.println(com2.Demo2.a);
                               ^
1 个错误
```
&ensp;&ensp;&ensp;&ensp;程序包com2显然是存在的，这里之所以报错是因为-cp指定的路径有问题——实际上只需要指定class文件的根目录就可以了，而由类的包名自动生成的目录树，javac会自动根据包名去搜索，不必指定。看起来与上篇文件说的“javac不会递归搜索子目录”相矛盾，但这是由于那时尚未引入包机制导致的。重新编译如下：
```shell
➜  show javac -cp target -d target src/Demo1.java 
src/Demo1.java:5: 错误: Demo2在com2中不是公共的; 无法从外部程序包中对其进行访问
        System.out.println(com2.Demo2.a);
                               ^
1 个错误
➜  show 
```
&ensp;&ensp;&ensp;&ensp;终于出现了想要的错误（汗，有点像设计剧情）——由于Demo2没有和Demo1在同一个包下，且Demo2没有被public修饰，所以Demo1无法访问到Demo2。
&ensp;&ensp;&ensp;&ensp;想要让Demo1编译成功，有两个办法：将二者的包名改为同一个；或者将类Demo2的访问权限改为public。如果Demo2是一个“私有类”，即开发者不希望这个类被别人依赖或访问，那么将其访问权限限制在包内是个好办法，这也就意味着该类只提供“包内服务”，不提供“公共服务”。反之，如果是开发者对外发布的接口，那么这个类必须设置为public，这样才会被别人访问到。
### 回答
&ensp;&ensp;&ensp;&ensp;好，现在回答最初的那个问题：为什么源代码文件要以public类的名字命名？
&ensp;&ensp;&ensp;&ensp;还是为了自动化编译提高效率。public类是可以被任何类访问的类，那么当javac因其他类依赖这个public类而要编译它的时候，必然需要根据类名去寻找这个类的源代码文件，所以源代码文件必须要根据public类去命名。实际上，只要某个类想要被javac自动编译，就必然需要单独保存在以它自己的名字命名的源代码文件中，不管它是不是public类。虽然一个源代码文件中可以出现多个类的声明，但是只要有一个类被public修饰了，当其他类引用这个类的时候，就必须要保证javac可以根据类名找到这份源代码。而同一份源代码文件中，非public类会因为public类的编译而同时被编译。当然，如果类A依赖了同包下的非public类B，但是类B的源代码文件是以同一份文件中public类C的名字命名的，那么类B将不会因为A的依赖而被自动编译——除非类C因为某种原因被编译了。不过话说回来，既然想要类B被自动编译，为啥不把它单独存放在一个以它自己的名字命名的文件中呢？实际上，如果一份源代码文件中同时出现了public和非public类，那么一般情况下，这些非public类都只是为这一个public类服务的，设计者其实不希望这些非public类被这个public类以外的任何类访问到——哪怕是在同一个包下的类。
&ensp;&ensp;&ensp;&ensp;上面这个需求实际上使用内部类可能更合适些，不知道是不是java的遗留问题，目前我在java类库的一些源代码文件确实发现了public类和非public共存的现象。
### 包结构与目录
#### class文件目录
&ensp;&ensp;&ensp;&ensp;接上面，采用第二种办法，将类Demo2的访问权限修改为public：
```java
package com2;

public class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
&ensp;&ensp;&ensp;&ensp;继续编译：
```shell
➜  show javac -d target src/Demo2.java 
➜  show javac -cp target -d target src/Demo1.java 
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target
    ├── com1
    │   └── Demo1.class
    └── com2
        └── Demo2.class

4 directories, 4 files
```
&ensp;&ensp;&ensp;&ensp;编译成功，现在target目录下有com1和com2两个子目录，分别存放Demo1.class和Demo2.class。
#### 源代码目录
&ensp;&ensp;&ensp;&ensp;然而，Demo2.class是手动编译出来的，但是javac有自动编译依赖类的功能，为啥不用呢？
```shell
➜  show rm -rf target/*.class
➜  show tree
.
├── src
│   ├── Demo1.java
│   └── Demo2.java
└── target

2 directories, 2 files
➜  show javac -sourcepath src -d target src/Demo1.java
src/Demo1.java:5: 错误: 程序包com2不存在
        System.out.println(com2.Demo2.a);
                               ^
1 个错误
```
&ensp;&ensp;&ensp;&ensp;看来包名影响的不光是class文件的目录组织，源代码也受到了影响。想要使用javac的自动编译功能，源代码的目录结构也需要按照包名去组织，如下：
```shell
➜  show tree
.
├── src
│   ├── com1
│   │   └── Demo1.java
│   └── com2
│       └── Demo2.java
└── target

4 directories, 2 files
➜  show javac -sourcepath src -d target src/com1/Demo1.java
➜  show tree
.
├── src
│   ├── com1
│   │   └── Demo1.java
│   └── com2
│       └── Demo2.java
└── target
    ├── com1
    │   └── Demo1.class
    └── com2
        └── Demo2.class

6 directories, 4 files
```
#### 执行
```
➜  show java target/com1/Demo1
错误: 找不到或无法加载主类 target.com1.Demo1
➜  show java -cp target com1.Demo1
9
➜  show cd target 
➜  target java com1.Demo1 
9
➜  target cd com1
➜  com1 java Demo1 
错误: 找不到或无法加载主类 Demo1
```
&ensp;&ensp;&ensp;&ensp;上面的试错说明了两件事：
&ensp;&ensp;&ensp;&ensp;1. 不同于javac命令，使用java命令执行一个类时，不可以直接指定class文件的实际路径，只能写全限定类名，java会自动根据包名去实际路径下寻找；
&ensp;&ensp;&ensp;&ensp;2. -classpath参数对于java命令是一样的，用来指定**class文件的根目录**（顶级包所在的目录），并且java默认的搜索路径就是当前目录。
### 默认包
&ensp;&ensp;&ensp;&ensp;如果源码文件中没有使用package关键字来声明该类的包名（语法上这是合法的），那么这个类就属于默认包。
#### 与目录无关
&ensp;&ensp;&ensp;&ensp;默认包下的所有类都是可以互相访问的，不管它们有没有实际上在同一个目录下。上面两个Demo类，去掉package相关的代码后，编译过程如下：
```
➜  show tree
.
├── src
│   ├── com1
│   │   └── Demo1.java
│   └── com2
│       └── Demo2.java
└── target
    ├── com1
    └── com2

6 directories, 2 files
➜  show javac -d target/com2 src/com2/Demo2.java
➜  show javac -cp target/com2 -d target/com1 src/com1/Demo1.java
➜  show tree
.
├── src
│   ├── com1
│   │   └── Demo1.java
│   └── com2
│       └── Demo2.java
└── target
    ├── com1
    │   └── Demo1.class
    └── com2
        └── Demo2.class

6 directories, 4 files
➜  show java -cp target/com1:target/com2 Demo1
9
```
&ensp;&ensp;&ensp;&ensp;target下的com1和com2是故意保留的两个目录，分别存放编译后的Demo1.class和Demo2.class。类Demo1的编译和执行都成功了，验证了上面的猜测。
#### 引用默认包的public类
&ensp;&ensp;&ensp;&ensp;问题：如果想要引用默认包中的public类，该怎么做？代码更改如下：
```
package com1;

class Demo1 {
    public static void main(String[] args) {
        System.out.println(Demo2.a);
    }
}
```
```
public class Demo2 {
    public static int a = 9;

    public static void main(String[] args) {
        System.out.println(a);
    }
}
```
&ensp;&ensp;&ensp;&ensp;编译：
```
➜  show javac -d target/com2 src/com2/Demo2.java
➜  show javac -cp target/com2 -d target/com1 src/com1/Demo1.java
src/com1/Demo1.java:5: 错误: 找不到符号
        System.out.println(Demo2.a);
                           ^
  符号:   变量 Demo2
  位置: 类 Demo1
1 个错误
```
&ensp;&ensp;&ensp;&ensp;报错了，咋办？参考这篇[文章](https://blog.csdn.net/xupan_jsj/article/details/8985189)吧，懒得写了。