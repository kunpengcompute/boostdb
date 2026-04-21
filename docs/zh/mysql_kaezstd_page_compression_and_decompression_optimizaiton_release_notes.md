# MySQL KAEzstd页压缩解压缩优化 版本说明书

## 版本配套说明<a name="ZH-CN_TOPIC_0000002055262804"></a>

### 产品版本信息<a name="ZH-CN_TOPIC_0000002055262808"></a>

<a name="table215mcpsimp"></a>
<table><tbody><tr id="row220mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p222mcpsimp"><a name="p222mcpsimp"></a><a name="p222mcpsimp"></a>产品名称</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p226mcpsimp"><a name="p226mcpsimp"></a><a name="p226mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row234mcpsimp"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p236mcpsimp"><a name="p236mcpsimp"></a><a name="p236mcpsimp"></a>产品版本</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p240mcpsimp"><a name="p240mcpsimp"></a><a name="p240mcpsimp"></a>24.0.0</p>
</td>
</tr>
<tr id="row1482115559559"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p166661233362"><a name="p166661233362"></a><a name="p166661233362"></a>软件名称</p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p20218194113219"><a name="p20218194113219"></a><a name="p20218194113219"></a>MySQL KAEZstd页压缩解压缩优化</p>
</td>
</tr>
</tbody>
</table>


### 软件版本配套说明<a name="ZH-CN_TOPIC_0000002055421116"></a>

|软件类型|版本|
|--|--|
|操作系统|openEuler 22.03 LTS SP4 for Arm|
|MySQL|8.0.25|
|KAE|KAE2.0|



### 硬件版本配套说明<a name="ZH-CN_TOPIC_0000002091420049"></a>

|项目|要求|
|--|--|
|服务器|鲲鹏服务器|
|CPU|鲲鹏920新型号处理器|
|硬盘|进行性能测试时，数据目录需使用单独硬盘，即一个系统盘，一个数据盘（数据盘可选用性能较好的SSD盘，NVMe盘等），至少两块硬盘。非性能测试时，直接在系统盘上建数据目录即可。具体硬盘数量根据实际需求配置。|



### 病毒扫描结果<a name="ZH-CN_TOPIC_0000002055262800"></a>

本软件包、版本文档、产品文档经过奇安信、Bitdefender、Kaspersky防病毒软件扫描，未发现病毒。详细信息如下：

<a name="table3642172024219"></a>
<table><tbody><tr id="row1164218206425"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p464262014429"><a name="p464262014429"></a><a name="p464262014429"></a><strong id="b5642142017429"><a name="b5642142017429"></a><a name="b5642142017429"></a>Engine Name</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p1618395718179"><a name="p1618395718179"></a><a name="p1618395718179"></a>奇安信</p>
</td>
</tr>
<tr id="row10642920114213"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p86421120114217"><a name="p86421120114217"></a><a name="p86421120114217"></a><strong id="b964216206429"><a name="b964216206429"></a><a name="b964216206429"></a>Engine Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p17183165720177"><a name="p17183165720177"></a><a name="p17183165720177"></a>8.0.5.5260</p>
</td>
</tr>
<tr id="row364362018424"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p2643112034218"><a name="p2643112034218"></a><a name="p2643112034218"></a><strong id="b8643202015422"><a name="b8643202015422"></a><a name="b8643202015422"></a>Virus Lib Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p1018355716171"><a name="p1018355716171"></a><a name="p1018355716171"></a>2024-11-03 08:00:00.0</p>
</td>
</tr>
<tr id="row1864362014425"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.4.1"><p id="p176431207422"><a name="p176431207422"></a><a name="p176431207422"></a><strong id="b1464313204421"><a name="b1464313204421"></a><a name="b1464313204421"></a>Scan Time</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.4.1 "><p id="p218375715175"><a name="p218375715175"></a><a name="p218375715175"></a>2024-11-04 19:13:18</p>
</td>
</tr>
<tr id="row1764342004212"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.5.1"><p id="p8643152012429"><a name="p8643152012429"></a><a name="p8643152012429"></a><strong id="b164315203424"><a name="b164315203424"></a><a name="b164315203424"></a>Scan Result</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.5.1 "><p id="p3183057111715"><a name="p3183057111715"></a><a name="p3183057111715"></a>OK</p>
</td>
</tr>
</tbody>
</table>

