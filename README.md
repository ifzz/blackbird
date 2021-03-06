<p align="center">
<img src="https://cloud.githubusercontent.com/assets/11370278/10808535/02230d46-7dc3-11e5-92d8-da15cae8c6e9.png" width="50%" alt="Blackbird Bitcoin Arbitrage">
</p>

### Introduction

Blackbird Bitcoin Arbitrage is a C++ trading system that does automatic long/short arbitrage between Bitcoin exchanges.

### How It Works

Bitcoin is still a new and inefficient market. Several Bitcoin exchanges exist around the world and the bid/ask prices they propose can be briefly different from an exchange to another. The purpose of Blackbird is to automatically profit from these temporary price differences while being market-neutral.

Here is a real example where an arbitrage opportunity exists between Bitstamp (long) and Bitfinex (short):

<p align="center">
<img src="https://cloud.githubusercontent.com/assets/11370278/11164055/5863e750-8ab3-11e5-86fc-8f7bab6818df.png"  width="60%" alt="Spread Example">
</p>

At the first vertical line, the spread between the exchanges is high so Blackbird buys Bitstamp and short sells Bitfinex. Then, when the spread closes (second vertical line), Blackbird exits the market by selling Bitstamp and buying Bitfinex back.

#### Advantages

Unlike other Bitcoin arbitrage systems, Blackbird doesn't sell but actually _short sells_ Bitcoin on the short exchange. This feature offers two important advantages:

1. The strategy is always market-neutral: the Bitcoin market's moves (up or down) don't impact the strategy returns. This removes a huge risk from the strategy. The Bitcoin market could suddenly lose twice its value that this won't make any difference in the strategy returns.

2. The strategy doesn't need to transfer funds (USD or BTC) between Bitcoin exchanges. The buy/sell and sell/buy trading activities are done in parallel on two different exchanges, independently. Advantage: no need to deal with transfer latency issues.

### Disclaimer

__USE THE SOFTWARE AT YOUR OWN RISK. YOU ARE RESPONSIBLE FOR YOUR OWN MONEY. PAST PERFORMANCE IS NOT NECESSARILY INDICATIVE OF FUTURE RESULTS.__

__THE AUTHORS AND ALL AFFILIATES ASSUME NO RESPONSIBILITY FOR YOUR TRADING RESULTS.__

### Code Information

The trade results are stored in CSV files and the detailed activity is stored in log files. New files are created every time Blackbird is started.

It is possible to automatically stop Blackbird after the next trade has closed by creating, at any time, an empty file named _stop_after_exit_.

Blackbird uses functions written by <a href="http://www.adp-gmbh.ch/cpp/common/base64.html" target="_target">René Nyffenegger</a> to encode and decode base64.

### How To Test Blackbird

Please make sure that you understand the disclaimer above if you want to test Blackbird with real money, and start with a small amount of money.

__IMPORTANT: all your BTC accounts must be empty before starting Blackbird. Make sure that you only have USD on your accounts and no BTC.__

It is never entirely safe to just tell Blackbird to use, say, $25 per exchange (CashForTesting=25.00). You also need to only have $25 available on each of your trading accounts as well as 0 BTC. In this case you are sure that even with a bug with CashForTesting your maximum loss on an exchange won't be greater than $25 no matter what.

#### Covered Exchanges

| Exchange | Long | Short | Implemented | Tested  | Note |
| -------- |:----:|:-----:|:-----------:|:-------:| ---- |
| <a href="https://www.bitfinex.com" target="_blank">Bitfinex</a> | ✓ | ✓ | ✓ | ✓ | |
| <a href="https://www.okcoin.com" target="_blank">OKCoin</a> | ✓ |  | ✓ | ✓ |their API now offers short selling: <a href="https://www.okcoin.com/about/rest_api.do" target="_blank">link here</a> |
| <a href="https://www.bitstamp.net" target="_blank">Bitstamp</a> | ✓ |  | ✓ | ✓ | |
| <a href="https://gemini.com" target="_blank">Gemini</a> | ✓ |  | ✓ | ✓ | |
| <a href="https://www.kraken.com" target="_blank">Kraken</a> | ✓ |  | ✓ |  | Validation in progress |
| <a href="https://796.com/?lang=en" target="_blank">796.com</a> | ✓ |  | ✓ |  | Bitcoin futures. Troubleshooting in progress |
| <a href="https://poloniex.com" target="_blank">Poloniex</a> | ✓ | ✓ |  |  | |
| <a href="https://btc-e.com" target="_blank">BTC-e</a> | ✓ |  |  |  | |
| <a href="https://www.itbit.com" target="_blank">itBit</a> | ✓ |  |  |  | |

