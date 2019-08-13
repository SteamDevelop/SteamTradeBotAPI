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
The robot's login and logout commands are asynchronous operations, it usually takes some time to complete the operation.
So after sending the above interactive command, please wait 10 seconds before call the api of robot status inquiry.


### Transaction API
```
POST http://bot.5uskin.com/api/steambot/trade/
```
API parameter inclusions:
```json
{
  "steamid": "705xxxxx", // Steamid of robot that initiate a transaction
  "my_items": [
    // List of robot's skins to be traded, if robot is the traded one, leave a blank array
    {
      "appid": 570, // skin's appid
      "contextid": "2", // skin's contextid
      "assetid": "15680233392" // skin's assetid
    }
  ],
  "their_items": [
    // List of transaction party's skins to be traded, if do not request the party's skins, leave a blank array
    {
      "appid": 570, // skin's appid
      "contextid": "2", // skin's contextid
      "assetid": "15680235255" // skin's assetid
    }
  ], 
  "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx", // The transaction party's steam quote link
  "cancel_time": 600000, // The order cancellation time in milliseconds; after the quote is initiated, the robot automatically cancels the quote after the duration
  "message": "my trade offer", // Quote description information attached to the steam quotation,
  "uid": "xxxxxxxxxxxxx", // The only uuid of the quote, after the quote is successfully launched, this uuid can be use to query the quote status.
}
```

API response
```json
{
  "code": 0,
  "body": {
    "uid": "xxxxxxx"
  },
  "message": "succeed"
}
```

### Steam transaction status inquiry
```
GET http://bot.5uskin.com/api/steambot/trade/
```
API parameters:
Parameter name | Parameter value | Required | Description
----- | ----- | ----- | -----
uid | xxxxxxxx | Yes | The uuid to be queried

##### Robot's quotation status
Status value | Description
----- | -----
0 | The robot is queued for processing
1 | Successful transaction
2 | Failed quotation, cancellation of quote is also a quote failure
11 | Robot is submitting the quotation
12 | The quotation was successfully launched and the steam quotation number has been obtained from steam.
13 | The quotation of the robot is completed, the quotation is activated, and the customer is waiting for the quotation.
14 | Quotation is pending

##### The quotation status on Steam
Status value | Status | Remarks | Scene supplement
----- | ----- | ---- | ----
1 | Invalid quote | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2 | 
2 | Quote activation, waiting to accept quotes | In this state, the robot will update the robot's quotation status to 13 |
3 | Quote has been accepted | In this state, the robot is classified as a successful quote, and the robot's quotation status is updated to 2 |
4 | Quote is bargained | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2; |  In this status, the original quotation will be cancelled, a new quotation will be generated according to the customer's counter-offer (different from the original quotation number), and the robot will ignore the new quotation; for example, the robot give a quotation #123 requests the user A's Item1, item2, after receiving the quotation #123, user A click on the quotation, operate the counter-offer, remove the item1 in #123, then the #123 quotation will be canceled and a new quotation named #124 will be generated. In the quotation #124, only request item2 is requested; the robot will ignore the quotation #124.
5 | Quote has expired | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2 |
6 | Quote has been cancelled | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2 |
7 | Quote is rejected | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2 |
8 | Quotation item is invalid | In this state, the robot is classified as a quote failure, and the robot's quotation status is updated to 2 | For example, quotation #123 requests user A's inventory item1, item2, user A has not processed quotation #123, user A and user B perform another quotation behavior, and user A's item1 is traded to user B, which The quotation #123 that is active will fail because the quotation item is invalid.
11 | Quotation is pending |In this state, the robot's quotation status is updated to 14 | Since the pending transaction needs to wait for 15 days and there are too many variables, we recommend that the suspension be classified as a quotation failure.
API response
```json
{
  "code": 0,
  "body": {
    "uid": "xxxxxxxxxxxxx",
    "their_items": [
      {
        "appid": 570, // skin's appid
        "contextid": "2", // skin's contextid
        "assetid": "15680235255", // skin's assetid
        "new_assetid": "15680235256" // After the successful transaction, the new assetid of the skin
      }
    ],
    "my_items": [
      {
        "appid": 570, // skin's appid
        "contextid": "2", // skin's contextid
        "assetid": "15680233392" // skin's assetid
      }
    ],
    "updatedAt": "2019-04-15T10:02:38.495Z",
    "createdAt": "2019-04-15T10:02:07.693Z",
    "trade_no": "3538xxxxxx", // The steam quotation number obtained after the robot quotation is successfully launched
    "exchanged": true,
    "state": 1, // Robot's quotation status；0 means the robot is queued for processing，1 means successful transaction，2 means failed quotation, cancellation of quote is also a quote failure，11 means robot is submitting the quotation，12 means the quotation was successfully launched and the steam quotation number has been obtained from steam，13 means The quotation of the robot is completed, the quotation is activated, and the customer is waiting for the quotation；
    "offer_state": 3, // The quotation status in Steam, the status information participates in the above quotation Steam Status Form
    "cancel_time": 600000,
    "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx",
    "message": "my trade offer",
    "steamid": "765xxxxxxxxxx"
  },
  "message": "succeed"
}
```

