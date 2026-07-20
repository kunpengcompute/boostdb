# MySQL内存压缩 特性指南

## 特性描述<a name="ZH-CN_TOPIC_0000002603100002"></a>

### 简介<a name="ZH-CN_TOPIC_0000002603100003"></a>

本文主要介绍如何在鲲鹏服务器上使能MySQL内存压缩特性。

在服务器物理内存减配场景中，如果仅按照常规比例缩小`InnoDB Buffer Pool`，会导致缓存能力明显下降，磁盘访问增加，并进一步带来后台回收压力增大、业务线程进入direct reclaim甚至OOM等问题。针对这类问题，鲲鹏BoostKit提供了一套MySQL配合OS自动内存压缩的方案，通过优化Linux内存回收策略、引入zram承接少量低活跃匿名页、启用KSM合并重复匿名页、利用NUMA节点间压力分摊机制，在控制性能劣化的同时提升减配后的可用缓存空间。

本文以Percona-Server 5.7.44-53为例，介绍如何编译依赖的OS内核、安装内核、配置OS内存压缩参数，并完成MySQL侧的配套配置与验证。

### 原理描述<a name="ZH-CN_TOPIC_0000002603100004"></a>

MySQL内存压缩特性主要由以下几个部分组成：

- 通过下调`min_free_kbytes`释放原先由内核静态保留、但在当前数据库场景下利用率较低的内存，提升减配后的实际可用空间。
- 通过watermark机制更早唤醒`kswapd`，尽量由后台线程完成回收，降低MySQL业务线程进入direct reclaim的概率。
- 通过设置`watermark_boost_factor=0`与`shrink_lruvec_strict=1`，控制单轮回收量和回收范围，减少超量回收带来的抖动。
- 通过配置zram swap，在回收文件页之外增加匿名页回收能力，降低高内存水位下的OOM风险。
- 通过配置KSM（Kernel Samepage Merging）与ksmd扫描，将内容相同的匿名页合并为共享页，并将全`0`页合并到系统`0`页，进一步降低匿名页占用。
- 在多NUMA节点场景下，通过`numa_demotion_enabled=1`让系统优先在节点间分摊内存压力，提升整机内存利用效率。

该特性不新增MySQL应用层接口，核心能力由OS侧提供，MySQL侧主要配合调整`InnoDB Buffer Pool`等运行参数。

## 环境要求<a name="ZH-CN_TOPIC_0000002603100005"></a>

本文基于特定环境提供指导，在正式操作前请确保软硬件环境满足要求。

**表1** 硬件要求<a id="memory_compression_hardware_requirements"></a>

|项目|规格|
|--|--|
|CPU|鲲鹏920系列处理器、鲲鹏950处理器|
|内存|建议验证环境具备`512GB`及以上物理内存|

**表2** 操作系统和软件要求<a id="memory_compression_software_requirements"></a>

