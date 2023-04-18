# Breaking financial news summarisation and sentiment classification using GPT-3


``` python
# Step 1: Install required packages
!pip install feedparser beautifulsoup4 openai
```


``` python
# Step 2: Download and parse the RSS feed
import feedparser
def get_feed(url):
    return feedparser.parse(url)
```

``` python
# Step 3: Extract HTML from the description
from bs4 import BeautifulSoup

def extract_text(html):
    soup = BeautifulSoup(html, "html.parser")
    return soup.get_text()
```

``` python
import openai
openai.api_key = "insert-openAI-API-here"


# Step 4: Summarize using GPT-3
def summarize_text(text):
    response = openai.Completion.create(
        engine="text-davinci-003", 
        prompt=f"remove all unnecesary whitespace and formatting, then provide a short summary of less than 30 words of the following text: {text}",
        temperature=0.7,
        max_tokens=50,
        top_p=1,
        frequency_penalty=0.5,
        presence_penalty=0,
    )
    return response.choices[0].text.strip()

# Step 5: Sentiment classification
def sentiment_classification(text):
    response = openai.Completion.create(
        engine="text-davinci-003", 
        prompt=f"classify the sentiment of this breaking financial news article as a numerical float, with a range of -1 to 1 with 0.1 granularity, where -1 is maximum bearishness and 1 is maximum bullishness in relation to equities: {text}",
        temperature=0.7,
        max_tokens=50,
        top_p=1,
        frequency_penalty=0.5,
        presence_penalty=0,
    )
    return response.choices[0].text.strip()
```


``` python
# Step 6: Store the summarized information in a SQLite database
import sqlite3

def create_db():
    conn = sqlite3.connect('news.db')
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS news (published TEXT, title TEXT, summary TEXT, sentiment TEXT)''')
    return conn, cursor

def insert_news(cursor, published, title, summary, sentiment):
    cursor.execute("INSERT INTO news (published, title, summary, sentiment) VALUES (?, ?, ?, ?)", (published, title, summary, sentiment))
    # Commit the changes to the database
    conn.commit()
```


``` python
# Step 7: Continuously monitor the RSS feed for updates
import time

def monitor_feed(url, conn, cursor, interval=60):
    seen_titles = set()
    while True:
        feed = get_feed(url)
        # print(feed.entries[0])
        for entry in feed.entries:
            title = entry.title
            if title not in seen_titles:
                if entry.category == "News" or entry.category =="Central Banks" or entry.category =="Technical Analysis":
                    seen_titles.add(title)
                    category = extract_text(entry.category)
                    published = extract_text(entry.published)
                    description = extract_text(entry.description)
                    summary = summarize_text(description[:4000])
                    sentiment = sentiment_classification(description[:4000])  
                    insert_news(cursor, published, title, summary, sentiment)  
                    print(f"{published} >>> {category} >>> {title} >>> Summary: {summary} >>> Sentiment: {sentiment}")
        # Close the database connection and sleep
        conn.close()
        time.sleep(interval)
```


``` python
'''
# This code will download the RSS feed and process new entries every 60 seconds. 
Adjust the interval parameter in monitor_feed() to control how often the feed is checked.
'''
if __name__ == "__main__":
    url = "https://www.forexlive.com/feed"
    conn, cursor = create_db()
    monitor_feed(url, conn, cursor)
```

