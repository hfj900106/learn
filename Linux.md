#### 常用命令
1. 查看内核版本
cat /proc/version
2. 查看发行版本
lsb_release -a   (lsb是linux standard base)
。。。

##### 生产的系统变慢，我们可以从哪方便分析

整机：  
top    
   主要看： loan average:1.51, 0.91, 0.42 （代表：系统1分钟、5分钟、15分钟的负载值，若三数之和平均值>0.6 说明系统负载高 ）
   其他：%CPU %MEM ，按1 CPUs->多个CPU罗列出来
   
   uptime（top的精简版，输出：11:17:47 up 104 days, 15:25,  1 user,  load average: 0.00, 0.01, 0.05）

CPU：  
总CPU：  
    vmstat -n n1 n2 （每n1秒打印CPU的采样数，采样n2次）  
    eg： vmstat -n 2 3 （2秒采样一次，一共采3次） 
        主要看：proc（主要 r-运行、b-阻塞） 、 cpu（主要 us-用户、sy-系统）
      proc  
        r（runtime） 运行和等待CPU时间片的进程数，原则1核的CPU上运行队列不要超过2，总的不要超过总核数的2倍，否则压力过大；
        b（blocking）阻塞等待资源的进程数，比如正在等待磁盘io、网络io等；
      cpu  
        us 用户进程消耗cpu占比，如果长期>50%,程序需要优化；
        sy 内核进程消耗cpu占比；
        us + sy > 80% ，可能cpu不足；   
        其他：id（idle-空闲） cpu 空闲率

单个CPU：
    mpstat -P ALL n1 n2 （每n1秒打印出每个CPU的采样数，采样n2次）
    
单个进程CPU使用量：
    ps -ef  查看进程    |grep XXX 用于筛选
    pidstat -u n1 n2 -p pid  每n1秒打印CPU的采样数，采样n2次,pid进程id
    
查看进程的各个线程堆栈信息（JVM性能调优中使用得非常多）：
    jstack  pid    
    
linux里查看最耗CPU的线程：    
   步骤1：top 查看最耗CPU的进程 pid
   步骤2：top -Hp  pid 查看该进程里的线程资源使用情况，找到最耗资源的线程的tid（显示pid）;可以使用ps -Lfp pid 或者 ps -mp pid -o THREAD, tid, time 或者 top -Hp pid;
   步骤3：jstack  pid |grep tid的16进制小写  -A打印前多少行  找到最耗资源的线程的栈信息
   
内存：free  默认单位 KB
    free -m  查看内存，单位MB（推荐）； -g 单位是GB；

硬盘：df -h  查看磁盘使用率，单位GB；支持 -m（MB）;-k（KB）
 
磁盘IO：iostat -xdk n1 n2  查看磁盘io  也可以用 pidstat 查看读写占比

网络IO：ifstat  

##### Java服务，内存OOM问题如何快速定位？
1. jmap -heap pid  查看进程的堆信息
    可以查看新生代，老生代堆内存的分配大小以及使用情况，看是否本身分配过小。
2. jmap -histo:live pid | more  找到最耗内存的对象
    输入命令后，会以表格的形式显示存活对象的信息，并按照所占内存大小排序：
    instant 实例数
    bytes 所占内存大小
    class name 类名  
如果发现某类对象占用内存很大（例如几个G），很可能是类对象创建太多，且一直未释放。例如：
申请完资源后，未调用close()或dispose()释放资源
消费者消费速度慢（或停止消费了），而生产者不断往队列中投递任务，导致队列中任务累积过多；

3. 确认是否是资源耗尽
/proc/${PID}/fd | wc -l 查看句柄数
/proc/${PID}/task | wc -l 查看线程数

##### 生成 Heap Dump（堆转储文件）文件 的几种方式
Heap Dump 是 Java进程所使用的内存情况在某一时间的一次快照。以文件的形式持久化到磁盘中，不同格式存储内容不同，当你在执行一个转储操作时，往往会触发一次GC，所以你转储得到的文件里包含的信息通常是有效的内容（包含比较少，或没有垃圾对象了） 。

Heap Dump 包含的信息
    所有的对象信息 对象的类信息、字段信息、原生值(int, long等)及引用值
    所有的类信息 类加载器、类名、超类及静态字段
    垃圾回收的根对象 根对象是指那些可以直接被虚拟机触及的对象
    线程栈及局部变量 包含了转储时刻的线程调用栈信息和栈帧中的局部变量信息

1. 使用 jmap 命令生成 dump 文件  
    jmap -dump:live,format=b,file=d:\dump\heap.hprof <pid>

2. 使用 jcmd 命令生成 dump 文件  
    jcmd <pid> GC.heap_dump d:\dump\heap.hprof（服务器具体地址）  

3. 使用 JVM 参数获取 dump 文件
     -XX:+HeapDumpOnOutOfMemoryError  当OutOfMemoryError发生时自动生成 Heap Dump 文件，一个非常有用的参数   
     -XX:+HeapDumpBeforeFullGC    当 JVM 执行 FullGC 前执行 dump
     -XX:+HeapDumpAfterFullGC    当 JVM 执行 FullGC 后执行 dump
     -XX:HeapDumpPath=d:\test.hprof   指定 dump 文件存储路径  
     
##### 线上配置
-Djava.awt.headless=true 
-Djava.net.preferIPv4Stack=true 
-Dspring.profiles.active=prod 
-Dserver.port=9201 
-javaagent:/data/pp-agent-1.8.3/pinpoint-bootstrap-1.8.3.jar 
-Dpinpoint.applicationName=loan-api 
-Dpinpoint.agentId=api01 
-server -Xms8g -Xmx8g -Xmn2048m -Xss256k 
-XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled 
-XX:LargePageSizeInBytes=128m -XX:MaxPermSize=512m -XX:PermSize=256M 
-XX:+UseFastAccessorMethods 
-XX:+UseCMSInitiatingOccupancyOnly 
-XX:CMSInitiatingOccupancyFraction=70 
-XX:+PrintGC -XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-Xloggc:/tmp/jvm.log 
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath=/tmp/heapdump.hprof 
-jar /www/wwwroot/loan/loan-api/loan-api-2.0.0.jar     
     