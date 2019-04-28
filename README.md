# ELITEX交易所官方API文档
## 介绍

欢迎使用ELITEX开发者文档。

本文档提供了现货(Spot)业务的账户管理、行情查询、交易功能等相关API的使用方法介绍。 行情API提供市场的公开的行情数据接口，账户和交易API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。

## 开始使用    
REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如下：

* 在RESTful架构中，每一个URL代表一种资源；
* 客户端和服务器之间，传递这种资源的某种表现层；
* 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。

建议开发者使用REST API进行币币交易或者资产提现等操作。

## API接口加密验证
### 生成API Key
在对任何请求进行签名之前，您必须通过 ELITEX 网站【用户中心】-【API】创建一个API Key。 创建API Ksey之前需要先绑定谷歌验证器以及创建资金密码。创建Key后，您将获得2个必须记住的信息：

* API Key
* Secret Key

API Key 和 Secret Key将由交易所系统随机生成。

### 发起请求
所有REST请求都必须包含以下标题：

* ACCESS-KEY：API KEY作为一个字符串。
* ACCESS-SIGN：使用base64编码签名（请参阅签名消息）。
* ACCESS-TIMESTAMP：作为您的请求的时间戳。

所有请求都应该含有application/json类型内容，并且是有效的JSON。

### 签名
ACCESS-SIGN的请求头是对 timestamp + method + requestPath + "?" + queryString + body 字符串(+表示字符串连接)使用 HMAC SHA256 方法加密，通过BASE64 编码输出而得到的。其中，timestamp 的值与 ACCESS-TIMESTAMP 请求头相同。

* method 是请求方法(POST/GET/PUT/DELETE)，字母全部大写。
* requestPath 是请求接口路径。
* queryString GET请求中的查询字符串
* body 是指请求主体的字符串，如果请求没有主体(通常为GET请求)则body可省略。

#### 例如：对于如下的请求参数进行签名

```
url "https://api.elitex.io/api/v1/orders?limit=100"
```

* 获取获取深度信息，以 ETH-BTC 币对为例

```
Timestamp = 1540286290170
Method = "GET"
requestPath = "/api/v1/public/ETH-BTC/orderbook"
queryString= "?size=100"
```
生成待签名的字符串
```
Message = '1540286290170GET/api/v1/public/ETH-BTC/orderbook?size=100'  
```
* 下单，以 ETH-BTC 币对为例
```
Timestamp = 1540286476248 
Method = "POST"
requestPath = "/api/v1/order"
body = {"code":"ETH_BTC","side":"buy","type":"limit","amount":"1","price":"1.001"}
```
生成待签名的字符串
```
Message = '1540286476248POST/api/v1/order{"code":"LTC-BTC","side":"buy","type":"limit","amount":"1","price":"1.001"}'  
```
然后，将待签名字符串添加私钥参数生成最终待签名字符串。

例如：
```
Signature = hmac(secretkey, Message, SHA256)
```
在使用前需要对于Signature进行base64编码
```
Signature = base64.encode(Signature.digest())
```

### 请求交互  
REST访问的根URL：https://api.elitex.io

#### 请求
所有请求基于Https协议，请求头信息中Content-Type 需要统一设置为:'application/json’。

#### 请求交互说明

1. 请求参数：根据接口请求参数规定进行参数封装。
2. 提交请求参数：将封装好的请求参数通过POST/GET/DELETE等方式提交至服务器。
3. 服务器响应：服务器首先对用户请求数据进行参数安全校验，通过校验后根据业务逻辑将响应数据以JSON格式返回给用户。
4. 数据处理：对服务器响应数据进行处理。

#### 成功

HTTP状态码200表示成功响应，并可能包含内容。如果响应含有内容，则将显示在相应的返回内容里面。

#### 常见错误码
* 400 Bad Request – Invalid request forma 请求格式无效
* 401 Unauthorized – Invalid API Key 无效的API Key
* 403 Forbidden – You do not have access to the requested resource 请求无权限
* 404 Not Found 没有找到请求
* 429 Too Many Requests 请求太频繁被系统限流
* 500 Internal Server Error – We had a problem with our server 服务器内部错误
* 如果失败，response body 带有错误描述信息

#### response body 常见code
* 200 成功请求无异常
* 501 请求头中没有携带参数
* 502 时间戳超时
* 503 时间戳非数值型
* 504 签名校验失败

## 标准规范
### 时间戳

除非另外指定，API中的所有时间戳均以毫秒为单位返回。

请求签名中的ACCESS-TIMESTAMP的单位是毫秒，允许用小数表示更精确的时间。请求的时间戳必须在API服务时间的30秒内，否则请求将被视为过期并被拒绝。如果本地服务器时间和API服务器时间之间存在较大的偏差，那么我们建议您使用通过查询API服务器时间来更新http header。

