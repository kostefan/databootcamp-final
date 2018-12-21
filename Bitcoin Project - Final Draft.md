
# Describing Bitcoin

Bitcoin was the first ever decentralized cryptocurrency. It utilizes Blockchain, a distributed ledger, to ensure the accuracy and transparency of exchanges. Although it had been around for nearly a decade, it was around 2017 that Bitcoin really caught world-wide attention and we saw the prices peak. This project will describe and visualize the story behind the rise of Bitcoin and its Network, and also examine its current state.

## Set Up Files & Packages


```python
# import packages

import pandas as pd
import datetime
import numpy as np
import matplotlib.pyplot as plt
import matplotlib
import seaborn as sns
import plotly.plotly as py
import plotly.graph_objs as go
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
import csv
import xlrd
import os

%matplotlib inline
init_notebook_mode(connected=True)  #allow visualization in notebook
```


<script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script><script type="text/javascript">if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script><script>requirejs.config({paths: { 'plotly': ['https://cdn.plot.ly/plotly-latest.min']},});if(!window._Plotly) {require(['plotly'],function(plotly) {window._Plotly=plotly;});}</script>



```python
folder = "C:/Users/Stefan/Documents/Data_Bootcamp/Final_Draft/"

# Bitcoin Sentiment Imports
BTrend = pd.read_csv(folder+'BitcoinInterestGTrends.csv')
BFactiva = pd.read_csv(folder+'BitcoinFactivaArticleCount.csv')

# Bitcoin Price Imports
BTC_Data_Coinbase = pd.read_csv(folder+'Coinbase_BTCUSD.csv')
BTC_Data_Kraken = pd.read_csv(folder+'Kraken_BTCUSD.csv')
BTC_Bitstamp_Data = pd.read_csv(folder+'Bitstamp_BTCUSD.csv')
BTC_Bitfinex_Data = pd.read_csv(folder+'Bitfinex_BTCUSD.csv')

# Bitcoin Blockchain Imports
hash_rate = pd.read_csv(folder+'BitcoinHashRate.csv')
difficulty = pd.read_csv(folder+'BitcoinDifficulty.csv')
fees_per_day = pd.read_csv(folder+'BitcoinFeesPerDay.csv')
transaction_per_day = pd.read_csv(folder+'BitcoinTransactionPerDay.csv')
block_reward_per_day = pd.read_csv(folder+'BitcoinBlockRewardPerDay.csv')
bitcoin_chain_size = pd.read_csv(folder+'BitcoinChainSize.csv')
bitcoin_node = pd.read_csv(folder+'BitcoinNodes-20181219.csv')
countryCodes = pd.read_csv(folder+'country-codes.csv')
```


```python
# Definitions to convert date time series
def dateifier(date):
    v = list(map(int, date.split('/')))
    return datetime.date(v[2], v[0], v[1])


# Function to reformat time
def reformatTime(dataSet):
    dataSet['DateTime'] = dataSet['DateTime'].apply(lambda x: x.split()[0])
    dataSet['DateTime'] = dataSet['DateTime'].apply(
        lambda x: x.split('-')).apply(
            lambda x: datetime.date(int(x[0]), int(x[1]), int(x[2])))
```

## Bitcoin Sentiment

At the end of 2017, there was an incredible amount of craze over Bitcoin and seemingly everyone was talking about how cryptocurrencies will takeover fiat currencies because the security and transparency of Blockchain. It was also around this time that Bitcoin hit its peak prices. News and media was filled with analysts predicting Bitcoin prices to hit $30,000, while others were  calling it a bubble and claiming it will crash soon. To understand exactly how much people were talking about Bitcoin, we took data from Google Trends to understand the buzz around Bitcoin, and examined the number of articles published on Bitcoin from 2016 - today through Factiva.


```python
plt.style.use('seaborn')

# Setting the Index
BTrend['Month'] = pd.to_datetime(BTrend['Month'])
BTrend.set_index('Month', inplace=True)

# Plot
fig, ax = plt.subplots(figsize=(7, 4))

x = BTrend.index
y = BTrend['Interest']

fig.autofmt_xdate()
ax.set_xlim([datetime.date(2015, 12, 1), datetime.date(2018, 12, 31)])
ax.set_title("Google Trends for Bitcoin since 2016")
ax.plot(x, y)
```




    [<matplotlib.lines.Line2D at 0x22db2056f28>]




