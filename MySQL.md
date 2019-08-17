

### mysql相关问题 WAL机制、crash safe如何实现、redo log作用 

WAL （Write-Ahead Logging） ，关键点是：先写日志在写磁盘

当执行一条sql的时候，WAL体现如下：

