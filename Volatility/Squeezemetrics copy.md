[sqzme](https://squeezemetrics.com/monitor) [~~~~~~~~](https://squeezemetrics.com/monitor/docs#)

## Documentation

* * *

1. [The Four Axes](https://squeezemetrics.com/monitor/docs#axes)

1. [P (price-trend)](https://squeezemetrics.com/monitor/docs#axes1)
2. [V (volatility-trend)](https://squeezemetrics.com/monitor/docs#axes2)
3. [G (gamma-ratio)](https://squeezemetrics.com/monitor/docs#axes3)
4. [D (dark-ratio)](https://squeezemetrics.com/monitor/docs#axes4)

3. [The Chart](https://squeezemetrics.com/monitor/docs#charts)
4. [The Sheets (and the API)](https://squeezemetrics.com/monitor/docs#sheets)
5. [The Ideas Page](https://squeezemetrics.com/monitor/docs#research)

## I. The Four Axes

There are four indicators that work very nicely together— _(a) price-trend, (b) volatility-trend, (c) gamma-ratio,_ and _(d) dark-ratio_. Let's describe each as clearly as possible.

#### _P_ (price-trend)

![](https://squeezemetrics.com/monitor/static/images/P.png)
We need a de-trended variable here to show us whether price has been rising or falling recently. Are we trending up or down, and by how much?

Importantly, we want it to be volatility-adjusted, so that a small absolute upside or downside move in conditions of low realized volatility registers the same as a large upside or downside move in high realized volatility. Here's how we do that.

![Hand-written derivation of the price indicator](https://squeezemetrics.com/monitor/static/images/Pmath.svg)

```

#!/usr/bin/python3

import numpy as np

def period_change(a, n=1):
    chg = ((a - np.roll(a, n)) / np.roll(a, n))
    chg[:n] = np.nan
    return chg

ccr = period_change(closes, 1)		# Day-to-day % changes, 0th = nan
ccv = np.abs(ccr)			# Same-period absolute-ized (volatility)

def moving_average(a, n=21):
    ret = np.cumsum(a, dtype=float)
    ret[n:] = ret[n:] - ret[:-n]
    ret[:n] = np.nan
    return ret / n

ma   = moving_average(ccr, 21)		# 1-month moving average of % changes
mad  = moving_average(ccv, 21)		# 1-month moving average of abs(%)

P = ma / mad				# "price-trend" indicator


```
The result is an indicator that oscillates primarily between +1 and –1, where +1 suggests that nearly every daily move in the past month was up (–1 indicates down). Since we're normalizing by volatility, this indicator is comparable between time periods as well as between assets.

#### _V_ (volatility-trend)

![](https://squeezemetrics.com/monitor/static/images/V.png)
We need another de-trended variable here to show us whether realized volatility has been rising or falling recently. Are we trending up or down, and by how much?

![Hand-written derivation of the volatility indicator](https://squeezemetrics.com/monitor/static/images/Vmath.svg)

```

#!/usr/bin/python3

import numpy as np

def period_change(a, n=1):
    chg = ((a - np.roll(a, n)) / np.roll(a, n))
    chg[:n] = np.nan
    return chg

ccr = period_change(closes, 1)		# Day-to-day % changes, 0th = nan
ccv = np.abs(ccr)			# Same-period absolute-ized (volatility)

def moving_average(a, n=21):
    ret = np.cumsum(a, dtype=float)
    ret[n:] = ret[n:] - ret[:-n]
    ret[:n] = np.nan
    return ret / n

ma    = moving_average(ccr, 21)		# 1-month moving average of % changes
mad   = moving_average(ccv, 21)		# 1-month moving average of abs(%)
madMA = moving_average(mad, 21)		# 1-month MA of 1-month MA of abs(%)

V = mad - madMAD			# "volatility-trend" indicator


```

The result is an indicator that oscillates primarily between +1 and –1, where the units are exactly the same (1-month average daily moves) as with the P indicator.

When V is positive, realized volatility denominated in average daily moves is trending up; and when V is negative, realized volatility denominated in average daily moves is trending down.

#### _G_ (gamma-ratio)

![](https://squeezemetrics.com/monitor/static/images/G.png)
We want to know where there's more call gamma or put gamma, because we want to know whether convexity is leaning toward upside or downside structures. Are we call-driven or put-driven?

Specifically, we want a constant-volatility BSM-delta–derived number, since we want to use a percent gamma (Pgamma), and we don't want to factor in skew or ignore lower-liquidity options.

![Hand-written derivation of the volatility indicator](https://squeezemetrics.com/monitor/static/images/Gmath.svg)

```

#!/usr/bin/python3

import numpy as np
from scipy.stats import norm

def delta(cp_flag, S, K, T, r, v):
    d1 = (np.log(S/K)+(r+v*v/2.)*T)/(v*np.sqrt(T))
    if cp_flag == 'c' or cp_flag == 'C':
        return norm.cdf(d1)
    else:
        return -norm.cdf(-d1)

call_gamma = 0
for c in calls:
    delta    = delta('C', c['S'], c['K'], c['T'], 0.00, 0.20)
    up_delta = delta('C', c['S']*1.01, c['K'], c['T'], 0.00, 0.20)
    Pgamma = up_delta - delta
    call_gamma += Pgamma * c['OI']	# multiply by open interest

put_gamma = 0
for p in puts:
    delta    = delta('P', p['S'], p['K'], p['T'], 0.00, 0.20)
    down_delta = delta('P', p['S']*0.99, p['K'], p['T'], 0.00, 0.20)
    Pgamma = down_delta - delta
    put_gamma += abs(Pgamma * p['OI'])	# ensure absolute value of put gamma

total_gamma = call_gamma + put_gamma

G = call_gamma / total_gamma		# "gamma-ratio"


```

The result is an indicator that moves between 0 and 1, telling us the proportion of call gamma to total gamma. A G of 0.5 means call and put gamma are balanced. A G of 1.0 means it's all calls (0.0 all puts).

#### _D_ (dark-ratio)

![](https://squeezemetrics.com/monitor/static/images/D.png)
We want to know whether there is more short-selling or long-selling in daily OTC (dark) volume. Generally, short sales are indicative of non-aggressive buying behavior.

![Hand-written derivation of the volatility indicator](https://squeezemetrics.com/monitor/static/images/Dmath.svg)

```

#!/usr/bin/python3

import numpy as np

q_short = np.array(load_column("finra_short.csv"))
q_total = np.array(load_column("finra_total.csv"))

dark_ratio = q_short / q_total

def moving_average(a, n=5):
    ret = np.cumsum(a, dtype=float)
    ret[n:] = ret[n:] - ret[:-n]
    ret[:n] = np.nan
    return ret / n

dark_5dma = moving_average(dark_ratio, 5)	# Nice and smooth, but not too smooth


```

The result is an indicator that moves between 0 and 1, telling us the proportion of short sales in all dark venues over the past week.

## II. The Chart

![](https://squeezemetrics.com/monitor/static/images/docs_thechart.png)

Well, you can just click the "Nearest" button, which runs a k-nearest-neighbor (k-NN) algorithm on each of the four axes to find out which data, historically, is nearest to today's data, and iteratively updates the forecast in the top-right. This is the lazy way.

Or you can take it a step further and try to figure out for yourself what data seems to work and why. To do this, you can click on the "Best" ("Worst") button to see what combination of data has given us the best (worst) performance in the past. Are we currently in a "best" / "worst" scenario? What would it take to get there?

Or you can click-and-drag on the scatterplots yourself, and see how returns change as you home in on a particular combination of data. Add and remove yourself. Take your own slice. What are the _characteristics_ of a given stock? Yes, falling price-trend (P) is bullish for Verizon, but 1-month forward returns _double_ when we combine low P with low G, suggesting that put options play a big role in generating rallies.

Whichever way you want to approach it, there are a few things to keep in mind:

1. Use the context plot at the top of the page to test whether patterns have changed over time, and how. You'll notice that higher prices, listed options, and "meme" status can dramatically change the way a stock trades, and which indicators are more predictive. Try looking at discrete timeframes!
2. By default, the scatterplots are set to look at 21-day (1-month) returns. You can switch that to 5-day (1-week) by clicking on the 5-day dot in the forecast pane.
3. You can change the ticker you're looking at by clicking on the ticker to the left of the stock chart. You can _also_ change the indicators you're looking at by clicking on the letter to the left of the indicator charts—a little drop-down will show up. Right now, we also have IV (1-month ATM straddle) as an option that you can look at.

If there's something not intuitive about any of this, just ask!

## III. The Sheets (and the API)

Every security in our database can be exported as a spreadsheet document (CSV). Here are the columns in those spreadsheets, and some numerical methods for extending the data:

- **DATE**: YYYY-MM-DD
- **TIMESTAMP*: Unix time, seconds from epoch.
- **MEAN**: Robot Jim's mean 1-week forecast, expressed in mean absolute deviation (MAD).
- **MEAN\_SPOT**:Robot Jim's mean 1-week forecast, expressed in stock price.
- **MEDIAN**: Robot Jim's median 1-week forecast, expressed in MAD.
- **VOL**: Robot Jim's mean 1-week volatility forecast, expressed in MAD.
- **VOL\_MEDIAN**:Robot Jim's median 1-week volatility forecast, expressed in MAD.
- **MEAN\_PCT**:Robot Jim's mean 1-week forecast, expressed in percent.
- **MEDIAN\_PCT**:Robot Jim's median 1-week forecast, expressed in percent.
- **VOL\_PCT**:Robot Jim's mean 1-week volatility forecast, expressed in percent.
- **VOL\_MEDIAN\_PCT**:Robot Jim's median 1-week volatility forecast, expressed in percent.
- **P0**:The 'price' predictor, not normalized.
- **V0**:The 'volatility' predictor, not normalized.
- **G0**:The 'gamma' predictor, not normalized.
- **D0**:The 'dark' predictor, not normalized.
- **P**:The 'price' predictor, normalized to \[-1, 1\].
- **V**:The 'volatility' predictor, normalized to \[-1, 1\].
- **G**:The 'gamma' predictor, normalized to \[-1, 1\].
- **D**:The 'dark' predictor, normalized to \[-1, 1\].
- **xx\_MEAN**: Robot Jim's mean 1-week forecast derived exclusively from the 'xx' pair, expressed in mean absolute deviation (MAD).
- **xx\_MEAN\_SPOT**:Robot Jim's mean 1-week forecast derived exclusively from the 'xx' pair, expressed in stock price.
- **xx\_MEDIAN**:Robot Jim's median 1-week forecast derived exclusively from the 'xx' pair, expressed in MAD.
- **xx\_VOL**:Robot Jim's mean 1-week volatility forecast derived exclusively from the 'xx' pair, expressed in MAD.
- **xx\_VOL\_MEDIAN**:Robot Jim's median 1-week forecast derived exclusively from the 'xx' pair, expressed in MAD.
- **1MAD\_PCT**:This is the 1-month trailing realized volatility (daily), expressed in mean absolute deviation (MAD). As such, it is also the conversion factor for turning a MAD-denominated number into a percent. E.g., if you'd like to have a "PV\_MEDIAN\_PCT" column (which is not in this data), simply multiply "PV\_MEDIAN" by "1MAD\_PCT".
- **1MAD\_SPOT**:The conversion factor for turning a MAD-denominated number into a spot price. E.g., if you'd like to have a "PV\_MEDIAN\_SPOT" column (which is not in this data), simply multiply "PV\_MEDIAN" by "1MAD\_SPOT".
- **IV**:A rough 30-day implied volatility, expressed an an annualized standard deviation (standard).
- **IV\_PCT**:A rough 30-day implied volatility, expressed as a 1-week average expected move (%).
- **IV\_USD**:A rough 30-day implied volatility, expressed as a 1-week average expected move ($).
- **OPEN, HIGH, LOW, CLOSE**: Nothing fancy.
- **VOLUME**:Total (lit and dark, all tapes) daily volume.


This data is meant to be extensive and extensible. With the two conversion factors ("1MAD\_x") and implied volatility (IV), every mean return, median return, and volatility can be placed in the context of historical and implied vol. With the availability of mean, median, and vol returns, split by predictor pair, the efficacy of innumerable mix-and-match strategies can be evaluated. Does AAPL perform better with a strategy that focuses on the _price, volatility_ relationship, or on the _gamma, dark_ relationship? _Et cetera._

At this point, we ought to mention the API, because if you plan on doing any such extensive testing, you'll want batch processing and programmatic access. [More on that here](https://squeezemetrics.com/monitor/static/api/).

Now, finally, let's talk about the tiny bit of data that gets squeezed out the other end of all this insanity.

## IV. The Ideas Page

This humble page is for context and idea generation. It presents and sorts securities by a normalized MEDIAN 1-week forecasted return, by default, using a k-nearest-neighbor algorithm with time- and distance-weights.

If you have the need to add some complexity to this process, or to apply your own sorting and weighting schemes, there's an [API call](https://squeezemetrics.com/monitor/static/api) (/latest) that returns all of the most recent day's data, in JSON or CSV format, for your nerdy pleasure.

- [Documentation](https://squeezemetrics.com/monitor/docs)