![png](output_8_1.png)



```python
# Setting Index and Cleaning
BFactiva['Date'] = pd.to_datetime(BFactiva['Date'])
BFactiva.set_index('Date', inplace=True)
BFactiva = BFactiva.dropna(thresh=1)

# Plot
fig, ax = plt.subplots(figsize=(7, 4))

x = BFactiva.index
y = BFactiva['Article Count']

fig.autofmt_xdate()
ax.set_xlim([datetime.date(2016, 1, 1), datetime.date(2018, 12, 31)])
ax.set_title("Total Number of Articles on Bitcoin")
ax.plot(x, y)
```




    [<matplotlib.lines.Line2D at 0x22db2123eb8>]




![png](output_9_1.png)


From the Google Trends interest in the search term "Bitcoin" and the total number of articles published on Bitcoin from 2016, we definitely see there was a rise in interest starting around May of 2017. At this point we'd like to dive further into understanding the general sentiment on Bitcoin from this time, and have pulled the number of articles that displayed sentiment (both positive and negative) from Factiva.


```python
# Making Data Frame
df = pd.DataFrame({
    'x':
    BFactiva.index, 'Total Number of Articles with Sentiment': (BFactiva['Positive'] + BFactiva['Negative']),
    'Number of Articles with Positive Sentiment': BFactiva['Positive'],
    'Number of Articles with Negative Sentiment': BFactiva['Negative']
})

# Create a color palette
palette = plt.get_cmap('Set2')

# Configure Figuresize
fig = plt.figure(figsize=(10, 10))

# Multiple line plot
num = 0
for column in df.drop('x', axis=1):
    num += 1

    plt.subplot(3, 1, num)

    # Plot Total
    plt.plot(df['x'], df['Total Number of Articles with Sentiment'],
             color='midnightblue', linewidth=0.6, alpha=0.3)

    # Plot the lineplot
    plt.plot( df['x'], df[column], marker='', color=palette(num), linewidth=1.4, alpha=0.9, label=column)

    # Add title
    plt.title(column, loc='center', fontsize=12, fontweight=0, color=palette(num))
```


![png](output_11_0.png)


Indeed, we see that around May of 2017 we see a rising trend of positive articles. We also see that there was a similar amount of positive and negative sentiment around this time, but soon the overall sentiment has become overwhelmingly positive, especially leading up to today. In fact, considering that the price of Bitcoin is driven by supply and demand, this strong media presence and positive sentiment likely contributed to the rising price of Bitcoin. Next, we would like to analyze the prices of Bitcoin.

## Bitcoin Price & Volatility 

In order to more accurately analyze trends in the price data, we decided to look at Bitcoin's average price across the four largest exchanges the currency is traded on. These exchanges are Coinbase, Kraken, Bitstamp, and Bitfinex. We then looked at graphs of Bitcoin's price, as well as the daily percentage changes in that price.


```python
# Coinbase
BTC_Coinbase = pd.DataFrame(BTC_Data_Coinbase)
BTC_Coinbase['Date'] = BTC_Coinbase['Date'].map(dateifier)
BTC_Coinbase.set_index(['Date'], inplace=True)

BTC_Coinbase['%Change'] = (
    BTC_Coinbase['Close'] - BTC_Coinbase['Open']) / BTC_Coinbase['Open']

# Kraken
BTC_Kraken = pd.DataFrame(BTC_Data_Kraken)
BTC_Kraken['Date'] = BTC_Kraken['Date'].map(dateifier)
BTC_Kraken.set_index(['Date'], inplace=True)

BTC_Kraken_Diff = BTC_Kraken['Close'] - BTC_Kraken['Open']
BTC_Kraken_Change = BTC_Kraken_Diff / BTC_Kraken['Open']
BTC_Kraken['%Change'] = BTC_Kraken_Change

# Bitstamp

BTC_Bitstamp = pd.DataFrame(BTC_Bitstamp_Data)
BTC_Bitstamp['Date'] = BTC_Bitstamp['Date'].map(dateifier)
BTC_Bitstamp.set_index(['Date'], inplace=True)

BTC_Bitstamp_Diff = BTC_Bitstamp['Close'] - BTC_Bitstamp['Open']
BTC_Bitstamp_Change = BTC_Bitstamp_Diff / BTC_Bitstamp['Open']
BTC_Bitstamp['%Change'] = BTC_Bitstamp_Change

# Bitfinex
BTC_Bitfinex = pd.DataFrame(BTC_Bitfinex_Data)
BTC_Bitfinex['Date'] = BTC_Bitfinex['Date'].map(dateifier)
BTC_Bitfinex.set_index(['Date'], inplace=True)

BTC_Bitfinex_Diff = BTC_Bitfinex['Close'] - BTC_Bitfinex['Open']
BTC_Bitfinex_Change = BTC_Bitfinex_Diff / BTC_Bitfinex['Open']
BTC_Bitfinex['%Change'] = BTC_Bitfinex_Change
```

