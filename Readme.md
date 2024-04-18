
API Integration for Compute Power Demand Side
=======
* When using AINNGPU for API integration, the compute power demand side needs to provide their workspace (a Docker image encapsulating their own AI capabilities). AINNGPU will deploy this Docker image across multiple nodes and make intelligent node selections. Since AINNGPU employs a parameter pass-through method, the following steps can be used during API integration.
-----------

# Integration Steps

+ The compute power demand side provides their runnable Docker image, including the Docker image's startup commands.
+ The compute power demand side provides the type and number of GPUs required.
+ The compute power demand side provides the Docker-supported APIs (AINNGPU will use a pass-through mode for API parameter data transmission).

# Example of API Integration

## Synchronous Mode

### Task Request

#### Original API Interface

* Protocol Type: POST
* Protocol Address: https://***/img2img
* Request Body
```shell
{
    "image_url": "https://ainngpu.io/image/1683730315image.jpeg",
    "prompt": "manicured claws, score_9, score_8_up, score_7_up, score_6_up, score_5_up, score_4_up, 1girl, realistic, extremely detailed, high quality, , beautiful eyes, cat woman, black suit, character concept,night time, dark shadows, dutch angle"
}
```
* Response Body
```shell
{
	"result_code": 100,
	"msg": "SUCCESS! The video is currently in the queue waiting to be processed",
    "image_url": "https://ainngpu.io/image/16837303689.jpeg"
}
```

#### Post-Integration API Interface
* Protocol Type: POST
* Protocol Address: https://ainngpu.io/user/img2img
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
result_code|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
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
    "image_url": "https://ainngpu.io/image/1683730315image.jpeg",
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
* Protocol Address：https://ainngpu.io/user/schedulingTask?paramUrl=sdapi/v1/img2img&paramPort=8080

Parameter:
- paramUrl: URL path used when calling the compute node.
- paramPort: Port used when calling the compute node (the port mapped when the Docker image starts).

**_These parameters ensure that the apiServer can correctly transfer data to the compute node's Docker image._**

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
result_code|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
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
result_code|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
msg|string|Message content
taskID|string|Returned task ID


#### Post-Integration API Interface
* Protocol Type: GET
* Protocol Address: https://ainngpu.io/user/queryTask&taskID=20230423_11_23_05_10279

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
result_code|string|Optional user's BTC address (checks for AINN assets if provided, skips if empty)
msg|string|Message content
taskID|string|Returned task ID
data|string|Query return result


# Common API Interface

## BRC20 Inscription Detection Interface
* Function: Checks if a specified BRC20 inscription exists on a given BTC address.

* Protocol Type: GET
* Protocol Address: https://ainngpu.io/brc20/checkAddress&address=bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus

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
