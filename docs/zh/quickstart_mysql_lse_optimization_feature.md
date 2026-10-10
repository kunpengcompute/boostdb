# 快速入门：MySQL LSE 优化特性

MySQL LSE（Large System Extensions）优化特性是鲲鹏BoostKit MySQL加速套件针对鲲鹏服务器OLTP场景提供的基础计算优化功能。该特性通过引入ARMv8.1规范的原子操作硬件加速方案，将计算操作移至L3 Cache执行，有效缓解了高并发读写环境下因锁竞争激烈导致的性能线性劣化问题。在Sysbench 8U16G规格、256并发测试条件下，使能该特性可使MySQL数据库的综合性能（含只读、读写、只写）提升约5%。

## 组件介绍

MySQL LSE优化特性是鲲鹏BoostKit MySQL加速套件的基础计算优化特性之一，用于在鲲鹏服务器的MySQL OLTP场景中，缓解高并发读写下锁竞争激烈导致的性能线性劣化问题。

LSE（Large System Extensions，大系统扩展）是专为现代多核、高并发环境设计的原子操作硬件加速方案，通过单指令原子操作，解决了传统LL/SC（Load\-Link/Store\-Conditional）指令在扩展性上的瓶颈。LL/SC原子指令需要把共享变量先加载到本核所在的L1 Cache中进行修改，在锁竞争少的情况下性能较好，但在锁竞争激烈时会导致系统性能下降严重。ARMv8.1规范引入的LSE将计算操作放到L3 Cache执行，增大数据共享范围，减少Cache一致性耗时，在锁竞争激烈时可以提升操作效率。

本文以Percona\-Server 5.7.44\-53为例，指导用户通过"获取RPM 包 → 安装 → 启动数据库 → 性能对比验证"的流程，快速上手使用MySQL LSE优化特性。使能本特性后，Sysbench 8U16G规格下256并发综合性能（只读、读写、只写）可提升约 5%。

> ![引出说明信息的图标](public_sys-resources/icon-note.gif)**说明**
>
> * Percona Server是一款与MySQL完全兼容的独立数据库产品，本特性以BoostDB RPM包形式提供，安装后默认已使能LSE优化。
> * 多核、原子锁争抢严重的场景下，也可在GCC编译选项中添加`-march=armv8-a+lse`选项，从源码层面使能LSE特性。

## 快速安装

### 环境准备

在正式操作前，请确保软硬件满足以下要求：

* 硬件：鲲鹏920新型号处理器或鲲鹏950处理器。

* 操作系统：openEuler 22.03 LTS SP4或openEuler 24.03 LTS SP3。

请按下表逐项检查环境：

