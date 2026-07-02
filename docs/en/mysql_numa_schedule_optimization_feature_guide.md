# MySQL NUMA Scheduling Tuning Feature Guide

## Feature Overview

In MySQL OLTP applications, the system schedules MySQL threads on the CPUs running the OS by default in the case of high concurrency, as shown in the left part in  [Figure 1](#fig1258516181109). As a result, cross-NUMA access happens frequently, which increases CPU overheads and restricts the performance. Therefore, thread scheduling needs to be tuned to reduce the overhead of cross-NUMA access and improve system performance, as shown in the right part in  [Figure 1](#fig1258516181109).

**Figure  1**  MySQL NUMA scheduling tuning framework<a name="fig1258516181109"></a>  
![](figures/mysql-numa-scheduling-tuning-framework.png "mysql-numa-scheduling-tuning-framework")

The MySQL NUMA scheduling tuning feature implements fine-grained scheduling of MySQL foreground and background threads. This feature improves the processing efficiency of key threads and reduces remote memory access to improve system performance. For details, see  [Figure 2](#fig10678181442314).

- Background thread: This feature involves seven types of background threads that affect the system performance. The threads are related to the redo log and purge logic. The number of background threads is fixed. Each type of thread has only one thread instance and is started when the MySQL instance is started. You can specify a type of threads to run only on specified CPU cores. Proper parameter settings can isolate the CPU cores of different threads from each other so that the CPU cores can be fully scheduled, preventing from system bottlenecks.
- Foreground thread: MySQL assigns one thread for each client connection, so the number of foreground threads increases with the number of sessions. Similar to background thread scheduling, you can specify foreground threads to run on specified CPU cores. In addition, CPU cores can be grouped based on the  NUMA  information. In the lifecycle of the current session, foreground threads are migrated only on the CPU cores in the same group, but not across NUMA nodes, exploiting spatial locality in data access. To implement load balancing between groups, new foreground threads are scheduled to the group with fewer sessions.

**Figure  2**  Workflow<a name="fig10678181442314"></a>  
![](figures/workflow.png "workflow")

## Code Implementation<a name="EN-US_TOPIC_0000001827441036"></a>

[Table 1](#table10243123362211)  lists the new classes of the feature.

**Table  1**  New classes<a id="table10243123362211"></a>

|Class|Description|
|--|--|
|Sched_affinity_manager|Scheduling manager interface.The register_thread method is used to register itself with the scheduling manager when a thread starts, and then its scheduling is managed by the scheduling manager.The unregister_thread method is used to deregister a thread from the scheduling manager before the thread is destroyed.The rebalance_group method is used to update the internal status of the scheduling manager and the scheduling status of the existing threads when CPU core parameters are changed.The update_numa_aware method is used to update the internal status of the scheduling manager and the scheduling status of the existing threads when the **sched_affinity_numa_aware** parameter is changed.The take_group_snapshot method is used to return a snapshot of the internal status of the scheduler manager, in the form of a string. The snapshot can be queried by the user.The get_total_node_number method is used to return the total number of NUMA nodes in the system.The get_cpu_number_per_node method is used to return the number of cores on each NUMA node in the system.The check_cpu_string method is used to check the validity of the input CPU cores.|
|Sched_affinity_manager_numa|Implements the scheduling manager.|
|Sched_affinity_manager_dummy|Standby for implementing the scheduling manager. The implementation of all interfaces only returns values that meet the caller's expectation.If Sched_affinity_manager_numa is unavailable (for example, the libnuma dependency does not meet the requirements), Sched_affinity_manager_dummy is enabled.|

## Usage Description<a name="EN-US_TOPIC_0000001874280737"></a>

Fix vulnerabilities as soon as possible based on the Common Vulnerabilities and Exposures \(CVE\) of MySQL 8.0.20 on the  [MySQL official website](https://www.mysql.com/).

**Release Notes<a name="section1798114341485"></a>**

This feature is released with Kunpeng BoostKit 21.0.0 and Kunpeng BoostKit 22.0.0, which correspond to MySQL 8.0.20 and MySQL 8.0.25, respectively.

**Application Scenarios<a name="section032842114615"></a>**

When there are a large number of write operations \(update, insert, and delete\) in the OLTP load, plenty of redo log write requests are generated. The log write thread in the MySQL background may be overloaded, affecting the system throughput. If you find that the log thread is busy using  **log\_on\_write\_waits**  in InnoDB Monitors, use this feature to improve the log thread efficiency. In addition, if the service is suitable for NUMA affinity, this feature can be used to improve the memory access efficiency of user threads on the multi-channel server with the NUMA architecture.

After the patch is applied, recompile the MySQL database and configure system variables for the patch to take effect. For details, see  [Table 3](#table10139557193012).

**Restrictions<a name="section19587144610394"></a>**

The libnuma library is required to implement this feature. For the libnuma library, the number of configured CPU cores for API calling cannot exceed the core pinning range for starting the parent process. Otherwise, a conflict occurs. In addition, the MySQL scheduler cannot detect the change of core pinning policy implemented by other tools. Comply with the following rules when using this feature:

- When a MySQL instance is started, if a core pinning policy is set by using tools such as taskset and numactl, ensure that the configured MySQL scheduler parameters do not conflict with the core pinning policy. Otherwise, the MySQL instance fails to be started and an error log is output.
- When a MySQL instance is running, if you want to modify the MySQL scheduler parameters, ensure that they do not conflict with the core pinning policy set by using tools such as taskset and numactl when the MySQL process is started. Otherwise, the MySQL instance continues to run, but the scheduler enters the fallback mode, stops responding to internal thread scheduling requests, and generates an alarm log.
- After the MySQL instance is started, do not modify the thread pinning policy by means other than using MySQL. Otherwise, the MySQL instance continues to run, but the load information of the scheduler is inaccurate, which affects the scheduling performance.

    If the value of  **SHOW STATUS LIKE 'Sched\_affinity\_group\_number'**  is  **-1**, the feature is disabled.

**Compilation and Installation Method<a name="section92212257463"></a>**

The MySQL NUMA scheduling tuning feature is provided as a patch file. This patch is developed based on MySQL 8.0.20 or MySQL 8.0.25 and is open-sourced in the Gitee community. Before using this feature, apply the patch to the MySQL source code, and then compile and install MySQL.

1. Download the MySQL source code based on  [Table 1](#table742610259715)  and upload it to the  **/home**  directory on the server.

    **Table  1**  Download URLs for different MySQL versions<a id="table742610259715"></a>

    |Version|Download URL|
    |--|--|
    |MySQL 8.0.20|Link|
    |MySQL 8.0.25|Link|

2. Download the MySQL NUMA scheduling tuning patch based on  [Table 2](#table8561118076)  and upload it to the root directory of the MySQL source code.

    **Table  2**  Download URLs of the MySQL NUMA scheduling tuning patches<a id="table8561118076"></a>

    |Version|Download URL|
    |--|--|
    |MySQL 8.0.20 and MySQL 8.0.25|Link|

3. If the Yum source is not configured, configure it. For details, see  [Configuring the Yum Source](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0013.html).
4. This feature depends on libnuma. Install related dependencies before compiling MySQL \(take CentOS as an example\):

    ```shell
    yum install -y numactl numactl-devel numactl-libs
    ```

    >![](public_sys-resources/icon_note.gif) **NOTE:** 
    >MySQL can still be compiled even if the libnuma dependencies are not found during compilation. But this feature will not take effect.

5. Upload the MySQL source package to the  **/home**  directory, decompress the source package, and go to the root directory of the MySQL source code. \(Assume that the MySQL version is 8.0.20.\)

    ```shell
    cd /home
    tar -zxvf mysql-boost-8.0.20.tar.gz
    cd mysql-8.0.20
    ```

6. In the root directory of the source code, run the  **git init**  command to create Git management information.

    ```shell
    git init
    git add -A
    git commit -m "Initial commit"
    ```

    >![](public_sys-resources/icon_note.gif) **NOTE:** 
    >- Generally, Git is provided by the system. If not, configure the Yum source by following instructions in  [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0013.html)  and then install Git.
    >
    > ```shell
    > yum install git
    >    ```
    >
    >- If the Git commit user information is not configured, configure the user email and user name before running the  **git commit**  command.
    >
    > ```shell
    > git config user.email "123@example.com"
    > git config user.name "123"
    >    ```

7. If dos2unix is not installed, run the following command to install it:

    ```shell
    yum install dos2unix
    ```

8. Apply the NUMA scheduling tuning patch.

    ```shell
    dos2unix 0001-SCHED-AFFINITY.patch
    git apply --check 0001-SCHED-AFFINITY.patch
    git apply --whitespace=nowarn 0001-SCHED-AFFINITY.patch
    ```

    >![](public_sys-resources/icon_note.gif) **NOTE:** 
    >This step uses MySQL 8.0.20 as an example. If you want to apply this feature to other versions, modify the preceding commands based on the actual patch name.

    If no error information is displayed, the patch is successfully installed.

9. Compile and install the MySQL source code. For details, see  [MySQL Porting Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html).
10. After recompiling MySQL, configure system variables in the configuration file or boot parameters or during system running for the recompilation to take effect.

    The MySQL system variables described in  [Table 3](#table10139557193012)  are added, which can be set in the configuration file, in the startup parameters, or during system running as required.

    **Table  3**  Parameter description and recommended configuration of MySQL NUMA scheduling tuning<a id="table10139557193012"></a>

    |Parameter|Description|Recommended Configuration|
    |--|--|--|
    |sched_affinity_numa_aware|A global parameter of the Boolean type. If it is set to **ON** and **sched_affinity_foreground_thread** is not left blank, the CPU cores specified by **sched_affinity_foreground_thread** are grouped by NUMA node, and the thread of a session is migrated only between CPU cores in a specified group.This parameter can be modified when the database is running. The default value is **OFF**.|Specifies whether to enable core binding for foreground processes. If **sched_affinity_foreground_thread** is not left blank, CPU cores specified by **sched_affinity_foreground_thread** are grouped by NUMA node, and the thread of a session is migrated only between cores in a specified group. You are advised to set this parameter to **ON**.|
    |sched_affinity_foreground_thread|A global parameter of the String type. It is used to set the CPU cores that can be used for MySQL foreground threads.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that this type of threads is scheduled by the OS, that is, this parameter is not used.|Specifies the CPU cores on which MySQL foreground threads (user threads) run. You are advised to bind foreground threads and background threads to different cores.|
    |sched_affinity_log_writer|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_writer thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_writer thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL log_writer thread. You are advised to bind background threads to cores of the same NUMA node.|
    sched_affinity_log_flusher|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_flusher thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_flusher thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL log_flusher thread. You are advised to bind background threads to cores of the same NUMA node.|
    |sched_affinity_log_write_notifier|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_write_notifier thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_write_notifier thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL log_write_notifier thread. You are advised to bind background threads to cores of the same NUMA node.|
    |sched_affinity_log_flush_notifier|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_flush_notifier thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_flush_notifier thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL log_flush_notifier thread. You are advised to bind background threads to cores of the same NUMA node.|
    |sched_affinity_log_checkpointer|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_checkpointer thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_checkpointer thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL log_checkpointer thread. You are advised to bind background threads to cores of the same NUMA node.|
    |sched_affinity_purge_coordinator|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL purge_coordinator thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the purge_coordinator thread is scheduled by the OS.|Specifies the CPU cores that can be used for the MySQL purge_coordinator thread. You are advised to bind background threads to cores of the same NUMA node.|
    |sched_affinity_log_closer|A global parameter of the String type. It is used to set the CPU cores that can be used for the MySQL log_closer thread.The value is a character string consisting of digits representing core IDs. Core IDs can be separated by commas (,) and the value range can be represented by a minus sign (-). For example, the following lists valid values of CPU cores:Blank50,5,70,2-5,7This parameter can be modified when the database is running. It is left blank by default, indicating that the log_closer thread is scheduled by the OS.The log_closer thread is deleted from MySQL 8.0.25. Therefore, this parameter is not provided in the corresponding patch version.|Specifies the CPU cores that can be used for the MySQL log_closer thread. You are advised to bind background threads to cores of the same NUMA node.|

    - Method 1: Modify the configuration file. This method takes effect only after the database is restarted.
        1. Configure system variables in the configuration file. Example:

            ```txt
            sched_affinity_numa_aware=ON
            sched_affinity_foreground_thread=0-29
            sched_affinity_log_writer=30
            sched_affinity_log_flusher=30
            sched_affinity_log_write_notifier=31
            sched_affinity_log_flush_notifier=31
            sched_affinity_log_checkpointer=31
            sched_affinity_purge_coordinator=31
            ```

            >![](public_sys-resources/icon_note.gif) **NOTE:** 
            >The default path to the database configuration file is  **/etc/my.cnf**. You can also run the following command to set the  **defaults-file**  option, where  **/tmp/myconfig.txt**  indicates the configuration file path.
>
            >```
            >mysqld --defaults-file=/tmp/myconfig.txt
            >```

        2. Restart the database.

    - Method 2: Modify the database startup parameters.
        1. When starting the database, add system variable configurations to the boot command. This method takes effect only after the database is restarted. Example:

            ```
            mysqld --defaults-file=/etc/my.cnf \
            --sched_affinity_numa_aware=ON \
            --sched_affinity_foreground_thread=0-29 \
            --sched_affinity_log_writer=30 \
            --sched_affinity_log_flusher=30 \
            --sched_affinity_log_write_notifier=31 \
            --sched_affinity_log_flush_notifier=31 \
            --sched_affinity_log_checkpointer=31 \
            --sched_affinity_purge_coordinator=31
            ```

        2. Restart the database.

    - Method 3: Connect to the database during system running and configure system variables using SQL statements. This method does not require restarting the database. Example:

        ```
        set global sched_affinity_numa_aware=ON;
        set global sched_affinity_foreground_thread="0-29";
        set global sched_affinity_log_writer="30";
        set global sched_affinity_log_flusher="30";
        set global sched_affinity_log_write_notifier="31";
        set global sched_affinity_log_flush_notifier="31";
        set global sched_affinity_log_checkpointer="31";
        set global sched_affinity_purge_coordinator="31";
        ```

11. Check the MySQL status variables.

    The MySQL status variables described in  [Table 4](#table16657323173817)  are added in this feature, enabling you to query the internal status of the scheduling manager.

    **Table  4**  MySQL status variables<a id="table16657323173817"></a>

    |Status Variable|Description|
    |--|--|
    |Sched_affinity_status|Returns the load status of each group in the scheduling manager.|
    |Sched_affinity_group_number|Returns the total number of NUMA nodes in the system.|
    |Sched_affinity_group_capacity|Returns the number of cores on each NUMA node.|

    After the MySQL NUMA scheduling tuning feature is enabled, execute the following SQL statement to view information about MySQL status variables:

    ```
    show status like "%status variable name%";
    ```

12. Perform a TPC-C test to obtain the performance improvement data after the MySQL NUMA scheduling tuning feature is used. For details about the test, see  [BenchMarkSQL Test Guide](https://www.hikunpeng.com/document/detail/en/kunpengdbs/testguide/tstg/kunpengbenchmarksql_06_0001.html).

    The MySQL NUMA scheduling tuning feature improves the comprehensive TPC-C performance by 10%.  [Figure 1](#fig20274152011365)  shows the effect before and after the tuning.

    **Figure  1**  Performance comparison before and after MySQL NUMA scheduling tuning is used<a name="fig20274152011365"></a>  
    ![](figures/performance-comparison-before-and-after-mysql-numa-scheduling-tuning-is-used.png "performance-comparison-before-and-after-mysql-numa-scheduling-tuning-is-used")

## Change History<a name="EN-US_TOPIC_0000001827281212"></a>

|Date|Description|
|--|--|
|2023-07-25|This issue is the second official release.Updated the commands for applying the patch of the MySQL NUMA scheduling tuning feature in Usage Description.|
|2021-06-30|This issue is the first official release.|
