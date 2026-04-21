# MySQL计算路径优化 版本说明书

## 版本配套说明<a name="ZH-CN_TOPIC_0000002521463536"></a>

### 产品版本信息<a name="ZH-CN_TOPIC_0000002552623507"></a>

<a name="table62675726"></a>
<table><tbody><tr id="row41561572"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.1.1"><p id="p11044137"><a name="p11044137"></a><a name="p11044137"></a>产品名称</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.1.1 "><p id="p243mcpsimp"><a name="p243mcpsimp"></a><a name="p243mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row33192131"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.2.1"><p id="p4208106"><a name="p4208106"></a><a name="p4208106"></a>产品版本</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.2.1 "><p id="p248mcpsimp"><a name="p248mcpsimp"></a><a name="p248mcpsimp"></a>25.3.0</p>
</td>
</tr>
<tr id="row24726251"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.3.1"><p id="p56669300"><a name="p56669300"></a><a name="p56669300"></a>特性名称</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.3.1 "><p id="p11923034"><a name="p11923034"></a><a name="p11923034"></a>MySQL计算路径优化</p>
</td>
</tr>
</tbody>
</table>


### 软件版本配套说明<a name="ZH-CN_TOPIC_0000002552623509"></a>

|软件类型|版本|
|--|--|
|OS|openEuler 22.03 LTS SP4|
|Percona|Percona-Server 5.7.44-53、Percona-Server 8.0.43-34|



### 硬件版本配套说明<a name="ZH-CN_TOPIC_0000002521623550"></a>

|项目|要求|
|--|--|
|处理器|鲲鹏920新型号处理器、鲲鹏950处理器|



### 病毒扫描结果<a name="ZH-CN_TOPIC_0000002552663491"></a>

不涉及软件包发布，不涉及病毒扫描。



## v25.3.0<a name="ZH-CN_TOPIC_0000002521623548"></a>

### 更新说明<a name="ZH-CN_TOPIC_0000002552663493"></a>

在MySQL OLTP的只读场景下，针对主要热点函数即记录匹配相关函数以及字符集处理相关函数，鲲鹏BoostKit提供了一系列针对性优化机制。此外，鉴于鲲鹏920系列处理器已支持非对齐内存访问，原在x86架构中实现的非对齐内存访问优化策略也可迁移至ARM架构，以进一步增强性能。


### 已解决的问题<a name="ZH-CN_TOPIC_0000002552623511"></a>

无


### 遗留问题<a name="ZH-CN_TOPIC_0000002521623546"></a>

无



## 版本配套文档<a name="ZH-CN_TOPIC_0000002521463534"></a>

### 版本配套文档<a name="ZH-CN_TOPIC_0000002552663489"></a>

|文档名称|内容简介|交付形式|
|--|--|--|
|《Kunpeng BoostKit 25.3.0 MySQL计算路径优化 版本说明书》|本文档提供MySQL计算路径优化特性的版本发布及其配套信息。|开源仓|
|《Kunpeng BoostKit 25.3.0 MySQL计算路径优化 特性指南》|本文档提供MySQL计算路径优化特性的环境要求、特性使能指导。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002552663495"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/mysql)浏览和获取相关文档。



