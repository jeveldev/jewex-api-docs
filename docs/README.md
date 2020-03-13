# Introduction

> An awesome project.

Welcome to the Exchange API docs!

Service provides open API for trading operations and broadcasting of all trading events.

[filename](./websockets.md ':include')

## Orderbook events

All you need to start receiving live information about orderbook is connect to **wss://jewex.io** and subscribe to necessary channel.

There are channel names have pattern **book_{market_name}** where **{market_name}** is any pair available in service. For example **book_BTC_USD** or **book_BTC_ETH**.

> Example json event payload data.

```json
{
  "date": 1485686413.5688234,
  "id": 15235,
  "price": 935.23,
  "side": "sell",
  "amount": 0.0123,
  "market": "BTC_USD",
  "status": "partially-filled"
}
```

# HTTP API

HTTP API endpoint available on https://jewex.io/api/v1/.

Some methods requires authorization read below.

> We have limit in 2 calls per second from single account to authorization required methods and 100 calls per second from single IP address to public methods.

All HTTP methods accept JSON formats of requests and responses if it not specified by headers.

## Authorization

In order for access to private API methods, generate authorization keys in profile settings.

All request to these methods must contain the following headers:

* **X-KEY** - your key.
* **X-SIGNATURE** - query’s POST data, sorted by keys and signed by your key’s **“secret”** according to the HMAC-SHA256 method.
* **X-NONCE** - integer value, must be greater then nonce in previous api call.

<!-- tabs:start -->

#### ** Node.js **

```js
var crypto = require('crypto')
var _ = require('underscore')

var createSignature = function (options, secret) {
  options = _.omit(options, ['__proto__'])
  payload = Object
  .keys(options) // get keys of payload object
  .sort() // sort keys
  .map((key) => key + "=" + encodeURIComponent(options[key])) // each value should be url encoded. the most sensitive part for sign checking
  .join('&'); // to sting, separate with ampersand
  hmac = crypto.createHmac('sha256', secret)
  hmac.update(payload)
  hmac.digest('hex')
}
```

<!-- tabs:end -->

# Currencies

## List all currencies

Returns information about all available currencies.

**HTTP Request**

`GET https://jewex.io/api/v1/assets/`

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get('https://jewex.io/api/v1/assets/', function (error, response, body) {
    // process response    
});
```


<!-- tabs:end -->

# Pairs

<!-- panels:start -->
<!-- div:title-panel -->

## List all pairs

<!-- div:left-panel -->

Returns information about all available pairs.

**HTTP Request**
`GET https://jewex.io/api/v1/pairs/`

**Query parameters**

| Parameter | Required | Description |
| --------- | ------- | ----------- |
| fromSymbol | No | Base currency in market pair |
| toSymbol | No | Quote currency in market pair |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get('https://jewex.io/api/v1/pairs/', function (error, response, body) {
    // process response    
});
```

> Example output

```json
[
  {
    "fromSymbol": "BTC",
    "toSymbol": "USD",
    "name": "BTC_USD",
    "maximumAmount": 100000000.00000000,
    "minimumAmount": 0.00000001,
    "priceDecimals": 4
   }
]
```

<!-- tabs:end -->

<!-- panels:end -->

# Wallets

<!-- panels:start -->
<!-- div:title-panel -->

## List own wallets

!> This method requires authorization.

<!-- div:left-panel -->

Returns information about all wallets of account.

**HTTP Request**

`GET https://jewex.io/api/v1/wallets/`

**Query parameters**

| Parameter | Required | Description |
| --------- | ------- | ----------- |
| currency | No | Filter by currency |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/wallets/',
  {
    headers: getAuthHeaders({})
  },
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
[
  {
    "currency": "BTC",
    "balance": 0.00000000,
    "blocked": 0.00000000
   }
]
```

<!-- tabs:end -->

<!-- panels:end -->

# Orders

## Order statuses

| Value | Status Name | Description |
| ----- | ----------- | ----------- |
| active | Active | New order |
| partially-filled | Partially filled | Some amount of order was executed |
| filled | Filled | Order fully executed |
| cancelled | Cancelled | Order cancelled |

<!-- panels:start -->
<!-- div:title-panel -->

## Get orderbook

<!-- div:left-panel -->

Get full orderbook of active orders

**HTTP Request**

`GET https://jewex.io/api/v1/orderbook/<market>/`

**Query parameters**

| Parameter | Default | Required | Description |
| --------- | ------- | -------- | ----------- |
| limitSell | null | No | Sell orders limit |
| limitBuy | null | No | Buy orders limit |
| group | 1 | No | If set 1, then order will grouped by price |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

var url = 'https://jewex.io/api/v1/orderbook/BTC_USD/';

var params = {
  limitSell: 1,
  limitBuy: 1
};

