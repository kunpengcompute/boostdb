# 快速入门：MySQL CRC32指令优化特性

本文档介绍了鲲鹏BoostKit MySQL加速套件中的CRC32指令优化特性，旨在通过利用鲲鹏处理器（如鲲鹏920处理器）的硬件 CRC32指令替换软件实现，降低计算开销，从而提升MySQL在openEuler操作系统上的性能（Sysbench 写场景性能可提升约 5%）。

## 组件介绍

CRC32指令优化特性是鲲鹏BoostKit MySQL加速套件的基础计算优化特性之一，用于在使用openEuler操作系统的鲲鹏服务器上提升 MySQL的CRC（Cyclic Redundancy Check，循环冗余校验）计算性能。该特性通过使用鲲鹏CRC32硬件指令替换CRC32算法的软件实现，减小了CRC32的计算开销。

Linux内核中虽然包含了CRC32算法的C语言实现，但是由于性能较低，当内核态中CRC3 函数调用占比较高时，可能成为系统性能瓶颈。采用鲲鹏CRC32硬件指令替换软件实现后，MySQL Sysbench 写场景性能可提升约5%。

本文以MySQL 8.0.25为例，指导用户通过“获取补丁 → 应用补丁 → 编译安装 → 测试验证 → 性能对比”的完整流程，快速上手使用CRC32指令优化特性。其他场景也可参考本文的方法进行适配优化。

