# MySQL表锁队列检查优化 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002602100002"></a>

### 简介<a name="ZH-CN_TOPIC_0000002602100003"></a>

本文主要介绍如何在鲲鹏服务器上安装和使用MySQL表锁队列检查优化特性。

在MySQL OLTP场景中，高并发读写下，InnoDB事务系统中的表锁队列检查很容易成为热点，进而引发链表扫描开销高的问题。本文以Percona-Server为例介绍如何在鲲鹏服务器上对InnoDB事务系统进行优化，以提升高并发写场景下的性能。

在OLTP场景中，事务在访问记录之前，通常会先申请表级意向锁。其中意向锁用于表示后续会继续申请更加细粒度的行锁。在以DML为主的负载下，这类意向锁的请求很多，但真正与其冲突的表锁（比如`LOCK_S`、`LOCK_X`）并不多。

原有实现中，意向锁进入表锁队列时，仍然需要整个队列扫描一遍做兼容性检查；释放锁后，队列中的等待关系也需要再次检查。当并发数较高时，这部分遍历开销会被放大，成为表锁路径上额外开销。

本特性在表锁队列上增加了更直接的状态判断，使系统能够先判断队列中是否存在会与意向锁冲突的表锁类型。对于不存在`LOCK_S`或`LOCK_X`的场景，可跳过不必要的队列遍历，直接完成兼容性检查。

通过减少重复扫描，该优化能够降低高并发DML场景下表锁检查、等待判断和队列唤醒的开销。同时，`AUTOINC`相关表锁状态也采用统一方式维护，使表锁队列状态管理更简洁。

## 环境要求<a name="ZH-CN_TOPIC_0000002602100005"></a>

本文基于特定环境提供指导，在正式操作前请确保软硬件环境满足要求。

**表1** 硬件要求<a id="hardware_requirements"></a>

|项目|规格|
|--|--|
|CPU|鲲鹏920系列处理器、鲲鹏950处理器|

**表2** 操作系统和软件要求<a id="software_requirements"></a>

|项目|名称|版本|获取地址|
|--|--|--|--|
|操作系统|openEuler|22.03 LTS SP4|[获取链接](https://repo.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)|
|Percona|Percona-Server|5.7.44-53|请参见《[BoostDB-Percona 安装指南](./boostdb-percona-install.md)》|

## 安装和使用特性<a name="ZH-CN_TOPIC_0000002602100006"></a>

BoostDB-Percona优化版本已默认集成本特性，无需单独获取补丁并重新编译安装。

以Percona-Server 5.7.44-53为例说明如何安装和使用本特性，具体步骤如下。

1. 请参见《[BoostDB-Percona 安装指南](./boostdb-percona-install.md)》安装BoostDB-Percona优化版本。
2. 启动数据库。启动数据库的操作请参见《MySQL移植指南》的[运行MySQL](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)章节。
3. （可选）通过Sysbench测试可以得到使能优化特性前后的性能提升效果，详细测试步骤请参见《[Sysbench 0.5&1.0测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)》。本特性可以使Sysbench只写场景性能提升2%，优化前后对比效果如[图1 表锁队列检查优化特性优化前后性能对比](#表锁队列检查优化特性优化前后性能对比)所示。

    **图1** 表锁队列检查优化特性优化前后性能对比<a name="fig937192253919"></a><a id="表锁队列检查优化特性优化前后性能对比"></a><br>

    ![](figures/表锁队列检查优化特性Sysbench写场景优化前后性能对比.png  "表锁队列检查优化特性优化前后性能对比")

## 安全检查与加固<a name="ZH-CN_TOPIC_0000002602100008"></a>

ASLR（Address Space Layout Randomization，地址空间布局随机化）是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。

```bash
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](figures/zh_cn_image_0000002504021297.png)