#### Blackbird Parameters

Parameter | Default Value | Description
| ------------ | ------------------- | ------------- |
| DemoMode | true | The demo mode will show the spreads but won't actually trade anything |
| UseFullCash | false | When true, all the USD available on your accounts will be used. Otherwise, the amount defined by CashForTesting will be used. Note: the cash used for a trade will be the minimum of the two exchanges, minus 1.00% as a small margin: if there is $1,000 on the first account and $1,100 on the second one, $990 will be used on each exchange, i.e. $1,000 - (1.00% * $1,000). The total exposure will be $1,980 |
| CashForTesting | $25.00 | If UseFullCash is false, that parameter defines the USD amount that will be used. The minimum has to be $10 otherwise some exchanges might reject the orders |
| MaxExposure | $25,000.00 | Maximum exposure per exchange. If the limit is $25,000 then Blackbird won't send any order larger than that on each exchange, regardless of the current balances |
| MaxLength | 5,184,000 | The maximum length of a trade in number of iterations. If this value is reached then Blackbird will exit the market regardless of the spread. Warning: with this value the system can exit with a loss so It's recommended to use a large value. The default is 180 days with GapSec at 3 seconds, or 120 days with GapSec at 2 seconds |
| DebugMaxIteration | 3,200,000 | The maximum number of iteration. Once DebugMaxIteration is reached Blackbird is terminated with return=0. Useful to troubleshoot memory leaks for example |
| Verbose | true | Write the bid/ask and the spreads to the log file at every iteration. The log file size will be larger but it will show how Blackbird analyses the spreads |
| GapSec | 3 seconds | Time lapse in seconds of an iteration. By default the quotes download, the spread computations and the opportunity analysis for all the exchanges are done every 3 seconds  |
| SpreadEntry | 0.0080 | The spread threshold above which the trailing spreads are generated to capture an arbitrage opportunity |
| SpreadTarget | 0.0050 | This is the targeted profit. It represents the net profit and takes the exchange fees into account. If SpreadEntry is at 0.80% and trades are generated at that level on two exchanges with 0.25% fees each, Blackbird will set the exit threshold at -0.70% (0.80% spread entry - 4x0.25% fees - 0.50% target = -0.70% exit threshold) |
| PriceDeltaLimit | 0.10 | The maximum difference between the target limit price and the computed limit price of an order. That is the price generated by looking at the current liquidity in the order books. If the difference is greater than PriceDeltaLimit then no trades will be generated because there is not enough liquidity at that price and there is a risk of slippage |
| TrailingSpreadLim | 0.0008 | The limit under which the trailing spread is generated. If the current spread is above SpreadTarget and at 0.70%, then by default the trailing spread will be generated at 0.62% |
| TrailingSpreadCount | 1 |  The number of time the spread must be between SpreadTarget and the trailing spread before sending the orders to the market. By default the system waits only one iteration |
| OrderBookFactor | 3.0 | In order to be executed as fast as possible and avoid slippage, Blackbird checks the liquidity in the order books of the exchanges and makes sure there is at least 3.0 times the needed liquidity before executing the order |
| UseVolatility | false |  If true, display the spreads volatility infirmation in the log file. This is not used for the moment and only displayed as information |
| VolatilityPeriod | 600 | The period length of the volatility in number of iterations. This is not used for the moment and only displayed as information |
| SendEmail | false | When true, an e-mail will be sent every time an arbitrage trade is completed, with information such as the names of the exchanges and the trade return |
| UseDatabase | false | When true,  the bid/ask information of the exchanges will be stored into a MySQL database for reference |

