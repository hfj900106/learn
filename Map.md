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