###Java漫谈（一）

​    Java是什么？狭义的说，java是一门计算机编程语言。这句话包含两个要点：1、java是一门语言，这种语言可以被人类识别、书写；2、这门语言最终要应用到计算机上。关于第一点，java这门语言有着自己特殊的语法，非专业的人也许能看懂每个单词，但是单词连成一句话就不理解了，这需要训练；关于第二点，大家都知道，计算机只认识0和1，不认识人类的语言，那么java程序从源码到真正执行，中间肯定需要经过一系列的转换，而java也提供了配套的工具，比如编译器、JVM，这些工具也是java技术体系的不可或缺的一部分，它们的具体作用将在后文说明。 

​    Java的源代码保存在以.java为后缀的文件中，java的字节码保存在以.class为后缀的文件中。从工程的角度，java编译器的工作是把java源码文件转换成java字节码文件，即.java文件变成.class文件。 字节码文件可以说是java程序的最终形态或者可执行文件，虽然同样是二进制文件，但是class文件与传统的可执行文件还是不同的：传统的可执行文件，比如由c/c++编写的程序，可以由操作系统直接执行；但是class文件只能由java虚拟机来执行，不能由操作系统直接执行，这里操作系统可以理解为计算机本身。当然，java虚拟机也是一个程序，并且源码一般也是由c/c++编写的。那么java为什么要这么设计呢？java推广的时候有一个著名的口号：一次编写，到处运行。其实这句话准确的理解应该是“一次编译，到处运行”，也就是java的源代码被编译一次生成class文件后，这些class文件可以在任何安装了java虚拟机的机器上运行。作为对比，传统的程序比如c/c++编写的程序，并不能“一次编译，到处运行”，同一套源代码在IBM的机器编译出来的可执行程序，到了惠普的机器上也许根本跑不动，必须要重新编译一次。其中的原因在于不同机器的硬件架构以及机器指令等底层设计都不一样，所以不同机器上的编译器也要做对应的调整，将源代码编译成不同的二进制指令。而这显然是很麻烦的，想想吧，全世界有多少种不同的主流机器，c/c++的源代码就要编译多少个不同版本的可执行程序，而java拿到编译好的程序就可以执行，这种所谓跨平台的特性使其大受欢迎并迅速流行起来。不过呢，深究本质，java的虚拟机是要直接被机器执行的，所以虚拟机的源代码仍然是要被编译很多版本以适应不同的机器的，然而这件事由java的设计者操心就好了，使用java进行开发的人则完全不用关心，只需要在目标机器上安装好对应版本的虚拟机就可以进行开发了。说穿了，java之所以能够“一次编写，到处运行”，是因为java设计者精心准备了java虚拟机这个东西，包揽了面对各种机器多次编译的脏活累活。Java虚拟机则屏蔽了不同机器的底层细节，让所有的机器对java开发者来说都是一样的。

​    从上面的叙述中可以看出，java虚拟机对于java语言来说是至关重要的。但是虚拟机这个话题实在太大了，以后慢慢说，本篇的重点是java程序被虚拟机执行的前一个步骤——编译，也就是java源代码是如何被编译成class文件的。说到编译，离不开源代码和编译器，而这两个同样是非常大的话题，一篇文章根本说不完，好吧，本文就此结束。。