# BoostDB-Percona 安装指南

## 简介

### BoostDB-Percona-5.7.44-53优化版

BoostDB-Percona-5.7.44-53.aarch64.rpm基于Percona-Server-5.7.44-53源码，结合鲲鹏处理器进行深度优化，集成以下性能优化补丁，覆盖硬件指令加速、存储引擎优化、查询优化等多个方面，显著增强了Percona-5.7在鲲鹏服务器上的运行性能。在此基础上运用了鲲鹏GCC CFGO（Continuous Feature Guided Optimization）反馈优化特性，在8U规格下，Sysbench综合性能提升25%以上。

- 热点函数inline优化
- 调整Cacheline大小到128B
- 使用硬件指令加速crc32
- 使用NEON指令集向量化处理字符集
- 使用NEON指令集优化字符串转换
- THD数据结构优化
- 优化page_id_t的结构，移除不必要的成员变量以及相关逻辑
- 使用哈希表代替线性搜索优化字符集排序规则查找
- 优化InnoDB元组比较的性能
- 优化MVCC读视图数据结构
- 新增查询计划缓存功能
- binlog空间预分配和复用
- binlog拆锁优化
- binlog gtid unordered_map替换

### BoostDB-Percona-8.0.43-34优化版

BoostDB-Percona-8.0.43-34.aarch64.rpm基于Percona-Server-8.0.43-34源码，结合鲲鹏处理器进行深度优化，集成以下性能优化补丁，覆盖硬件指令加速、binlog加速等多个方面，显著增强了Percona-8.0.43在鲲鹏服务器上的运行性能，在此基础上运用了鲲鹏GCC CFGO（Continuous Feature Guided Optimization）反馈优化特性，在8U规格下，Sysbench综合性能提升30%以上。

- 使用NEON指令集向量化处理字符集
- 支持非对齐内存访问优化
- 优化InnoDB元组比较的性能
- binlog空间预分配
- binlog拆锁优化

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

## 安装boostdb特性<a name="ZH-CN_TOPIC_0000002550177571"></a>

以下步骤同时适用于Percona-Server 5.7.44-53和Percona-Server 8.0.43-34，请根据实际版本选择对应分支命令。

1. 请参见《Percona移植指南》中的[配置编译环境](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0014.html)章节安装依赖。
2. 请参见[**表 2** 操作系统和软件要求](#操作系统和软件要求)下载对应版本rpm包并存放至目标路径，例如“/home”。
3. 执行如下命令安装rpm包。安装完成后，默认安装目录位于“/usr/local/mysql”。

    - Percona-Server 5.7.44-53：

    ```shell
    cd /home
    rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm
    ```

    - Percona-Server 8.0.43-34：

    ```shell
    cd /home
    rpm -ivh BoostDB-Percona-8.0.43-34.aarch64.rpm
    ```

    >![](public_sys-resources/icon_note.gif) **说明：**
    >安装过程中，如果存在已安装依赖包但rpm相关检验不通过的情况，使用--nodeps跳过依赖检查，即执行如下命令。
>
    >```shell
    >rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps
    >rpm -ivh BoostDB-Percona-8.0.43-34.aarch64.rpm --nodeps
    >```

## 故障排除<a name="ZH-CN_TOPIC_0000002550137571"></a>

### 启动MySQL时报version `GLIBCXX_3.4.29' not found的解决方法<a name="ZH-CN_TOPIC_0000002550137567"></a>

**问题现象描述<a name="section642124153116"></a>**

启动MySQL时报错：/usr/local/mysql/bin/mysqld: /usr/local/mysql/bin/mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by /usr/local/mysql/bin/mysqld)。

**关键过程、根本原因分析<a name="section145813300553"></a>**

系统libstdc++.so.6版本低，缺少GLIBCXX_3.4.29。

**结论、解决方案及效果<a name="section164566494716"></a>**

1. 下载gcc 12.3.1（GCC for openEuler 3.0.3）。

    ```shell
    cd /home
    wget https://mirrors.huaweicloud.com/kunpeng/archive/compiler/kunpeng_gcc/gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

2. 执行以下命令解压。

    ```shell
    tar zxvf gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

3. 不替换系统库，仅在当前终端会话中临时指定高版本libstdc++.so.6。

    ```shell
    export GCC_HOME=/home/gcc-12.3.1-2024.12-aarch64-linux
    export LD_LIBRARY_PATH=$GCC_HOME/lib64:$LD_LIBRARY_PATH
    ```

4. 检查当前会话使用的库是否包含GLIBCXX_3.4.29，若有输出，则说明已满足需求。

    ```shell
    strings $GCC_HOME/lib64/libstdc++.so.6 | grep GLIBCXX_3.4.29
    ```

5. 在同一终端会话中重新启动MySQL（需继承上述环境变量）。

    >![](public_sys-resources/icon_note.gif) **说明：**
    >上述export方式仅对当前会话生效，关闭终端后失效，不会修改系统默认libstdc++.so.6。

## 安全检查与加固<a name="ZH-CN_TOPIC_0000002518537816"></a>

ASLR（Address Space Layout Randomization，地址空间布局随机化）是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。

```shell
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](figures/zh_cn_image_0000002518697736.png)
