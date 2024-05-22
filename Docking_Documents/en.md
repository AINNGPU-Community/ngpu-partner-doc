
API Integration for Compute Power Demand Side
=======
* When using NGPU for API integration, the compute power demand side needs to provide their workspace (a Docker image encapsulating their own AI capabilities). NGPU will deploy this Docker image across multiple nodes and make intelligent node selections. Since NGPU employs a parameter pass-through method, the following steps can be used during API integration.
-----------

# Integration Steps

+ The compute power demand side provides their runnable Docker image, including the Docker image's startup commands.
+ The compute power demand side provides the type and number of GPUs required.
+ The compute power demand side provides the Docker-supported APIs (NGPU will use a pass-through mode for API parameter data transmission).

# Example of API Integration

## Synchronous Mode

### Task Request

#### Original API Interface

* Protocol Type: POST
* Protocol Address: https://***/img2img
* Request Body
```shell
{
    "image_url": "https://ngpu.ai/image/1683730315image.jpeg",
    "prompt": "manicured claws, score_9, score_8_up, score_7_up, score_6_up, score_5_up, score_4_up, 1girl, realistic, extremely detailed, high quality, , beautiful eyes, cat woman, black suit, character concept,night time, dark shadows, dutch angle"
}
```
* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "image_url": "https://ngpu.ai/image/16837303689.jpeg"
}
```

#### Post-Integration API Interface
* Protocol Type: POST
* Protocol Address: https://ngpu.ai/user/img2img
* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
data|string|Data content for pass-through (can retain original API request body)


* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content
data|string|Task result (using pass-through, includes original API response body)


## Asynchronous Mode
-----------
* Differences between synchronous and asynchronous modes lie in that the asynchronous mode provides a taskID upon submitting the task request. The taskID is then periodically checked to see if the task has been completed.


### Task Request

####  Original API Interface

* Protocol Type: POST
* Protocol Address: https://***/sdapi/v1/img2img
* Request Body

```shell
{
    "image_url": "https://ngpu.ai/image/1683730315image.jpeg",
    "prompt": "manicured claws, score_9, score_8_up, score_7_up, score_6_up, score_5_up, score_4_up, 1girl, realistic, extremely detailed, high quality, , beautiful eyes, cat woman, black suit, character concept,night time, dark shadows, dutch angle"
}
```

* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "taskID": "20230423_11_23_05_10279"
}
```

#### Post-Integration API Interface
* Protocol Type: POST
* Protocol Address：https://ngpu.ai/user/schedulingTask?paramUrl=sdapi/v1/img2img&paramPort=8080
* Header：Authorization：Bearer Workspace ID

Parameter:
- paramUrl: URL path used when calling the compute node.
- paramPort: Port used when calling the compute node (the port mapped when the Docker image starts).

**_These parameters ensure that the apiServer can correctly transfer data to the compute node's Docker image._**

Field Name | Field Type | Field Meaning
----|:----:|:----:|
sdapi/v1/img2img|string|Use the img2img interface with stable Diffusion
sadTalker|string|Use the open source sadTalker interface
videoReTalker|string|Use the open source videoReTalker interface

* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
data|string|Data content for pass-through (can retain original API request body)

* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "taskID": "20230423_11_23_05_10279"
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content
data|string|Query return result
taskID|string|Returned task ID


### Task Query

####  Original API Interface

* Protocol Type: GET
* Protocol Address: https://***/img2img&taskID=

* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "taskID": "20230423_11_23_05_10279"
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content
taskID|string|Returned task ID


#### Post-Integration API Interface
* Protocol Type: GET
* Protocol Address: https://ngpu.ai/user/queryTask&taskID=20230423_11_23_05_10279

* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "taskID": "20230423_11_23_05_10279",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content
taskID|string|Returned task ID
data|string|Query return result


# Common API Interface

## BRC20 Inscription Detection Interface
* Function: Checks if a specified BRC20 inscription exists on a given BTC address.

* Protocol Type: GET
* Protocol Address: https://ngpu.ai/brc20/checkAddress&address=bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus

