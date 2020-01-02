## GC 
（A）、Minor GC
       又称新生代GC，指发生在新生代的垃圾收集动作；
       因为Java对象大多是朝生夕灭，所以Minor GC非常频繁，一般回收速度也比较快；  
       特殊：老年代可能存在对新生代对象的引用，为了避免Minor GC对老年代的全扫描，java虚拟机引入了“卡表”技术，大概记录了老年代中可能存在引用新生代的内存区域；

（B）、Full GC
       又称Major GC或老年代GC，指发生在老年代的GC；
       出现Full GC经常会伴随至少一次的Minor GC（不是绝对，Parallel Sacvenge收集器就可以选择设置Major GC策略）；
 Major GC速度一般比Minor GC慢10倍以上；
 
### 垃圾回收算法

1.引用计数  
  只要对象的引用计数器的值为0，则立刻回收。每次对对象赋值需要维护计数器，而且计数器本身有一定损耗，比较难处理循环引用的问题，一般不用；  
2.复制  
  内存分区（开始是对半分，后来优化8:1:1），其中一个区被占用完之后将本区中仍然存活的对象复制到另一个区，如此循环；  
  新生代很多对象都是朝生夕死，所以没必要留50%给存活区，HotSpot默认的Eden和Survivor比例是8:1:1，就是说，每次能使用90%的内存容量。当然，也可能会出现剩余10%的Survivor空间不够复制原有存活对象的情况，那就需要依赖其它内存（这里指老年代）进行分配担保（Handle Promotion)。通过分配担保机制，这些对象会直接进入老年代。  
3.标记-清除（Mark-Sweep）   
  3.1 标记阶段：找到所有可访问的对象，做个标记  
  3.2 清除阶段：遍历堆，把未被标记的对象回收    
  为了能够区分对象是live的,可以为每个对象添加一个marked字段，该字段在对象创建的时候，默认值是false，被标记后未true，被清理之后修改为false。  
  优点：可以解决循环引用的问题，内存不足时才回收。  
  缺点：回收是STW（应用线程挂起）；效率低，尤其是对象很多的时候；造成内存碎片（由于内存空间不连续，这样给大对象分配内存的时候可能会提前触发full gc）；
  
4.标记-压缩（Mark-Compact） 
这种算法和标记-清除算法很像，唯一不同的是，当标记完成后，不是清理掉需要回收的对象，而是将所有存活的对象向一端移动，然后将边界以外的内存全部清理掉，这样可以有效避免空间碎片的产生。
缺点：压缩算法的消耗；
### 空间分配担保
在发生Minor GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间。如果这个条件成立，那么Minor GC可以确保是安全的。如果不成立，则虚拟机会查看HandlerPromotionFailure设置是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小。如果大于，将尝试着进行一次Monitor GC，尽管这次GC是有风险的。如果小于，或者HandlerPromotionFailure设置不允许冒险，那这时也要改为进行一次Full GC了。
上述所说的冒险到底是冒的什么险呢？
前面提到过，新生代使用复制收集算法，但是为了内存利用率。只使用其中一个Survivor空间来作为轮换备份，因此当出现大量对象在Minor GC后仍然存活的情况（最极端的情况是内存回收之后，新生代中所有的对象都存活），就需要老年代进行分配担保，把Survivor无法容纳的对象直接进入老年代。老年代要进行这样的担保，前提是老年代本身还有容纳这些对象的剩余空间，一共有多少对象存活下来在实际完成内存回收之前是无法明确知道的，所以只好取之前每一次回收晋升到老年代对象容量的平均大小值作为经验值，与老年代的剩余空间进行比较，决定是否进行Full GC来让老年代腾出更多空间。
取平均值进行比较其实仍然是一种动态概率的手段，也就是说，如果某次Minor GC存活后的对象突增，远远高于平均值的话，依然会导致担保失败。如果出现HandlerPromotionFailure失败，那就只好在失败后重新发起一次FULL GC。虽然担保失败时绕的圈子是最大的，但大部分情况下都还是将HandlerPromotionFailure开关打开，避免Full GC过于频繁。

### 垃圾回收的时候如何确定对象是垃圾，什么是GC Roots，哪些对象可以作为GC Roots？
可作为GC Roots的对象有：
1. 虚拟机栈中引用的对象；
2. 方法区中的类静态属性引用对象；
3. 方法区中常量引用的对象；
4. 本地方法栈中（native方法）引用的对象；

判断方法：枚举根节点可达性分析
从GC Roots对象开始往下查找，在同一条链路上的称为可达，不在的不可达，将会被垃圾回收


### 垃圾收集器

#### 新生代：Serial、ParNew、Parallel Scavenge
全部是复制算法

#### 老年代：Serial Old、Parallel Old、CMS（Concurrent Mark and Sweep）
Serial Old、Parallel Old 用 标记-压缩 算法；CMS用 标记-清除 算法；

