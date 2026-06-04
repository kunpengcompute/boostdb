# MySQL Pluggable Online Vectorized Analysis Engine Release Notes

## Version Mapping<a name="EN-US_TOPIC_0000002550183837"></a>

### Product Version Information<a name="EN-US_TOPIC_0000002518703998"></a>

<a name="table215mcpsimp"></a>
<table><tbody><tr id="row220mcpsimp"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.1.1"><p id="p222mcpsimp"><a name="p222mcpsimp"></a><a name="p222mcpsimp"></a>Product Name</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.1.1 "><p id="p224mcpsimp"><a name="p224mcpsimp"></a><a name="p224mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row234mcpsimp"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.2.1"><p id="p236mcpsimp"><a name="p236mcpsimp"></a><a name="p236mcpsimp"></a>Product Version</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.2.1 "><p id="p240mcpsimp"><a name="p240mcpsimp"></a><a name="p240mcpsimp"></a>24.0.0</p>
</td>
</tr>
<tr id="row143412191233"><th class="firstcol" valign="top" width="30%" id="mcps1.1.3.3.1"><p id="p93610193311"><a name="p93610193311"></a><a name="p93610193311"></a>Software Name</p>
</th>
<td class="cellrowborder" valign="top" width="70%" headers="mcps1.1.3.3.1 "><p id="p93718191231"><a name="p93718191231"></a><a name="p93718191231"></a>MySQL pluggable online vectorized analysis engine</p>
</td>
</tr>
</tbody>
</table>


### Software Version Mapping<a name="EN-US_TOPIC_0000002518703996"></a>

|Type|Version|Description|
|--|--|--|
|MySQL|8.0.25|This product is a dynamic library plugin and needs to be loaded by the MySQL server.|
|OS|openEuler 20.03 LTS SP1 for Arm<br>openEuler 22.03 LTS SP1 for Arm|-|



### Hardware Version Mapping<a name="EN-US_TOPIC_0000002518703994"></a>

The current product version supports the Kunpeng 920 Arm platform only, and does not support x86 or non-Arm Kunpeng platforms.

The following table lists the recommended configurations.

|Type|Attribute|Description|
|--|--|--|
|CPU|Model|2 × Kunpeng 920 processor|
|CPU|Number of cores|2 × 64|
|CPU|MHz|2600|
|Memory|Capacity|16 × 32 GB = 512 GB|
|Drive|Capacity|40 GB (for the OS) + 1 TB (for data)|



### Virus Scan Results<a name="EN-US_TOPIC_0000002550143829"></a>

The software packages and related documents have been scanned by antivirus software OfficeScan, Bitdefender, and Kaspersky, and no risks have been found. The virus scan results are as follows.

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



## V1.0.0<a name="EN-US_TOPIC_0000002518544098"></a>

### Change Description<a name="EN-US_TOPIC_0000002518544094"></a>

This issue is the first official release.

To improve the performance of online analytical processing (OLAP), MySQL provides a reserved interface for the secondary engine. MySQL Pluggable Kunpeng Online Vectorized Analysis Engine (KOVAE) is a lightweight implementation of the secondary engine. KOVAE presents itself as a MySQL plugin.

Providing features of the secondary engine, KOVAE as a plugin has the following advantages:

- Free of intrusive modification on the MySQL source code
- Hot-installable or hot-uninstallable without restarting or interrupting services
- Offloading suitable SQL statements to the secondary engine for execution, significantly shortening the execution time of the SQL statements through parallel execution
- KAEzip in Kunpeng Accelerator Engine (KAE) is utilized by drive flushing operators to compress flushed files, saving drive space

The parallel acceleration technology provided by KOVAE improves the query performance of OLAP by over three times.

**Related Concepts<a name="section184141303214"></a>**

KAE is a hardware acceleration solution based on the Kunpeng 920 processor. It includes KAE encryption and decryption as well as KAEzip. KAEzip is the compression module of KAE. It implements the deflate algorithm and works with the lossless user-space driver framework to provide high-performance gzip or zlib interfaces. For more information about KAEzip, see [Kunpeng Accelerator Engine User Guide](https://www.hikunpeng.com/document/detail/en/kunpengaccel/kae/usermanual/kunpengaccel_16_0002.html).


### Resolved Issues<a name="EN-US_TOPIC_0000002550143835"></a>

None


### Known Issues<a name="EN-US_TOPIC_0000002518544092"></a>

None



## Related Documentation<a name="EN-US_TOPIC_0000002550143833"></a>

### V1.0.0 Documentation<a name="EN-US_TOPIC_0000002518703992"></a>

|Document|Description|Delivery Method|
|--|--|--|
|*Kunpeng BoostKit 24.0.0 MySQL Pluggable Online Vectorized Analysis Engine Release Notes*|This document provides the version release and mapping information of KOVAE that is pluggable on MySQL.|Open-source repository|
|*Kunpeng BoostKit 24.0.0 MySQL Pluggable Online Vectorized Analysis Engine Feature Guide*|This document provides guidance for using the features of KOVAE that is pluggable on MySQL.|Open-source repository|



### Obtaining Documentation<a name="EN-US_TOPIC_0000002550183835"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/mysql) to view or download related documents.



## Acronyms and Abbreviations<a name="EN-US_TOPIC_0000002550143831"></a>

|**Acronym/Abbreviation**|**Full Spelling**|
|--|--|
|KAE|Kunpeng Accelerator Engine|
|KOVAE|Kunpeng Online Vectorized Analysis Engine|
|OLAP|online analytical processing|



## List of Fixed Vulnerabilities<a name="EN-US_TOPIC_0000002518544096"></a>

None
