# 版本说明书
### 版本配套说明
#### 产品版本信息
| 产品名称      | 产品版本     |
| --------- | -------- |
| Boostdb | 26.0.RC1 |
#### 软件版本配套说明
|特性名称|软件类型|版本|
|--|--|--|
|MySQL LSE及rec_get_offsets优化|OS|openEuler 22.03 LTS SP4、openEuler 24.03 LTS SP3|
|MySQL LSE及rec_get_offsets优化|Percona|Percona-Server 5.7.44-53、Percona-Server 8.0.43-34|                                                                             |
#### 硬件版本配套说明
| 特性名称          | 硬件项目          | 要求说明                                      |
| ------------- | ------------- | ----------------------------------------- |
| MySQL LSE及rec_get_offsets优化      | 处理器           | - 鲲鹏920系列处理器<br>- 鲲鹏950处理器                    |

#### 病毒扫描结果
不涉及软件包发布，不涉及病毒扫描。
### 版本使用注意事项
无
### 版本说明
#### 更新说明
##### MySQL LSE及rec_get_offsets优化

**新增功能：**

新增MySQL LSE及rec\_get\_offsets优化特性，通过引入LSE硬件加速和优化rec\_get\_offsets的调用逻辑，来有效缓解高并发下的性能劣化问题，提升MySQL在鲲鹏服务器中的整体性能表现与系统稳定性。

#### 已解决的问题
无
#### 遗留问题
无
### 版本配套文档

|文档名称|内容简介|交付形式|
|--|--|--|
|《MySQL LSE优化 特性指南》|本文档提供MySQL LSE优化特性的环境要求、特性使能指导。|开源仓|
|《MySQL rec_get_offsets优化 特性指南》|本文档提供MySQL rec_get_offsets优化特性的环境要求、特性使能指导。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002544372643"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/boostdb/tree/master/docs/zh)浏览和获取相关文档。
