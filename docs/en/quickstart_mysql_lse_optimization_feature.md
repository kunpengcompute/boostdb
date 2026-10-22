# Quick Start: MySQL LSE Optimization Feature

<!-- md-trans-meta sourceCommit=2cdb5fa93a6c147cfea1a61507642c0535dc21c6 translatedAt=2026-10-20T06:48:49.831Z pushedAt=2026-10-20T09:42:35.376Z nl=2 -->

The MySQL Large System Extensions (LSE) optimization feature is a basic computation optimization function provided by the Kunpeng BoostKit MySQL acceleration kit for Kunpeng server OLTP scenarios. This feature introduces the atomic operation hardware acceleration solution of the Armv8.1 specification to move computation operations to the L3 cache for execution, effectively alleviating the linear performance degradation caused by intense lock contention in high-concurrency read/write environments. In sysbench tests with the 8U16G specification (8 vCPUs and 16 GB memory) and 256 concurrent threads, enabling this feature improves overall MySQL performance (read-only, read-write, and write-only) by about 5%.

## Component Introduction

The MySQL LSE optimization feature is one of the basic computation optimization features of the Kunpeng BoostKit MySQL acceleration kit, used to alleviate the linear performance degradation caused by intense lock contention under high-concurrency read/write in MySQL OLTP scenarios on Kunpeng servers.

LSE is an atomic operation hardware acceleration solution designed specifically for modern multi-core, high-concurrency environments. Through single-instruction atomic operations, it resolves the scalability bottleneck of Load-link/Store-conditional (LL/SC) instructions. LL/SC atomic instructions need to load shared variables into the L1 cache where the current core is located for modification. The performance is good when there is little lock contention. Under intense lock contention, system performance deteriorates severely. The Armv8.1 specification introduces LSE, which puts computation operations into the L3 cache for execution to increase the scope of data sharing, reduce cache-coherence overhead, and improve operation efficiency when lock contention is intense.

This document uses Percona-Server 5.7.44-53 as an example to describe how to obtain the RPM package, perform the installation, start the database, and conduct performance comparison and verification, helping users quickly get started with the MySQL LSE optimization feature. In sysbench tests with the 8U16G specification and 256 concurrent threads, enabling this feature improves overall MySQL performance (read-only, read-write, and write-only) by about 5%.

> ![NOTE icon](public_sys-resources/icon-note.gif)**NOTE:**
>
> * Percona Server is a standalone database product fully compatible with MySQL. This feature is provided in the form of a BoostDB RPM package, and LSE optimization is enabled by default after installation.
> * In multi-core scenarios with severe atomic lock contention, you can also add `-march=armv8-a+lse` to GCC compilation options to enable the LSE feature at the source code level.

## Quick Installation

### Environment Setup

Before performing operations, ensure that the hardware and software meet the following requirements:

* Hardware: New Kunpeng 920 processor model or Kunpeng 950 processor.

* OS: openEuler 22.03 LTS SP4 or openEuler 24.03 LTS SP3.

Check the environment item by item according to the following table:

|Check Item|Command|Expected Output|Failure Handling|
|---|---|---|---|
|CPU model check|lscpu|New Kunpeng 920 processor model or Kunpeng 950 processor|Replace it with a server that meets the requirements.|
|OS version check|cat /etc/*-release|openEuler 22.03 LTS SP4 or 24.03 LTS SP3|Upgrade the OS to a version that meets the requirements.|
|Compilation dependency check|See the "Configuring the Compilation Environment" section in the *Percona Porting Guide*|Dependency packages installed|Install dependencies according to the porting guide.|

### Installation Procedure

#### Obtaining the Software Package

|Software Type|Required/Optional|Package Description|Package Name|How to Obtain|
|---|---|---|---|---|
|Main installation package|Required|BoostDB Percona RPM package (AArch64) with built-in LSE optimization feature|BoostDB-Percona-5.7.44-53.aarch64.rpm|[Percona-Server 5.7.44-53](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|Baseline installation package|Required (for baseline testing)|Percona Server 5.7.44-53 baseline RPM package (AArch64) without built-in LSE optimization feature, used for baseline comparison with BoostDB-Percona-5.7.44-53|BoostDB-Percona-5.7.44-53.aarch64.rpm|[Percona-Server 5.7.44-53 baseline RPM](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-5.7.44-53-v3/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|Main installation package|Optional (alternative)|8.0 version RPM package|BoostDB-Percona-8.0.43-34.aarch64.rpm|[Percona-Server 8.0.43-34](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/BoostDB-Percona-8.0.43-34.aarch64.rpm)|
|Baseline installation package|Optional (alternative)|Percona Server 8.0.43-34 baseline RPM package (AArch64) without built-in LSE optimization feature, used for baseline comparison with BoostDB-Percona-8.0.43-34|BoostDB-Percona-8.0.43-34.aarch64.rpm|[Percona-Server 8.0.43-34 baseline RPM](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-8.0.43-34-v2/BoostDB-Percona-8.0.43-34.aarch64.rpm)|

#### Performing Installation

1. Install the dependencies as instructed in the "Configuring the Compilation Environment" section in the *Percona Porting Guide*.

2. Store the downloaded RPM package in the target path (for example, `/home`), and run the following commands to install it.

    ```bash
    cd /home
    rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm
    ```

    After the installation is complete, the default installation directory is `/usr/local/mysql`.

    > ![NOTE](public_sys-resources/icon-note.gif) **NOTE:**
    >
    > If dependencies have been installed but the RPM-related check fails, use `--nodeps` to skip the dependency check.
    >
    > ```bash
    > rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps
    > ```

#### Starting the Database

The core steps for starting the database are summarized as follows. For detailed operation instructions (including the expected output and precautions of each step), see the "Running MySQL" section in the *MySQL Porting Guide*.

1. Modify the configuration file `my.cnf` (configure parameters such as `basedir`, `datadir`, `socket`, `port`, and `user`; modify the paths as required).

2. Configure the environment variable: add the MySQL binary file path (`/usr/local/mysql/bin` by default) to `PATH`, and make it effective.

3. Initialize the database.

    ```bash
    mysqld --defaults-file=/etc/my.cnf --initialize
    ```

    In the initialization command output, a temporary initial password is generated following `root@localhost:`. You need to save it, as it is required for login.

4. Start the database service.

    ```bash
    service mysql start
    ```

5. Log in to the database using the temporary initial password and change the password.

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    > **NOTE:**
    > The installation path and socket path in the preceding commands are subject to the actual configuration in `my.cnf`. If initialization or startup fails, see the "FAQs" section.

### Installation Verification

1. Check the database process.

    ```bash
    ps -ef | grep mysql
    ```

    Expected output: The mysqld process is displayed, indicating that the database service has been started.

2. Log in to the database.

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    The expected output (partial) is as follows:

    ```output
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 8
    ```

    The `mysql>` command-line prompt appears, indicating that the database has been installed and is running successfully.

## Quick Start

### Scenario A: Overall High-Concurrency sysbench Performance Comparison and Verification

#### Scenario Description

Take sysbench stress tests with the 8U16G specification and 256 concurrent threads as an example (sysbench is a widely used open-source, cross-platform, multi-threaded performance testing tool in the industry, and can be used for MySQL database performance testing). Performance tests based on read-only, read-write, and write-only models are executed on MySQL without the LSE feature enabled and BoostDB Percona with the LSE feature enabled respectively, to demonstrate the performance improvement before and after the LSE optimization feature is used.

#### Procedure

1. Basic configuration: Prepare two MySQL environments (one with the feature disabled and the other with it enabled). For details about the sysbench tool installation, data loading, and environment tuning, see [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html).

2. Running verification (the version with the feature disabled): Take the sysbench 1.0 read-write model as an example. First load the test data, and then execute the test with 256 concurrent threads.

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=256 --report-interval=10 oltp_read_write prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=256 --report-interval=10 oltp_read_write run
    ```

    Expected output: Extract transactions (transactions per second) from the test results as the test metric, and establish the pre-optimization performance baseline. For the read-only model, replace `oltp_read_write` with `oltp_read_only`; for the write-only model, replace it with `oltp_write_only`, and then execute the commands repeatedly.

3. Running verification (the version with the LSE feature enabled): On the BoostDB Percona environment with the LSE optimization feature enabled, repeat step 2 using exactly the same commands and data size, and record the performance data.

#### Result Description

After the MySQL LSE and `rec_get_offsets` optimization features are enabled, the overall MySQL performance (read-only, read-write, and write-only) can be improved by about 5% in sysbench tests with the 8U16G specification and 256 concurrent threads.

> **NOTE:**
>
> * `rec_get_offsets` is a hotspot function in the InnoDB storage engine used to calculate record field offsets. The optimization of this function is usually incorporated into the BoostDB version together with the LSE optimization, jointly contributing to the overall performance improvement.
> * The IP addresses, users, and passwords in this document are for reference only. Modify them based on the actual situation. For detailed sysbench test steps (data loading, test model description, environment tuning, and test result interpretation), see [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html).  
> **Security note**: The reference user password `123456` for running MySQL is a weak password used only for demonstration. In production environments, you must use a strong password and configure it based on the actual customer situation. The `--mysql-password` parameter in the sysbench test command leaves the plaintext password in the shell command history, for example, in files such as `~/.bash_history`. It is recommended that you clear the command history after the test or use a dedicated test account.

### Scenario B: ASLR Security Check and Hardening

#### Scenario Description

The following describes how to enable the OS address space layout randomization (ASLR), helping users complete basic security checks and hardening while enabling the LSE optimization feature. ASLR is a security technology against buffer overflow. It randomizes the layout of linear areas such as heap, stack, and shared library mapping to make it difficult for attackers to predict target addresses and directly locate code, thereby preventing overflow attacks.

#### Procedure

1. Enable ASLR.

    ```bash
    echo 2 > /proc/sys/kernel/randomize_va_space
    ```

2. Verify the configuration.

    ```bash
    cat /proc/sys/kernel/randomize_va_space
    ```

    The expected output is as follows:

    ```output
    2
    ```

#### Result Description

The return value is `2`, indicating that ASLR is fully enabled (the addresses of the heap, stack, shared libraries, and so on are all randomized), and the system has basic protection capabilities against buffer overflow attacks.

## FAQs

The following frequently asked questions (FAQs) are organized based on the operation constraints in the feature guide and the porting guide. For detailed solutions, see the corresponding documents.

Question: What should I do if the dependency check fails when running `rpm -ivh` for installation, but the dependencies are confirmed to be installed?

Answer: Use the `--nodeps` parameter to skip the dependency check, that is, run `rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps`.

Question: What should I do if, during database initialization, the message "--initialize specified but the data directory has files in it." is displayed?

Answer: Initialization fails because files already exist in the data directory. After confirming the contents of the data directory, run `rm -rf /data/mysql/data/*` (or the actually configured `datadir` path) to clear the data directory, and then run the initialization command again.

Question: What should I do if the first startup with `service mysql start` as the `root` user fails and reports that the `mysql.log` file is missing?

Answer: First switch to the `mysql` user (`su - mysql`) to start the database service and generate the `mysql.log` file, then stop the database service, and start it again as the `root` user. For details, see [MySQL Porting Guide (Running MySQL)](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html).

Question: How do I check whether the LSE optimization feature is enabled?

Answer: The BoostDB Percona RPM package used in this document has the LSE optimization built in and enabled by default. It takes effect immediately after installation, and no additional configuration is required. You can verify the performance improvement through the sysbench comparison benchmark in section 3.1.

## More Functions

For more usage of the MySQL LSE optimization feature and the MySQL acceleration kit, see the following documents:

* [MySQL LSE Optimization Feature Guide](https://www.hikunpeng.com/document/detail/en/boostdb/mysql/basic_computation_opt/docs/en/mysql_lse_optimization_feature_guide.md)

* [Percona Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0001.html)

* [MySQL Porting Guide (Running MySQL)](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)

* [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

## Change History

|Version|Date|Description|
|---|---|---|
|01|2026-09-30|This is the first official release.|
