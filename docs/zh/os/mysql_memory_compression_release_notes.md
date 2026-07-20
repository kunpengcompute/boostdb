# MySQL内存压缩 版本说明书

## 版本配套说明<a name="ZH-CN_TOPIC_0000002603100102"></a>

### 产品版本信息<a name="ZH-CN_TOPIC_0000002603100103"></a>

<a name="table_memory_compression_release_info"></a>
<table><tbody><tr><th class="firstcol" valign="top" width="26.61%">产品名称</th>
<td class="cellrowborder" valign="top" width="73.39%">Kunpeng BoostKit</td>
</tr>
<tr><th class="firstcol" valign="top" width="26.61%">产品版本</th>
<td class="cellrowborder" valign="top" width="73.39%">26.1.T7</td>
</tr>
<tr><th class="firstcol" valign="top" width="26.61%">特性名称</th>
<td class="cellrowborder" valign="top" width="73.39%">MySQL内存压缩</td>
</tr>
</tbody></table>

### 软件版本配套说明<a name="ZH-CN_TOPIC_0000002603100104"></a>

|软件类型|版本|
|--|--|
|OS|openEuler定制内核，源码分支`OLK-5.10`|
|Kernel Config|`64K`基础页|
|Percona|Percona-Server 5.7.44-53|
|Kernel Patch|内存压缩特性相关2个OS补丁|
|OS Runtime Config|`min_free_kbytes`、`watermark_boost_factor`、`shrink_lruvec_strict`、`numa_demotion_enabled`、zram、KSM|

### 硬件版本配套说明<a name="ZH-CN_TOPIC_0000002603100105"></a>

|项目|要求|
|--|--|
|处理器|鲲鹏920系列处理器|
|内存|推荐`512GB`及以上物理内存|

### 病毒扫描结果<a name="ZH-CN_TOPIC_0000002603100106"></a>

不涉及软件包发布，不涉及病毒扫描。

## v26.1.T7<a name="ZH-CN_TOPIC_0000002603100107"></a>

### 更新说明<a name="ZH-CN_TOPIC_0000002603100108"></a>

新增MySQL内存压缩特性。该特性依赖OS内核补丁能力和`64K`基础页内核配置，通过调优`min_free_kbytes`、`watermark_boost_factor`、`shrink_lruvec_strict`、`numa_demotion_enabled`等内存管理参数，并结合zram swap、KSM重复页合并和MySQL `470GB InnoDB Buffer Pool`配置，提升减配场景下的可用缓存空间，降低高内存水位下的direct reclaim和OOM风险。

### 已解决的问题<a name="ZH-CN_TOPIC_0000002603100109"></a>

无。

### 遗留问题<a name="ZH-CN_TOPIC_0000002603100110"></a>

无。

## 版本配套文档<a name="ZH-CN_TOPIC_0000002603100111"></a>

### 版本配套文档<a name="ZH-CN_TOPIC_0000002603100112"></a>

|文档名称|内容简介|交付形式|
|--|--|--|
|《Kunpeng BoostKit 26.1.T7 MySQL内存压缩 版本说明书》|提供MySQL内存压缩特性的版本发布及其配套信息。|开源仓|
|《Kunpeng BoostKit 26.1.T7 MySQL内存压缩 特性指南》|提供MySQL内存压缩特性的环境要求、内核编译安装、OS参数配置、KSM与zram使能和使用说明。|开源仓|

### 获取文档的方法<a name="ZH-CN_TOPIC_0000002603100113"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostdb)浏览和获取相关文档。