* Response Body
```shell
{
	"exists": true,
	"names": [
		"AINN"
	],
	"balances": [
		100
	]
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
exists|bool|Whether a specified BRC20 inscription exists on the user's BTC address
names|[]string|Names of the inscriptions on the user's BTC address
balances|[]int|Quantity of each inscription on the user's BTC address (corresponding to the inscription names)

## Interface to Query Task List by Workspace ID
* Purpose: Query the task list using the Workspace ID and optional start and end times.

* Protocol Type: GET
* URL：https://ngpu.ai/user/queryTaskList?startTime=2024-04-11&endTime=2024-04-20
* Header：Authorization：Bearer Workspace ID  

**_startTime is optional; if not provided, the default value is 1970-01-01. endTime is optional; if not provided, the default value is 2099-01-01._**

* Response Body
```shell
{
	"result_code": 200,
	"msg": "success",
	"result_size": 4,
	"taskIds": [
		"20240411_13_36_11_922124",
		"20240411_14_37_10_405715",
		"20240411_14_53_35_843805",
		"20240411_14_54_23_785178"
	]
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content returned.
result_size|int|Number of tasks.
taskIds|[]string|List of task IDs.

## Interface to Query Task Details by Task ID
* Purpose: Query task details using the Task ID.

* Protocol Type: GET
* URL: https://ngpu.ai/user/getDetailed?taskID=20240415_14_30_09_891041

* Response Body
```shell
{
	"result_code": 200,
	"msg": "success",
	"taskInfo": {
		"id": 2687,
		"userkey": "ts-22a51e64-a535-4c2a-8c20-3e7f2b4fbc2c",
		"btcaddress": "0000000000000000000000000GFg7xJaNVN2",
		"workspaceid": "367614711220405248",
		"taskid": "20240415_14_30_09_891041",
		"requrl": "/user/schedulingTask?paramUrl=create",
		"method": "schedulingTask",
		"userip": "94.74.66.215",
		"nodeip": "43.240.1.180",
		"nodeaddr": "0xCac10F51814E8e425a6877951fecb7746161c669",
		"requesttime": "2024-04-15 14:30:11",
		"responsetime": "2024-04-15 14:30:12",
		"request": "{\"gz\": true, \"py\": \"H4sIAHHJHGYC/61UXW/TMBR9z6+wvIc20LpNOyZaqRM8ARIaExsSaKosL3EaM8fXsp2Wahq/HSemXbdkExK7L3HuOffr+EOUGoxDYKPcQIncVgu1QiJ4PwvrogDcCK1J5YS0O9ByV2kqYbXiZoBMpWgKZclUFkXBiRYPOH1sUyO0o1poHEdRlPEc+TUVyjomZV+z9IatuJ03da+sM8sYDU/RGSg+j5A3jPGnQEY7Mqps3bDPQ6KG896sbGDXtqf173POkSv4PeIA/e2A7Io039A1ESqHfo59gR0N3e5i7/wcNRUqpyvn5z1Q4XFMD/XITxBqP2Z8hwcoGaNX6GQcd1TcRfp8WnLHB+g21KmrNuJxxa4lp9rAr20/DkODJVythQF1hQvndEDx0jfX/M9HI8mEvdnOJ1NTiGlh8moC03fXCQl+4ut57HiW4M6E9kUzfry8PKfnX798//GCCS/+K2PQNhP2OXGJBt0/FDh+kmCfZRwI8DThYs+IRI4oVazklKKFH4/SkglFKQ4dPjwSjevwiu3vxdV+VdsRwtrWlxsPHvut5jxz3LphKkUb/nD+rTNMb4epruqT3IWpddkRwwTUerWB+u1peY/Q8F+sFeWF8Ar5/c1z/3SAsi+Ut6e3aZWxXqv7UIkb+3sxJpOT9ngOTFqcLiZkTJKOMfEwR8058qc4g42SwDKit00UAbMabQo5av5omI0UrpSo8Sx81oQkr9MqSd52VDZM2RxM2XR3TKazjl1JUy65YY537KSQEjanixl5Q8ZtGDRX6Xroey1APYJxkOvAu2xWcfQHYMkP3JIGAAA=\", \"ts\": \"1713162609.246125\", \"sig\": \"qkwm0rSayLeFxavtMkaTmwqIxzuzHI1BucQ8UwGiEm2N23/5uRSm6i9AJvWVCCpJKbia3s9grB/59BsLDLbgBA==\"}",
		"state": 3,
		"response": "{\"task_id\": \"37a41eba-23d2-4490-aca9-b84b1c2408f6\"}",
		"recordDursion": 1,
		"videodursion": 0,
		"gpudursion": 1,
		"watermarktime": "",
		"watermarkres": ""
	}
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|Return code: 200 indicates success, non-200 indicates failure.
msg|string|Message content returned.
id|int|Identifier of the task.
userkey|string|User's unique userKey.
btcaddress|string|User's BTC address.
workspaceid|string|Workspace ID.
taskid|string|Task ID.
requrl|string|URL path of the request.
method|string|Method used.
userip|string|IP address of the user initiating the request.
nodeip|string|IP address of the compute node processing the request.
nodeaddr|string|Address of the compute node processing the request.
requesttime|string|Time the request was initiated.
responsetime|string|Time of the response.
request|string|Request data content (recorded here; encryption is possible in the app).
state|int|Current state of the task (3: task has been completed).
response|string|Data content returned by the compute node (recorded here; encryption is possible in the workspace).
recordDursion|int|Time taken to record.
~~videodursion|int|Deprecated~~
gpudursion|int|Time taken by GPU processing.
~~watermarktime|string|Deprecated~~
~~watermarkres|string|Deprecated~~








