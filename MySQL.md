

### mysql相关问题 WAL机制、crash safe如何实现、redo log作用 

WAL （Write-Ahead Logging） ，关键点是：先写日志在写磁盘

当有一条记录需要更新的时候：

InnoDB引擎就会先把记录写到 redo log 里面，并更新内存（db buffer），这个时候更新就算完成了。

此时，内存（db buffer）中的数据和磁盘数据（data file）对应的数据不同，我们认为内存中的数据是脏数据（即：脏页）；

同时，InnoDB引擎会在适当的时候，将这个操作记录更新到磁盘里面（data file），而这个更新往往是在系统比较空闲的时候做。

### 加锁规则

1. 先加 next-key lock，前开后闭；
2. 扫描到的数据都会加锁；
3. 索引上等值查询，对唯一索引加锁，next-key lock 会变成 行锁；
4. 索引上等值查询，对非唯一索引加锁，向后会锁到不符合条件的第一条为止，这时候 next-key lock 会变成 间隙锁；