|检查项|执行命令|预期输出|失败处理|
|---|---|---|---|
|CPU型号检查|lscpu|鲲鹏920新型号或鲲鹏950处理器|更换为符合要求的服务器|
|OS版本检查|cat /etc/*-release|openEuler 22.03 LTS SP4 或 24.03 LTS SP3|升级OS至符合要求的版本|
|编译依赖检查|参见《Percona 移植指南》"配置编译环境"章节|依赖包安装完成|按移植指南安装依赖|

### 安装步骤

#### 获取软件包

|软件类型|必选/可选|软件包说明|软件包名称|获取链接|
|---|---|---|---|---|
|主要安装包|必选|已内置LSE优化特性的BoostDB Percona RPM包（aarch64）|BoostDB-Percona-5.7.44-53.aarch64.rpm|[Percona\-Server 5.7.44\-53](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|基线安装包|必选（基线测试用）|未内置LSE优化特性的Percona Server5.7.44-53基线RPM包（aarch64），用于与BoostDB-Percona-5.7.44-53做基线对比|BoostDB-Percona-5.7.44-53.aarch64.rpm|[Percona\-Server 5.7.44\-53基线RPM](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-5.7.44-53-v3/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|主要安装包|可选（二选一）|8.0 版本RPM包|BoostDB-Percona-8.0.43-34.aarch64.rpm|[Percona\-Server 8.0.43\-34](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/BoostDB-Percona-8.0.43-34.aarch64.rpm)|
|基线安装包|可选（二选一）|未内置LSE优化特性的Percona Server8.0.43-34基线RPM包（aarch64），用于与BoostDB-Percona-8.0.43-34做基线对比|BoostDB-Percona-8.0.43-34.aarch64.rpm|[Percona\-Server 8.0.43\-34基线RPM](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-8.0.43-34-v2/BoostDB-Percona-8.0.43-34.aarch64.rpm)|

#### 执行安装

1. 安装依赖：请参见《Percona 移植指南》中的"配置编译环境"章节安装依赖。

2. 将下载的RPM包存放至目标路径（例如"/home"），执行如下命令安装。

    ```bash
    cd /home
    rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm
    ```

    安装完成后，默认安装目录位于"/usr/local/mysql"。

    > ![说明](public_sys-resources/icon-note.gif) **说明**
    >
    > 安装过程中，如果存在已安装依赖包但rpm相关检验不通过的情况，使用`--nodeps`跳过依赖检查。
    >
    > ```bash
    > rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps
    > ```

#### 启动数据库

启动数据库的核心步骤概览如下，详细操作说明（含每步的预期回显与注意事项）请参见《MySQL 移植指南》的"运行 MySQL"章节。

1. 修改配置文件my.cnf（配置 basedir、datadir、socket、port、user 等参数，路径根据实际情况修改）。

2. 配置环境变量，将MySQL二进制文件路径（默认"/usr/local/mysql/bin"）加入PATH并使其生效。

3. 初始化数据库：

    ```bash
    mysqld --defaults-file=/etc/my.cnf --initialize
    ```

    初始化回显中`root@localhost:`后会生成临时初始密码，请注意保存，登录时需要使用。

4. 启动数据库服务：

    ```bash
    service mysql start
    ```

5. 使用临时初始密码登录数据库并修改密码：

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    > **说明**
    > 上述命令中的安装路径、socket路径以my.cnf中的实际配置为准；如遇初始化或启动失败，请参见第 4 章"常见问题（FAQ）"。

### 安装验证

1. 查看数据库进程：

    ```bash
    ps -ef | grep mysql
    ```

    预期输出：可以看到mysqld进程，表示数据库服务已启动。

2. 登录数据库：

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    预期输出（部分）：

    ```output
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 8
    ```

    出现`mysql>`命令行提示符，表示数据库安装并运行成功。

## 快速开始

### A场景：Sysbench 高并发综合性能对比验证

#### 场景描述

以Sysbench 8U16G规格下256并发压测为例（Sysbench是业内广泛使用的开源、跨平台多线程性能测试工具，可用于 MySQL 数据库性能测试），通过对"未使能LSE特性的MySQL"与"已使能LSE特性的BoostDB Percona"分别执行只读、读写、只写三种模型的性能测试，展示使用LSE优化特性前后的性能提升效果。

#### 操作步骤

1. 基础配置。准备两套MySQL环境（使能前版本与使能后版本），Sysbench工具的安装、数据加载与环境调优请参见[《Sysbench 0.5&amp;1.0 测试指导》](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)。

2. 运行验证（使能前版本）。以Sysbench 1.0读写模型为例，先加载测试数据，再执行256并发测试。

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=256 --report-interval=10 oltp_read_write prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=256 --report-interval=10 oltp_read_write run
    ```

    预期输出：取结果中的transactions（每秒事务数）作为测试指标，记录使能前的性能基线。只读模型`oltp_read_write`替换为`oltp_read_only`，只写模型替换为`oltp_write_only`后重复执行。

3. 运行验证（使能 LSE 特性版本）。在已使能LSE优化特性的BoostDB Percona环境上，使用完全相同的命令与数据规模重复步骤2，记录使能后的性能数据。

#### 结果说明

使能MySQL LSE及rec\_get\_offsets优化特性后，Sysbench 8U16G规格下256并发综合性能（只读、读写、只写）提升约5%。

> **说明**
>
> * rec\_get\_offsets是InnoDB存储引擎中用于计算记录字段偏移量的热点函数，对该函数的优化与LSE优化通常一并合入BoostDB版本，共同贡献综合性能提升。
> * 文档中的IP地址、用户和密码仅为参考示例，请根据实际情况修改；详细的Sysbench测试步骤（数据加载、测试模型说明、环境调优、测试结果解读）请参见[《Sysbench 0.5&amp;1.0 测试指导》](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)。
> **安全提示**：运行MySQL的参考用户密码`123456`仅为演示用弱密码，生产环境请务必使用强密码，根据客户实际情况进行配置；执行测试Sysbench命令中的`--mysql-password`参数会在shell命令历史中留存明文密码，例如 ~/.bash\_history 等文件中，建议在测试完成后清理命令历史或改用专用测试账号。

### B场景：ASLR安全检查与加固

#### 场景描述

以开启操作系统ASLR安全保护为例，帮助用户在使能LSE优化特性的同时，完成基础的安全检查与加固。ASLR（Address Space Layout Randomization，地址空间布局随机化）是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。

#### 操作步骤

1. 开启ASLR。

    ```bash
    echo 2 > /proc/sys/kernel/randomize_va_space
    ```

2. 验证配置。

    ```bash
    cat /proc/sys/kernel/randomize_va_space
    ```

    预期输出：

    ```output
    2
    ```

#### 结果说明

回显值为2，表示ASLR已完全开启（堆、栈、共享库等地址全部随机化），系统具备针对缓冲区溢出攻击的基础防护能力。

## 常见问题（FAQ）

以下高频问题根据特性指南与移植指南的操作约束整理，详细解决方案请参见对应文档。

问题：执行`rpm -ivh`安装时依赖检查不通过，但确认依赖包已安装怎么办？

答复：使用`--nodeps`参数跳过依赖检查，即执行`rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps`。

问题：初始化数据库时提示 "\-\-initialize specified but the data directory has files in it." 怎么办？

答复：数据目录中已存在文件导致初始化失败。确认数据目录内容后，执行`rm -rf /data/mysql/data/*`（或实际配置的datadir路径）清空数据目录，再重新执行初始化命令。

问题：以root用户执行`service mysql start`首次启动失败，提示缺少mysql.log文件怎么办？

答复：先切换到 mysql 用户（`su - mysql`）启动数据库服务生成 mysql.log 文件，然后停止数据库服务，再次以 root 用户启动即可。详细说明请参见[MySQL 移植指南（运行 MySQL 章节）](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)。

问题：如何确认 LSE 优化特性是否已使能？

答复：本文使用的BoostDB Percona RPM包默认已内置并使能LSE优化，安装完成即生效，无需额外配置。可通过 3.1 章节的Sysbench对比压测验证性能提升效果。

## 更多功能

关于MySQL LSE优化特性和MySQL加速套件的更多使用方式请参见以下文档：

* [MySQL LSE优化 特性指南](https://www.hikunpeng.com/document/detail/zh/boostdb/mysql/basic_computation_opt/docs/zh/mysql_lse_optimization_feature_guide.md)

* [Percona 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0001.html)

* [MySQL 移植指南（运行 MySQL 章节）](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)

* [Sysbench 0.5&1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

## 修订记录

|文档版本|发布日期|修改说明|
|---|---|---|
|01|2026-09-30|第一次正式发布。|
