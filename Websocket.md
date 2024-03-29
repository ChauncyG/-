# Websocket行情数据

## 简介

WebSocket协议是基于TCP的一种新的网络协议。它实现了客户端与服务器之间在单个 tcp 连接上的全双工通信，由服务器主动发送信息给客户端，减少了频繁的身份验证等不必要的开销。其最大优点有两个：

- 两方请求的 header 数据很小，大概只有2 Bytes。
- 服务器不再是被动的接到客户端的请求后才返回数据，而是有了新数据后主动推送给客户端。

以上 WebSocket 协议带来的优点使得其十分适用于数字货币行情和交易这种实时性强的接口。

### 接入URL

**行情请求地址**

- 行情请求地址为：**`wss://{HOST}/ws`**
- HOST常用的格式说明：

需要验签的接口：www.xxxx.com

不需要验签的接口：www.xxxx.com/api

**（如果host的格式有问题，请咨询host提供方）**

### 数据压缩

WebSocket API 返回的所有数据都进行了 GZIP 压缩，需要 client 在收到数据之后解压，推荐使用pako。（<a href="<https://github.com/nodeca/pako>">【pako】</a> 是一个支持压缩和解压 GZIP 的库）

### 心跳消息

<aside class="notice">注：返回的数据里面的 `"pong"` 的值为收到的 `"ping"` 的值</aside>

当用户的Websocket客户端连接到网站Websocket服务器后，服务器会定期（当前设为5秒）向其发送`ping`消息并包含一整数值如下：

> {"ping": 1492420473027}

当用户的Websocket客户端接收到此心跳消息后，应返回`pong`消息并包含同一整数值：

> {"pong": 1492420473027}

<aside class="warning">当Websocket服务器连续两次发送了`ping`消息却没有收到任何一次`pong`消息返回后，服务器将主动断开与此客户端的连接。</aside>

### 订阅主题

成功建立与Websocket服务器的连接后，Websocket客户端发送如下请求以订阅特定主题：

```json
{
  "sub": "market.btccny.kline.1min",
  "id": "id1"
}
```

{
  "sub": "topic to sub",
  "id": "id generate by client"
}

成功订阅后，Websocket客户端将收到确认：

```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btccny.kline.1min",
  "ts": 1489474081631
}
```

之后, 一旦所订阅的主题有更新，Websocket客户端将收到服务器推送的更新消息（push）：

```json
{
  "ch": "market.btccny.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```

### 取消订阅

取消订阅的格式如下：

```json
{
  "unsub": "market.btccny.trade.detail",
  "id": "id4"
}
```

{
  "unsub": "topic to unsub",
  "id": "id generate by client"
}

取消订阅成功确认：

```json
{
  "id": "id4",
  "status": "ok",
  "unsubbed": "market.btccny.trade.detail",
  "ts": 1494326028889
}
```

### 请求数据

Websocket服务器同时支持一次性请求数据（pull）。

请求数据的格式如下：

```json
{
  "req": "market.ethbtc.kline.1min",
  "id": "id10"
}
```

{
  "req": "topic to req",
  "id": "id generate by client"
}

一次性返回的数据：

```json
{
  "status": "ok",
  "rep": "market.btccny.kline.1min",
  "tick": [
    {
      "amount": 1.6206,
      "count":  3,
      "id":     1494465840,
      "open":   9887.00,
      "close":  9885.00,
      "low":    9885.00,
      "high":   9887.00,
      "vol":    16021.632026
    },
    {
      "amount": 2.2124,
      "count":  6,
      "id":     1494465900,
      "open":   9885.00,
      "close":  9880.00,
      "low":    9880.00,
      "high":   9885.00,
      "vol":    21859.023500
    }
  ]
}
```

## K线数据

### 主题订阅

一旦K线数据产生，Websocket服务器将通过此订阅主题接口推送至客户端：

`market.$symbol$.kline.$period$`

> 订阅请求

