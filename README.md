# Public Rest API v2 for Kryptono Exchange (July 6, 2018)

## Deprecation Notice

**The Kryptono Exchange API v1 will be deprecated in July 20, 2018**

## Update History
### July 6, 2018
* Publish API v2


# General API Information
* All endpoints return either a JSON object or array.
* Data is returned in **descending** order. Newest first, oldest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `400` return codes is used for invalid data.
* HTTP `401` return codes is used for unauthorized request.
* HTTP `404` return codes is used for data not found.
* HTTP `406` return codes is used for not acceptable data.
* HTTP `429` return code is used when breaking a request rate limit.
* HTTP `500` return codes is used for invalid format request or wrong from server's side.
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "error": "400xxx",
  "error_description": "Invalid param."
}
```

* For `GET` endpoints, the parameters may be sent as a `request param` or not.
* For `POST` and `DELETE` endpoints, the parameters must be sent as a `request body` with content type
  `application/json`.
* Parameters may be sent in any order.


# LIMITS
* The `/api/v2/exchange-info` `rate_limits` array contains objects related to the exchange's `REQUESTS` rate limits.
* A `429` will be returned when either rather limit is violated.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a `429` is recieved, it's your obligation as an API to back off and not spam the API.


# Endpoint security type
* Each endpoint has a security type that determines the how you will interact with it.
* API-keys are passed into the Rest API via the `Authorization` header.
* Signature are passed into the Rest API via the `Signature` header.
* API-keys and secret-keys are **case sensitive.**
* API-keys can be configured to only access certain types of secure endpoints. For example, one API-key could be used for TRADE only, while another API-key can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
MARKET_DATA | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.

* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.


# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `Signature`, to be
  sent in the header.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` or `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.
* An additional parameter, `recvWindow`, may be sent to specific the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**It's recommended to use a small recvWindow of 5000 or less!**


## SIGNED Endpoint Examples
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9
secretKey | R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw=

## Example 1: As a query string
* `GET` `/api/v2/account/details`

Parameter | Value
------------ | ------------
timestamp | 1530532714999
recvWindow | 5000

* **Query string:**
  ```
  timestamp=1530532714999&recvWindow=5000
  ```
* **HMAC SHA256 signature:**
  ```
  [linux]$ echo -n 'timestamp=1530532714999&recvWindow=5000' | openssl dgst -sha256 -hmac 'R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw='
  (stdin)= dda4cb640ddc8ff870058b30b5b9faf618f5e465066fcaa906f0a533afc17106
  ```
* **curl command:**
  ```
  (HMAC SHA256)
  [linux]$ curl -X GET 'https://p.kryptono.exchange/k/api/v2/account/details?timestamp=1530532714999&recvWindow=5000' -H 'Authorization: WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9' -H 'Signature: dda4cb640ddc8ff870058b30b5b9faf618f5e465066fcaa906f0a533afc17106' -H 'X-Requested-With: XMLHttpRequest'
  ```

## Example 2: As a request body
* `POST` `/api/v2/order/test`

Parameter | Value
------------ | ------------
order_symbol | KNOW_BTC
order_side | BUY
order_price | 0.00001234
order_size | 1234
type | limit
timestamp | 1530532714999
recvWindow | 5000

* **Request body as json string**
  ```
  {"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}
  ```
* **HMAC SHA256 signature:**
  ```
  [linux]$ echo -n '{"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}' | openssl dgst -sha256 -hmac 'R0Ml3XQpUvowNe3Su+53q53GVAPh/dYWOGXWIBuPDUw='
  (stdin)= 89c3482ce87fd6cce46a2c4452222a87be1a020f14257da894b55a4366d2a914
  ```
* **curl command:**
  ```
  (HMAC SHA256)
  [linux]$ curl -X POST https://p.kryptono.exchange/k/api/v2/order/test -H 'Authorization: WC9NeS96R2lZZStMYUEzQndRbUFEcm5ESkh2R1Rvckd6c3ExZ2ZKS0svYlBKNGU1RzVpLzFBQjJqc0pyYVdoRzN0U2Y2U0ducUY4RE83VmIrK1lVOTBmQ0tqNW1EcWVOZXFMUFpTN0lnYXM9' -H 'Signature: 89c3482ce87fd6cce46a2c4452222a87be1a020f14257da894b55a4366d2a914' -H 'Content-Type: application/json' -H 'X-Requested-With: XMLHttpRequest' -d '{"order_symbol":"KNOW_BTC","order_side":"BUY","order_price":"0.00001234","order_size":"1234","type":"limit","timestamp":1530532714999,"recvWindow":5000}'
  ```
  
  
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

Name | Value | Note
--------------- | --------------- | ---------------
Content-Type | application/json | 
X-Requested-With | XMLHttpRequest | 
Authorization | Your_api_key | 
Signature | signature | Required for SIGNED Endpoints

## General endpoints
The base endpoint is: https://p.kryptono.exchange/k/

### Test connectivity
```
GET /api/v2/ping
```
Test connectivity to the Rest API.

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "result": true
}
```

### Check server time
```
GET /api/v2/time
```
Test connectivity to the Rest API and get the current server time.

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "server_time": 1530682662257
}
```

### Exchange information
```
GET /api/v2/exchange-info
```
Current exchange trading rules and symbol information

**Weight:** 1

**Parameters:** NONE

**Response:**
```javascript
{
    "timezone": "UTC",
    "server_time": 1530683054384,
    "rate_limits": [
        {
            "type": "REQUESTS",
            "interval": "MINUTE",
            "limit": 1000
        }
    ],
    "base_currencies": [
        {
            "currency_code": "KNOW",
            "minimum_total_order": "100"
        }
    ],
    "coins": [
        {
            "currency_code": "USDT",
            "name": "Tether",
            "minimum_order_amount": "1"
        }
    ],
    "symbols": [
        {
            "symbol": "GTO_ETH",
            "amount_limit_decimal": 0,
            "price_limit_decimal": 8
        }
    ]
}
```
* `base_currencies` array contains all currencies that is in the right side of symbol. Eg: with symbol `KNOW_BTC`, base currency is `BTC`.
* Each base currency has `minimum_order_total` value that defines the minimum total of an order. Eg: An order is created with `symbol=GTO_KNOW, amount=100, price=1.45` then `total=amount*price=145`, it is a valid order because `145 > 100`.
* `coins` array contains all supported currencies in Krytono Exchange.
* Each coin has `minimum_order_amount` value that defines the minimum amount of an order.
* `symbols` array contains all supported symbol in Kryptono Exchange. 
* Each symbol has 2 values `amount_limit_decimal` and `price_limit_decimal` that define the maximum no. of digit after decimal point of `amount` and `price`.

### Market price
```
GET /api/v2/market-price
```
Get market price for specific symbol or all symbols

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | Default is all symbols

**Response:**
```javascript
[
    {
        "symbol": "GTO_BTC",
        "price": "0.00002542",
        "updated_time": 1530682938651
    }
]
```

## Market data endpoints
The base endpoint is: https://engines.kryptono.exchange/

### Trade History (MARKET_DATA)
```
GET /api/v1/ht
```
Get Trade history for specific symbol

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | 

**If symbol is not sent, an empty response will be returned.**

**Response:**
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

### Order Book (MARKET_DATA)
```
GET /api/v1/dp
```
Get order book (depth)

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | 

**If symbol is not sent, an empty response will be returned.**

**Response:**
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

## Account endpoints
The base endpoint is: https://p.kryptono.exchange/k/

### New order (TRADE)
```
POST /api/v2/order/add     (HMAC SHA256)
```
Send in a new order

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_symbol | STRING | YES | |
order_side | STRING | YES | |
order_price | STRING | YES | |
order_size | STRING | YES | |
stop_price | STRING | NO | Used with `STOP_LOSS` and `TAKE_PROFIT` orders. |
type | STRING | YES | [Order Types](#order-types:) |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

Trigger for Stop-Limit orders:
* `stop_price` is above market price: `type` is `TAKE_PROFIT`
* `stop_price` is below market price: `type` is `STOP_LOSS`

**Request Body:**
```javascript
{
	"order_symbol" : "KNOW_ETH",
	"order_side" : "BUY",
	"order_price" : "0.0000123",
	"order_size" : "7777",
	"type" : "LIMIT",
	"timestamp" : 1507725176599,
	"recvWindow" : 5000
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

### Test new order (TRADE)
```
POST /api/v2/order/test     (HMAC SHA256)
```
Test new order creation and signature/recvWindow long. Creates and validates a new order but does not send it into the matching engine.

**Weight:** 1

**Parameters:** Same as `/api/v2/order/add`

**Response:**
```
{
    "result": true
}
```

### Get order detail (USER_DATA)
```
POST /api/v2/order/details      (HMAC SHA256)
```
Get order's detail

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

**Request Body:**
```javascript
{
  "order_id" : "0e3f05e0-912c-4957-9322-d1a34ef6e312",
  "timestamp" : 1429514463299,
  "recvWindow" : 5000
}
```

**Response:**
```javascript
{
    "order_id": "0e3f05e0-912c-4957-9322-d1a34ef6e312",
    "account_id": "14ce3690-4e86-4f69-8412-b9fd88535f8z",
    "order_symbol": "KNOW_BTC",
    "order_side": "SELL",
    "status": "open",
    "createTime": 1429514463266,
    "type": "limit",
    "order_price": "0.00001234",
    "order_size": "1000",
    "executed": "0",
    "stop_price": "0.00000000",
    "avg": "0.00001234",
    "total": "0.1234 BTC"
}
```

### Cancel Order (TRADE)
```
DELETE /api/v2/order/cancel       (HMAC SHA256)
```
Cancel an open order.

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |
order_symbol | STRING | YES | |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

**Request Body:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
  "order_symbol" : "KNOW_ETH",
  "timestamp" : 1429514463299,
  "recvWindow" : 5000
}
```

**Response:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
  "order_symbol" : "KNOW_ETH"
}
```

