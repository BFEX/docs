---
title: BFEX API 文档 v1.0

language_tabs: # must be one of https://git.io/vQNgJ
  - json

toc_footers:

includes:

search: true
---

# 更新日志

<style>
table {
    max-width:100%
}
table th {
    white-space: nowrap; /*表头内容强制在一行显示*/
}
</style>
| 生效时间<BR>(UTC +8) | 接口 | 变化      | 摘要 |
|-----|-----|-----|-----|
|2020.8.13 19:00|`POST /open/spot/kline`,<br> `GET /open/spot/products`,<br> `GET /open/spot/ticker`,<br> `POST /open/spot/depth`,<br> `POST /open/spot/trades`,<br> `POST /open/spot/order/place`,<br> `GET /open/user/assets`,<br> `GET /open/spot/order/open`,<br> `GET /open/spot/order/history`,<br> `GET /open/spot/order/dealhistory`,<br> `POST /open/spot/order/cancel`,|新增|创建相关对外接口 |

# 简介

### API 简介

欢迎使用BFEX API！  

此文档是BFEX API的唯一官方文档，API提供的能力会在此持续更新，请大家及时关注。  

文档右侧是针对请求参数以及响应结果的示例。

# 快速入门

## 接入准备
如需使用API ，请先登录网页端，完成API key的申请和权限配置，再据此文档详情进行开发和交易。

每个用户可创建5组Api Key，每个Api Key 目前均有交易权限。

创建成功后请务必记住以下信息：

- Access Key API 访问密钥
- Secret Key 签名认证加密所使用的密钥（仅申请时可见）

<aside class="warning">
<red><b>风险提示</b></red>：这两个密钥与账号安全紧密相关，无论何时都请勿将二者<b>同时</b>向其它人透露。API Key的泄露可能会造成您的资产损失（即使未开通提币权限），若发现API Key泄露请尽快删除该API Key。
</aside> 

**测试环境**

您在正式环境交易之前，可通过测试环境提前体验BFEX API的相关功能，包含已上线功能和即将上线的新功能。

测试环境域名如下：

 **Restful**

 **`https://demo.com/`**

**REST API**

REST，即Representational State Transfer的缩写，是目前较为流行的基于HTTP的一种通信机制，每一个URL代表一种资源。

交易或资产提币等一次性操作，建议开发者使用REST API进行操作。

**接口鉴权**

每个请求必须使用您的API Key进行签名验证。

## 签名说明

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，所有接口均必须使用您的 API Key 做签名认证，以校验参数或参数值在传输途中是否发生了更改。

一个合法的请求由以下几部分组成：

- 方法请求地址：即访问服务器地址 demo.com，比如 demo.com/open/spot/kline。
- 请求头部：Content-Type: application/json 
- API 访问Id（AccessKeyId）：您申请的 API Key 中的 Access Key。
- 签名方法（SignatureMethod）：用户计算签名的基于哈希的协议，此处使用 HmacSHA256。
- 时间戳（Timestamp）：您发出请求的时间 (unixstamp 时间) 。如：1597299511。在查询请求中包含此值有助于防止第三方截取您的请求。
- 必选和可选参数：每个方法都有一组用于定义 API 调用的必需参数和可选参数。可以在每个方法的说明中查看这些参数及其含义。
- 对于 GET 请求，每个方法自带的参数都需要进行签名运算。
- 对于 POST 请求，每个方法的某些参数可以不进行签名认证，具体看方法说明，并且参数需要放到 body 中。
- 签名：签名计算得出的值，用于确保签名有效和未被篡改。

## 签名步骤

规范要计算签名的请求 因为使用 HMAC 进行签名计算时，使用不同内容计算得到的结果会完全不同。所以在进行签名计算前，请先对请求进行规范化处理。下面以查询某产品K线请求为例进行说明：

查询K线时完整的请求URL
`https://demo.com/open/spot/kline?`

`apikey=843a48d61525578f6bc16932b51c69f3`

`&ts=1597300582`

`&sign=ac2e9f0ecdef5c51f928d42b000c08a792c5b4fe28b1a65b43df53c4e50a38c6`

**1. 先将所有参数包括 body 里的参数按照ASCII码顺序进行排序**
`apikey=843a48d61525578f6bc16932b51c69f3&period=1min&symbol=MSVUSDT&ts=1597300582`

**2. 用上一步生成的字符串和你的密钥 (Secret Key) 生成最终的签名字符串，注意密匙签名需要加上'&'符号**
`&21618F1D-22F9-F397-7ABE-01A99F6E56B5`

