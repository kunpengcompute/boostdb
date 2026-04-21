# MySQL记录匹配优化 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002518697734"></a>

### 简介<a name="ZH-CN_TOPIC_0000002518537818"></a>

本文主要介绍如何在鲲鹏服务器上安装和使能记录匹配优化特性。

在MySQL OLTP场景中，使用Sysbench工具进行只读模型测试时，性能监控显示的主要热点函数包括记录匹配相关的函数（如row_search_mvcc、rec_get_offsets_func、page_cur_search_with_match、btr_cur_search_to_nth_level）。针对这些热点函数，鲲鹏BoostKit进行了针对性优化，显著提升了执行效率。

通过记录匹配优化、字符集处理的SIMD和非对齐内存访问优化三特性叠加，在Percona-Server 5.7.44-53版本8C16G容器规格中，Sysbench只读测试场景下的性能获得了约10%的提升。

### 原理描述<a name="ZH-CN_TOPIC_0000002550177569"></a>

**记录匹配优化<a name="section479993815484"></a>**

本优化针对采用固定行格式的记录，将原有逐字段遍历匹配的方式优化为基于memcmp的整体记录比较，从而提升比较效率。

在InnoDB存储引擎中，原比较逻辑通过函数cmp_dtuple_rec_with_match_low逐字段对比数据结构dtuple_t（内存记录格式）与数据结构rec_t（页物理记录格式），比较前还需调用rec_get_offsets获取字段偏移量。优化后，通过memcmp直接比较整条记录，不仅减少了比较次数，也避免了频繁调用rec_get_offsets带来的开销。

在代码实现层面，主要调整包括：

- 在row_search_mvcc函数中，以memcmp替换原有的cmp_dtuple_rec比较逻辑。
- 在page_cur_search_with_match函数中，采用新的字节级比较方法rec_direct_memcmp进行调用，替代原有的rec_get_offsets方法与cmp_dtuple_rec_with_match方法。

使用memcmp进行记录比较需满足以下条件：

- 所有字段均支持通过memcmp进行比较（参见函数dtype_is_memcmp_deterministic）。
- 所有字段为固定长度且定义为非空。
- 所有字段按相同升序排列。

此外，InnoDB原仅缓存内部表的记录偏移量。本优化进一步将偏移量缓存机制扩展至所有含固定长度字段的索引，实现偏移量的复用，减少重复计算，从而提升B-tree搜索的整体性能。

## 环境要求<a name="ZH-CN_TOPIC_0000002518697732"></a>

本文基于特定环境提供指导，在正式操作前请确保软硬件均满足要求。

**表 1** 硬件要求<a id="硬件要求"></a>

|项目|规格|
|--|--|
|CPU|鲲鹏920新型号处理器、鲲鹏950处理器|

**表 2** 操作系统和软件要求<a id="操作系统和软件要求"></a>

|项目|版本|获取地址|
|--|--|--|
|操作系统|openEuler 22.03 LTS SP4|[获取链接](https://repo.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)|
|Percona|Percona-Server 5.7.44-53|[获取链接](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-5.7.44-53-v3/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|Percona|Percona-Server 8.0.43-34|[获取链接](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-8.0.43-34-v2/BoostDB-Percona-8.0.43-34.aarch64.rpm)|

## 安装和使能特性<a name="ZH-CN_TOPIC_0000002550177571"></a>

以Percona-Server 5.7.44-53为例介绍如何安装和使能记录匹配优化特性，具体操作步骤如下。

1. 请参见《Percona移植指南》中的[配置编译环境](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0014.html)章节安装依赖。
2. 请参见[**表 2** 操作系统和软件要求](#操作系统和软件要求)下载Percona-Server 5.7.44-53对应的rpm包并存放至目标路径，例如"/home"。
3. 执行如下命令安装rpm包。安装完成后，默认安装目录位于"/usr/local/mysql"。

    ```
    cd /home
    rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm
    ```

    >![](public_sys-resources/icon_note.gif) **说明：**
    >安装过程中，如果存在已安装依赖包但rpm相关检验不通过的情况，使用--nodeps跳过依赖检查，即执行如下命令。
    >```
    >rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps
    >```

4. 启动数据库。启动数据库的操作请参见《MySQL移植指南》的[运行MySQL](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)章节。

5. （可选）通过Sysbench测试可以得到使能记录匹配优化特性前后的性能提升效果，详细测试步骤请参见《[Sysbench 0.5&1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)》。<br>通过记录匹配优化、字符集处理的SIMD和非对齐内存访问优化三特性叠加，可以使Sysbench只读场景性能提升10%，优化前后对比效果如[图 三特性叠加优化前后性能对比](#fig937192253919)所示。

    **图 1** 三特性叠加优化前后性能对比<a name="fig937192253919"></a><a id="计算路径优化特性优化前后性能对比"></a><br>
    ![](figures/计算路径优化特性优化前后性能对比.png "记录匹配优化特性优化前后性能对比")

## 故障排除<a name="ZH-CN_TOPIC_0000002550137571"></a>

### 启动MySQL时报version `GLIBCXX_3.4.29' not found的解决方法<a name="ZH-CN_TOPIC_0000002550137567"></a>

**问题现象描述<a name="section642124153116"></a>**

启动MySQL时报错：/usr/local/mysql/bin/mysqld: /usr/local/mysql/bin/mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by /usr/local/mysql/bin/mysqld)。

**关键过程、根本原因分析<a name="section145813300553"></a>**

系统libstdc++.so.6版本低，缺少GLIBCXX_3.4.29。

**结论、解决方案及效果<a name="section164566494716"></a>**

1. 下载gcc 12.3.1（GCC for openEuler 3.0.3）。

    ```
    cd /home
    wget https://mirrors.huaweicloud.com/kunpeng/archive/compiler/kunpeng_gcc/gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

2. 执行以下命令解压。

    ```
    tar zxvf gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

3. 备份当前系统的libstdc++.so.6，创建高版本libstdc++.so.6软链接。

    ```
    mv /usr/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6.bak
    ln -s /home/gcc-12.3.1-2024.12-aarch64-linux/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6
    ```

4. 检查当前库版本，若有输出，则说明已满足需求。

    ```
    strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX_3.4.29
    ```

5. 重新启动MySQL。

## 安全检查与加固<a name="ZH-CN_TOPIC_0000002518537816"></a>

ASLR（Address Space Layout Randomization，地址空间布局随机化）是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。

```
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](figures/zh_cn_image_0000002518697736.png)