```json
{
  "sub": "market.ethbtc.kline.1min",
  "id": "id1"
}
```

### 参数

| 参数   | 数据类型 | 是否必需 | 描述     | 取值范围                                                  |
| ------ | -------- | -------- | -------- | --------------------------------------------------------- |
| symbol | string   | true     | 交易代码 | All supported trading symbols, e.g. btcusdt, bccbtc       |
| period | string   | true     | K线周期  | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |

> Response

```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.ethbtc.kline.1min",
  "ts": 1489474081631
}

```

> Update example

```json
{
  "ch": "market.ethbtc.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```

### 数据更新字段列表

| 字段   | 数据类型 | 描述                                        |
| ------ | -------- | ------------------------------------------- |
| id     | integer  | unix时间，同时作为K线ID                     |
| amount | float    | 成交量                                      |
| count  | integer  | 成交笔数                                    |
| open   | float    | 开盘价                                      |
| close  | float    | 收盘价（当K线为最晚的一根时，是最新成交价） |
| low    | float    | 最低价                                      |
| high   | float    | 最高价                                      |
| vol    | float    | 成交额, 即 sum(每一笔成交价 * 该笔的成交量) |

<aside class="notice">当symbol被设为“hb10”或“huobi10”时，amount，count，vol均为零值。</aside>

### 数据请求

用请求方式一次性获取K线数据需要额外提供以下参数：

```json
{
  "req": "market.$symbol.kline.$period",
  "id": "id generated by client",
  "from": "from time in epoch seconds",
  "to": "to time in epoch seconds"
}
```

| 参数 | 数据类型 | 是否必需 | 缺省值                                | 描述                            | 取值范围                                                     |
| ---- | -------- | -------- | ------------------------------------- | ------------------------------- | ------------------------------------------------------------ |
| from | integer  | false    | 1501174800(2017-07-28T00:00:00+08:00) | 起始时间 (epoch time in second) | [1501174800, 2556115200]                                     |
| to   | integer  | false    | 2556115200(2050-01-01T00:00:00+08:00) | 结束时间 (epoch time in second) | [1501174800, 2556115200] or ($from, 2556115200] if "from" is set |

## 市场深度行情数据

当市场深度发生变化时，此主题发送最新市场深度更新数据。

### 主题订阅

`market.$symbol.depth.$type`

> Subscribe request

```json
{
  "sub": "market.btcusdt.depth.step0",
  "id": "id1"
}
```

### 参数

| 参数   | 数据类型 | 是否必需 | 缺省值 | 描述         | 取值范围                                            |
| ------ | -------- | -------- | ------ | ------------ | --------------------------------------------------- |
| symbol | string   | true     | NA     | 交易代码     | All supported trading symbols, e.g. btcusdt, bccbtc |
| type   | string   | true     | step0  | 合并深度类型 | step0, step1, step2, step3, step4, step5            |

**"type" 合并深度类型**

| Value | Description                          |
| ----- | ------------------------------------ |
| step0 | 不合并深度                           |
| step1 | Aggregation level = precision*10     |
| step2 | Aggregation level = precision*100    |
| step3 | Aggregation level = precision*1000   |
| step4 | Aggregation level = precision*10000  |
| step5 | Aggregation level = precision*100000 |

> Response

```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.depth.step0",
  "ts": 1489474081631
}

```

> Update example

```json
{
  "ch": "market.btcusdt.depth.step0",
  "ts": 1489474082831,
  "tick": {
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560]
    // more Market Depth data here
    ],
    "asks": [
    [10010.9800,0.0099],
    [10011.3900,2.0000]
    //more data here
    ]
  }
}
```

### 数据更新字段列表

<aside class="notice">在'tick'object下方呈现买盘卖盘深度列表</aside>

| 字段 | 数据类型 | 描述                                                 |
| ---- | -------- | ---------------------------------------------------- |
| bids | object   | The current all bids in format [price, quote volume] |
| asks | object   | The current all asks in format [price, quote volume] |

