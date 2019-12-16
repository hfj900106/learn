## 常用集合接口

常用接口有 List、Set、Queue
### List 
有序、可重复，常用实现类 ArrayList、LinkedList、Vector

1.ArrayList 
底层数据结构是数组，线程不安全，内存地址连续，删除容易造成内存碎片

2.Vector
底层数据结构是数组，线程安全， synchronized修饰

3.LinkedList
底层数据结构是双向链表，线程不安全，内存可以不连续，不会造成内存碎片，插入块，查询慢

4.CopyOnWriteArrayList
写时复制，线程安全


### Set
元素不能保证有序且不可重复的集合，常用实现类 HashSet、LinkedHashSet、TreeSet

1.HashSet
底层是HashMap对象，key不可以重复，value是一个Object对象，元素无序，线程不安全，key可以为 NULL

2.LinkedHashSet
HashSet的子类，底层是HashMap对象，不可重复，有序，线程不安全，使用HashSet的构造方法new LinkedHashMap()-》new HashMap()，Entry包含before, after属性，HashMap.Node subclass for normal LinkedHashMap entries

3.TreeSet
底层元素 NavigableMap接口对象; 无参构造方法默认实现是TreeMap，红黑树，元素有序，不可重复，线程不安全，擅长于范围查询

### Queue
 * @see PriorityQueue 
 * @see java.util.concurrent.LinkedBlockingQueue
 * @see java.util.concurrent.BlockingQueue
 * @see java.util.concurrent.ArrayBlockingQueue
 * @see java.util.concurrent.LinkedBlockingQueue
 * @see java.util.concurrent.PriorityBlockingQueue
 
#### 阻塞队列（BlockingQueue）
1. ArrayBlockingQueue
2. LinkedBlockingQueue
3. SynchronousQueue
线程池中有应用BlockingQueue

##### BlockingQueue 核心方法

add(e)  无法立即执行，抛一个异常
offer(e)  无法立即执行，返回一个特定的值(常常是 true / false)
put(e)  无法立即执行，该方法调用将会发生阻塞，直到能够执行
offer(e, time, unit)  无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)

remove()  无法立即执行，抛一个异常
poll()  无法立即执行，返回一个特定的值(常常是 true / false)
take()  无法立即执行，该方法调用将会发生阻塞，直到能够执行
poll(time, unit)  无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)

#### 非阻塞队列
1. ConcurrentLinkedQueue
