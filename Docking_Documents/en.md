
Computing Power Demand Side API Integration
=======
* When integrating with the AINNGPU API, computing power demanders must provide their own workspace (a Docker image encapsulating their AI functionality). AINNGPU will deploy this Docker image across multiple locations and select nodes intelligently. Since AINNGPU uses a parameter pass-through method, the following steps can be used for API integration。
-----------

# Integration Steps

+ The computing power demander provides their own runnable Docker image, including the startup command for the Docker image。
+ The computing power demander specifies the type and number of GPUs required。
+ The computing power demander provides the Docker-supported API (AINNGPU will use a pass-through mode for API parameter data transmission)。

# Example of API Integration

## Synchronous Method

### Task Request

#### Original API Interface

* Protocol type: POST
* URL: https://***/img2img
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
* Protocol type: POST
* URL: https://ainngpu.io/user/img2img
* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|User's BTC address (optional, checks for AINN assets if provided, otherwise no check)
data|string|Data content for pass-through (can contain the original API request body)


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
result_code|string|The operation result code
msg|string|Message content
data|string|Task result returned (using pass-through, the original API response body is included here)


## Asynchronous Method
-----------
* The difference between the asynchronous and synchronous methods is that in the asynchronous method, after submitting a task request, a taskID is obtained. The status of this taskID is checked after a certain period of time to see if the task has been completed。


### Task Request

####  Original API Interface

* Protocol type: POST
* URL: https://***/img2img
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
* Protocol type: POST
* URL: https://ainngpu.io/user/img2img
* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|User's BTC address (optional, checks for AINN assets if provided, otherwise no check)
data|string|Data content


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
result_code|string|The operation result code
msg|string|Message content returned
data|string|Task result returned (uses pass-through method, inserting the original API interface's Response Body into this parameter)
taskID|string|Returned task ID for inquiry


### Task Inquiry

####  Original API Interface

* Protocol Type: GET
* URL: https://***/img2img&taskID=20230423_11_23_05_10279

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
result_code|string|The operation result code
msg|string|Message content returned
taskID|string|Returned task ID for inquiry


#### Post-Integration API Interface
* Protocol Type: GET
* URL: https://***/img2img&taskID=20230423_11_23_05_10279

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
result_code|string|The operation result code
msg|string|Message content returned
taskID|string|Returned task ID for inquiry
data|string|Inquiry return result


# General API Interface

## BRC20 Inscription Detection Interface
* Purpose: Detects whether the input BTC address possesses a specified BRC20 inscription.

* Protocol Type: GET
* URL: https://***/brc20/checkAddress&address=bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus

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
exists|bool|Whether the specified BRC20 inscription exists on the user's BTC address
names|[]string|Names of the inscriptions on the user's BTC address
balances|[]int|Quantities of the inscriptions on the user's BTC address (corresponding to the inscription names)
