#### Map接口：key-value的键值对，key不允许重复，value可以，key-value通过映射关系关联，常用实现类HashMap、TreeMap、HashTable
1.HashMap

2.TreeMap

3.HashTable

4.ConcurrentHashMap

1、HashMap是非线程安全的，HashTable是线程安全的。
2、HashMap的键和值都允许有null值存在，而HashTable都不行。
3、因为线程安全的问题，HashMap效率比HashTable的要高。
4、Hashtable是同步的，而HashMap不是。因此，HashMap更适合于单线程环境，而Hashtable适合于多线程环境。
    一般现在不建议用HashTable, ①是HashTable是遗留类，内部实现很多没优化和冗余。②即使在多线程环境下，现在也有同步的ConcurrentHashMap替代，没有必要因为是多线程而用HashTable。

1 // 采用哈希表算法，key无序且不允许重复，key判断重复的标准是：key1和key2是否equals为true，并且hashCode相等 
2 Map<String, String> hashMap = new HashMap<String, String>();
3 // 采用红黑树算法，key有序且不允许重复，key判断重复的标准是：compareTo或compare返回值是否为0
4 Map<String, String> treeMap = new TreeMap<String, String>();


####  HashMap 什么情况下链表转红黑树
1. 链表转红黑树阈值是8，但是实际链表长度=9；
2. 数组容量>64；如果数组容量<64,优先扩容；

#### ConcurrentHashMap

1）ConcurrentHashMap在JDK8里结构
    采用数组+链表+红黑树
2）ConcurrentHashMap的put方法、szie方法等
3）ConcurrentHashMap的扩容
4）HashMap、Hashtable、ConccurentHashMap三者的区别
5）ConcurrentHashMap在JDK7和JDK8的区别
6) 如何实现线程安全的？
   抛弃了原有的Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性

##### put 方法 
1、判断Node[]数组是否初始化，没有则进行初始化操作；  
2、通过hash定位数组的索引坐标，是否有Node节点，如果没有则使用CAS进行添加（链表的头节点），添加失败则进入下次循环；  
3、检查到内部正在扩容，就帮助它一块扩容；  
4、如果Node[]数组对应位置Node f不为空，则使用synchronized锁住f元素（链表/红黑树的头元素）。如果是Node（链表结构）则执行链表的添加操作；如果是TreeNode（树型结构）则执行树添加操作；  
5、判断链表长度已经达到临界值8（默认值），并且数组容量>=64(小的话会先扩容，不转树)，当节点超过这个值就需要把链表转换为树结构；  
6、如果添加成功就调用addCount（）方法统计size，并且检查是否需要扩容（先添加再检查扩容）；  

##### initTable 方法 
sizeCtl：表初始化和大小调整控制。如果为负，则表示Node[]正在初始化或调整大小。  
-1：初始化；-n（调整大小线程数=n-1）；当table为null时，sizeCtl=初始表大小（带参构造方法指定），或者默认为0（无参构造方法）；初始化后，sizeCtl=调整表大小的 下一个 元素计数值
 1. 保证只有一个线程正在进行初始化操作；
 
##### size
 ConcurrentHashMap 来说，这个 table 里到底装了多少东西其实是个不确定的数量，是个估计值。 
 
