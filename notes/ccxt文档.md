# Exchange 交易所

['binance', 'fcoin', 'gateio', 'huobi', 'kucoin', 'okex']

['BTC', 'ETH', 'XRP', 'BCH', 'EOS', 'XLM', 'LTC', 'ADA', 'XMR', 'TRX', 'BNB', 'ONT', 'NEO', 'DCR']

https://github.com/ccxt/ccxt/wiki/Manual#markets


The structure of the library can be outlined as follows:

                                 User
    +-------------------------------------------------------------+
    |                            CCXT                             |
    +------------------------------+------------------------------+
    |            Public            |           Private            |
    +=============================================================+
    │                              .                              |
    │                    The Unified CCXT API                     |
    │                              .                              |
    |       loadMarkets            .           fetchBalance       |
    |       fetchMarkets           .            createOrder       |
    |       fetchCurrencies        .            cancelOrder       |
    |       fetchTicker            .             fetchOrder       |
    |       fetchTickers           .            fetchOrders       |
    |       fetchOrderBook         .        fetchOpenOrders       |
    |       fetchOHLCV             .      fetchClosedOrders       |
    |       fetchTrades            .          fetchMyTrades       |
    |                              .                deposit       |
    |                              .               withdraw       |
    │                              .                              |
    +=============================================================+
    │                              .                              |
    |                     Custom Exchange API                     |
    |         (Derived Classes And Their Implicit Methods)        |
    │                              .                              |
    |       publicGet...           .          privateGet...       |
    |       publicPost...          .         privatePost...       |
    |                              .          privatePut...       |
    |                              .       privateDelete...       |
    |                              .                   sign       |
    │                              .                              |
    +=============================================================+
    │                              .                              |
    |                      Base Exchange Class                    |
    │                              .                              |
    +=============================================================+

实例化交易所

```
# Python
import ccxt
exchange = ccxt.okcoinusd () # default id
okcoin1 = ccxt.okcoinusd ({ 'id': 'okcoin1' })
okcoin2 = ccxt.okcoinusd ({ 'id': 'okcoin2' })
id = 'btcchina'
btcchina = eval ('ccxt.%s ()' % id)
gdax = getattr (ccxt, 'gdax') ()

# from variable id
exchange_id = 'binance'
exchange_class = getattr(ccxt, exchange_id)
exchange = exchange_class({
    'apiKey': 'YOUR_API_KEY',
    'secret': 'YOUR_SECRET',
    'timeout': 30000,
    'enableRateLimit': True,
})

```

--------------------------------------------------------------------------------

# Exchange Structure 交易所结构
Every exchange has a set of properties and methods, most of which you can override by passing an associative array of params to an exchange constructor. You can also make a subclass and override everything.

Here's an overview of base exchange properties with values added for example:

```
{
    'id':   'exchange'                  // lowercase string exchange id
    'name': 'Exchange'                  // human-readable string
    'countries': [ 'US', 'CN', 'EU' ],  // array of ISO country codes
    'urls': {
        'api': 'https://api.example.com/data',  // string or dictionary of base API URLs
        'www': 'https://www.example.com'        // string website URL
        'doc': 'https://docs.example.com/api',  // string URL or array of URLs
    },
    'version':         'v1',            // string ending with digits
    'api':             { ... },         // dictionary of api endpoints
    'has': {                            // exchange capabilities
        'CORS': false,
        'publicAPI': true,
        'privateAPI': true,
        'cancelOrder': true,
        'createDepositAddress': false,
        'createOrder': true,
        'deposit': false,
        'fetchBalance': true,
        'fetchClosedOrders': false,
        'fetchCurrencies': false,
        'fetchDepositAddress': false,
        'fetchMarkets': true,
        'fetchMyTrades': false,
        'fetchOHLCV': false,
        'fetchOpenOrders': false,
        'fetchOrder': false,
        'fetchOrderBook': true,
        'fetchOrders': false,
        'fetchTicker': true,
        'fetchTickers': false,
        'fetchBidsAsks': false,
        'fetchTrades': true,
        'withdraw': false,
    },
    'timeframes': {                     // empty if the exchange !has.fetchOHLCV
        '1m': '1minute',
        '1h': '1hour',
        '1d': '1day',
        '1M': '1month',
        '1y': '1year',
    },
    'timeout':          10000,          // number in milliseconds
    'rateLimit':        2000,           // number in milliseconds
    'userAgent':       'ccxt/1.1.1 ...' // string, HTTP User-Agent header
    'verbose':          false,          // boolean, output error details
    'markets':         { ... }          // dictionary of markets/pairs by symbol
    'symbols':         [ ... ]          // sorted list of string symbols (traded pairs)
    'currencies':      { ... }          // dictionary of currencies by currency code
    'markets_by_id':   { ... },         // dictionary of dictionaries (markets) by id
    'proxy': 'https://crossorigin.me/', // string URL
    'apiKey':   '92560ffae9b8a0421...', // string public apiKey (ASCII, hex, Base64, ...)
    'secret':   '9aHjPmW+EtRRKN/Oi...'  // string private secret key
    'password': '6kszf4aci8r',          // string password
    'uid':      '123456',               // string user id
}

```




