

[TOC]



# Change log
|Doc Version|API Version|Date|Modification|Author|
|-------|----|--------------|--------|--------|
|v0.1|v1.0|2022/03/21|Initial version|Dingchun|


# 1. Getting Started

The exchange provides two interface modes for API users:

* **REST**: Use synchronous calls (request/response) to complete functions such as order creation, order cancellation, query transaction history,order history, portfolio, transfer history, etc.
* **WebSocket**: Use an asynchronous method (pubsub) to complete the subscribed push notification of orders, transactions, market data and other information.
  In order to facilitate users understanding following are the common steps to access and implement API to complete trading programs

## 1.1 Preparations

### 1.1.1 API-KEY Management

* Users have to log in the exchange website and apply for an API-KEY, please make sure to remember the following information when creating an API key:
  * **Access key**: API access key
  * **secret key**：the key used for signature authentication encryption (visible to the application only)

* Users have to assign permissions to API-KEY. There are four kinds of permissions,
  * **READ** read permission is used for data query interfaces such as order query, transaction query, etc.
  * **TRADE** trade permission is used for order placing, order cancelling, etc.
  * **TRANSFER** transfer permission is used for transfer interfaces, user can transfer between sub-accounts under the same main trading account.

* Users can set IP whitelist for API-KEY. If user set IP for the API-KEY, only the IPs in the whitelist can call the API. Each API-KEY will be bound to a maximum of 5 IPs. If the user has multiple  API-KEYs, they have to set IP whitelist for each API-KEY respectively.

* Both REST and WebSocket modes require users to authenticate the transaction through the API-KEY. Refer to the following chapters for the signature algorithm of the API-KEY..

### 1.1.2 Access Preparation

Before making a transaction, users have to query the server information to ensure that the program is in the correct state:

* Users have to query the server time to ensure that the local time and the server time are consistent
* Users need to query the version number to ensure that the version number is included in each request header. The request may be rejected if the version number is incorrect.

## 1.2 Signature Authentication

### 1.2.1 Signature

API requests are likely to be tampered during transmission through the internet. To ensure that the request remains unchanged, all private interfaces other than public interfaces (basic information, market data) must be verified by signature authentication via API-KEY to make sure the parameters or configurations are unchanged during transmission. Each created API-KEY need to be assigned with appropriate permissions in order to access the corresponding interface. Before using the interface, users is required to check the permission type for each interface and confirm there is appropriate permissions.
All HTTP requests to API endpoints require authentication and authorization. Users can obtain x-access-key and x-access-secret  by creating an API key flow. x-access-key and x-access-secret are used to verify and authorize all requests. The following headers should be added to all HTTP requests:

| Key                | Value                  | Description                                                                                                |
| ------------------ |------------------------|------------------------------------------------------------------------------------------------------------|
| x-access-key       | < API-KEY >            | The API Access Key you applied for                                                                         |
| x-access-sign      | < signatureOfRequest > | The value calculated by the hash value from request to ensure that the signature is valid and not tampered |
| x-access-timestamp | < timeOfRequest >      | The timestamp represents the time of request (in milliseconds)                                             |
| x-access-version   | < versionOfRequest >   | Signature protocol version.                                                       |
| Content-Type       | application/json       | Content-Type                                                                                               |

### 1.2.2 Signature Procedures

**Example:**
* The request is in a Map format and is serialized into a Json string. The map requires at least 3 keys: x-access-key, x-access-timestamp, x-access-version, include all columns in body when HTTP method is POST, include all columns in params where HTTP method is GET or DELETE. The example below illustrates how to sign a map.

```java
/*
  *Signature description：required fields for Signature include apiKey, apiSecret, timestamp, version, interface fields of each request.
  *field name and field value are in the form of key-value, and the Key is sorted by ASCII code.
  *The Following is test data demo:
*/

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.TreeMap;
import com.alibaba.fastjson.JSONObject;
import org.apache.commons.codec.binary.Hex;

public class SignDemo {

    public static void main(String[] args) throws Exception {
        String key="84dd8e670471a888e3a7547e120886cb";
        String secret = "3211b4306fcb1d9670cbee5abc69dead";
        String timeOfRequest ="1478692862000";

        TreeMap<String,String> treeMap = new TreeMap<>();

        treeMap.put("x-access-key",key); //api-key
        treeMap.put("x-access-timestamp",timeOfRequest); //current timestamp
        treeMap.put("x-access-version","v1.0");

        treeMap.put("field1","1");//request field mock1
        treeMap.put("field2","2");//request field mock2
        treeMap.put("field3","3");//request field mock3

        String requestText = JSONObject.toJSONString(treeMap);
        //Sort by ASCII of Map key {"field1":"1","field2":"2","field3":"3","x-access-key":"84dd8e670471a888e3a7547e120886cb","x-access-timestamp":"1478692862000","x-access-version":"1"}

        System.out.println(createSignature(requestText,secret));
        //cVfsAceIN7w+3vDf4WEIpA+iWJnK2TjKcmwgVARi8DI=
    }

    private static String createSignature(String requestText,String secret)
            throws Exception {
        return Base64.getEncoder().encodeToString(hmacSha1(requestText, secret));
    }

    private static byte[] hmacSha1(String plainData, String secret)
            throws Exception{
        Mac mac = Mac.getInstance("HmacSHA256");
        SecretKeySpec secretKey =
                new SecretKeySpec(Hex.decodeHex(secret.toCharArray()), "HmacSHA256");
        mac.init(secretKey);
        return mac.doFinal(plainData.getBytes());
    }
}
```

## 1.2 Environment

### 1.2.1 Testing Environment

Restful URL: <https://api-pro-sim.hashkey.com>

WebSocket: wss://api-pro-sim.hashkey.com

### 1.2.2 Production Environment

**Note:** Not public access, application needed

Restful URL: <https://api-pro.hashkey.com>

WebSocket: wss://api-pro.hashkey.com

# 2. REST API

## 2.1 Access

### 2.1.1 REST API Rate Limit
* Unless specified, each API Key has a rate limit of 10 requests per second. For instance, order-related endpoints have a rate limit of 10 requests per second.
* Unless specified, each IP has a rate limit of 10 requests per second.

### 2.1.2 Request Format

All API requests are under RESTful framework. Parameters can be set and sent by the request body in JSON format.

### 2.1.3 Response Format

All endpoints are in JSON standard format.  There are three fields, namely  **error_code**, **error_message**, and **data**. These specific business data are contained in the field **data**.

| **PARAMETER** | **TYPE** | **DESCRIPTION**   |
| ------------- | -------- | ----------------- |
| error_code    | string   | API return status |
| error_message | string   | API error message |
| data          | Object   | API return data   |

## 2.2 Universal

### 2.2.1 Get Timestamp

**Http Request:** GET info/time

**Request Content：** null

**Response Content：**

| **PARAMETER** | **TYPE** | **DESCRIPTION**        |
| ------------- | -------- | ---------------------- |
| timestamp     | int64    | millisecond time-stamp |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": {
        "timestamp": 1478692862000 // Server timestamp
    }
}
```

### 2.2.2 Get API Version

**Http Request:** GET info/version

**Request Content：** null

**Response Content：**

| **PARAMETER** | **TYPE** | **DESCRIPTION** |
| ------------- | -------- | --------------- |
| version       | string   | Version No.     |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": {
        "version": "v1.0" // Server version
    }
}
```

### 2.2.3 Query instruments

**Http Request:** GET info/instruments

