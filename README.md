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
        prompt=f"please classify the sentiment of this breaking financial news article on a range of -1 to 1 with 0.1 granularity, where -1 is maximum bearishness and 1 is maximum bullishness in relation to equities: {text}",
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
    conn = sqlite3.connect(":memory:")
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE news (title TEXT, summary TEXT, sentiment TEXT)''')
    return conn, cursor

def insert_news(cursor, title, summary, sentiment):
    cursor.execute("INSERT INTO news (title, summary, sentiment) VALUES (?, ?, ?)", (title, summary, sentiment))
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
                    insert_news(cursor, title, summary, sentiment)  
                    print(f"{published} >>> {category} >>> {title} >>> Summary: {summary} >>> Sentiment: {sentiment}")
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
Tue, 18 Apr 2023 01:15:16 GMT >>> Central Banks >>> PBOC sets USD/ CNY reference rate for today at 6.8814 (vs. estimate at 6.8828) >>> Summary: PBOC sets reference rate for the onshore yuan, allowing +/-2% of USD/CNY, no restrictions for USD/CNH. 38bn yuan injected in OMOs. >>> Sentiment: 0.0
Tue, 18 Apr 2023 00:52:39 GMT >>> News >>> "ChatGPT outperforms traditional sentiment analysis methods" to forecast stock prices >>> Summary: ChatGPT language models can predict stock market returns with sentiment analysis of news headlines, outperforming traditional sentiment analysis methods. >>> Sentiment: 0.7
Tue, 18 Apr 2023 00:41:45 GMT >>> Central Banks >>> BOJ is considering a 2025 CPI forecast that is still under target >>> Summary: BOJ is considering a projection to raise consumer prices in 2025; new Governor Ueda will be at first policy meeting. BOJ's communication policies could lead to market volatility. >>> Sentiment: 0.4
Tue, 18 Apr 2023 00:23:41 GMT >>> Central Banks >>> PBOC is expected to set the USD/CNY reference rate at 6.8828 – Reuters estimate >>> Summary: People's Bank of China sets daily midpoint for yuan against USD, allowing +/- 2% fluctuation within a trading band. PBOC intervenes if value approaches limit or experiences volatility to stabilize currency. Moving towards more market-oriented exchange rate >>> Sentiment: 0.0
Tue, 18 Apr 2023 00:16:50 GMT >>> News >>> China Q1 GDP and March activity data due at 0200 GMT (10pm US Eastern time) >>> Summary: China to release Q1 GDP, activity data for March; expect 3.8%YoY growth with consumption and infrastructure investment driving. Loan growth in March could help infrastructure investments. Retail sales recovery monitored for clues on broad-based consumption growth. >>> Sentiment: 0.5
Mon, 17 Apr 2023 23:57:13 GMT >>> News >>> Australian weekly Consumer Confidence comes in at its 7th lowest since March of 2020 >>> Summary: Australian Consumer Confidence falls to 7th weakest since March 2020 despite RBA no-hike decision, rising housing prices and low unemployment. >>> Sentiment: -0.2
Mon, 17 Apr 2023 23:29:45 GMT >>> Central Banks >>> Bank of Japan Governor Ueda will speak in the Japanese parliament Tuesday, 18 April 2023 >>> Summary: Bank of Japan Governor Ueda appears before Diet financial committee to reiterate dovish stance. >>> Sentiment: 0.0
Mon, 17 Apr 2023 23:06:44 GMT >>> News >>> JP Morgan forecasts Brent crude oil to US$94 a barrel in Q4 2023 >>> Summary: OPEC's production cut will lead to a market deficit, while Fed rate hikes and US recession forecasts affect oil prices. JPMorgan expects Brent at $94/bbl in Q4 and a possible bull market. >>> Sentiment: 0.1
Mon, 17 Apr 2023 22:24:43 GMT >>> News >>> Deutsche Bank see a near-term S&P 500 rally above 4250 >>> Summary: Bankim Chadha of Deutsche Bank predicted a near-term S&P 500 rally due to strong Q1 earnings and a weakening USD, but warned of a potential US economic crash. >>> Sentiment: 0.5
Mon, 17 Apr 2023 21:55:07 GMT >>> News >>> Morgan Stanley on US Stocks -  "we are far from out of the woods with this bear market" >>> Summary: Fed policy shift has caused banking failures and could bring further surprises. Market breadth is low, and JP Morgan Kolanovic predicts a 15% drop in US stocks with mild recession. >>> Sentiment: The sentiment expressed in the article is -0.7. The article points to worrying trends in the economy, including a collapse in market breadth and lower earnings forecasts, which suggest that the stock market could be headed for a bearish downturn. The analyst
Mon, 17 Apr 2023 21:44:38 GMT >>> News >>> JP Morgan Kolanovic says even a mild recession could cause US stocks to fall by 15% >>> Summary: Marko Kolanovic, JPM's Chief Global Markets Strategist, recommends underweighting equity allocations & overweighting cash due to irrational factors driving U.S. stocks, & suggests buying Japanese stocks. He predicts a 15%+ downside in the >>> Sentiment: -0.8
Mon, 17 Apr 2023 21:16:01 GMT >>> Central Banks >>> ICYMI - BIS' head Carstens says interest rates may need to be higher, for longer >>> Summary: Agustin Carstens spoke at Colombia University, noting the need for higher and longer-term rates to avoid a long-term "high-inflation regime" and the complex task of central banks due to high debt levels. >>> Sentiment: 0.0
Mon, 17 Apr 2023 20:59:28 GMT >>> News >>> Forexlive Americas FX news wrap 17 Apr: Empire manufacturing index strong. USD rallies. >>> Summary: 3% last month.US stocks rallied into the close and ended near the highs, WTI crude oil settled at $80.83, US two year yield pushed 4.20% for first time since March 22, Feds Barkin wants evidence >>> Sentiment: 4% last monthUS CPI at 8:30 AM. Estimate 0.2% versus 0.4% last month0.9
Mon, 17 Apr 2023 20:55:02 GMT >>> Central Banks >>> CIBC targeting USD/JPY down to 123 by end of Q3 >>> Summary: CIBC Research predicts JPY gains due to BoJ yield curve adjustment, noting further adjustments should not be communicated beforehand. >>> Sentiment: 0.8
Mon, 17 Apr 2023 20:15:39 GMT >>> Central Banks >>> Economic calendar in Asia 18 April 2023 - RBA minutes, China Q1 GDP & March economic data >>> Summary: The Reserve Bank of Australia paused its rate hike cycle due to high inflation, while the People's Bank of China remained subdued, suggesting they are not worried about economic recovery. >>> Sentiment: 0.5
Mon, 17 Apr 2023 20:14:33 GMT >>> News >>> Trade ideas thread - Tuesday, 18 April 2023 >>> Summary: Share and discuss ForexLive trade ideas, charts, technical analysis, and views with fellow traders. >>> Sentiment: 0.0
Mon, 17 Apr 2023 20:07:22 GMT >>> News >>> US stocks rally into the close and end the day near the highs >>> Summary: US indices rally, closing near day highs; Dow, S&P, and NASDAQ all traded negative; Russell 2000 stayed positive. >>> Sentiment: 0.9
Mon, 17 Apr 2023 19:41:59 GMT >>> Technical Analysis >>> EURUSD rebounds into the close >>> Summary: EURUSD is rising as stocks rebound; 200 hour MA at 1.09436 is key barometer for new day. >>> Sentiment: 0.4
Mon, 17 Apr 2023 18:50:10 GMT >>> News >>> WTI crude oil settle at $80.83 >>> Summary: Price of WTI crude oil futures down 2.05%, reached low of $80.47 and high of $82.71, sellers forced price to downside. >>> Sentiment: -0.4
Mon, 17 Apr 2023 18:44:43 GMT >>> Technical Analysis >>> US a two year yield pushes 4.20% for the first time since March 22 >>> Summary: 2-year yields cycle high since Fed tightening, currently trading above 100/200 hour MAs with momentum; 10-year yield testing March 29 high. >>> Sentiment: 0.8
Mon, 17 Apr 2023 17:52:04 GMT >>> Technical Analysis >>> Gold market dynamics: The battle between buyers and sellers is on. >>> Summary: Gold market experiences fluctuations, double bottom at $1981.20, current prices within 200 and 100-hour bands, $1981 break could shift trend downward. >>> Sentiment: 0.6
Mon, 17 Apr 2023 17:29:40 GMT >>> Central Banks >>> Feds Barkin: Wants further evidence that inflation is settling back to target >>> Summary: Fed's Barkin wants evidence of inflation settling to target; believes economy is operating fine with current level of rates. >>> Sentiment: 0
Mon, 17 Apr 2023 17:23:42 GMT >>> News >>> What major earnings releases are expected this week >>> Summary: This week's most influential S&P 500 companies include Johnson & Johnson, Bank of America, Netflix, Goldman Sachs and Lockheed Martin. Later in the week, AT&T, American Express, CSX and AutoNation will be reporting. April 24 >>> Sentiment: 0.2
Mon, 17 Apr 2023 16:36:29 GMT >>> News >>> GBPUSD trades to a new session low. Tests swing area. >>> Summary: GBPUSD moved to a new session low, testing a swing area between 1.2343 and 1.23603, with a lower low of 1.23576. If sellers take control, target is 1.2260-1. >>> Sentiment: -0.3
'''
```