--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Markets 市场交易对信息
Each exchange is a place for trading some kinds of valuables. Sometimes they are called with various different terms like instruments, symbols, trading pairs, currencies, tokens, stocks, commodities, contracts, etc, but they all mean the same – a trading pair, a symbol or a financial instrument.

In terms of the ccxt library, every exchange offers multiple markets within itself. The set of markets differs from exchange to exchange opening possibilities for cross-exchange and cross-market arbitrage. A market is usually a pair of traded crypto/fiat currencies.

Market Structure

```
{
    'id':     'btcusd',   // string literal for referencing within an exchange
    'symbol': 'BTC/USD',  // uppercase string literal of a pair of currencies
    'base':   'BTC',      // uppercase string, base currency, 3 or more letters
    'quote':  'USD',      // uppercase string, quote currency, 3 or more letters
    'active': true,       // boolean, market status
    'precision': {        // number of decimal digits "after the dot"
        'price': 8,       // integer
        'amount': 8,      // integer
        'cost': 8,        // integer
    },
    'limits': {           // value limits when placing orders on this market
        'amount': {
            'min': 0.01,  // order amount should be > min
            'max': 1000,  // order amount should be < max
        },
        'price': { ... }, // same min/max limits for the price of the order
        'cost':  { ... }, // same limits for order cost = price * amount
    },
    'info':      { ... }, // the original unparsed market info from the exchange
}

```


gate_markets['BTC/USDT']


Naming Consistency
There is a bit of term ambiguity across various exchanges that may cause confusion among newcoming traders. Some exchanges call markets as pairs, whereas other exchanges call symbols as products. In terms of the ccxt library, each exchange contains one or more trading markets. Each market has an id and a symbol. Most symbols are pairs of base currency and quote currency.

Exchanges → Markets → Symbols → Currencies

remember the following rule: base is always before the slash, quote is always after the slash in any symbol and with any market.

base currency ↓
             BTC / USDT
             ETH / BTC
            DASH / ETH
                    ↑ quote currency

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# API 接口列表

The endpoints definition is a full list of ALL API URLs exposed by an exchange. This list gets converted to callable methods upon exchange instantiation. Each URL in the API endpoint list gets a corresponding callable method. This is done automatically for all exchanges, therefore the ccxt library supports all possible URLs offered by crypto exchanges.
gate.api['public']['get']
gate.api['private']['post']


Each implicit method gets a unique name which is constructed from the .api definition. For example, a private HTTPS PUT https://api.exchange.com/order/{id}/cancel endpoint will have a corresponding exchange method named .privatePutOrderIdCancel()/.private_put_order_id_cancel(). A public HTTPS GET https://api.exchange.com/market/ticker/{pair} endpoint would result in the corresponding method named .publicGetTickerPair()/.public_get_ticker_pair(), and so on.



Public/Private API
API URLs are often grouped into two sets of methods called a public API for market data and a private API for trading and account access. These groups of API methods are usually prefixed with a word 'public' or 'private'.


Public APIs include the following:

instruments/trading pairs
price feeds (exchange rates)
order books (L1, L2, L3...)
trade history (closed orders, transactions, executions)
tickers (spot / 24h price)
OHLCV series for charting
other public endpoints


Private APIs allow the following:

