[sqzme](https://squeezemetrics.com/monitor) [~~~~~~~~](https://squeezemetrics.com/monitor/docs#)

- [Benefits](https://squeezemetrics.com/monitor/benefits)
- [Plans](https://squeezemetrics.com/monitor/plans)
- [Documentation](https://squeezemetrics.com/monitor/docs)
- [API](https://squeezemetrics.com/monitor/static/api/)

- [Signup](https://squeezemetrics.com/monitor/signup)
- [Login](https://squeezemetrics.com/monitor/login)

## Documentation

* * *

1. [The Four Axes](https://squeezemetrics.com/monitor/docs#axes)

1. [Price](https://squeezemetrics.com/monitor/docs#axes1)
2. [Volatility](https://squeezemetrics.com/monitor/docs#axes2)
3. [Gamma](https://squeezemetrics.com/monitor/docs#axes3)
4. [Dark](https://squeezemetrics.com/monitor/docs#axes4)

3. [The Charts](https://squeezemetrics.com/monitor/docs#charts)
4. [The Sheets (and the API)](https://squeezemetrics.com/monitor/docs#sheets)
5. [The Research Page](https://squeezemetrics.com/monitor/docs#research)

## I. The Four Axes

There are four predictors that form the basis of all of our data— _(a) price, (b) volatility, (c) gamma,_ and _(d) dark_—let's address each of them in turn.

#### a. Price (P)

Movement in price is usually denominated in dollars, points, or percent. Movement in price is rarely, if ever, denominated in what really matters—the price change in relation to historical volatility, i.e., the price change relative to how much it usually moves.

If two stocks, A and B, both move 5.00% tomorrow, that's not very useful information on its own. If we know, however, that Stock A has been moving 1.00% per day, on average; and Stock B has been moving 10.00% per day, on average; then we're able to place those percentage moves in context:

- Stock A moved 5.00x its average expected move.
- Stock B moved 0.50x its average expected move.

Another way to express the multiple of the expected move is in "mean absolute deviation" (MAD). A 1.00x move is a 1 MAD move. A 5.00x move is a 5 MAD move. A 5 MAD move is big.

But what if we want to get a contextual sense of how a stock has moved over not the past day, but the past _month?_ Well, usually this, too, is denominated in percent moves. So, e.g., if a stock (priced at $100) that usually moves 1.00% per day starts off a month with a _nasty_ one-day, 20% decline ($100 → $80), then gradually claws back 1.00% per day for the next 20 market days (→ $97.61), the "monthly return" is -2.38%. But that's... not very useful information, because it doesn't at all describe the _magnitude_ of the tail events we experienced during the month.

Rather than refer to this monthly return, we want to have some metric that considers each of the daily moves that comprise the past month. But, like we said before, daily _percent_ moves just aren't able to provide enough context on their own. In order to get the context we need, we have to compare every day in the past month to the trailing historical daily moves at the time. So if the 20% decline followed a period of 1.00% average returns, then that -20% move was a 20-MAD event _(!)_. And since the 20% decline would substantially raise the trailing historical average daily move of the past month (1.00% → 1.90%), then the next few days of 1.00% returns would be a mere 0.53 MAD.

In this context, the mean return over the period, despite 20 straight days of +0.53 MAD returns, would be a lousy -0.45 MAD. This result emphasizes the outsized impact of that 20-MAD event at the beginning of the month.

As juxtaposition, imagine that the 20% decline followed a period of 5.00% average daily moves instead. The decline would be a 4 MAD event instead (not nearly as big a deal). In this case, the subsequent mean return over the period would be not -0.45 MAD, but +0.26 MAD. Comparatively bullish!

This is why, if we only looked at percentages, we'd get the totally wrong idea. Changes in price must be viewed in the context of volatility, or else they lose meaning. And while we may know this intuitively, and we may have a sense for the way volatility impacts true returns, we have to declare all of this explicitly when we're attempting to use numerical methods. If we don't, it's garbage-in, garbage-out.

Summary: 'Price' is the 1-month sum of daily MAD returns, with a rolling window of 1-month volatility (defined as average daily moves) as its denominator. It will generally move between -1 and +1. This number tells us, better than anything else, how price has really been moving, and is comparable across any and all assets.

#### b. Volatility

Why is volatility important? Because if a stock is becoming more or less volatile, that gives us crucial context about how market participants are engaging with the stock. If volatility is decreasing while price rises, that's a _very_ different situation from volatility _increasing_ as price rises, and you would never want to mistake one for the other.

So, in the same way that 'price' records the change in price as a function of volatility, 'volatility' records the change in volatility as a function of volatility. Because in the same way that change in price, denominated in percent, fails to capture the reality of the change; change in volatility denominated in vol points (like VIX), also misses the mark.

Return to the example above: The average daily move of a stock, before that 20% decline, was always 1.00%. I.e., the mean absolute deviation (MAD) was 1.00%. But as we said, the subsequent 20-MAD event raised the average daily move over the past month to 1.90%. Since, a month prior, MAD was 1.00%, and suddenly, MAD became 1.90%, all we really need to do to describe the change in volatility is to difference these (1.90 - 1.00). I.e., 'volatility' is +0.90 MAD.

Like 'price', 'volatility' conveniently tends to move within the domain \[-1, +1\]. So a 'volatility' of +0.90 MAD means there was a huge relative increase in volatility, which is the perspective we need to go alongside 'price'.

Now we have two predictors, 'price' and 'volatility', that both use the same units (MAD), are measuring nothing more than simple averages on a 1-month period, move in the domain \[-1, 1\], and can be used on any asset.

This is quite powerful already, but wait—there's more!

#### c. Gamma

Ever since our [2016 paper](https://squeezemetrics.com/monitor/download/pdf/white_paper.pdf) on gamma exposure (GEX), people have been eager to replicate and extend the concept. Here, though, we're peeling away any and every layer of complexity to the computation and revealing a simple ratio: The gamma of call open interest to the gamma of combined call and put open interest. A call-to-put ratio, but measured in gamma.

So, e.g., if the gamma of all call open interest in a stock, across all expirations and strikes, adds up to 500,000 shares per 1.00% move; and the gamma of all of the same puts adds up to 500,000 shares per 1.00% move, then 'gamma' is 0.50. I.e., "there is the same amount of call gamma as put gamma." Or, "call gamma comprises 50% of total gamma." Likewise, if call gamma is 90,000 and put gamma is 10,000, 'gamma' is 0.90.

Any time 'gamma' is under 0.50, puts are relatively more important; any time 'gamma' is over 0.50, calls are relatively more important. By design, the critically important concept of "zero GEX" is completely preserved here (as gamma = 0.50), while removing all other confounding factors.

Why the simplification? Because we have the luxury, now, of viewing gamma in the context of both (a) 'price' and (b) 'volatility'. This gives us a much clearer view of changes in option exposures, and isolates the impact that options have in individual stocks in a way that could never be done with one-dimensional data.

And the same can be said of our fourth and final predictor:

#### d. Dark

Dark pool short volume, as reported by FINRA in its [Reg SHO daily files](https://www.finra.org/finra-data/browse-catalog/short-sale-volume-data/daily-short-sale-volume-files), is the basis for the [Dark Index (DIX)](https://squeezemetrics.com/monitor/dix), and the subject of the paper, ["Short is Long."](https://squeezemetrics.com/monitor/download/pdf/short_is_long.pdf) Like gamma, dark is a simple ratio: _Short_ off-exchange volume is the numerator, and _all_ off-exchange volume is the denominator.

The result can be expressed as a decimal (0.0 to 1.0) or as a percentage (like DIX), but in either case, it tells us the relative amount of trade volume marked "short" in off-exchange (dark) trading.

As with gamma, dark becomes more powerful when viewed in the context of the other predictors. Because it rises and falls without regard to price, volatility, or gamma, it provides an uncorrelated signal that frequently tracks sentiment, whether or not that sentiment has been reflected in price.

This fourth axis is the final dimension of the data. The need for _each_ of these four dimensions as inputs is what drives the presentation and visualization of the data (it's not easy to think in 4-D), as well as the algorithmic methods with which we derive probability distributions.

## II. The Charts

![](https://squeezemetrics.com/monitor/static/images/docs_thechart.png)

These are fast, interactive charts, designed to cram over ten years and 15 time-series onto just one x-axis and two y-axes. Functionality is as minimal as possible.

- click-and-drag to zoom in on an area
- double-click to zoom back out
- mouse over to see more detailed data
- click on the key to the right to hide and show

And that's all there is to it. Fast charts, no frills. The data on the charts is described below, with their spreadsheet column names in parentheses.

1. _Open, High, Low, Close (OPEN, HIGH, LOW, CLOSE):_


    Pretty self-explanatory, right? We like candlesticks. You should too.
2. _Price (P):_


    The _price_ predictor, as described above.
3. _Volatility (V):_


    The _volatility_ predictor, as described above.
4. _Gamma (G):_


    The _gamma_ predictor, as described above.
5. _Dark (D):_


The _dark_ predictor, as described above. _(The normalized predictors are in spreadsheet columns P, V, G, and D.)_
6. _Mean (MEAN\_SPOT):_


    The average price of Robot Jim's 1-week forecast.
7. _xx Mean (xx\_MEAN\_SPOT):_


    The average price of Robot Jim's 1-week forecast for the denoted pair. E.g., "PV Mean" is the forecast of only the _price, volatility_ pair.

The charts are meant to provide just enough historical context to understand what the patterns visible in the weather maps really mean, and how events actually unfold. But some of you will want to test your own signals, create your own moving averages, look for your own patterns, and run your own backtests. And that brings us to...

## III. The Sheets (and the API)

Every security in our database can be exported as a spreadsheet document (CSV). Here are the columns in those spreadsheets, and some numerical methods for extending the data:

- **DATE**:



YYYY-MM-DD

- **TIMESTAMP**:



Unix time, seconds from epoch.

- **MEAN**:



Robot Jim's mean 1-week forecast, expressed in mean absolute deviation (MAD).

- **MEAN\_SPOT**:



Robot Jim's mean 1-week forecast, expressed in stock price.

- **MEDIAN**:



Robot Jim's median 1-week forecast, expressed in MAD.

- **VOL**:



Robot Jim's mean 1-week volatility forecast, expressed in MAD.

- **VOL\_MEDIAN**:



Robot Jim's median 1-week volatility forecast, expressed in MAD.

- **MEAN\_PCT**:



Robot Jim's mean 1-week forecast, expressed in percent.

- **MEDIAN\_PCT**:



Robot Jim's median 1-week forecast, expressed in percent.

- **VOL\_PCT**:



Robot Jim's mean 1-week volatility forecast, expressed in percent.

- **VOL\_MEDIAN\_PCT**:



Robot Jim's median 1-week volatility forecast, expressed in percent.

- **P0**:



The 'price' predictor, not normalized.

- **V0**:



The 'volatility' predictor, not normalized.

- **G0**:



The 'gamma' predictor, not normalized.

- **D0**:



The 'dark' predictor, not normalized.

- **P**:



The 'price' predictor, normalized to \[-1, 1\].

- **V**:



The 'volatility' predictor, normalized to \[-1, 1\].

- **G**:



The 'gamma' predictor, normalized to \[-1, 1\].

- **D**:



The 'dark' predictor, normalized to \[-1, 1\].

- **xx\_MEAN**:



Robot Jim's mean 1-week forecast derived exclusively from the 'xx' pair, expressed in mean absolute deviation (MAD).

- **xx\_MEAN\_SPOT**:



Robot Jim's mean 1-week forecast derived exclusively from the 'xx' pair, expressed in stock price.

- **xx\_MEDIAN**:



Robot Jim's median 1-week forecast derived exclusively from the 'xx' pair, expressed in MAD.

- **xx\_VOL**:



Robot Jim's mean 1-week volatility forecast derived exclusively from the 'xx' pair, expressed in MAD.

- **xx\_VOL\_MEDIAN**:



Robot Jim's median 1-week forecast derived exclusively from the 'xx' pair, expressed in MAD.

- **1MAD\_PCT**:



This is the 1-month trailing realized volatility (daily), expressed in mean absolute deviation (MAD). As such, it is also the conversion factor for turning a MAD-denominated number into a percent. E.g., if you'd like to have a "PV\_MEDIAN\_PCT" column (which is not in this data), simply multiply "PV\_MEDIAN" by "1MAD\_PCT".

- **1MAD\_SPOT**:



The conversion factor for turning a MAD-denominated number into a spot price. E.g., if you'd like to have a "PV\_MEDIAN\_SPOT" column (which is not in this data), simply multiply "PV\_MEDIAN" by "1MAD\_SPOT".

- **IV**:



A rough 30-day implied volatility, expressed an an annualized standard deviation (standard).

- **IV\_PCT**:



A rough 30-day implied volatility, expressed as a 1-week average expected move (%).

- **IV\_USD**:



A rough 30-day implied volatility, expressed as a 1-week average expected move ($).

- **OPEN, HIGH, LOW, CLOSE**:



Nothing fancy.

- **VOLUME**:



Total (lit and dark, all tapes) daily volume.


This data is meant to be extensive and extensible. With the two conversion factors ("1MAD\_x") and implied volatility (IV), every mean return, median return, and volatility can be placed in the context of historical and implied vol. With the availability of mean, median, and vol returns, split by predictor pair, the efficacy of innumerable mix-and-match strategies can be evaluated. Does AAPL perform better with a strategy that focuses on the _price, volatility_ relationship, or on the _gamma, dark_ relationship? _Et cetera._

At this point, we ought to mention the API, because if you plan on doing any such extensive testing, you'll want batch processing and programmatic access. [More on that here](https://squeezemetrics.com/monitor/static/api/).

Now, finally, let's talk about the tiny bit of data that gets squeezed out the other end of all this insanity.

## IV. The Research Page

This humble page is for context and idea generation. It searches through the top 1000 securities by dollar volume and finds the best bullish and bearish opportunities (50 from each category), then sorts them according to Robot Jim's return forecast (MEAN).

If you have the need to add some complexity to this process, or to apply your own sorting and weighting schemes, there's an [API call](https://squeezemetrics.com/monitor/static/api) (/latest) that returns all of the most recent day's data, in JSON or CSV format, for your nerdy pleasure.

- [Documentation](https://squeezemetrics.com/monitor/docs)
- [API](https://squeezemetrics.com/monitor/static/api/)
- [Chart](https://squeezemetrics.com/monitor/p/chart)
- [Research](https://squeezemetrics.com/monitor/p/research)
- [Benefits](https://squeezemetrics.com/monitor/benefits)
- [Plans](https://squeezemetrics.com/monitor/plans)
- [Signup](https://squeezemetrics.com/monitor/signup)
- [Contact](https://squeezemetrics.com/monitor/contact)
- [Terms](https://squeezemetrics.com/monitor/terms)
- [Login](https://squeezemetrics.com/monitor/login)

🍋


A product of SqueezeMetrics.

Copyright © 2015–2025

Prior Analytics LLC