request.get({url: url, qs: params}, function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
{
  "sell": [
    {
      "_id": 12345,
      "price": 911.519,
      "amount": 0.000446,
      "timestamp": 1485777324.410015
     }
   ],
  "buy": [
    {
      "_id": 12345,
      "price": 911.122,
      "amount": 0.001233,
      "timestamp": 1485777124.415542
    }
  ]
}
```

> Sample output when grouping enabled

```json
{
  "sell": [
    {
      "price": 911.519,
      "amount": 0.000446
     }
   ],
  "buy": [
    {
      "price": 911.122,
      "amount": 0.001233
    }
  ]
}
```

<!-- tabs:end -->

<!-- panels:end -->

<!-- panels:start -->
<!-- div:title-panel -->

## List own orders

!> This method requires authorization.

<!-- div:left-panel -->

List all orders created in your account. Can be filtered by query parameters.

**HTTP Request**

`GET https://jewex.io/api/v1/orders/own/`

**Query parameters**

| Parameter | Default | Description |
| --------- | ------- | ----------- |
| side | null | Filter by orders side (sell or buy) |
| market | null | Filter by pair |
| status | null | Filter by status |
| page | 1 | Filter by status |
| limit | 1000 | Limiting results |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/orders/own/',
  {
    headers: getAuthHeaders({})
  },
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
[
  {
    "_id": 11249,
    "market": "ETH_USD",
    "amount": 0.500000000,
    "actualAmount": 0.500000000,
    "side": "buy",
    "status": "active",
    "price": 0.00113000,
    "timestamp": 1584067830130
  }
]
```

<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:title-panel -->

## Retrieve single order

<!-- div:left-panel -->

Get detailed information about order by its id

**HTTP Request**

`GET https://jewex.io/api/v1/order/<orderId>/`

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');
var orderId = 12345;
request.get(
  `https://jewex.io/api/v1/order/${orderId}/`,
  function (error, response, body) {
    // process response    
  }
);
```

> Sample output

```json
{
  "_id": 12345,
  "amount": 0.1250000,
  "actualAmount": 0.1250000,
  "market": "BTC_USD",
  "side": "buy",
  "status": "active",
  "price": 870.69000000,
  "timestamp": 1584067830130
}
```

<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:title-panel -->

## Create order

!> This method requires authorization.

<!-- div:left-panel -->

Create new order that will be automatically executed.


**HTTP Request**

`POST https://jewex.io/api/v1/order/`

**Post parameters**

| Parameter | Description |
| --------- | ----------- |
| side | Side of order (sell or buy) |
| market | Pair of order |
| amount | Amount of first currency of pair |
| price | Price of order. This param have limited precision (See pairs) |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

var order = {
    side: 'buy',
    market: 'BTC_USD',
    amount: '1.0',
    price: '870.69'    
};

request.post({
    url: 'https://jewex.io/api/v1/order/',
    form: order,
    headers: getAuthHeaders(order)
  }, function (error, response, body) {
    // process response
    // save order id, check if it's executed
  }
);
```

> Example output

```json
{
  "success": true,
  "_id": 11268,
  "side": "buy",
  "market": "BTC_USD",
  "date": 1483721079.51632,
  "price": 870.69000000,
  "amount": 0.00000000,
  "timestamp": 1584067830130,
  "trades": [
    {
      "side": "sell",
      "price": 870.69000000,
      "orderId": 11266,
      "amount": 0.00010000,
      "_id": 6049
    }
  ]
}
```

<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:title-panel -->

## Cancel order

!> This method requires authorization.

<!-- div:left-panel -->

Cancel your active order. If order not exists, it’s done or cancelled error message will be returned.


**HTTP Request**

`POST https://jewex.io/api/v1/order-cancel/`

**Post parameters**

| Parameter | Description |
| --------- | ----------- |
| orderId | ID of order to cancel |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

var order = {
    orderId: 123456,
};

request.post({
    url: 'https://jewex.io/api/v1/order-cancel/',
    form: order,
    headers: getAuthHeaders(order)
  }, function (error, response, body) {
    // process response
  }
);
```

> Example output

```json
{
  "_id": 123456,
}
```

<!-- tabs:end -->
<!-- panels:end -->

# Trades

<!-- panels:start -->
<!-- div:title-panel -->

## List all trades

<!-- div:left-panel -->

List all orders created in your account. Can be filtered by query parameters.

**HTTP Request**

`GET https://jewex.io/api/v1/trades/`

**Query parameters**

| Parameter | Default | Description |
| --------- | ------- | ----------- |
| market | null | Filter by pair |
| fromDate | null | From date |
| toDate | null | To date |
| page | null | Result page |
| limit | 100 | Limiting results (max 1000) |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/trades/',
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
{
  "trades": [
    {
      "_id": 6030,
      "price": 839.36000000,
      "market": "BTC_USD",
      "side": "sell",
      "timestamp": 1483705817.735508,
      "amount": 0.00281167
    }
  ],
  "total": 1,
  "page": 1
}
```

<!-- tabs:end -->
<!-- panels:end -->

<!-- panels:start -->
<!-- div:title-panel -->

## List own trades

!> This method requires authorization.

<!-- div:left-panel -->

List only own exchanges where your account was buyer or seller.

**HTTP Request**

`GET https://jewex.io/api/v1/trades/own/`

**Query parameters**

| Parameter | Default | Description |
| --------- | ------- | ----------- |
| side | null | Filter by orders side (sell or buy) |
| market | null | Filter by pair |
| fromDate | null | From date |
| toDate | null | To date |
| page | null | Result page |
| limit | 100 | Limiting results (max 1000) |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/trades/own/',
  {
    headers: getAuthHeaders({})
  },
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
{
  "trades": [
    {
      "_id": 6030,
      "price": 839.36000000,
      "market": "BTC_USD",
      "side": "sell",
      "timestamp": 1483705817.735508,
      "amount": 0.00281167
    }
  ],
  "total": 1,
  "page": 1
}
```

<!-- tabs:end -->
<!-- panels:end -->

# Deposits

<!-- panels:start -->
<!-- div:title-panel -->

## List own deposits

!> This method requires authorization.

<!-- div:left-panel -->

List your deposits

**HTTP Request**

`GET https://jewex.io/api/v1/deposits/`

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/deposits/',
  {
    headers: getAuthHeaders({})
  },
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
[
  {
    "timestamp": 1485363039.18359,
    "_id": 317,
    "currency": "BTC",
    "amount": 10.00000000
  }
]
```

<!-- tabs:end -->
<!-- panels:end -->

# Withdrawals

## Withdraw statuses

| Value | Status Name | Description |
| ----- | ----------- | ----------- |
| new | New | Withdraw created, verification need |
| verified | Verified | Withdraw verified, waiting for approving |
| approved | Approved | Approved by moderator |
| refused | Refused | Refused by moderator. See your email for more details |
| pending | Pending | Pending blockchain confirmation |
| confirmed | Confirmed | Added to blockchain |
| cancelled | Cancelled | Cancelled by user |

<!-- panels:start -->
<!-- div:title-panel -->

## List own made withdraws

!> This method requires authorization.

<!-- div:left-panel -->

Get list of your withdraws

**HTTP Request**

`GET https://jewex.io/api/v1/withdrawals/`

**Query parameters**

| Parameter | Default | Description |
| --------- | ------- | ----------- |
| currency | null | Filter currency |
| status | null | Filter by status |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get(
  'https://jewex.io/api/v1/withdrawals/',
  {
    headers: getAuthHeaders({})
  },
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
[
  {
    "_id": 403,
    "timestamp": 1485363466.868539,
    "currency": "BTC",
    "amount": 0.53000000,
    "status": "verified"
  }
]
```

<!-- tabs:end -->
<!-- panels:end -->

# History Data

<!-- panels:start -->
<!-- div:title-panel -->

## Candles

Get price history candlestick data

<!-- div:left-panel -->

**Data interval**

| Value | Description |
| --------- | ----------- |
| 1 | 1 minute |
| 15 | 15 minutes |
| 30 | 30 minutes |
| 60 | 1 hour |
| 240 | 4 hours |
| D | 1 day |

**HTTP Request**

`GET https://jewex.io/api/v1/history/candles/<market>/<interval>/`

**Query parameters**

| Parameter | Default | Description |
| --------- | ------- | ----------- |
| limit | 720 | Limiting results |
| fromTime | null | Date from |
| toTime | null | Date to |

<!-- div:right-panel -->

<!-- tabs:start -->

#### ** Node.js **

```js
var request = require('request');

request.get({
    url: 'https://jewex.io/api/v1/history/candles/BTC_USD/60/'
  }
  function (error, response, body) {
    // process response    
  }
);
```

> Example output

```json
[
  {
    "volume": 0.262929,
    "high": 912.236,
    "low": 910.086,
    "close": 911.915,
    "time": 1485777600,
    "open": 910.424
   }
]
```

<!-- tabs:end -->
<!-- panels:end -->

# HTTP Errors

HTTP API uses following error statuses:

| Error code | Meaning |
| ---------- | -------- |
| 400 | Bad Request – Something wrong with input data. See response body for mode details. Also check API documentation. |
| 401 | Unauthorized – Your API key is wrong. Check public and secret keys, check auth headers. |
| 403 | Forbidden – Forbidden to process method with specified parameters. |
| 404 | Not Found – Method or resource not found. |
| 405 | Method Not Allowed – Check HTTP method for specified endpoint. |
| 406 | Not Acceptable – You requested a format that isn’t json. |
| 429 | Too Many Requests – You’re requesting fast! Slow down! See documentation for more details. |
| 500 | Internal Server Error – We had a problem with our server. Try again later. |
| 503 | Service Unavailable – We’re temporarially offline for maintanance. Please try again later. |