|项目|名称|版本|获取地址|
|--|--|--|--|
|操作系统|openEuler内核源码仓|`OLK-5.10`分支|[获取链接](https://atomgit.com/openeuler/kernel.git)|
|内核配置|基础页大小|`64K`页|内核源码编译配置|
|Percona|Percona-Server|5.7.44-53|[获取链接](https://github.com/percona/percona-server/archive/refs/tags/Percona-Server-5.7.44-53.tar.gz)|
|内核补丁|内存压缩特性OS补丁|2个补丁文件|[获取链接](https://www.atomgit.com/openeuler/kernel/pull/24858)|

>![](../public_sys-resources/icon_note.gif) **说明：**
>
>- 本特性依赖OS内核补丁能力，需先完成定制内核编译与安装。
>- 文中使用`git am *.patch`合入补丁，要求两个内核补丁文件已放置在内核源码根目录或当前执行目录。

## 安装和使用特性<a name="ZH-CN_TOPIC_0000002603100006"></a>

以Percona-Server 5.7.44-53配合定制OS内核为例，具体操作如下。

### 编译并安装内核

1. 获取openEuler内核源码并切换到`OLK-5.10`分支。

   ```bash
   git clone https://atomgit.com/openeuler/kernel.git
   cd kernel
   git checkout OLK-5.10
   ```

2. 将2个内核补丁文件放到源码根目录后执行补丁合入。

   ```bash
   git am *.patch
   ```

3. 加载默认配置。

   ```bash
   make openeuler_defconfig
   ```

4. 执行`menuconfig`，打开`64K`基础页配置。

   ```bash
   make menuconfig
   ```

5. 检查`64K`页配置是否已生效。

   ```bash
   cat .config | grep 64K
   ```

6. 再次执行`menuconfig`，按发布要求设置内核包名。

   ```bash
   make menuconfig
   ```

7. 安装内核编译依赖。

   ```bash
   yum install -y rpm-build openssl-devel bc rsync gcc gcc-c++ flex bison m4 dwarves ncurses-devel
   ```

   >![](../public_sys-resources/icon_note.gif) **说明：**
   >部分环境在执行内核编译时，可能会因为缺少`rpm-build`、`bc`、`dwarves`、`ncurses-devel`等依赖而报错。建议在编译前先安装上述依赖包。

8. 编译内核rpm包。

   ```bash
   make INSTALL_MOD_STRIP=1 binrpm-pkg -j380
   ```

9. 安装生成的内核rpm包。

   ```bash
   rpm -ivh xxx.rpm --oldpackage
   ```

10. 将新内核设置为默认启动项，并确认默认内核信息。

    ```bash
    grubby --set-default-index=0
    grubby --default-kernel
    grubby --info=0
    ```

11. 重启系统后，确认已进入新内核。

    ```bash
    uname -r
    getconf PAGE_SIZE
    ```

    预期结果：

    - `uname -r`显示为新编译安装的内核版本。
    - `getconf PAGE_SIZE`输出为`65536`。

### 安装MySQL

1. 根据Percona-Server 5.7.44-53的常规流程编译并安装MySQL。详细信息请参见《[Percona移植指南](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/Percona/kunpengpercona_02_0001.html)》。

2. 完成实例初始化并启动MySQL服务，启动数据库的操作请参见《MySQL 移植指南》的[运行MySQL](https://www.hikunpeng.com/document/detail/zh/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0013.html)章节。

3. 根据减配方案调整`InnoDB Buffer Pool`大小，例如将`Buffer Pool`设置为`470GB`。

   ```ini
   [mysqld]
   innodb_buffer_pool_size=470G
   ```

4. 重启MySQL并确认参数生效。

   ```sql
   SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
   ```

### 配置OS内存压缩参数

本节按功能点说明MySQL内存压缩特性涉及的OS接口、合法取值与本特性推荐取值。取值范围依据openEuler内核源码`OLK-5.10`分支核实。

#### 禁用THP并下调min水线

本功能点涉及透明大页开关和最低空闲内存水线，配置如下。

|接口|功能|合法取值|本特性取值|
|--|--|--|--|
|`/sys/kernel/mm/transparent_hugepage/enabled`|透明大页使能开关|`always`表示全开启<br/>`madvise`表示仅对`MADV_HUGEPAGE`项开启<br/>`never`表示全关闭|`never`|
|`/proc/sys/vm/min_free_kbytes`|设置各zone的最低空闲内存水线，单位为KB。下调后释放原先由内核静态保留的内存|整数，`>=0`|`91589`|

#### 配置KSM合并重复匿名页

本功能点采用全局使能KSM、专用cgroup定向扫描的方式，只把`mysqld`纳入扫描。

|接口|功能|合法取值|本特性取值|
|--|--|--|--|
|`/sys/kernel/mm/ksm/pages_to_scan`|设置每轮扫描的页数。值越大合并越快，但CPU占用越高|整数，`0`到`UINT_MAX`|`100000`|
|`/sys/kernel/mm/ksm/use_zero_pages`|是否把内容全为`0`的页合并到系统`0`页|`0`表示关闭<br>`1`表示开启|`1`|
|`/sys/kernel/mm/ksm/run`|启动或停止KSM扫描|`0`表示停止<br>`1`表示运行<br>`2`表示解除已有合并|`1`|
|`/sys/fs/cgroup/memory/<cg>/memory.ksm`|对该cgroup内的进程开启或关闭KSM扫描|`0`表示关闭<br>`1`表示开启|`1`|
|`/sys/fs/cgroup/memory/<cg>/cgroup.procs`|把进程加入该cgroup，使其匿名页被KSM扫描|pid列表|`pidof mysqld`|

KSM只读统计接口用于观察合并效果。

|接口|含义|
|--|--|
|`/sys/kernel/mm/ksm/pages_shared`|已合并的共享页数。值越大，省下的内存越多|
|`/sys/kernel/mm/ksm/pages_sharing`|指向共享页的页数。`pages_sharing`与`pages_shared`的差值越大，合并收益越高|
|`/sys/kernel/mm/ksm/pages_unshared`|扫描过但未找到重复、未能合并的页数|
|`/sys/kernel/mm/ksm/full_scans`|全量扫描已完成轮数|

`memory.ksm`接口也可以读取该cgroup的合并页数、节省内存等统计信息，用于验证定向扫描效果。

#### 配置zram swap承接匿名页回收

本功能点使用zram作为匿名页回收的swap承接载体，相关接口如下。

|接口|功能|合法取值|本特性取值|
|--|--|--|--|
|`/sys/block/zram0/reset`|重置zram设备|非`0`整数|`1`|
|`/sys/block/zram0/comp_algorithm`|设置zram压缩算法|内核已注册的压缩算法名，例如`zstd`、`lzo`、`lz4`|`zstd`|
|`/sys/block/zram0/disksize`|设置zram块设备大小|支持`K`、`M`、`G`后缀，`0`无效|`512G`|

配置zram设备后，还需要执行`mkswap /dev/zram0`格式化，并执行`swapon /dev/zram0`或带优先级参数的`swapon -p 100 /dev/zram0`启用为swap。

#### 保持高内存利用率并降低压缩量

本功能点通过回收控制和NUMA demotion降低超量回收与实际压缩量，相关接口如下。

|接口|功能|合法取值|本特性取值|
|--|--|--|--|
|`/proc/sys/vm/watermark_boost_factor`|碎片化水位提升因子。发生内存碎片化时，`kswapd`会额外回收内存，上限为high水位的`watermark_boost_factor`/`10000`倍，默认`15000`表示`150%`。该机制用于给内存规整让路，提高THP等高阶分配成功率，但会带来超量回收，本特性置`0`关闭|整数，`>=0`|`0`|
|`/proc/sys/vm/shrink_lruvec_strict`（新增）|严格回收开关。置`1`后回收达到目标量即提前退出本轮，不再多扫其他LRU，不再超量回收|`0`表示关闭<br/>`1`表示开启|`1`|
|`/proc/sys/vm/numa_demotion_enabled`（新增）|NUMA demotion开关。置`1`后回收前优先把页迁移到其他有空闲内存的NUMA节点，而非直接压缩或换出|`0`表示关闭<br/>`1`表示开启|`1`|

完成上述接口确认后，按以下步骤配置OS内存压缩参数。

1. 关闭透明大页。

   ```bash
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
   ```

2. 设置内存回收和NUMA相关参数。

   ```bash
   echo 91589 > /proc/sys/vm/min_free_kbytes
   echo 0 > /proc/sys/vm/watermark_boost_factor
   echo 1 > /proc/sys/vm/shrink_lruvec_strict
   echo 1 > /proc/sys/vm/numa_demotion_enabled
   ```

3. 全局使能KSM/ksmd扫描。

   ```bash
   echo 100000 > /sys/kernel/mm/ksm/pages_to_scan
   echo 1 > /sys/kernel/mm/ksm/use_zero_pages
   echo 1 > /sys/kernel/mm/ksm/run
   ```

4. 创建专用memory cgroup并打开KSM能力。

   ```bash
   mkdir /sys/fs/cgroup/memory/ksm_test
   echo 1 > /sys/fs/cgroup/memory/ksm_test/memory.ksm
   ```

   >![](../public_sys-resources/icon_note.gif) **说明：**
   >上述`mkdir`和`memory.ksm`配置在每次服务器重启后执行一次即可。

5. 关闭原有swap。

   ```bash
   swapoff -a
   ```

6. 加载zram模块并完成zram swap配置。

   ```bash
   modprobe zram
   echo 1 > /sys/block/zram0/reset
   echo zstd > /sys/block/zram0/comp_algorithm
   echo 512G > /sys/block/zram0/disksize
   mkswap /dev/zram0
   swapon -p 100 /dev/zram0
   ```

7. 在`mysqld`启动后，将数据库进程加入KSM扫描队列。

   ```bash
   echo `pidof mysqld` > /sys/fs/cgroup/memory/ksm_test/cgroup.procs
   ```

8. 验证参数、KSM和zram状态。

   ```bash
   cat /proc/sys/vm/min_free_kbytes
   cat /proc/sys/vm/watermark_boost_factor
   cat /proc/sys/vm/shrink_lruvec_strict
   cat /proc/sys/vm/numa_demotion_enabled
   cat /sys/kernel/mm/transparent_hugepage/enabled
   cat /sys/kernel/mm/ksm/pages_to_scan
   cat /sys/kernel/mm/ksm/use_zero_pages
   cat /sys/kernel/mm/ksm/run
   cat /sys/fs/cgroup/memory/ksm_test/memory.ksm
   swapon --show
   ```

9. 如需观察KSM合并效果，可检查ksm统计信息。

   ```bash
   cat /sys/kernel/mm/ksm/pages_shared
   cat /sys/kernel/mm/ksm/pages_sharing
   cat /sys/kernel/mm/ksm/pages_unshared
   cat /sys/kernel/mm/ksm/full_scans
   ```

### 功能验证

1. 检查系统页大小、透明大页、KSM、zram和内存参数状态。

   ```bash
   getconf PAGE_SIZE
   cat /sys/kernel/mm/transparent_hugepage/enabled
   cat /sys/kernel/mm/ksm/run
   cat /sys/fs/cgroup/memory/ksm_test/cgroup.procs
   swapon --show
   numactl -H
   ```

2. 观察回收与swap行为。

   ```bash
   vmstat 1
   cat /proc/vmstat | egrep "pgscan|pgsteal|pswpin|pswpout"
   ```

3. 在压测过程中观察是否存在明显direct reclaim、swap抖动或OOM，并观察KSM是否持续产生共享页。

   ```bash
   cat /sys/kernel/mm/ksm/pages_shared
   cat /sys/kernel/mm/ksm/pages_sharing
   ```

4. 以如下两组配置进行对比测试：

   - 基线组：`768GB`物理内存，`576GB Buffer Pool`。
   - 优化组：`512GB`物理内存，`470GB Buffer Pool`，使能本特性全部OS参数。

5. 若优化组相较基线组的性能劣化控制在`10%`以内，且系统运行过程中未出现明显OOM、zram异常增长或持续direct reclaim，同时KSM存在有效共享页增长，则说明特性使能成功。

## 功能限制<a name="ZH-CN_TOPIC_0000002603100007"></a>

- 本特性依赖定制OS内核，未安装对应内核补丁时无法完整生效。
- 本特性要求使用`64K`基础页内核。
- KSM扫描会带来一定CPU开销，建议仅将`mysqld`加入专用cgroup进行定向扫描，不建议对全系统匿名页无差别开启。
- `numa_demotion_enabled=1`仅在多NUMA节点场景下具备实际收益。
- zram容量、`min_free_kbytes`和`Buffer Pool`大小需要结合具体服务器规格与业务负载评估，不建议在生产环境中直接照搬。
- 本特性不改变MySQL协议和SQL语义，优化重点在于OS内存管理路径。

## 安全检查与加固<a name="ZH-CN_TOPIC_0000002603100008"></a>

ASLR（Address Space Layout Randomization，地址空间布局随机化）是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。

```bash
echo 2 >/proc/sys/kernel/randomize_va_space
```

![](../figures/zh_cn_image_0000002504021297.png)
