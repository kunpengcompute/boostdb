# 快速入门：在鲲鹏服务器上部署MySQL

本文档旨在帮助开发者快速在鲲鹏ARM架构服务器上完成MySQL的部署与基础配置。内容涵盖获取软件包，配置编译环境，执行安装，安装验证等关键步骤。同时提供初始化并运行MySQL数据库和创建数据库并执行基本SQL操作的快速开始场景，适合需要快速搭建测试环境或生产基座的开发人员参考。

## 组件介绍

MySQL是一个开源的关系型数据库管理系统（RDBMS），是业界最流行的关系数据库之一，尤其在Web应用方面被广泛使用，开发语言为C/C++。MySQL将数据保存在不同的表中，而非将所有数据放在一个大仓库内，因此速度快、灵活性高；其体积小、总体拥有成本低，且开放源码，一般中小型网站的开发都选择MySQL作为网站数据库。MySQL所使用的SQL语言是用于访问数据库的最常用标准化语言。

本文以MySQL 8.0.25为例，指导用户在鲲鹏服务器（aarch64架构）上通过源码编译方式快速完成MySQL的安装、初始化与首次使用。MySQL 5.7.27、MySQL 8.0.17及其他MySQL 8.0.x版本也可参考本文档。

>![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
>
> * 使用开源软件时需遵守开源软件的许可协议。
> * MySQL 8.0.16版本需自行通过官网补丁升级，建议直接使用MySQL 5.7.27、MySQL 8.0.17或MySQL 8.0.25及以上版本。
> * 关于MySQL的更多信息请访问[MySQL官网](https://www.mysql.com/)。

## 快速安装

### 环境准备

在正式操作前，请确保软硬件满足以下要求：

* 硬件：鲲鹏服务器（鲲鹏920处理器）。进行性能测试时，数据目录需使用单独硬盘（一个系统盘、一个数据盘）；非性能测试时直接在系统盘上建数据目录即可。

* 操作系统：openEuler 20.03 LTS SP1、openEuler 22.03 LTS SP1或CentOS 7.6。若需全新安装操作系统，建议选择“Minimal Install”安装方式并勾选Development Tools套件，否则很多软件包需要手动安装。

已验证的MySQL版本与操作系统版本组合如下：

|MySQL版本|操作系统版本|
|---|---|
|MySQL 5.7.27、MySQL 8.0.17|CentOS 7.6、openEuler 20.03 LTS SP1|
|MySQL 8.0.25|openEuler 20.03 LTS SP1、openEuler 22.03 LTS SP1|

### 安装步骤

#### 获取软件包

|软件类型|必选/可选|软件包说明|软件包名称|获取链接|
|---|---|---|---|---|
|主要安装包|必选|MySQL源码包（includes Boost Headers，编译必须包含Boost头文件）|mysql-boost-8.0.25.tar.gz|[MySQL官网下载页](https://downloads.mysql.com/archives/community/)|
|编译工具|可选|CMake ≥ 3.4.3（OS自带版本不满足要求时需要）|cmake-3.5.2.tar.gz|[CMake官网](https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz)|
|编译工具|可选|GCC ≥ 5.3.0（CentOS 7.6需要升级，openEuler不需要）|gcc-7.3.0.tar.gz|[GNU镜像站](https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz)|

若服务器可访问外网，可直接下载MySQL源码包：

```bash
cd /home
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-8.0.25.tar.gz --no-check-certificate
```

#### 配置编译环境

1. 创建mysql用户组和用户（操作系统层面，用于权限隔离）：

   ```bash
    groupadd mysql
    useradd -g mysql mysql
    passwd mysql
    ```

2. 创建数据目录（非性能测试场景）：

    ```bash
    mkdir -p /data/mysql
    cd /data/mysql
    mkdir data tmp run log relaylog
    chown -R mysql:mysql /data
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    > 进行性能测试时，需将单独磁盘格式化并挂载到数据目录，详细操作参见[搭建数据盘](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0007.html)。

3. 安装依赖包：

    ```bash
    yum -y install bison bison-devel ncurses ncurses-devel libaio-devel openssl openssl-devel gmp gmp-devel mpfr mpfr-devel libmpc libmpc-devel libcurl-devel
    yum -y install wget tar gcc gcc-c++ git rpcgen cmake make
    yum -y install numactl numactl-devel*
    yum -y install libtirpc libtirpc-devel m4
    yum -y install zstd-devel
    ```

4. 若CMake或GCC版本不满足要求，请先升级（openEuler系统自带GCC满足要求，可跳过GCC升级）：

    * 升级CMake至3.4.3或以上（本文以3.5.2为例）：[升级CMake](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0006.html)

    * CentOS 7.6升级GCC至5.3.0或以上（本文以7.3.0为例）：[安装GCC](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0007.html)

#### 执行安装

1. 解压源码包并进入源码目录，创建编译目录：

    ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    cd /home/mysql-8.0.25
    mkdir build
    ```

2. 进入编译目录，配置MySQL：

    ```bash
    cd build
    cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0
    ```

    关键参数说明：

    |参数|说明|
    |---|---|
    |DBUILD_CONFIG|设置为mysql\_release，表示CMake编译参数采用MySQL官方发布release版本时的编译参数。|
    |DCMAKE_INSTALL_PREFIX|指定软件的安装路径，本文安装路径为"/usr/local/mysql"，请根据实际情况配置。|
    |DMYSQL_DATADIR|创建数据库时数据文件存放的路径，本文路径为"/data/mysql/data"。|
    |DWITH_BOOST|解压MySQL源码包后，解压文件中boost\_1\_73\_0文件夹所在路径。本文解压到"/home"目录下，则路径为"/home/mysql\-8.0.25/boost/boost\_1\_73\_0"。|

3. 编译安装MySQL：

    ```bash
    make -j 96
    make -j 96 install
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    > `-j 96`参数充分利用多核CPU优势加快编译速度，参数`-j`后数字为CPU核数，可用`cat /proc/cpuinfo | grep processor | wc -l`查看，此数值应小于等于CPU核数。
    > 如果编译安装过程失败，需要执行`rm -rf /home/mysql-8.0.25`清理环境后，再重新解压MySQL源码包并编译安装。

### 安装验证

1. 查看MySQL安装目录：

    ```bash
    ls /usr/local/mysql/
    ```

    预期输出：

    ```output
    bin  docs  include  lib  LICENSE  LICENSE.router  LICENSE-test  man  mysqlrouter-log-rotate  mysql-test  README  README.router  README-test  run  share  support-files  var
    ```

2. 查看数据库版本：

    ```bash
    /usr/local/mysql/bin/mysql --version
    ```

    预期输出：

    ```output
    /usr/local/mysql/bin/mysql  Ver 8.0.25 for Linux on aarch64 (Source distribution)
    ```

## 快速开始

### A场景：初始化并运行MySQL数据库

#### 场景描述

以MySQL 8.0.25编译安装完成为起点，通过修改配置文件、设置环境变量、初始化数据库、启动数据库并完成首次登录，帮助用户快速完成MySQL在鲲鹏服务器上的首次运行。

#### 操作步骤

1. 修改配置文件。

    ```bash
    rm -f /etc/my.cnf
    echo -e "[mysqld_safe]\nlog-error=/data/mysql/log/mysql.log\npid-file=/data/mysql/run/mysqld.pid\n[mysqldump]\nquick\n[mysql]\nno-auto-rehash\n[client]\ndefault-character-set=utf8\n[mysqld]\nbasedir=/usr/local/mysql\nsocket=/data/mysql/run/mysql.sock\ntmpdir=/data/mysql/tmp\ndatadir=/data/mysql/data\ndefault_authentication_plugin=mysql_native_password\nport=3306\nuser=mysql\n" > /etc/my.cnf
    ```

    ```bash
    cat /etc/my.cnf
    chown mysql:mysql /etc/my.cnf
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    > 其中文件路径（包括软件安装路径basedir、数据路径datadir等）请根据实际情况修改。`user=mysql`指操作系统层的用户，即2.2.2章节创建的用户。如需进行性能调测，请参考《数据库参数调优》修改my.cnf中的配置参数。

2. 初始化数据库。

    ```bash
    chmod 755 /data/mysql/data/
    su - mysql
    mysqld --defaults-file=/etc/my.cnf --initialize
    ```

    预期输出（部分）：

    ```output
    [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: t<7Xtw3adX>B
    [System] [MY-013170] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.25) initializing of server has completed
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    >
    > * 初始化回显`root@localhost:`后面有生成的临时初始密码，请注意保存，步骤4中会用到。
    > * 如果初始化失败，提示"\-\-initialize specified but the data directory has files in it."，则执行`ls /data/mysql/data`确认后，执行`rm -rf /data/mysql/data/*`删除数据并重新初始化。

3. 启动数据库。

    ```bash
    /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf &
    ```

    查看数据库进程和监听端口：

    ```bash
    ps -ef | grep mysql
    netstat -anpt | grep 3306
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    >
    > * 如果以root用户第一次执行`service mysql start`启动失败（提示缺少mysql.log文件），请先切换到mysql用户（`su - mysql`）启动数据库服务生成mysql.log，停止服务后再以root用户启动。
    >* 如果netstat命令执行失败，需先执行`yum -y install net-tools`安装依赖包。

4. 登录数据库并修改初始密码。

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    提示输入密码时，输入步骤2生成的临时初始密码。登录成功后修改root用户密码并创建全域root用户（允许root从其他服务器访问）：

    ```sql
    alteruser 'root'@'localhost' identified by '123456';
    create user 'root'@'%' identified by '123456';
    grant all privileges on *.* to 'root'@'%';
    flush privileges;
    ```

    预期输出：

    ```output
    Query OK, 0 rows affected (0.01 sec)
    ```

    >![引出说明信息的图标](public_sys-resources/icon-note.gif) **说明：**
    > 文档中的用户和密码只是参考，请根据客户实际情况进行配置。
    > **安全提示**：`123456`仅为演示用弱密码，生产环境请务必使用强密码（建议12位以上，包含大小写字母、数字和特殊字符）；同时应避免在命令行参数中直接携带明文密码（如`mysql -uroot -p123456`），以防密码留存在shell命令历史中被泄露。

5. 用新密码重新登录验证。

    执行`exit`退出数据库，然后重新登录：

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    预期输出（部分）：

    ```output
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 8
    Server version: 8.0.25
    ```

#### 结果说明

使用新密码成功登录数据库，出现`mysql>`命令行提示符，说明MySQL已完成初始化并正常运行，数据库监听在3306端口。

> **须知**
> 关闭数据库请使用与启动方式一一对应的命令。以`service mysql start`启动的数据库，使用`service mysql stop`关闭（重启可执行`service mysql restart`）；以mysqld或mysqld\_safe方式启动的数据库，可执行`mysqladmin -uroot -p123456 shutdown -S /data/mysql/run/mysql.sock`关闭。

### B场景：创建数据库并执行基本SQL操作

#### 场景描述

以创建一个测试数据库并执行基本的建表、插入、查询操作为例，帮助用户快速体验MySQL的基本使用流程。

#### 操作步骤

1. 登录数据库。

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

2. 创建数据库和数据表。

    ```sql
    create database testdb;
    use testdb;
    create table users (id int primary key auto_increment, name varchar(32) not null, created_at timestamp default current_timestamp);
    ```

    预期输出：

    ```output
    Query OK, 1 row affected (0.01 sec)
    Database changed
    Query OK, 0 rows affected (0.02 sec)
    ```

    建表语句中各字段属性说明：`id int primary key auto_increment`表示整型主键并自动递增；`name varchar(32) not null`表示最长32字符的字符串且不允许为空；`created_at timestamp default current_timestamp`表示时间戳字段，默认取当前时间。

3. 插入并查询数据。

    ```sql
    insert into users (name) values ('kunpeng');
    select * from users;
    ```

    预期输出：

    ```output
    Query OK, 1 row affected (0.00 sec)

    +----+---------+---------------------+
    | id | name    | created_at          |
    +----+---------+---------------------+
    |  1 | kunpeng | 2026-08-27 10:00:00 |
    +----+---------+---------------------+
    1 row in set (0.00 sec)
    ```

#### 结果说明

数据成功写入并可正常查询，说明MySQL的建库、建表、插入、查询等基本功能在鲲鹏服务器上运行正常。

## 常见问题（FAQ）

以下高频问题整理自移植指南故障排除章节，详细解决方案请参见对应链接文档。

**问题1：初始化数据库时提示"\-\-initialize specified but the data directory has files in it."怎么办？**

答复：数据目录中已存在文件导致初始化失败。执行`ls /data/mys问题l/data`确认目录内容后，执行`rm -rf /data/mysql/data/*`清空数据目录，再重新执行初始化命令。

**问题2：以root用户执行`service mysql start`首次启动失败，提示缺少mysql.log文件怎么办？**

答复：先切换到mysql用户（`su- mysql`）启动数据库服务，系统会在"/data/mysql/log"目录下生成mysql.log文件；然后停止数据库服务（`/usr/local/mysql/bin/mysqladmin -u root -p shutdown`），再次以root用户启动即可。

**问题3：执行netstat命令提示命令不存在怎么办？**

答复：缺少net\-tools依赖包，执行`yum-y install net-tools`安装后重试。

**问题4：升级CMake后，查询版本发现CMake版本未生效怎么办？**

答复：参见故障排除文档[升级CMake后CMake版本未生效的解决方法](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0010.html)。

**问题5：编译安装MySQL过程中执行cmake命令失败怎么办？**

答复：参见故障排除文档[编译安装MySQL过程中执行CMake命令失败的解决方法](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0011.html)。

## 5. 更多功能

关于MySQL在鲲鹏服务器上的更多场景使用方式请参见以下文档：

* [MySQL移植指南（含编译故障排除）](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

* [升级CMake后CMake版本未生效的解决方法](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0010.html)

* [编译安装MySQL过程中执行CMake命令失败的解决方法](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0011.html)

* [MySQL迁移最佳实践：从x86迁移到鲲鹏并保持主从复制](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengdbs_mysql_bestpractices_21_0001.html)

* [MySQL视频帮助](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/video_004.html)

* [MySQL官网](https://www.mysql.com/)

## 修订记录

|文档版本 | 发布日期 | 修改说明 |
| --------| -------- | -------- |
|01| 2026-09-30 | 第一次正式发布。 |