<a name="table6517328174210"></a>
<table><tbody><tr id="row85171128134216"><th class="firstcol" valign="top" width="27.96%" id="mcps1.1.3.1.1"><p id="p45171428154212"><a name="p45171428154212"></a><a name="p45171428154212"></a><strong id="b55171728104214"><a name="b55171728104214"></a><a name="b55171728104214"></a>Engine Name</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72.04%" headers="mcps1.1.3.1.1 "><p id="p103553471812"><a name="p103553471812"></a><a name="p103553471812"></a>Bitdefender</p>
</td>
</tr>
<tr id="row75177288422"><th class="firstcol" valign="top" width="27.96%" id="mcps1.1.3.2.1"><p id="p11517122810426"><a name="p11517122810426"></a><a name="p11517122810426"></a><strong id="b165172287421"><a name="b165172287421"></a><a name="b165172287421"></a>Engine Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72.04%" headers="mcps1.1.3.2.1 "><p id="p435517413185"><a name="p435517413185"></a><a name="p435517413185"></a>7.0.3.2050</p>
</td>
</tr>
<tr id="row115171528104219"><th class="firstcol" valign="top" width="27.96%" id="mcps1.1.3.3.1"><p id="p1751712811421"><a name="p1751712811421"></a><a name="p1751712811421"></a><strong id="b45173282426"><a name="b45173282426"></a><a name="b45173282426"></a>Virus Lib Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72.04%" headers="mcps1.1.3.3.1 "><p id="p6355194121811"><a name="p6355194121811"></a><a name="p6355194121811"></a>7.97800</p>
</td>
</tr>
<tr id="row12517122854211"><th class="firstcol" valign="top" width="27.96%" id="mcps1.1.3.4.1"><p id="p55174281428"><a name="p55174281428"></a><a name="p55174281428"></a><strong id="b11517112844217"><a name="b11517112844217"></a><a name="b11517112844217"></a>Scan Time</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72.04%" headers="mcps1.1.3.4.1 "><p id="p1435510461819"><a name="p1435510461819"></a><a name="p1435510461819"></a>2024-11-04 19:13:28</p>
</td>
</tr>
<tr id="row8517192834210"><th class="firstcol" valign="top" width="27.96%" id="mcps1.1.3.5.1"><p id="p19517132819426"><a name="p19517132819426"></a><a name="p19517132819426"></a><strong id="b751711289422"><a name="b751711289422"></a><a name="b751711289422"></a>Scan Result</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72.04%" headers="mcps1.1.3.5.1 "><p id="p18355114181815"><a name="p18355114181815"></a><a name="p18355114181815"></a>OK</p>
</td>
</tr>
</tbody>
</table>

