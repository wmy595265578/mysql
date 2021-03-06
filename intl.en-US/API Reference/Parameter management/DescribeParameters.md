# DescribeParameters {#doc_api_1104897 .reference}

You can call this operation to query the current parameter configurations of instances.

This operation is applicable to the following database engine versions:

-   MySQL 5.5, 5.6, 5.7, and 8.0
-   SQL Server 2008 R2
-   PostgreSQL 9.4 and 10.0
-   PPAS 9.3 and 10.0
-   MariaDB 10.3

## Debugging {#apiExplorer .section}

You can use [OpenAPI Explorer](https://api.aliyun.com/#product=Rds&api=DescribeParameters) to perform debugging. OpenAPI Explorer allows you to perform various operations to simplify API usage. For example, you can retrieve APIs, call APIs, and dynamically generate SDK example code.

## Request parameters {#parameters .section}

|Parameter|Type|Required?|Example|Description|
|---------|----|---------|-------|-----------|
|Action|String|Yes|DescribeParameters| The operation that you want to perform. Set this parameter to DescribeParameters.

 |
|DBInstanceId|String|Yes|rm-uf6wjk5xxxxxxx| The ID of the instance.

 |
|AccessKeyId|String|No|LTAIfCxxxxxxx| The AccessKey ID issued by Alibaba Cloud for users to access services.

 |
|ClientToken|String|No|ETnLKlblzczshOTUbOCzxxxxx| The client token that is used to guarantee the idempotency of requests. The client token is generated by the client and is unique among different requests. It is a string of up to 64 ASCII characters.

 |

## Response parameters {#resultMapping .section}

|Parameter|Type|Example|Description|
|---------|----|-------|-----------|
|Engine|String|MySQL| The database type.

 |
|Engineversion|String|5.5| The version of the database.

 |
|RunningParameters| | | The list of running parameters.

 |
|ParameterName|String|fill factor| The name of the parameter.

 |
|ParameterValue|String|0| The value of the parameter.

 |
|ParameterDescription|String|This option sets the default fill factor value at the server scope. A fill factor is provided to optimize index data storage and performance.| The description of the parameter.

 |
|ConfigParameters| | | The list of parameters being synchronized. After modifying and submitting parameters, you need to wait for the parameters to be synchronized with the instance. After synchronization, you can delete the parameters from the list.

 |
|ParameterName|String|fill factor| The name of the parameter.

 |
|ParameterValue|String|50| The value of the parameter.

 |
|ParameterDescription|String|This option sets the default fill factor value at the server scope. A fill factor is provided to optimize index data storage and performance.| The description of the parameter.

 |
|RequestId|String|1AD222E9-E606-4A42-BF6D-8A4442913CEF| The ID of the request.

 |

## Examples {#demo .section}

Sample requests

``` {#request_demo}

http(s)://rds.aliyuncs.com/? Action=DescribeParameters
&DBInstanceId=rm-uf6wjk5xxxxxxx
&<Common request parameters>

```

Successful response examples

`XML` format

``` {#codeblock_oh4_26f_thk}
<DescribeParametersResponse>
	  <ConfigParameters>
		    <DBInstanceParameter>
			      <ParameterDescription> This option sets the default fill factor value
at the server scope. A fill factor is provided to optimize index data storage
and performance. </ParameterDescription>
			      <ParameterName>fill factor</ParameterName>
			      <ParameterValue>50</ParameterValue>
		    </DBInstanceParameter>
	  </ConfigParameters>
	  <Engine>mssql</Engine>
	  <EngineVersion>2008r2</EngineVersion>
	  <RunningParameters>
		    <DBInstanceParameter>
			      <ParameterDescription> This option sets the default fill factor value
at the server scope. A fill factor is provided to optimize index data storage
and performance. </ParameterDescription>
			      <ParameterName>fill factor</ParameterName>
			      <ParameterValue>0</ParameterValue>
		    </DBInstanceParameter>
	  </RunningParameters>
	  <RequestId>2A748162-8040-4D6B-813E-6910C8C033F1</RequestId></DescribeParametersResponse>
```

`JSON` format

``` {#codeblock_jtt_fin_hsl}
{
	"ConfigParameters":{
		"DBInstanceParameter":[
			{
				"ParameterDescription":"This option sets the default fill factor value
at the server scope. A fill factor is provided to optimize index data storage
and performance.",
				"ParameterValue":"50",
				"ParameterName":"fill factor"
			}
		]
	},
	"RequestId":"2A748162-8040-4D6B-813E-6910C8C033F1",
	"RunningParameters":{
		"DBInstanceParameter":[
			{
				"ParameterDescription":"This option sets the default fill factor value
at the server scope. A fill factor is provided to optimize index data storage
and performance.",
				"ParameterValue":"0",
				"ParameterName":"fill factor"
			}
		]
	},
	"EngineVersion":"2008r2",
	"Engine":"mssql"
}
```

## Error codes { .section}

For a list of error codes, visit the [API Error Center](https://error-center.alibabacloud.com/status/product/Rds).