<aside class="notice">当symbol被设为"hb10"时，amount, count, vol均为零值 </aside>

### 数据请求

支持数据请求方式一次性获取市场深度数据：

```json
{
  "req": "market.btcusdt.depth.step0",
  "id": "id10"
}
```

## 成交明细

### 主题订阅

此主题提供市场最新成交明细。

`market.$symbol.trade.detail`

> Subscribe request

```json
{
  "sub": "market.btcusdt.trade.detail",
  "id": "id1"
}
```

### 参数

| 参数   | 数据类型 | 是否必需 | 缺省值 | 描述     | 取值范围                                            |
| ------ | -------- | -------- | ------ | -------- | --------------------------------------------------- |
| symbol | string   | true     | NA     | 交易代码 | All supported trading symbols, e.g. btcusdt, bccbtc |

> Response

```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.trade.detail",
  "ts": 1489474081631
}
```

> Update example

```json
{
  "ch": "market.btcusdt.trade.detail",
  "ts": 1489474082831,
  "tick": {
        "id": 14650745135,
        "ts": 1533265950234,
        "data": [
            {
                "amount": 0.0099,
                "ts": 1533265950234,
                "id": 146507451359183894799,
                "price": 401.74,
                "direction": "buy"
            }
            // more Trade Detail data here
        ]
  }
}
```

### 数据更新字段列表

| 字段      | 数据类型 | 描述                                           |
| --------- | -------- | ---------------------------------------------- |
| id        | integer  | 唯一成交ID                                     |
| amount    | float    | 成交量                                         |
| price     | float    | 成交价                                         |
| ts        | integer  | 成交时间 (UNIX epoch time in millisecond)      |
| direction | string   | 成交主动方 (taker的订单方向) : 'buy' or 'sell' |

### 数据请求

支持数据请求方式一次性获取成交明细数据（仅能获取最多最近300个成交记录）：

```json
{
  "req": "market.btcusdt.trade.detail",
  "id": "id11"
}
```

## 市场概要

### 主题订阅

此主题提供24小时内最新市场概要。

`market.$symbol.detail`

> Subscribe request

```json
{
  "sub": "market.btcusdt.detail",
  "id": "id1"
}

```

### 参数

| 参数   | 数据类型 | 是否必需 | 缺省值 | 描述     | 取值范围                                            |
| ------ | -------- | -------- | ------ | -------- | --------------------------------------------------- |
| symbol | string   | true     | NA     | 交易代码 | All supported trading symbols, e.g. btcusdt, bccbtc |

> Response

```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.detail",
  "ts": 1489474081631
}

```

> Update example

```json
  "tick": {
    "amount": 12224.2922,
    "open":   9790.52,
    "close":  10195.00,
    "high":   10300.00,
    "ts":     1494496390000,
    "id":     1494496390,
    "count":  15195,
    "low":    9657.00,
    "vol":    121906001.754751
  }
```

### 数据更新字段列表

| 字段   | 数据类型 | 描述                     |
| ------ | -------- | ------------------------ |
| id     | integer  | unix时间，同时作为消息ID |
| ts     | integer  | unix系统时间             |
| amount | float    | 24小时成交量             |
| count  | integer  | 24小时成交笔数           |
| open   | float    | 24小时开盘价             |
| close  | float    | 最新价                   |
| low    | float    | 24小时最低价             |
| high   | float    | 24小时最高价             |
| vol    | float    | 24小时成交额             |

### 数据请求

支持数据请求方式一次性获取市场概要数据：

```json
{
  "req": "market.btcusdt.detail",
  "id": "id11"
}
```

# Websocket资产及订单

## 简介

### 接入URL

**Websocket资产及订单**

**`wss://｛Host｝/ws/v1`**

请使用中国大陆以外的服务器访问网站API。

### 数据压缩

WebSocket API 返回的所有数据都进行了 GZIP 压缩，需要 client 在收到数据之后解压。

