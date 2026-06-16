# 版本说明书

## 2026-06-30

### 修改记录

| 文档版本 | 发布日期       | 修改说明                                                                                                                                 |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 |- 首次发布MySQL Plan Cache特性。<br>- 首次发布MySQL事务锁优化特性。<br>- 首次发布MySQL字符集处理SIMD优化特性。 |
### 版本配套说明
#### 产品版本信息
| 产品名称      | 产品版本     |
| --------- | -------- |
| BoostDB | 26.1.RC1 |
#### 软件版本配套说明
|特性名称|软件类型|版本|
|--|--|--|
|MySQL事务锁优化|OS|openEuler 22.03 LTS SP4|
|MySQL事务锁优化|Percona|Percona-Server 5.7.44-53|
|MySQL事务锁优化|Patch|001-Refactor-InnoDB-lock_sys-sharding-and-latching.patch <br>0002-Optimize-InnoDB-table-lock-queue-checks.patch <br>0003-innodb-enhance-read-view-management-with-version-tracking.patch|
|MySQL Plan Cache|OS|openEuler 24.03 LTS SP3|
|MySQL Plan Cache|Percona|Percona-Server 8.0.43-34|
|MySQL Plan Cache|Boost|boost_1_77_0|
|MySQL Plan Cache|Patch|0001-add-plan-cache-to-improve-select-performance.patch|
|MySQL字符集处理SIMD优化|OS|openEuler 24.03 LTS SP3、openEuler 22.03 LTS SP4|
|MySQL字符集处理SIMD优化|Percona|Percona-Server 8.0.43-34、Percona-Server 5.7.44-53|
|MySQL字符集处理SIMD优化|Boost|boost_1_77_0、boost_1_59_0|
|MySQL字符集处理SIMD优化|Patch|0001-refactor-character-set-handling-for-aarch64-architec.patch|

#### 硬件版本配套说明
| 特性名称          | 硬件项目          | 要求说明                                      |
| ------------- | ------------- | ----------------------------------------- |
| MySQL事务锁优化、MySQL Plan Cache、MySQL字符集处理SIMD优化     | 处理器           | 鲲鹏920系列处理器                   |

#### 病毒扫描结果
不涉及软件包发布，不涉及病毒扫描。
### 版本使用注意事项
无
### 版本说明
#### 更新说明
##### MySQL事务锁优化
新增MySQL事务锁优化特性。该特性包含3个补丁，分别从锁系统分片与闩锁优化、表锁队列检查优化以及Read View版本跟踪优化三个方面增强Percona-Server 5.7.44-53的InnoDB事务与锁管理能力，以降低高并发场景中的全局互斥竞争、表锁队列扫描开销和Read View管理成本。

##### MySQL Plan Cache
新增MySQL Plan Cache特性。该特性面向Prepared Statement场景缓存`SELECT`语句的执行计划，在后续执行中优先复用已缓存的执行计划，从而减少重复优化开销，提升热点查询的执行性能和OLTP场景下的系统稳定性，在Sysbench只读场景中可获得10%的性能提升。

##### MySQL字符集处理SIMD优化
新增MySQL字符集处理SIMD优化特性。在MySQL OLTP场景中，针对字符集处理相关的热点函数（如my_strnxfrm_unicode、my_ismbchar_utf8、my_charpos_mb、my_hash_sort_utf8、my_lengthsp_8bit），利用SIMD指令进行了向量化加速优化，在Sysbench只读场景中可获得10%的性能提升。

#### 已解决的问题
无
#### 遗留问题
无
### 版本配套文档

|文档名称|内容简介|交付形式|
|--|--|--|
|《MySQL事务锁优化 特性指南》|提供MySQL事务锁优化特性的环境要求、安装步骤和使用说明。|开源仓|
|《MySQL Plan Cache 特性指南》|本文档提供MySQL Plan Cache特性的环境要求、特性使能指导和使用说明。|开源仓|
|《MySQL字符集处理SIMD优化 特性指南》|本文档提供MySQL字符集处理SIMD优化特性的环境要求、特性使能指导和使用说明。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002544372643"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostdb/tree/master/docs)浏览和获取相关文档。

## 2026-03-30

### 修改记录

| 文档版本 | 发布日期       | 修改说明                                                                                                                                 |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 |- 首次发布MySQL LSE优化特性。<br>- 首次发布MySQL rec_get_offsets优化特性。 |
### 版本配套说明
#### 产品版本信息
| 产品名称      | 产品版本     |
| --------- | -------- |
| BoostDB | 26.0.RC1 |
#### 软件版本配套说明
|特性名称|软件类型|版本|
|--|--|--|
|MySQL LSE优化、MySQL rec_get_offsets优化|OS|openEuler 22.03 LTS SP4、openEuler 24.03 LTS SP3|
|MySQL LSE优化、MySQL rec_get_offsets优化|Percona|Percona-Server 5.7.44-53、Percona-Server 8.0.43-34|                                                                             |
#### 硬件版本配套说明
| 特性名称          | 硬件项目          | 要求说明                                      |
| ------------- | ------------- | ----------------------------------------- |
| MySQL LSE优化、MySQL rec_get_offsets优化     | 处理器           | - 鲲鹏920系列处理器<br>- 鲲鹏950处理器                    |

