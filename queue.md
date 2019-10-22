### 队列

#### 阻塞队列（BlockingQueue）
1. ArrayBlockingQueue
2. LinkedBlockingQueue
3. SynchronousQueue

##### BlockingQueue 核心方法

add(e)  无法立即执行，抛一个异常
offer(e)  无法立即执行，返回一个特定的值(常常是 true / false)
put(e)  无法立即执行，该方法调用将会发生阻塞，直到能够执行
offer(e, time, unit)  无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)

remove()  无法立即执行，抛一个异常
poll()  无法立即执行，返回一个特定的值(常常是 true / false)
take()  无法立即执行，该方法调用将会发生阻塞，直到能够执行
poll(time, unit)  无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)

#### 非阻塞队列：
1. ConcurrentLinkedQueue

