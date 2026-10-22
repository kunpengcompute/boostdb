# Quick Start: MySQL CRC32 Instruction Optimization Feature

<!-- md-trans-meta sourceCommit=0a3a7bcc08936318bbf4eef365fc44d1e73ebd9f translatedAt=2026-10-16T05:58:01.711Z pushedAt=2026-10-20T06:44:26.629Z nl=2 -->

This document describes the CRC32 instruction optimization feature in the Kunpeng BoostKit MySQL acceleration kit. This feature aims to replace the CRC32 software implementation with CRC32 hardware instructions of Kunpeng processors (such as Kunpeng 920 processors) to reduce calculation overhead, thereby improving MySQL performance on openEuler (by about 5% in sysbench write scenarios).

## Component Introduction

The CRC32 instruction optimization feature is one of the basic computation optimization features of the Kunpeng BoostKit MySQL acceleration kit. It is used to improve the cyclic redundancy check (CRC) computation performance of MySQL on Kunpeng servers running openEuler. This feature replaces the software implementation of the CRC32 algorithm with Kunpeng CRC32 hardware instructions, reducing the CRC32 calculation overhead.

The Linux kernel contains the C language-based implementation of the CRC32 algorithm, but its performance is low and may become a system performance bottleneck when the proportion of CRC32 function calls in kernel space is high. After replacing the software implementation with Kunpeng CRC32 hardware instructions, MySQL performance can be improved by about 5% in sysbench write scenarios.

This document uses MySQL 8.0.25 as an example to describe how to obtain and apply the patch, and perform compilation, installation, testing, verification, and performance comparison, to help users quickly get started with the CRC32 instruction optimization feature. The methods in this document can also be used as a reference for adaptation and optimization in other scenarios.