#### 病毒扫描结果
不涉及软件包发布，不涉及病毒扫描。
### 版本使用注意事项
无
### 版本说明
#### 更新说明
##### MySQL LSE优化
通过引入LSE硬件加速，来有效缓解高并发下的性能劣化问题，提升MySQL在鲲鹏服务器中的整体性能表现与系统稳定性。

##### MySQL rec_get_offsets优化
通过优化rec_get_offsets的调用逻辑，来有效提升MySQL在鲲鹏服务器中的整体性能表现与系统稳定性。
#### 已解决的问题
无
#### 遗留问题
无
### 版本配套文档

|文档名称|内容简介|交付形式|
|--|--|--|
|《MySQL LSE优化 特性指南》|本文档提供MySQL LSE优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL rec_get_offsets优化 特性指南》|本文档提供MySQL rec_get_offsets优化特性的环境要求、特性使能指导。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002544372643"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostdb/tree/master/docs/zh)浏览和获取相关文档。

## 2025-12-30

### 修改记录

| 文档版本 | 发布日期       | 修改说明                                                                                                                                 |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2025-12-30 |- 首次发布MySQL Binlog预分配优化特性。<br>- 首次发布MySQL Binlog拆锁优化特性。<br>- 首次发布MySQL Binlog writeset_history数据结构优化特性。<br>- 首次发布MySQL记录匹配优化特性。<br>- 首次发布MySQL字符集处理SIMD优化特性。<br>- 首次发布MySQL非对齐内存访问优化特性。 |
### 版本配套说明
#### 产品版本信息
| 产品名称      | 产品版本     |
| --------- | -------- |
| BoostDB | 25.3.0 |
#### 软件版本配套说明

|特性名称|软件类型|版本|
|--|--|--|
|MySQL Binlog预分配优化<br>MySQL Binlog拆锁优化<br>MySQL Binlog writeset_history数据结构优化<br>MySQL记录匹配优化<br>MySQL字符集处理SIMD优化<br>MySQL非对齐内存访问优化|OS|openEuler 22.03 LTS SP4|
|MySQL Binlog预分配优化<br>MySQL Binlog拆锁优化<br>MySQL Binlog writeset_history数据结构优化<br>MySQL记录匹配优化<br>MySQL字符集处理SIMD优化<br>MySQL非对齐内存访问优化|Percona|Percona-Server 5.7.44-53<br>Percona-Server 8.0.43-34|

#### 硬件版本配套说明
| 特性名称          | 硬件项目          | 要求说明                                      |
| ------------- | ------------- | ----------------------------------------- |
| MySQL Binlog预分配优化<br>MySQL Binlog拆锁优化<br>MySQL Binlog writeset_history数据结构优化<br>MySQL记录匹配优化<br>MySQL字符集处理SIMD优化<br>MySQL非对齐内存访问优化      | 处理器           | 鲲鹏920新型号处理器、鲲鹏950处理器                  |

#### 病毒扫描结果
不涉及软件包发布，不涉及病毒扫描。
### 版本使用注意事项
无
### 版本说明

#### 更新说明

##### MySQL Binlog预分配优化
通过在Binlog文件创建时将其大小预分配为max_binlog_size（最大文件大小），可避免写入过程中因文件动态增长所引入的元数据操作，从而降低I/O开销并提高系统性能。

##### MySQL Binlog拆锁优化
通过锁拆分，让处于FLUSH/SYNC/COMMIT各个阶段的follower等待不同的锁，降低不同组之间误唤醒的可能，来提升系统性能。

##### MySQL Binlog writeset_history数据结构优化
通过使用hash map数据结构替换writeset_history数据结构，将插入和查找的时间复杂度降为O(1)，以提高效率。

##### MySQL记录匹配优化
针对采用固定行格式的记录，将原有逐字段遍历匹配的方式优化为基于memcmp的整体记录比较，从而提升比较效率。

##### MySQL字符集处理SIMD优化
利用SIMD指令对utf8/utf8mb4字符集处理过程进行向量化加速，提升相关运算效率。

##### MySQL非对齐内存访问优化
鲲鹏920处理器已具备非对齐内存访问能力，因此可将x86中相应的优化策略移植至ARM架构，直接将指针转换为整型变量，从而提高类型转换效率。

#### 已解决的问题
无
#### 遗留问题
无
### 版本配套文档

|文档名称|内容简介|交付形式|
|--|--|--|
|《MySQL Binlog预分配优化 特性指南》|本文档提供MySQL Binlog预分配优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL Binlog拆锁优化 特性指南》|本文档提供MySQL Binlog拆锁优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL Binlog writeset_history数据结构优化 特性指南》|本文档提供MySQL Binlog writeset_history数据结构优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL记录匹配优化 特性指南》|本文档提供MySQL记录匹配优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL字符集处理SIMD优化 特性指南》|本文档提供MySQL字符集处理SIMD优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL Binlog预分配优化 特性指南》|本文档提供MySQL Binlog预分配优化特性的环境要求、特性使能指导。|开源仓|




### 获取文档的方法<a name="ZH-CN_TOPIC_0000002544372643"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostdb/tree/master/docs)浏览和获取相关文档。