**Request Content：** null

**Response Content：**

| **PARAMETER**           | **TYPE** | **DESCRIPTION**          |
|-------------------------|----------|--------------------------|
| instrument_id           | string   | Instrument ID.           |
| base_asset              | string   | Base  Asset.             |
| quote_asset             | string   | Quote Asset.             |
| product_type            | string   | Product Type(Token/Token:digital assets exchange, Token/Fiat:exchange digital assets to fiat currency, ST/Token:exchange ST to digital assets, ST/Fiat:exchange ST to fiat currency)           |
| price_tick              | string   | Price tick.              |
| max_market_order_volume | string   | Max market order volume. |
| min_market_order_volume | string   | Min market order volume. |
| max_limit_order_volume  | string   | Max limit order volume.  |
| min_limit_order_volume  | string   | Min limit order volume.  |

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "instrument_id": "BTC-USDC",
            "base_asset": "BTC",
            "quote_asset": "USDC",
            "product_type": "Token/Token",
            "price_tick": "1",
            "max_market_order_volume": "100",
            "min_market_order_volume": "0.0001",
            "max_limit_order_volume": "100",
            "min_limit_order_volume": "0.0001"
        },
        {
            "instrument_id": "ETH-BTC",
            "base_asset": "ETH",
            "quote_asset": "BTC",
            "product_type": "Token/Token",
            "price_tick": "0.000001",
            "max_market_order_volume": "1000",
            "min_market_order_volume": "0.001",
            "max_limit_order_volume": "1000",
            "min_limit_order_volume": "0.001"
        },
        {
            "instrument_id": "ETH-USDC",
            "base_asset": "ETH",
            "quote_asset": "USDC",
            "product_type": "Token/Token",
            "price_tick": "0.1",
            "max_market_order_volume": "3000",
            "min_market_order_volume": "0.001",
            "max_limit_order_volume": "3000",
            "min_limit_order_volume": "0.001"
        },
        {
            "instrument_id": "USDT-USDC",
            "base_asset": "USDT",
            "quote_asset": "USDC",
            "product_type": "Token/Token",
            "price_tick": "0.0001",
            "max_market_order_volume": "5000000",
            "min_market_order_volume": "1",
            "max_limit_order_volume": "5000000",
            "min_limit_order_volume": "1"
        }
    ]
}
```

### 2.2.4 Query instrument status

**Http Request:** GET info/instrument_status/{:instrument_id}
* instrument_id is required in the above url

**Request example：**

```context
 GET "https://domain/info/instrument_status/ETH-BTC"
```

**Response Content：**

| **PARAMETER** | **TYPE** | **DESCRIPTION** |
|---------------|----------|-----------------|
| instrument_id | string   | Instrument ID.  |
| status        | string   | Status  "BeforeTrading","NoTrading","Continuous","AuctionOrdering","AuctionBalance","AuctionMatch","Closed" |

| instrument status       | description        |
|-------------------------|--------------------|
| BeforeTrading           | Before trading     |
| NoTrading               | No trading         |
| Continuous              | Continuous trading |
| AuctionOrdering         | Auction ordering   |
| AuctionBalance          | Auction balance    |
| AuctionMatch            | Auction match      |
| Closed                  | Closed             |

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": {
        "instrument_id": "ETH-BTC",
        "status": "Continuous"
    }
}
```

## 2.3 Trading

### 2.3.1 Create An Order(TRADE permission is required)

**Http Request:** POST /order

**Request Content：**

| **PARAMETER**   | **TYPE** | **REQUIRED**| **DESCRIPTION**                                            |
| --------------- | -------- |-------------|------------------------------------------------------------|
| type            | string   | true        | "limit": limit order; "market": market order;  |
| client_order_id | string   | true        | Max length: 20. Must be unique                             |
| instrument_id   | string   | true        | e.g. "ETH-BTC"                                             |
| direction       | string   | true        | "buy" or "sell"                                            |
| price           | string   | false       | Limit price. Required when order type is limit |
| volume          | string   | true        | Total Volume                                               |
| post_only       | bool     | false       | Only maker  default false                                  |
| time_in_force (currently unused) | string   | default: limit : GTC, market : IOC |

**Response Content：**

| **PARAMETER**   | **TYPE** | **DESCRIPTION** |
| --------------- | -------- |-----------------|
| type            | string   | "limit": limit order; "market": market order; |
| client_order_id | string   | Client order id.                                                                                             |
| sys_order_id    | string   | Server order id.                                                                                             |
| instrument_id   | string   | e.g. "ETH-BTC"                                                                                               |
| direction       | string   | "buy" or "sell"                                                                                              |
| price           | string   | Limit Price. Required when order type is limit                                                 |
| volume          | string   | Total Volume                                                                                                 |
| post_only       | bool     | Only maker                                                                                                   |
| timestamp       | int64    | millisecond time-stamp                                                                                       |
| time_in_force (currently unused) | string   | default: limit and stopLimit: GTC, market and stopMarket: IOC                               |

**Request Example：**

```json
{
    "type": "limit", // Order type:
    "client_order_id": "000000001", // Client order ID
    "instrument_id": "ETH-BTC", // Instrument ID
    "direction": "buy", // Trade direction: buy、sell
    "price": "0.07", // Limit price
    "volume": "1", // Total Volume
    "post_only": false // Whether to only be a maker
}
```

**Response Example：**

```json
{
    "error_code": "0000",  // Errorcode
    "error_message": "",   // Errormessage
    "data": {
        "type": "limit",   // Order type
        "client_order_id": "000000001",     // Client order ID
        "sys_order_id": "1581479448132075", // System order ID
        "instrument_id": "ETH-BTC",         // Instrument ID
        "direction": "buy",                 // Trade direction
        "price": "0.07",                    // Limit price
        "volume": "1",                      // Total Volume
        "time_in_force": "GTC",             // Time in Force
        "post_only": false,                 // Whether to only be a maker
        "timestamp": 1681874571605          // Order timestamp
    }
}
```

### 2.3.2 Cancel An Order(TRADE permission is required)

**Http Request:** DELETE /order/

**Query Parameters：**

| **PARAMETER**   | **TYPE**   | **REQUIRED**  | **DESCRIPTION**                                         |
|-----------------|------------|---------------|---------------------------------------------------------|
| sys_order_id    | string     | false         | This filed is required when client_order_id is null     |
| client_order_id | string     | false         | This filed is required when sys_order_id is null        |
| volume          | string     | false         | If it is not input, the whole order will be cancelled   |

Note: If neither sys_order_id nor client_order_id is empty, then client_order_id is ignored.

**Response Content：**
null

**Request example：**

```context
 DELETE "https://domain/order?&sys_order_id=1550849345000001"
```
**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "" // Error message
}
```

### 2.3.3 Cancel all orders(TRADE permission is required)

**Http Request:** DELETE /orders

**Query Parameters：**

| **PARAMETER** | **TYPE**   | **REQUIRED** | **DESCRIPTION**                         |
|---------------|------------|--------------|-----------------------------------------|
| instrument_id | string     | false        | Cancel orders on a specific instrument_id only |


**Response Content：**
null

**Request example：**

```context
 DELETE "https://domain/orders?instrument_id=BTC-ETH"
```

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "" // Error message
}
```

### 2.3.4 Get Order Data(READ permission is required)

**Http Request:**  GET /orders

**Query Parameters：**

