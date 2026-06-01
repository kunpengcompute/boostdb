# BoostKit MySQL

## Project Brand Name

MySQL

### Latest Updates

* [2026-01-21] Updated the `MySQL-Percona-Server-8.0.43-34` branch, with the LSE atomic instruction and rec_get_offset optimization feature released. The latest commit ID is `3e35796`.
* [2026-01-21] Updated the `MySQL-Percona-Server-5.7.44-53` branch, with the LSE atomic instruction and rec_get_offset optimization feature released. The latest commit ID is `63b229f`.
* [2025-12-20] Released the `MySQL-Percona-Server-8.0.43-34-v2` RPM package, which integrates five performance optimization patches and uses the Kunpeng GCC Continuous Feature Guided Optimization (CFGO) feature.
* [2025-12-20] Released the `MySQL-Percona-Server-5.7.44-53-v3` RPM package, which integrates five performance optimization patches and uses the Kunpeng GCC CFGO feature.

### Project Introduction

BoostKit MySQL is a repository with build and optimization patches for Percona Server based on the Kunpeng platform. It provides:

- Percona version build resources with branch-based maintenance
- `boostdb-patches` patch set, including patches for vectorization processing based on the NEON instruction set, unaligned memory access optimization, InnoDB, and binlog
- Regular `rpmbuild` and GCC CFGO (GCC for openEuler) build entry

The repository branches are as follows:


| Branch                            | Description              | Link                                                                                                    |
| ---------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------- |
| `master`                         | Overview and entry        | [master](https://gitcode.com/boostkit/mysql/tree/master)                                                 |
| `MySQL-8.0.32`                   | Reserved branch          | [MySQL-8.0.32](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.32)                                     |
| `MySQL-8.0.36`                   | Reserved branch          | [MySQL-8.0.36](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.36)                                     |
| `MySQL-Percona-Server-5.7.44-53` | Build and patch content for 5.7| [MySQL-Percona-Server-5.7.44-53](https://gitcode.com/boostkit/mysql/tree/MySQL-Percona-Server-5.7.44-53) |
| `MySQL-Percona-Server-8.0.43-34` | Build and patch content for 8.0| [MySQL-Percona-Server-8.0.43-34](https://gitcode.com/boostkit/mysql/tree/MySQL-Percona-Server-8.0.43-34) |
| `MySQL-5.7.27`                   | Patch branch for 5.7    | [MySQL-5.7.27](https://gitcode.com/boostkit/mysql/tree/MySQL-5.7.27)                                     |
| `MySQL-8.0.20`                   | Patch branch for 8.0.20 | [MySQL-8.0.20](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.20)                                     |
| `MySQL-8.0.25`                   | Patch branch for 8.0.25 | [MySQL-8.0.25](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.25)                                     |
| `MySQL-8.0.30`                   | Patch branch for 8.0.30 | [MySQL-8.0.30](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.30)                                     |
| `MySQL-8.0.35`                   | Patch branch for 8.0.35 | [MySQL-8.0.35](https://gitcode.com/boostkit/mysql/tree/MySQL-8.0.35)                                     |

### Directory Structure

```text
mysql/
├── README.md                                   # Entry to the repository overview, providing branch details and feature navigation
├── docs/                                       # Repository document directory
│   ├── LICENSE                                 # Document license
│   └── en/                                     # Feature documents and images
├── (branch) MySQL-Percona-Server-5.7.44-53/
│   ├── README.md                               # 5.7.44-53 branch overview and patch description
│   ├── boostdb-patches/                        # Optimization patch directory of the 5.7.44-53 branch
│   │   ├── series                              # Patch enabling sequence and application list
│   │   └── *.patch                             # Performance optimization patch files (character set, binlog, InnoDB, LSE, etc.)
│   ├── regular_rpmbuild/                       # Regular rpmbuild directory
│   │   ├── README.md                           # Regular build steps and dependencies
│   │   └── percona.spec                        # SPEC file for regular RPM build
│   ├── gcc_for_openEuler_enhanced_rpmbuild/    # CFGO build directory (GCC for openEuler)
│   │   ├── README.md                           # Build description (GCC for openEuler)
│   │   ├── percona_a_fot.spec                  # SPEC file for CFGO RPM build
│   │   └── profile_data_20251230.tar.gz        # Profile data package required for CFGO
│   └── gcc_for_opensource_enhanced_rpmbuild/   # Profile-guided optimization (PGO) build directory (open-source GCC)
│       └── README.md                           # PGO build description (open-source GCC)
├── (branch) MySQL-Percona-Server-8.0.43-34/
│   ├── README.md                               # 8.0.43-34 branch overview and patch description
│   ├── boostdb-patches/                        # Optimization patch directory of the 8.0.43-34 branch
│   │   ├── series                              # Patch enabling sequence and application list
│   │   └── *.patch                             # Performance optimization patch files (character set, binlog, InnoDB, LSE, etc.)
│   ├── regular_rpmbuild/                       # Regular rpmbuild directory
│   │   ├── README.md                           # Regular build steps and dependencies
│   │   └── percona.spec                        # SPEC file for regular RPM build
│   ├── gcc_for_openEuler_enhanced_rpmbuild/    # CFGO build directory (GCC for openEuler)
│   │   ├── README.md                           # Build description (GCC for openEuler)
│   │   ├── percona_a_fot.spec                  # SPEC file for CFGO RPM build
│   │   └── profile_data_20251230.tar.gz        # Profile data package required for CFGO
│   └── gcc_for_opensource_enhanced_rpmbuild/   # PGO build directory (open-source GCC)
│       └── README.md                           # PGO build description (open-source GCC)
├── (branch) MySQL-5.7.27/
│   └── boostdb-patches/                        # Patch directory of the MySQL 5.7.27 branch
│       └── *.patch                             # Patch files for features such as thread pool
├── (branch) MySQL-8.0.20/
│   └── boostdb-patches/                        # Patch directory of the MySQL 8.0.20 branch
│       └── *.patch                             # Patch files for features such as thread pool, parallel query, and lock tuning
├── (branch) MySQL-8.0.25/
│   └── boostdb-patches/                        # Patch directory of the MySQL 8.0.25 branch
│       └── *.patch                             # Patch files for features such as CRC32, parallel query, and pluggable thread pool
├── (branch) MySQL-8.0.30/
│   └── boostdb-patches/                        # Patch directory of the MySQL 8.0.30 branch
│       └── *.patch                             # Pluggable thread pool patch file
└── (branch) MySQL-8.0.35/
    └── boostdb-patches/                        # Patch directory of the MySQL 8.0.35 branch
        └── *.patch                             # Pluggable thread pool patch file
```

### Feature Introduction

<table>
<thead>
<tr>
<th>Category</th>
<th>Feature Name</th>
<th>Feature Document</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="3">Basic computing acceleration</td>
<td>CRC32 instruction optimization</td>
<td><a href="docs/en/crc32_instruction_optimization_feature_guide.md">crc32_instruction_optimization_feature_guide.md</a></td>
<td>The software implementation is replaced with the Kunpeng CRC32 hardware instructions, reducing the verification and computing overhead.</td>
</tr>
<tr>
<td>Computing path optimization</td>
<td><a href="docs/en/mysql_computing_path_optimization_feature_guide.md">mysql_computing_path_optimization_feature_guide.md</a></td>
<td>The OLTP hotspot functions and memory access paths are optimized.</td>
</tr>
<tr>
<td>LSE and rec_get_offsets optimization</td>
<td><a href="docs/en/mysql_lse_and_rec_get_offsets_optimization_feature_guide.md">mysql_lse_and_rec_get_offsets_optimization_feature_guide.md</a></td>
<td>The LSE atomic instruction and rec_get_offsets call optimizations are used to improve high-concurrency performance.</td>
</tr>
<tr>
<td rowspan="3">Pluggable thread pool</td>
<td>MySQL 5.7.27 thread pool</td>
<td><a href="docs/en/mysql_5_7_27_thread_pool_feature_guide.md">mysql_5_7_27_thread_pool_feature_guide.md</a></td>
<td>The thread pool connector is used to reduce thread switching and lock contention in massive-connection scenarios.</td>
</tr>
<tr>
<td>MySQL 8.0.20 thread pool</td>
<td><a href="docs/en/mysql_8_0_20_thread_pool_feature_guide.md">mysql_8_0_20_thread_pool_feature_guide.md</a></td>
<td>The connection scheduling is optimized in the thread pool model to improve the performance of high-concurrency short queries.</td>
</tr>
<tr>
<td>MySQL 8.0.25, 8.0.30, and 8.0.35 pluggable thread pool</td>
<td><a href="docs/en/mysql_8_0_25_8_0_30_and_8_0_35_pluggable_thread_pool_feature_guide.md">mysql_8_0_25_8_0_30_and_8_0_35_pluggable_thread_pool_feature_guide.md</a></td>
<td>The pluggable thread pool plugin that can be dynamically loaded is supported to enhance the throughput in scenarios with a large number of connections.</td>
</tr>
<tr>
<td>Binlog acceleration</td>
<td>Binlog optimization</td>
<td><a href="docs/en/mysql_binlog_optimization_feature_guide.md">mysql_binlog_optimization_feature_guide.md</a></td>
<td>Binlog is optimized based on the key path of two-phase commit to improve the performance in write scenarios.</td>
</tr>
<tr>
<td rowspan="2">AP enhancement</td>
<td>Parallel query tuning</td>
<td><a href="docs/en/mysql_parallel_query_tuning_feature_guide.md">mysql_parallel_query_tuning_feature_guide.md</a></td>
<td>Parallel query tuning and patch enabling capabilities are provided for OLAP scenarios.</td>
</tr>
<tr>
<td>Pluggable online vectorized analysis engine</td>
<td><a href="docs/en/mysql_pluggable_online_vectorized_analysis_engine_feature_guide.md">mysql_pluggable_online_vectorized_analysis_engine_feature_guide.md</a></td>
<td>Parallel query tuning and pluggable capabilities are provided for OLAP scenarios.</td>
</tr>
<tr>
<td rowspan="3">Lock and atomic variable optimization</td>
<td>Lock-free tuning</td>
<td><a href="docs/en/mysql_lock_free_tuning_feature_guide.md">mysql_lock_free_tuning_feature_guide.md</a></td>
<td>The transaction management path is reconstructed using a lock-free hash table to reduce concurrent lock conflicts.</td>
</tr>
<tr>
<td>Fine-grained lock tuning</td>
<td><a href="docs/en/mysql_fine_grained_lock_tuning_feature_guide.md">mysql_fine_grained_lock_tuning_feature_guide.md</a></td>
<td>The global lock is replaced with fine-grained hash bucket locks to improve the DML concurrency capability.</td>
</tr>
<tr>
<td>Other features</td>
<td><a href="docs/en/other_mysql_features.md">other_mysql_features.md</a></td>
<td>hash_table_locks, undo_spaces_lock, and thread counter optimization are provided.</td>
</tr>
</tbody>
</table>

### MySQL Compatibility Information


| Version Branch                                 | Upstream Version                | CPU           | OS           | GCC/Toolchain                                            |
| ------------------------------------------- | -------------------------- | --------------- | --------------------- | -------------------------------------------------------- |
| `MySQL-Percona-Server-5.7.44-53` regular  | Percona-Server-5.7.44-53 | Kunpeng 920 series| openEuler 22.03 SP4 | Open-source GCC 10.3.1                                       |
| `MySQL-Percona-Server-5.7.44-53` enhanced | Percona-Server-5.7.44-53 | Kunpeng 920 series| openEuler 22.03 SP4 | GCC for openEuler 12.3.1/Open-source GCC 12.3.0|
| `MySQL-Percona-Server-8.0.43-34` regular  | Percona-Server-8.0.43-34 | Kunpeng 920 series| openEuler 22.03 SP4 | Open-source GCC                            |
| `MySQL-Percona-Server-8.0.43-34` enhanced | Percona-Server-8.0.43-34 | Kunpeng 920 series| openEuler 22.03 SP4 | GCC for openEuler 12.3.1/Open-source GCC 12.3.0|

### Constraints and Precautions

- The function content of the `MySQL-8.0.32` and `MySQL-8.0.36` branches are to be supplemented.
- During RPM installation, conflicts with `mysql-config` of the local MariaDB may occur. In this case, use `--replacefiles` as needed.
- If the dependency check fails, you can use `--nodeps` as needed when the environment is controllable.
- The `gcc_for_opensource_enhanced_rpmbuild` directory mainly contains description documents. For details about the build scripts, see the subsequent repository updates.

### Branch Maintenance Strategy


| Branch                            | Current Status    | Last Update Date| Description                              |
| ---------------------------------- | -------------- | -------------- | ------------------------------------ |
| `master`                         | Maintenance        | 2025-08-30   | Currently used for repository entry and navigation.            |
| `MySQL-8.0.32`                   | Development (reserved)| 2025-08-30   | Reserved.                              |
| `MySQL-8.0.36`                   | Development (reserved)| 2025-08-30   | Reserved.                              |
| `MySQL-Percona-Server-5.7.44-53` | Maintenance        | 2026-01-21   | Has complete patches and build content, and multiple tags.|
| `MySQL-Percona-Server-8.0.43-34` | Maintenance        | 2026-01-21   | Has complete patches and build content, and multiple tags.|
| `MySQL-5.7.27`                   | Maintenance        | 2026-03-16   | The corresponding <code>boostdb-patches</code> content has been uploaded.   |
| `MySQL-8.0.20`                   | Maintenance        | 2026-03-16   | The corresponding <code>boostdb-patches</code> content has been uploaded.   |
| `MySQL-8.0.25`                   | Maintenance        | 2026-03-16   | The corresponding <code>boostdb-patches</code> content has been uploaded.   |
| `MySQL-8.0.30`                   | Maintenance        | 2026-03-16   | The corresponding <code>boostdb-patches</code> content has been uploaded.   |
| `MySQL-8.0.35`                   | Maintenance        | 2026-03-16   | The corresponding <code>boostdb-patches</code> content has been uploaded.   |

### Version Maintenance Strategy


| Version                | Maintenance Strategy| Current Status| Next Status                   | EOL Date|
| ---------------------- | ---------- | ---------- | ----------------------------- | ---------- |
| Percona-5.7.44-53    | Regular branch| Maintenance    |                             | -        |
| Percona-8.0.43-34    | Regular branch| Maintenance    |                             | -        |
| MySQL-8.0.32 (reserved)| Reserved branch| Development    | It is expected to enter the maintenance phase from June 30, 2026.| -        |
| MySQL-8.0.36 (reserved)| Reserved branch| Development    | It is expected to enter the maintenance phase from June 30, 2026.| -        |
| MySQL-5.7.27         | Patch branch| Maintenance   |                             | -        |
| MySQL-8.0.20         | Patch branch| Maintenance   |                             | -        |
| MySQL-8.0.25         | Patch branch| Maintenance   |                             | -        |
| MySQL-8.0.30         | Patch branch| Maintenance   |                             | -        |
| MySQL-8.0.35         | Patch branch| Maintenance   |                             | -        |

### Contribution Statement

You are welcome to contribute to the community. If you have any questions, suggestions, feature requirements, or bug reports, please submit:

- [Issues](https://gitcode.com/boostkit/mysql/issues)

For details about the contribution process, see:

- [BoostKit Community Contribution Guide](https://gitcode.com/boostkit/community/blob/master/docs/contributor/contributing.md)

You are also welcome to share insights in:

- [BoostKit Discussions](https://gitcode.com/boostkit/community/discussions)

### License

The documents of this project are licensed under CC-BY 4.0. For details, see [LICENSE](docs/LICENSE).