### Cancel the transaction
```
POST http://bot.5uskin.com/api/steambot/canceltrade/
```
API request parameter inclusions
```json
{
  "uid": "xxxxxxx" // The uuid of cancelled quotation
}
```

API response
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

### Quotation status callback
```
POST http://www.yoursite.com/api/tradeoffer/callback/
```
The quotation status callback is initiated by the robot, the callback status change information is similar to the callback address
Callback inclusion
```json
{
  "type": "tradeoffer",  // Callback type，Default tradeoffer refers to the quotation
  "action": "update", // Message type, default update, refers to the quotation status update
  "body": {
    "uid": "xxxxxxxxxxxxx",
    "their_items": [
      {
        "appid": 570, // skin's appid
        "contextid": "2", // skin's contextid
        "assetid": "15680235255", // skin's assetid
        "new_assetid": "15680235256" // After the successful transaction, the new assetid of the skin
      }
    ],
    "my_items": [
      {
        "appid": 570, // skin's appid
        "contextid": "2", // skin's contextid
        "assetid": "15680233392" // skin's assetid
      }
    ],
    "updatedAt": "2019-04-15T10:02:38.495Z",
    "createdAt": "2019-04-15T10:02:07.693Z",
    "trade_no": "3538xxxxxx", // The steam quotation number obtained after the robot quotation is successfully launched
    "exchanged": true,
    "state": 1, // Robot's quotation status；0 means the robot is queued for processing，1 means successful transaction，2 means failed quotation, cancellation of quote is also a quote failure，11 means robot is submitting the quotation，12 means the quotation was successfully launched and the steam quotation number has been obtained from steam，13 means The quotation of the robot is completed, the quotation is activated, and the customer is waiting for the quotation；
    "offer_state": 3, // The quotation status in Steam, the status information participates in the above quotation Steam Status Form
    "cancel_time": 600000,
    "tradeurl": "https://steamcommunity.com/tradeoffer/new/?partner=81xxxxxx&token=tOxxxxxx",
    "message": "my trade offer",
    "steamid": "765xxxxxxxxxx"
  }
}
```

Callback response format
```json
{
  "code": 0, // Code 0 means the callback is normal, and non-0 means the callback is abnormal.
  "body": {},
  "message": "ok"
}
```
Each time the quotation status changes, the robot will send a callback to the above address;
After you receive the callback, should return the response packet as above. Code 0 means the callback is normal, and non-0 means the callback is abnormal.
If the callback response is abnormal, the robot will resend it once after 1 minute, 10 minutes, and 1 hour.
If the callback response is still abnormal after 1 hour, the robot will not continue to resend;
