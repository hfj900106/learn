### JVM 内存模型（运行时数据区）
栈 （线程，栈帧，JVM会给每个线程在栈中开辟专属的内存，每个线程栈中还会开辟该线程方法的内存区-栈帧，存放局部变量，先调用的方法会先开辟内存，方法调用结束后释放，符合java数据结构栈的先进先出的特点）

程序计数器 （线程私有，记录指令码执行的当前行数，以防线程挂起又恢复之后不知从哪行开始）

本地方法栈 （线程私有，如果线程需要调用到native方法，那么也会开辟本地方法栈内存）

堆 （new出来的对象）

方法区（元空间）（存放 常量、静态变量（如果是new出来的对象，那么存储指向堆的指针）、类元信息（类的信息））

##### 栈帧
局部变量表（装方法的局部变量，也可以是指向堆内存对象的指针）  
操作数栈（局部变量在计算过程中的临时内存区，计算结果放入局部变量表，或者是返回给调用的方法）  
动态链表（）  
方法出口（记录方法被调用时的返回地址，从哪来回哪去）  


### CPU、主内存（RAM）、CPU缓存
CPU性能远超主内存，两者间若是直接交互，那么CPU的速度必定受主内存瓶颈影响，所以中间加了高速缓存，解决两者间性能差异导致的问题。

### JMM （java线程内存模型）
看图

### 内存模型交互
lock  unlock  read  load  use  assign  store  write 

### volatile
被修饰的变量具有两个特性：
1. 此变量对所有线程可见；
2. 禁止指令重排，保证对变量的赋值操作顺序和程序代码中的执行顺序一致；

注：不能保证原子性，必要时要加锁

volatile底层现在是借助《MEIS缓存一致性协议》实现的，CPU会开启对总线的的嗅探，当数据经过总线时，发现数据和CPU当前执行线程的工作内存变量相同但是值发生改变时，将工作内存的数据置失效，当执行引擎从工作内存中拿数据的时候发现失效，那么会再触发read操作。

volatile可见性的实现：CPU的lock指令，通过在写volatile变量的机器指令前加上lock前缀，使写volatile具有以下两个原则：  
  1. 写volatile时处理器会将缓存写回到主内存。  
  2. 一个处理器的缓存写回到内存会导致其他处理器的缓存失效。
  
  javap -c -s -v -l xxx.class 查看字节码

### 内存屏障


### JVM参数类型
1.标配参数（-version 等）
2.X参数（了解，非重点,-Xmixed 混合模式，JVM自己来决定是否编译成本地代码）

3.XX参数（重点）
  3.1 Boolean类型（用于开启或关闭某个属性）
  公式：-XX：+/-（具体属性） 
  eg：是否打印GC回收细节  -XX:+PrintGCDetails (开启)；-XX:-PrintGCDetails(关闭)
  eg：是否使用串行垃圾回收器  -XX:+UseSerialGC  ； -XX:-UserSerialGC
  
  3.2 KV类型(用于设置值)
  公式：-XX:key=value
  eg：-XX:MateSpaceSize=1024m (元空间大小赋值)
#### 两个经典参数 -Xms和-Xmx如何解释
-Xms等价于 -XX:InitailHeapSize 
-Xmx等价于 -XX:MaxHeapSize

#### 常用的JVM参数
-Xms 初始堆内存大小，默认物理机内存的1/64；
-Xmx 最大堆内存大小，默认物理机内存的1/4；
-Xss 等价于-XX:ThreadStackSize 每个线程内存大小，一般为512k-1024k；
-Xmn 年轻代大小，一般用默认
-XX:MateSpaceSize 元空间大小，元空间不在虚拟机，而是使用本地内存，但是初始分配的一般是不够的，我们可以修改为1024M左右
-XX:PrintGCDetails 输出详细的GC信息，GC、FullGC 哥哥参数可以参见百度，哈哈
-XX:SurvivorRatio 默认8，Eden：from：to=8:1:1，如果是4，那么eden:S0:S1=4:1:1
-XX:NewRatio 默认2， 老年代：新生代=2：1 ；=4说明新生代1，老年代4
-XX:MaxTenuringThreshold 设置在young区经过多少次GC才能到老年代，默认15，取值0-15；超过会报错；