### 例子
```
1524801032573
```
### 数字

为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。

整数（如交易编号和顺序）不加引号。

### 限流
每个IP允许瞬间应发50连接，超过50，阻塞连接，允许5个放入阻塞队列，超过55外的连接丢弃返回503

#### REST API
* 公共接口：我们通过IP限制公共接口的调用：每2秒最多6个请求。
* 私人接口：我们通过用户ID限制私人接口的调用：每2秒最多6个请求。
* 某些接口的特殊限制在具体的接口上注明

## 现货(Spot)业务API参考
### 币币行情API
1. 获取所有币对列表

#### HTTP请求
```
    # Request
    GET /api/v1/public/products
```
```
    # Response
    {
        "code": 200,
        "data": [
            {
            "baseCurrency": "ETH",
            "baseMaxSize": 10,
            "baseMinSize": 1,
            "code": "ETH_BTC",
            "counterCurrency": "BTC",
            "counterIncrement": 1e-8,
            "counterPrecision": 8,
            "volumeIncrement": 8
            },
            ...
        ],
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
code|币对代码
baseCurrency|基础币
baseMinSize|最小委托量
baseMaxSize|最大委托量
counterCurrency|计价币
counterIncrement|最小报价单位
counterPrecision|报价精度
volumeIncrement|委托变动单位

2. 获取币对交易深度
#### HTTP请求
```
    # Request
    GET /api/v1/public/<code>/orderbook
```
```
    # Response
    {
        "code": 200,
        "data": {
            "ask": [
            {
                "amount": 628.8642,
                "price": 0.1
            },
            ...
            ],
            "bid": [
            {
                "amount": 103.632,
                "price": 0.0353
            },
            ...
            ]
        },
        "msg": "success"
    }

```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
asks|卖方深度，数组队列
bids|买方深度，数组队列
price|买卖价格
amount|对应数量

#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
code|String|是|币对, 如 ETH_BTC

3. 获取币对Ticker
#### HTTP请求
```
    # Request
    GET /api/v1/public/<code>/ticker
```
```
    # Response
    {
        "code": 200,
        "data": {
            "c": 0,
            "change": "0",
            "code": "ETH_BTC",
            "h": 0,
            "l": 0,
            "o": 0,
            "t": 1555819258573,
            "v": 0
        },
        "msg": "success"
    }
```
#### 返回值说明（从上到下按顺序)

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
code|币对代码
t|时间戳
h|24小时最高价格
l|24小时最低价格
o|24小时开盘价格
c|24小时收盘价格
v|24小时交易量
change|24小时价格变化幅度，正数为价格上涨，负数为价格下跌
#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
code|String|是|币对,如 ETH_BTC

4. 获取币对历史成交记录，支持分页查询

#### HTTP请求
```
    # Request
    GET /api/v1/public/<code>/trade
```
```
    # Response
    {
        "code": 200,
        "data": [
            {
            "amount": 10,
            "id": 32340,
            "price": 0.1,
            "side": "sell",
            "time": 1554758222000
            },
            ...
        ],
        "msg": "success"
    }
```
#### 返回值说明（按顺序）

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
price|成交价格
amount|成交量
side|Maker成交方向
time|成交时间戳
id|交易编号

#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
code|String|是|币对，如ETH_BTC
limit|Integer|否|请求返回数据量，默认最大值 100
#### 解释说明

* 交易方向 side 表示每一笔成交订单中 maker 下单方向,maker 是指将订单挂在订单深度列表上的交易用户，即被动成交方。
* buy 代表行情下跌，因为 maker 是买单，maker 的买单被成交，所以价格下跌；相反的情况下，sell代表行情上涨，因为此时maker是卖单，卖单被成交，表示上涨。

5. 获取K线数据
#### HTTP请求
```
    # Request
    GET  /api/v1/public/<code>/candles?type=1min&start=start_time&end=end_time
```
```
    # Response
    {
        "code": 200,
        "data": [
            {
            "c": 0.0354,
            "h": 0.0354,
            "l": 0.0354,
            "o": 0.0354,
            "t": 1554084960000,
            "v": 28.24858757
            },
            ...
        ],
        "msg": "success"
    }
```

#### 返回值说明（按顺序）

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
t|K线开始时间戳
l|最低价
h|最高价
o|开盘价（第一笔交易）
c|收盘价（最后一笔交易）
v|交易量

#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
code|String|是|币对如ETH_BTC
type|String|是|K线周期类型如1m/1h/1d/1w
start|String|是|基于ISO 8601标准的开始时间
end|String|是|基于ISO 8601标准的结束时间

6. 获取服务器时间
#### HTTP请求
```
    # Request
    GET /api/v1/public/time
    # Reponse
