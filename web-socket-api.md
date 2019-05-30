# Web Socket Data Feeds for Namebase

The host for the streams is wss://namebase.io:443/
The /ticker feeds send data every second. 
The /stream feeds send data on each event as soon as it happens, e.g. on each trade.

## Trade stream
The Trades feed sends trade information with the buyer and seller ids scrubbed out.

**Feed name:** '/ws/v0/ticker/trades' (in flux)

**Payload:**
```javascript
{
  "eventType": "trade",       // Event type
  "eventTime": 1555553960000, // Event time
  "symbol": "HNSBTC",         // Symbol
  "tradeId": 12345,           // Trade id
  "price": "0.001",           // Price
  "quantity": "100",          // Quantity
  "quoteQuantity": "100",     // Quote quantity
  "createdAt": 1555553960000, // The time the trade occurred
  "isBuyerMaker": true,       // True iff the buy order is the maker
}
```

## Klines ticker
Connect to the klines feed to receive klines (i.e. candlesticks) every second.

**Valid kline intervals:**

* `1m` (one minute)
* `5m` (five minutes)
* `15m` (fifteen minutes)
* `1h` (one hour)
* `4h` (four hours)
* `12h` (twelve hours)
* `1d` (one day)
* `1w` (one week)

**Feed name:** '/ws/v0/ticker/kline_<interval, e.g. "1m">'

**Payload:**
```javascript
{
  "eventType": "kline",           // Event type
  "eventTime": 1555553960000,     // Event time
  "symbol": "HNSBTC",             // Symbol
  "kline": {
    "interval": "1m",             // Interval duration
    "isClosed": false,            // Is this kline still being updated?
    "openTime": 1557190800000,    // Kline start time
    "closeTime": 1557190859999,   // Kline close time
    "openPrice": "0.00002247",    // Open price
    "highPrice": "0.00002256",    // High price
    "lowPrice": "0.00002243",     // Low price
    "closePrice": "0.00002253",   // Close price
    "volume": "10.001301",        // Total base asset traded in interval
    "quoteVolume": "0.000224824", // Total quote asset traded in interval
    "numberOfTrades": 42,         // Number of trades
    "firstTradeId": 13001,        // First trade id	
    "lastTradeId": 13042,         // Last trade id
  }
}
```

## Price ticker
This feed provides statistics for the last 24 hours of trading activity for a HNSBTC, every second. Importantly, these statistics are taken over a _rolling_ window and not over a calendar day. Thus, a request sent on `2019/05/05 16:01:33` will return statistics for trades that occurred between `2019/05/04 16:01:34` and `2019/05/05 16:01:33`. The seconds are inclusive on both ends.

**Feed name:**  '/ws/v0/ticker/day'

**Payload:**
```javascript
{
  "eventType": "24hr",                        // Event type
  "eventTime": 1555553960000,                 // Event time
  "symbol": "HNSBTC",                         // Symbol
  "volumeWeightedAveragePrice": "0.00001959", // Volume weighted average price
  "priceChange": "0.00000019",                // Price change
  "priceChangePercent": "0.8528",             // Price change percent
  "openPrice": "0.00002228",                  // Open price
  "highPrice": "0.00002247",                  // High price
  "lowPrice": "0.00001414",                   // Low price
  "closePrice": "0.00002247",                 // Close price
  "volume": "11413.935399",                   // Total base asset traded in interval
  "quoteVolume": "0.22363732",                // Total quote asset traded in interval
  "openTime": 1555467560001,                  // Time these statistics began (inclusive)
  "closeTime": 1555553960000,                 // Time these statistics end (inclusive)
  "firstTradeId": 19761,                      // First trade id
  "lastTradeId": 20926,                       // Last trade Id
  "numberOfTrades": 1166                      // Total number of trades
  "bestBidPrice": "0.00002079",               // (TODO) Best bid price
  "bestBidQty": "10.130000",                  // (TODO) Best bid quantity
  "bestAskPrice": "0.00002091",               // (TODO) Best ask price
  "bestAskQty": "32.901102",                  // (TODO) Best ask quantity
}
```

## Depth ticker
This feed allows you to mirror the Namebase order book on your computer. It sends price and quantity updates every second. Note: this API provides no guarantees on the order the price levels appear in. Do not assume that bids and asks are sorted in decreasing order by price.

**Feed name:** '/ws/v0/ticker/depth'

**Payload:**
```javascript
{
  "eventType": "depthUpdate",   // Event type
  "eventTime": 1555553960000,   // Event time
  "symbol": "HNSBTC",           // Symbol
  "firstEventId": 256,          // First event id
  "lastEventId": 258,           // Last event id
  "asks": [
    ["0.00002091", "32.901102"] // [Price level to change, New quantity]
  ],
  "bids": [
    ["0.00002079", "10.130000"] // [Price level to change, New quantity]
  ]
}
```

### Buffering depth updates
Since this feed only provides updates (it's a differential feed), you need to obtain the current orderbook state by querying the `GET /api/v0/depth` REST endpoint. However, it's important that you don't do this _first_.

To properly maintain a local copy of the Namebase orderbook, begin by connecting to the depth feed and storing all of the depth update events in a buffer. Only once you've successfully connected should you query the REST endpoint to get the current orderbook.

With this in hand, you can review your buffer and apply all of the updates that happend _after_ the REST response's `lastEventId`. This procedure ensures that you do not miss any events.
