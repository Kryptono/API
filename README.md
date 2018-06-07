# Public Rest API for Kryptono Exchange (2018-06-07)
# General API Information
* The base endpoint is: **https://kryptono.exchange/k/api/**
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `404` return codes is used for data not found.
* HTTP `400` return codes is used for invalid data.
* HTTP `406` return codes is used for not acceptable data.
* HTTP `500` return codes is used for invalid format request or wrong from server's side.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "error": -1212,
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
* MARKET (currently not supported)
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
    "email": "ahihi2@mailinator.com",
    "phone": null,
    "enable_google_2fa": true,
    "status": "offline",
    "create_at": 1524567654822,
    "nick_name": "Ahihi 2",
    "chat_id": "29371524567654821@kryptono.exchange",
    "chat_password": "VMyBENGYrp",
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
        "nick_name": "Ahihi 2",
        "email": "ahihi2@mailinator.com",
        "ip_address": "14.161.20.103",
        "login_at": 1528199468073,
        "os_name": "Mac OS X",
        "browser_name": "Chrome",
        "country": "Vietnam",
        "city": "Ho Chi Minh City",
        "sentEmail": true
    },
    "commission_status": true,
    "account_kyc": null,
    "kyc_reject_infos": [],
    "allow_order": 1,
    "disable_withdraw": 0,
    "referral_id": "XWSMQ0",
    "favorite_pairs": [
        "KNOW_ETH"
    ],
    "chat_server": "wss://chat.kryptono.exchange:5280/ws",
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

### Get Market Price (Updated every 15 seconds)
```
GET /v1/market_price?symbol=KNOW_BTC
```
**Request Params:**
Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | Default is all of symbols |

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

