##### 线程池4种拒绝策略
1. Abortpolicy(默认)：直接抛异常（RejectedExecutionException）
2. CallerRunsPolicy：不抛弃任务也不抛异常，而是把任务回退给调用者，降低任务流量
3. DiscardOldestPolicy：丢弃丢列中等待最久的任务尝试加入当前的任务
4. DiscardPolicy：直接丢弃

##### 线程池的创建
不可以在代码中显示创建线程，必须通过线程池获取
1. Executors 工具类获取（不推荐，因为不定指定合理的maximumPoolSize或者阻塞队列的大小，而是用Integer.MAX_VALUE，高并发环境下可能导致OOM ）；
2. 自己创建线程池 

##### ThreadPoolExecutor 参数

ThreadPoolExecutor(
                    int corePoolSize, 
                    int maximumPoolSize,
                    long keepAliveTime,
                    TimeUnit unit,
                    BlockingQueue<Runnable> workQueue,
                    ThreadFactory threadFactory,
                    RejectedExecutionHandler handler
                    )                        

##### 自建线程池怎么合理配置线程数

分析下应用是属于CPU密集型还是IO密集型，了解服务器的核数（Runtime.getRuntime.availableProcessor()）
1. CPU 密集型（需要大量运算，没有阻塞，或者阻塞极少）
maximumPoolSize = 核数 + 1

2. IO 密集型（大量的核数据库交互，阻塞较多）
maximumPoolSize = 核数 * 2  或者  核数/(1-阻塞系数) 
阻塞系数通常是0.8-0.9
根据生产实际情况选择配置方式