manage personal account info
query account balances
trade by making market and limit orders
create deposit addresses and fund accounts
request withdrawal of fiat and crypto funds
query personal open / closed orders
query positions in margin/leverage trading
get ledger history
transfer funds between accounts
use merchant services


Unified API
The unified ccxt API is a subset of methods common among the exchanges. It currently contains the following methods:


fetchOrderBook (symbol[, limit = undefined[, params = {}]]): Fetch L2/L3 order book for a particular market trading symbol.
fetchL2OrderBook (symbol[, limit = undefined[, params]]): Level 2 (price-aggregated) order book for a particular symbol.
fetchTrades (symbol[, since[, [limit, [params]]]]): Fetch recent trades for a particular trading symbol.
fetchTicker (symbol): Fetch latest ticker data by trading symbol.

---------------------------------------------------------------------------------------------------------------------------------------

# Market Data 市场行情数据

Order Book / Market Depth
Market Price
Price Tickers
Individually By Symbol
All At Once
OHLCV Candlestick Charts
Public Trades


## Order Book 订单，可以获取买一卖一
gate.fetch_order_book('BTC/USDT')

```
{
'bids': [[6545.05, 0.5], [6544.11, 0.02], [6543.69, 0.5933], [6542.81, 0.4021], [6542.14, 0.5442], [6540.0, 2.38], [6536.27, 0.0601], [6536.25, 0.1], [6530.77, 0.461], [6530.76, 0.6], [6530.71, 0.6], [6530.44, 0.3], [6530.1, 0.0289], [6530.05, 0.2322], [6530.0, 3.2058], [6529.55, 0.265533285], [6529.1, 0.0183], [6528.72, 0.68], [6528.51, 5.0177], [6528.31, 0.007], [6527.5, 0.034], [6524.93, 1.2], [6520.0, 0.0082], [6518.63, 6.1051], [6514.9, 0.91], [6510.58, 0.4959], [6510.5, 0.138237], [6507.63, 0.76822334], [6504.44, 0.41602745], [6504.43, 7.2795]], 
'asks': [[6550.25, 0.5665], [6551.87, 0.0029], [6553.0, 7.5009], [6561.36, 0.054], [6561.37, 0.05], [6568.99, 0.0599], [6569.0, 6.4556], [6571.62, 1.2], [6572.41, 0.007], [6573.53, 0.714], [6575.41, 0.3], [6575.42, 0.9], [6576.2, 0.3], [6577.0, 0.2], [6578.0, 0.08909449], [6579.0, 0.0025], [6583.32, 1.5236206], [6584.18, 6.9668], [6586.67, 0.13], [6598.49, 8.1152], [6598.67, 0.2], [6600.0, 0.031], [6609.28, 12.0], [6610.0, 0.30119], [6612.31, 1.19257268], [6612.32, 0.21], [6617.25, 0.04], [6617.3, 0.0438], [6617.37, 0.0656], [6617.65, 0.1472]], 'timestamp': None, 'datetime': None, 'nonce': None
}
```


The levels of detail or levels of order book aggregation are often number-labelled like L1, L2, L3...

L1: less detail for quickly obtaining very basic info, namely, the market price only. It appears to look like just one order in the order book.
L2: most common level of aggregation where order volumes are grouped by price. If two orders have the same price, they appear as one single order for a volume equal to their total sum. This is most likely the level of aggregation you need for the majority of purposes.
L3: most detailed level with no aggregation where each order is separate from other orders. This LOD naturally contains duplicates in the output. So, if two orders have equal prices they are not merged together and it's up to the exchange's matching engine to decide on their priority in the stack. You don't really need L3 detail for successful trading. In fact, you most probably don't need it at all. Therefore some exchanges don't support it and always return aggregated order books.

Market Price

买一价：bid = orderbook['bids'][0][0] if len (orderbook['bids']) > 0 else None

卖一价：ask = orderbook['asks'][0][0] if len (orderbook['asks']) > 0 else None

差价：spread = (ask - bid) if (bid and ask) else None

时间可能缺失

orderbook['timestamp'] is the time when the exchange generated this orderbook response (before replying it back to you). This may be missing (undefined/None/null), as documented in the Manual, not all exchanges provide a timestamp there. If it is defined, then it is the UTC timestamp in milliseconds since 1 Jan 1970 00:00:00.