| **PARAMETER**   | **TYPE**     | **REQUIRED** | **DESCRIPTION**                                              |
|-----------------| ------------ |--------------|--------------------------------------------------------------|
| sys_order_id    | string       | false        | Server Order ID                                              |
| instrument_id   | string       | false        | e.g. "ETH-BTC"                                               |
| sorting         | string       | false        | "desc" or "asc"  default "asc"                               |
| direction       | string       | false        | "buy" or "sell"                                              |
| type            | string       | false        | Order type                                                   |
| status          | array string | false        | Array with order statuses to filter by.                      |
| start_timestamp | string       | true         | millisecond time-stamp                                       |
| end_timestamp   | string       | true         | millisecond time-stamp                                       |
| limit           | string       | true         | Limit on number of results to return. min 1 max 200.         |
| page            | string       | true         | Used for pagination. Page number.                            |

**Response Content：**

| **PARAMETER**     | **TYPE** | **DESCRIPTION**                                                                                           |
|-------------------| -------- |-----------------------------------------------------------------------------------------------------------|
| sys_order_id      | string   | Server Order ID                                                                                           |
| client_order_id   | string   | Client order id.                                                                                          |
| instrument_id     | string   | e.g. "ETH-BTC"                                                                                            |
| direction         | string   | "buy" or "sell"                                                                                           |
| type              | string   | "limit": limit order; "market": market order;  |
| price             | string   | Limit Price. Required when order type is limit                                                   |
| volume            | string   | Original Total Volume                                                                                     |
| status            | string   | Order status                                                                                              |
| post_only         | bool     | Only maker                                                                                                |
| timestamp         | int64    | millisecond time-stamp                                                                                    |
| filled_size       | string   | The size that has been filled                                                                             |
| unfilled_size     | string   | The size that has not been filled                                                                         |
| avg_filled_price  | string   | Average filled price                                                                                      |
| sum_trade_amount  | string   | cumulative trading amount(turnover)                                                                       |

**Order status**

| order status               | description                                                                            |
|----------------------------|----------------------------------------------------------------------------------------|
| NEW                        | The order has been accepted by the engine                                              |
| FILLED                     | The order has been completed                                                           |
| PARTIALLY_FILLED           | A part of the order has been filled, and the rest remains in the order book            |
| CANCELED                   | The order has been canceled by the user, whitout any trade                             |
| PARTIALLY_CANCELED         | A part of the order has been filled, and the  remaining part has been canceled by user |
| NOT_TRIGGERED              | The Stop order  is not triggered                                                       |
| REJECTED                   | The order was not accepted by the engine and not processed. (Currently not enabled)    |

**Request Example：**

```context
 GET "http://domain/orders?sys_order_id=1550849345000001&instrument_id=ETH-BTC&start_timestamp=1681874400000&end_timestamp=1681874817300&limit=50&page=1"
```

**Response Example：**

```json
{
    "error_code":"0000",    // Error code
    "error_message":"",     // Error message
    "data": [
        {
            "sys_order_id": "1581479448131991",    // System order ID
            "instrument_id": "ETH-BTC",            // InstrumentID
            "direction": "buy",                    // Trade direction
            "type": "limit",                       // Order type
            "price": "0.070000",                   // Limit Price
            "volume": "1.0000",                    // Original Total Volume
            "status": "FILLED",                    // Order status
            "timestamp": 1681874546357,            // Order timestamp
            "avg_filled_price": "0.069334",        // Average filled price
            "client_order_id": "000000001",        // Client order ID
            "filled_size": "1.0000",               // The size that has been filled
            "unfilled_size": "0.0000",             // The size that has not been filled
            "post_only": true,                     // Only as "maker"
            "sum_trade_amount": "0.069334"         // Turnover
        }
    ]
}
```

### 2.3.5 Retrieve Trade Data(READ permission is required)

**Http Request:** GET /trades

**Query Parameters** **:**

| **PARAMETER**   | **TYPE** | **REQUIRED** | **DESCRIPTION**                                              |
|-----------------| -------- |--------------|--------------------------------------------------------------|
| instrument_id   | string   | false        | e.g. "ETH-BTC"                                               |
| sys_order_id    | string   | false        | Server Order ID                                              |
| direction       | string   | false        | "buy" or "sell"                                              |
| sorting         | string   | false        | "desc" or "asc"    default "asc"                             |
| limit           | string   | true         | Limit on number of results to return. min 1 max 200.         |
| page            | string   | true         | Used for pagination. Page number.                            |
| start_timestamp | string   | true         | millisecond time-stamp                                       |
| end_timestamp   | string   | true         | millisecond time-stamp                                       |

**Response Content:**

| **PARAMETER**       | **TYPE** | **DESCRIPTION**        |
| ------------------- | -------- |------------------------|
| account_id          | string   | Account ID             |
| trade_id            | string   | Trade ID               |
| sys_order_id        | string   | Order ID               |
| instrument_id       | string   | e.g.  "ETH-BTC"        |
| direction           | string   | "buy" or "sell"        |
| price               | string   | Price                  |
| volume              | string   | Volume                 |
| fee                 | string   | Fee                    |
| fee_ccy             | string   | Fee Currency           |
| timestamp           | int64    | millisecond time-stamp |
| trade_type          | string   | "Taker", "Maker", "Invalid" |
| base_asset_id       | string   | "ETH"                  |
| base_asset_balance  | string   | Base Asset Balance     |
| quote_asset_id      | string   | "USDT"                 |
| quote_asset_balance | string   | Quote Asset Balance    |

**Trade Type**

| Trade type         | description |
|--------------------|-------------|
| Common             | Common Trade        |
| Invalid            | Invalid Trade       |


**Request Example:**

```context
 GET "http://domain/trades?sys_order_id=1550849345000001&instrument_id=ETH-BTC&start_timestamp=1681874400000&end_timestamp=1681874817300&limit=50&page=1"
```

**Response Example：**

```json
{
    "error_code":"0000",    // Error code
    "error_message":"",     // Error message
    "data": [
        {
            "trade_id": "1581446893129120",     // Trade ID
            "sys_order_id": "1581479448131991", // System order ID
            "instrument_id": "ETH-BTC",         // Instrument ID
            "direction": "buy",                 // Trade direction
            "price": "0.069334",                // Price
            "volume": "1.0000",                 // Volume
            "fee": "0.00017333",                // Transaction Fee
            "timestamp": 1681874546357,         // Trade timestamp
            "fee_ccy": "BTC",                   // Transaction Fee currency
            "trade_type": "Taker"               // Trade type
        }
    ]
}
```

## 2.4 Asset

### 2.4.1 Query Trading Account Assets(READ permission is required)

**Http Request:** GET /assets

**Request Content：** null

**Response Content：**

| **PARAMETER** | **TYPE** | **DESCRIPTION**        |
| ------------- | -------- | ---------------------- |
| asset         | string   | Asset type, e.g. "BTC" |
| free          | string   | Avalible balance       |
| freeze        | string   | Frozen balance         |

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "asset": "BTC",
            "free": "8.86098332",
            "freeze": "0.00000000"
        },
        {
            "asset": "ETH",
            "free": "31.999950000",
            "freeze": "0.000000000"
        },
        {
            "asset": "USDT",
            "free": "1921.389000",
            "freeze": "0.000000"
        },
        {
            "asset": "USDC",
            "free": "102960.184832",
            "freeze": "80.200000"
        }
    ]
}
```

### 2.4.2 Query Custody Account Assets(READ permission is required)

**Http Request:** GET /assets/custodyaccount

**Request Content：** null

**Response Content：**

|   **PARAMETER**   | **TYPE** | **DESCRIPTION**        |
| ----------------- | -------- | ---------------------- |
| asset             | string   | Asset type, e.g. "BTC" |
| available_balance | string   | Avalible balance       |
| total_balance     | string   | Total balance          |
| timestamp         |  int64   | millisecond time-stamp |

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "asset": "ETH",
            "available_balance": "1",
            "total_balance": "1",
            "timestamp": 1478692862000
        },
        {
            "asset": "USDC",
            "available_balance": "1",
            "total_balance": "1",
            "timestamp": 1478692862000
        }
    ]
}
```

