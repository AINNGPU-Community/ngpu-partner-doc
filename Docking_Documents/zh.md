
算力需求方API对接
=======
* 算力需求方在使用AINNGPU进行API对接时，需要提供给AINNGPU自己的工作空间（封装了自己AI功能的Docker镜像），AINNGPU将会根据根据这个Docker镜像进行多点部署与节点智能选择。由于AINNGPU采用的是参数透传的方式，因此在对接API的时候可以采用以下步骤进行。
-----------

# 对接步骤

+ 算力需求方提供自己可运行的Docker镜像，包括Docker镜像的启动指令。
+ 算力需求方提供自己所需GPU的类型以及部署数量。
+ 算力需求方提供Docker所支持的API（AINNGPU将采用透传模式进行API参数数据的透传）

# API对接例子

## 同步方式

### 任务请求

#### 原有API接口

* 协议类型：POST
* 协议地址：https://***/img2img
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

#### 对接后API接口
* 协议类型：POST
* 协议地址：https://ainngpu.io/user/img2img
* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|用户的BTC地址（可选项，如果存在则会检测用户的地址中是否有AINN资产，如果为空则不检查）
data|string|进行透传的数据内容（可以保存原API接口中的Request Body内容）


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
result_code|int|返回码
msg|string|返回的信息内容
data|string|返回任务结果（这里采用透传方式，将原API接口返回的Response Body放入此参数中）


## 异步方式
-----------
* 异步方式与同步方式的不同在于异步方式提交任务请求后会得到一个taskID，同时间隔一定时间查询这个taskID是否已经完成。


### 任务请求

####  原有API接口

* 协议类型：POST
* 协议地址：https://***/img2img
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

#### 对接后API接口
* 协议类型：POST
* 协议地址：https://ainngpu.io/user/img2img
* Request Body
```shell
{
    "btc_address": "bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus",
    "data": ""
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
btc_address|string|用户的BTC地址（可选项，如果存在则会检测用户的地址中是否有AINN资产，如果为空则不检查）
data|string|进行透传的数据内容（可以保存原API接口中的Request Body内容）


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
result_code|int|返回码
msg|string|返回的信息内容
data|string|返回任务结果（这里采用透传方式，将原API接口返回的Response Body放入此参数中）
taskID|string|返回查询的任务ID


### 任务查询

####  原有API接口

* 协议类型：GET
* 协议地址：https://***/img2img&taskID=

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
result_code|int|返回码
msg|string|返回的信息内容
taskID|string|返回查询的任务ID


#### 对接后API接口
* 协议类型：GET
* 协议地址：https://***/img2img&taskID=

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
result_code|int|返回码
msg|string|返回的信息内容
taskID|string|返回查询的任务ID
data|string|查询返回结果


# 通用API接口

## BRC20铭文检测接口
* 作用：检测输入的BTC地址是否拥有BRC20指定的铭文。

* 协议类型：GET
* 协议地址：https://***/brc20/checkAddress&address=bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus

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
exists|bool|用户BTC地址上是否存在指定的BRC20铭文
names|[]string|用户BTC地址上的铭文名称
balances|[]int|用户BTC地址上存在的铭文数量（与铭文名称对应）
