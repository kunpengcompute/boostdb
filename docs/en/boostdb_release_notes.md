# Release Notes

## 2026-06-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 |- Released the MySQL plan cache feature.<br>- Released the MySQL transaction lock optimization feature.<br>- Updated the SIMD-based MySQL character set processing optimization feature.|

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| BoostDB | 26.1.RC1 |

#### Software Version Mapping

|Feature Name|Software Type|Version|
|--|--|--|
|MySQL transaction lock optimization|OS|openEuler 22.03 LTS SP4|
|MySQL transaction lock optimization|Percona|Percona-Server 5.7.44-53|
|MySQL transaction lock optimization|Patch|001-Refactor-InnoDB-lock_sys-sharding-and-latching.patch <br>0002-Optimize-InnoDB-table-lock-queue-checks.patch <br>0003-innodb-enhance-read-view-management-with-version-tracking.patch|
|MySQL plan cache|OS|openEuler 24.03 LTS SP3|
|MySQL plan cache|Percona|Percona-Server 8.0.43-34|
|MySQL plan cache|Boost|boost_1_77_0|
|MySQL plan cache|Patch|0001-add-plan-cache-to-improve-select-performance.patch|
|SIMD-based MySQL character set processing optimization|OS|openEuler 24.03 LTS SP3 or openEuler 22.03 LTS SP4|
|SIMD-based MySQL character set processing optimization|Percona|Percona-Server 8.0.43-34 or Percona-Server 5.7.44-53|
|SIMD-based MySQL character set processing optimization|Boost|boost_1_77_0 or boost_1_59_0|
|SIMD-based MySQL character set processing optimization|Patch|0001-refactor-character-set-handling-for-aarch64-architec.patch|

#### Hardware Version Mapping

| Feature Name         | Item         | Requirement                                     |
| ------------- | ------------- | ----------------------------------------- |
| MySQL transaction lock optimization, MySQL plan cache, and SIMD-based MySQL character set processing optimization    | Processor          | Kunpeng 920 series                  |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### MySQL Transaction Lock Optimization

The MySQL transaction lock optimization feature is added. This feature includes three patches to enhance the InnoDB transaction and lock management capabilities of Percona-Server 5.7.44-53 from three aspects: lock system sharding and latch optimization, table lock queue check optimization, and ReadView version tracking. This reduces global contention, table lock queue scanning overhead, and ReadView management costs in high-concurrency scenarios.

##### MySQL Plan Cache

The MySQL plan cache feature is added. This feature caches the execution plans of `SELECT` statements in prepared statement scenarios. In subsequent execution, the cached execution plans are preferentially reused to reduce repeated optimization overheads. This improves the execution performance of hotspot queries, and enhances system stability in OLTP scenarios. In sysbench read-only scenarios, the performance can be improved by 10%.

##### SIMD-based MySQL Character Set Processing Optimization

The SIMD-based MySQL character set processing optimization feature is updated. In MySQL OLTP scenarios, SIMD instructions are used for vectorization-based acceleration for hotspot functions related to character set processing (such as `my_strnxfrm_unicode`, `my_ismbchar_utf8`, `my_charpos_mb`, `my_hash_sort_utf8`, and `my_lengthsp_8bit`). In sysbench read-only scenarios, the performance can be improved by 10%.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

|Document Name|Description|Delivery Method|
|--|--|--|
|*MySQL Transaction Lock Optimization Feature Guide*|Describes the environment requirements and provides guidance on installing and using the MySQL transaction lock optimization feature.|Open-source repository|
|*MySQL Plan Cache Feature Guide*|Describes the environment requirements and provides guidance on enabling and using the MySQL plan cache feature.|Open-source repository|
|*SIMD-based MySQL Character Set Processing Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling and using the SIMD-based MySQL character set processing optimization feature.|Open-source repository|

### Obtaining Documentation<a name="EN-US_TOPIC_0000002544372643"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/boostdb/tree/master/docs) to view or download required documents.

## 2026-03-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-03-30 |- Released the MySQL LSE optimization feature.<br>- Released the MySQL rec_get_offsets optimization feature.|

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| BoostDB | 26.0.RC1 |

#### Software Version Mapping

|Feature Name|Software Type|Version|
|--|--|--|
|MySQL LSE optimization and MySQL rec_get_offsets optimization|OS|openEuler 22.03 LTS SP4 or openEuler 24.03 LTS SP3|
|MySQL LSE optimization and MySQL rec_get_offsets optimization|Percona|Percona-Server 5.7.44-53 or Percona-Server 8.0.43-34|                                                                             |

#### Hardware Version Mapping