**3. 调用HmacSHA256哈希函数来获得哈希值。生成一个数字签名 HmacSHA256 之后得到一个二进制内容，将内容转换成字符串**

**4. 最终，发送到服务器的 API 请求应该为**
`https://demo.com/open/spot/kline?apikey=843a48d61525578f6bc16932b51c69f3&ts=1597300582&sign=ac2e9f0ecdef5c51f928d42b000c08a792c5b4fe28b1a65b43df53c4e50a38c6`

**签名方法示例(PHP)**
<aside class="sources">
<pre class="code">
/**
 * 生成请求签名
 * @param array $param 请求参数
 * @param string $secret 密钥
 */ 
function createSign(array $param, $secret) {
    $str = ArrayToString($param) . "&" . $secret;
    return strtolower(bin2hex(hash_hmac("sha256", $str, "", true)));
}

/**
 * 将请求参数转成字符串
 * @param array $arr 请求参数
 */
function ArrayToString(array $arr) {
    ksort($arr);
    foreach ($arr as $k => $v) {
        if (strtolower($k) == "sign" || (is_string($v) && trim($v) == "")) continue;
        if (is_array($v)) $v = ArrayToString($v);
        $tmp[] = sprintf("%s=%s", $k, $v);
    }
    return implode("&", $tmp);
}
</pre>
</aside>

## 请求格式

所有的API请求都是restful，目前只有两种方法：GET和POST

* GET请求：所有的参数都在路径参数里
* POST请求，所有参数以JSON格式发送在请求主体（body）里

## 返回格式

所有的接口都是JSON格式。在JSON最上层有三个字段：`status`, `msg`,  和 `data`。前两个个字段表示请求状态和状态文字说明，实际的业务数据在`data`字段里。当status=200为请求成功其余状态均属失败，具体原因在msg里会有说明。

以下是一个返回格式的样例：

```json
{
    "status": 200,
    "msg": "ok",
    "data": [
        {
            "id": 1,
            "symbol": "BTCUSDT",
            "name": "BTC/USDT",
            "group": 0,
            "sort": 1,
            "state": 1,
            "min_amount": 0.01,
            "min_cashed": 2,
            "min_tick": 0.01,
            "amount_precision": 1.0e-6,
            "cashed_precision": 1.0e-8,
            "maker_fee_rate": 0.002,
            "taker_fee_rate": 0.002
        }
    ]
}
```

参数名称| 数据类型 | 描述
--------- | --------- | -----------
status    | string    | API接口返回状态
msg       | string    | API接口返回状态的文字描述
data      | object    | 接口返回数据主体

## 数据类型

本文档对JSON格式中数据类型的描述做如下约定：

- `string`: 字符串类型，用双引号（"）引用
- `int`: 32位整数，主要涉及到状态码、大小、次数等
- `long`: 64位整数，主要涉及到Id和时间戳
- `float`: 浮点数，主要涉及到金额和价格，建议程序中使用高精度浮点型

# 行情数据

## 市场产品

### HTTP 请求

- GET `/open/spot/products`

此接口不接受任何参数。

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": [
        {
            "id": 1,
            "symbol": "BTCUSDT",
            "name": "BTC/USDT",
            "group": 0,
            "sort": 1,
            "state": 1,
            "min_amount": 0.01,
            "min_cashed": 2,
            "min_tick": 0.01,
            "amount_precision": 1.0e-6,
            "cashed_precision": 1.0e-8,
            "maker_fee_rate": 0.002,
            "taker_fee_rate": 0.002
        }
        ...
    ]
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id | integer | 产品唯一id
symbol | string | 产品标识
group | integer | 分组id
sort | integer | 排序
state | integer | 状态，1=可用，0=停用
min_amount | float | 最小交易数量
min_cashed | float | 最小交易数额
min_tick | float | 最小价格变动单位
amount_precision | float | 交易量精度
cashed_precision | float | 交易额精度
maker_fee_rate | float | maker 手续费率
taker_fee_rate | float | taker 手续费率


## K线数据（蜡烛图）

此接口返回历史K线数据。

### HTTP 请求
- POST `/open/spot/kline`

