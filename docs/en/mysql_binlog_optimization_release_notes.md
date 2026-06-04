# MySQL Binlog Optimization Release Notes

## Version Mapping<a name="EN-US_TOPIC_0000002552623583"></a>

### Product Version Information<a name="EN-US_TOPIC_0000002521463606"></a>

<a name="table62675726"></a>
<table><tbody><tr id="row41561572"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.1.1"><p id="p11044137"><a name="p11044137"></a><a name="p11044137"></a>Product Name</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.1.1 "><p id="p243mcpsimp"><a name="p243mcpsimp"></a><a name="p243mcpsimp"></a>Kunpeng BoostKit</p>
</td>
</tr>
<tr id="row33192131"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.2.1"><p id="p4208106"><a name="p4208106"></a><a name="p4208106"></a>Product Version</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.2.1 "><p id="p248mcpsimp"><a name="p248mcpsimp"></a><a name="p248mcpsimp"></a>25.3.0</p>
</td>
</tr>
<tr id="row24726251"><th class="firstcol" valign="top" width="26.26%" id="mcps1.1.3.3.1"><p id="p56669300"><a name="p56669300"></a><a name="p56669300"></a>Feature Name</p>
</th>
<td class="cellrowborder" valign="top" width="73.74000000000001%" headers="mcps1.1.3.3.1 "><p id="p11923034"><a name="p11923034"></a><a name="p11923034"></a>MySQL binlog optimization</p>
</td>
</tr>
</tbody>
</table>


### Software Version Mapping<a name="EN-US_TOPIC_0000002521623612"></a>

|Type|Version|
|--|--|
|OS|openEuler 22.03 LTS SP4|
|Percona|Percona-Server 5.7.44-53<br>Percona-Server 8.0.43-34|



### Hardware Version Mapping<a name="EN-US_TOPIC_0000002552663559"></a>

|Item|Requirement|
|--|--|
|Processor|New Kunpeng 920 processor model or Kunpeng 950 processor|



### Virus Scan Results<a name="EN-US_TOPIC_0000002521463602"></a>

Virus scanning is not involved because no software package is released.



## v25.3.0<a name="EN-US_TOPIC_0000002552663563"></a>

### Change Description<a name="EN-US_TOPIC_0000002521623616"></a>

MySQL's binary log (binlog) records all database data changes (such as INSERT, UPDATE, and DELETE), and is primarily used for data replication and restoration. It has a significant impact on performance in high-concurrency write scenarios.

The binlog optimization feature identifies hotspot functions that affect performance in write-only scenarios and optimizes these functions through binlog pre-allocation, lock splitting, and writeset_history data structure replacement. This reduces binlog write overhead and improves performance in write-only scenarios.


### Resolved Issues<a name="EN-US_TOPIC_0000002552623579"></a>

None


### Known Issues<a name="EN-US_TOPIC_0000002552623577"></a>

None



## Related Documentation<a name="EN-US_TOPIC_0000002521463604"></a>

### Documentation<a name="EN-US_TOPIC_0000002552623581"></a>

|Document|Description|Delivery Method|
|--|--|--|
|*Kunpeng BoostKit 25.3.0 MySQL Binlog Optimization Release Notes*|Provides the version release and mapping information of the MySQL binlog optimization feature.|[Open-source repository](https://gitcode.com/boostkit/mysql/tree/master/docs/en)|
|*Kunpeng BoostKit 25.3.0 MySQL Binlog Optimization Feature Guide*|Describes the environment requirements and provides guidance on enabling the MySQL binlog optimization feature.|[Open-source repository](https://gitcode.com/boostkit/mysql/tree/master/docs/en)|



### Obtaining Documentation<a name="EN-US_TOPIC_0000002521623614"></a>

Visit the [open-source repository](https://gitcode.com/boostkit/mysql/tree/master/docs/en) to view or download required documents.