### 2.4.3 Transfer Between Trading Accounts(TRANSFER permission is required)

**Http Request:** POST /assets/transfer

**Request Content：**

| **PARAMETER**   | **TYPE** | **REQUIRED** | **DESCRIPTION** |
|-----------------| -------- | ------------ |-----------------|
| asset           | string   | true         | Asset ID        |
| amount          | string   | true         | Amount          |
| to_account_id   | string   | true         | To Account ID   |
| from_account_id | string   | true         | From Account ID |

**Response Content：**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION** |
| ------------- | -------- | ------------ | --------------- |
| error_code    | string   | true         | Error Code      |
| error_message | string   | true         | Error Message   |
| data          | string   | true         | Data            |

**Request Example：**

```json
{
    "asset": "ETH",
    "amount": "1",
    "from_account_id": "B000000000001",
    "to_account_id": "B000000000002"
}
```

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": {}
}
```

### 2.4.4 Transfer With Custody Accounts(TRANSFER permission is required)

**Http Request:** POST /assets/transfer/custodyaccount

**Request Content：**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION** |
|---------------| -------- | ------------ |-----------------|
| asset         | string   | true         | Asset ID        |
| amount        | string   | true         | Amount          |
| type          | string   | true         | Operation type: <br> 01-Custody to trading main account <br> 02-Trading main account to custody account <br> 03-Fiat custody account to trading main account <br> 04-Trading main account to fiat custody|

**Response Content：**

|  **PARAMETER**  | **TYPE** | **DESCRIPTION** |
| --------------- | -------- | --------------- |
| transaction_id  | string   | Transfer ID     |
| asset_id        | string   | Asset ID        |
| amount          | string   | Amount          |
| timestamp       |  int64   | millisecond time-stamp |

**Request Example：**

```json
{
    "asset": "ETH",
    "amount": "1",
    "type": "01"
}
```

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": {
        "transaction_id": "1578170686002699",
        "asset": "ETH",
        "amount": "1",
        "timestamp": 1478692862000
    }
}
```

### 2.4.5 Query Withdrawal History(READ permission is required)

**Http Request:** GET /withdraw/history

**Query Parameters** **:**

| **PARAMETER**     | **TYPE** | **REQUIRED** | **DESCRIPTION**                                            |
|-------------------| -------- |--------------|------------------------------------------------------------|
| currency          | string   | false        | Currency                                                   |
| status            | string   | false        | Status <br>  "failed";"withdrawing";"successful"; <br> "cancelling";"cancelled" |
| limit             | string   | true         | Limit on number of results to return. min 1 max 200        |
| page              | string   | true         | Used for pagination. Page number.                          |
| start_timestamp   | string   | true         | millisecond time-stamp                                     |
| end_timestamp     | string   | true         | millisecond time-stamp                                     |


**Response Content：**

| **PARAMETER**     | **TYPE** | **DESCRIPTION**                |
|-------------------|----------|--------------------------------|
| withdraw_order_id | string   | withdraw order ID              |
| txn_id            | string   | Txn ID                         |
| network           | string   | Network (currently unused)     |
| currency          | string   | Currency                       |
| address           | string   | Withdrawal destination address |
| memo              | string   | Memo                           |
| volume            | string   | Volume                         |
| status            | string   | Status                         |
| gas_fee           | string   | Gas Fee                        |
| gas_fee_ccy       | string   | Gas Fee Currency               |
| fee               | string   | Fee                            |
| fee_ccy           | string   | Fee Currency                   |
| timestamp         | int64    | Timestamp                      |

**Request example：**

```context
 GET "https://domain/withdraw/history?currency=BTC&start_timestamp=1656928657000&end_timestamp=1681874817290&limit=50&page=1"
```


**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "withdraw_order_id": "00000001",
            "txn_id": "60fd9007ebfddc753455f95fafa808c4302c836e4d1eebc5a132c36c1d8ac354",
            "currency": "BTC",
            "address": "1FZdVHtiBqMrWdjPyRPULCUceZPJ2WLCsB",
            "memo": "",
            "volume": "1",
            "status": "successful",
            "fee": "0.004",
            "fee_ccy": "BTC",
            "gas_fee": "0.0001",
            "gas_fee_ccy": "BTC",
            "timestamp": 1677758330261
        }
    ]
}
```

### 2.4.6 Query Deposit History(READ permission is required)

**Http Request:** GET /deposit/history

**Query Parameters** **:**

| **PARAMETER**     | **TYPE** | **REQUIRED** | **DESCRIPTION**                                     |
|-------------------| -------- |--------------|-----------------------------------------------------|
| currency          | string   | false        | Currency  |
| status            | string   | false        | Status: <br>"addressToBeVerified";"underReview";"successful";<br>"failed"; "refundInProgress";"refundComplete";<br>"refundFailed";"receivingAccountCredit" |
| page              | string   | true         | Used for pagination. Page number.                   |
| limit             | string   | true         | Limit on number of results to return. min 1 max 200 |
| start_timestamp   | string   | true         | millisecond time-stamp                              |
| end_timestamp     | string   | true         | millisecond time-stamp                              |


**Response Content：**

| **PARAMETER**    | **TYPE** | **DESCRIPTION**            |
|------------------|----------|----------------------------|
| deposit_order_id | string   | Deposit order ID           |
| txn_id           | string   | Txn ID                     |
| network          | string   | Network (currently unused) |
| currency         | string   | Currency                   |
| address          | string   | Deposit source address     |
| memo             | string   | Memo                       |
| volume           | string   | Volume                     |
| status           | string   | Status                     |
| fee              | string   | Fee                        |
| fee_ccy          | string   | Fee Currency               |
| timestamp        | int64    | Timestamp                  |

**Request example：**

```context
 GET "https://domain/deposit/history?currency=BTC&start_timestamp=1656928657000&end_timestamp=1681874817290&limit=50&page=1"
```

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "deposit_order_id": "00000001",
            "txn_id": "60fd9007ebfddc753455f95fafa808c4302c836e4d1eebc5a132c36c1d8ac354",
            "currency": "BTC",
            "address": "1FZdVHtiBqMrWdjPyRPULCUceZPJ2WLCsB",
            "memo": "",
            "volume": "1",
            "status": "successful",
            "fee": "0.004",
            "fee_ccy": "BTC",
            "timestamp": 1677048471991
        }
    ]
}
```

### 2.4.7 Query Fiat Account Deposit/Withdraw History(READ permission is required)

**Http Request:** GET /fiat/account/history

**Query Parameters** **:**