### Average Bitcoin Price


```python
BTC_Price = pd.DataFrame()

BTC_Price['Kraken'] = BTC_Kraken['Close']
BTC_Price['Coinbase'] = BTC_Coinbase['Close']
BTC_Price['Bitstamp'] = BTC_Bitstamp['Close']
BTC_Price['Bitfinex'] = BTC_Bitfinex['Close']

BTC_Average_Price = BTC_Price.mean(axis=1)

BTC_Average_Price.plot(figsize=(15, 5)).set_ylabel('USD')

plt.show()
```


![png](output_17_0.png)



```python
BTC_Price['Average'] = BTC_Price.mean(axis=1)
```


```python
# Bitcoin Price Peak
BTC_Average_Price[BTC_Average_Price == BTC_Average_Price.max()]
```




    Date
    2017-12-16    19351.17
    dtype: float64



Our analysis seems to show a positive relationship between Bitcoin's price and sentiment. This can be seen clearly by the relatively stable prices between 2014 and 2017. Both sentiment and Bitcoin's price both began to rise precipitously midway through 2017, culminating in Bitcoin's peak price on December 16th, 2017.

### Average BTC Price Change


```python
BTC_Changes = pd.DataFrame()

BTC_Changes['Kraken'] = BTC_Kraken['%Change']
BTC_Changes['Coinbase'] = BTC_Coinbase['%Change']
BTC_Changes['Bitstamp'] = BTC_Bitstamp['%Change']
BTC_Changes['Bitfinex'] = BTC_Bitstamp['%Change']

BTC_Average_Change = BTC_Changes.mean(axis=1)

BTC_Average_Change.plot(figsize=(15, 5)).set_ylabel('%')

plt.axhline(0, color='darkorange')

plt.show()
```


![png](output_22_0.png)



```python
previous_two_years = pd.DataFrame(BTC_Average_Change)
previous_two_years['Date'] = pd.to_datetime(previous_two_years.index)
previous_two_years['Average'] = BTC_Average_Change
previous_two_years.set_index('Date', inplace=True)

previous_two_years.drop(0, axis=1)

fig, ax = plt.subplots(figsize=(15, 5))

x = previous_two_years.index
y = previous_two_years['Average']

fig.autofmt_xdate()
ax.set_xlim([datetime.date(2016, 1, 26), datetime.date(2018, 12, 31)])
ax.set_ylim(-0.2, 0.3)
ax.plot(x, y)

plt.axhline(0, color='darkorange')

plt.show()
```


![png](output_23_0.png)


Our analysis of price volatility yielded similar results. Bitcoin experienced relatively low volatility between 2015 and 2017, at which point there is a large demarcation from the previous trend. This spike in volatility can be seen clearly in the graph immediately above and coincides almost identically with the sharp increase in impressions that occured during this period.

The volality of Bitcoin's price at this point seems to bring many skepticisms true about whether Bitcoin can be a sound alternative for fiat currency as it was initially intended to. The fall and abruptive rise in price can have various impacts. But due to the limited amount of work we can do, our group has decided to focus specifically on the Bitcoin network alone as it is the underlying bed for Bitcoin to operate.

## The Bitcoin Network

