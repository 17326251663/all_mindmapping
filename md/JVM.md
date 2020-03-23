# 	JVM

##  1.什么是jvm?

java虚拟机 (java [virtual](javascript:void(0);) [machine](javascript:void(0);)),

jvm是运行在操作系统上的,它与硬件没有直接的交互.

class文件的运行过程:

![1578461485032](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1578461485032.png)

灰色的矩形块,代表属于线程私有,内存占用很小.



# 2.类装载器ClassLoaader

概念:

负责加载class文件,class文件在<font color='red'>文件开头有特殊的文件标识</font>,将class文件字节码内容加载到内存中,

并将这些内容转换为方法区中的运行时数据结构并且ClassLoader只负责class文件的加载.至于它

是否可以运行,则有Execution Engine(执行引擎)决定.



结构图:

![1578483284058](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1578483284058.png)



![1581497322983](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581497322983.png)

ClassLoader的双亲委派机制和沙箱安全机制.

![1581500297045](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581500297045.png)



# 3.执行引擎

![1581500408637](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581500408637.png)



# 4.本地方法栈,本地方法接口,本地方法库

![1581501521849](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581501521849.png)

# 5.pc寄存器

概念: 每个线程都有一个程序计数器,是线程私有的,就是一个指针.指向方法区中的方法字节码(用来存储指向下一条指令的地址,也即将要执行的代码指令),由执行引擎读取下一条指令,是一个非常小的内存空间,可以忽略不计.

这块内存区域很小,它是当前线程所执行的字节码的行号指示器,字节码解释器通过改变这个计数器的值来选取下一条需要执行的字节码指令.



如果执行的是一个Native方法 ,那这个计数器就是空的.



用于完成分支,循环,跳转,异常处理,线程恢复等基础功能.不会发生oom(OutOfMemory)错误.

#  6.方法区 Method Area

供各个线程共享的内存区域.<font color='red'>它存储了每一个类的结构信息</font>,例如运行时常量池(Runtime Constant Pool),字段和方法数据,构造方法和 普通方法的字节码内容.上面讲的是规范,在不同的虚拟机里头实现是不一样的,最典型的就是永久代(PerGen space jdk1.8之前) 和 元空间(Metaspace)



# 7.栈 stack

栈管运行,堆管存储.

调用方法将在栈内存中开辟空间,成为入栈(压栈).

栈内存存放基本数据类型和引用数据类型的地址值.

栈内存中数据,没有默认的初始化值,需要手动设置.

方法调用完毕,栈内存立即释放,成为出栈(弹栈)

先进后出.

栈也叫栈内存,主管Java程序的运行.是在线程创建时创建,它的声明周期跟随线程的生命周期,线程结束释放也就释放,<font color='blue'>对栈来说不存在垃圾回收问题</font>,只要该线程一结束该栈就over,生命周期和线程是一致的,是线程私有的.

<font color='red'>8种基本类型的变量和对象的引用变量和实例方法都是在栈内存中完成分配的</font>



<font color='red'>栈存储什么?</font>

栈帧中主要保存3类数据:

本地变量(Local Variables): 输入参数和输出参数以及方法内变量

栈操作(Operand Stack): 记录出栈,入栈的操作.

栈帧(压到栈中的方法)数据: 包括类文件,方法等. 

![1581576953398](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581576953398.png)

![.1581577042414](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1581577042414.png)



# 8.栈 堆 方法区 的交互关系



当一个栈帧创建一个引用对象时,栈会存放引用变量的地址值指向堆中的对象,方法区中存放对象的类信息(模版).



# 9.堆(heap)

堆=>(

1.新生代 - > (1.伊甸区,2.幸存者0区,3.幸存者1区)

2.永久代 - > ()

3.元空间 - > ()

)

![1582157381162](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582157381162.png)



## 堆体系结构

heap 堆

一个JVM实例只存在一个堆内存,堆内存的大小是可以调节的.

类加载器读取了类文件后,需要把类,方法 , 常变量放到堆内存中,保存所有引用类型的真实信息,以方便执行器执行.

堆内存分为3个部分:

. Young Generation Space  新生区

. Tenure generation space 养老区

. Permanent Space 永久区

