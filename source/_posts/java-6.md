---
title: Java总结——类与对象
tags: java
categories: 技术
abbrlink: 4618
date: 2019-08-18 18:31:10
---

### 基本数据类型与对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java是面向对象的语言，所以光有基本数据类型是不够的，对象才是主角。基本数据类型的存在，一方面是向计算机本身物理结构的妥协，另一方面是为了构造对象。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编程语言所面对的问题和事物是复杂的，多维度的，需要用一个组合体来囊括方方面面，各个角度。比如描述一个人，他有姓名，年龄，性别，住址，身高，体重等等，这么多信息，不是哪个基本数据类型可以表示的。可以理解为，每个基本数据类型都是一维的，只有把他们组合起来，才能表示多维的事物。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基本数据类型，说穿了其实就两类：数字和文字。除了char类型表示文字，其他的类型可以视为数字。在人类的自然语言中，其实描述事物不外乎也就两个手段：文字和数字——文字描述“质”，数字描述“量”。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对象则可以包含二者,它是现实中的事物在计算机中的映射。但是对象可以映射的不仅仅是具体事物，还可以是抽象的事物，比如一个查询。只要这个事物可以被描述，或者可度量。对象在计算机中代表了世界的一切，当编程语言可以在计算机中描述一切时，就可以模拟它们的运行。

