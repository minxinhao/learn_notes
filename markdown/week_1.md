# README.md

大致介绍了LevelDb。包括它的性能和大致使用。对我主要有用的是最后一节Repository contents.介绍了主要的介绍文档和LevelDb的主要模块以及对应的文件。让我有了一个开始看的方向。

主要有：

* doc/index.md **leveldb库用法描述**
* doc/impl.md **leveldb实现的简单描述**
* include/leveldb/*.h **leveldb的接口**

leveldb的主要模块包括：

* db
* options
* comparator
* iterator
* write_batch
* slice
* status
* env
* table

## index.md

这一部分介绍了levelDb库的基本用法，从外部接口上描述了levelDb。LevelDb基本用法和涉及到的数据结构包括：

1. open db
2. check db status
3. close db
4. reads and writes
5. atomic updates (WriteBatch)
6. synchronous writes (WriteOptions)
7. concurrency (iteration and WriteBatch needs extern synchroniztion)
8. iteration
9. snapshots
10. slice
11. comparators
12. backwards compatibility (the name of comparator must keep consist although the structure of key and comparator stay changing)
13. performance (adjust paramater)
    * block size (adjacent keys are organized in adjacent or the same block.Change block size for the need of read )
    * compression (set option.compression kNoCompression if it's needed )
    * cache (choose the size of option.block_cache according to read performance)
    * key layout (add extra charater to keys to acheive your access need)

14. filter (bloomfilter:reduce disk reads)
15. checksum
16. approximate sizes (range)
17. env
18. port

这些的基本用法的代码和详细解释[index.md](https://github.com/minxinhao/leveldb/blob/master/doc/index.md)中已经给出了，我就不重复了。

    到基本目前为止只看了leveldb的库接口的设计和基本用法，后续要继续看底层的实现。

## impl.md

介绍了leveldb的实现模型。主要是数据按新旧存放在不同文件中，然后文件按层存放的格式和compation操作。阅读的主要障碍还是compaction。

leveldb主要的文件格式如下图：

![leveldb_structure]()