| **PARAMETER**     | **TYPE** | **REQUIRED** | **DESCRIPTION**                                            |
|-------------------| -------- |--------------|------------------------------------------------------------|
| transaction_type  | string   | true         | 0-deposit,1-withdraw                                       |
| status            | string   | false        | Status: <br> 0001-under review <br> 0002-successful <br> 0003-failed <br> 1001-withdrawing <br> 1002-successful |
| start_timestamp   | string   | true         | millisecond time-stamp                                     |
| end_timestamp     | string   | true         | millisecond time-stamp                                     |
| limit             | string   | true         | Limit on number of results to return. min 1 max 200        |
| page              | string   | true         | Used for pagination. Page number.                          |

**Response Content：**

| **PARAMETER**    | **TYPE** | **DESCRIPTION**        |
|------------------|----------|------------------------|
| order_id         | string   | Order ID               |
| fiat_id          | string   | Asset ID               |
| fiat_type        | string   | "USD"                  |
| indicated_amount | string   | Order Amount           |
| amount           | string   | Real Amount            |
| fee              | string   | Fee                    |
| remark           | string   | Remark                 |
| status           | string   | Status: <br> 0001-under review <br> 0002-successful <br> 0003-failed <br> 1001-withdrawing <br> 1002-successful |
| create_timestamp | string   | Order create millisecond time-stamp |
| update_timestamp | string   | Order update millisecond time-stamp |

**Request example：**

```context
 GET "https://domain/fiat/account/history?transaction_type=0&start_timestamp=1656928657000&end_timestamp=1656928717000&limit=50&page=1"
```

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "order_id": "00000001",
            "fiat_id": "USD",
            "fiat_type": "USD",
            "indicated_amount": "100",
            "amount": "100",
            "fee": "10",
            "remark": "",
            "status": "0002",
            "create_timestamp": 1478692862000,
            "update_timestamp": 1478692862000
        }
    ]
}
```

### 2.4.8 Query Assets Transfer History(READ permission is required)

**Http Request:** GET /assets/transfer/history

**Query Parameters** **:**

| **PARAMETER**     | **TYPE** | **REQUIRED** | **DESCRIPTION**                                     |
|-------------------| -------- |--------------|-----------------------------------------------------|
| start_timestamp   | string   | true         | millisecond time-stamp                              |
| end_timestamp     | string   | true         | millisecond time-stamp                              |
| limit             | string   | true         | Limit on number of results to return. min 1 max 200 |
| page              | string   | true         | Used for pagination. Page number.                   |
| type              | string   | false        | Operation type: <br> 01-Custody account to trading main account <br> 02-Trading main account to custody account <br> 03-Fiat custody account to trading main account <br> 04-Trading main account to fiat custody account <br> 05-Between Trading account <br> Default: 05|

**Response Content：**

| **PARAMETER**    | **TYPE** | **DESCRIPTION**        |
|------------------|----------|------------------------|
| asset            | string   | Asset ID               |
| amount           | string   | Amount                 |
| from_account_id  | string   | From Account ID        |
| to_account_id    | string   | To Account ID          |
| status           | string   | "successful", "failed" |
| timestamp        | int64    | Timestamp              |


**Request example：**

```context
 GET "https://domain/assets/transfer/history?start_timestamp=1656928657000&end_timestamp=1656928717000&limit=50&page=1"
```

**Response Example：**

```json
{
    "error_code": "0000",
    "error_message": "",
    "data": [
        {
            "asset": "ETH",
            "amount": "1",
            "from_account_id": "B000000000001",
            "to_account_id": "B000000000002",
            "status": "successful",
            "timestamp": 1478692862000
        }
    ]
}
```



## 2.5 Market

### 2.5.1 Get Kline

**Http Request:** GET market/kline

**Query Parameters：**

| **PARAMETER**    | **TYPE** | **REQUIRED** | **DESCRIPTION**                                         |
|------------------|----------|--------------|---------------------------------------------------------|
| instrument_id    | string   | true         | e.g. "ETH-BTC"                                          |
| period           | string   | true         | m -> minutes; h -> hours; d -> days; w -> weeks; M -> months;<br/> "1m", "3m", "5m", "15m", "30m", "1h", "2h", "4h",<br/> "6h", "8h", "12h", "1d", "3d", "1w", "1M" |
| start_timestamp  | string   | true         | millisecond time-stamp  start from 000 milliseconds of this period |
| end_timestamp    | string   | true         | millisecond time-stamp  end at 000 milliseconds of the next period |
| page             | string   | true         | Used for pagination. Page number.                       |
| limit            | string   | true         | min 1 max 200                                           |

**Response Content：**

| **PARAMETER**   | **TYPE** | **DESCRIPTION**        |
|-----------------| -------- | ---------------------- |
| instrument_id   | string   | e.g. "ETH-BTC"               |
| open            | string   | Open Price                   |
| close           | string   | Close Price                  |
| high            | string   | High Price                   |
| low             | string   | Low Price                    |
| volume          | string   | Volume in base asset, e.g, ETH in ETH-BTC |
| start_timestamp | int64    | Start millisecond time-stamp |
| end_timestamp   | int64    | End millisecond time-stamp   |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": [
        {
            "instrument_id": "ETH-BTC", // Instrument ID
            "open": "10", // Open price
            "close": "10", // Close price
            "high": "10", // High price
            "low": "10", // Low price
            "volume": "100", // Volume
            "start_timestamp": 1646213700000, // Start time
            "end_timestamp": 1646213760000 // End time
        }
    ]
}
```

### 2.5.2 Get Trade List

**Http Request:** GET /trades/market

**Query Parameters：**

| **PARAMETER**   | **TYPE** | **REQUIRED** | **DESCRIPTION**                   |
|-----------------|----------|--------------|-----------------------------------|
| instrument_id   | string   | false        | e.g. "ETH-BTC"                    |
| start_timestamp | string   | true         | millisecond time-stamp            |
| end_timestamp   | string   | true         | millisecond time-stamp            |
| limit           | string   | true         | min 1 max 200.                    |
| page            | string   | true         | Used for pagination. Page number. |

**Response Content：**

| **PARAMETER** | **TYPE** | **DESCRIPTION**  |
| ------------- | -------- | ---------------- |
| instrument_id | string   | e.g. "ETH-BTC"   |
| trade_id      | string   | Trade ID         |
| price         | string   | Price            |
| volume        | string   | Volume           |
| timestamp     | int64    | millisecond time-stamp |
| direction     | string   | Taker direction        |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": [
        {
            "trade_id": "1559150521200027", // Trade ID
            "instrument_id": "ETH-BTC", // Instrument ID
            "price": "0.069212", // Price
            "volume": "100", // Volume
            "timestamp": 1478692862000, // Trade timestamp
            "direction": "buy" // Taker direction
        }
    ]
}
```

## 2.6 Account

### 2.6.1 Query Main Account Info(READ permission is required)

**Http Request:** GET /account/trading/main

**Response Content：** 无

**Response Content：**

| **PARAMETER**            | **TYPE** | **DESCRIPTION**         |
| ------------------------ | -------- | ----------------------- |
| client_id                | string   | client ID               |
| sub_account_quantity     | string   | Under the current account, the number of linked sub-accounts |
| max_sub_account_quantity | string   | The maximum number of linked sub-accounts |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": {
        "client_id": "C0000010001", // client ID
        "sub_account_quantity": "5", // the number of linked sub-accounts
        "max_sub_account_quantity": "9" // the maximum number of linked sub-accounts
    }
}
```

### 2.6.1 Query Sub Account List(READ permission is required)

**Http Request:** GET /accounts/trading/sub

**Response Content：** null

**Response Content：**