``` python
'''
Tue, 18 Apr 2023 10:30:56 GMT >>> News >>> Stocks in a better mood so far today >>> Summary: Stocks up on optimistic tone; Eurostoxx, DAX, CAC 40 and FTSE all rising. >>> Sentiment: 0.7
Tue, 18 Apr 2023 09:40:04 GMT >>> Technical Analysis >>> GBPUSD Technical Analysis - Rangebound >>> Summary: GBPUSD has failed to break above range top, boosted lower by US data. Now bouncing near prior swing, possible rally towards 1.2444. Sellers await 1.2344 break for bearish setup. >>> Sentiment: 0.0
Tue, 18 Apr 2023 09:36:26 GMT >>> Technical Analysis >>> USDCAD Technical Analysis - Bearish Trend Returns >>> Summary: Bearish trend for USDCAD intact, Retail Sales and hawkish Fed remarks boost USD, buyers need break of trendline to gain conviction. >>> Sentiment: -0.2
Tue, 18 Apr 2023 09:32:03 GMT >>> Technical Analysis >>> EURUSD Technical Analysis >>> Summary: EURUSD rejected at February high, double top likely; buyers may lean on trendline for support. >>> Sentiment: 0.3
Tue, 18 Apr 2023 09:00:13 GMT >>> News >>> Germany April ZEW survey current conditions -32.5 vs -40.0 expected >>> Summary: German economic recovery continues; ZEW headline reading highest since June, but outlook reading misses estimates. >>> Sentiment: 0.8
Tue, 18 Apr 2023 08:54:16 GMT >>> News >>> Dollar extends drop on the session >>> Summary: Dollar falls in European morning trade; GBP/USD buoyed by strong wages data, prompting BOE rate hike anticipation; US futures hold slight gains. >>> Sentiment: 0.7
Tue, 18 Apr 2023 07:57:47 GMT >>> News >>> JP Morgan, Citi upgrade China 2023 full-year GDP growth forecast >>> Summary: JP Morgan and Citi upgrade China's 2023 GDP growth forecast to 6.4% and 6.1%, respectively, due to post-Covid recovery. >>> Sentiment: 0.9
Tue, 18 Apr 2023 07:21:12 GMT >>> News >>> Dollar pinned lower to start the session >>> Summary: Dollar drops across the board; EUR/USD, GBP/USD, AUD/USD, NZD/USD all up. >>> Sentiment: 0.0
Tue, 18 Apr 2023 07:17:21 GMT >>> News >>> European equities slightly higher at the open today >>> Summary: European stocks rise slightly; US futures more tentative; optimism in Europe despite possible central bank tightening. >>> Sentiment: 0.6
Tue, 18 Apr 2023 06:31:08 GMT >>> Central Banks >>> BOJ's Ueda: Positive signs are emerging in prices, wage growth >>> Summary: Central Bank likely to achieve inflation target, but no rush to review fiscal policy. Eyes on next week's meeting for inflation outlook. >>> Sentiment: 0.3
Tue, 18 Apr 2023 06:09:05 GMT >>> News >>> Eurostoxx futures +0.2% in early European trading >>> Summary: German DAX and UK FTSE futures up 0.2% after slight drop yesterday, US futures flat. >>> Sentiment: 0.0
Tue, 18 Apr 2023 06:07:42 GMT >>> Technical Analysis >>> S&P 500 technical analysis from 2009 todate: The big picture >>> Summary: S&P 500 technical analysis reveals long-term uptrend with bullish outlook. Channel patterns, candlestick analysis, and understanding of market/economy disconnect used to navigate stock market. >>> Sentiment: 0.7
Tue, 18 Apr 2023 06:01:21 GMT >>> News >>> UK March payrolls change 31k vs 98k prior >>> Summary: Unemployment rate slightly higher than expected, but payrolls still positive and wages running hot, keeping BOE alert to inflation. >>> Sentiment: 0.7
Tue, 18 Apr 2023 05:20:54 GMT >>> News >>> Trade ideas thread - European session 18 April 2023 >>> Summary: Dollar is making gains, Fed pricing being repriced, balance between recession and safety flows for dollar. >>> Sentiment: 0.2
Tue, 18 Apr 2023 04:42:48 GMT >>> News >>> UK labour market data on the agenda in Europe today >>> Summary: Dollar rises after last week's dip, higher bond yields and a less dovish Fed outlook drive markets; UK jobs report to affirm job market strength but wages will determine BOE's next move. >>> Sentiment: 0.6
Tue, 18 Apr 2023 03:35:23 GMT >>> News >>> ForexLive Asia-Pacific FX news wrap: Chinese economy rebounds strongly in Q1 >>> Summary: China's Q1 GDP data and retail sales numbers impressed but markets remain unphased. Asian equities are mostly higher, and the aussie is slightly higher reacting to RBA minutes. BOJ will not be perturbed by fiscal constraints if policy >>> Sentiment: 0.7
Tue, 18 Apr 2023 02:57:52 GMT >>> News >>> Fairly muted reaction to China's GDP data >>> Summary: China's Q1 optimism based on Covid-19 restrictions being lifted, but global headwinds persist, causing muted market reaction. >>> Sentiment: 0.4
Tue, 18 Apr 2023 02:23:33 GMT >>> Central Banks >>> BOJ's Ueda: Buying government debt is part of monetary policy >>> Summary: BOJ bond purchases not aimed at monetising government debt; debate expected if BOJ moves away from ultra-easy policy. >>> Sentiment: 0.2
Tue, 18 Apr 2023 02:08:18 GMT >>> News >>> China Q1 GDP +2.2% vs +2.2% q/q expected >>> Summary: GDP grows faster than expected in Q1 at 4.5% y/y, with retail sales and industrial output also improving. >>> Sentiment: 0.8
Tue, 18 Apr 2023 01:56:03 GMT >>> Central Banks >>> BOJ's Uchida: Fiscal constraints won't undermine ability to carry out monetary policy >>> Summary: BOJ unlikely to make any major changes at next week's monetary policy meeting. >>> Sentiment: 0.0
Tue, 18 Apr 2023 01:34:36 GMT >>> Central Banks >>> RBA minutes show strong case to pause and reassess need for tightening at future meetings >>> Summary: Board considered rate hike in April, decided to pause; assessing inflation, jobs, consumer spending, business conditions; inflation high, labour market loosened but tight, bank stress causing tighter global financial conditions. >>> Sentiment: 0.3
Tue, 18 Apr 2023 01:33:17 GMT >>> News >>> Major currencies little changed so far after dollar rebound yesterday >>> Summary: USD narrowly held its ground before falling, leading to a rebound in US trading. Major currencies in Asia Pacific trading are little changed with narrow ranges. USD/JPY is up to one-month highs, EUR/USD is slipping back below 100 >>> Sentiment: 0.0
Tue, 18 Apr 2023 01:15:16 GMT >>> Central Banks >>> PBOC sets USD/ CNY reference rate for today at 6.8814 (vs. estimate at 6.8828) >>> Summary: PBOC set onshore and offshore yuan rates, a strong or weak rate is a signal and OMOs inject 38bn yuan with 50bn maturing, net drain of 12bn. >>> Sentiment: 0.0
---------------------------------------------------------------------------
'''
```
