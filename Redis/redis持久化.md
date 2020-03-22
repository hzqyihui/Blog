# redis持久化

Redis支持的持久化的2种机制：RDB和AOF
持久化功能：持久化功能有效地避免因进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数 据恢复

一、配置和运行流程

1、RDB持久化：
RDB持久化：把当前进程的数据生成快照保存到硬盘的过程
触发：手动触发、自动触发

1.1 手动触发 

save命令：

    //阻塞当前的Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会长时间的
    //阻塞，线上环境不建议使用，输出日志如下
    [1336] 06 Nov 14:12:02.428 * DB saved on disk

bgsave命令

save命令的优化。Redis进程执行fork操作创建子进程，RDB持久化过程由子 进程负责，完成后自动结束。阻塞只发生在fork阶段，时间很短。Redis内部所有的涉及RDB操作都是采用的bgsave方式。

    #输出日志
    [1336] 06 Nov 14:12:21.354 * Background saving started by pid 14367
    [14367] 06 Nov 14:12:21.397 * DB saved on disk
    [14367] 06 Nov 14:12:21.399 * RDB: 2 MB of memory used by copy-on-write
    [1336] 06 Nov 14:12:21.474 * Background saving terminated with success

1.2 自动触发

    1）使用save相关的配置，如“save m n”在m秒内数据集存在n次修改时，自动触发
    2）从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件发送给从节点
    3）执行debug reload 命令重新加载Redis时，会触发save操作
    4）默认情况下执行shutdown命令时，如果没有开启AOF持久化则会自动执行bgsave

![](bgsave1.jpg)

步骤：

    1）：执行bgsave命令，Redis父进程判断当前是否存在正在执行的子进 程，如RDB/AOF子进程，如果存在bgsave命令直接返回。
    2）：父进程执行fork操作创建子进程，fork操作过程中父进程会阻塞，通 过info stats命令查看latest_fork_usec选项，可以获取最近一个fork操作的耗时，单位为微秒。https://redis.io/commands/info
    127.0.0.1:6379> info stats
    # Stats
    total_connections_received:1             #服务器接收的连接总数Total number of connections accepted by the server
    total_commands_processed:4               #服务器处理的命令总数 Total number of commands processed by the server
    instantaneous_ops_per_sec:0              #每秒处理的命令数 Number of commands processed per second
    total_net_input_bytes:167                #从网络读取的字节总数
    total_net_output_bytes:0                 #写入网络的字节总数
    instantaneous_input_kbps:0.00            #网络每秒的读取速率（KB/秒）
    instantaneous_output_kbps:0.00           #网络每秒的写入速率（KB/秒）
    rejected_connections:0                  #由于maxclients限制而拒绝的连接数
    sync_full:0                             #全量同步数 The number of full resyncs with replicas
    sync_partial_ok:0                       #接受的部分重新同步请求数
    sync_partial_err:0                      #拒绝的部分重新同步请求数
    expired_keys:0                          #过期key总数
    evicted_keys:0                          #由于最大内存限制而逐出的key
    keyspace_hits:0                         #主词典中成功查找键的次数，命中的次数
    keyspace_misses:0                       #主字典中键查找失败的次数，未命中的次数
    pubsub_channels:0                       #具有客户端订阅的发布/订阅频道的总数
    pubsub_patterns:0                       #具有客户端订阅的发布/订阅模式的全局数量
    latest_fork_usec:102927                 #最近一次fork操作耗时
    3）：父进程fork完成后，bgsave命令返回“Background saving started”信息并不再阻塞父进程，可以继续响应其他命令。
    127.0.0.1:6379> bgsave
    Background saving started
    127.0.0.1:6379>
    4）：子进程创建RDB文件，根据父进程内存生成临时快照文件，完成后 对原有文件进行原子替换。执行lastsave命令可以获取最后一次生成RDB的 时间，对应info统计的rdb_last_save_time选项。
    127.0.0.1:6379> lastsave
    (integer) 1572878462
    5）：进程发送信号给父进程表示完成，父进程更新统计信息。命令info Persistence相关选项
    127.0.0.1:6379> info Persistence
    # Persistence
    loading:0                                   #指示转储文件的加载是否正在进行的标志
    rdb_changes_since_last_save:0               #自上次转储以来的更改数量
    rdb_bgsave_in_progress:0                    #指示RDB保存正在进行的标志
    rdb_last_save_time:1572879797               #上次成功保存RDB的时间戳
    rdb_last_bgsave_status:ok                   #上次RDB保存操作的状态
    rdb_last_bgsave_time_sec:-1                 #上次RDB保存操作的持续时间（以秒为单位）
    rdb_current_bgsave_time_sec:-1              #正在进行的RDB保存操作的持续时间（如果有）
    aof_enabled:0                              #是否开启AOF持久化
    aof_rewrite_in_progress:0                  #是否正在进行AOF重写操作
    aof_rewrite_scheduled:0                    #AOF重写是否完成
    aof_last_rewrite_time_sec:-1               #上一次AOF重写操作的持续时间（以秒为单位）
    aof_current_rewrite_time_sec:-1            #正在进行的AOF重写操作的持续时间（如果有）
    aof_last_bgrewrite_status:ok               #上一次AOF重写操作的状态
    aof_last_write_status:ok                   #对AOF的最后写入操作的状态