| **PARAMETER**    | **TYPE** | **DESCRIPTION**         |
| ---------------- | -------- | ----------------------- |
| client_id        | string   | client ID               |
| sub_account      | string   | sub account name        |
| sub_account_id   | string   | sub account ID          |
| label            | string   | sub account label       |
| timestamp        | int64    | millisecond time-stamp  |

**Response Example：**

```json
{
    "error_code": "0000", // Error code
    "error_message": "", // Error message
    "data": [
        {
            "client_id": "C0000010001", // client ID
            "sub_account": "test", // sub account name
            "sub_account_id": "B0000010003", // sub account ID
            "label": "test", // sub account label
            "timestamp": 1478692862000 // millisecond time-stamp
        }
    ]
}
```

# 3. Websocket API

## 3.1 Access

### 3.1.1 WebSocket API session Limit
* Use apikey to authorize websocket sessions, and 10 sessions can be authorized at the same time.
* Only authorized sessions can subscribe to private data and public data.

**URL for Access:**
/stream

### 3.1.2 Heartbeat Message

When user's Websocket client application gets connected with the HashKey Websocket server, the server will periodically (every 10 seconds currently) send a ping message containing the sessionID and current timestamp.

```json
 {"type":"ping","sessionID":"74939a43-0523-4cb1-a870-0dbadfda6a62","data":"1492420473027"}
```

When the user receives the mentioned message from the Websocket client application, a pong message containing the same timestamp should be returned.

```json
{"type":"pong","data":"1492420473027"}
```

**Note**: When the Websocket server sends two ping messages successively but gets no pong message in return, the connection will be disconnected.

### 3.1.3 Request Format

All subscription request bodies are expected to be in valid JSON format. Each topic has its own parameter set. For details, please refer to the endpoint request example.

### 3.1.4 Response Format

All response bodies are expected to be in valid JSON format.  For details, please refer to the endpoint response example.

### 3.1.5 Authentication

**Request Content:**

| **PARAMETER**      | **TYPE** | **REQUIRED** | **DESCRIPTION**                                                                          |
| ------------------ |----------| ------------ |------------------------------------------------------------------------------------------|
| type               | string   | true         | Default: auth                                                                            |
| x-access-key       | string   | true         | The API Access Key you applied for.                                                      |
| x-access-sign      | string   | true         | A value calculated by the hash value from request to ensure it is valid and not tampered. |
| x-access-timestamp | string   | true         | The timestamp represents the time of request (in milliseconds).                          |
| x-access-version   | string   | true         | Signature protocol version. Default version is 1.                                        |

**Request Example：**

```json
{
    "type": "auth",
    "auth": {
        "x-access-key": "xxxxxxxxx",
        "x-access-sign": "xxxxxxxxx",
        "x-access-timestamp": "1478692862000",
        "x-access-version": "v1.0"
    },
    "id": 1
}
```

**Response Example：**

```json
{
    "type": "auth-resp",
    "error_code": "0000",
    "error_message": "success"
}
```

### 3.1.6 Subscribe

**Request Content:**

| **PARAMETER** | **TYPE**     | **REQUIRED** | **DESCRIPTION**      |
|---------------| ------------ | ------------ |----------------------|
| type          | string       | true         | "sub"                |
| id            | integer      | true         | Unique request id    |
| parameters    | object array | true         | Subscribe parameters |
| => topic      | string       | true         | Topic                |

**Response Content:**

| **PARAMETER** | **TYPE**     | **DESCRIPTION**                                |
| ------------- | ------------ |------------------------------------------------|
| id            | integer      | Unique request id                              |
| result        | object array | Topic list                                     |
| error_code    | string       | 0000: success 0100: partial success 0001: fail |

**Request Example:**

```json
{
    "type": "sub", // Message type, sub: subscribe
    "parameters": [
        {
            "topic": "order_rtn", // Subscribe topoic
            "instrument_id": "ETH-USDT" // Instrument ID
        }
    ],
    "id": 1 // Message ID
}
```

**Response Example:**

```json
{
    "id": 1,                          // Message ID
    "result": null,                   // Result is null when all topics are successfully subscribed; otherwise, it returns the topics that are successfully subscribed
    "error_code": "0000"              // Error code is 0000 when all topics are successfully subscribed; otherwise, it returns the error code that unsuccessfully subscribed
}
```

### 3.1.7 Unsubscribe

**Request Content:**

| **PARAMETER** | **TYPE**     | **REQUIRED** | **DESCRIPTION**        |
|---------------|--------------| ------------ |------------------------|
| type          | string       | true         | "unsub"                |
| id            | int64        | true         | Unique request id      |
| parameters    | object array | true         | Unsubscribe parameters |
| => topic      | string       | true         | Topic                  |
| error_code    | string       | true         | Error code             |
| error_message | string       | true         | Explain error info     |

**Response Content:**

| **PARAMETER** | **TYPE**     | **DESCRIPTION**                                |
| ------------- |--------------|------------------------------------------------|
| id            | int64        | Unique request id                              |
| result        | object array | Topic list                                     |
| error_code    | string       | 0000: success 0100: partial success 0001: fail |

**Request Example:**

```json
{
    "type": "unsub",
    "parameters": [
        {
            "topic": "xxxxxxxxx"
        }
    ],
    "id": 2
}
```

**Response Example:**

```json
{
    "id": 1,
    "result": null,
    "error_code": "0000"
}
```



## 3.2 Market Data (Public stream)

### 3.2.1 Kline

**Push Frequency**
Push every 1000 milliseconds

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**                                              |
| ------------- | -------- | ------------ | ------------------------------------------------------------ |
| type          | string   | true         | "sub"                                                        |
| topic         | string   | true         | "kline"                                                      |
| period        | string   | true         | m -> minutes; h -> hours; d -> days; w -> weeks; M -> months;<br/> "1m", "3m", "5m", "15m", "30m", "1h", "2h", "4h",<br/> "6h", "8h", "12h", "1d", "3d", "1w", "1M" |
| instrument_id | string   | true         | e.g. "ETH-USDT", "ETH-BTC"                                   |

**Response Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**                               |
| ------------- | -------- | --------------------------------------------- |
| type          | string   | "sub-resp"                                    |
| topic         | string   | "kline"                                       |

**Data Content:**

| **PARAMETER**   | **TYPE** | **DESCRIPTION**                               |
|-----------------| -------- | --------------------------------------------- |
| instrument_id   | string   | e.g. "ETH-USDT", "ETH-BTC"                    |
| open            | string   | Open price                                    |
| high            | string   | High  price                                   |
| low             | string   | Low  price                                    |
| close           | string   | Close  price                                  |
| volume          | string   | Volume in base asset, e.g, ETH in ETH-BTC     |
| start_timestamp | int64    | millisecond time-stamp     e.g. 1646213700000 |
| end_timestamp   | int64    | millisecond time-stamp     e.g. 1646213800000 |

**How to Subscribe：**

```json
{
    "type": "sub", // Message type
    "parameters": [
        {
            "topic": "kline", // Subscribe topic(kline)
            "period": "1m", // Time period
            "instrument_id": "ETH-USDT" // Instrument ID
        }
    ],
    "id": 1 // Message ID
}
```

**Response Example：**