### 请求参数

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
symbol    | string    | true     | NA      | 交易对  | btcusdt, msvusdt 等
period    | string    | true     | NA      | 返回数据时间粒度，也就是每根蜡烛的时间区间 | 1min 5min 1hour 1day
size      | integer   | false    | 150     | 返回 K 线数据条数 | [1, 2000]
page      | integer   | false    | 1       | 翻页参数 | 视乎数据量有多少
startDate | string    | false    | NA      | 开始时间 | 自定义时间区间, 格式：YYYY-MM-DD HH:MM:SS
endDate   | string    | false    | NA      | 结束时间 | 自定义时间区间, 格式：YYYY-MM-DD HH:MM:SS

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "list": [
            {
                "open": 21.7859,
                "high": 21.7859,
                "low": 21.7859,
                "close": 21.7859,
                "amount": 0,
                "vol": 0,
                "count": 0,
                "ts": 1597387800,
                "period": "1MIN"
            }
        ]
    }
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
amount    | float     | 成交量
count     | integer   | 成交笔数
open      | float     | 本阶段开盘价
close     | float     | 本阶段收盘价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 成交额
ts        | long      | 时间戳，单位秒


## 最新价

### HTTP 请求
- GET `/open/spot/ticker`

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "tick": {
            "MSVUSDT": {
                "id": 1597212859,
                "ts": 1597212859,
                "open": 21.5212,
                "high": 21.7859,
                "low": 21.7859,
                "close": 21.7859,
                "amount": 32.413154839319404,
                "count": 6,
                "vol": 700
            },
            ...
        }
    }
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id        | long      | id值
ts        | long      | 时间戳，单位秒
close     | float     | 最新价
open      | float     | 24小时开盘价
low       | float     | 24小时最低价
high      | float     | 24小时最高价
vol       | float     | 24小时成交额
amount    | float     | 24小时成交量
count     | float     | 24小时成交笔数


## 深度行情数据

### HTTP 请求
- GET `/open/spot/depth`

### 参数

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
symbol    | string    | true     | NA      | 交易对  | btcusdt, msvusdt 等


> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "asks": null,
        "bids": [
            [
                11270.36,
                2
            ],
            [
                11270.34,
                2
            ],
            [
                11260.34,
                0.04
            ]
        ],
        "ts": 1597322283128
    }
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
asks      | object     | 当前的所有卖单, 无数据时是 null 值
bids      | object     | 当前的所有买单, 无数据时是 null 值
ts        | long       | 时间戳，单位毫秒

## 最新成交

### HTTP 请求
- GET `/open/spot/trades`

### 参数

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
symbol    | string    | true     | NA      | 交易对  | btcusdt, msvusdt 等
size      | integer   | false    | 50      | 每页数据量 | [5, 50]
page      | integer   | false    | 1       | 翻页参数 | 视乎数据量有多少


> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "total": 6,
        "hasMore": false,
        "currentPage": 1,
        "lastPage": 1,
        "pageSize": 10,
        "list": [
            {
                "seqid": 12,
                "price": 21.7859,
                "quantity": 9.1802496109869,
                "side": "BUY",
                "symbol": "MSVUSDT",
                "time": 1597212859
            },
            ...
        ]
    }
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
total     | integer    | 总数据量
hasMore   | boolean    | 是否还有下一页
currentPage | integer  | 当前页码
lastPage  | integer    | 最后一页页码
list      | object     | 最新成交价数据
{ seqid | integer | 数据唯一id
price | float | 成交价格
quantity | float | 成交数量
side | string | 方向，buy=买入，sell=卖出
time } | long | 时间戳，单位秒


# 账户相关

## 账户余额

### HTTP 请求
- GET `/open/user/assets`

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "spot": [
            {
                "currency": "USDT",
                "balance": 0,
                "margin": 0
            },
            {
                "currency": "BTC",
                "balance": 0,
                "margin": 0
            }
            ...
        ]
    }
}
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
spot     | object    | 现货账户余额
{ currency | string | 币种
balance | float | 可用余额
margin } | float | 冻结金额

# 现货

## 下单

限频值: 10次/1s

发送一个新订单到BFEX以进行撮合.

### HTTP 请求

- POST `/open/spot/order/place`

> Query body:

```json
{
    "symbol": "btcusdt",
    "price": 11260.34,
    "type": "buy-limit",
    "amount": 0.01,
    "client_order_id": "a001"
}
```

### 请求参数

参数名称    | 数据类型   | 是否必需 | 默认值   | 描述
---------  | --------- | -------- | ------- | -----------
symbol | string | true | NA | 交易对,即 btcusdt, ethusdt... (取值参考 `GET /open/spot/products`)
type | string | true | NA | 订单类型,包括 buy-market, sell-market, buy-limit, sell-limit, buy-trigger, sell-trigger
amount | string | true | NA | 订单交易量(市价买单为订单交易额)
price | string | false | NA | 订单价格 (对市价单无效)
stop-price | string | false | NA | 止盈止损订单触发价格
operator | string | false | NA | 止盈止损订单触发价运算符 gte – greater than and equal (>=), lte – less than and equal (<=)
client_order_id | string | fase | NA | 用户自编订单号（最大长度64个字符，须在24小时内保持唯一性）