| Feature Name         | Item         | Requirement                                     |
| ------------- | ------------- | ----------------------------------------- |
| MySQL LSE optimization and MySQL rec_get_offsets optimization    | Processor          | - Kunpeng 920 series<br>- Kunpeng 950                   |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### MySQL LSE Optimization

This feature introduces LSE hardware-based acceleration to alleviate performance deterioration under high concurrency and improve the overall performance and system stability of MySQL on Kunpeng servers.

##### MySQL rec_get_offsets Optimization

This feature optimizes the rec_get_offsets call logic to improve the overall performance and system stability of MySQL on Kunpeng servers.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

|Document Name|Description|Delivery Method|
|--|--|--|
|*MySQL LSE Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL LSE optimization feature.|Open-source repository|
|*MySQL rec_get_offsets Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL rec_get_offsets optimization feature.|Open-source repository|

### Obtaining Documentation<a name="EN-US_TOPIC_0000002544372643"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/boostdb/tree/master/docs/en) to view or download required documents.

## 2025-12-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2025-12-30 |- Released the MySQL binlog pre-allocation feature.<br>- Released the MySQL binlog lock splitting feature.<br>- Released the MySQL binlog writeset_history data structure optimization feature.<br>- Released the MySQL record matching optimization feature.<br>- Released the SIMD-based MySQL character set processing optimization feature.<br>- Released the MySQL unaligned memory access optimization feature.|

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| BoostDB | 25.3.0 |

#### Software Version Mapping

|Feature Name|Software Type|Version|
|--|--|--|
|MySQL binlog pre-allocation<br>MySQL binlog lock splitting<br>MySQL binlog writeset_history data structure optimization<br>MySQL record matching optimization<br>SIMD-based MySQL character set processing optimization<br>MySQL unaligned memory access optimization|OS|openEuler 22.03 LTS SP4|
|MySQL binlog pre-allocation<br>MySQL binlog lock splitting<br>MySQL binlog writeset_history data structure optimization<br>MySQL record matching optimization<br>SIMD-based MySQL character set processing optimization<br>MySQL unaligned memory access optimization|Percona|Percona-Server 5.7.44-53<br>Percona-Server 8.0.43-34|

#### Hardware Version Mapping

| Feature Name         | Item         | Requirement                                     |
| ------------- | ------------- | ----------------------------------------- |
| MySQL binlog pre-allocation<br>MySQL binlog lock splitting<br>MySQL binlog writeset_history data structure optimization<br>MySQL record matching optimization<br>SIMD-based MySQL character set processing optimization<br>MySQL unaligned memory access optimization     | Processor          | New Kunpeng 920 processor model or Kunpeng 950 processor                 |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### MySQL Binlog Pre-Allocation

The size of binlog files when being created is pre-allocated as `max_binlog_size`. This prevents metadata operations caused by dynamic file size growth during writes, thereby reducing I/O overhead and improving system performance.

##### MySQL Binlog Lock Splitting

Locks are split so that followers in the FLUSH/SYNC/COMMIT phase wait for different locks, reducing the possibility of false wakeups between different groups and improving system performance.

##### MySQL Binlog writeset_history Data Structure Optimization

The writeset_history data structure is replaced with the hash map data structure to reduce the time complexity of insertion and search to O(1), improving efficiency.

##### MySQL Record Matching Optimization

For records that use the fixed row format, this feature replaces field-by-field traversal matching with whole-record comparison using `memcmp`, improving comparison efficiency.

##### SIMD-based MySQL Character Set Processing Optimization

This feature uses SIMD instructions to vectorize utf8/utf8mb4 character set processing for acceleration, improving the efficiency of related operations.

##### MySQL Unaligned Memory Access Optimization

Kunpeng 920 processors support unaligned memory access. Therefore, the corresponding optimization policies on the x86 architecture can be ported to the Arm architecture to directly convert pointers to the integer type, improving the type conversion efficiency.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

|Document Name|Description|Delivery Method|
|--|--|--|
|*MySQL Binlog Pre-Allocation Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL binlog pre-allocation feature.|Open-source repository|
|*MySQL Binlog Lock Splitting Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL binlog lock splitting feature.|Open-source repository|
|*MySQL Binlog writeset_history Data Structure Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL binlog writeset_history data structure optimization feature.|Open-source repository|
|*MySQL Record Matching Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL record matching optimization feature.|Open-source repository|
|*SIMD-based MySQL Character Set Processing Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the SIMD-based MySQL character set processing optimization feature.|Open-source repository|
|*MySQL Unaligned Memory Access Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL unaligned memory access optimization feature.|Open-source repository|

### Obtaining Documentation<a name="EN-US_TOPIC_0000002544372643"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/boostdb/tree/master/docs) to view or download required documents.
