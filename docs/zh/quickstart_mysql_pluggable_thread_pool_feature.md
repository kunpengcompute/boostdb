# MySQL可插拔线程池特性快速入门

本文以MySQL 8.0.25为例，指导用户通过"获取补丁、合入补丁、编译安装、安装插件、验证使用"的流程，快速上手可插拔线程池特性。根据特性指南中基于BenchMarkSQL的TPC\-C（数据库事务处理性能基准测试模型）测试结果：MySQL运行10000个并发任务时，启用线程池前性能只有原来的10%左右。启用线程池后性能可维持在85%对于大量连接的OLTP（Online Transaction Processing，联机事务处理）短查询场景收益最大，大量连接的只读短查询也有明显收益。

## 特性介绍

MySQL可插拔线程池特性是鲲鹏BoostKit MySQL加速套件的性能优化特性之一，用于解决默认MySQL连接器在高连接数下的性能劣化问题。在默认的MySQL连接器下，每个接入的连接都会分配一个线程，当连接数非常大时，线程的上下文切换及线程之间热锁的竞争将占用大量CPU资源，导致服务性能下降。

使用线程池连接器模块后，连接的建立与调度由线程池接管：整个线程池分为若干个线程组（默认为CPU逻辑核数）和一个timer线程，通过每个分组上的listener线程借助epoll（Linux内核的事件通知机制，用于高效侦听大量网络连接的可读事件）进行网络任务侦听。将任务放入高优先级队列或普通队列，由空闲的worker线程按优先级取出处理，并动态伸缩worker线程数量，使服务器在大量客户端连接下仍保持最佳性能。该特性还支持事务优先调度、防线程池停滞，并在information\_schema中新增四张状态信息表，可实时监管线程池状态。

>![说明](public_sys-resources/icon-note.gif) **说明**
>
> * 本特性以patch文件方式提供，补丁`code-threadpool-for-MySQL-8.0.patch`同时适用于MySQL 8.0.25、8.0.30和8.0.35。
> * 本特性与MySQL NUMA调度优化patch不冲突，但使用线程池插件特性后会使MySQL NUMA调度优化中的用户连接线程调度优化失效。
> * 线程池依赖numactl库，若编译时未安装numactl依赖，安装线程池插件时会提示"undefined symbol: numa\_xxxxx"错误。

## 安装MySQL源码

### 环境准备

在正式操作前，请确保软硬件满足以下要求

|环境|要求|
|---|---|
|硬件|鲲鹏服务器（鲲鹏920系列处理器）。进行性能测试时，数据目录需使用单独硬盘；非性能测试时直接在系统盘上建数据目录即可。|
|操作系统|openEuler 20.03 LTS SP1或openEuler 22.03 LTS SP1（for ARM）。若需全新安装操作系统，可选择"Minimal Install"安装方式并勾选Development Tools套件。|

请按下表逐项检查环境。

