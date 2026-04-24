# MySQL Binlog优化 版本说明书

## 版本配套说明<a name="ZH-CN_TOPIC_0000002552623583"></a>

### 产品版本信息<a name="ZH-CN_TOPIC_0000002521463606"></a>

<a name="table62675726"></a>
<table><tbody><tr id="row41561572"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.1.1"><p id="p11044137"><a name="p11044137"></a><a name="p11044137"></a>产品名称</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.1.1 "><p id="p243mcpsimp"><a name="p243mcpsimp"></a><a name="p243mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row33192131"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.2.1"><p id="p4208106"><a name="p4208106"></a><a name="p4208106"></a>产品版本</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.2.1 "><p id="p248mcpsimp"><a name="p248mcpsimp"></a><a name="p248mcpsimp"></a>25.3.0</p>
</td>
</tr>
<tr id="row24726251"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.3.1"><p id="p56669300"><a name="p56669300"></a><a name="p56669300"></a>特性名称</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.3.1 "><p id="p11923034"><a name="p11923034"></a><a name="p11923034"></a>MySQL Binlog优化</p>
</td>
</tr>
</tbody>
</table>


### 软件版本配套说明<a name="ZH-CN_TOPIC_0000002521623612"></a>

|软件类型|版本|
|--|--|
|OS|openEuler 22.03 LTS SP4|
|Percona|Percona-Server 5.7.44-53<br>Percona-Server 8.0.43-34|



### 硬件版本配套说明<a name="ZH-CN_TOPIC_0000002552663559"></a>

|项目|要求|
|--|--|
|处理器|鲲鹏920新型号处理器、鲲鹏950处理器|



### 病毒扫描结果<a name="ZH-CN_TOPIC_0000002521463602"></a>

不涉及软件包发布，不涉及病毒扫描。



## v25.3.0<a name="ZH-CN_TOPIC_0000002552663563"></a>

### 更新说明<a name="ZH-CN_TOPIC_0000002521623616"></a>

MySQL的二进制日志（Binlog）是记录所有数据库数据变更操作（如 INSERT、UPDATE、DELETE）的日志文件，主要用于数据复制和恢复，对高并发写入场景性能影响显著。

Binlog优化特性通过识别只写场景下影响性能的多个函数，针对这些热点函数进行多个方面的优化，包括Binlog预分配、Binlog拆锁优化、Binlog writeset\_history数据结构的替换。从而减少写入Binlog日志的开销，实现只写场景性能的提升。


### 已解决的问题<a name="ZH-CN_TOPIC_0000002552623579"></a>

无


### 遗留问题<a name="ZH-CN_TOPIC_0000002552623577"></a>

无



## 版本配套文档<a name="ZH-CN_TOPIC_0000002521463604"></a>

### 版本配套文档<a name="ZH-CN_TOPIC_0000002552623581"></a>

|文档名称|内容简介|交付形式|
|--|--|--|
|《Kunpeng BoostKit 25.3.0 MySQL Binlog优化 版本说明书》|本文档提供MySQL Binlog优化特性的版本发布及其配套信息。|[开源仓](https://gitcode.com/boostkit/mysql/tree/master/docs/zh)|
|《Kunpeng BoostKit 25.3.0 MySQL Binlog优化 特性指南》|本文档提供MySQL Binlog优化特性的环境要求、特性使能指导。|[开源仓](https://gitcode.com/boostkit/mysql/tree/master/docs/zh)|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002521623614"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/mysql/tree/master/docs/zh)浏览和获取相关文档。


