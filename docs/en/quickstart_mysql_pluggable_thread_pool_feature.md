# Quick Start: MySQL Pluggable Thread Pool Feature

<!-- md-trans-meta sourceCommit=0329bc1cbb5916e473f54b25de21737377ed72cb translatedAt=2026-10-20T06:48:10.427Z pushedAt=2026-10-21T02:48:56.706Z nl=2 -->

This document uses MySQL 8.0.25 as an example to describe how to obtain and apply the patch, compile and install MySQL, install the plugin, and verify the usage, helping users quickly get started with the pluggable thread pool feature. According to the test results based on BenchmarkSQL TPC-C (a database transaction processing performance benchmark model) in the feature guide: before the thread pool feature is enabled, the performance of MySQL running 10,000 concurrent tasks is only about 10% of the original performance. After the thread pool feature is enabled, the performance can be maintained at 85% of the original performance. This feature provides the greatest benefits in online transaction processing (OLTP) short-query scenarios with a large number of connections, and also shows significant benefits in read-only short queries with a large number of connections.

## Feature Description

The MySQL pluggable thread pool feature is one of the performance optimization features of the Kunpeng BoostKit MySQL acceleration kit, used to address performance degradation of the default MySQL connector under a large number of connections. The default MySQL connector allocates a thread to each connection. As the number of connections increases, context switching and the contention for hot locks can consume a large amount of CPU resources, deteriorating service performance.

When the thread pool connector module is used, it takes over connection establishment and scheduling: the entire thread pool is divided into several thread groups (the number of thread groups is the number of logical CPU cores on the server by default) and one timer thread. The listener thread in each group uses epoll (an event notification mechanism in the Linux kernel for efficiently listening on readable events of a large number of network connections) to listen for network tasks. Tasks are placed into a high-priority or normal queue, and idle worker threads take them out and process them by priority. The number of worker threads can be dynamically scaled so that the server maintains optimal performance even under a large number of client connections. This feature also supports transaction-priority-based scheduling and thread pool lockup prevention, and adds four status information tables to `information_schema` to monitor the thread pool status in real time.

>![NOTE](public_sys-resources/icon-note.gif) **NOTE:**
>
> * This feature is provided in the form of a patch file. The patch `code-threadpool-for-MySQL-8.0.patch` applies to MySQL 8.0.25, 8.0.30, and 8.0.35.
> * This feature does not conflict with the MySQL NUMA scheduling tuning patch. However, if the thread pool plugin is used, the scheduling tuning for user connection threads in MySQL NUMA scheduling tuning becomes invalid.
> * The thread pool depends on the numactl library. If the numactl dependency is not installed during compilation, the error "undefined symbol: numa_xxxxx" is reported when installing the thread pool plugin.

## Installing MySQL Source Code

### Environment Setup

Before performing operations, ensure that the hardware and software meet the following requirements:

|Environment|Requirement|
|---|---|
|Hardware|Kunpeng server (Kunpeng 920 series processors). For performance testing, the data directory must be placed on a separate drive. For non-performance testing, you can create the data directory directly on the system drive.|
|OS|openEuler 20.03 LTS SP1 or openEuler 22.03 LTS SP1 (for Arm). If a fresh OS installation is required, choose `Minimal Install` and select `Development Tools`.|

Check the environment item by item according to the following table:

|Check Item|Command|Expected Output|Failure Handling|
|---|---|---|---|
|OS version check|cat /etc/*-release|openEuler 20.03 LTS SP1 or 22.03 LTS SP1|Upgrade the OS to a version that meets the requirements.|
|Processor check|lscpu|AArch64 architecture (Kunpeng 920 series)|Replace it with a Kunpeng server.|
|CMake version check|cmake --version|Version higher than 3.7 (3.7.2 provided with openEuler 20.03 and 3.11.4 with 22.03)|Upgrade CMake.|
|GCC version check|gcc -v|7.3.0 for openEuler 20.03 and 10.3.1 for 22.03|Upgrade GCC. See the *MySQL Porting Guide*.|
|numactl dependency check|rpm -qa | grep numactl|numactl and numactl-devel are installed.|Run `yum install -y numactl numactl-devel*` to install them.|
|Git availability check|git --version|Git version number|Configure the Yum repository and then run `yum install git` to install it.|

### Installation Procedure

#### Obtaining the Software Package

|Software Type|Required/Optional|Package Description|Package Name|How to Obtain|
|---|---|---|---|---|
|Main installation package|Required|MySQL source package (includes Boost headers)|mysql-boost-8.0.25.tar.gz|[Official MySQL download page](https://downloads.mysql.com/archives/community/)|
|Feature patch|Required|Thread pool feature patch package (including `code-threadpool-for-MySQL-8.0.patch`, applicable to 8.0.25/8.0.30/8.0.35)|boostdb-patch-release-20260330.zip|[BoostDB patch release page](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip)|

If the server can access the Internet, run the following commands to download the packages directly:

```bash
cd /home
wget https://cdn.mysql.com/archives/mysql-8.0/mysql-boost-8.0.25.tar.gz --no-check-certificate
wget https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip --no-check-certificate
```

#### Applying the Thread Pool Feature Patch

1. Decompress the MySQL source package and enter the root directory of the source code.

   ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    cd mysql-8.0.25
    ```

2. Decompress the patch package and place `code-threadpool-for-MySQL-8.0.patch` in the root directory of the MySQL source code.

    ```bash
    cd /home
    unzip boostdb-patch-release-20260330.zip
    cp /home/boostdb-patch-release-20260330/code-threadpool-for-MySQL-8.0.patch /home/mysql-8.0.25/
    cd /home/mysql-8.0.25
    ```

3. In the root directory of the source code, run the `git init` command to create Git management information.

    ```bash
    git init
    git add -A
    git commit -m "Initial commit"
    ```

    >![NOTE](public_sys-resources/icon-note.gif) **NOTE:**
    >
    > If the Git commit user information is not configured, configure it before running `git commit`.
    >
    > ```bash
    > git config user.email "123@example.com"
    > git config user.name "123"
    > ```

4. Apply the thread pool feature patch.

    ```bash
    git apply --check --whitespace=nowarn code-threadpool-for-MySQL-8.0.patch
    git apply --whitespace=nowarn code-threadpool-for-MySQL-8.0.patch
    ```

5. Confirm that the patch is applied successfully and check the newly added `thread_pool` directory.

    ```bash
    ll ./plugin/thread_pool/
    ```

    Expected output: You can see the newly added source files in the `thread_pool` directory.

#### Compiling and Installing MySQL

Compile and install MySQL by following the standard MySQL source code compilation and installation steps. For details, see the "Compiling and Installing MySQL" section in the *MySQL Porting Guide*. The following uses 96 cores as an example, and the core commands are as follows:

```bash
cd /home/mysql-8.0.25
mkdir build
cd build
cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0
make -j 96
make -j 96 install
```

> ![NOTE icon](public_sys-resources/icon-note.gif) **NOTE:**
>
> The `-j 96` parameter fully utilizes multiple CPU cores to speed up compilation. The number following `-j` indicates the number of CPU cores, which can be checked using `cat /proc/cpuinfo | grep processor | wc -l`. This value should be less than or equal to the total number of CPU cores.

After compilation and installation are complete, initialize and start the database. The core steps are summarized as follows. For detailed operations, see the "Running MySQL" section in the *MySQL Porting Guide*.

1. Modify the configuration file `my.cnf` (configure parameters such as `basedir`, `datadir`, `socket`, `port`, and `user`; modify the paths as required).

2. Configure the environment variable: add the MySQL binary file path (`/usr/local/mysql/bin`) to `PATH`, and make it effective.

3. Initialize the database (a temporary initial password is generated following `root@localhost:` in the command output. Save it for later use).

   ```bash
   mysqld --defaults-file=/etc/my.cnf --initialize
   ```

4. Start the database service.

   ```bash
    /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf &
    ```

    Check the database process and listening port to confirm that the service has started properly.

    ```bash
    ps -ef | grep mysql
    netstat -anpt | grep 3306
    ```

#### Verifying the Compilation Output

Confirm that the thread pool plugin dynamic library has been generated in the compilation output.

```bash
ls $CMAKE_INSTALL_PREFIX/lib/plugin/thread_pool.so
```

Expected output: The `thread_pool.so` file exists, indicating that the thread pool plugin has been generated along with the compilation.

## Using MySQL Pluggable Thread Pool

### Installing and Enabling the Thread Pool Plugin

#### Scenario Description

MySQL uses one thread per connection by default. This section uses the example of installing the thread pool plugin online through SQL commands and verifying that it takes effect, to help users quickly enable the thread pool connector. After successful installation, the thread pool plugin connector is used to handle query requests of new connections, while the original default connector is retained and continues to handle connections established before the installation.

#### Procedure

1. Log in to the database and change the initial password.

   ```bash
   mysql -uroot -p -S /data/mysql/run/mysql.sock
   ```

   When prompted for a password, enter the temporary initial password generated in step 3. After a successful login, change the `root` user password and create a `root` user that can connect from other servers.

   ```sql
   alter user 'root'@'localhost' identified by '123456';
   create user 'root'@'%' identified by '123456';
   grant all privileges on *.* to 'root'@'%';
   flush privileges;
   ```

   The expected output is as follows:

   ```output
   Query OK, 0 rows affected (0.01 sec)
   ```

   >![NOTE icon](public_sys-resources/icon-note.gif) **NOTE:**
   > The users and passwords in this document are for reference only. Configure them according to the actual customer environment.  
   > **Security note**: `123456` is a weak password used only for demonstration. In production environments, you must use a strong password (12 or more characters recommended, including uppercase and lowercase letters, digits, and special characters). In addition, avoid passing plaintext passwords directly in command-line parameters (such as `mysql -uroot -p123456`) to prevent passwords from being leaked in shell command history.

2. Log in again with the new password for verification.

   Run `exit` to exit the database, and then log in again.

   ```bash
   mysql -uroot -p -S /data/mysql/run/mysql.sock
   ```

   The expected output (partial) is as follows:

   ```output
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 8
   Server version: 8.0.25
   ```

3. Install the thread pool plugin.

   `thread_pool.so` contains five plugins: `thread_pool` is the thread pool connector plugin, and `THREAD_POOL_GROUPS`, `THREAD_POOL_QUEUES`, `THREAD_POOL_STATS`, and `THREAD_POOL_WAITS` are thread pool status monitoring table plugins. Run the following SQL statements to install them. The installation takes effect immediately.

   ```sql
   INSTALL PLUGIN thread_pool SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_GROUPS SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_QUEUES SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_STATS SONAME "thread_pool.so";
   INSTALL PLUGIN THREAD_POOL_WAITS SONAME "thread_pool.so";
   ```

   The expected output is as follows:

   ```output
   Query OK, 0 rows affected (0.01 sec)
   ```

4. Verify the plugin installation result.

   ```sql
   show plugins;
   ```

   Expected output: In the command output, the status of the `thread_pool` plugin is `ACTIVE`, indicating that the plugin has been installed successfully.

5. View the thread pool running status.

   Monitor the thread pool status in real time through the status tables newly added in `information_schema`. For example, view the thread group information.

   ```sql
   select * from information_schema.THREAD_POOL_GROUPS;
   ```

   Expected output: The status information of each thread group, such as the number of connections, number of threads, and queue depth, is returned.

#### Result Description

The thread pool plugin status is `ACTIVE` and the status tables can be queried normally, indicating that the thread pool connector has taken effect. New connections will be scheduled by the thread pool.

> ![NOTE icon](public_sys-resources/icon-note.gif) **NOTE:**
>
> * You can also add `plugin-load-add=thread_pool.so` to the MySQL configuration file `my.cnf` to install the plugin. This method requires restarting the database to take effect.
> * If you copy `thread_pool.so` to another MySQL service for use after compilation, place the file in the directory specified by the `plugin_dir` variable of the target service, and then run the preceding `INSTALL PLUGIN` statements.
> * To uninstall the thread pool plugin, see the "Uninstalling the Thread Pool Plugin" section in the feature guide.

### Adjusting `thread_pool_size` to Optimize Peak Performance

#### Scenario Description

This section describes how to adjust the number of thread groups in OLTP scenarios to optimize peak performance, helping users experience the tuning of core thread pool parameters. For OLTP scenarios, it is recommended that the default thread pool parameter configuration be used. You can optimize peak performance by adjusting the value of `thread_pool_size` (number of thread groups).

#### Procedure

1. Check the current number of thread groups.

   ```sql
   show variables like 'thread_pool_size';
   ```

   The expected output is as follows:

   ```output
   +------------------+-------+
   | Variable_name    | Value |
   +------------------+-------+
   | thread_pool_size | 96    |
   +------------------+-------+
   ```

   The default value indicates that the number of thread groups is the same as the number of CPU cores.

2. Dynamically adjust the number of thread groups.

   For scenarios where the number of connections exceeds the number of logical CPU cores, the performance bottleneck is not lock contention, and the CPU is not fully utilized, you can set the number of thread groups to 1 to 3 times the number of CPU cores or the optimal concurrency to achieve better performance.

   ```sql
   set global thread_pool_size=128;
   ```

   The expected output is as follows:

   ```output
   Query OK, 0 rows affected (0.00 sec)
   ```

#### Result Description

`thread_pool_size` can be set via the command line or a configuration file, and can also be modified dynamically (global scope; valid values: 1–1024). After the change, the thread pool schedules connections according to the new number of thread groups. Under high-concurrency TPC-C workloads, enabling the thread pool maintains performance at approximately 85% of the original level; without it, performance drops to only about 10%. For detailed test steps of performance verification, see [BenchmarkSQL Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengbenchmarksql_06_0001.html).

> ![NOTE icon](public_sys-resources/icon-note.gif) **NOTE:**
>
> MySQL configuration parameters (system variables) can be used to adjust the functions and performance of the database service. For details, see the official MySQL document *Server System Variables*.

## FAQs

The following frequently asked questions (FAQs) are organized based on the operation constraints in the feature guide. For detailed solutions, see the corresponding documents.

**Question**: What should I do if the error "undefined symbol: numa_xxxxx" is reported when installing the thread pool plugin?

**Answer**: `undefined symbol` is a typical **runtime dynamic linking error**. The cause is that `mysqld` cannot find the `libnuma.so` library file at runtime. It is possible that the numactl dependency was not installed when compiling MySQL, so the compiled MySQL program did not load the numa library. Run `yum install -y numactl numactl-devel*` to install the dependency first, and then recompile and install MySQL.

**Question**: What should I do if an error is reported when running `git apply --check` to check the patch and the patch cannot be applied?

**Answer**: `code-threadpool-for-MySQL-8.0.patch` applies to MySQL 8.0.25, 8.0.30, and 8.0.35. Confirm that the source code version you are using is within the supported range and that the source code has not been modified after decompression (you are advised to run `git init` first to establish a baseline before applying the patch). If the source code has been modified, decompress a clean source package and try again. If it still fails, add the `--ignore-whitespace` parameter after the `git apply` command to ignore whitespace differences and try again.

**Question**: After the execution of `INSTALL PLUGIN`, what should I do if the plugin status shown by `show plugins` is not `ACTIVE`?

**Answer**: Confirm that the `thread_pool.so` file is located in the directory specified by the `plugin_dir` variable of the target MySQL service (you can query it by running `show variables like 'plugin_dir';`). If the file is missing, copy `thread_pool.so` from `plugin_output_directory/` under the compilation path and reinstall it. In addition, confirm that the `thread_pool.so` file has read permission (`chmod +rx`) for the MySQL user (usually `mysql`). You can also check the MySQL error log (`error.log`), which usually contains more detailed information about the loading failure.

**Question**: What should I do if the warning "plugin is busy" is reported when uninstalling the thread pool plugin?

**Answer**: This indicates that there are still user connections on the thread pool connector, and the plugin status changes from `ACTIVE` to `DELETE` (the uninstallation process status). First, disconnect the service connections, or run ` show processlist ` to confirm that no connections depend on the thread pool, and then uninstall the thread pool connector plugin (`UNINSTALL PLUGIN thread_pool`). At this point, because there are no connections, the uninstallation completes immediately. Next, uninstall the four status table plugins.

## More Functions

For more usage of the pluggable thread pool feature (full description of configuration parameters, priority session handling, NUMA affinity, connection count balancing, plugin uninstallation, and so on), see the following documents:

- [MySQL 8.0.25, 8.0.30, and 8.0.35 Pluggable Thread Pool Feature Guide](https://www.hikunpeng.com/document/detail/en/boostdb/mysql/pluggablethreadpool/docs/en/mysql_8_0_25_8_0_30_and_8_0_35_pluggable_thread_pool_feature_guide.md)

- [MySQL 8.0.20 Thread Pool Feature Guide](https://www.hikunpeng.com/document/detail/en/boostdb/mysql/pluggablethreadpool/docs/en/mysql_8_0_20_thread_pool_feature_guide.md)

- [MySQL 5.7.27 Thread Pool Feature Guide](https://www.hikunpeng.com/document/detail/en/boostdb/mysql/pluggablethreadpool/docs/en/mysql_5_7_27_thread_pool_feature_guide.md)

- [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

- [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

- [BenchmarkSQL Test Guide (TPC-C)](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengbenchmarksql_06_0001.html)

## Change History

|Version|Date|Description|
|---|---|---|
|01|2026-09-30|This is the first official release.|