```
```
    {
        "code": 200,
        "data": {
            "epoch": 1555906478384,
            "iso": "2019-04-22T12:14:38.384Z"
        },
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
iso|为iso 8061标准的时间字符串表达的服务器时间
epoch|时间戳形式表达的服务器时间

### 币币账户API
1. 获取账户信息
#### HTTP请求
```
    # Request
    POST /api/v1/account
```
```

    # Response
    {
        "code": 200,
        "data": [
            {
            "available": 15791.879833478282,
            "balance": 15806.267923091284,
            "coin": "BTC",
            "freezed": 14.388089613
            },
        ...
        ],
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
available|可用资金
balance|币种数量
coin|币种代码
freezed|冻结资金

2. 交易委托

ELITEX 提供限价和市价两种订单类型。
#### HTTP请求
```
    # Request
    POST /api/v1/order
    {
        "amount": 1,
        "code": "ETH_BTC",
        "funds": 0,
        "price": 1,
        "side": 0,
        "type": "limit"
    }
```
```
    # Response
    {
        "code": 200,
        "data": 31479,
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，订单id
msg|返回信息

#### 请求参数
参数名|参数类型|必填|描述
-|-|-|-
code|String|是|币对如ETH_BTC
side|Integer|是|买入为1，卖出为0
type|String|是|限价委托为limit，市价委托为market
amount|String|否|发出限价委托以及市价卖出委托时传递，代表交易币的数量
price|String|否|发出限价委托时传递，代表币对价格
funds|String|否|发出市价买入委托时传递，代表计价币的数量

3. 撤销所有委托

#### HTTP请求
```
    # Request
    POST /api/v1/orders/cancelAll/{code}?limit=50
```
```
    # Response
        {
        "code": 200,
        "data": {
            "failNum": 5,
            "failOrderId": [
                666,
                ...
            ],
            "successNum": 10
        },
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
failNum|撤单失败订单数量
successNum|撤单成功订单数量
failOrderId|撤单失败订单id数组

#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
code|String|否|币对, 如ETH_BTC
limit|Integer|否|撤单数量，默认50，最大1000

4. 按订单撤销委托
#### HTTP请求
```
    # Request
    POST /api/v1/orders/cancel/{orderId}
```
```
    # Response
        {
        "code": 200,
        "data": {
            "failNum": 1,
            "failOrderId": [
                666,
                ...
            ],
            "successNum": 0
        },
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
failNum|撤单失败订单数量
successNum|撤单成功订单数量
failOrderId|撤单失败订单id数组

#### 请求参数

参数名|参数类型|必填|描述
-|-|-|-
orderId|Integer|是|需要撤销的未成交委托的id

5. 查询所有订单，支持分页查询

#### HTTP请求
```
    # Request
    POST /api/v1/orders/getAll/{code}?pageNum=1&pageSize=20
```
```
    # Response
    {
        "code": 200,
        "data": [
            {
            "amount": 1,
            "code": "ETH_BTC",
            "funds": 0,
            "id": 31479,
            "last": 1,
            "avgPrice": 0,
            "price": 1,
            "side": "sell",
            "state": 4,
            "time": 1555884317000,
            "type": "limit"
            }
        ],
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
id|订单id
code|币对代码，如ETH_BTC
avgPrice|成交平均价,未成交则为0
price|订单委托价格
amount|订单委托数量
funds|订单委托资金
last|剩余委托数量
side|
type|订单类型，限价单为limit，市价单为market
state|订单状态
time|订单创建时间

###请求参数
参数名|参数类型|必填|描述
-|-|-|-
code|String|否|币对如ETH_BTC
pageNum|Integer|否|页数，默认1
pageSize|Integer|否|请求每页返回数据量，默认值20,最大值1000

6. 按id查询订单

#### HTTP请求
```
    # Request
    POST /api/v1/orders/get/{id}
```
```
    # Response 
    {
        "code": 200,
        "data": {
            "amount": 1,
            "code": "ETH_BTC",
            "funds": 0,
            "id": 31479,
            "last": 1,
            "avgPrice": 0,
            "price": 1,
            "side": "sell",
            "state": 4,
            "time": 1555884317000,
            "type": "limit"
        },
        "msg": "success"
    }
```
#### 返回值说明

返回字段|字段说明
-|-
code|状态编码，200-成功无异常
data|数据返回，无数据则为空，具体数据类型参考如下
msg|返回信息

#### data值说明

返回字段|字段说明
-|-
id|订单id
code|币对代码，如ETH_BTC
avgPrice|成交平均价,未成交则为0
price|订单委托价格
amount|订单委托数量
funds|订单委托资金
last|剩余委托数量
side|
type|订单类型，限价单为limit，市价单为market
state|订单状态
time|订单创建时间

#### 请求参数
参数名|参数类型|必填|描述
-|-|-|-
id|Integer|是|订单Id
