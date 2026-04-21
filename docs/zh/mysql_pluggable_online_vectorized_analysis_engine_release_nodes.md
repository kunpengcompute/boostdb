# MySQL可插拔在线向量化分析引擎 版本说明书

## 版本配套说明<a name="ZH-CN_TOPIC_0000002550183837"></a>

### 产品版本信息<a name="ZH-CN_TOPIC_0000002518703998"></a>

<a name="table215mcpsimp"></a>
<table><tbody><tr id="row220mcpsimp"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.1.1"><p id="p222mcpsimp"><a name="p222mcpsimp"></a><a name="p222mcpsimp"></a>产品名称</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.1.1 "><p id="p224mcpsimp"><a name="p224mcpsimp"></a><a name="p224mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row234mcpsimp"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.2.1"><p id="p236mcpsimp"><a name="p236mcpsimp"></a><a name="p236mcpsimp"></a>产品版本</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.2.1 "><p id="p240mcpsimp"><a name="p240mcpsimp"></a><a name="p240mcpsimp"></a>24.0.0</p>
</td>
</tr>
<tr id="row143412191233"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.3.1"><p id="p93610193311"><a name="p93610193311"></a><a name="p93610193311"></a>软件名称</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.3.1 "><p id="p93718191231"><a name="p93718191231"></a><a name="p93718191231"></a>MySQL可插拔在线向量化分析引擎</p>
</td>
</tr>
</tbody>
</table>


### 软件版本配套说明<a name="ZH-CN_TOPIC_0000002518703996"></a>

|软件类型|版本|备注|
|--|--|--|
|MySQL|8.0.25|本产品为动态库插件，需要由MySQL Server程序加载使用。|
|操作系统|openEuler 20.03 LTS SP1 for ARM<br>openEuler 22.03 LTS SP1 for ARM|-|



### 硬件版本配套说明<a name="ZH-CN_TOPIC_0000002518703994"></a>

当前仅支持鲲鹏920 ARM平台，不支持x86及鲲鹏非ARM平台。

推荐配置例如下表所示。

|类别|属性|详细说明|
|--|--|--|
|CPU|型号|2*鲲鹏920 7260处理器|
|CPU|核数|2*64|
|CPU|兆赫|2600|
|内存|容量|32GB * 16 = 512GB|
|磁盘|容量|40GB（操作系统）+1TB（数据盘）|



### 病毒扫描结果<a name="ZH-CN_TOPIC_0000002550143829"></a>

本软件包及相关文档经过OfficeScan、Bitdefender和Kaspersky防病毒软件扫描，没有发现病毒。详细信息如下：

<a name="table299mcpsimp"></a>
<table><tbody><tr id="row304mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p306mcpsimp"><a name="p306mcpsimp"></a><a name="p306mcpsimp"></a>Engine Name</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p8542230195516"><a name="p8542230195516"></a><a name="p8542230195516"></a>OfficeScan</p>
</td>
</tr>
<tr id="row310mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p312mcpsimp"><a name="p312mcpsimp"></a><a name="p312mcpsimp"></a>Engine Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p1154363055519"><a name="p1154363055519"></a><a name="p1154363055519"></a>12.000.1008</p>
</td>
</tr>
<tr id="row316mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p318mcpsimp"><a name="p318mcpsimp"></a><a name="p318mcpsimp"></a>Virus Lib Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p154323014556"><a name="p154323014556"></a><a name="p154323014556"></a>1935160</p>
</td>
</tr>
<tr id="row322mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.4.1"><p id="p324mcpsimp"><a name="p324mcpsimp"></a><a name="p324mcpsimp"></a>Scan Time</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.4.1 "><p id="p13543203045518"><a name="p13543203045518"></a><a name="p13543203045518"></a>2024-06-22 10:50:26</p>
</td>
</tr>
<tr id="row328mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.5.1"><p id="p330mcpsimp"><a name="p330mcpsimp"></a><a name="p330mcpsimp"></a>Scan Result</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.5.1 "><p id="p65431030105513"><a name="p65431030105513"></a><a name="p65431030105513"></a>OK</p>
</td>
</tr>
</tbody>
</table>

<a name="table334mcpsimp"></a>
<table><tbody><tr id="row339mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p341mcpsimp"><a name="p341mcpsimp"></a><a name="p341mcpsimp"></a>Engine Name</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p1594333725519"><a name="p1594333725519"></a><a name="p1594333725519"></a>Bitdefender</p>
</td>
</tr>
<tr id="row345mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p347mcpsimp"><a name="p347mcpsimp"></a><a name="p347mcpsimp"></a>Engine Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p29433374553"><a name="p29433374553"></a><a name="p29433374553"></a>7.0.3.2050</p>
</td>
</tr>
<tr id="row351mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p353mcpsimp"><a name="p353mcpsimp"></a><a name="p353mcpsimp"></a>Virus Lib Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p8944123718559"><a name="p8944123718559"></a><a name="p8944123718559"></a>7.96792</p>
</td>
</tr>
<tr id="row357mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.4.1"><p id="p359mcpsimp"><a name="p359mcpsimp"></a><a name="p359mcpsimp"></a>Scan Time</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.4.1 "><p id="p1094423715558"><a name="p1094423715558"></a><a name="p1094423715558"></a>2024-06-22 10:50:30</p>
</td>
</tr>
<tr id="row363mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.5.1"><p id="p365mcpsimp"><a name="p365mcpsimp"></a><a name="p365mcpsimp"></a>Scan Result</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.5.1 "><p id="p494473735517"><a name="p494473735517"></a><a name="p494473735517"></a>OK</p>
</td>
</tr>
</tbody>
</table>