>![说明](public_sys-resources/icon-note.gif)  **说明**
>
> * 本特性与其他MySQL加速特性兼容，特性之间的兼容性信息请参见《[特性之间的兼容性](https://www.hikunpeng.com/document/detail/zh/boostdb/compbf/kunpengdbsmysqlfeaturecompatibility_20_0001.html)》。
> * GCC编译时通过`-march=armv8-a+crc`编译选项指定ARM架构版本及扩展指令集，使能CRC32硬件指令。

## 快速安装

### 环境准备

在正式操作前，请确保软硬件满足以下要求：

* 硬件：鲲鹏服务器（鲲鹏920处理器）。

* 操作系统：openEuler 20.03 LTS SP1或openEuler 22.03 LTS SP1。操作系统镜像可从[openEuler 官网下载页](https://www.openeuler.org/zh/download/)获取。【对应的20.03版本的下载地址：[openEuler 20.03 LTS SP1下载 \| 下载中心 \| openEuler社区官网](https://www.openeuler.openatom.cn/zh/download/archive/detail/?version=openEuler%2020.03%20LTS%20SP1)；对应的22.03版本的下载地址：[openEuler 22.03 LTS SP1下载 \| 下载中心 \| openEuler社区官网](https://www.openeuler.openatom.cn/zh/download/archive/detail/?version=openEuler%2022.03%20LTS%20SP1)】

请按下表逐项检查环境：

|检查项|执行命令|预期输出|失败处理|
|---|---|---|---|
|CPU CRC32 指令支持检查|cat /proc/cpuinfo|回显Features行包含crc32，表示CPU支持CRC32硬件指令优化|更换为鲲鹏920处理器的服务器|
|OS 版本检查|cat /etc/*-release|openEuler 20.03 LTS SP1 或 22.03 LTS SP1|升级OS至符合要求的版本|
|Git 可用性检查|git --version|输出git版本号|执行yum install git安装|
|MySQL 编译环境检查|gcc -v && /usr/local/bin/cmake --version|GCC ≥ 5.3.0，CMake ≥ 3.4.3|请参见《[MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)》配置编译环境|

### 安装步骤

#### 获取软件包

|软件类型|必选/可选|软件包说明|软件包名称|获取链接|
|---|---|---|---|---|
|主要安装包|必选|MySQL源码包（includes Boost Headers）|mysql-boost-8.0.25.tar.gz|[MySQL 官网下载页](https://downloads.mysql.com/archives/community/)|
|特性补丁|必选|CRC32指令优化特性补丁，针对MySQL 8.0.25版本开发|0001-CRC32-AARCH64.patch|[CRC32指令优化特性补丁下载地址](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip)|

#### 应用补丁

针对MySQL的CRC32指令优化特性以补丁文件形式提供，需在MySQL源码中应用该补丁文件后再编译安装MySQL。

1. 将下载的MySQL源码包解压。

    ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    ```

2. 将补丁文件 `0001-CRC32-AARCH64.patch` 上传到解压后的MySQL源码目录（`/home/mysql-8.0.25`）下。

3. 在源码根目录，使用git初始化命令建立git管理信息。

    ```bash
    cd /home/mysql-8.0.25
    git init
    git add -A
    git commit -m "Initial commit"
    ```

    > ![说明](public_sys-resources/icon-note.gif) **说明**
    >
    > 若未配置git的提交用户信息，`git commit`前需要先配置用户邮件及用户名称：
    >
    > ```bash
    > git config user.email "123@example.com"
    > git config user.name "123"
    > ```

4. 在MySQL源码目录下执行以下命令，合入CRC32指令优化特性补丁。

    ```bash
    # 查看补丁文件的统计信息
    git apply --stat 0001-CRC32-AARCH64.patch
    # 检查补丁文件是否能够成功应用到当前的代码库中
    git apply --check 0001-CRC32-AARCH64.patch
    # 将补丁文件应用到当前的代码库中
    git apply 0001-CRC32-AARCH64.patch
    ```

    预期输出如下。

    ```output
    storage/innobase/ut/crc32.cc |   38 ++++++++++++++++++++++++++++++-----
    1 file changed, 35 insertions(+), 3 deletions(-)
    ```

#### 编译安装 MySQL

补丁应用合入后，编译安装MySQL，详细步骤请参见《[MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)》中的"编译和安装 MySQL"章节，核心命令如下。

```bash
cd /home/mysql-8.0.25
mkdir build
cd build
cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0
make -j 96
make -j 96 install
```

>![说明](public_sys-resources/icon-note.gif) **说明**
>
> `-j 96` 参数充分利用多核 CPU 优势加快编译速度，参数 `-j` 后数字为 CPU 核数，可用 `cat /proc/cpuinfo | grep processor | wc -l` 查看，此数值应小于等于 CPU 核数。

### 安装验证

执行如下命令，使用objdump（目标文件反汇编工具）检查mysqld二进制文件中是否已使能CRC32硬件指令（crc32cb 为ARM CRC32校验硬件指令的汇编助记符）。

```bash
objdump -d /usr/local/mysql/bin/mysqld | grep crc32cb
```

预期输出：返回如下crc32cb反汇编信息，表示CRC32指令优化特性已使能成功。

```output
  1df3e44:       1ac05060        crc32cb w0, w3, w0
  1df3e94:       1ac45044        crc32cb w4, w2, w4
  1df3f94:       1ac05040        crc32cb w0, w2, w0
  1df3fe4:       1ac35043        crc32cb w3, w2, w3
  1df4134:       1ac05040        crc32cb w0, w2, w0
```

## 快速开始

### A场景：Sysbench写场景性能对比验证

#### 场景描述

以Sysbench写场景压测为例（Sysbench是业内广泛使用的开源、跨平台多线程性能测试工具，可用于 MySQL 数据库性能测试），通过对"未合入 CRC32 补丁的 MySQL"与"已合入CRC32补丁的MySQL"分别执行相同模型的性能测试，展示使用CRC32指令优化特性前后的性能提升效果。

#### 操作步骤

1. 基础配置。

   分别准备两套MySQL环境（优化前版本与优化后版本），安装和运行MySQL的操作请参见《[MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)》，Sysbench工具的安装与环境调优请参见《[Sysbench 0.5&amp;1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)》。

2. 运行验证（优化前，基线版本）。

   以Sysbench 1.0 为例，先加载测试数据，再执行写场景测试：

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only run
    ```

    预期输出：取结果中的transactions（每秒事务数）和read/write requests（每秒读写请求数）作为测试指标，记录优化前的写场景性能基线。

3. 运行验证（优化后，合入CRC32补丁版本）。

   在合入CRC32指令优化特性补丁的MySQL环境上，使用完全相同的命令与数据规模重复步骤 2：

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only run
    ```

    预期输出：记录优化后的写场景性能数据。

#### 结果说明

使用CRC32指令优化特性后，Sysbench写场景性能从优化前的归一化值1.0提升到1.05，即性能提升约 5%。

文档中的IP地址、用户和密码仅为参考示例，请根据实际情况修改；详细的 Sysbench 测试步骤（数据加载、测试模型说明、环境调优、测试结果解读）请参见《[Sysbench 0.5&amp;1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)》。

**安全提示**：运行MySQL的参考用户密码`123456` 仅为演示用弱密码，生产环境请务必使用强密码，根据客户实际情况进行配置；执行测试 Sysbench 命令中的 `--mysql-password` 参数会在 shell 命令历史中留存明文密码，例如 ~/.bash\_history 等文件中，建议在测试完成后清理命令历史或改用专用测试账号。

### B场景：添加 LSE 编译选项优化多核锁竞争场景

#### 场景描述

以在多核、原子锁争抢严重的场景下进一步提升MySQL性能为例，通过在GCC编译选项中添加LSE（Large System Extensions，大系统扩展）相关选项，帮助用户体验 ARMv8.1 原子操作指令扩展带来的锁性能优化。

传统的LL/SC（Load\-link/Store\-condition）原子指令需要把共享变量先加载到本核所在的L1 Cache中进行修改，在锁竞争激烈时会导致系统性能下降严重。ARMv8.1规范引入的LSE将计算操作放到L3 Cache执行，增大数据共享范围，减少Cache一致性耗时，从而在锁竞争激烈时提升锁的性能。

#### 操作步骤

1. 修改编译选项。

    在MySQL源码的CMakeLists.txt文件中，将编译选项修改为：

    ```txt
    -march=armv8-a+lse
    ```

2. 安装。

    按照步骤重新编译安装MySQL，然后启动数据库并执行Sysbench压测。 

#### 结果说明

在锁竞争激烈的多核场景下，使用LSE编译选项后原子指令由LL/SC（ldaxr/stlxr）切换为LSE（ldaddal），锁竞争开销降低，系统性能得到提升。

## 常见问题（FAQ）

以下高频问题根据特性指南的操作约束整理，详细解决方案请参见对应文档。

**问题：执行 ****`git apply --check 0001-CRC32-AARCH64.patch`**** 报错，补丁无法合入**

答复：该补丁针对 MySQL 8.0.25 版本开发。请确认两点：一是使用的源码包为 MySQL 8.0.25 且解压后未做任何修改（建议在 `git init` 建立基线后再合入补丁）；二是补丁文件在源码根目录（`/home/mysql-8.0.25`）下执行合入命令。若源码已被修改，请重新解压干净的源码包后重试。

**问题：`cat /proc/cpuinfo`** 的Features行中没有crc32 

答复：表示当前CPU不支持CRC32硬件指令优化，本特性无法使能。请更换为搭载鲲鹏920处理器的服务器。

**问题：编译安装后执行 ****`objdump -d /usr/local/mysql/bin/mysqld | grep crc32cb`**** 无任何输出**

答复：表示CRC32指令优化特性未使能成功。请检查：补丁是否已成功合入（可查看 `storage/innobase/ut/crc32.cc` 是否被修改）；编译是否在合入补丁后的源码上重新执行。确认后重新编译安装。

**问题：执行git命令提示命令不存在**

答复：缺少git工具。请先参见《[MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)》中配置Yum源相关内容，再执行 `yum install git` 安装。

## 更多功能

关于CRC32指令优化特性和MySQL加速套件的更多使用方式请参见以下文档：

* [CRC32 指令优化 特性指南](https://www.hikunpeng.com/document/detail/zh/boostdb/mysql/basic_computation_opt/docs/zh/crc32_instruction_optimization_feature_guide.md)

* [MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

* [Sysbench 0.5&amp;1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

* [MySQL 8.0.x 调优指南（数据库参数调优、操作系统调优）](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0013.html)

## 修订记录

|文档版本 | 发布日期 | 修改说明 |
| --------| -------- | -------- |
|01| 2026-09-30 | 第一次正式发布。 |