exchange.last_response_headers['Date'] is the date-time string of the last HTTP response received (from HTTP headers). The 'Date' parser should respect the timezone designated there. The precision of the date-time is 1 second, 1000 milliseconds. This date should be set by the exchange server when the message originated according to the following standards:

## Price Tickers 行情

A price ticker contains statistics for a particular market/symbol for some period of time in recent past, usually last 24 hours. The structure of a ticker is as follows:
gate.fetch_ticker('BTC/USDT')

```
{
    'symbol':        string symbol of the market ('BTC/USD', 'ETH/BTC', ...)
    'info':        { the original non-modified unparsed reply from exchange API },
    'timestamp':     int (64-bit Unix Timestamp in milliseconds since Epoch 1 Jan 1970)
    'datetime':      ISO8601 datetime string with milliseconds
    'high':          float, // highest price
    'low':           float, // lowest price
    'bid':           float, // current best bid (buy) price
    'bidVolume':     float, // current best bid (buy) amount (may be missing or undefined)
    'ask':           float, // current best ask (sell) price
    'askVolume':     float, // current best ask (sell) amount (may be missing or undefined)
    'vwap':          float, // volume weighed average price
    'open':          float, // opening price
    'close':         float, // price of last trade (closing price for current period)
    'last':          float, // same as `close`, duplicated for convenience
    'previousClose': float, // closing price for the previous period
    'change':        float, // absolute change, `last - open`
    'percentage':    float, // relative change, `(change/open) * 100`
    'average':       float, // average price, `(last + open) / 2`
    'baseVolume':    float, // volume of base currency traded for last 24 hours
    'quoteVolume':   float, // volume of quote currency traded for last 24 hours
}

```


ohlcv是open,high,low,close and volume

The main purpose of a ticker is to serve statistical data, as such, treat it as "live 24h OHLCV".

To get historical prices and volumes use the unified fetchOHLCV method where available.

All At Once

Some exchanges (not all of them) also support fetching all tickers at once.
gate.fetch_tickers()

## OHLCV Candlestick Charts 开高低收和成交量

gate.fetch_ohlcv('BTC/USDT')

## Trades, Executions, Transactions 成交记录


You can call the unified fetchTrades / fetch_trades method to get the list of most recent trades for a particular symbol. 
gate.fetch_trades('BTC/USDT')

```
[
    {
        'info':       { ... },                  // the original decoded JSON as is
        'id':        '12345-67890:09876/54321', // string trade id
        'timestamp':  1502962946216,            // Unix timestamp in milliseconds
        'datetime':  '2017-08-17 12:42:48.000', // ISO8601 datetime with milliseconds
        'symbol':    'ETH/BTC',                 // symbol
        'order':     '12345-67890:09876/54321', // string order id or undefined/None/null
        'type':      'limit',                   // order type, 'market', 'limit' or undefined/None/null
        'side':      'buy',                     // direction of the trade, 'buy' or 'sell'
        'price':      0.06917684,               // float price in quote currency
        'amount':     1.5,                      // amount of base currency
    },
    ...
]

```

# Trading 个人交易


# Proxy 代理
The python version of the library uses the python-requests package for underlying HTTP and supports all means of customization available in the requests package, including proxies.
修改环境变量

You can configure proxies by setting the environment variables HTTP_PROXY and HTTPS_PROXY.

```
$ export HTTP_PROXY="http://10.10.1.10:3128"
$ export HTTPS_PROXY="http://10.10.1.10:1080"
```

代码指定

```
import ccxt
exchange = ccxt.poloniex({
    'proxies': {
        'http': 'http://10.10.1.10:3128',  # these proxies won't work for you, they are here for example
        'https': 'https://10.10.1.10:1080',
    },
})
```
Or

```
import ccxt
exchange = ccxt.poloniex()
exchange.proxies = {
  'http': 'http://10.10.1.10:3128', # these proxies won't work for you, they are here for example
  'https': 'https://10.10.1.10:1080',
}
```
python修改环境变量

```
os.environ.setdefault('http_proxy', 'http://127.0.0.1:1080')
os.environ.setdefault('https_proxy', 'http://127.0.0.1:1080')
```