<a name="table369mcpsimp"></a>
<table><tbody><tr id="row374mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p376mcpsimp"><a name="p376mcpsimp"></a><a name="p376mcpsimp"></a>Engine Name</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p074114812559"><a name="p074114812559"></a><a name="p074114812559"></a>Kaspersky</p>
</td>
</tr>
<tr id="row380mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p382mcpsimp"><a name="p382mcpsimp"></a><a name="p382mcpsimp"></a>Engine Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p117413483552"><a name="p117413483552"></a><a name="p117413483552"></a>11.2.0.4528</p>
</td>
</tr>
<tr id="row386mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p388mcpsimp"><a name="p388mcpsimp"></a><a name="p388mcpsimp"></a>Virus Lib Version</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p2074134817556"><a name="p2074134817556"></a><a name="p2074134817556"></a>2024-06-22 00:38:00</p>
</td>
</tr>
<tr id="row392mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.4.1"><p id="p394mcpsimp"><a name="p394mcpsimp"></a><a name="p394mcpsimp"></a>Scan Time</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.4.1 "><p id="p1674124825516"><a name="p1674124825516"></a><a name="p1674124825516"></a>2024-06-22 10:50:24</p>
</td>
</tr>
<tr id="row398mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.5.1"><p id="p400mcpsimp"><a name="p400mcpsimp"></a><a name="p400mcpsimp"></a>Scan Result</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.5.1 "><p id="p574204845516"><a name="p574204845516"></a><a name="p574204845516"></a>OK</p>
</td>
</tr>
</tbody>
</table>



## V1.0.0<a name="ZH-CN_TOPIC_0000002518544098"></a>

### 更新说明<a name="ZH-CN_TOPIC_0000002518544094"></a>

本次发布为第一次正式发布。

为提升OLAP（Online Analytical Processing）场景的性能，MySQL提供了第二执行引擎（Secondary Engine）预留接口。MySQL可插拔在线向量化分析引擎（以下简称KOVAE）是Secondary Engine的一种轻量实现，使用形式和MySQL标准插件的使用形式一致。

KOVAE以插件形式提供第二执行引擎特性的优点如下：

- 不侵入修改MySQL源码。
- 支持热安装或热卸载，不需要重启服务或者中断在线业务。
- 将符合条件的SQL语句卸载到第二引擎上执行，可以通过并行执行显著缩短SQL语句的执行时间。
- 落盘算子采用鲲鹏加速引擎（以下简称KAE）中的KAEzip来压缩落盘文件，节省磁盘空间。

采用可插拔在线向量化分析引擎提供的并行加速技术，可以将OLAP查询性能提升到3倍以上。

**相关概念<a name="section184141303214"></a>**

KAE（Kunpeng Accelerator Engine）是基于鲲鹏920处理器提供的硬件加速解决方案，包含了KAE加解密和KAEzip。KAEzip是KAE的压缩模块，使用鲲鹏硬加速模块实现deflate算法，结合无损用户态驱动框架，提供高性能Gzip/zlib格式压缩接口。关于KAEzip的更多详细信息，请参见《[鲲鹏加速引擎 用户指南](https://www.hikunpeng.com/document/detail/zh/kunpengaccel/kae/usermanual/kunpengaccel_16_0002.html)》。


### 已解决的问题<a name="ZH-CN_TOPIC_0000002550143835"></a>

无。


### 遗留问题<a name="ZH-CN_TOPIC_0000002518544092"></a>

无。



## 版本配套文档<a name="ZH-CN_TOPIC_0000002550143833"></a>

### V1.0.0版本配套文档<a name="ZH-CN_TOPIC_0000002518703992"></a>

|文档名称|内容简介|交付形式|
|--|--|--|
|《Kunpeng BoostKit 24.0.0 MySQL可插拔在线向量化分析引擎 版本说明书》|本文档提供MySQL可插拔在线向量化分析引擎版本发布及其配套信息。|开源仓|
|《Kunpeng BoostKit 24.0.0 MySQL可插拔在线向量化分析引擎 特性指南》|本文档提供MySQL可插拔在线向量化分析引擎特性使用指导。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002550183835"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/mysql)浏览和获取相关文档。



## 缩略语<a name="ZH-CN_TOPIC_0000002550143831"></a>

|**缩略语**|英文全称|中文全称|
|--|--|--|
|KAE|Kunpeng Accelerator Engine|鲲鹏加速引擎|
|KOVAE|Kunpeng Online Vectorized Analysis Engine|可插拔在线向量化分析引擎|
|OLAP|Online Analytical Processing|在线分析处理|



## 漏洞修补列表<a name="ZH-CN_TOPIC_0000002518544096"></a>

无。


