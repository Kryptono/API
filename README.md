# Public Rest API for Kryptono Exchange (June 18, 2018)
## Update History
### June 18, 2018
* Support Market type for create order
* Provide api/websocket for market information, include order books and transaction history: [Get Market Information](#get-market-information)


# General API Information
* The base endpoint is: **https://p.kryptono.exchange/k/api/**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in milliseconds.
* The limit for number of requests per minute is 100
* HTTP `404` return codes is used for data not found.
* HTTP `400` return codes is used for invalid data.
* HTTP `406` return codes is used for not acceptable data or the api key already reach maximum number of requests per minute.
* HTTP `500` return codes is used for invalid format request or wrong from server's side.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "error": "400xxx",
  "error_description": "Invalid param."
}
```

* For `GET` endpoints, the parameters may be sent as a `request param` or not.
* For `POST`, and `DELETE` endpoints, the parameters must be sent as a `request body` with content type
  `application/json`.
* Parameters may be sent in any order.


# Public API Endpoints
## ENUM definitions
**Order status:**

* OPEN
* FILLED
* PARTIAL_FILL
* CANCELED
* CANCELING

**Order types:**

* LIMIT
* MARKET
* STOP_LOSS
* TAKE_PROFIT

**Order side:**

* BUY
* SELL

## Header requirements
Name | Value
--------------- | ---------------
Content-Type | application/json
X-Requested-With | XMLHttpRequest
Authorization | Your_api_key

## General endpoints
### Test connectivity
```
GET /v1/ping
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
```javascript
{}
```

### Get Account's detail
```
GET /v1/account/details
```

**Request Params:**
NONE

**Response:**
```javascript
{
    "account_id": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
    "email": "email@email.com",
    "phone": null,
    "enable_google_2fa": true,
    "status": "offline",
    "create_at": 1524567654822,
    "nick_name": "Nickname 1",
    "chat_id": "xxxx@kryptono.exchange",
    "chat_password": "xxxxxxxxxxx",
    "banks": [],
    "country": "US",
    "language": "en",
    "kyc_status": null,
    "kyc_level": "level1",
    "last_login_history": {
        "id": {
            "timestamp": 1528199468,
            "machineIdentifier": 8990639,
            "processIdentifier": 20156,
            "counter": 7772354,
            "time": 1528199468000,
            "date": 1528199468000,
            "timeSecond": 1528199468
        },
        "account_id": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
        "nick_name": "Nickname 1",
        "email": "email@email.com",
        "ip_address": "xxx.xxx.xxx.xxx",
        "login_at": 1528199468073,
        "os_name": "Mac OS X",
        "browser_name": "Chrome",
        "country": "Country",
        "city": "City",
        "sentEmail": true
    },
    "commission_status": true,
    "account_kyc": null,
    "kyc_reject_infos": [],
    "allow_order": 1,
    "disable_withdraw": 0,
    "referral_id": "XXXXXX",
    "favorite_pairs": [
        "KNOW_ETH"
    ],
    "chat_server": "wss://xxx.kryptono.exchange:xxxx/ws",
    "exchange_fee": {
        "standard_fee": "0.1",
        "know_fee": "0.05"
    }
}
```

### Get Balance list
```
GET /v1/account/balances
```
**Request Params:**
NONE

**Response:**
```javascript
[
    {
        "currency_code": "BTC",
        "address": "2MxctvXExQofAVqakPfBjKqVipfwTqwyQyF",
        "total": "1000.00275",
        "available": "994.5022",
        "in_order": "5.50055"
    }
]
```

### Get Market Price (Updated every 5 seconds)
```
GET /v1/market_price?symbol=KNOW_BTC
```
**Request Params:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | Default is all of symbols

**Response:**
```javascript
[
    {
        "symbol": "KNOW_BTC",
        "price": "0.00010677",
        "updated_time": 1528347586009
    }
]
```

### Get Exchange Information
```
GET /v1/exchange_info
```
**Request Params:**
NONE