### Get trade details (USER_DATA)
```
POST /api/v2/order/trade-detail      (HMAC SHA256)
```
Get order's trade details

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
order_id | STRING | YES | |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

**Request Body:**
```javascript
{
  "order_id" : "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
  "timestamp" : 1429514463299,
  "recvWindow" : 5000
}
```

**Response:**
```javascript
[
        {
            "hex_id" : "5b39efc89f1ad4347f29b6be"
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
```

### Get Open Orders (USER_DATA)
```
POST /api/v2/order/list/open     (HMAC SHA256)
```
Get current open orders for specific symbol: OPEN and CANCELING

**You can only get 1000 orders maximum. Use `/api/v2/order/list/all` to get more.**

**Weight:** 1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
limit | INTEGER | NO | Default and Maximum are 50 |
page | INTEGER | NO | Default and first page are 0 |
symbol | STRING | YES | |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

* Value of `page` must be between `0` and `total` that is returned in the first request.

**Request Body:**
```javascript
{
  "limit" : 10,
  "page" : 0,
  "symbol" : "KNOW_BTC",
  "timestamp" : 1429514463299,
  "recvWindow" : 5000
}
```

**Response:**
```javascript
{
    "total": 5,  // total number of pages
    "list": [
        {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_BTC",
            "order_side": "BUY",
            "status": "open",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "0",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 BTC"
        }
    ]
}
```

