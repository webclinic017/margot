[![version](https://img.shields.io/pypi/v/margot)](https://pypi.org/project/margot/)
![python](https://img.shields.io/pypi/pyversions/margot)
![wheel](https://img.shields.io/pypi/wheel/margot)
[![license](https://img.shields.io/github/license/atkinson/margot)](https://github.com/atkinson/margot/blob/master/LICENSE)
[![build](https://img.shields.io/travis/com/atkinson/margot)](https://travis-ci.com/github/atkinson/margot)
[![Documentation Status](https://readthedocs.org/projects/margot/badge/?version=latest)](https://margot.readthedocs.io/en/latest/?badge=latest)
[![codecov](https://codecov.io/gh/atkinson/margot/branch/master/graph/badge.svg)](https://codecov.io/gh/atkinson/margot)

# An algorithmic trading framework for pydata.
Margot is a library of components that may be used together or separately. The first
major component is now available for public preview. It should be considered early-beta.

- margot.data

# Margot Data
Margot data makes it easy to create neat and tidy dataframes.

Margot manages data collection, caching, cleaning, time-series feature generation and
Pandas Dataframe organisaiton and management using a clean, declarative API. If you've
ever used Django you'll find this approach similar to the Django ORM.

## Columns
The heart of a time-series dataframe is the original data. Margot can retreive time series
data from external sources (currently AlphaVantage). To add a time series such as
"closing_price" or "volume", we declare a Column.

e.g. to get closing_price and volume from AlphaVantage:

    adjusted_close = av.Column(function='historical_daily_adjusted', 
                               column='adjusted_close')

    daily_volume = av.Column(function='historical_daily_adjusted',
                             column='volume')

## Features
Columns are useful, but we usually want to derive new time series from them, such 
as "log_returns" or "SMA20". Margot does this for you; we've called these derived
time-series, Features.

    simple_returns = feature.SimpleReturns(column='adjusted_close')
    log_returns = feature.LogReturns(column='adjusted_close')
    sma20 = feature.SimpleMovingAverage(column='adjusted_close', window=20)

Features can be piled on top of one another. For example, to create a time series of
realised volatility based on log_returns with a lookback of 30 trading days, simply
add the following feature:

    realised_vol = feature.RealisedVolatility(column='log_returns', window=30)

Margot includes many common financial Features, and we'll be adding more soon. It's 
also very easy to add your own.


## Symbols
Often, you want to make a dataframe combining a number of columns and features.
Margot makes this very easy by providing the Symbol class e.g.

    class MyEquity(Symbol):

        adjusted_close = av.Column(function='historical_daily_adjusted', 
                                   column='adjusted_close')
        log_returns = feature.LogReturns(column='adjusted_close')
        realised_vol = feature.RealisedVolatility(column='log_returns', 
                                                  window=30)
        upper_band = feature.UpperBollingerBand(column='adjusted_close', 
                                                window=20, 
                                                width=2.0)
        sma20 = feature.SimpleMovingAverage(column='adjusted_close', 
                                            window=20)
        lower_band = feature.LowerBollingerBand(column='adjusted_close', 
                                                window=20, 
                                                width=2.0)

    spy = MyEquity(symbol='SPY')

## MargotDataFrames
You usually you want to look at more than one symbol. That's where
ensembles come in. MargotDataFrame really brings power to margot.data.

    class MyEnsemble(MargotDataFrame):
        spy = Equity(symbol='SPY')
        iwm = Equity(symbol='IWM')
        spy_iwm_ratio = Ratio(numerator=spy.adjusted_close, 
                              denominator=iwm.adjusted_close,
                              label='spy_iwm_ratio')

    my_df = MyEnsemble().to_pandas() 

The above code creates a Pandas DataFrame of both equities, and an additional
feature that calculates a time-series of the ratio of their respective
adjusted close prices.

# Margot's other parts
**not yet released.**

Margot also provides a simple framework for writing and backtesting trading
signal generation algorithms using margot.data.

Results from margot's trading algorithms can be analysed with pyfolio.

## Getting Started

    pip install margot

Next you need to make sure you have a couple of environment variables set:

    export ALPHAVANTAGE_API_KEY=YOUR_API_KEY
    export DATA_CACHE=PATH_TO_FOLDER_TO_STORE_HDF5_FILES

Once you've done that, try running the code in the [notebook](https://margot.readthedocs.io/en/latest/notebook.margot.data.html).

## Status
This is still an early stage software project, and should not be used for live trading.

## Documentation

in progress - for examples see the [notebook](https://margot.readthedocs.io/en/latest/notebook.margot.data.html).

## Contributing

Feel free to make a pull request or chat about your idea first using [issues](https://github.com/atkinson/margot/issues).

Dependencies are kept to a minimum. Generally if there's a way to do something in the standard library (or numpy / Pandas), let's do it that way rather than add another library. 

## License
Margot is licensed for use under Apache 2.0. For details see [the License](https://github.com/atkinson/margot/blob/master/LICENSE).