Within the Bitcoin network, there are three main participants. These include the miners, the full nodes and the users. Out of the three, the miners and the full nodes are the most important participants as they are the ones who maintain and keep the network alive. However, they are often conflated with each other since they sometimes share the same responsbility of maintaining the bitcoin's data records of transactions on the blockchain, which is mostly the work of the nodes. Unfortunately at the moment, due to the limited scope of the course and the relatively small data sets there exist on the public internet, we were unable to gather data on a historical timeline for the Bitcoin's miners and nodes. But with basic tools and instructions, we have been able gather data on the current numbers of nodes and their distributions over the world. 

### Bitcoin Nodes


```python
bitcoin_node.rename(columns={'Country code.1': "Country"}, inplace=True)
bitcoin_country_node = pd.DataFrame(bitcoin_node[['Country code', 'Country']])

# create a new default column
bitcoin_country_node['Number of Nodes'] = 0

# counting the number of nodes in each country
for i in bitcoin_country_node['Country code'].unique():
    country_count = bitcoin_country_node['Country code'][
        bitcoin_country_node['Country code'] == i].count()
    bitcoin_country_node['Number of Nodes'][
        bitcoin_country_node['Country code'] == i] = int(country_count)

bitcoin_country_node.drop_duplicates(inplace=True)

bitcoin_country_node.drop_duplicates(inplace=True)
bitcoin_country_node.dropna(inplace=True)

letterCode2 = [country for country in countryCodes['Alpha-2 code']]
letterCode3 = [country for country in countryCodes['Alpha-3 code']]
countrydf3 = bitcoin_country_node.replace(letterCode2, letterCode3)
```

    C:\Users\Stefan\Anaconda3\lib\site-packages\ipykernel_launcher.py:12: SettingWithCopyWarning:
    
    
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
    
    


```python
data = [dict(
        type='choropleth',
        locations=countrydf3['Country code'],
        z=countrydf3['Number of Nodes'],
        text=countrydf3['Country'],)]

layout = dict(
    title='Bitcoin Nodes Spread',
    geo=dict(
        showframe=True, showcoastlines=True, projection=dict(type='mercator')))

fig = go.Figure(data=data, layout=layout)
iplot(fig, validate=False, filename='Bitcoin Nodes Spread')
```


<div id="f530183a-2a67-4ef6-8c8e-2928ca254c14" style="height: 525px; width: 100%;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("f530183a-2a67-4ef6-8c8e-2928ca254c14", [{"locations": ["FRA", "ESP", "NLD", "ISR", "USA", "FIN", "CHN", "IND", "LTU", "RUS", "SGP", "CHE", "HKG", "JPN", "ITA", "CZE", "HUN", "CAN", "DEU", "NOR", "KOR", "TUR", "AUS", "GBR", "SWE", "UKR", "BRA", "BLZ", "IRL", "BGR", "NZL", "POL", "MEX", "AGO", "BLR", "ROU", "ZAF", "TWN", "ISL", "MDA", "LUX", "AUT", "THA", "EST", "CRI", "CHL", "IRN", "ARG", "BEL", "SVK", "LAO", "MYS", "PHL", "CUW", "MAC", "HRV", "SVN", "DOM", "VEN", "VNM", "DNK", "TTO", "ALB", "KAZ", "COL", "FRO", "PRT", "GRC", "ARE", "KGZ", "LVA", "PRI", "SAU", "IDN", "GGY", "REU", "QAT", "SRB", "LIE", "IMN", "KEN", "ABW", "HND", "KHM", "AND", "BOL", "CYP", "VGB", "SYC", "JEY", "URY", "BMU", "NGA", "SMR", "MNE"], "text": ["France", "Spain", "Netherlands", "Israel", "United States of America", "Finland", "China", "India", "Lithuania", "Russia", "Singapore", "Switzerland", "Hong Kong", "Japan", "Italy", "Czech Republic", "Hungary", "Canada", "Germany", "Norway", "Korea", "Turkey", "Australia", "United Kingdom", "Sweden", "Ukraine", "Brazil", "Belize", "Ireland", "Bulgaria", "New Zealand", "Poland", "Mexico", "Angola", "Belarus", "Romania", "South Africa", "Taiwan", "Iceland", "Moldova", "Luxembourg", "Austria", "Thailand", "Estonia", "Costa Rica", "Chile", "Iran", "Argentina", "Belgium", "Slovakia", "Lao PDR", "Malaysia", "Phillipines", "Curucao", "Macao", "Croatia", "Slovenia", "Dominican Republic", "Venezuela", "Viet Nam", "Denmark", "Trinidad and Tobago", "Albania", "Kazakhstan", "Colombia", "Faroe Islands", "Portugal", "Greece", "United Arab Emirates", "Kyrgyzstan", "Latvia", "Puerto Rico", "Saudi Arabia", "Indonesia", "Guernsey", "Reunion", "Qatar", "Serbia", "Liechtenstein", "Isle of Man", "Kenya", "Aruba", "Honduras", "Cambodia", "Andorra", "Bolivia", "Cyprus", "British Virgin Islands", "Seychelles", "Jersey", "Uruguay", "Bermuda", "Nigeria", "San Marino", "Montenegro"], "z": [653, 65, 468, 19, 2085, 110, 439, 72, 66, 269, 217, 130, 147, 227, 58, 78, 22, 383, 1649, 48, 183, 11, 156, 324, 97, 93, 54, 2, 101, 51, 19, 57, 14, 1, 31, 63, 23, 14, 6, 5, 11, 42, 20, 7, 5, 2, 1, 17, 27, 22, 1, 17, 2, 3, 1, 6, 21, 1, 3, 8, 18, 1, 1, 8, 3, 1, 8, 19, 6, 4, 9, 2, 1, 2, 1, 1, 2, 4, 1, 1, 2, 1, 1, 2, 1, 1, 2, 1, 3, 2, 3, 1, 1, 2, 1], "type": "choropleth", "uid": "77f92647-c88b-4622-8a7a-5f8ebcad66ed"}], {"geo": {"projection": {"type": "mercator"}, "showcoastlines": true, "showframe": true}, "title": "Bitcoin Nodes Spread"}, {"showLink": true, "linkText": "Export to plot.ly", "plotlyServerURL": "https://plot.ly"})});</script><script type="text/javascript">window.addEventListener("resize", function(){window._Plotly.Plots.resize(document.getElementById("f530183a-2a67-4ef6-8c8e-2928ca254c14"));});</script>



