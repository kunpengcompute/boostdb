# SIMD-based MySQL Character Set Processing Optimization Feature Guide

## Feature Description<a name="EN-US_TOPIC_0000002518697734"></a>

### Overview<a name="EN-US_TOPIC_0000002518537818"></a>

This document describes how to install and enable the Single Instruction Multiple Data (SIMD)-based character set processing optimization feature on a Kunpeng server.

In MySQL online transaction processing (OLTP) scenarios, when the sysbench tool is used to perform read-only tests, performance monitoring reveals that the main hotspot functions are character set processing-related functions (such as `my_strnxfrm_unicode`, `my_ismbchar_utf8`, `my_charpos_mb`, `my_hash_sort_utf8`, and `my_lengthsp_8bit`). For these hotspot functions, Kunpeng BoostKit uses SIMD instructions for vectorization-based acceleration, significantly improving the execution efficiency.

With the combined use of record matching optimization, SIMD-based character set processing optimization, and unaligned memory access optimization, the performance of Percona-Server 5.7.44-53 with a container configuration of 8 vCPU and 16 GB memory is improved by about 10% in sysbench read-only test scenarios.

### Principles<a name="EN-US_TOPIC_0000002550177569"></a>

**SIMD-based Character Set Processing Optimization<a name="section695124719482"></a>**

This optimization uses SIMD instructions to vectorize utf8/utf8mb4 character set processing for acceleration, improving the efficiency of related operations.

As utf8/utf8mb4 is a variable-length encoding, the length of a character needs to be calculated or the character needs to be converted to a fixed-length Unicode format (2 bytes) before subsequent processing. ASCII characters (ranging from 0 to 127) are single-byte fixed-length, and their Unicode representation remains the same as the original value (with higher bytes padded with zeros). Therefore, SIMD-based parallel processing can be implemented for these characters to improve the processing throughput.

In terms of code implementation, the SIMD-based optimization scope varies depending on the Percona version.

- Percona-Server 5.7.44-53:
  - In the code path for `utf8_general_ci`/`utf8mb4_general_ci` rule processing, `my_strnxfrm_unicode`, `my_hash_sort_utf8`, and `my_hash_sort_utf8mb4` are optimized. In the code path for character set processing, `my_numchars_mb` and `my_charpos_mb` are optimized.
  - In the code path for Unicode Collation Algorithm (UCA)-based rule (`utf8_unicode_ci` and `utf8mb4_unicode_ci`) processing, SVE optimization is also enabled, covering the UCA-based comparison, hashing, and weight conversion processes (`my_uca_simd_support_check`, `my_hash_sort_uca`, and `my_strnxfrm_uca`).

- Percona-Server 8.0.43-34:
  - In the code path for `utf8_general_ci`/`utf8mb4_general_ci` rule processing, SVE optimization is injected through `ctype-opt.cc`, covering `my_strnxfrm_unicode`, `my_hash_sort_utf8mb3`, `my_hash_sort_utf8mb4`, `my_numchars_mb`, and `my_charpos_mb3` (utf8mb3) /`my_charpos_mb4` (utf8mb4).
  - In the code path for UCA-based rule (`utf8mb3_unicode_ci`, `utf8mb4_unicode_ci`, and `utf8mb4_0900_ai_ci`) processing, SVE optimization is also enabled, covering the UCA-based comparison, hashing, and weight conversion processes (such as `my_simd_support_check_uca_any`, `my_simd_support_check_uca_900_primary`, `my_hash_sort_uca`, and `my_strnxfrm_uca`/`my_strnxfrm_uca_900_tmpl`).

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
|Percona|Percona-Server 5.7.44-53|For details, see [BoostDB-Percona Installation Guide](./boostdb-percona-install.md).|
|Percona|Percona-Server 8.0.43-34|For details, see [BoostDB-Percona Installation Guide](./boostdb-percona-install.md).|

## Feature Installation and Enablement<a name="EN-US_TOPIC_0000002550177571"></a>

The following uses Percona-Server 5.7.44-53 as an example to describe how to install and enable the SIMD-based character set processing optimization feature. The procedure is as follows:

1. Install the optimized BoostDB-Percona version as instructed in [BoostDB-Percona Installation Guide](./boostdb-percona-install.md).
2. Add the collation configuration to the MySQL configuration file `/etc/my.cnf`.
    1. Open the `/etc/my.cnf` file.

        ```shell
        vi /etc/my.cnf
        ```

    2. Press `i` to enter the insert mode. If the character set is `utf8`, add the character set and comparison rule configuration to the `[mysqld]` section. Select one of the following configurations based on your application scenario:

        ```txt
        character_set_server = utf8
        collation_server = utf8_general_ci
        ```

        ```txt
        character_set_server = utf8
        collation_server = utf8_unicode_ci
        ```

        >![](public_sys-resources/icon_note.gif) **NOTE:**
        >For Percona-Server 8.0.43-34, perform the following configurations:
        >
        > ```txt
        > character_set_server = utf8mb3
        > collation_server = utf8mb3_general_ci
        >    ```
        >
        > ```txt
        > character_set_server = utf8mb3
        > collation_server = utf8mb3_unicode_ci
        >    ```

    3. If the character set is `utf8mb4`, add the character set and comparison rule configuration to the `[mysqld]` section. Select one of the following configurations based on your application scenario:

        ```txt
        character_set_server = utf8mb4
        collation_server = utf8mb4_general_ci
        ```

        ```txt
        character_set_server = utf8mb4
        collation_server = utf8mb4_unicode_ci
        ```
 
        >![](public_sys-resources/icon_note.gif) **NOTE:**
        >In addition to the preceding configurations, the following configuration is supported for Percona-Server 8.0.43-34:
        >
        > ```txt
        > character_set_server = utf8mb4
        > collation_server = utf8mb4_0900_ai_ci
        > ```

    4. Press `Esc` to exit the insert mode. Type `:wq!` and press `Enter` to save the file and exit.

3. Start the database. For details, see [Running MySQL](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html) in the *MySQL Porting Guide*.

4. Query the character set and collation configuration of a database and table. (The following uses the `sbtest` database and the `sbtest1` table in the database as an example.)

    ```sql
    show create database sbtest;
    SHOW VARIABLES LIKE 'collation_database';
    show create table sbtest1;
    show full columns from sbtest1;
    show table status from sbtest like 'sbtest%';
    ```

    >![](public_sys-resources/icon_note.gif) **NOTE:**
    >For details about how to access the client, see [Running MySQL](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html) in the *MySQL Porting Guide*.

    **Figure 1** Querying the character set and collation of the database<a name="fig1447410214484"></a><a id="querying-the-character-set-and-collation-of-the-database"></a><br>
    ![](figures/querying_the_character_set_and_collation_of_the_database.png "Querying the character set and collation of the database")

    **Figure 2** Querying the character set and collation of the table<a name="fig13338178174811"></a><a id="querying-the-character-set-and-collation-of-the-table"></a><br>
    ![](figures/querying_the_character_set_and_collation_of_the_table.png "Querying the character set and collation of the table")

    **Figure 3** Obtaining the table information of the database (The `collation` column indicates the collation.)<a name="fig762681216487"></a><a id="obtaining-the-table-information-of-the-database"></a><br>
    ![](figures/obtaining_the_table_information_of_the_database.png "Obtaining the table information of the database")

5. (Optional) Perform the sysbench test to compare the performance before and after the SIMD-based character set processing optimization feature is enabled. For details about the test procedure, see [Sysbench 0.5 & 1.0 Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengsysbench_02_0001.html).<br>With the combined use of record matching optimization, SIMD-based character set processing optimization, and unaligned memory access optimization, the performance can be improved by about 10% in sysbench read-only test scenarios. [**Figure 4**](#performance-comparison-before-and-after-optimization-with-the-three-features) shows the performance comparison before and after the optimization.

    **Figure 4** Performance comparison before and after optimization with the three features<a name="fig937192253919"></a><a id="performance-comparison-before-and-after-optimization-with-the-three-features"></a><br>
    ![](figures/performance_comparison_character_set.png "Performance comparison before and after optimization with the three features")

## Security Check and Hardening<a name="EN-US_TOPIC_0000002518537816"></a>

Address space layout randomization (ASLR) is a security technology against buffer overflow. It randomizes the layout of linear areas such as heap, stack, and shared library mapping to make it difficult for attackers to predict target addresses and directly locate code, thereby preventing overflow attacks.

```shell
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](figures/en-us_image_0000002518697736.png)
