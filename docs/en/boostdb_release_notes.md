# Release Notes

<!-- md-trans-meta sourceCommit=0452e01f56edb20545774f216f0837dca944de8b translatedAt=2026-10-14T01:16:52.076Z pushedAt=2026-10-14T09:28:05.010Z -->

## 2026-09-30

### Change History

| Version | Date | Description |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-09-30 |- Released the Percona thread pool dynamic concurrency scheduling optimization feature.<br>- Released the MySQL redo file reuse feature.<br>- Released the MySQL container-aware optimization feature. |

### Version Mapping

#### Product Version Information

| Product Name      | Version     |
| --------- | -------- |
| Kunpeng BoostKit | 26.2.RC1 |

#### Software Version Mapping

| Feature | Software Type | Version |
|--|--|--|
| Percona thread pool dynamic concurrency scheduling optimization | OS | openEuler 22.03 LTS SP4 |
| Percona thread pool dynamic concurrency scheduling optimization | Percona | Percona-Server 8.0.43-34 |
| Percona thread pool dynamic concurrency scheduling optimization | Patch | 0001-WL-16484-InnoDB-container-aware-cgroup-memory-limit.patch<br>0002-adjust-threadpool-concurrency-using-scheduler-statis.patch |
| MySQL redo file reuse | OS | openEuler 22.03 LTS SP4 or openEuler 24.03 LTS SP3 |
| MySQL redo file reuse | Percona | Percona-Server 8.0.43-34 |
| MySQL container-aware optimization | OS | openEuler 22.03 LTS SP4 or openEuler 24.03 LTS SP3 |
| MySQL container-aware optimization | Percona | Percona-Server 8.0.43-34 |

#### Hardware Version Mapping

| Feature | Item | Requirement |
| ------------- | ------------- | ---------- |
| Percona thread pool dynamic concurrency scheduling optimization, MySQL redo file reuse, and MySQL container-aware optimization | Processor | Kunpeng 920 processor, new Kunpeng 920 processor model, or Kunpeng 950 processor |

#### Virus Scan Results

Virus scanning is not involved because no software package is released.

### Important Notes

None

### Release Notes

#### Change Description

##### Percona Thread Pool Dynamic Concurrency Scheduling Optimization

The original thread pool of Percona Server does not adjust the number of worker threads based on the process CPU usage and context switching frequency. When a follower thread in ordered commit waits for the leader thread to complete flush or sync, it is still counted in the active thread count of the thread group. After the thread group reaches the concurrency limit, requests in the queue of the same group must wait for the existing worker threads to continue processing.

This version adds Percona thread pool dynamic concurrency scheduling optimization. The thread pool timer periodically collects the CPU usage and context switching frequency of the mysqld process and updates the global active worker thread quota. When a follower thread in ordered commit enters the waiting state, the thread pool determines whether to wake up or create a worker thread based on the global quota, the task queue of the thread group to which it belongs, and the concurrency limit.

This version adds the following global parameters, all of which support modification at runtime without restarting the instance:

|Parameter|Default Value|Description|
|--|--|--|
|thread_pool_commit_burst_threads|ON|Controls whether worker threads are allowed to be woken up or created according to the quota during ordered commit waiting.|
|thread_pool_cpu_usage_threshold|90|Sets the process CPU usage threshold.|
|thread_pool_nvcsw_freq_threshold|10000|Sets the voluntary context switching frequency threshold.|
|thread_pool_nivcsw_freq_threshold|4294967295|Sets the involuntary context switching frequency threshold.|

This version does not change the transaction commit semantics, disk format, binlog event format, or client protocol. After `thread_pool_commit_burst_threads` is set to `OFF`, ordered commit waiting no longer triggers the operation of adding threads, and the thread pool retains the original processing behavior.

##### MySQL Redo File Reuse

This feature optimizes the processing logic for unused redo files during the InnoDB startup phase. When the `#innodb_redo` directory already contains unused redo files that meet the conditions, the system can directly reuse these files instead of deleting and then recreating them.

This change introduces the read-only startup parameter `innodb_redo_log_reuse_unused_files`, which is enabled (set to `ON`) by default. When this parameter is disabled (set to `OFF`), the system falls back to the original startup-phase cleanup logic.

##### MySQL Container-Aware Optimization

This feature optimizes the waiting strategy of InnoDB redo log background threads in high-CPU-usage scenarios. It adds the `srv_cpu_usage.utime_pct >= srv_log_spin_cpu_pct_hwm` check in `log_should_wait_for_events_without_spinning()`: When the user-space CPU usage of an instance reaches or exceeds the high water mark, the log background threads directly bypass spin waiting and switch to event waiting, thereby reducing the additional CPU consumption under high CPU load.

To make this check equally effective in containers or in cgroup-based resource-constrained scenarios (excluding containers), this change adds the capability to obtain the CPU resources and memory available to instances, and preferentially reads the resource limits of cgroup v2 or cgroup v1. If no limit is detected, the original system interfaces continue to be used. The resulting CPU count is used for the calculation of `srv_cpu_usage.utime_pct` and for other resource-related logic.

#### Resolved Issues

None

#### Known Issues

None

### Related Documentation

| Document | Description | Delivery Method |
|--|--|--|
| *Percona Thread Pool Dynamic Concurrency Scheduling Optimization Feature Guide* | Provides the environment requirements and guidance on enabling the current feature. | Open-source repository |
| *MySQL Redo File Reuse Feature Guide* | Provides the environment requirements and guidance on enabling the current feature. | Open-source repository |
| *MySQL Container-Aware Optimization Feature Guide* | Provides the environment requirements and guidance on enabling the current feature. | Open-source repository |

### Obtaining Documentation<a name="EN-US_TOPIC_0000002544372643"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/boostdb/tree/master/docs) to view or download required documents.

## 2026-06-30

### Change History

| Version| Date      | Description                                                                                                                                |
| ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 01   | 2026-06-30 |- Released the MySQL plan cache feature.<br>- Released the MySQL transaction lock optimization feature.<br>- Updated the SIMD-based MySQL character set processing optimization feature. |

### Version Mapping

#### Product Version Information

| Product Name     | Version    |
| --------- | -------- |
| Kunpeng BoostKit | 26.1.RC1 |

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
| Kunpeng BoostKit | 26.0.RC1 |

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
| Kunpeng BoostKit | 25.3.0 |

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