```python
bitcoin_country_node.nunique()
```




    Country code       95
    Country            95
    Number of Nodes    49
    dtype: int64




```python
bitcoin_country_node[bitcoin_country_node["Number of Nodes"] ==
                     bitcoin_country_node['Number of Nodes'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Country code</th>
      <th>Country</th>
      <th>Number of Nodes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>US</td>
      <td>United States of America</td>
      <td>2085</td>
    </tr>
  </tbody>
</table>
</div>



The current snapshot of the Bitcoin Nodes show that Bitcoin nodes are spread over six continents of the world, of which the most concentrated continent is North America. Of the six continents, nodes are spread over 95 countries and reasonabliy United States has the most nodes. From the data that we have seen online (https://bitnodes.earn.com/dashboard/?days=730) but wasn't able to retrieve, this number of Bitcoin nodes have more than doubled over the past two years. But this number when compared to the time when the Bitcoin nodes peaked in 2017, showed that the number have been gradually decreasing over time.

### Bitcoin Miners

At the moment, there isn't an exact number for the amount of miners there are in the world. Nonetheless, there exists estimations that go below as 5,000 to as high as nearly a 1,000,000 miners. An Australian Bitcoin miner named Andrew Geyl (https://bravenewcoin.com/insights/number-of-bitcoin-miners-far-higher-than-popular-estimates), who mined Bitcoin since 2012 estimated in 2015 that there were around 150,000 miners. This number is nearly 20 times higher than the number of nodes in the Bitcoin network. This is because miners don't have to be nodes. There can be a number of miners that use only ones nodes at a time. Nonetheless, because there aren't any data on the amount of miners, we can see their general trend through the hash rate and difficultly level of the Bitcoin network over the years. 

### Hash rate and Difficulty

In the Bitcoin Network, Hash Rate or Hash Power is the unit measurement for the number of calculations or computer power that a device can devote to mine. The higher the hash rate is, the quicker the device will be able to mine a Bitcoin and vice versa. To avoid the problem of centralization, where a computer can mine Bitcoin quickly and take a large portion of the network, the Difficulty Level was created to limit block rewards per 10 minutes. So if Bitcoin are mined more quickly than intended, the difficulty level will adjustly increased to limit the number of bitcoin mine. This was also adjust accordingly if the Bitcoin are being mined more slowly than programmed to be.


```python
reformatTime(difficulty)
reformatTime(hash_rate)

