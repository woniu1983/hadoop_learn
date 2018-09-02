# 表拆分
    大表拆分成小表，分区表，外部表，临时表

# MR 优化
        map和reduce的个数： 
        一个分片就是一个块，一个块对应一个maptask；
        Hadoop源码中有一个计算公式，决定了map的个数
            min(max_split_size,max(min_split_size,block_size))
                min_split_size默认值0（最小分片大小）
                block_size，block_size默认是128
                max_split_size默认值256（最大分片大小）
        一般在实际的生产环境中HDFS一旦format格式化之后，block_size大小不会去修改的；
        通过修改max_split_size和min_split_size来影响map的个数；


# Hive并行执行
    针对有些互相没有依赖关系的独立的job，可以选择并发的执行job
    hive.exec.parallel
    hive.exec.parallel.thread.number
    一般在工作中会选择去开启该功能
    根据实际的集群的状况和服务器的性能合理的设置线程数目


# JVM重用
        MR默认jvm运行，开启JVM开启多个任务
        开启数目需测试
        mapreduce.job.jvm.numtasks


============================================================

# Hadoop/Hive压缩
* gzip/zlib/bzip2/lz4/snappy等

* 首先看一下你的集群是否支持/安装了snappy
    | hadoop checknative -a



