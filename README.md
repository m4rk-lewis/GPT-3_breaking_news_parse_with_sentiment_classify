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
        for entry in feed.entries:
            title = entry.title
            if title not in seen_titles:
                if entry.category == "News" or entry.category =="Central Banks" or entry.category =="Technical Analysis":
                    seen_titles.add(title)
                    category = extract_text(entry.category)
                    description = extract_text(entry.description)
                    summary = summarize_text(description[:4000])
                    sentiment = sentiment_classification(description[:4000])  
                    insert_news(cursor, title, summary, sentiment)  
                    print(f"{category} >>> {title} >>> Summary: {summary} >>> Sentiment: {sentiment}")
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
News >>> China Q1 GDP and March activity data due at 0200 GMT (10pm US Eastern time) >>> Summary: China to release GDP and activity data, expecting growth from consumption & infrastructure investment; looking at impact of loan growth, retail sales recovery. >>> Sentiment: 0.3
News >>> Australian weekly Consumer Confidence comes in at its 7th lowest since March of 2020 >>> Summary: ANZ-Roy Morgan Australian Consumer Confidence fell to 7th weakest since Mar 2020 despite RBA no-hike decision, rising housing prices & low unemployment; Inflation expectations rose 0.5ppt to 5.6%. >>> Sentiment: -0.7
Central Banks >>> Bank of Japan Governor Ueda will speak in the Japanese parliament Tuesday, 18 April 2023 >>> Summary: Bank of Japan Governor Ueda to appear before Diet financial committee, reiterating dovish comments to stay the course. >>> Sentiment: 0.0
News >>> JP Morgan forecasts Brent crude oil to US$94 a barrel in Q4 2023 >>> Summary: It discusses how investors' sentiment was hit by weaker demand in Asia, caused by OPEC's decision to cut output, though this will not take effect until next month. JPMorgan forecasts Brent to $94/bbl in Q4 and expects recession at the >>> Sentiment: The sentiment of the article is 0.2 on the scale of -1 to 1 with 0.1 granularity, where -1 is maximum bearishness and 1 is maximum bullishness in relation to equities.
News >>> Deutsche Bank see a near-term S&P 500 rally above 4250 >>> Summary: Deutsche Bank's Bankim Chadha expects Q1 earnings to beat expectations and a weakening US dollar to drive a near term SPX rally above 4250, but cautions of considerable uncertainty. >>> Sentiment: 0.4
News >>> Morgan Stanley on US Stocks -  "we are far from out of the woods with this bear market" >>> Summary: Fed policy shift in March could mean higher yields, negative surprises for investors, Q1 earnings estimates down 15%, business sentiment down, market breadth at a record low and potential for a 15% US stock market fall in even a mild recession. >>> Sentiment: -0.7
News >>> JP Morgan Kolanovic says even a mild recession could cause US stocks to fall by 15% >>> Summary: Marko Kolanovic, JPM's chief global markets strategist, recommends that clients keep equity allocation underweight, & overweight in cash and Japanese stocks due to irrational market factors and potential 15%+ downside. >>> Sentiment: -0.1
Central Banks >>> ICYMI - BIS' head Carstens says interest rates may need to be higher, for longer >>> Summary: BIS's Carstens speaks at Colombia University; warns of high-inflation regime, debt levels, and financial instability. >>> Sentiment: 0.5
News >>> Forexlive Americas FX news wrap 17 Apr: Empire manufacturing index strong. USD rallies. >>> Summary: 3% last monthUS industrial production at 9:15 AM. Estimate 0.5% versus -0.5% last monthUS Empire manufacturing index rose to 10.8%, US yields pushed up, USD strongest, EUR weakest, gold down >>> Sentiment: 3% last monthUS housing starts at 8:30 AM ET. Estimate 1.724M versus 1.723M last monthOverall sentiment: 0.5
Central Banks >>> CIBC targeting USD/JPY down to 123 by end of Q3 >>> Summary: CIBC Research maintains bullish bias on JPY; BoJ's Uchida underlined no market communication of YCC adjustments before adjustment. >>> Sentiment: 0.8
Central Banks >>> Economic calendar in Asia 18 April 2023 - RBA minutes, China Q1 GDP & March economic data >>> Summary: The Reserve Bank of Australia paused its 10-month rate hike cycle and awaits incoming data to assess the effects. China's PBOC injected 20 billion yuan, suggesting an unconcerned view of economic recovery. >>> Sentiment: 0.3
News >>> Trade ideas thread - Tuesday, 18 April 2023 >>> Summary: Share and discuss ForexLive traders' charts, technical analysis, trade ideas, thoughts, and views. >>> Sentiment: 0.0
News >>> US stocks rally into the close and end the day near the highs >>> Summary: Major US indices close near highs, Dow +100.82 pts, S&P +13.69 pts, NASDAQ +34.25 pts, Russell 2000 +21.68 pts, small-cap stocks favored despite no negative territory. >>> Sentiment: 0.9
Technical Analysis >>> EURUSD rebounds into the close >>> Summary: EURUSD is rising as stocks rebound, traders watch 200 hour MA at 1.09436, looking to break above or below it. >>> Sentiment: 0.3
News >>> WTI crude oil settle at $80.83 >>> Summary: Price of WTI crude oil futures down $1.69 or 2.05%, sellers take control and force price to downside. >>> Sentiment: -0.7
Technical Analysis >>> US a two year yield pushes 4.20% for the first time since March 22 >>> Summary: 2-year yield near highs for the day, with Fed's rate hike expectations pushing yields higher; 10-year yield above 100/200 hour MA with momentum. >>> Sentiment: 0.7
Technical Analysis >>> Gold market dynamics: The battle between buyers and sellers is on. >>> Summary: Gold market fluctuates with double bottom at $1981; bullish bias needs to rise above 200-hour and 100-hour moving averages for hope of buyers, sellers challenged to cause further declines. >>> Sentiment: 0.0
Central Banks >>> Feds Barkin: Wants further evidence that inflation is settling back to target >>> Summary: Fed's Barkin wants evidence inflation is settling, economy operating fine; reassured by banking sector. May meeting likely to raise rates 25 points, but inflation still too high. >>> Sentiment: 0.2
News >>> What major earnings releases are expected this week >>> Summary: Companies reporting April 17-21: Johnson & Johnson, Bank of America, Netflix, Goldman Sachs, Lockheed Martin; April 24-28: Coca-Cola, Microsoft, Alphabet, Visa, PepsiCo, McDonald's, Verizon. >>> Sentiment: 0.0
News >>> GBPUSD trades to a new session low. Tests swing area. >>> Summary: GBPUSD has moved to a new session low, testing a swing area near 1.2343-1.23603. Last week's rally failed; sharp selloff today targets 1.2260-1.2282 area. >>> Sentiment: -0.3
Central Banks >>> Feds Barkin is due to speak. Be aware. >>> Summary: Richmond Fed Pres. Barkin is set to speak on inflation and bank lending. >>> Sentiment: 0.2
Technical Analysis >>> Major European indices closed session with mixed results >>> Summary: European indices end mixed; France declined after four-session gains, UK and Spain rose. >>> Sentiment: 0
Central Banks >>> Lagarde: On changing 2% goal, once inflation objective is achieved, we can discuss >>> Summary: ECB President says once inflation objective is achieved, discussion of changing 2% target goal can be had. >>> Sentiment: 0.0
Technical Analysis >>> USDJPY follows yields higher. >>> Summary: USDJPY has risen to highest level since March 15, reaching 134.567 with support from buyers near swing area at 133.74 and 133.87. >>> Sentiment: 0.9
---------------------------------------------------------------------------
'''
```
