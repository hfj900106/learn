## 常用集合接口
##### List接口：一个有序、可以重复的集合，常用实现类ArrayList和LinkedList
```java
1 // 底层数据结构是数组，查询快，增删慢，线程不安全，效率高
2 List arrayList = new ArrayList(); 
3 // 底层数据结构是数组，查询快，增删慢，线程安全，效率低，耗性能
4 List vector = new Vector();
5 // 底层数据结构是链表，查询慢，增删快，线程不安全，效率高
6 List linkedList = new LinkedList();
```
##### Set接口：元素不能保证有序且不可重复的集合，常用实现类HashSet、LinkedHashSet、TreeSet
```java
1 // 元素无序，不可重复，线程不安全，集合元素可以为 NULL
2 Set hashSet = new HashSet();
3 // 底层采用链表和哈希表的算法，保证元素有序，唯一性（即不可以重复，有序），线程不安全
4 Set linkedHashSet = new LinkedHashSet();
5 // 底层使用红黑树算法，擅长于范围查询，元素有序，不可重复，线程不安全
6 Set treeSet = new TreeSet();
```
##### Map接口：key-value的键值对，key不允许重复，value可以，key-value通过映射关系关联，常用实现类HashMap和TreeMap
```java
1 // 采用哈希表算法，key无序且不允许重复，key判断重复的标准是：key1和key2是否equals为true，并且hashCode相等 
2 Map<String, String> hashMap = new HashMap<String, String>();
3 // 采用红黑树算法，key有序且不允许重复，key判断重复的标准是：compareTo或compare返回值是否为0
4 Map<String, String> treeMap = new TreeMap<String, String>();
```