### 下单类型
类型 | 描述
---- | ----
buy-market | 市价买入
sell-market | 市价卖出
buy-limit | 限价买入
sell-limit | 限价卖出
buy-trigger | 买入触发, 止盈止损
sell-trigger | 卖出触发, 止盈止损

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "amount": 0.01,
        "cashed": 0,
        "client_order_id": "",
        "closed_at": null,
        "coin": "BTC",
        "created_at": 1597306135,
        "currency": "USDT",
        "fee": 0,
        "fee_currency": "USDT",
        "filled": 0,
        "instrument": "BTC/USDT",
        "pid": "273904828417",
        "price": 11260.34,
        "quantity": 112.6034,
        "status": "已创建",
        "status_code": 2,
        "symbol": "BTCUSDT",
        "trigger_price": 0,
        "trigger_type": "",
        "type": "buy-limit"
    }
}
```

### 响应数据

字段名称 | 数据类型 | 描述
---------| --------- | -----------
pid | integer | 订单号
client_order_id | string | 用户自编订单号
symbol | string | 交易对
instrument | string | 交易对名称
amount | float | 数量
price | float | 价格
quantity | float | 金额。市价买时等于 Amount，其他情况等于 Amount*Price
coin | string | 交易币种。对应 BTC/USDT 中的 BTC
currency | string | 基础币种。对应 BTC/USDT 中的 USDT
fee | float | 手续费
fee_currency | string | 手续费币种
filled | float | 成交数量
cashed | float | 成交金额
status | string | 订单状态文字描述
status_code | integer | 订单状态值
trigger_price | float | 触发价
trigger_type | string | 订单触发价运算符 gte – greater than and equal (>=), lte – less than and equal (<=)
type | string | 下单类型
created_at | integer | 下单时间戳,单位秒
closed_at | integer | 成交时间戳,单位秒

## 撤销委托

此接口发送一个撤销订单的请求。

<aside class="warning">此接口只提交取消请求，实际取消结果需要通过订单状态，撮合状态等接口来确认。</aside>

#### HTTP 请求

- POST `/open/spot/order/cancel`

### 请求参数

| 参数名称     | 是否必须 | 类型     | 描述           | 默认值  | 取值范围 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| pid | true | string | 订单ID |      |      |
| client_order_id | false | string | 用户自编订单号

<aside class="notice">如果两个参数都填入,系统自动忽略 `client_order_id`</aside>

> Query body:

```json
{
    "pid": "1373064724486"
}
```

### 响应数据

返回的主数据对象是一个对应下单单号的字符串


> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "pid": "1373064724486"
    }
}
```

## 当前委托

查询已提交但是仍未完全成交或未被撤销的订单

### HTTP 请求

- GET `/open/spot/order/open`

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
size      | integer   | false    | 50      | 每页数据量 | [5, 50]
page      | integer   | false    | 1       | 翻页参数 | 视乎数据量有多少


> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "total": 894,
        "hasMore": true,
        "currentPage": 1,
        "lastPage": 90,
        "pageSize": 10,
        "list": [
            {
                "amount": 1,
                "client_order_id": "",
                "coin": "BTC",
                "created_at": 1597473866,
                "currency": "USDT",
                "fee": 0,
                "fee_currency": "BTC",
                "filled": 0,
                "instrument": "BTC/USDT",
                "pid": "254317814611969",
                "price": 11260.34,
                "quantity": 11260.34,
                "status": "已创建",
                "status_code": 2,
                "symbol": "BTCUSDT",
                "trigger_price": 0,
                "trigger_type": "",
                "type": "buy-limit"
            }
            ...
        ]
    }
}
```

### 响应数据

字段名称 | 数据类型 | 描述
------- | ------- | -----
total     | integer    | 总数据量
hasMore   | boolean    | 是否还有下一页
currentPage | integer  | 当前页码
lastPage  | integer    | 最后一页页码
list      | object     | 当前委托数据
{ amount | float | 数量
client_order_id | string | 用户自编订单号
symbol | string | 交易对
instrument | string | 交易对名称
amount | float | 数量
price | float | 价格
quantity | float | 金额。市价买时等于 Amount，其他情况等于 Amount*Price
coin | string | 交易币种。对应 BTC/USDT 中的 BTC
currency | string | 基础币种。对应 BTC/USDT 中的 USDT
fee | float | 手续费
fee_currency | string | 手续费币种
filled | float | 成交数量
cashed | float | 成交金额
status | string | 订单状态文字描述
status_code | integer | 订单状态值
trigger_price | float | 触发价
trigger_type | string | 订单触发价运算符 gte – greater than and equal (>=), lte – less than and equal (<=)
type | string | 下单类型
created_at } | integer | 成交时间戳,单位秒

## 历史委托

- GET `/open/spot/order/history`

### 请求参数

 参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
-------|-------|-------|-------|-------
page|integer|false| 1 |页码
size|integer|false| 50 | 每页数据量, 最小5条每页, 最大50条每页

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "total": 141,
        "hasMore": true,
        "currentPage": 1,
        "lastPage": 29,
        "pageSize": 5,
        "list": [
            {
                "amount": 1,
                "coin": "BTC",
                "created_at": 1597473878,
                "currency": "USDT",
                "fee": 22.54072,
                "fee_currency": "USDT",
                "filled": 1,
                "instrument": "BTC/USDT",
                "pid": "254337578172417",
                "price": 11260.34,
                "quantity": 11260.34,
                "status": "完全成交",
                "status_code": 5,
                "symbol": "BTCUSDT",
                "trigger_price": 0,
                "trigger_type": "",
                "type": "sell-limit"
            }
            ...
        ]
    }
}
```

### 响应数据

字段名称 | 数据类型 | 描述
------- | ------- | -----
total     | integer    | 总数据量
hasMore   | boolean    | 是否还有下一页
currentPage | integer  | 当前页码
lastPage  | integer    | 最后一页页码
list      | object     | 历史委托数据
{ amount | float | 数量
client_order_id | string | 用户自编订单号
symbol | string | 交易对
instrument | string | 交易对名称
amount | float | 数量
price | float | 价格
quantity | float | 金额。市价买时等于 Amount，其他情况等于 Amount*Price
coin | string | 交易币种。对应 BTC/USDT 中的 BTC
currency | string | 基础币种。对应 BTC/USDT 中的 USDT
fee | float | 手续费
fee_currency | string | 手续费币种
filled | float | 成交数量
cashed | float | 成交金额
status | string | 订单状态文字描述
status_code | integer | 订单状态值
trigger_price | float | 触发价
trigger_type | string | 订单触发价运算符 gte – greater than and equal (>=), lte – less than and equal (<=)
type | string | 下单类型,含义请查看 `POST /open/spot/order/place` 下单类型部分
created_at } | integer | 成交时间戳,单位秒

## 历史成交

- GET `/open/spot/order/dealhistory`

### 请求参数

 参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
-------|-------|-------|-------|-------
page|integer|false| 1 |页码
size|integer|false| 50 | 每页数据量, 最小5条每页, 最大50条每页

> Response:

```json
{
    "status": 200,
    "msg": "ok",
    "data": {
        "total": 119,
        "hasMore": true,
        "currentPage": 1,
        "lastPage": 24,
        "pageSize": 5,
        "list": [
            {
                "amount": 1,
                "coin": "BTC",
                "created_at": 1597473878,
                "currency": "USDT",
                "fee": 0.002,
                "fee_currency": "BTC",
                "instrument": "BTC/USDT",
                "pid": "41976659969",
                "price": 11270.36,
                "seqid": 13597,
                "symbol": "BTCUSDT",
                "trigger_type": "",
                "type": "buy-limit"
            }
        ]
    }
}
```

### 响应数据

字段名称 | 数据类型 | 描述
------- | ------- | -----
total     | integer    | 总数据量
hasMore   | boolean    | 是否还有下一页
currentPage | integer  | 当前页码
lastPage  | integer    | 最后一页页码
list      | object     | 历史成交数据
{ seqid | integer | 顺序id 
pid | string | 订单id 
amount | float | 数量
symbol | string | 交易对
instrument | string | 交易对名称
amount | float | 数量
price | float | 价格
coin | string | 交易币种。对应 BTC/USDT 中的 BTC
currency | string | 基础币种。对应 BTC/USDT 中的 USDT
fee | float | 手续费
fee_currency | string | 手续费币种
trigger_type | string | 订单触发价运算符 gte – greater than and equal (>=), lte – less than and equal (<=)
type | string | 下单类型,含义请查看 `POST /open/spot/order/place` 下单类型部分
created_at } | integer | 创建时间,单位秒

