# MySQL Computing Path Optimization Release Notes

## Version Mapping<a name="EN-US_TOPIC_0000002521463536"></a>

### Product Version Information<a name="EN-US_TOPIC_0000002552623507"></a>

<a name="table62675726"></a>
<table><tbody><tr id="row41561572"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.1.1"><p id="p11044137"><a name="p11044137"></a><a name="p11044137"></a>Product Name</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.1.1 "><p id="p243mcpsimp"><a name="p243mcpsimp"></a><a name="p243mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row33192131"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.2.1"><p id="p4208106"><a name="p4208106"></a><a name="p4208106"></a>Product Version</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.2.1 "><p id="p248mcpsimp"><a name="p248mcpsimp"></a><a name="p248mcpsimp"></a>25.3.0</p>
</td>
</tr>
<tr id="row24726251"><th class="firstcol" valign="top" width="26.61%" id="mcps1.1.3.3.1"><p id="p56669300"><a name="p56669300"></a><a name="p56669300"></a>Feature Name</p>
</th>
<td class="cellrowborder" valign="top" width="73.39%" headers="mcps1.1.3.3.1 "><p id="p11923034"><a name="p11923034"></a><a name="p11923034"></a>MySQL computing path optimization</p>
</td>
</tr>
</tbody>
</table>


### Software Version Mapping<a name="EN-US_TOPIC_0000002552623509"></a>

|Type|Version|
|--|--|
|OS|openEuler 22.03 LTS SP4|
|Percona|Percona-Server 5.7.44-53 or Percona-Server 8.0.43-34|



### Hardware Version Mapping<a name="EN-US_TOPIC_0000002521623550"></a>

|Item|Requirement|
|--|--|
|Processor|New Kunpeng 920 processor model or Kunpeng 950 processor|



### Virus Scan Results<a name="EN-US_TOPIC_0000002552663491"></a>

Virus scanning is not involved because no software package is released.



## v25.3.0<a name="EN-US_TOPIC_0000002521623548"></a>

### Change Description<a name="EN-US_TOPIC_0000002552663493"></a>

In MySQL read-only online transaction processing (OLTP) scenarios, Kunpeng BoostKit provides a series of optimization mechanisms for hotspot functions, including those related to record matching and character set processing. In addition, Kunpeng 920 series processors now support unaligned memory access. Therefore, the unaligned memory access optimization policies implemented on the x86 architecture can be migrated to the Arm architecture for higher performance.


### Resolved Issues<a name="EN-US_TOPIC_0000002552623511"></a>

None


### Known Issues<a name="EN-US_TOPIC_0000002521623546"></a>

None



## Related Documentation<a name="EN-US_TOPIC_0000002521463534"></a>

### Documentation<a name="EN-US_TOPIC_0000002552663489"></a>

|Document|Description|Delivery Method|
|--|--|--|
|*Kunpeng BoostKit 25.3.0 MySQL Computing Path Optimization Release Notes*|Describes the version release and mapping information of the MySQL computing path optimization feature.|Open-source repository|
|*Kunpeng BoostKit 25.3.0 MySQL Computing Path Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL computing path optimization feature.|Open-source repository|



### Obtaining Documentation<a name="EN-US_TOPIC_0000002552663495"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/mysql) to view or download related documents.