#### Credentials

For each of your exchange accounts you need to create the API authentication keys. This is usually done in the _Settings_ section of your accounts.

Then, you need to add your API keys into the file _blackbird.conf_. You need at least two exchanges and one of them should allow short selling. __Never__ share this file as it will contain your personal exchange credentials!

### Run the software

You need the following libraries: <a href="https://www.openssl.org/docs/crypto/crypto.html" target="_blank">Crypto</a>, <a href="http://www.digip.org/jansson" target="_blank">Jansson</a>, <a href="http://curl.haxx.se" target="_blank">cURL</a>, <a href="http://dev.mysql.com/doc" target="_blank">MySQL Client</a> and <a href="http://caspian.dotconf.net/menu/Software/SendEmail" target="_blank">sendEmail</a>. Usually this is what you need to install:

```
libssl-dev
libjansson-dev
libcurl4-openssl-dev
libmysqlclient-dev
sendemail
```

__Note:__ you need Jansson version 2.7 minimum otherwise you will get the following compilation error:

`'json_boolean_value' was not declared in this scope`


Once you have downloaded the source code, build Blackbird by typing:

`make`

Then start it by typing:

`./blackbird`

### Tasks And Issues

Please check the <a href="https://github.com/butor/blackbird/issues" target="_blank">issues page</a> for the current tasks and issues. If you face any problems with Blackbird please open a new issue on that page.

### License

* <a href="https://en.wikipedia.org/wiki/MIT_License" target="_blank">MIT</a>

### Contact

* If you found a bug, please open a new <a href="https://github.com/butor/blackbird/issues" target="_blank">issue</a> with the label _bug_
* If you have a general question or have troubles running Blackbird, you can open a new  <a href="https://github.com/butor/blackbird/issues" target="_blank">issue</a> with the label _question_ or _help wanted_
* For anything else you can contact me at julien.hamilton@gmail.com or on <a href="https://www.linkedin.com/in/julienhamilton" target="_blank">LinkedIn</a>.

### Recent Changes

* Fixed a segmentation fault that occurs when the credentials are wrong
* At least $10 of exposure is now needed with real trading, otherwise orders will be rejected by some exchanges
* Array with the parameters in the README file

### Log Output Example

This is what the log file looks like when Blackbird is started:


```
Blackbird Bitcoin Arbitrage
DISCLAIMER: USE THE SOFTWARE AT YOUR OWN RISK.

[ Targets ]
   Spread Entry:  0.80%
   Spread Target: 0.30%

[ Current balances ]
   Bitfinex:    1,857.79 USD    0.000000 BTC
   OKCoin:      1,801.38 USD    0.000436 BTC
   Bitstamp:    1,694.15 USD    0.000000 BTC
   Gemini:      1,720.38 USD    0.000000 BTC

[ Cash exposure ]
   FULL cash used!

[ 10/31/2015 08:32:45 ]
   Bitfinex:    325.21 / 325.58
   OKCoin:      326.04 / 326.10
   Bitstamp:    325.37 / 325.82
   Gemini:      325.50 / 328.74
   ----------------------------
   OKCoin/Bitfinex:     -0.27% [target  0.80%, min -0.27%, max -0.27%]
   Bitstamp/Bitfinex:   -0.19% [target  0.80%, min -0.19%, max -0.19%]
   Gemini/Bitfinex:     -1.07% [target  0.80%, min -1.07%, max -1.07%]

[ 10/31/2015 08:32:48 ]
   Bitfinex:    325.21 / 325.58
   OKCoin:      326.04 / 326.10
   Bitstamp:    325.39 / 325.68
   Gemini:      325.50 / 328.67
   ----------------------------
   OKCoin/Bitfinex:     -0.27% [target  0.80%, min -0.27%, max -0.27%]
   Bitstamp/Bitfinex:   -0.14% [target  0.80%, min -0.19%, max -0.14%]
   Gemini/Bitfinex:     -1.05% [target  0.80%, min -1.07%, max -1.05%]
```

