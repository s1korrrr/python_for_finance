# ALE HECA - super fund app!!

## PYTHON LANGUAGE !

thanks to sentdex !!


In case you do not know:

Open - When the stock market opens in the morning for trading, what was the price of one share?
High - over the course of the trading day, what was the highest value for that day?
Low - over the course of the trading day, what was the lowest value for that day?
Close - When the trading day was over, what was the final price?
Volume - For that day, how many shares were traded?
Adj Close - This one is slightly more complicated, but, over time, companies may decide to do something
called a stock split. For example, Apple did one once their stock price exceeded $1000. Since in most cases,
people cannot buy fractions of shares, a stock price of $1,000 is fairly limiting to investors.
Companies can do a stock split where they say every share is now 2 shares, and the price is half.
Anyone who had 1 share of Apple for $1,000, after a split where Apple doubled the shares,
they would have 2 shares of Apple (AAPL), each worth $500. Adj Close is helpful,
since it accounts for future stock splits, and gives the relative price to splits. For this reason,
the adjusted prices are the prices you're most likely to be dealing with.


"""
print(df.head())

The .head() is something you can do with Pandas DataFrames, and it will output the first n rows,
where n is the optional parameter you pass. If you don't pass a parameter, 5 is the default value.
We mosly will use .head() to just get a quick glimpse of our data to make sure we're on the right track.
Looks great to me!

print(df.tail())
"""

# print(df.head())
# print(df[['High','Low']])


"""
The idea of a simple moving average is to take a window of time, and calculate the average price in that window.
Then we shift that window over one period, and do it again. In our case, we'll do a 100 day rolling moving average.
So this will take the current price, and the prices from the past 99 days, add them up, divide by 100, and there's
your current 100-day moving average. Then we move the window over 1 day, and do the same thing again. Doing this in
Pandas is as simple as:

df['100ma'] = df['Adj Close'].rolling(window=100,min_periods=0).mean()
print(df.head())

Doing df['100ma'] allows us to either re-define what comprises an existing column if we had one called '100ma,'
or create a new one, which is what we're doing here. We're saying that the df['100ma'] column is equal to being
the df['Adj Close'] column with a rolling method applied to it, with a window of 100, and this window is going
to be a mean() (average) operation.
"""

First, we need proper OHLC data. Our current data does have OHLC values, and, unless I am mistaken, Tesla has never had a split, but you wont always be this lucky. Thus, we're going to create our own OHLC data, which will also allow us to show another data transformation that comes from Pandas:

df_ohlc = df['Adj Close'].resample('10D').ohlc()
What we've done here is created a new dataframe, based on the df['Adj Close']column, resamped with a 10 day window, and the resampling is an ohlc (open high low close). We could also do things like .mean() or .sum() for 10 day averages, or 10 day sums. Keep in mind, this 10 day average would be a 10 day average, not a rolling average. Since our data is daily data, resampling it to 10day data effectively shrinks the size of our data significantly. This is how you can normalize multiple datasets. Sometimes, you might have data that tracks once a month on the 1st of the month, other data that logs at the end of each month, and finally some data that logs weekly. You can resample this dataframe to the end of the month, every month, and effectively normalize it all! That's a more advanced Pandas feature that you can learn more about from the Pandas series if you like.

We'd like to graph both the candlestick data, as well as the volume data. We don't HAVE to resample the volume data, but we should, since it would be too granular compared to our 10D pricing data.

df_volume = df['Volume'].resample('10D').sum()
We're using sum here, since we really want to know the total volume traded over those 10 days, but you could also use mean instead. Now if we do:

print(df_ohlc.head())



That's expected, but, we want to now move this information to matplotlib, as well as convert the dates to the mdates version. Since we're just going to graph the columns in Matplotlib, we actually don't want the date to be an index anymore, so we can do:

df_ohlc = df_ohlc.reset_index()
Now dates is just a regular column. Next, we want to convert it:

df_ohlc['Date'] = df_ohlc['Date'].map(mdates.date2num)
Now we're going to setup the figure:

fig = plt.figure()
ax1 = plt.subplot2grid((6,1), (0,0), rowspan=5, colspan=1)
ax2 = plt.subplot2grid((6,1), (5,0), rowspan=1, colspan=1,sharex=ax1)
ax1.xaxis_date()
Everything here you've already seen, except ax1.xaxis_date(). What this does for us is converts the axis from the raw mdate numbers to dates.

Now we can graph the candlestick graph:

candlestick_ohlc(ax1, df_ohlc.values, width=2, colorup='g')
Then do volume:

ax2.fill_between(df_volume.index.map(mdates.date2num),df_volume.values,0)
The fill_between function will graph x, y, then what to fill to/between. In our case, we're choosing 0.

plt.show()


"""
The tickers/symbols in Wikipedia are organized on a table. To handle for this, we're going to use the HTML parsing library, Beautiful Soup. If you would like to learn more about Beautiful Soup, I have a quick 4-part tutorial on web scraping with Beautiful Soup.
bs4 is for Beautiful Soup, pickle is so we can easily just save this list of companies, rather than hitting Wikipedia every time we run (though remember, in time, you will want to update this list!), and we'll be using requests to grab the source code from Wikipedia's page.