### 心跳消息

当用户的Websocket客户端连接到网站Websocket服务器后，服务器会定期（当前设为5秒）向其发送`ping`消息并包含一整数值如下：

> {"ping": 1492420473027}

当用户的Websocket客户端接收到此心跳消息后，应返回`pong`消息并包含同一整数值：

> {"pong": 1492420473027}

<aside class="warning">当Websocket服务器连续两次发送了`ping`消息却没有收到任何一次`pong`消息返回后，服务器将主动断开与此客户端的连接。</aside>

### 订阅主题

成功建立与Websocket服务器的连接后，Websocket客户端发送如下请求以订阅特定主题：

```json
{
  "op": "operation type, 'sub' for subscription, 'unsub' for unsubscription",
  "topic": "topic to sub",
  "cid": "id generate by client"
}
```

成功订阅后，Websocket客户端将收到确认：

```json
{
  "op": "operation type, refer to the operation which triggers this response",
  "cid": "id1",
  "error-code": 0, // 0 for no error
  "topic": "topic to sub if the op is sub",
  "ts": 1489474081631
}
```

之后, 一旦所订阅的主题有更新，Websocket客户端将收到服务器推送的更新消息（push）：

```json
{
  "op": "notify",
  "topic": "topic of this notify",
  "ts": 1489474082831,
  "data": {
    // data of specific topic update
  }
}
```

### 取消订阅

取消订阅的格式如下：

```json
{
  "op": "unsub",
  "topic": "accounts",
  "cid": "client generated id"
}
```

取消订阅成功确认：

```json
{
  "op": "unsub",
  "topic": "accounts",
  "cid": "id generated by client",
  "err-code": 0,
  "ts": 1489474081631
}
```

### 请求数据

Websocket服务器同时支持一次性请求数据（pull）。

当与Websocket服务器成功建立连接后，以下三个主题可供用户请求：

- accounts.list
- orders.list
- orders.detail

具体请求方式请见后文。

**数据请求限频规则**

限频规则基于API key而不是连接。当请求频率超出限值时，Websocket客户端将收到"too many request"错误码。以下为各主题当前限频设定：

- accounts.list: once every 25 seconds
- orders.list AND orders.detail: once every 5 seconds

### 鉴权

资产及订单主题鉴权请求数据格式如下：

```json
{
  "op": "auth",
  "AccessKeyId": "e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx",
  "SignatureMethod": "HmacSHA256",
  "SignatureVersion": "2",
  "Timestamp": "2017-05-11T15:19:30",
  "Signature": "4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=",
}
```

**鉴权请求数据格式说明**

| filed            | type   | instruction                                                  |
| ---------------- | ------ | ------------------------------------------------------------ |
| op               | string | 必填；操作名称，鉴权固定值为 auth；                          |
| cid              | string | 选填；Client 请求唯一 ID                                     |
| AccessKeyId      | string | 必填；API 访问密钥, 您申请的 APIKEY 中的 AccessKey           |
| SignatureMethod  | string | 必填；签名方法, 用户计算签名的基于哈希的协议，此处使用 HmacSHA256 |
| SignatureVersion | string | 必填；签名协议的版本，此处使用 2                             |
| Timestamp        | string | 必填；时间戳, 您发出请求的时间 (UTC 时区) (UTC 时区) (UTC 时区) 。在查询请求中包含此值有助于防止第三方截取您的请求。如：2017-05-11T16:22:06。再次强调是 (UTC 时区) |
| Signature        | string | 必填；签名, 计算得出的值，用于确保签名有效和未被篡改         |

> **注：**
>
> - 签名计算中请求方法固定值为`GET`

## 订阅账户更新

API Key 权限：读取

订阅账户资产变动更新。

### 主题订阅

`accounts`

> Subscribe request

```json
{
  "op": "sub",
  "cid": "40sG903yz80oDFWr",
  "topic": "accounts",
  "model": "0"
}
```

