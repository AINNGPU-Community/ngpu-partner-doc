
算力需求方API对接
=======
* 算力需求方在使用NGPU进行API对接时，需要提供给NGPU自己的工作空间（封装了自己AI功能的Docker镜像），NGPU将会根据根据这个Docker镜像进行多点部署与节点智能选择。由于NGPU采用的是参数透传的方式，因此在对接API的时候可以采用以下步骤进行。
-----------

# 对接步骤

+ 算力需求方提供自己可运行的Docker镜像，包括Docker镜像的启动指令。
+ 算力需求方提供自己所需GPU的类型以及部署数量。
+ 算力需求方提供Docker所支持的API（NGPU将采用透传模式进行API参数数据的透传）

# API对接例子

## 同步方式

### 任务请求

#### 原有API接口

* 协议类型：POST
* 协议地址：https://***/img2img
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

#### 对接后API接口
* 协议类型：POST
* 协议地址：https://ngpu.ai/user/img2img
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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
data|string|返回任务结果（这里采用透传方式，将原API接口返回的Response Body放入此参数中）


## 异步方式
-----------
* 异步方式与同步方式的不同在于异步方式提交任务请求后会得到一个taskID，同时间隔一定时间查询这个taskID是否已经完成。


### 任务请求

####  原有API接口

* 协议类型：POST
* 协议地址：https://***/sdapi/v1/img2img
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

#### 对接后API接口
* 协议类型：POST
* 协议地址：https://ngpu.ai/user/schedulingTask?paramUrl=sdapi/v1/img2img&paramPort=8080
* 协议头：Authorization：Bearer 工作空间ID

Parameter:
- paramUrl: 在调用计算节点时使用的URL路径。
- paramPort: 调用算力节点是使用的端口（Docker镜像在启动时所映射出来的端口）

**_这两个参数是为了保障apiServer在进行数据中转时能都正确的中转给算力节点的Docker镜像_**

**后续将增加的接口**
Field Name | Field Type | Field Meaning
----|:----:|:----:|
sadTalker|string|使用开源sadTalker接口
videoReTalker|string|使用开源videoReTalker接口

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
result_code|string|返回值 200为成功，非200为不成功
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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
taskID|string|返回查询的任务ID


#### 对接后API接口
* 协议类型：GET
* 协议地址：https://ngpu.ai/user/queryTask&taskID=20230423_11_23_05_10279

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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
taskID|string|返回查询的任务ID
data|string|查询返回结果


# 通用API接口

## BRC20铭文检测接口
* 作用：检测输入的BTC地址是否拥有BRC20指定的铭文。

* 协议类型：GET
* 协议地址：https://ngpu.ai/brc20/checkAddress&address=bc1pp8vyhh2ma0ntzjwr26xxrn5r0w296yu68wdwle5rrhgtv3a2lgkqtyayus

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


## 照片生成视频
* 作用：用户上传一张照片和文字即可生成一段视频。

* 协议类型：POST
* 协议地址：https://ngpu.ai/user/sadTalker

* Request Body
```shell
{
    "image_url": "https://twinsync.oss-cn-hangzhou.aliyuncs.com/video/1683730315image.jpeg",
    "text": "你好，我是马斯克。听说今天是中国的腊八节，不知道大家吃了腊八粥了没有？我在美国祝大家腊八节快乐！",
    "pronouncer": "fable",
    "backGroundName": "laba",
    "btc_address": "bc1pv0egqadxd60ae4r7v77gewpvjvsza09kccsxztzx8c8pqccn0rgqqw7np5"
}
```


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





## 根据工作空间Id查询任务列表接口
* 作用：根据工作空间ID与起止时间（可以省略），查询任务列表

* 协议类型：GET
* 协议地址：https://ngpu.ai/user/queryTaskList?startTime=2024-04-11&endTime=2024-04-20
* 协议头：Authorization：Bearer 工作空间ID  

**_startTime非必填，如果没有，则按照1970-01-01时间取值，endTime非必填，如果没有则按照2099-01-01取值_**

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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
result_size|int|任务数量
taskIds|[]string|任务ID列表

## 根据工作空间Id查询分发算力节点列表接口
* 作用：根据工作空间ID与起止时间（可以省略），查询分发的算力节点列表

* 协议类型：GET
* 协议地址：https://ngpu.ai/user/queryNodes
* 协议头：Authorization：Bearer 工作空间ID  

* Response Body
```shell
{
	"result_code": 200,
	"msg": "success",
	"result_size": 2,
	"nodes": [
		{
			"nodeAddr": "0x732637A3A3E0D335Dc00d9F8fba8a1033831Bf13",
			"progress": 100,
			"status": 1
		},
		{
			"nodeAddr": "0xf54B48DBD69d4f0841e458a551422Cf8eC1DEaaa",
			"progress": 100,
			"status": 0
		}
	]
}
```
Field Name | Field Type | Field Meaning
----|:----:|:----:|
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
result_size|int|任务数量
nodes|[]string|分发的算力节点
nodeAddr|string|算力节点地址
progress|int|分发进度
status|int|分发状态

## 根据工作空间Id、算力节点地址查询任务列表接口
* 作用：根据工作空间ID、算力节点地址与起止时间（可以省略），查询任务列表

* 协议类型：GET
* 协议地址：https://ngpu.ai/user/getTasksFromNode?nodeAddr=0xf54B48DBD69d4f0841e458a551422Cf8eC1DEaaa&startTime=2024-04-11&endTime=2024-04-20
* 协议头：Authorization：Bearer 工作空间ID  

**_startTime非必填，如果没有，则按照1970-01-01时间取值，endTime非必填，如果没有则按照2099-01-01取值_**

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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
result_size|int|任务数量
taskIds|[]string|任务ID列表

## 根据任务ID查询任务详细内容接口
* 作用：根据工作空间ID与起止时间（可以省略），查询任务列表

* 协议类型：GET
* 协议地址：https://ngpu.ai/user/getDetailed?taskID=20240415_14_30_09_891041

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
result_code|string|返回值 200为成功，非200为不成功
msg|string|返回的信息内容
id|int|标记任务的id
userkey|string|用户使用的userKey
btcaddress|string|用户的BTC地址
workspaceid|string|工作空间ID
taskid|string|任务ID
requrl|string|请求的url路径
method|string|使用的方法
userip|string|发起请求的用户Ip地址
nodeip|string|处理请求的算力节点Ip
nodeaddr|string|处理请求的算力节点地址
requesttime|string|请求发起时间
responsetime|string|相应时间
request|string|请求的数据内容（此处只做记录，app可以进行加密）
state|int|任务当前状态（3：任务已经完成）
response|string|算力节点返回的数据内容（此处只做激励，算力节点中的工作空间可以进行加密）
recordDursion|int|记录消耗时间
~~videodursion|int|已经废弃~~
gpudursion|int|GPU消耗时间
~~watermarktime|string|已废弃~~
~~watermarkres|string|已废弃~~