<a name="table9969193324218"></a>
<table><tbody><tr id="row1796923311429"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.1.1"><p id="p8969333144219"><a name="p8969333144219"></a><a name="p8969333144219"></a><strong id="b12969633104216"><a name="b12969633104216"></a><a name="b12969633104216"></a>Engine Name</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.1.1 "><p id="p3471136188"><a name="p3471136188"></a><a name="p3471136188"></a>Kaspersky</p>
</td>
</tr>
<tr id="row18969633154220"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.2.1"><p id="p296914330428"><a name="p296914330428"></a><a name="p296914330428"></a><strong id="b1396910337428"><a name="b1396910337428"></a><a name="b1396910337428"></a>Engine Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.2.1 "><p id="p1647151351819"><a name="p1647151351819"></a><a name="p1647151351819"></a>12.0.0.6672</p>
</td>
</tr>
<tr id="row196973315423"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.3.1"><p id="p096919335425"><a name="p096919335425"></a><a name="p096919335425"></a><strong id="b13969123394215"><a name="b13969123394215"></a><a name="b13969123394215"></a>Virus Lib Version</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.3.1 "><p id="p848171391816"><a name="p848171391816"></a><a name="p848171391816"></a>2024-11-04 02:06:00</p>
</td>
</tr>
<tr id="row396910337420"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.4.1"><p id="p8969173314220"><a name="p8969173314220"></a><a name="p8969173314220"></a><strong id="b8969173324217"><a name="b8969173324217"></a><a name="b8969173324217"></a>Scan Time</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.4.1 "><p id="p248413201816"><a name="p248413201816"></a><a name="p248413201816"></a>2024-11-04 19:13:14</p>
</td>
</tr>
<tr id="row1897012335429"><th class="firstcol" valign="top" width="28.000000000000004%" id="mcps1.1.3.5.1"><p id="p10970183314215"><a name="p10970183314215"></a><a name="p10970183314215"></a><strong id="b10970143394215"><a name="b10970143394215"></a><a name="b10970143394215"></a>Scan Result</strong></p>
</th>
<td class="cellrowborder" valign="top" width="72%" headers="mcps1.1.3.5.1 "><p id="p1748121317188"><a name="p1748121317188"></a><a name="p1748121317188"></a>OK</p>
</td>
</tr>
</tbody>
</table>



## 版本说明<a name="ZH-CN_TOPIC_0000002055421112"></a>

### 更新说明<a name="ZH-CN_TOPIC_0000002102772993"></a>

KAEZstd是鲲鹏加速引擎KAE（Kunpeng Accelerator Engine）的压缩模块，使用鲲鹏硬加速模块实现lz77\_zstd算法，提供ZSTD库标准接口。通过加速引擎可以实现不同场景下应用性能的提升，压缩效率有显著提升。关于KAEZstd的更多详细信息，请参见《[<u>鲲鹏加速引擎 开发指南（KAEzip）</u>](https://support.huawei.com/enterprise/zh/doc/EDOC1100433052)》。

MySQL透明页压缩是MySQL InnoDB存储引擎提供的一种数据压缩技术，它能够在页面级别对数据进行压缩，从而节省磁盘空间。在MySQL KAEZstd页压缩解压缩优化特性中，MySQL透明页压缩利用鲲鹏加速引擎中的KAEZstd来压缩数据页，从而有效节省磁盘空间。以Sysbench测试场景下测试64张1000万行表为例，使用MySQL KAEZstd页压缩解压缩优化特性方案后，减少大约一半的磁盘占用空间，并且在高负载并发的情况下，TPS（Transactions Per Second）劣化程度不超过15%。


### 已解决的问题<a name="ZH-CN_TOPIC_0000002067172950"></a>

无。


### 遗留问题<a name="ZH-CN_TOPIC_0000002067013158"></a>

无。



## 版本配套文档<a name="ZH-CN_TOPIC_0000002091420053"></a>

### 版本配套文档<a name="ZH-CN_TOPIC_0000002091420057"></a>

|文档名称|内容简介|交付形式|
|--|--|--|
|鲲鹏BoostKit 24.0.0 MySQL KAEZstd页压缩解压缩优化 版本说明书|本文档提供MySQL KAEZstd页压缩解压缩优化特性的版本发布及其配套信息。|开源仓|
|鲲鹏BoostKit 24.0.0 MySQL KAEZstd页压缩解压缩优化 特性指南|本文档提供MySQL KAEZstd页压缩解压缩优化特性的使用指导。|开源仓|



### 获取文档的方法<a name="ZH-CN_TOPIC_0000002091501465"></a>

您可以通过访问[开源仓](https://gitcode.com/boostkit/mysql)浏览和获取相关文档。



## 漏洞修补列表<a name="ZH-CN_TOPIC_0000002055421108"></a>

无。


## 缩略语<a name="ZH-CN_TOPIC_0000002091501469"></a>

|**缩略语**|**英文全称**|**中文全称**|
|--|--|--|
|KAE|Kunpeng Acceleration Engine|鲲鹏加速引擎|
|ZSTD|Zstandard|高效无损压缩算法|
|TPS|Transactions Per Second|每秒处理的事务数|