#### 查看Java中正在运行的进程
jps -l
#### 查看java进程信息

jinfo -flag 属性名 pid（进程ID） 查看某个属性值

jinfo -flags  pid（进程ID）  查看所有属性值  

#### 查看java JVM初始化参数
java -XX:+PrintFlagsInitail
打印的值前面是 :=值  说是人为修改之后的
#### 查看 java JVM被修改之后的参数
java -XX:+PrintFlagsFinal 

#### 查看Java JVM参数配置信息命令
java -XX:+PrintCommandLineFlags（打印的配置最后一个是默认的垃圾回收机制）
这个参数让JVM打印出那些已经被用户或者JVM设置过的详细的XX参数的名称和值。
换句话说，它列举出 -XX:+PrintFlagsFinal的结果中第三列有":="的参数。以这种方式，我们可以用-XX:+PrintCommandLineFlags作为快捷方式来查看修改过的参数。

### 引用
#### 强引用
只要还有一个对象应用这个对象A，那么即使OOM了也不会回收A，我们通常用的new一个对象然后赋值给某个变量就是强引用；
eg：Student a = new Student();
#### 软引用
内存不足的才会用回收
Student a = new Student();
SoftReference b = new SoftReference(a);
a=null,要是内存够用时，那么b.get()还能拿到对象，要是不足的时候，就拿不到对象了，已经被回收；

#### 弱引用
不管内存够不够，只要GC就会被回收
Student a = new Student();
WeakReference b = new WeakReference(a);
a=null,GC之后b.get()获取不到对象，已经被回收了；

##### 强软弱引用适用各种场景
场景，每次都需要从本地读取大量的图片，占用内存较大，但是每次直接从硬盘读取浪费性能，可以用弱引用或者软引用
引伸 WeakHashMap

##### WeakHashMap
key表为null时，则原来对象被回收；

#### 虚引用（幽灵引用）
任何时候都有可能会被回收，必须要和ReferenceQueue联合使用（前面的引用也用，单是不是只能，回收之前都会放到引用队列，做完最后的事情后就销毁对象），get()方法永远返回null，作用：跟踪某对象呗垃圾回收的状态，确保这个对象在回收的时候会收到一个系统通知或者后续添加进一步操作

### OOM（out of memory）（错误，不是异常）
#### java.lang.StackOverflowError
jvm规定了栈的最大深度，当执行时栈的深度大于了规定的深度，就会抛出StackOverflowError错误；
一般是递归调用出问题；栈默认512k-1024k;

#### java.lang.OutOfMemoryError:java head space
对象占空间太多，撑爆了堆内存
eg:大对象，或者代码错误导致产生大量的对象；

#### java.lang.OutOfMemoryError:GC overhead limit exceeded
垃圾回收时间过长，超过98%的时间去回收垃圾，CPU使用率高，可是连续GC都达不到2%的回收内存效果；
eg：list加了超多元素；一个生产事故就是这个，一个没有加条件的SQL查询，结果加入list返回，可是这个表的数据有几百万，全都放到list里面，直接崩溃了；

#### java.lang.OutOfMemoryError:Direct buffer memory
直接内存（堆外内存）溢出；常见于NIO，因为会用到ByteBuffer，两种缓冲区对应的API如下：

JVM堆内存：ByteBuffer.allocate(size)  需要在native堆和java堆间来回拷贝，费性能
本地内存：ByteBuffer.allocateDirect(size) 不会GC，直接拷贝内存，速度快
弊端，没有GC去回收，那么可能导致本地内存被用完

#### java.lang.OutOfMemoryError:unable to create new native thread
多线程高并发的时候容易报，和平台有关有关，thread.start();start()调用的是native start0();Linux默认单进程最多可以创建1024个线程（理论个数，通常8、9百就开始报错了）；
解决办法：先确认是否单进程中需要这么多线程，若不是，修改代码将数据降到最低；若是，则修改Linux配置，扩大限制的数量；
eg:main方法for循环中创建线程
#### java.lang.OutOfMemoryError:Matespace
MateSpace在直接内存中，存放：加载的类的信息、常量池、静态变量、编译后的代码，空间被撑爆 就会报错；




### 类加载器

### 双亲委派

### 安全沙箱


