S0 (幸存者0区,别名) from

 S1 (幸存者1区,别名) to

当伊甸区的内存占用达到98%左右时,Eden Space会触发YGC(轻量级GC)进行清理,存活下来的对象会被移入幸存者0区.

当伊甸区内存再次到达阈值时,会再次进行清理,活下来的对象会进行区<font color='red'>交换</font>: 伊甸区=>幸存者0区, 幸存者0区 =>幸存者1区,多次清理(15次)依然活下来的对象 : 幸存者1区=>养老区.

如果养老区内存到达阈值后,会开启FGC(Full GC 重GC).进行养老区清理,full GC 多次后发现养老区空间 没办法腾出来,会报出OOM(Out Of Memory Error),堆内存溢出

![1582161571092](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582161571092.png)



## 对象的生命周期和GC

from区和to区,他们的位置和名分,不是固定的,每次GC后会交换

GC之后有交换,谁空谁是to.(这涉及到回收算法)



![1582161836571](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582161836571.png)



## 永久代

![1582164618224](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582164618224.png)

![1582164789407](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582164789407.png)

![1582165112572](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582165112572.png)

##  堆参数调优

![1582197418942](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582197418942.png)

![1582197518869](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1582197518869.png)



在java8 中,永久代已经被移除,被一个成为元空间的区域所取代.

元空间的本质与永久代类似.

元空间与永久代最大的区别在于:

永久代使用J的VM的堆内存,但是java8之后元空间并不在虚拟机中而使用本机物理内存.

因此,默认的情况下,元空间的大小仅受本地内存限制.类的元数据放入native memory中,字符串常量池和类的静态

变量放入java堆内存中,这样可以加载多少类的元数据就不再由MaxPermSize控制,而是由系统的实际可用空间来控

制(1/4).

### 堆内存调优简介01

| -Xms                | 设置初始分配大小,默认为物理内存的"1/64" |
| ------------------- | --------------------------------------- |
| -Xmx                | 最大分配内存,默认为物理内存的的"1/4"    |
| -XX:+PrintGCDetails | 输出详细的GC处理日志                    |

````java
   
//-Xms1024m -Xmx1024m -XX:+PrintGCDetails

public static void main(String[] args) {
        //获取主机核数
        System.out.println(Runtime.getRuntime().availableProcessors());
        //返回Java虚拟机试图使用的最大内存量
        System.out.println(Runtime.getRuntime().maxMemory());
        //返回Java虚拟机中的内存总量
        System.out.println(Runtime.getRuntime().totalMemory());
    }
````

### GC收集日志信息

\[DefNew: 2629K->2629K(3072K), 0.0000259 secs][Tenured: 5733K->1226K(6848K), 0.0013100 secs] 8362K->1226K(9920K), [Metaspace: 233K->233K(4480K)], 0.0013704 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

:

\[新生区:Minor GC前内存占用->Minor GC后内存占用(新生区总内存),耗时][养老区:full GC前内存占用,full GC后内存占用(养老区总内存),耗时]GC前堆内存占用->GC后堆内存占用(堆内存大小)[方法区:GC前内存占用->GC后内存占用(方法区总内存),耗时]\[时间: 用户损耗,系统损耗,真实耗时]



# GC

## GC是什么(分代收集算法)?

次数上频繁收集Young区

次数上较少收集Old区

基本不动元空间



## GC 4大算法

GC算法整体概述

![1584610499776](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1584610499776.png)



4大算法:

1.引用计数法

![1584611444200](C:\Users\喜静\AppData\Roaming\Typora\typora-user-images\1584611444200.png)

一个对象被三个地方引用,就计为3,每多一次引用,计数就+1.

每少一次引用计数就减一,当计数为0时,GC回收时就会回收.

缺点:

每次对对象赋值时均要维护引用计数器,且计数器本身也有一定的消耗.

较难处理循环引用（a=b;b=a;a=null;b=null;无法被GC回收）.

**JVM的实现一般不采用这种方式.**



2.复制算法

参照上方对象的生命周期，存在于新生代，新生代GC算法

-XX:MaxTenuringThreshold - 设置对象在新生代中存活的次数（计数多少次进入老年代）

3.标记清除

4.标记压缩