|检查项|执行命令|预期输出|失败处理|
|---|---|---|---|
|OS版本检查|cat /etc/*-release|openEuler 20.03 LTS SP1 或 22.03 LTS SP1|升级OS至符合要求的版本|
|处理器检查|lscpu|架构为aarch64（鲲鹏920系列）|更换为鲲鹏服务器|
|CMake版本检查|cmake --version|版本高于3.7（openEuler 20.03 为 3.7.2，22.03自带3.11.4）|升级CMake|
|GCC版本检查|gcc -v|openEuler 20.03 为 7.3.0，22.03自带10.3.1|升级GCC，参见《MySQL 移植指南》|
|numactl依赖检查|rpm -qa \| grep numactl|已安装numactl及numactl\-devel|执行yum install -y numactl numactl-devel*安装|
|Git可用性检查|git --version|输出git版本号|配置Yum源后执行yum install git安装|

### 安装步骤

#### 获取软件包

|软件类型|必选/可选|软件包说明|软件包名称|获取链接|
|---|---|---|---|---|
|主要安装包|必选|MySQL 源码包（includes Boost Headers）|mysql-boost-8.0.25.tar.gz|[MySQL 官网下载页](https://downloads.mysql.com/archives/community/)|
|特性补丁|必选|线程池特性补丁包（含 code\-threadpool\-for\-MySQL\-8.0.patch，适用于 8.0.25/8.0.30/8.0.35）|boostdb-patch-release-20260330.zip|[BoostDB 补丁发布页](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip)|

若服务器可访问外网，可直接执行以下命令下载。

```bash
cd /home
wget https://cdn.mysql.com/archives/mysql-8.0/mysql-boost-8.0.25.tar.gz --no-check-certificate
wget https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip --no-check-certificate
```

#### 合入线程池特性补丁

1. 解压MySQL源码包并进入源码根目录。

   ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    cd mysql-8.0.25
    ```

2. 解压补丁包，并将`code-threadpool-for-MySQL-8.0.patch`放置在MySQL源码根目录下。

    ```bash
   unzip boostdb-patch-release-20260330.zip
   ```

3. 在源码根目录，使用git初始化命令建立git管理信息。

    ```bash
    git init
    git add -A
    git commit -m "Initial commit"
    ```

    >![说明](public_sys-resources/icon-note.gif) **说明**
    >
    > 若未配置git的提交用户信息，`git commit`前需要先配置。
    >
    > ```bash
    > git config user.email "123@example.com"
    > git config user.name "123"
    > ```

4. 合入线程池特性补丁。

    ```bash
    git apply --check boostdb-patch-release-20260330/code-threadpool-for-MySQL-8.0.patch
    git apply --whitespace=nowarn boostdb-patch-release-20260330/code-threadpool-for-MySQL-8.0.patch
    ```

5. 确认补丁合入成功，查看新增的thread\_pool目录。

    ```bash
    ll ./plugin/thread_pool/
    ```

    预期输出：可看到thread\_pool目录下新增的源码文件。

#### 编译安装MySQL

按照正常的MySQL源码编译安装步骤进行编译安装，详细操作请参见《MySQL 移植指南》中的"编译和安装MySQL"章节，以下以96核为例，核心命令如下。

```bash
cd /home/mysql-8.0.25
mkdir build
cd build
cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0
make -j 96
make -j 96 install
```

> ![引出说明信息的图标](public_sys-resources/icon-note.gif)**说明**：
>
> `-j 96`参数充分利用多核CPU优势加快编译速度，参数`-j`后数字为CPU核数，可用`cat /proc/cpuinfo | grep processor | wc -l`查看，此数值应小于等于CPU核数。

编译安装完成后，初始化并启动数据库，核心步骤概览如下，详细操作说明请参见《MySQL 移植指南》的"运行MySQL"章节。

1. 修改配置文件my.cnf（配置basedir、datadir、socket、port、user等参数，路径根据实际情况修改）。

2. 配置环境变量，将MySQL二进制文件路径（"/usr/local/mysql/bin"）加入PATH并使其生效。

3. 初始化数据库（回显中`root@localhost:`后会生成临时初始密码，请注意保存）。

   ```bash
   mysqld --defaults-file=/etc/my.cnf --initialize
   ```

4. 启动数据库服务。

   ```bash
    service mysql start
    ```

#### 验证编译产物

确认编译产物中已生成线程池插件动态库。

```bash
ls $CMAKE_INSTALL_PREFIX/lib/plugin/thread_pool.so
```

预期输出：存在`thread_pool.so`文件，表示线程池插件已随编译生成。

## 使用MySQL可插拔线程池

### 安装并使能线程池插件

#### 场景描述

默认情况下MySQL使用每线程每连接的连接器。以通过SQL命令在线安装线程池插件并验证其生效为例，帮助用户快速完成线程池连接器的启用。安装成功后，线程池插件连接器用于处理新建连接的查询请求，原始默认连接器仍保留并处理安装前的连接。

#### 操作步骤

1. 登录数据库。

   ```bash
   mysql -uroot -p -S /data/mysql/run/mysql.sock
   ```

2. 安装线程池插件。

   thread\_pool.so 中包含 5 个插件：thread\_pool 是线程池连接器插件，THREAD\_POOL\_GROUPS、THREAD\_POOL\_QUEUES、THREAD\_POOL\_STATS、THREAD\_POOL\_WAITS 是线程池状态监管表插件。执行以下 SQL 语句安装，安装后立即生效：

   ```sql
   INSTALL PLUGIN thread_pool SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_GROUPS SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_QUEUES SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_STATS SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_WAITS SONAME "thread_pool.so";
   ```

   预期输出结果如下。

   ```output
   Query OK, 0 rows affected (0.01 sec)
   ```

3. 验证插件安装结果。

   ```sql
   show plugins;
   ```

   预期输出：回显信息中thread\_pool插件状态为"ACTIVE"，表示插件已安装成功。

4. 查看线程池运行状态。

   通过 information\_schema 中新增的状态表实时监管线程池状态，例如查看线程组信息。

   ```sql
   select * from information_schema.THREAD_POOL_GROUPS;
   ```

   预期输出：返回各线程组的连接数、线程数、队列深度等状态信息。

#### 结果说明

线程池插件状态为ACTIVE且状态表可正常查询，表示线程池连接器已生效，后续新建连接将由线程池接管调度。

> ![引出说明信息的图标](public_sys-resources/icon-note.gif)**说明**
>
> * 也可以在MySQL的配置文件my.cnf中添加`plugin-load-add=thread_pool.so`安装插件，该方式需要重启数据库才能生效。
> * 若编译后将thread\_pool.so拷贝到其他MySQL服务使用，需将文件放入目标服务plugin\_dir变量指定的目录下，再执行上述INSTALL PLUGIN语句。
> * 如需卸载线程池插件，请参见《特性指南》中的"卸载线程池插件"章节。

### 调整thread\_pool\_size优化峰值性能

#### 场景描述

以OLTP场景下调整线程组数量优化峰值性能为例，帮助用户体验线程池核心参数调优。对于OLTP场景，线程池配置参数推荐使用默认配置，可通过调整thread\_pool\_size（线程组数量）的值来优化峰值性能。

#### 操作步骤

1. 查看当前线程组数量。

   ```sql
   show variables like 'thread_pool_size';
   ```

   预期输出结果如下。

   ```output
   +------------------+-------+
   | Variable_name    | Value |
   +------------------+-------+
   | thread_pool_size | 96    |
   +------------------+-------+
   ```

   默认值表示线程组数与CPU核数一致。

2. 动态调整线程组数量。

   对于连接数超过CPU逻辑核数、性能瓶颈不在锁争用且CPU压不满的场景，可将线程组数设置为1至3倍CPU核数或最优并发数，以获取更佳性能。

   ```sql
   set global thread_pool_size=128;
   ```

   预期输出结果如下。

   ```output
   Query OK, 0 rows affected (0.00 sec)
   ```

#### 结果说明

thread\_pool\_size支持命令行、配置文件和动态修改（Global范围，允许值1至1024）。调整后线程池按新的线程组数量调度连接，在TPC\-C大量并发场景下，启用线程池后性能可维持在原来的85%（未启用时仅剩约10%）。性能验证的详细测试步骤请参见《[BenchMarkSQL测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengbenchmarksql_06_0001.html)》。

> ![引出说明信息的图标](public_sys-resources/icon-note.gif)**说明**
>
> MySQL的配置参数（系统变量）可用于调整数据库服务的功能和性能，详细信息请参见MySQL官方文档《Server System Variables》。

## 常见问题

以下高频问题根据特性指南的操作约束整理，详细解决方案请参见对应文档。

**问题**：安装线程池插件时提示"undefined symbol: numa\_xxxxx"错误怎么办？

**答复**：`undefined symbol`是典型的**运行时动态链接错误**，原因是运行时`mysqld`找不到`libnuma.so`库文件。可能是编译MySQL时未安装numactl依赖，导致编译出的MySQL程序未加载numa库。请先执行`yum install -y numactl numactl-devel*`安装依赖，再重新编译安装MySQL。

**问题**：执行`git apply --check`检查补丁时报错，补丁无法合入怎么办？

**答复**：`code-threadpool-for-MySQL-8.0.patch`适用于MySQL 8.0.25、8.0.30和8.0.35。请确认使用的源码版本在支持范围内且解压后未做修改（建议先执行`git init`，建立基线再合入补丁）；若源码已被修改，请重新解压干净的源码包后重试。如果仍失败，可尝试在git apply命令后添加`--ignore-whitespace`参数，忽略空白符差异后重试。

**问题**：执行INSTALL PLUGIN后，通过`show plugins`查看插件状态不是ACTIVE怎么办？

**答复**：请确认thread\_pool.so文件位于目标MySQL服务plugin\_dir变量指定的目录下（可通过`show variables like 'plugin_dir';`查询）。若文件缺失，请从编译路径下的 plugin\_output\_directory/目录拷贝thread\_pool.so后重新安装。此外，确认 `thread_pool.so`文件对MySQL用户（通常是mysql）有读取权限（chmod +rx）。同时，检查MySQL错误日志（error.log），通常会有更详细的加载失败原因。

**问题**：卸载线程池插件时提示"plugin is busy"警告怎么办？

**答复**：表示线程池连接器上仍有用户连接，插件状态会由ACTIVE变为DELETE（卸载过程状态）。先断开业务连接，或者通过` show processlist `确认无连接依赖线程池，然后先卸载线程池连接器插件（`UNINSTALL PLUGIN thread_pool`），此时由于没有连接，会立即完成。之后再卸载那4张状态表插件。

## 更多功能

关于可插拔线程池特性的更多使用方式（配置参数全量说明、优先Session处理、NUMA亲和、连接数均衡、卸载插件等）请参见以下文档

- [MySQL 8.0.25&amp;8.0.30&amp;8.0.35 可插拔线程池 特性指南](https://www.hikunpeng.com/document/detail/zh/boostdb/mysql/pluggablethreadpool/docs/zh/mysql_8_0_25_8_0_30_and_8_0_35_pluggable_thread_pool_feature_guide.md)

- [MySQL 8.0.20 线程池 特性指南](https://www.hikunpeng.com/document/detail/zh/boostdb/mysql/pluggablethreadpool/docs/zh/mysql_8_0_20_thread_pool_feature_guide.md)

- [MySQL 5.7.27 线程池 特性指南](https://www.hikunpeng.com/document/detail/zh/boostdb/mysql/pluggablethreadpool/docs/zh/mysql_5_7_27_thread_pool_feature_guide.md)

- [MySQL 移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

- [Sysbench 0.5&amp;1.0 测试指导](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

- [BenchMarkSQL 测试指导（TPC\-C）](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/testguide/tstg/kunpengbenchmarksql_06_0001.html)

## 修订记录

|文档版本|发布日期|修改说明|
|---|---|---|
|01|2026-09-30|第一次正式发布。|