**Response Description:**

Name | Type | Description
------------ | ------------ | ------------ 
total | INTEGER | Number of pages, only returned in the first page request, other page request returned -1 |


### Get Completed Orders (USER_DATA)
```
POST /api/v2/order/list/completed     (HMAC SHA256)
```
Get completed orders for specific symbol: FILLED, PARTIAL_FILL and CANCELED

**You can only get 1000 orders maximum. Use `/api/v2/order/list/all` to get more.**

**Weight:** 1

**Parameters:** Same as `/api/v2/order/list/open`

**Response:** Same as `/api/v2/order/list/open`


### Get all orders (USER_DATA)
```
POST /api/v2/order/list/all      (HMAC SHA256)
```
Get all account orders for specific symbol.

**Weight:** 5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |
from_id | STRING | NO | Order Id to fetch from. Use `order_id` from response. |
limit | INTEGER | NO | Default and Maximum are 50 |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

* If `from_id` is set, it will get orders < that `from_id` (older orders). Otherwise most recent orders are returned.

**Request Body:**
```
{
    "symbol" : "KNOW_BTC",
    "limit" : 50,
    "timestamp" : 1530682938651,
    "recvWindow" : 5000
}
```

**Response:**
```
[
    {
            "order_id": "02140bef-0c98-4997-9412-9e7ca6f1cc0e",
            "account_id": "bzbf4991-ad06-44e5-908c-691fdd55da14",
            "order_symbol": "KNOW_BTC",
            "order_side": "BUY",
            "status": "open",
            "createTime": 1528277973947,
            "type": "limit",
            "order_price": "0.00001230",
            "order_size": "7777",
            "executed": "0",
            "stop_price": "0.00000000",
            "avg": "0.00001230",
            "total": "0.09565710 BTC"
    }
]
```

### Get trade list (USER_DATA)
```
POST /api/v2/order/list/trades        (HMAC SHA256)
```
Get account trade history for specific symbol.

**Weight:** 10

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | |
from_id | STRING | NO | Hex Id to fetch from. Use `hex_id` from response. |
limit | INTEGER | NO | Default and Maximum are 50 |
timestamp | LONG | YES | 
recvWindow | LONG | NO |

* If `from_id` is set, it will get trade < that `from_id` (older trades). Otherwise most recent trades are returned.

**Request Body:**
```
{
    "symbol" : "KNOW_BTC",
    "limit" : 50,
    "timestamp" : 1530682938651,
    "recvWindow" : 5000
}
```

**Response:**
```
[
        {
            "hex_id" : "5b39efc89f1ad4347f29b6be"
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
```

### Account information (USER_DATA)
```
GET /api/v2/account/details       (HMAC SHA256)
```
Get account information.

**Weight:** 5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 
recvWindow | LONG | NO |

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

### Account balances (USER_DATA)
```
GET /api/v2/account/balances       (HMAC SHA256)
```
Get account balances

**Weight:** 5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 
recvWindow | LONG | NO |

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


## Market Information Stream
### Trade History
```
GET wss://engines.kryptono.exchange/ws/v1/ht/<symbol>
```

**Parameters:**

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

### Order Book
```
GET wss://engines.kryptono.exchange/ws/v1/dp/<symbol>
```

**Parameters:**

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

