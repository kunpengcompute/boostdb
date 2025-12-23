# boostdb

BoostKit数据库解决方案主页仓

## MySQL

[BoostKit数据库解决方案mysql仓](https://gitcode.com/boostkit/mysql)，包含Kunpeng MySQL，提供包含NEON指令集向量化处理、非对齐内存访问优化、InnoDB、binlog等优化patch，以及基于GCC For openEuler的反馈编译优化版本。

### 1.1  版本说明
当前包含Percona-Server-5.7.44.53构建库、Percona-Server-8.0.43-34构建库。

### 1.2  特性说明

#### 1.2.1 基础计算加速

本特性包含硬件CRC32/SIMD指令加速，非对齐内存访问等关键技术。数据库中频繁调用CRC32校验，如进行文件page页正确性的检验等，基于鲲鹏CRC32硬指令优化CRC32软件校验算法，可以提升校验性能。SIMD指令加速对utf8/utf8mb4字符集处理过程进行向量化加速，提升相关运算效率。在早期ARM架构（如ARMv5）中，由于不支持非对齐内存访问，MySQL在指针到整型的转换过程中采用逐字节读取并移位累加的方式，鲲鹏920服务器已具备非对齐访问的能力，非对齐内存访问优化直接将指针转换为整型变量，从而提高类型转换效率。本特性在Sysbench读场景性能提升10%，写场景性能共提升5%。

#### 1.2.2 Binlog优化

本特性主要从BInlog预分配、Binlog拆锁优化和Binlog writeset_history数据结构优化三个部分进行优化来提高系统性能。Binlog日志文件的动态增长会带来额外的元数据开销，Binlog预分配通过在Binlog文件创建时将其大小预分配为max_binlog_size，避免写入过程中因文件动态增长所引入的元数据操作，从而降低I/O开销并提高系统性能。在事务组提交过程中，处于FLUSH/SYNC、COMMIT阶段的所有follower会共享同一把锁和条件变量，通过Binlog拆锁优化，让处于不同阶段的follower等待不同的锁，降低等待锁的开销。Binlog writeset_history数据结构优化使用hash_map数据结构替换std::map，提高效率。本特性在Sysbench只写场景性能提升13%。

#### 1.2.2 编译器优化

GCC for openEuler编译器是面向鲲鹏和openEuler生态的优选高性能编译器，基于开源GCC（GNU Compiler Collection，GNU编译器套装）开发和全场景优化，生态成熟度高。GCC for openEuler反馈编译组件在不改变程序功能的前提下，可以收集程序的执行路径、函数调用次数、变量的使用情况等信息，并将这些信息反馈给编译器。编译器利用这些信息进行更精细的代码优化，以获得性能更优的目标程序。在整机场景下，此优化可使数据库TPC-C综合性能提升10%；在8U32GB场景下，Sysbench综合性能（最优性能）提升30%。

## Redis

[BoostKit数据库解决方案Redis仓](https://gitcode.com/boostkit/Redis)，包含Kunpeng Redis，即鲲鹏参与开源社区的Redis，做了网络异步化优化，提升Redis吞吐量。

### 2.1 版本说明
当前特性适用于Redis 6.0.20和Redis 7.0.15。

### 2.2 特性说明

#### 2.2.1 KRAIO网络异步化优化

KRAIO（Kunpeng Redis Asynchronous I/O）是鲲鹏自研的批量异步IO算法，Redis网络异步化通过将Redis中网络IO操作交由KRAIO异步批量执行，减少系统调用和上下文切换，实现Redis业务无阻塞执行，从而提高Redis吞吐量。通过sqpoll模式，KRAIO启用一个内核线程，自动处理网络IO事件，实现无需系统调用完成IO操作。

## Milvus
[BoostKit数据库解决方案Milvus仓](https://gitcode.com/boostkit/milvus)，包含Kunpeng Milvus，提供包含Milvus向量指令和预取处理、Milvus KBest、Milvus KScaNN等优化patch。
### 3.1 版本说明
当前包含Milvus 2.4.5。
### 3.2 特性说明
#### 3.2.1 Milvus向量指令优化
该特性使用sve指令集和软硬件预取实现，减小了距离函数计算的开销。
#### 3.2.2 Milvus KBest优化
该特性通过Milvus预留接口，对接鲲鹏自研召回算法KBest，发挥鲲鹏优势，使查询性能(QPS)在高召回率(0.99)的前提下，获得30%以上的提升。
#### 3.2.3 Milvus KScaNN优化
该特性通过Milvus预留接口，通过对接鲲鹏自研召回算法KScaNN，发挥鲲鹏优势，使查询性能(QPS)在高召回率(0.95以上)的前提下，获得30%以上的提升。