1.3 RDB文件保存位置

    # Compress string objects using LZF when dump .rdb databases?
    # For default that's set to 'yes' as it's almost always a win.
    # If you want to save some CPU in the saving child set it to 'no' but
    # the dataset will likely be bigger if you have compressible values or keys.
    rdbcompression yes
    
    # Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
    # This makes the format more resistant to corruption but there is a performance
    # hit to pay (around 10%) when saving and loading RDB files, so you can disable it
    # for maximum performances.
    #
    # RDB files created with checksum disabled have a checksum of zero that will
    # tell the loading code to skip the check.
    rdbchecksum yes
    
    # The filename where to dump the DB
    dbfilename dump.rdb
    
    # The working directory.
    #
    # The DB will be written inside this directory, with the filename specified
    # above using the 'dbfilename' configuration directive.
    # 
    # The Append Only File will also be created inside this directory.
    # 
    # Note that you must specify a directory here, not a file name.
    dir ./    
    
    如果Redis加载RDB文件时拒绝启动，并有如下日志：
    # Short read or OOM loading DB. Unrecoverable error, aborting now.
    //可以使用Redis提供的redis-check-dump工具检测RDB文件并获取对应的错误报告。
1.4 RDB的优缺点

    优点：
    RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据 快照。非常适用于备份，全量复制等场景。比如每6小时执行bgsave备份， 并把RDB文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复
    Redis加载RDB恢复数据远远快于AOF的方式。
    缺点：
    RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运 行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高
    针对RDB不适合实时持久化的问题，Redis提供了AOF持久化方式来解决。

2、AOF持久化
AOF持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的。主要作用：解决数据持久化的实时性。
开启AOF功能：默认不开启

    appendonly yes  #开启AOF功能
    appendfilename appendonly.aof #AOF文件名，默认appendonly.aof

工作流程：
![](AOF.jpg)

图2-1 AOF工作流程

步骤1：所有的写入命令会追加到aof_buf（缓冲区）中

步骤2：AOF缓冲区根据对应的策略向硬盘做同步操作

步骤3：随着AOF文件越来越大，需要定期对AOF文件进行重写，达到压缩
的目的

步骤4：当Redis服务器重启时，可以加载AOF文件进行数据恢复

命令写入：

    写入的内容是文本协议格式。

文件同步（AOF缓冲区像硬盘同步）

Redis提供了多种AOF缓冲区同步文件测虐，由参数appendfsync控制
    
    # appendfsync always   #命令写入aof_buf后调用系统fsync操作同步到AOF文件，fsync完成后线程返回
    appendfsync everysec   #命令写入aof_buf后调用系统的write操作，write完成后线程返回,fsync同步文件操作由专门的线程每秒调用一次
    # appendfsync no #命令写入aof_buf后调用write操作，不对AOF文件做fsync同步，同步硬盘由操作系统辅助
1）配置为always时，每次命令写入都要同步AOF文件，在一般的SATA硬盘 上，Redis只能支持大约几百TPS写入，显然跟Redis高性能特性背道而驰，不建议配置。
2）配置为no，由于操作系统每次同步AOF文件的周期不可控，而且会加大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证。
3）配置为everysec，是建议的同步策略，也是默认配置，做到兼顾性能和 数据安全性。理论上只有在系统突然宕机的情况下丢失1、2秒的数据。

重写机制

    Redis 引入AOF重写机制压缩文件体积。AOF文件重写是把Redis进程内的数据转 化为写命令同步到新AOF文件的过程。
    重写后的AOF文件为什么可以变小？有如下原因：
    1）AOF含有的无效命令不会被写入，如del key1、hdel key2等。重写使用进程内数据直接生成，新的AOF文件只保留最终数据的写入命令。
    2）多条命令合并为1个，如：lpush list a、lpush list b等转化为 lpush list a b c.

AOF重写触发：手动触发和自动触发

手动触发：

    bgrewriteaof
自动触发：根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数配置确定自动触发时机
    
    #自动触发机制：aof_current_size>auto-aof-rewrite-min-size && (aof_current_size-aof_base_size)/aof_base_size>=auto-aof-rewrite-percentage
    auto-aof-rewrite-percentage 100 #运行AOF重写时文件的最小体积
    auto-aof-rewrite-min-size 64mb  #当前AOF文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值
![](AOFrewrite.jpg)
图2-2 AOF重写流程

步骤1：当进程正在执行AOF重写，请求不执行。

步骤3.1：fork操作完成后继续响应其他的命令。

步骤3.2：父进程依然响应命令，使用“AOF重写缓冲区”保存新数据

步骤4：子进程根据内存快照，按照命令合并规则写到新的AOF文件（批量写入数据的配置：aof-rewrite-incremental-fsync）

步骤5.1:AOF写入完成后，子进程发送信号给父进程，父进程更新统计。

3、重启加载

AOF和RDB文件都可以用于服务器重启时的数据恢复
![](重启加载.png)
说明：
1）、AOF持久化开启且存在AOF文件，优先加载AOF文件，打印如下：

    * DB loaded from append only file: 5.841 seconds
2）、AOF关闭或者AOF文件不存在，加载RDB文件，打印如下：
    
    * DB loaded from disk: 0.065 seconds
3）、文件AOF/RDB文件成功，Redis启动成功

4）、加载文件错误，失败

注：如果开启了AOF持久化，启动redis时就直接从AOF持久化文件中读取数据。开启AOF持久化之前，最好手动触发一次AOF重写的机制（bgrewriteaof），将原始的数据初始化到AOF持久化文件。下次启动redis时才不会造成原始的数据丢失。
