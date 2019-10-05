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

### 类加载器

### 双亲委派

### 安全沙箱

### GC 

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