#### 全能：G1

标记-清除 会产生内存碎片，其他的都不会；
其中Serial Old作为CMS出现"Concurrent Mode Failure"失败的后备预案


#### ParNew  搭配  CMS
停顿时间短
"-XX:+UseConcMarkSweepGC"：指定old区使用CMS，会默认使用ParNew作为新生代收集器；  
 
#### Parallel Scavenge  搭配 Parallel Old
目标 最大吞吐量
jdk1.8 默认的收集器。
  
系统的吞吐量 = 用户代码运行时间 / (用户代码运行时间 + 垃圾收集时间)。    

设置参数：可以相互激活  
-XX:+UseParallelGC / -XX:+UseParallelOldGC：新生代（ParallelGC），老年代（parallel old ），都用并行；
-XX:ParallelGCThreads：指定垃圾收集的线程数量；  

#### CMS 搭配 ParNew
目标 最短回收停顿时间，适用于要求服务响应速度较快的应用上。

-XX:+UseCMSCompactAtFullCollection  开启后 Full GC之后进行碎片整理；
-XX:CMSFullGCsBeforeCompaction  设置在执行多少次不压缩的Full GC后, 进行一次带碎片整理的Full GC；

过程：
1.初始标记；2.并发标记；3.重新标记；4.并发清除
第1、3步会STW（停顿整个应用）

 具体：
 初始标记 Initial Mark
 STW，扫描GC roots 能够直接达到的对象。
 
 并发标记 Concurrent Mark
 并发过程，应用线程重新启动，从上一步标记出的活对象开始进行跟踪判断可达。
 
 预清理阶段 Preclean
 并发过程，在上一步的并发标记过程中，应用线程也一直在运行，因此可能有对象引用状态的修改。CMS需要找到所有这些修改来修正标记。预清理过程寻找并记录下这些改变。
 
 重新标记 remark phase
 STW过程，如果应用一直修改对象引用状态，CMS不能准确地确定哪个对象是活的。所以CMS需要停止应用线程来追上之前的这些修改。
 
 并发清除 sweep phase
 并发过程，CMS将之前标记死掉的对象的空间放到空闲链表中。
 
 重置 reset phase
 并发过程，CMS清理它的状态，准备下一次收集。
 
 CMS不进行压缩,不移动对象，使用空闲链表记录空闲内存。所以存在内存碎片，可以设置参数来控制多少次后进行一次压缩，另外由于并发收集，可能导致应用程序产生垃圾的速度比收集的要快，这时造成Concurrent Mode Failure会降级到Serial Old进行STW的回收，引发比较久的应用停顿，可以设置CMS开始触发的老年代使用内存比例。
 
 
###### 重点 G1 （Garbage Frist）
目标：实现可预期、可配置的STW停顿时间；  
JDK 1.9 默认的收集器；  

相比CMS收集器, G1收集器有两个改进点  
1.G1基于标记-整理算法, 不会产生空间碎片  
2.G1可以设置应用停顿时间  
命令：  
-XX:+UseG1GC（The G1 garbage collector is fully supported in Oracle JDK 7 update 4 and later releases）  
常见调优：  
-XX:MaxGCPauseMillis=200 （最大停顿时间200ms默认）；  
-XX:InitiatingHeapOccupancyPercent=45 （开始第一个标记周期的堆占比，默认45%）；  
-XX:G1HeapRegionSize=n （设置每个Region的大小，这里需要是1MB到32MB的2的指数的大小）；  
-XX:ConcGCThreads （垃圾收集线程个数，和CPU数量有关，CPU小于8的最多可以配置8；超过8的，可配置n * 5/8）；  
-XX:G1ReservePercent  （预留内存，默认值是10，代表使用10%的堆内存为预留内存，当Survivor区域没有足够空间容纳新晋升对象时会尝试使用预留内存）；  

原理：大小相等的region（1-32M，2的指数），逻辑上连续，小块标记清理

Region：  
为实现大内存空间的低停顿时间的回收，将划分为多个大小相等的Region。每个小堆区都可能是 Eden区，Survivor区或者Old区，但是在同一时刻只能属于某个代
       在逻辑上, 所有的Eden区和Survivor区合起来就是新生代，所有的Old区合起来就是老年代，且新生代和老年代各自的内存Region区域由G1自动控制，不断变动
堆空间region：  
Eden、Survivor、old、Humongous regions（多个连续的region组成，存放巨型对象(Humongous Object)）；

巨型对象：  
当对象大小超过Region的一半，则认为是巨型对象(Humongous Object)，直接被分配到老年代的巨型对象区(Humongous regions)，这些巨型区域是一个连续的区域集，每一个Region中最多有一个巨型对象，巨型对象可以占多个Region

划分region的意义：  
1.避免全堆扫描  
2.GC时间可控

https://zhuanlan.zhihu.com/p/22591838