```json
{
    "topic":"kline",                        // Subscribe topic
    "data":[
      {
        "close":"0.07",                      // Close price
        "high":"0.07",                       // Highest price
        "low":"0.07",                        // Lowest price
        "open":"0.07",                       // Open price
        "instrument_id":"ETH-BTC",           // Instrument ID
        "volume":"0.00000",                  // Volume in base asset
        "start_timestamp":1679624460000,     // Start time
        "end_timestamp":1679624519999        // End time
     } 
    ]
}
```

### 3.2.2 Market Data

**Push frequency**
Push every 1000 milliseconds

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- | ------------ | -------------------------- |
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "market_data"              |
| instrument_id | string   | true         | e.g. "ETH-USDT", "ETH-BTC" |

**Response Content:**

| **PARAMETER**        | **TYPE** | **DESCRIPTION**        |
| -------------------- | -------- | ---------------------- |
| type                 | string   | "sub-resp"             |
| topic                | string   | "market_data"          |

**Data Content:**

| **PARAMETER**        | **TYPE** | **DESCRIPTION**                           |
|----------------------| -------- |-------------------------------------------|
| high                 | string   | High price                                |
| low                  | string   | Low price                                 |
| open                 | string   | Open price                                |
| instrument_id        | string   | e.g. "ETH-USDT", "ETH-BTC"                |
| base                 | string   | Base asset, e.g, ETH in ETH-BTC           |
| quote                | string   | Quote asset, e.g, BTC in ETH-BTC          |
| last_price           | string   | Price of the latest trade                 |
| price_change_rate    | string   | Price change rate of 24 hours             |
| price_change         | string   | Price change  of 24 hours                 |
| volume               | string   | Volume in base asset, e.g, ETH in ETH-BTC |

**How to Subscribe：**

```json
{
    "type": "sub", // Message type
    "parameters": [
        {
            "topic": "market_data", // Subscribe topoic
            "instrument_id": "BTC-USDC" // Instrument ID
        }
    ],
    "id": 1 // Message ID
}
```

**Response Example：**

```json
{
    "topic": "market_data", // Subscribe topic
    "data": [
        {
            "open": "24468", // Open price
            "high": "24468", // Highest price
            "low": "24468", // Lowest price
            "base": "BTC", // Base asset
            "quote": "USDC", // Quote asset
            "instrument_id": "BTC-USDC", // Instrument ID
            "price_change_rate": "0.00000", // Price change rate of 24 hours
            "price_change": "0.00000", // Price change rate of 24 hours
            "last_price": "24468", // Price of the latest trade
            "volume": "0.00000" // Volume of 24 hours
        }
]
}
```

### 3.2.3 Order book Data

**Push frequency**
Push every 500 milliseconds(If there is any change)

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- | ------------ | -------------------------- |
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "depth_market_data"        |
| instrument_id | string   | true         | e.g. "ETH-USDT", "ETH-BTC" |

**Response Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**              |
| ------------- | -------- |------------------------------|
| type          | string   | "sub-resp"                   |
| topic         | string   | "depth_market_data"          |

**Data Content:**

| **PARAMETER**   | **TYPE** | **DESCRIPTION** |
|-----------------|----------|-----------------|
| instrument_id   | string   | Instrument Id   |
| sequence_no     | int64    | Sequence No     |
| timestamp       | int64    | Timestamp       |

**Ask and Bid Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION** |
|---------------|----------|-----------------|
| volume        | string   | Volume          |
| price         | string   | Price           |

**How to Subscribe：**

```json
{
    "type": "sub", // Message type
    "parameters": [
        {
            "topic": "depth_market_data", // Subscribe topoic
            "instrument_id": "ETH-USDT" // Instrument ID
        }
    ],
    "id": 1 // Message ID
}
```

**Response Example：**

```json
{
    "topic":"depth_market_data",            // Subscribe topic
    "data":[
      {
        "instrument_id":"ETH-USDT",
        "sequence_no": 100,
        "timestamp": 1646213700000,
        "ask":[                            // Sell 50 levels, sorted from small to large according to the price
            {
                "volume":"3",               // volume
                "price":"1.7"                // price
            },
            {
                "volume":"3",
                "price":"2"
            }
        ],
        "bid":[
            {                              // Buy 50 levels, sorted from large to small according to the price
                "volume":"3",
                "price":"1.5"
            }
        ]
      }
    ]
}
```

### 3.2.4 Total Trade Data

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- |--------------|----------------------------|
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "trade_rtn_all"            |
| instrument_id | string   | false        | e.g. "ETH-USDT", "ETH-BTC" |

**Response Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**  |
| ------------- | -------- | ---------------- |
| type          | string   | "sub-resp"       |
| topic         | string   | Topic            |


**Data Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**        |
|---------------| -------- |------------------------|
| instrument_id | string   | Instrument Id          |
| trade_id      | string   | Trade Id               |
| volume        | string   | Volume                 |
| price         | string   | Price                  |
| timestamp     | int64x   | millisecond time-stamp |
| direction     | string   | Taker direction        |

**How to Subscribe：**

```json
{
    "type": "sub",
    "parameters": [
        {
            "topic": "trade_rtn_all",
            "instrument_id": "ETH-USDT"
        }
    ],
    "id": 1
}
```

**Request Response：**

```json
{
    "topic": "trade_rtn_all", // Subscribe topic
    "data": [
        {
            "instrument_id": "ETH-USDT", // Instrument ID
            "trade_id": "1578862103000011", // Trade ID
            "volume": "2", // Volume
            "price": "2", // Price
            "timestamp": 1478692862000, // Trade time
            "direction": "buy" // Taker direction
        },
        {
            "instrument_id": "ETH-USDT",
            "trade_id": "1578862103000012",
            "volume": "2",
            "price": "2",
            "timestamp": 1478692862000,
            "direction": "buy"
        }
    ]
}
```

### 3.2.5 Instruments status change

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**             |
| ------------- | -------- | ------------ |-----------------------------|
| type          | string   | true         | "sub"                       |
| topic         | string   | true         | "instruments_status_change" |
| instrument_id | string   | false        | e.g. "ETH-USDT", "ETH-BTC"  |

**Response Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**  |
| ------------- | -------- | ---------------- |
| type          | string   | "sub-resp"               |
| topic         | string   | Topic                    |

**Data Content:**

| **PARAMETER** | **TYPE** | **DESCRIPTION**        |
|---------------| -------- |------------------------|
| instrument_id | string   | Instrument Id          |
| status        | string   | Status                 |


**How to Subscribe：**

```json
{
    "type": "sub",
    "parameters": [
        {
            "topic": "instruments_status_change",
        }
    ],
    "id": 1
}
```

**Request Response：**

```json
{
    "topic": "instruments_status_change",
    "data": [
        {
            "instrument_id": "ETH-BTC",
            "status": ""
        }
    ]
}
```


## 3.3  Transaction Data (Private Stream)

### 3.3.1 Order Data

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- | ------------ | -------------------------- |
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "order_rtn"                |
| instrument_id | string   | false        | e.g. "ETH-USDT", "ETH-BTC" |

**Response Content:**

| **PARAMETER**   | **TYPE** | **DESCRIPTION**                                              |
| --------------- | -------- | ------------------------------------------------------------ |
| type            | string   | "sub-resp"                                                   |
| topic           | string   | "order_rtn"                                                  |

**Data Content:**