>![NOTE](public_sys-resources/icon-note.gif)  **NOTE:**
>
> * This feature is compatible with other MySQL acceleration features. For compatibility information between features, see [Compatibility Between Features](https://www.hikunpeng.com/document/detail/en/boostdb/compbf/kunpengdbsmysqlfeaturecompatibility_20_0001.html).
> * During GCC compilation, specify the Arm architecture version and instruction set extensions through the `-march=armv8-a+crc` compilation option to enable the CRC32 hardware instructions.

## Quick Installation

### Environment Setup

Before performing operations, ensure that the hardware and software meet the following requirements:

* Hardware: Kunpeng server (Kunpeng 920 processor).

* OS: openEuler 20.03 LTS SP1 or openEuler 22.03 LTS SP1. The OS image can be obtained from the [openEuler official download page](https://www.openeuler.org/en/download/). (Download address for 20.03: [openEuler 20.03 LTS SP1](https://www.openeuler.openatom.cn/en/download/archive/detail/?version=openEuler%2020.03%20LTS%20SP1); download address for version 22.03: [openEuler 22.03 LTS SP1](https://www.openeuler.openatom.cn/en/download/archive/detail/?version=openEuler%2022.03%20LTS%20SP1))

Check the environment item by item according to the following table:

|Check Item|Command|Expected Output|Failure Handling|
|---|---|---|---|
|CPU CRC32 instruction support check|cat /proc/cpuinfo|The `Features` line in the output contains `crc32`, which indicates that the CPU supports CRC32 hardware instruction optimization|Replace it with a server equipped with a Kunpeng 920 processor.|
|OS version check|cat /etc/*-release|openEuler 20.03 LTS SP1 or 22.03 LTS SP1|Upgrade the OS to a version that meets the requirements.|
|Git availability check|git --version|Git version number |Run `yum install git` to install it.|
|MySQL compilation environment check|gcc -v && /usr/local/bin/cmake --version|GCC ≥ 5.3.0; CMake ≥ 3.4.3|See [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html) to configure the compilation environment.|

### Installation Procedure

#### Obtaining the Software Package

|Software Type|Required/Optional|Package Description|Package Name|How to Obtain|
|---|---|---|---|---|
|Main installation package|Required|MySQL source package (includes Boost headers)|mysql-boost-8.0.25.tar.gz|[Official MySQL download page](https://downloads.mysql.com/archives/community/)|
|Feature patch|Required|CRC32 instruction optimization feature patch, developed for MySQL 8.0.25|0001-CRC32-AARCH64.patch|[CRC32 instruction optimization feature patch](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-patch-release/boostdb-patch-release-20260330.zip)|

#### Applying the Patch

The CRC32 instruction optimization feature for MySQL is provided as a patch file. You need to apply the patch file to the MySQL source code before compiling and installing MySQL.

1. Decompress the downloaded MySQL source package.

    ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    ```

2. Upload the patch file `0001-CRC32-AARCH64.patch` to the decompressed MySQL source code directory (`/home/mysql-8.0.25`).

3. In the root directory of the source code, run the `git init` command to create Git management information.

    ```bash
    cd /home/mysql-8.0.25
    git init
    git add -A
    git commit -m "Initial commit"
    ```

    > ![NOTE](public_sys-resources/icon-note.gif) **NOTE:**
    >
    > If the Git commit user information is not configured, configure the user email and user name before running `git commit`:
    >
    > ```bash
    > git config user.email "123@example.com"
    > git config user.name "123"
    > ```

4. Run the following commands in the MySQL source code directory to apply the CRC32 instruction optimization feature patch.

    ```bash
    # View the statistics of the patch file.
    git apply --ignore-whitespace --stat 0001-CRC32-AARCH64.patch
    # Check whether the patch file can be successfully applied to the current code repository.
    git apply --ignore-whitespace --check 0001-CRC32-AARCH64.patch
    # Apply the patch file to the current code repository.
    git apply --ignore-whitespace 0001-CRC32-AARCH64.patch
    ```

    The expected output is as follows:

    ```output
    storage/innobase/ut/crc32.cc |   38 ++++++++++++++++++++++++++++++-----
    1 file changed, 35 insertions(+), 3 deletions(-)
    ```

#### Compiling and Installing MySQL

After the patch is applied, compile and install MySQL. For the detailed procedure, see the "Compiling and Installing MySQL" section in [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html). The core commands are as follows:

```bash
cd /home/mysql-8.0.25
mkdir build
cd build
cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0 -DCMAKE_C_FLAGS="-march=armv8-a+crc" -DCMAKE_CXX_FLAGS="-march=armv8-a+crc"
make -j 96
make -j 96 install
```

>![NOTE](public_sys-resources/icon-note.gif) **NOTE:**
>
> The `-j 96` parameter fully utilizes multiple CPU cores to speed up compilation. The number following `-j` indicates the number of CPU cores, which can be checked using `cat /proc/cpuinfo | grep processor | wc -l`. This value should be less than or equal to the total number of CPU cores.

### Installation Verification

Run the following command to use `objdump` (an object file disassembly tool) to check whether the CRC32 hardware instruction is enabled in the `mysqld` binary (`crc32cb` is the assembly mnemonic of the Arm CRC32 checksum hardware instruction):

```bash
objdump -d /usr/local/mysql/bin/mysqld | grep crc32cb
```

Expected output: The following `crc32cb` disassembly output is displayed, which indicates that the CRC32 instruction optimization feature is successfully enabled.

```output
  1df3e44:       1ac05060        crc32cb w0, w3, w0
  1df3e94:       1ac45044        crc32cb w4, w2, w4
  1df3f94:       1ac05040        crc32cb w0, w2, w0
  1df3fe4:       1ac35043        crc32cb w3, w2, w3
  1df4134:       1ac05040        crc32cb w0, w2, w0
```

## Quick Start

### Scenario A: Performance Comparison and Verification in sysbench Write Scenarios

#### Scenario Description

Take sysbench write stress testing as an example (sysbench is a widely used open-source, cross-platform, multi-threaded performance testing tool in the industry, and can be used for MySQL database performance testing). The performance test is executed separately on MySQL without the CRC32 patch and MySQL with the CRC32 patch using the same test model to demonstrate the performance improvement before and after the CRC32 instruction optimization feature is used.

#### Procedure

1. Basic configuration.

   Prepare two MySQL environments separately (the pre-optimization version and the post-optimization version). For installing and running MySQL, see [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html). For installing the sysbench tool and tuning the environment, see [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html).

2. Running verification (pre-optimization, baseline version).

   Take sysbench 1.0 as an example. Load the test data, and then run the write-workload test.

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only run
    ```

    Expected output: Extract transactions (transactions per second) and read/write requests (read/write requests per second) from the test results as the test metrics, and establish the pre-optimization write-workload performance baseline.

3. Running verification (post-optimization, version with the CRC32 patch).

   On the MySQL environment with the CRC32 instruction optimization feature patch applied, repeat step 2 using exactly the same commands and data size.

    ```bash
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only prepare
    /home/sysbench-1.0/src/sysbench --db-driver=mysql --mysql-host=192.168.222.120 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --mysql-db=sysbench --table_size=1000000 --tables=100 --time=180 --threads=96 --report-interval=10 oltp_write_only run
    ```

    Expected output: Record the write-workload performance data after optimization.

#### Result Description

With the CRC32 instruction optimization enabled, sysbench write-workload performance increased from a pre-optimization normalized value of 1.0 to 1.05, representing a performance improvement of approximately 5%.

The IP addresses, users, and passwords in this document are for reference only. Modify them based on the actual situation. For detailed sysbench test steps (data loading, test model description, environment tuning, and test result interpretation), see [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html).

**Security note**: The reference user password `123456` for running MySQL is a weak password used only for demonstration. In production environments, you must use a strong password and configure it based on the actual customer situation. The `--mysql-password` parameter in the sysbench test command leaves the plaintext password in the shell command history, for example, in files such as `~/.bash_history`. It is recommended that you clear the command history after the test or use a dedicated test account.

### Scenario B: Adding LSE Compilation Options to Optimize Multi-Core Lock Contention

#### Scenario Description

To improve MySQL performance in multi-core scenarios with severe atomic lock contention, the Large System Extensions (LSE) options are added to the GCC compilation options to help users experience the lock performance optimization brought by the Armv8.1 atomic operation instruction extension.

Load-link/Store-conditional (LL/SC) atomic instructions need to load shared variables into the L1 cache where the current core is located for modification. Under intense lock contention, performance deteriorates severely. The Armv8.1 specification introduces LSE, which puts computation operations into the L3 cache for execution to increase the scope of data sharing, reduce cache-coherence overhead, and improve lock performance when lock contention is intense.

#### Procedure

1. Modify the compilation options.

    Add the `lse` option to the MySQL compilation command.

    ```bash
    cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0 -DCMAKE_C_FLAGS="-march=armv8-a+crc+lse" -DCMAKE_CXX_FLAGS="-march=armv8-a+crc+lse"
    ```

2. Perform the installation.

    Recompile and install MySQL by following the required steps, start the database, and then perform the sysbench stress testing. 

#### Result Description

In multi-core scenarios with intense lock contention, after using the LSE compilation options, atomic instructions switch from LL/SC (`ldaxr`/`stlxr`) to LSE (`ldaddal`), which reduces lock contention overhead and improves system performance.

## FAQs

The following frequently asked questions (FAQs) are organized based on the operation constraints in the feature guide. For detailed solutions, see the corresponding documents.

**Question: What should I do if running `git apply --ignore-whitespace --check 0001-CRC32-AARCH64.patch` reports an error, and the patch cannot be applied?**

Answer: This patch is developed for MySQL 8.0.25. Confirm the following two points: first, the source package in use is MySQL 8.0.25 and has not been modified after decompression (you are advised to run `git init` to establish a baseline before applying the patch); second, the patch application command must be executed in the root directory of the source code (`/home/mysql-8.0.25`). If the source code has been modified, decompress a clean source package and try again.

**Question: What should I do if the `Features` line of the `cat /proc/cpuinfo` output does not contain `crc32`?** 

Answer: This indicates that the current CPU does not support CRC32 hardware instruction optimization, and this feature cannot be enabled. Use a server equipped with a Kunpeng 920 processor instead.

**Question: What should I do if executing `objdump -d /usr/local/mysql/bin/mysqld | grep crc32cb` produces no output after compilation and installation?**

Answer: This indicates that the CRC32 instruction optimization feature has not been enabled successfully. Check the following: whether the patch has been applied successfully (check whether `storage/innobase/ut/crc32.cc` has been modified); whether the compilation was executed on the source code after the patch was applied. After confirmation, recompile and install MySQL.

**Question: What should I do if running a Git command returns an error indicating that the command does not exist?**

Answer: The Git tool is missing. See the Yum repository configuration content in [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html), and then run `yum install git` to install it.

## More Functions

For more usage of the CRC32 instruction optimization feature and the MySQL acceleration kit, see the following documents:

* [CRC32 Instruction Optimization Feature Guide](https://www.hikunpeng.com/document/detail/en/boostdb/mysql/basic_computation_opt/docs/en/crc32_instruction_optimization_feature_guide.md)

* [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

* [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html)

* [MySQL 8.0.x Tuning Guide (Modifying Database Parameters and OS Tuning)](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0013.html)

## Change History

|Version | Date | Description |
| --------| -------- | -------- |
|01| 2026-09-30 | This is the first official release. |