**Response:**
```javascript
{
    "base_currencies": [
        {
            "currency_code": "BTC",
            "minimum_total_order": "0.001"
        }
    ],
    "coins": [
        {
            "currency_code": "KNOW",
            "name": "Know",
            "minimum_order_amount": "1"
        }
    ],
    "symbols": [
        {
            "symbol": "KNOW_BTC",
            "amount_limit_decimal": 0,
            "price_limit_decimal": 8
        }
    ]
}
```

### Get Market Information
#### Transaction History
##### API
```
GET https://engines.kryptono.exchange/api/v1/ht?symbol=<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET https://engines.kryptono.exchange/api/v1/ht?symbol=KNOW_BTC
```
```
{
    "symbol":"KNOW_BTC",
    "limit":100,
    "history":[
        {
            "id":139638,
            "price":"0.00001723",
            "qty":"81.00000000",
            "isBuyerMaker":false,
            "time":1529262196270
        }
    ],
    "time":1529298130192
}
```

##### Websocket
```
GET wss://engines.kryptono.exchange/ws/v1/ht/<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET wss://engines.kryptono.exchange/ws/v1/ht/KNOW_USDT
```
```
{
    "c" : 1529298630675,
    "e" : "history_trade",
    "k" : 146851,
    "m" : false,
    "p" : "0.10400000",
    "q" : "107.00000000",
    "s" : "KNOW_USDT",
    "t" : 1529298630370
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
c | LONG | Event time |
e | STRING | Event type |
k | STRING | ID |
m | BOOLEAN | Is order made by buyer |
p | STRING | Price |
q | STRING | Quantity |
s | STRING | Symbol |
t | STRING | Trade time |

#### Order Book Data
##### API
```
GET https://engines.kryptono.exchange/api/v1/dp?symbol=<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET https://engines.kryptono.exchange/api/v1/dp?symbol=KNOW_BTC
```
```
{
    "symbol" : "KNOW_BTC",
    "limit" : 100,
    "asks" : [
        [
            "0.00001850",   // price
            "69.00000000"   // size
        ]
    ],
    "bids" : [
        [
            "0.00001651",       // price
            "11186.00000000"    // size
        ]
    ]
    "time" : 1529298130192
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
symbol | STRING | Symbol |
limit | INTEGER | Limit length of list |
asks | Array | List of Sell Order |
bids | Array | List of Buy Order |
time | LONG | Time |

##### Websocket
```
GET wss://engines.kryptono.exchange/ws/v1/dp/<symbol>
```

**Request Param:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |

**Response:**
```
GET wss://engines.kryptono.exchange/ws/v1/dp/KNOW_USDT
```
```
{
    "a" : [
        [
            "0.10400000",   // price
            "245.00000000"    // size
        ]
    ],
    "b" : [
        [
            "0.10400000", 
            "245.00000000"
        ]
    ]
    "e" : "depthUpdate"
    "s" : "KNOW_USDT"
    "t" : 1529299716639
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------
s | STRING | Symbol |
e | STRING | Event type |
a | Array | List of Sell Order |
b | Array | List of Buy Order |
t | LONG | Time |


### Create order
```
POST /v1/order/add_order
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_symbol | STRING | YES | |
order_side | STRING | YES | |
order_price | STRING | YES | |
order_size | STRING | YES | |
stop_price | STRING | NO | Required if type is "stop_loss" or "take_profit" |
type | STRING | YES | |

**Request Body:**
```javascript
{
	"order_symbol" : "KNOW_ETH",
	"order_side" : "BUY",
	"order_price" : "0.0000123",
	"order_size" : "7777",
	"stop_price" : "",
	"type" : "limit"
}
```

**Response:**
```javascript
{
    "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
    "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
    "order_symbol": "KNOW_ETH",
    "order_side": "BUY",
    "status": "open",
    "createTime": 1528277973947,
    "type": "limit",
    "order_price": "0.00001230",
    "order_size": "7777",
    "executed": "0",
    "stop_price": "0.00000000",
    "avg": "0.00001230",
    "total": "0.09565710 ETH"
}
```

### Cancel Order
```
DELETE /v1/order/cancel
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |
order_symbol | STRING | YES | |

**Request Body:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
  "order_symbol" : "KNOW_ETH"
}
```

**Response:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
  "order_symbol" : "KNOW_ETH"
}
```

### Cancel All Order
```
DELETE /v1/order/cancel_all
```

**Request Body:**
```javascript
{}
```

**Response:**
```javascript
200 OK
```

### Get Open Order List
```
POST /v1/order/list/open_order
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default is NO LIMIT |
page | INTEGER | NO | Default is 0 |
from_date | LONG | NO | |
to_date | LONG | NO | |
left | STRING | NO | |
right | STRING | NO | |
order_side | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "order_side" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 10,  // total of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_ETH",
            "order_side": "BUY",
            "status": "open",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "0",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 ETH"
        }
    ]
}
```

### Get Order History
```
POST /v1/order/list/order_history
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default is NO LIMIT |
page | INTEGER | NO | Default is 0 |
from_date | LONG | NO | |
to_date | LONG | NO | |
left | STRING | NO | |
right | STRING | NO | |
order_side | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "order_side" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 10,  // total of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_ETH",
            "order_side": "BUY",
            "status": "filled",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "7777",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 ETH"
        }
    ]
}
```

### Get Trade History
```
POST /v1/order/list/trade_history
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default is NO LIMIT |
page | INTEGER | NO | Default is 0 |
from_date | LONG | NO | |
to_date | LONG | NO | |
left | STRING | NO | |
right | STRING | NO | |
type | STRING | NO | |

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "from_date" : 1528277973947,
  "to_date" : 1528277973947,
  "left" : "KNOW",
  "right" : "ETH",
  "type" : "BUY"
}
```

**Response:**
```javascript
{
    "total": 100,	// total of pages
    "list": [
        {
            "id": {
                "timestamp": 1527488215,
                "machineIdentifier": 8990639,
                "processIdentifier": 29166,
                "counter": 16093657,
                "time": 1527488215000,
                "date": 1527488215000,
                "timeSecond": 1527488215
            },
            "restingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "incomingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "symbol": "GTO_BTC",
            "restingOrderId": "dc8a092e-ab6e-4856-9b95-93eb1d4732ad",
            "incomingOrderId": "6bc57c6d-4552-4ebb-ae9e-5fe75900b300",
            "incomingSide": "BUY",
            "price": "0.00003402",
            "executedQuantity": "500",
            "remainingQuantity": "400",
            "matchingTime": 1527488214898,
            "resting_fee": "0.59885764 KNOW",
            "incoming_fee": "0.66250000 KNOW",
            "total": "0.01701000 BTC"
        }
    ]
}
```

### Get Order's Trade History
```
POST /v1/order/order_trade_detail
```
**Request Body Description:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |

**Request Body:**
```javascript
{
  "order_id" : "dc8a092e-ab6e-4856-9b95-93eb1d4732ad"
}
```

**Response:**
```javascript
{
    "total": -1,
    "list": [
        {
            "id": {
                "timestamp": 1527488215,
                "machineIdentifier": 8990639,
                "processIdentifier": 29166,
                "counter": 16093657,
                "time": 1527488215000,
                "date": 1527488215000,
                "timeSecond": 1527488215
            },
            "restingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "incomingAccountId": "5377f2e2-4b0e-4b15-be17-28092ae0c346",
            "symbol": "GTO_BTC",
            "restingOrderId": "dc8a092e-ab6e-4856-9b95-93eb1d4732ad",
            "incomingOrderId": "6bc57c6d-4552-4ebb-ae9e-5fe75900b300",
            "incomingSide": "BUY",
            "price": "0.00003402",
            "executedQuantity": "500",
            "remainingQuantity": "400",
            "matchingTime": 1527488214898,
            "resting_fee": "0.59885764 KNOW",
            "incoming_fee": "0.66250000 KNOW",
            "total": "0.01701000 BTC"
        }
    ]
}
```

