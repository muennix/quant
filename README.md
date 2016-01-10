IbHandler - An easy-to-use interface to Interactive Brokers trading system
==============================================================================
released 10 Jan 2016

About
------------------------------------------------------------------------------
IbHandler implements the IbPy wrapper of the Interactive Brokers API [1] with recent improvements of Bryce Lampe [2]. It provides a simple way of of communicating to Trader Workstation or the IB Gateway through Python.

Installation
------------------------------------------------------------------------------
IbHandler requires Python 2.6 or newer. The easiest way of installing is by cloning (or downloading) this repository 

	git clone https://github.com/muennix/IbHandler.git

and installing the Python module by running

	Python setup.py install

The above command should atomically download and install the IbPy module in case you  have not already installed it.


How to use IbHandler?
------------------------------------------------------------------------------
### Basics

It's simple. Here are some examples. First, let's create an instance:

	import IbHandler
	mxh = IbHandler.mxIBhandler(account="UXXXXXXX")
	
Let's check the liquid funds available in a given currency, e.g. USD:

	mxh.get_available_cash(currency = "USD")
	
Now, let's see what the current BID price of Apple looks like

	mxh.getprice("AAPL",pricetype="BID")
	
or let's check multiple prices at once

	symbols = ["AAPL","GOOG","NFLX","MSFT","IBM"]
	mxh.getprices(symbols,timeout=10, pricetype="BID")
	
you can get hundreds of prices this way. They are automatically split up in blocks of 50 symbols that are requested at once. If you want to increase this number, e.g., because your account allows a larger number of concurrent lines of real-time market data, you can do it by
	
	mxh.getprices(symbols,timeout=10, pricetype="BID", maxdatalines = 100)
	
If you want to specify the primary exchange on which the market data should be gathered, use the primExchDict parameter to provide a dict with Symbol->Exchange tuples. Please refer to the source code for more information.

### Placing orders
WARNING: Although I am using this interface in a productive environment, it still in beta and it might not work properly in your envorement. Please test extensively with a paper trading account before placing orders on an actual account!

In order to place an order, we first create a contract:

	contract = IbHandler.makeStkContract("GOOG")

Currency in USD by default and exchange is set to SMART. If you want to change this, you can do so by

	contract.m_currency = 'EUR'
	contract.m_exchange = 'IBIS'	

or create the contract by calling ib.ext.Contract() directly

Now, let's buy 100 stocks Google with a limit price of $714.23.

	mxh.place_limitorder(contract,100,714.23,"BUY")
	
If you want to provide your own identifier for logging purposes (see below), you can do it by

	mxh.place_limitorder(contract,100,714.23,"BUY",unique_ID=123)
	
otherwise, the current system timestamp in ms resolution is used.

Next step: Market order. Lets by Google at the current market price, but as a precaution, only execute the order if the current price is above $700. The current BID quote is gathered just before executing the trade

	mxh.place_order_quote(contract,100,700.,"BUY")
	
If you would like to override this precaution and hist buy the stock "blind", you can do so by 

	mxh.place_order_quote(contract,100,0,"BUY")

At any point, open or partially executed orders are stored in

	mxh.openorders

###Advanced orders
Now, let's set the price in the middle of the bid/ask spread, but the order should not be placed if the current BID price is under $700

	mxh.place_limitorder_quote(contract,100,700,"BUY",ba_offset=0.5)

The order is automatically readjusted regarding ba_offest 5 times in total every 15 seconds. If you want to change that, do initialize by

	mxh = IbHandler.mxIBhandler(account="UXXXXXXX", limit_adjust_interval = 15, max_adjust_time=10)

You can manually trigger the readjusting of orders placed by place_limitorder_quote by calling 

	mxh.adjust_limits()

###Further order types: 

Pegged MID (exchange is automatically changed to ISLAND):

	mxh.place_peggedmid_order(contract,100,713,"BUY")

Market on close order, must be submitted 10 to 15 minutes before market close, depending on the exchange
	
	mxh.place_moc_order(contract,100,"BUY")

###Logging

The module implements the logging class. Default log-level is INFO, to change that, initialize by 

	import logging
	mxh = IbHandler.mxIBhandler(account= "UXXXXXXX", 
	loglevel = logging.DEBUG)

The logging instance is located at mxh.logger in case you want to modify it directly, e.g., to divert the output to a file. 

[1]: https://code.google.com/p/ibpy/
[2]: https://github.com/blampe/IbPy