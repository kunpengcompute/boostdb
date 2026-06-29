# BoostDB-Percona Installation Guide

## Introduction

### BoostDB-Percona-5.7.44-53

`BoostDB-Percona-5.7.44-53.aarch64.rpm` is built on the Percona-Server 5.7.44-53 source code and has been deeply optimized for Kunpeng processors. It integrates the following performance optimization patches that cover hardware instruction acceleration, storage engine optimization, query optimization, and other aspects, significantly enhancing the Percona-5.7 performance on Kunpeng servers. In addition, it uses the Kunpeng GCC Continuous Feature Guided Optimization (CFGO) feature, resulting in a sysbench overall performance improvement of over 25% with the configuration of 8 vCPUs.

- Inline optimization of hot functions
- Cache line size adjustment to 128 bytes
- CRC32 acceleration using hardware instructions
- Character set processing vectorization using the NEON instruction set
- String conversion optimization using the NEON instruction set
- THD data structure optimization
- page_id_t structure optimization with unnecessary member variables and logic removed
- Character set collation search optimization by replacing linear search with hash tables
- Performance optimization for InnoDB tuple comparison
- MVCC ReadView data structure optimization
- Query plan caching (new)
- Binlog space pre-allocation and reuse
- Binlog lock splitting
- Binlog gtid unordered_map replacement

### BoostDB-Percona-8.0.43-34

`BoostDB-Percona-8.0.43-34.aarch64.rpm` is built on the Percona-Server 8.0.43-34 source code and has been deeply optimized for Kunpeng processors. It integrates the following performance optimization patches that cover hardware instruction acceleration, binlog acceleration, and other aspects, significantly enhancing the Percona-8.0.43 performance on Kunpeng servers. In addition, it uses the Kunpeng GCC CFGO feature, resulting in a sysbench overall performance improvement of over 30% with the configuration of 8 vCPUs.

- Character set processing vectorization using the NEON instruction set
- Unaligned memory access optimization
- Performance optimization for InnoDB tuple comparison
- Binlog space pre-allocation
- Binlog lock splitting

## Environment Requirements<a name="EN-US_TOPIC_0000002518697732"></a>

This document provides guidance based on specific environments. Before performing operations, ensure that your hardware and software meet the requirements.

**Table 1** Hardware requirement<a id="hardware-requirement"></a>

|Item|Specifications|
|--|--|
|CPU|New Kunpeng 920 processor model or Kunpeng 950 processor|

**Table 2** OS and software requirements<a id="os-and-software-requirements"></a>

|Item|Version|How to Obtain|
|--|--|--|
|OS|openEuler 22.03 LTS SP4|[Link](https://repo.huaweicloud.com/openeuler/openEuler-22.03-LTS-SP4/ISO/aarch64/openEuler-22.03-LTS-SP4-everything-aarch64-dvd.iso)|
|Percona|Percona-Server 5.7.44-53|[Link](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-5.7.44-53-v3/BoostDB-Percona-5.7.44-53.aarch64.rpm)|
|Percona|Percona-Server 8.0.43-34|[Link](https://gitcode.com/boostkit/boostdb/releases/download/MySQL-Percona-Server-8.0.43-34-v2/BoostDB-Percona-8.0.43-34.aarch64.rpm)|

## BoostDB Feature Installation<a name="EN-US_TOPIC_0000002550177571"></a>

The following steps are applicable to both Percona-Server 5.7.44-53 and Percona-Server 8.0.43-34. Use the corresponding branch commands based on the actual version.

1. Install the dependencies as instructed in [Configuring the Compilation Environment](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0014.html) in the *Percona Porting Guide*.
2. Download the RPM package of the required version as described in [**Table 2**](#os-and-software-requirements) and save the package to the target path, for example, `/home`.
3. Run the following commands to install the RPM package. The default installation directory is `/usr/local/mysql`.

    - Percona-Server 5.7.44-53:

    ```shell
    cd /home
    rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm
    ```

    - Percona-Server 8.0.43-34:

    ```shell
    cd /home
    rpm -ivh BoostDB-Percona-8.0.43-34.aarch64.rpm
    ```

    >![](public_sys-resources/icon_note.gif) **NOTE:**
    >If dependency packages have been installed but the RPM-related check fails, run the following command to skip the dependency check (using `--nodeps`):
>
    >```shell
    >rpm -ivh BoostDB-Percona-5.7.44-53.aarch64.rpm --nodeps
    >rpm -ivh BoostDB-Percona-8.0.43-34.aarch64.rpm --nodeps
    >```

## Troubleshooting<a name="EN-US_TOPIC_0000002550137571"></a>

### "version `GLIBCXX_3.4.29' not found" Is Displayed During MySQL Startup<a name="EN-US_TOPIC_0000002550137567"></a>

**Symptom<a name="section642124153116"></a>**

The error message "/usr/local/mysql/bin/mysqld: /usr/local/mysql/bin/mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by /usr/local/mysql/bin/mysqld)" is displayed during MySQL startup.

**Key Process and Cause Analysis<a name="section145813300553"></a>**

The `libstdc++.so.6` version of the system is too early, and GLIBCXX_3.4.29 is missing.

**Conclusion and Solution<a name="section164566494716"></a>**

1. Download GCC 12.3.1 (GCC for openEuler 3.0.3).

    ```shell
    cd /home
    wget https://mirrors.huaweicloud.com/kunpeng/archive/compiler/kunpeng_gcc/gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

2. Decompress the installation package.

    ```shell
    tar zxvf gcc-12.3.1-2024.12-aarch64-linux.tar.gz
    ```

3. Temporarily specify `libstdc++.so.6` of a later version only in the current terminal session. Do not replace the system library.

    ```shell
    export GCC_HOME=/home/gcc-12.3.1-2024.12-aarch64-linux
    export LD_LIBRARY_PATH=$GCC_HOME/lib64:$LD_LIBRARY_PATH
    ```

4. Check whether the library used by the current session contains GLIBCXX_3.4.29. If any output is displayed, the requirement is met.

    ```shell
    strings $GCC_HOME/lib64/libstdc++.so.6 | grep GLIBCXX_3.4.29
    ```

5. Restart MySQL in the same terminal session (to inherit the preceding environment variables).

    >![](public_sys-resources/icon_note.gif) **NOTE:**
    >The preceding configuration using `export` takes effect only for the current session. After the terminal is closed, the configuration does not take effect and the default `libstdc++.so.6` in the system is not modified.

## Security Check and Hardening<a name="EN-US_TOPIC_0000002518537816"></a>

Address space layout randomization (ASLR) is a security technology against buffer overflow. It randomizes the layout of linear areas such as heap, stack, and shared library mapping to make it difficult for attackers to predict target addresses and directly locate code, thereby preventing overflow attacks.

```shell
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](figures/en-us_image_0000002518697736.png)
