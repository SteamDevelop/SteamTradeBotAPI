## Steam Skin Trading bot API Agreement
>The right to interpret this agreement belongs to [HaiDaoHai Technology](https://www.haidaoteam.com)

### Contact us
Email:Business@haidaoteam.com
QQ:304926518
Wechat:steamkaifa


### Public parameter description
The Steam Trading bot API use the standard HTTP protocol (the official environment uses HTTPS), the request and response packets are in json format; the returned response packet base format is:
```json
{
  "code": 0,  // Error code, 0 means success, non-0 means abnormal
  "body": {}, // Specific return data
  "message": "Succeed" // Error description
}
```


### Steam robot status inquiry
```
GET http://bot.5uskin.com/api/steambot/list/
```

API return data structure
```json
{
  "code": 0,
  "body": {
    "bots": [
      {
        "account": "xxxxx"
        "steamid": "705xxxxxxxxxx", // Botsteamid
        "enable": true,  // If the bot online switch is open or not
        "online": true // If the bot is online or not
      }
    ]
  },
  "message": "Succeed"
}
```

### Control Steam robot on line and off line
```
POST http://bot.5uskin.com/api/steambot/action/
```
API request parameter inclusions
```json
{
  "account": "xxxxxxx", // Current Steam account used by bot
  "action": "enable" // Operation instructions of the bot; currently supports two instructions, "enable": login, "disable": logout
}
```

API response
```json
{
  "code": 0,
  "body": {
    "account": "xxxxxx",
    "action": "enable"
  },
  "message": "xxxxx"
}
```
机器人上下线指令为异步操作，一般需要一些时间完成登录/注销的上下线操作；
所以发送完成上述交互指令后，请过10秒钟后再调用查询机器人状态的api查询机器人的最新状态


### 发起交易接口
```
POST http://bot.5uskin.com/api/steambot/trade/
```
接口参数包体:
```json
{
  "steamid": "705xxxxx", // 发起交易的机器人steamid
  "my_items": [
    // 要交易的我方饰品列表，如果交易我方，则留空数组
    {
      "appid": 570, // 饰品的appid
      "contextid": "2", // 饰品的contextid
      "assetid": "15680233392" // 饰品的assetid
    }
  ],
  "their_items": [
    // 要交易的对方饰品列表，如果不请求对方饰品，则留空数组
    {
      "appid": 570, // 饰品的appid
      "contextid": "2", // 饰品的contextid
      "assetid": "15680235255" // 饰品的assetid
    }
  ], 
  "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx", // 对方的steam报价链接
  "cancel_time": 600000, // 订单取消时长，单位毫秒；报价发起后超过该时长后，机器人自动取消报价
  "message": "my trade offer", // steam报价单附带的报价描述信息,
  "uid": "xxxxxxxxxxxxx", // 该笔报价的唯一uuid，报价发起成功后，可以用该uuid查询报价状态
}
```

接口响应
```json
{
  "code": 0,
  "body": {
    "uid": "xxxxxxx"
  },
  "message": "succeed"
}
```

### 查询交易状态
```
GET http://bot.5uskin.com/api/steambot/trade/
```
接口参数：
参数名 | 参数值 | 必需 | 说明
----- | ----- | ----- | -----
uid | xxxxxxxx | 是 | 要查询的报价uuid

##### 机器人报价单状态
状态值 | 描述
----- | -----
0 | 机器人正在排队处理中
1 | 交易成功
2 | 报价已失败(取消报价也是报价失败
11 | 机器人提交报价中
12 | 报价发起成功，已经从steam获取到steam报价单号
13 | 机器人报价发起完毕，报价单激活中，等待客户接收报价
14 | 报价单暂挂

##### 报价单在Steam平台状态
状态值 | 状态 | 备注 | 场景补充
----- | ----- | ---- | ----
1 | 无效报价 | 该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2 | 
2 | 报价激活中，等待接受报价 | 该状态下，机器人即会更新机器人的报价单状态为 13 |
3 | 报价已接受 |  该状态下，机器人归类为报价成功，会更新机器人的报价单状态为1 |
4 | 报价被还价 |  该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2；|  该场景下，原报价单会被取消，根据客户的还价会产生一个新的报价单（与原报价单单号不同），机器人会忽略新的报价单；例如 机器人报价单#123请求用户A的 item1,item2物品，用户A在接收到报价单#123后，点击报价单，操作还价，去掉了#123中的item1，这时#123报价单会被取消，并产生一笔新的报价单 #124，该报价单中为只请求交易item2；机器人将忽略#124报价单
5 | 报价已过期 | 该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2 |
6 | 报价已取消 | 该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2 |
7 | 报价被拒绝 | 该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2 |
8 | 报价单物品无效 | 该状态下，机器人归类为报价失败，会更新机器人的报价单状态为2 | 例如：报价单#123请求了用户A的库存item1，item2，用户A暂未处理报价单#123，用户A与用户B进行另外一笔报价行为，将用户A的item1交易给了用户B，这时处于激活状态的报价单#123即会因为报价单物品无效而失败
11 | 报价单交易暂挂 | 该状态下，机器人会更新机器人的报价单状态为14 | 由于暂挂交易需要等待15天，可能带来场景情况太多，我们建议是将暂挂都归类为报价交易失败

接口响应
```json
{
  "code": 0,
  "body": {
    "uid": "xxxxxxxxxxxxx",
    "their_items": [
      {
        "appid": 570, // 饰品的appid
        "contextid": "2", // 饰品的contextid
        "assetid": "15680235255", // 饰品的assetid
        "new_assetid": "15680235256" // 交易成功后，该饰品的新assetid
      }
    ],
    "my_items": [
      {
        "appid": 570, // 饰品的appid
        "contextid": "2", // 饰品的contextid
        "assetid": "15680233392" // 饰品的assetid
      }
    ],
    "updatedAt": "2019-04-15T10:02:38.495Z",
    "createdAt": "2019-04-15T10:02:07.693Z",
    "trade_no": "3538xxxxxx", // 机器人报价发起成功后得到的steam报价单号
    "exchanged": true,
    "state": 1, // 机器人报价单状态；0表示机器人正在排队处理中，1表示报价交易成功，2表示报价已失败(取消报价也是报价失败)，11表示机器人提交报价中，12表示报价发起成功，已经从steam获取到steam报价单号，13表示机器人报价发起完毕，报价单激活中，等待客户接收报价；
    "offer_state": 3, // 报价单在steam平台状态，状态信息参加上面的报价单Steam平台状态表格
    "cancel_time": 600000,
    "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx",
    "message": "my trade offer",
    "steamid": "765xxxxxxxxxx"
  },
  "message": "succeed"
}
```

### 取消交易
```
POST http://bot.5uskin.com/api/steambot/canceltrade/
```
接口请求参数包体
```json
{
  "uid": "xxxxxxx" // 要取消的报价单uuid
}
```

接口响应
```json
{
  "code": 0,
  "body": {
    "uid": "xxxxxx",
    "trade_no": "2342343255"
  },
  "message": "xxxxx"
}
```

### 报价状态回调
```
POST http://www.yoursite.com/api/tradeoffer/callback/
```
报价状态回调由机器人发起，向类似上面这样的你方回调地址回调订单状态变化信息

回调包体
```json
{
  "type": "tradeoffer",  // 回调类型，默认 tradeoffer，指代报价单
  "action": "update", // 消息类型，默认 update，指代报价单状态更新
  "body": {
    "uid": "xxxxxxxxxxxxx",
    "their_items": [
      {
        "appid": 570, // 饰品的appid
        "contextid": "2", // 饰品的contextid
        "assetid": "15680235255", // 饰品的assetid
        "new_assetid": "15680235256" // 交易成功后，该饰品的新assetid
      }
    ],
    "my_items": [
      {
        "appid": 570, // 饰品的appid
        "contextid": "2", // 饰品的contextid
        "assetid": "15680233392" // 饰品的assetid
      }
    ],
    "updatedAt": "2019-04-15T10:02:38.495Z",
    "createdAt": "2019-04-15T10:02:07.693Z",
    "trade_no": "3538xxxxxx", // 机器人报价发起成功后得到的steam报价单号
    "exchanged": true,
    "state": 1, // 报价单状态；0表示机器人正在排队处理中，1表示报价交易成功，2表示报价已失败(取消报价也是报价失败)，11表示机器人提交报价中，12表示报价发起成功，已经从steam获取到steam报价单号，13表示机器人报价发起完毕，报价单激活中，等待客户接收报价；
    "offer_state": 3, // 报价单在steam平台状态，状态信息参加上面的报价单Steam平台状态表格
    "cancel_time": 600000,
    "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx",
    "message": "my trade offer",
    "steamid": "765xxxxxxxxxx"
  }
}
```

回调响应格式
```json
{
  "code": 0, // code为0 表示接收回调正常，非0表示接受异常
  "body": {},
  "message": "ok"
}
```
每次报价单status变化时，机器人就会向以上地址发送一次回调；
你方收到回调后，应该返回如上格式响应包，code为0 表示接收回调正常，非0表示接受异常；
如果回调响应异常，则机器人会再间隔1分钟、10分钟、1小时分别重发一次；
如果1小时后仍然返回异常，则机器人不再继续重发；
