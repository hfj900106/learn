### JVM 内存结构

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
###### 两个经典参数 -Xms和-Xmx如何解释
-Xms等价于 -XX:InitailHeapSize 
-Xmx等价于 -XX:MaxHeapSize

###### 常用的JVM参数
-Xms 初始堆内存大小，默认物理机内存的1/64；
-Xmx 最大堆内存大小，默认物理机内存的1/4；
-Xss 等价于-XX:ThreadStackSize 每个线程内存大小，一般为512k-1024k；
-Xmn 年轻代大小，一般用默认
-XX:MateSpaceSize 元空间大小，元空间不在虚拟机，而是使用本地内存，但是初始分配的一般是不够的，我们可以修改为1024M左右
-XX:PrintGCDetails 输出详细的GC信息，GC、FullGC 哥哥参数可以参见百度，哈哈
-XX:SurvivorRatio 默认8，Eden：from：to=8:1:1，如果是4，那么eden:S0:S1=4:1:1
-XX:NewRatio 默认2， 老年代：新生代=2：1 ；=4说明新生代1，老年代4
-XX:MaxTenuringThreshold 设置在young区经过多少次GC才能到老年代，默认15，取值0-15；超过会报错；

###### 查看Java中正在运行的进程
jps -l
###### 查看java进程信息

jinfo -flag 属性名 pid（进程ID） 查看某个属性值

jinfo -flags  pid（进程ID）  查看所有属性值  

###### 查看java JVM初始化参数
java -XX:+PrintFlagsInitail
打印的值前面是 :=值  说是是人为修改之后的
###### 查看 java JVM被修改之后的参数
java -XX:+PrintFlagsFinal 

###### 查看Java JVM参数配置信息命令

java -XX:+PrintCommandLineFlags（打印的配置最后一个是默认的垃圾回收机制）

### 引用
###### 强引用
只要还有一个对象应用这个对象A，那么即使OOM了也不会回收A，我们通常用的new一个对象然后赋值给某个变量就是强引用；
eg：Student a = new Student();
###### 软引用
内存不足的才会用回收
Student a = new Student();
SoftReference b = new SoftReference(a);
a=null,要是内存够用时，那么b.get()还能拿到对象，要是不足的时候，就拿不到对象了，已经被回收；

###### 弱引用
不管内存够不够，只要GC就会被回收
Student a = new Student();
WeakReference b = new WeakReference(a);
a=null,GC之后b.get()获取不到对象，已经被回收了；

###### 强软弱引用适用各种场景
场景，每次都需要从本地读取大量的图片，占用内存较大，但是每次直接从硬盘读取浪费性能，可以用弱引用或者软引用
引伸 WeakHashMap

###### WeakHashMap
key表为null时，则原来对象被回收；

###### 虚引用（幽灵引用）
任何时候都有可能会被回收，必须要和ReferenceQueue联合使用（前面的引用也用，单是不是只能，回收之前都会放到引用队列，做完最后的事情后就销毁对象），get()方法永远返回null，作用：跟踪某对象呗垃圾回收的状态，确保这个对象在回收的时候会收到一个系统通知或者后续添加进一步操作

### OOM（out of memory）（错误，不是异常）
###### java.lang.StackOverflowError
jvm规定了栈的最大深度，当执行时栈的深度大于了规定的深度，就会抛出StackOverflowError错误；
一般是递归调用出问题；栈默认512k-1024k;

###### java.lang.OutOfMemoryError:java head space
对象占空间太多，撑爆了堆内存
eg:大对象，或者代码错误导致产生大量的对象；

###### java.lang.OutOfMemoryError:GC overhead limit exceeded
垃圾回收时间过长，超过98%的时间去回收垃圾，CPU使用率高，可是连续GC都达不到2%的回收内存效果；
eg：list加了超多元素；一个生产事故就是这个，一个没有加条件的SQL查询，结果加入list返回，可是这个表的数据有几百万，全都放到list里面，直接崩溃了；

