### 队列
非阻塞队列：ConcurrentLinkedQueue(无界线程安全)，采用CAS机制（compareAndSwapObject原子操作）；  实现并发安全无非两种方式，一种是锁，就像我们之前分析的容器，比如 concurrentHashMap，CopyOnWriteArrayList ， LinkedBolckingQueue，还有一种是 CAS，在这些容器里也用到了。


阻塞队列：ArrayBlockingQueue(有界)、LinkedBlockingQueue（无界）、DelayQueue、PriorityBlockingQueue，采用锁机制；使用 ReentrantLock 锁。