| **PARAMETER**    | **TYPE** | **DESCRIPTION**                                                                                              |
| ---------------- | -------- |--------------------------------------------------------------------------------------------------------------|
| sys_order_id     | string   | Server Order ID                                                                                              |
| client_order_id  | string   | Client order id.                                                                                             |
| instrument_id    | string   | e.g. "ETH-BTC"                                                                                               |
| direction        | string   | "buy" or "sell"                                                                                              |
| type             | string   | "limit": limit order; "market": market order; "stopLimit": stop limit order; "stopMarket": stop market order |
| stop_price       | string   | Required when order type is stopLimit or stopMarket                                                          |
| price            | string   | Limit Price. Required when order type is limit or stopLimit                                                  |
| volume           | string   | Original Total Volume                                                                                        |
| status           | string   | Order status                                                                                                 |
| post_only        | bool     | Only maker                                                                                                   |
| timestamp        | int64    | millisecond time-stamp                                                                                       |
| filled_size      | string   | The size that has been filled                                                                                |
| unfilled_size    | string   | The size that has not been filled                                                                            |
| avg_filled_price | string   | Average filled price                                                                                         |
| sum_trade_amount | string   | cumulative trading amount(turnover)                                                                          |

**How to Subscribe：**

```json
{
    "type": "sub",
    "parameters": [
        {
            "topic": "order_rtn",
            "instrument_id": "ETH-USDT"
        }
    ],
    "id": 1
}
```

**Response Example：**

```json
{
    "topic": "order_rtn",
    "data": [
        { 
            "sys_order_id":"1578862103000763",     // System order ID
            "client_order_id":"1679644419000111",  // Client order ID
            "type":"limit",                        // Order type
            "instrument_id":"ETH-USDC",            // InstrumentID
            "direction":"buy",                     // Trade direction
            "price":"1699.0",                      // Limit Price
            "volume":"0.0100",                     // Original Total Volume
            "filled_size":"0.0000",                // The size that has been filled 
            "status":"NEW",                        // Order status
            "timestamp":1679644419109,             // Order timestamp
            "post_only":False,                     // If "maker" only
            "avg_filled_price":"0.0",              // Average filled price
            "unfilled_size":"0.0100",              // The size that has not been filled
            "sum_trade_amount":"0.000000"          // Turnover
        } 
    ]
}
```

### 3.3.2 Trade Data

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- | ------------ | -------------------------- |
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "trade_rtn"                |
| instrument_id | string   | false        | e.g. "ETH-USDT", "ETH-BTC" |

**Response Content:**

| **PARAMETER**       | **TYPE** | **DESCRIPTION**                      |
| ------------------- | -------- |--------------------------------------|
| type                | string   | "sub-resp"                           |
| topic               | string   | "trade_rtn"                          |
| account_id          | string   | Account ID                           |
| trade_id            | string   | Trade Id                             |
| client_order_id     | string   | Client Order Id                      |
| sys_order_id        | string   | Server Order Id                      |
| instrument_id       | string   | e.g. "ETH-BTC"                       |
| direction           | string   | "buy" or "sell"                      |
| price               | string   | Price                                |
| volume              | string   | Volume                               |
| fee                 | string   | Fee                                  |
| fee_ccy             | string   | Transaction fee currency, e.g. "ETH" |
| timestamp           | int64    | Trade millisecond time-stamp         |
| trade_type          | string   | "Taker","Maker","Invalid"                  |
| base_asset_id       | string   | "ETH"                                |
| base_asset_balance  | string   | Base Asset Balance                   |
| quote_asset_id      | string   | "USDT"                               |
| quote_asset_balance | string   | Quote Asset Balance                  |


**How to Subscribe：**

```json
{
    "type": "sub",
    "parameters": [
        {
            "topic": "trade_rtn",
            "instrument_id": "ETH-USDT"
        }
    ],
    "id": 1
}
```

**Response Example：**

```json
{
    "topic": "trade_rtn",
    "data": [
        {
            "sys_order_id": "1550849345000001", // System order ID
            "client_order_id": "000000001", // Client order ID
            "trade_id": "1", // Trade ID
            "instrument_id": "ETH-BTC", // Instrument ID
            "direction": "buy", // Trade direction
            "price": "1", // Price
            "volume": "1", // Volume
            "fee": "0.05", // Transaction fee
            "fee_ccy": "BTC", // Transaction fee currency
            "trade_type": "Taker", // Trade 
            "timestamp": 1478692862000, // Trade time
        },
        {
            "sys_order_id": "1550849345000002",
            "client_order_id": "000000002",
            "trade_id": "2",
            "instrument_id": "ETH-BTC",
            "direction": "sell",
            "price": "1",
            "volume": "2",
            "fee": "0.05",
            "fee_ccy": "BTC",
            "trade_type": "Taker",
            "timestamp": 1478692862000
        }
    ]
}
```

### 3.3.3 Balance Data

**Request Content:**

| **PARAMETER** | **TYPE** | **REQUIRED** | **DESCRIPTION**            |
| ------------- | -------- | ------------ | -------------------------- |
| type          | string   | true         | "sub"                      |
| topic         | string   | true         | "balance"                  |
| account_type  | string   | true         | 01-Fiat Account <br> 02-Custody Account <br> 03-Trading Account |

**Response Content:**

| **PARAMETER**       | **TYPE** | **DESCRIPTION**                             |
| ------------------- | -------- | ------------------------------------------- |
| type                | string   | "sub-resp"                                  |
| topic               | string   | "trade_rtn"                                 |
| client_id           | string   | Client ID                                   |
| account_id          | string   | Account ID                                  |
| event_type          | string   | "snapshot","deposit","withdraw","transfer"  |
| asset_id            | string   | "ETH"                                       |
| asset_balance       | string   | Balance after change                        |
| event_timestamp     | int      | Balance change event millisecond time-stamp |
| timestamp           | int      | Message push millisecond time-stamp         |


**How to Subscribe：**

```json
{
    "type": "sub",
    "parameters": [
        {
            "topic": "balance",
            "account_type": "03"
        }
    ],
    "id": 1
}
```

**Response Example：**

```json
{
    "topic": "balance",
    "data": [
        {
            "client_id": "C0000010001",
            "account_id": "A0000010001",
            "event_type": "snapshot",
            "asset_id": "ETH",
            "asset_balance": "100",
            "event_timestamp": 1478692862000,
            "timestamp": 1478692862000
        }
    ]
}
```

# 4. Error code

## 4.1 Error Code Classification Standard

| **ERROR CODE RANGE** | **DESCRIPTION**    |
| -------------------- | ------------------ |
| 0000~0099            | Public error code      |
| 0100~0199            | WebSocket error code   |

## 4.2 Error Code Example

| **ERROR CODE**   | **TYPE**   | **ERROR MESSAGE**                                  |
|------------------| ---------- |----------------------------------------------------|
| 0000             | Public error code    | Success                                            |
| 0001             | Public error code    | common parameter error                                    |
| 0002             | Public error code    | System error                                       |
| 0003             | Public error code    | Request not allowed                                |
| 0004             | Public error code | Insufficient account funds                         | |
| 0005             | Public error code | Order not found                                    |
| 0006             | Public error code | Instrument not found                               |
| 0007             | Public error code | Invalid volume                               |
| 0008             | Public error code | Not enough volume to cancel                        |
| 0009             | Public error code | Unsatisfied with the price precision               |
| 0010             | Public error code | Invalid price                                      |
| 0011             | Public error code | Operation for the APIKey user not allowed               |
| 0012             | Public error code | Unsupported order type                             |
| 0013             | Public error code | duplicate orders                                   |
| 0100             | WebSocket error code | Subscription failed                                |
| 0101             | WebSocket error code | Partial subscriptions succeeded                    |