###### java.lang.OutOfMemoryError:Direct buffer memory
直接内存（堆外内存）溢出；常见于NIO，因为会用到ByteBuffer，两种缓冲区对应的API如下：

JVM堆内存：ByteBuffer.allocate(size)  需要在native堆和java堆间来回拷贝，费性能
本地内存：ByteBuffer.allocateDirect(size) 不会GC，直接拷贝内存，速度快
弊端，没有GC去回收，那么可能导致本地内存被用完

###### java.lang.OutOfMemoryError:unable to create new native thread
多线程高并发的时候容易报，和平台有关有关，thread.start();start()调用的是native start0();Linux默认单进程最多可以创建1024个线程（理论个数，通常8、9百就开始报错了）；
解决办法：先确认是否单进程中需要这么多线程，若不是，修改代码将数据降到最低；若是，则修改Linux配置，扩大限制的数量；
eg:main方法for循环中创建线程
###### java.lang.OutOfMemoryError:Matespace
MateSpace在直接内存中，存放：加载的类的信息、常量池、静态变量、编译后的代码，空间被撑爆 就会报错；




### 类加载器

### 双亲委派

### 安全沙箱

### GC 
（A）、Minor GC
       又称新生代GC，指发生在新生代的垃圾收集动作；
       因为Java对象大多是朝生夕灭，所以Minor GC非常频繁，一般回收速度也比较快；
（B）、Full GC
       又称Major GC或老年代GC，指发生在老年代的GC；
       出现Full GC经常会伴随至少一次的Minor GC（不是绝对，Parallel Sacvenge收集器就可以选择设置Major GC策略）；
 Major GC速度一般比Minor GC慢10倍以上；
 
### 垃圾回收算法
常用4中
1.引用计数
  每次对对象赋值需要维护计数器，而且计数器本身有一定损耗，比较难处理循环引用的问题，一般不用；
2.复制
  
3.标记清除
4.标记整理

### 垃圾回收的时候如何确定对象是垃圾，什么是GC Roots，哪些对象可以作为GC Roots？
可作为GC Roots的对象有：
1. 虚拟机栈中引用的对象；
2. 方法区中的类静态属性引用对象；
3. 方法区中常量引用的对象；
4. 本地方法栈中（native方法）引用的对象；

判断方法：枚举根节点可达性分析
从GC Roots对象开始往下查找，在同一条链路上的称为可达，不在的不可达，将会被垃圾回收


### 垃圾回收器
目前7种
可搭配使用：Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1；其中Serial Old作为CMS出现"Concurrent Mode Failure"失败的后备预案

###### 新生代：Serial、ParNew、Parallel Scavenge

###### 老年代：Serial Old、Parallel Old、CMS（Concurrent Mark and Sweep）

###### 老少通吃：G1

###### Serial（串行收集器）
新生代，复制算法，单线程，GC停顿，Client模式下默认的新生代收集器（现在一般不用Client模式）
命令：-XX:+UseSerialGC   

###### ParNew(并行收集器)
是Serial收集器的多线程版本，GC停顿比串行短
应用场景
      在Server模式下，ParNew收集器是一个非常重要的收集器，因为除Serial外，目前只有它能与CMS收集器配合工作；
      但在单个CPU环境中，不会比Serail收集器有更好的效果，因为存在线程交互开销。
设置参数
      "-XX:+UseConcMarkSweepGC"：指定使用CMS后，会默认使用ParNew作为新生代收集器；
      "-XX:+UseParNewGC"：强制指定使用ParNew；    
      "-XX:ParallelGCThreads"：指定垃圾收集的线程数量，ParNew默认开启的收集线程与CPU的数量相同；
      
###### Parallel Scavenge       
与吞吐量关系密切，也称为吞吐量收集器（Throughput Collector），jdk1.8默认的收集器，系统的吞吐量=用户代码运行时间/(用户代码运行时间+垃圾收集时间)，用于度量系统的运行效率。
新生代，复制，多线程，目标

设置参数：
-XX:+UseParallelGC






