### 参数

| 参数  | 数据类型 | 是否必需 | 缺省值 | 描述               | 取值范围                              |
| ----- | -------- | -------- | ------ | ------------------ | ------------------------------------- |
| model | string   | false    | 0      | 是否包含已冻结余额 | 1 to include frozen balance, 0 to not |

<aside class="notice">如果同时订阅可用和总余额，需要为 0 和 1 各开启一条websocket连接</aside>

> Response

```json
{
  "op": "sub",
  "cid": "40sG903yz80oDFWr",
  "err-code": 0,
  "ts": 1489474081631,
  "topic": "accounts"
}
```

> Update example

```json
{
  "op": "notify",
  "ts": 1522856623232,
  "topic": "accounts",
  "data": {
    "event": "order.place",
    "list": [
      {
        "account-id": 419013,
        "currency": "usdt",
        "type": "trade",
        "balance": "500009195917.4362872650"
      }
    ]
  }
}

```

### 数据更新字段列表

| 字段       | 数据类型 | 描述                                                         |
| ---------- | -------- | ------------------------------------------------------------ |
| event      | string   | 资产变化通知相关事件说明，比如订单创建(order.place) 、订单成交(order.match)、订单成交退款（order.refund)、订单撤销(order.cancel) 、点卡抵扣交易手续费（order.fee-refund)、杠杆账户划转（margin.transfer)、借贷本金（margin.loan)、借贷计息（margin.interest)、归还借贷本金利息(margin.repay)、其他资产变化(other) |
| account-id | integer  | 账户 id                                                      |
| currency   | string   | 币种                                                         |
| type       | string   | 账户类型, 交易子账户（trade),借贷子账户（loan），利息子账户（interest) |
| balance    | string   | 账户余额 (当订阅mode=0时，该余额为可用余额；当订阅mode=1时，该余额为总余额） |

## 订阅订单更新

API Key 权限：读取

订阅账户下的订单更新。

### 主题订阅

`orders.$symbol`

> Subscribe request

```json
{
  "op": "sub",
  "cid": "40sG903yz80oDFWr",
  "topic": "orders.htusdt",
}
```

### 参数

| 参数   | 数据类型 | 是否必需 | 缺省值 | 描述     | 取值范围                                            |
| ------ | -------- | -------- | ------ | -------- | --------------------------------------------------- |
| symbol | string   | true     | NA     | 交易代码 | All supported trading symbols, e.g. btcusdt, bccbtc |

> Response

```json
{
  "op": "sub",
  "cid": "40sG903yz80oDFWr",
  "err-code": 0,
  "ts": 1489474081631,
  "topic": "orders.htusdt"
}
```

> Update example

```json
{
  "op": "notify",
  "topic": "orders.htusdt",
  "ts": 1522856623232,
  "data": {
    "seq-id": 94984,
    "order-id": 2039498445,
    "symbol": "htusdt",
    "account-id": 100077,
    "order-amount": "5000.000000000000000000",
    "order-price": "1.662100000000000000",
    "created-at": 1522858623622,
    "order-type": "buy-limit",
    "order-source": "api",
    "order-state": "filled",
    "role": "taker|maker",
    "price": "1.662100000000000000",
    "filled-amount": "5000.000000000000000000",
    "unfilled-amount": "0.000000000000000000",
    "filled-cash-amount": "8301.357280000000000000",
    "filled-fees": "8.000000000000000000"
  }
}
```

### 数据更新字段列表

| 字段               | 数据类型 | 描述                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| seq-id             | integer  | 流水号(不连续)                                               |
| order-id           | integer  | 订单 id                                                      |
| symbol             | string   | 交易对                                                       |
| account-id         | string   | 账户 id                                                      |
| order-amount       | string   | 订单数量                                                     |
| order-price        | string   | 订单价格                                                     |
| created-at         | int      | 订单创建时间 (UNIX epoch time in millisecond)                |
| order-type         | string   | 订单类型, 有效取值: buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc, buy-limit-maker, sell-limit-maker |
| order-source       | string   | 订单来源, 有效取值: sys, web, api, app                       |
| order-state        | string   | 订单状态, 有效取值: submitted, partical-filled, cancelling, filled, canceled, partial-canceled |
| role               | string   | 成交角色: taker or maker                                     |
| price              | string   | 成交价格                                                     |
| filled-amount      | string   | 单次成交数量                                                 |
| filled-cash-amount | string   | 单次未成交数量                                               |
| filled-fees        | string   | 单次成交金额                                                 |
| unfilled-amount    | string   | 单次成交手续费（买入为币，卖出为钱）                         |

## 订阅订单更新 (NEW)

API Key 权限：读取

相比现有用户订单更新推送主题“orders.$symbol”， 新增主题“orders.$symbol.update”拥有更低的数据延迟以及更准确的消息顺序。建议API用户订阅此新主题接收订单更新推送，以替代现有订阅主题 “orders.$symbol”。（现有订阅主题 “orders.$symbol”仍将在Websocket API服务中被保留直至另行通知。）

### 主题订阅

`orders.$symbol.update`

> Subscribe request

```json
{
  "op": "sub",
  "cid": "40sG903yz80oDFWr",
  "topic": "orders.htusdt.update"
}
```

### 参数

| 参数   | 数据类型 | 是否必需 | 缺省值 | 描述     | 取值范围                                            |
| ------ | -------- | -------- | ------ | -------- | --------------------------------------------------- |
| symbol | string   | true     | NA     | 交易代码 | All supported trading symbols, e.g. btcusdt, bccbtc |



> Response

```json
{
  "op": "sub",
  "ts": 1489474081631,
  "topic": "orders.htusdt.update",
  "err-code": 0,
  "cid": "40sG903yz80oDFWr"  
}
```

> Update example

```json
{
  "op": "notify",
  "ts": 1522856623232,
  "topic": "orders.htusdt.update",  
  "data": {
    "unfilled-amount": "0.000000000000000000",
    "filled-amount": "5000.000000000000000000",
    "price": "1.662100000000000000",
    "order-id": 2039498445,
    "symbol": "htusdt",
    "match-id": 94984,
    "filled-cash-amount": "8301.357280000000000000",
    "role": "taker|maker",
    "order-state": "filled"
  }
}
```

### 数据更新字段列表

| Field              | Data Type | Description                                                  |
| ------------------ | --------- | ------------------------------------------------------------ |
| match-id           | integer   | 最近撮合编号（当order-state = submitted, canceled, partial-canceled时，match-id 为消息序列号；当order-state = filled, partial-filled 时，match-id 为最近撮合编号。） |
| order-id           | integer   | 订单编号                                                     |
| symbol             | string    | 交易代码                                                     |
| order-state        | string    | 订单状态, 有效取值: submitted, partical-filled, cancelling, filled, canceled, partial-canceled |
| role               | string    | 最近成交角色（当order-state = submitted, canceled, partial-canceled时，role 为缺省值taker；当order-state = filled, partial-filled 时，role 取值为taker 或maker。） |
| price              | string    | 最新价（当order-state = submitted 时，price 为订单价格；当order-state = canceled, partial-canceled 时，price 为零；当order-state = filled, partial-filled 时，price 为最近成交价。当role = taker，且该订单同时与多张对手方订单撮合时，price 为多笔成交均价。） |
| filled-amount      | string    | 最近成交数量                                                 |
| filled-cash-amount | string    | 最近成交数额                                                 |
| unfilled-amount    | string    | 最近未成交数量（当order-state = submitted 时，unfilled-amount 为原始订单量；当order-state = canceled OR partial-canceled 时，unfilled-amount 为未成交数量；当order-state = filled 时，如果 order-type = buy-market，unfilled-amount 可能为一极小值；如果order-type <> buy-market 时，unfilled-amount 为零；当order-state = partial-filled AND role = taker 时，unfilled-amount 为未成交数量；当order-state = partial-filled AND role = maker 时，unfilled-amount 为零。（后续将支持此场景下的未成交量，时间另行通知。）） |

## 请求用户资产数据

API Key 权限：读取

查询当前用户的所有账户余额数据。

### 数据请求

`accounts.list`

> Query request

```json
{
  "op": "req",
  "cid": "40sG903yz80oDFWr",
  "topic": "accounts.list",
}
```

成功建立和 WebSocket API 的连接之后，向 Server发送如下格式的数据来查询账户数据：

| 参数  | 数据类型 | 描述                         |
| ----- | -------- | ---------------------------- |
| op    | string   | 必填；操作名称，固定值为 req |
| cid   | string   | 选填；Client 请求唯一 ID     |
| topic | string   | 必填；固定值为accounts.list  |

### 返回

> Successful

```json
    {
      "op": "req",
      "topic": "accounts.list",
      "cid": "40sG903yz80oDFWr",
      "err-code": 0,
      "ts": 1489474082831,
      "data": [
        {
          "id": 419013,
          "type": "spot",
          "state": "working",
          "list": [
            {
              "currency": "usdt",
              "type": "trade",
              "balance": "500009195917.4362872650"
            },
            {
              "currency": "usdt",
              "type": "frozen",
              "balance": "9786.6783000000"
            }
          ]
        },
        {
          "id": 35535,
          "type": "point",
          "state": "working",
          "list": [
            {
              "currency": "eth",
              "type": "trade",
              "balance": "499999894616.1302471000"
            },
            {
              "currency": "eth",
              "type": "frozen",
              "balance": "9786.6783000000"
            }
          ]
        }
      ]
    }
```

> Failed

```javascript
    {
      "op": "req",
      "topic": "foo.bar",
      "cid": "40sG903yz80oDFWr",
      "err-code": 12001, //Response codes,0  represent success;others value  is error,the list of Response codes is in appendix
      "err-msg": "detail of error message",
      "ts": 1489474081631
    }

```

| 字段     | 数据类型 | 描述       |
| -------- | -------- | ---------- |
| id       | long     | 账户ID     |
| type     | string   | 账户类型   |
| state    | string   | 账户状态   |
| list     | string   | 账户列表   |
| currency | string   | 子账户币种 |
| type     | string   | 子账户类型 |
| balance  | string   | 子账户余额 |

## 请求当前及历史订单

API Key 权限：读取

根据设定条件查询当前委托、历史委托。

### 数据请求

`order.list`

> Query request

```json
{
  "op": "req",
  "topic": "orders.list",
  "cid": "40sG903yz80oDFWr",
  "symbol": "htusdt",
  "states": "submitted,partial-filled"
}
```

### 参数

| 参数       | 数据类型 | 是否必需 | 缺省值 | 描述                             | 取值范围                                                     |
| ---------- | -------- | -------- | ------ | -------------------------------- | ------------------------------------------------------------ |
| account-id | int      | true     | NA     | 账户 id                          | NA                                                           |
| symbol     | string   | true     | NA     | 交易对                           | All supported trading symbols, e.g. btcusdt, bccbtc          |
| types      | string   | false    | NA     | 查询的订单类型组合，使用','分割  | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |
| states     | string   | false    | NA     | 查询的订单状态组合，使用','分割  | submitted, partial-filled, partial-canceled, filled, canceled |
| start-date | string   | false    | -61d   | 查询开始日期, 日期格式yyyy-mm-dd | NA                                                           |
| end-date   | string   | false    | today  | 查询结束日期, 日期格式yyyy-mm-dd | NA                                                           |
| from       | string   | false    | NA     | 查询起始 ID                      | NA                                                           |
| direct     | string   | false    | next   | 查询方向                         | next, prev                                                   |
| size       | int      | false    | 100    | 查询记录大小                     | [1, 100]                                                     |

### 数据更新字段列表

> Successful

```json
{
  "op": "req",
  "topic": "orders.list",
  "cid": "40sG903yz80oDFWr",
  "err-code": 0,
  "ts": 1522856623232,
  "data": [
    {
      "id": 2039498445,
      "symbol": "htusdt",
      "account-id": 100077,
      "amount": "5000.000000000000000000",
      "price": "1.662100000000000000",
      "created-at": 1522858623622,
      "type": "buy-limit",
      "filled-amount": "5000.000000000000000000",
      "filled-cash-amount": "8301.357280000000000000",
      "filled-fees": "8.000000000000000000",
      "finished-at": 1522858624796,
      "source": "api",
      "state": "filled",
      "canceled-at": 0
    }
  ]
}
```

| 字段               | 数据类型 | 描述                         |
| ------------------ | -------- | ---------------------------- |
| id                 | long     | 订单ID                       |
| symbol             | string   | 交易对                       |
| account-id         | long     | 账户ID                       |
| amount             | string   | 订单数量                     |
| price              | string   | 订单价格                     |
| created-at         | long     | 订单创建时间                 |
| type               | string   | 订单类型，请参考订单类型说明 |
| filled-amount      | string   | 已成交数量                   |
| filled-cash-amount | string   | 已成交总金额                 |
| filled-fees        | string   | 已成交手续费                 |
| finished-at        | string   | 最后成交时间                 |
| source             | string   | 订单来源，请参考订单来源说明 |
| state              | string   | 订单状态，请参考订单状态说明 |
| cancel-at          | long     | 撤单时间                     |

## 以订单编号请求订单

API Key 权限：读取

以订单编号请求订单数据

### 数据请求

`order.detail`

> Query request

```json
{
  "op": "req",
  "topic": "orders.detail",
  "order-id": "2039498445",
  "cid": "40sG903yz80oDFWr"
}
```

### 参数

| 参数     | 是否必需 | 数据类型 | 描述                   | 缺省值 | 取值范围 |
| -------- | -------- | -------- | ---------------------- | ------ | -------- |
| op       | true     | string   | 操作名称，固定值为 req |        |          |
| cid      | true     | string   | Client 请求唯一 ID     |        |          |
| topic    | false    | string   | 固定值为 orders.detail |        |          |
| order-id | true     | string   | 订单ID                 |        |          |

### 返回

> Successful

```json
{
  "op": "req",
  "topic": "orders.detail",
  "cid": "40sG903yz80oDFWr",
  "err-code": 0,
  "ts": 1522856623232,
  "data": {
    "id": 2039498445,
    "symbol": "htusdt",
    "account-id": 100077,
    "amount": "5000.000000000000000000",
    "price": "1.662100000000000000",
    "created-at": 1522858623622,
    "type": "buy-limit",
    "filled-amount": "5000.000000000000000000",
    "filled-cash-amount": "8301.357280000000000000",
    "filled-fees": "8.000000000000000000",
    "finished-at": 1522858624796,
    "source": "api",
    "state": "filled",
    "canceled-at": 0
  }
}
```

| 字段               | 数据类型 | 描述                         |
| ------------------ | -------- | ---------------------------- |
| id                 | long     | 订单ID                       |
| symbol             | string   | 交易对                       |
| account-id         | long     | 账户ID                       |
| amount             | string   | 订单数量                     |
| price              | string   | 订单价格                     |
| created-at         | long     | 订单创建时间                 |
| type               | string   | 订单类型，请参考订单类型说明 |
| filled-amount      | string   | 已成交数量                   |
| filled-cash-amount | string   | 已成交总金额                 |
| filled-fees        | string   | 已成交手续费                 |
| finished-at        | string   | 最后成交时间                 |
| source             | string   | 订单来源，请参考订单来源说明 |
| state              | string   | 订单状态，请参考订单状态说明 |
| cancel-at          | long     | 撤单时间                     |
