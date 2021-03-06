# CalculateDBInstanceWeight {#doc_api_Rds_CalculateDBInstanceWeight .reference}

调用CalculateDBInstanceWeight接口查询系统权重分配值。

在开启[读写分离](~~51073~~)的情况下，该接口用于计算系统指定的权重。如果是自定义读权重，请参见[DescribeDBInstanceNetInfo](~~26237~~)。

仅适用于如下实例：

-   MySQL 5.7高可用版（本地SSD盘）
-   MySQL 5.6
-   SQL Server 2017集群版

## 调试 {#api_explorer .section}

[您可以在OpenAPI Explorer中直接运行该接口，免去您计算签名的困扰。运行成功后，OpenAPI Explorer可以自动生成SDK代码示例。](https://api.aliyun.com/#product=Rds&api=CalculateDBInstanceWeight&type=RPC&version=2014-08-15)

## 请求参数 {#parameters .section}

|名称|类型|是否必选|示例值|描述|
|--|--|----|---|--|
|Action|String|是|CalculateDBInstanceWeight|系统规定参数，取值：**CalculateDBInstanceWeight**。

 |
|DBInstanceId|String|是|rm-uf6wjk5xxxxxxx|主实例ID。

 |

## 返回数据 {#resultMapping .section}

|名称|类型|示例值|描述|
|--|--|---|--|
|Items| | |系统指定权重列表。

 |
|DBInstanceId|String|rm-uf6wjk5xxxxxxx|实例ID。

 |
|DBInstanceType|String|Master|实例类型，取值：

 -   **Master**：主实例；
-   **Readonly**：只读实例。

 |
|Weight|String|100|系统实时计算的实例权重。

 |
|ReadonlyInstanceSQLDelayedTime|String|30|只读实例延迟复制时间，只读实例延迟**ReadonlyInstanceSQLDelayedTime**的时间后再同步主实例数据，单位：秒。

 |
|RequestId|String|C816A4BF-A6EC-4722-95F9-2055859CCFD2|请求ID。

 |

## 示例 {#demo .section}

请求示例

``` {#request_demo}

http(s)://rds.aliyuncs.com/?Action=CalculateDBInstanceWeight
&DBInstanceId=rm-uf6wjk5xxxxxxx
&<公共请求参数>

```

正常返回示例

`XML` 格式

``` {#xml_return_success_demo}
<CalculateDBInstanceWeightResponse>
	  <items>
		    <DBInstanceId>rm-uf6wjk5xxxxxxx</DBInstanceId>
		    <DBInstanceType>Master</DBInstanceType>
		    <Weight></Weight>
		    <Availability></Availability>
	  </items>
	  <requestId>C816A4BF-A6EC-4722-95F9-2055859CCFD2</requestId></CalculateDBInstanceWeightResponse>
```

`JSON` 格式

``` {#json_return_success_demo}
{
	"requestId":"C816A4BF-A6EC-4722-95F9-2055859CCFD2",
	"items":[
		{
			"Weight":"",
			"Availability":"",
			"DBInstanceType":"Master",
			"DBInstanceId":"rm-uf6wjk5xxxxxxx"
		}
	]
}
```

## 错误码 { .section}

访问[错误中心](https://error-center.aliyun.com/status/product/Rds)查看更多错误码。