# merge to data sets
bitcoin_HD = pd.merge(hash_rate, difficulty)
```


```python
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(15, 5))

ax.plot(bitcoin_HD['DateTime'], bitcoin_HD['Hash Rate'])
ax.set_xlabel("Year")
ax.set_ylabel("Hash Rate")
ax.set_title("Bitcoin Hash Rate and Difficulty Over The Years")
ax.legend(['Hash Rate'], bbox_to_anchor=(1.1, 1.05))

ax2 = ax.twinx()

ax2.plot(bitcoin_HD['DateTime'], bitcoin_HD['Difficulty'], color='red')
ax2.set_xlabel("Year")
ax2.set_ylabel("Difficulty")
ax2.legend(['Difficulty'], bbox_to_anchor=(1.09, 1.09))
ax2.grid()

plt.tight_layout()
plt.show()
```


![png](output_37_0.png)



```python
BTC_Price.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Kraken</th>
      <th>Coinbase</th>
      <th>Bitstamp</th>
      <th>Bitfinex</th>
      <th>Average</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-11-15</th>
      <td>5436.0</td>
      <td>5464.32</td>
      <td>5462.34</td>
      <td>5683.10</td>
      <td>5511.4400</td>
    </tr>
    <tr>
      <th>2018-11-14</th>
      <td>5600.9</td>
      <td>5605.46</td>
      <td>5595.91</td>
      <td>5922.40</td>
      <td>5681.1675</td>
    </tr>
    <tr>
      <th>2018-11-13</th>
      <td>6260.0</td>
      <td>6259.35</td>
      <td>6260.91</td>
      <td>6459.07</td>
      <td>6309.8325</td>
    </tr>
    <tr>
      <th>2018-11-12</th>
      <td>6327.3</td>
      <td>6327.86</td>
      <td>6318.00</td>
      <td>6446.90</td>
      <td>6355.0150</td>
    </tr>
    <tr>
      <th>2018-11-11</th>
      <td>6358.2</td>
      <td>6357.60</td>
      <td>6357.54</td>
      <td>6447.90</td>
      <td>6380.3100</td>
    </tr>
  </tbody>
</table>
</div>