### 类与对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类与对象的关系，就是基本数据类型与其值之间的关系，比如
```
int a = 5；
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;变量a的类型是int，值是5。类型代表一类事物，int代表着整数，5只是整数中的一个而已，还有很多其他的整数值。类，是新创造出来的类型，也代表一类事物。一个该类型的对象，也只是类型中所有值中的一个而已。同一类型的对象之间，他们的属性个数和名称必然一样，但是每个属性的值可能不一样。属性的个数及名称决定了“质”，而属性的具体值决定了“量”，也可以称为状态。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从衍生的角度看，创建了类，然后由这个模板批量制造对象；从归纳的角度，存在一批相似的对象，然后将其抽象成了类，代表这批对象公共的、共同的特征。

<!-- more -->

### 语法
#### 类
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类主要由两部分构成：属性和方法。属性代表着状态，即这个类是什么；而方法代表行为，即这个类能做什么。还是先举例：
```
class Apple {
    float weight;
    void grow() {
        weight += 1.0;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Apple类只有一个属性weight，表示重量；只有一个方法grow，表示生长。创建Apple的对象后，可以通过点号运算符访问属性和方法：访问属性可以获取该属性的值；访问方法可以改变或者获取相应的属性值——大部分方法都是在操纵某些属性，当然也不全是。

#### 构造方法与创建对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有了类之后要如何创建对象呢？对于基本类型可以这样：
```
int a = 4；
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是对于复合类型，并没有像4这样的字面值可供使用，于是Java提供了new + 构造方法的方式来创建对象，如下：
```
Apple a = new Apple();
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很像对普通方法的调用，除了必须用new关键字来配合。那么构造方法哪儿来的呢？类中定义的：
```
class Apple {
    int weight;
    Apple() {
        weight = 10;
        System.out.println(“weight:” + weight);
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了不能有返回值以外，其他地方似乎没有区别，不过构造方法的名称必须和类名保持一致。构造方法主要是用来初始化属性值的，这样可以让对象在创建时就拥有指定的状态，当然也可以在其中做一些其他的工作，跟普通方法一样。有了构造方法之后，创建对象时，必须按照构造方法指定的方式来创建对象。上面的构造方法没带参数，所以给weight属性赋了一个固定值；但实际上构造方法可以带参数，用调用构造方法时传入的值来初始化各个属性，这样在创建对象时就可以指定任意的状态。
```
class Apple {
    int weight;
    Apple(int w) {
        weight = w;
    }
}

Apple a = new Apple(2);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么如果不在类中定义构造方法，是不是就无法创建对象了呢？理论上是这样的，如果最后编译成的类class文件中不含有构造方法，那么这个类就无法创建对象了。但是，从源代码到class文件有个编译的过程，所以编译器是可以在其中做一些手脚，比如检查是否存在构造方法，没有的话就自动帮你写一个默认的构造方法。所谓默认的构造方法，其实是就是空参且没有方法体的构造方法。但是如果存在构造方法，就不管了。比如上例，类中有且只有一个Apple(int w)方法，那么下面的代码将会报错：
```
Apple a = new Apple();
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为此时的class文件中没有了空参的构造方法。所以，如果要使用空参的构造方法来创建对象，要么不在类中写任何构造方法，让编译器替你做；要么手写一个空参的构造方法——无论类中有没有其他重载的带参数的构造方法。

#### 方法重载
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java提供了方法重载机制，即：一个类中可以存在同名的方法，只要它们的方法签名不同。所谓方法签名包括了方法名称和方法参数列表，不包含返回值；参数列表包括参数的个数、类型和排序，不包含参数名称。如下：
```
void grow（int，float，int）；
int grow（float， int， int）；
float grow（int，float）；
void grow（int， long）；
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上四个方法是可以共存于一个类中的，它们仅仅是名字一样。当然第二种重载的方式是不推荐的，它们仅仅是参数排序不同，个数和类型都是一样的，这样及其容易引起混淆。所以，重载中的“重”，仅仅指的是名字。老实说，不太理解为啥要搞这么个重载机制，多个方法共用一个名字能带来什么好处？我绞尽脑汁想到了两个理由：
1. 起名字确实是个费力的活儿，如果经常玩游戏或者生孩子你会懂的。。。。
2. 如果需要重载的方法是构造方法。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;单说第二条，为什么需要多个不同的构造方法？理想状态下，只需要一个全参数的构造方法即可，足够初始化类的所有属性。但是有些情况下我们想忽略其中的一些属性，即只传部分参数，其他的保持默认值即可。这样以来，每次调用全参数的构造方法就会麻烦又啰嗦。由于创建对象时只能按照构造方法规定的方式来，所以我们需要另外一个构造方法；而构造方法又必须和类名保持一致，于是我们需要多个同名且参数列表不同构造方法。这时只能引入重载机制，将方法签名（方法名加上参数列表）作为每个方法的唯一标识（被jvm识别），也就是方法可以重名了，重名的方法不会引起混淆，如下：
```
class Apple {
    int weight;
    Apple(int w) {
        weight = w;
    }
    
    Apple() { }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看起来，重载机制是为了使多个构造方法并存而发明的。虽然对普通方法也是有效，但更像是无心插柳。


#### this关键字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我将this理解为一个占位符，代表将来的、尚未创建出来的对象本身。因为在写代码时，写的是类本身，是个模版，而非具体的对象。在创建对象时，操作系统会为每个对象开辟存储空间，但仅仅是用来存放具体的属性值，存储空间的大小按照类中定义的属性分配，空间不能共享，因为每个对象的状态不同；但是方法却可以共享，因为方法的逻辑是通用的，没必要为每个对象单独存储一份方法代码，于是方法的代码存储在公共的空间中——也就是类本身的空间。于是，对象需要存储空间，也需要存储空间，但它们存储的内容不一样：类的空间主要存储方法，而对象的空间主要存储属性。
以grow方法为例：
```
void grow() {
    weight += 1.0;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;grow方法要对weight属性操作，注意这是类中的代码，是公共的，不是某个对象特有的。那么如何保证赋值给了正确的对象？毕竟Apple类会有很多个对象。所以就需要一个对象的指针，来保证方法是对正确的对象操作的，这个指针，就是this。上面提到方法是通过对象来调用的：
```
apple.grow();
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这看起来像是apple对象单独存储了一份grow方法的代码，grow方法属于这个对象。但其实方法是公共的，对象的存储空间中并没有方法代码，通过对象来调用方法似乎有点说不通。实际上，这只是为了向grow方法传递apple对象的地址（或者叫指针）而已，编译器会将这行代码编译成类似下面的代码（注意这里只是为了理解本质，编译器可能不是这么做的，但道理相通）：
```
grow(apple);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等等，类中的grow方法是空参的，这里强行插入一个Apple类型的参数合适吗？当然不合适！但是grow方法只是表面上空参，实际上它长这样：
```
void grow(Apple this);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样就说得通了。类中的所有方法（构造方法应该不在此列）都有一个隐式的参数，而且排在第一位，就是this。它可能实现为一个普通的属性变量，类型就是当前类，创建对象时会自动将这个变量赋值为当前对象的地址。构造方法应该是没有这个隐式参数的，毕竟调用构造方法时对象还不没创建出来，没办法给构造方法传参——语法上构造方法也不能在对象上调用。this指针是隐式的，编译器和jvm会自动处理，我们直接拿来用就可以了。在类中声明方法时也没有必要把它显式写进参数列表（不合法）；在方法中调用另一个方法时，也没有必要在被调用方法名前面加上”this.”（虽然合法），只是知道调用方法时有这么个机制就行。说完隐式，那么显式的this怎么用？用来干嘛？
1. 类中引用属性，防止重名
```
class Apple {
    int weight;
    Apple (int weight) {
        this.weight = weight;
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当方法尤其是构造方法的参数中有与属性同名的参数，且在方法体中操作了同名的属性（这种情况是怎么发生的呢？），则需要在属性前面饮用this关键字，防止混淆。虽然写代码时对象还没创建出来，但是当jvm执行到这段代码时，this已经被赋值——相当与提前占位了。this在构造方法中使用也是合法的，这意味着执行构造方法的方法体时，对象已经创建出来了，构造方法的方法体只是在初始化对象的属性。
2. 构造方法之间相互调用
```
class Apple {
    int weight;
    Apple(int weight) {
        this.weight = weight;
    }
    
    Apple() {
        this(4.5);
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;普通的重载方法之间可以直接调用，但是构造方法却不可以，像这样是会报错的：
```
Apple() {
    Apple(4.5);
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这时只能用this( )来代替对构造方法的调用。还需要注意的是，this()的调用只能发生在构造方法内，普通方法内部不能调用；而且必须是方法体的第一条语句。如下的语句会报错：
```
Apple() {
    int a = 9;
    this(4.5);
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面三个规则是为什么呢？

#### 初始化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对象的初始化指的是其属性被赋值的过程。这个过程有三个节点：第一个是对象刚被分配空间时，这个节点可以形象的理解为“清零”，即将这块内存区域以前使用的痕迹抹去，换成属性对应类型的默认值，比如int类型赋值0，引用类型赋值null。这个赋值过程是jvm自动执行的，无需人工干预。这个特点是Java语言的一大卖点，C++就不会做这种“多余”的动作。这么做的好处是更安全，对象的初始化状态不是一堆乱起八糟的“前任”值，而是经过清理的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要注意的是，这个清理动作只发生在对象创建的时刻，或者说只对属性变量有效；对于局部变量，也就是方法体内临时定义的变量，jvm不会有这个动作，所以必须手动初始化——给变量赋值，否则无法通过编译。（这又是为啥呢？一个猜测是对象的初始化发生在堆上，jvm可以干预属性变量的赋值；而局部变量在栈上，jvm干预不了）
第二个节点就是在声明变量的同时赋值，如下：
```
class Apple {
    int weight = 3;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样的属性值是写死的，或者说硬编码的，缺乏灵活性：每个new出来的对象，其weight属性都是3。当然某些情况下需要这样做。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第三个节点就是构造方法了，在构造方法内给属性赋值，可以是固定值，类似于第二个节点；也可以是参数值，更灵活。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;至此，初始化过程就结束了，构造方法是最后的机会。再往后对属性值的改变就不属于初始化了，比如各种setter方法。有啥子区别吗？我反正没看出来。Spring给对象的属性赋值时，基本靠的是setter方法，当然构造方法也是可以的。

#### static关键字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面提到类的存储空间中主要保存了方法代码，而属性保存在对象的存储空间中，这两个存储空间是分开的。而实际上类的存储空间中也可以保存属性值，只要将属性用static关键字修饰。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;static代表静态的，Java引入这个机制是为向所谓的“全局变量”妥协。被static修饰的变量，可以被每个该类的对象访问，可以看成是共享的；由于存储在类空间，所以可以直接通过类名访问，如下：
```
class Apple {
    static String name = “apple”;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后就可以这样访问了：
```
Apple.name;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从实现的角度，static变量跟对象无关，它存放在类空间；但是从语义的角度，static变量用来表达所有对象都共有的属性，比如上例中的name属性，每个Apple对象或许weight属性不同，但是name都是一致的。一旦name发生了变化，所有对象的这个属性也会跟着变化；从这个角度理解，static变量的确是全局的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;static还可以用来修饰方法，表示这个方法跟具体对象无关，不必再通过对象名来访问了（当然，static方法不会有隐式的this指针）。static方法表示那些可以直接可以直接以类名来访问的方法，毕竟创建对象是需要花费时间和空间的。static方法当然也可以通过对象来访问，只是需要先创建一个对象，不如直接使用类名省事。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;static还有一个重要用途是static块，当然它还是跟对象无关(static跟对象绝缘)，而是表示在加载类的时候需要做的事，如下：
```
class Apple {
    static {
        System.out.println(“init”);
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类加载是发生在创建对象之前的。与此对应，也可以创建没有static修饰的块，表示在创建对象时要做的事，如下：
```
class Apple {
    {
        System.out.println(“init”);
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以手动验证下是否在构造方法执行之前。

#### main方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先思考一个问题：java的代码是如何运转起来的？类中的确定义了很多变量和方法，问题是谁来调用它们呢？总不会自动执行吧？你会说是在另一个类中被调用的，可另一个类又是被谁调用的？这样调用来调用去，最终的调用者是谁？是某个固定的类吗？这些问题的其实可以归结为：java程序执行的起点在哪儿？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在C或着C++中，都会有一个main函数，操作系统以这个函数为入口，顺序执行其中的代码。Java也提供了这样的机制——main方法，举个例子：
```
public static void main(String[] args) {
    Apple apple = new Apple(4);
    apple.grow(4);
    System.out.println(apple.weight);
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java程序都是从这里启动并执行的。与C/C++不同的是，Java程序是由jvm来执行的，而不是直接由操作系统来执行。说到底，Java是解释型的语言，由jvm一行行解释执行java编译后的字节码；C/C++则直接编译成了二进制的系统指令，操作系统可以直接拿来执行。Java语言不是自举的，它无法自己执行自己，需要依赖外部力量——jvm一般是由C++编写实现的。jvm本身也是运行在操作系统中的程序，main方法是jvm程序与java程序之间的接口，像是约定好的暗号，在命令行输入以下命令：
```
java Apple
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jvm启动，寻找并加载Apple类，检查类中是否有入口——main方法，而不是别的其他方法，然后开始执行其中的代码。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以，java程序的启动工作都放在这个方法里面，比如创建一个对象并执行其中某个方法。main方法与其他普通方法的唯一区别在于：它是唯一可以被jvm识别的方法。当然，main方法也可以被其他类调用，此时它就是个普通静态方法。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;main方法可以出现在任何类中，只要你想将它作为程序的起点和入口；但每个jvm进程每次只能启动一个入口，如果项目中还有其他的类也有main方法，那么需要再启动一个jvm进程来执行它——两个进程之间是互不干扰的。