```python
bitcoin_HD[bitcoin_HD['Difficulty'] == bitcoin_HD['Difficulty'].max()].head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DateTime</th>
      <th>Hash Rate</th>
      <th>Difficulty</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3556</th>
      <td>2018-10-04</td>
      <td>5.558828e+19</td>
      <td>7.450000e+12</td>
    </tr>
  </tbody>
</table>
</div>




```python
bitcoin_HD[bitcoin_HD['Hash Rate'] == bitcoin_HD['Hash Rate'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DateTime</th>
      <th>Hash Rate</th>
      <th>Difficulty</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3518</th>
      <td>2018-08-27</td>
      <td>6.186626e+19</td>
      <td>6.730000e+12</td>
    </tr>
  </tbody>
</table>
</div>



In the graph above, with accordance to the rules of the Bitcoin network, we can see that the hash rate and the difficulty go along with each other in a general trend. Reasonably, the Hash Rate will lead the direction of the Difficulty level. This speaks for itself in their different peaks as well. While the Hash Rate level peaked in August of 2018, it took the Difficulty Level two months to adjust accordingly and limit the number of Bitcoin mined. 

One point to note about the graph is the decreasing number of Hash Rate and Difficulty level in recent time instead of the beginning of 2018, when the price of Bitcoin dramtically dropped from $19,351. This is a sign that miners might be moving out of the Bitcoin network to either mine at another network because it's more profitable or because they are shutting down due to the decreasing revenue that won't allow them to cover the cost. Thus to find a reasonable explanation, we have decided to turn towards the revenue that the miners receive from their mining.

### Bitcoin Block Rewards 

Miners nowadays have to employ specialized mining chips in order to mine Bitcoin due to the rising Difficulty Level. Thus in order, to provide these miners enough incentives and sufficient funds, the creator/group of creator - Satoshi Nakamoto have decided to program "Block Rewards" or bitcoin rewards each time a miner have been able to create a block within the Bitcoin network. In other words, each time a miner / or a group of miners have been able to create a block within the network, the indivial or the group will receive a designated amount of Bitcoin as rewards for their work. The Block Rewards are given every 10 minutes and halve at each additional 210,000 blocks. It started at 50 Bitcoin and is currently at 12.5 bitcoins per block.   


```python
reformatTime(block_reward_per_day)
```


```python
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(15, 5))

ax.plot(block_reward_per_day['DateTime'],
    block_reward_per_day['BTC'],
    color=palette(1),
    label="Bitcoin")
ax.set_ylabel("BTC")
ax.set_title(
    "Block rewards in BTC and its value in USD over the Years", fontsize=16)
ax.legend(["Bitcoin"], loc=2)

ax2 = ax.twinx()

ax2.plot(
    block_reward_per_day['DateTime'],
    block_reward_per_day['USD'],
    color=palette(2),
    label="USD")
ax2.set_xlabel("Year")
ax2.legend(['USD'], loc=0)
ax2.grid()

plt.show()
```


![png](output_45_0.png)


From the graph, we can see that the amount of Bitcoin and their values being rewarded to miners are different from each other. This is reasonable according to the price of Bitcoin over the years and the programmed amount of rewards that miners would receive. Thus, despite a low earnings of 12.5 Bitcoins per block since 2016, the Bitcoins value in US Dollars have provided the miners a high revenue especially in the end of 2017. Since there aren't any cost data at the moment on bitcoin miners due to the many factors that come into it, we can observe that this point that in recent times, Bitcoin miners profits have been decreasing with the drop in price. Another source of revenue that miners receive is the fees of transactions that users of the Bitcoins network incur each time a transaction is conducted between each other.

### Bitcoin Transactions and their Fees 

Aside from the block rewards that miners receive, they are also provided with transaction fees as part of the incentives to maintain the Bitcoin network. In fact, this will be the miners main source of revenue once the Bitcoin supply has reached its limit. 


```python
reformatTime(fees_per_day)
reformatTime(transaction_per_day)

bitcoin_FT = pd.merge(fees_per_day, transaction_per_day)
```


```python
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(15, 5))

ax.plot(bitcoin_FT["DateTime"], bitcoin_FT['USD'], color=palette(1))
ax.set_xlabel("Year")
ax.set_ylabel('USD')
ax.set_title("Bitcoin Transactions Fees and Transaction Volume over the Years", fontsize=16)

ax2 = ax.twinx()

ax2.plot(bitcoin_FT["DateTime"], bitcoin_FT['Total Per Day'], color=palette(2))
ax2.set_xlabel('Year')
ax2.set_ylabel("Number of Transactions")
ax2.grid()

plt.tight_layout()
plt.show()
```


![png](output_50_0.png)



```python
bitcoin_FT[bitcoin_FT['USD'] == bitcoin_FT['USD'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DateTime</th>
      <th>USD</th>
      <th>BTC</th>
      <th>Segwit</th>
      <th>Non-Segwit</th>
      <th>Total Per Day</th>
      <th>% Segwit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2653</th>
      <td>2017-12-22</td>
      <td>2.042737e+07</td>
      <td>1495.748916</td>
      <td>38546</td>
      <td>341466</td>
      <td>380012</td>
      <td>10.143364</td>
    </tr>
  </tbody>
</table>
</div>



Similar, to the price graphs and block rewards, the fees of transactions that miners receive peaked at around December of 2017 at $20,427.370. Yet, different from other graphs, the amount of fees that miners receive from transactions dramatically decreased in the first quarter of 2018 and went near the amount of fees that miners would receive in years before Bitcoin gained traction

## Conclusion

We definitely see that although Bitcoin prices were not as they once used to, there still seems to be a generally positive sentiment about Bitcoin. However, Bitcoin's volatile price has various effects, one of the most significant effect it has is on the miners and nodes of the network. We noted earlier a drop in the hash rate of the Bitcoin network. Coupled with the observation that the incentives for miners to remain in the network has been decreasing with the drop in block rewards and transaction fees, there does seem to be indication that miners are in fact leaving the network. Bitcoin's price volatilty has definitely outweighed its benefits with its use of Blockchain - the overall impact has been negative is deteriorating the health of the network as a whole. The world just may not yet be ready for an unregulated currency.
