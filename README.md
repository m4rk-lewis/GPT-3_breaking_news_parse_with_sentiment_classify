# Breaking financial news summarisation and sentiment classification using GPT-4


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
openai.api_key = "insert-your-API-key-here"


# Step 4: Summarize using GPT-4
def summarize_text(text):
    response = openai.Completion.create(
        engine="text-davinci-002", 
        prompt=f"Please provide a short summary of the following text and remove unnecesary swhitespace: {text}",
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
        engine="text-davinci-002", 
        prompt=f"Please classify this breaking financial news article on a range of -1 to 1 with 0.1 granularity, where -1 is maximum bearishness and 1 is maximum bullishness: {text}",
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

def insert_news(cursor, title, summary, sentiment=None):
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
                seen_titles.add(title)
                description = extract_text(entry.description)
                summary = summarize_text(description[:4000])
                sentiment = sentiment_classification(description[:4000])  
                insert_news(cursor, title, summary, sentiment)  
                print(f"Inserted news: {title} : {summary} : {sentiment}")
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



    Inserted news: It's been quiet this year in EUR/USD trading... too quiet : As we wrap up the first quarter of the year, Deutsche Bank has a look at the euro and notes that EUR/USD volatility and the range of the pair has been surprisingly narrow, particularly given the volatility in fixed income. : 0.5
    Inserted news: USDCAD approaches key 100 day moving average : The USDCAD has been down for four consecutive days and is approaching its 100 day moving average. A move below that level would increase the bearish bias. : 0
    Inserted news: US weekly EIA natural gas inventories -47B vs -54B expected : May Henry Hub natural gas fell below $2.00 yesterday but is up to $2.13 today as we get through the roll. This report won't help. : -0.4
    Inserted news: The GBPUSD moves away from it's 100 hour MA. More bullish bias for the pair. : The GBPUSD is pushing higher in trading today and in the process has seen the price move away from its 100 year moving average (blue line in the chart above). That moving average was sniffed in the early Asian session, but buyers : 0.5
    Inserted news: US stocks extend Wednesday's gains. Nasdaq leads the way. : The major US stock indices are off to a solid start to the day. The Dow and the S&P are on pace for the 2nd monthly increase in 3 months, while the Nasdaq and S&P are looking for the 3rd consecutive : 0.5
    Inserted news: USDJPY trades to a new high. Can the buyers get the pair above 133.00 now? : The US yields are moving to new highs (2 year up 6 bps to 4.14%), and that has given the USDJPY some underlying support as well. The pair is ticking to new highs at 132.91. : 0.5
    Inserted news: South African central bank raises repo rate by 50 bps to 7.75% : The South African Reserve Bank has hiked its main repo rate by 50 bps to 7.75%. USD/ZAR has dropped to a five-week low in the aftermath of the decision. : 0
    Inserted news: South Africa central bank sees 2023 CPI at 6.0% vs 5.4% prior : The original can be found here: https://www.forexlive.com/news/!/risk-to-the-inflation-outlook-seen-to-the-upside-20190424 : 0.2
    Inserted news: The EURUSD breaking higher : The German inflation was a bit higher than expectations and that has caused the EURUSD to move to the upside in early US trading. Looking at the 4-hour chart, the pair is extending above a swing area between 1.0866 : 0.5
    Inserted news: How to Become a Successful IB in Africa's Booming Forex and Crypto Markets : The Finance Magnates Africa Summit (FMAS:23) is coming soon, with the landmark event starting on May 8-10, 2023. Held at the luxurious Sandton Convention Centre in Johannesburg, South Africa, this is one event : It is classified as 0.5.
    Inserted news: US initial jobless claims 198K vs 196k estimate : US initial jobless claims for the week ending March 18 came in at 191k vs 197k estimate. Prior week 191K (unrevised). Initial jobless claims 198K vs 196K estimate. 4 week MA 198.25K : 0.2
    Inserted news: US final Q4 GDP +2.6% vs +2.7% expected : The text discusses the final reading of the US GDP for Q3 which was +3.2%. It also covers personal consumption, core PCE prices, and PCE prices. The article goes on to discuss GDP final sales and corporate profits after tax : 0.2
    Inserted news: OPEC unlikely to tweak oil output policy on Monday - report : OPEC+ is unlikely to change oil policy at Monday's meeting, according to five OPEC+ delegates cited by Reuters. This is no surprise as it's a monitoring meeting and members have repeatedly pledged to keep output unchanged through year end. : 0.0
    Inserted news: The CHF is the strongest and the USD is the weakest as the NA session begins : The CHF is the strongest and the USD is the weakest as the NA session begins. The major currencies are relatively scrunched together to start the trading day. The USDCHF has been trending to the downside after breaking below : 0.1
    Inserted news: Germany March preliminary CPI +7.4% vs +7.3% y/y expected : The views expressed in this article are those of the author and do not necessarily reflect the official policy or position of www.forexlive.com or its editorial staff. Prior +8.7%, CPI +0.8% vs + : 0.0
    Inserted news: ForexLive European FX news wrap: Inflation hope or false dawn? : Today's focus was on inflation numbers from German states and Spain. The early reports led to a fall in bond yields, as headline annual inflation came in softer than February and in the case of Spain, it even came in well below estimates. : 0.4
    Inserted news: Dollar trails amid better risk mood so far today : European indices are keeping gains near 1% while S&P 500 futures are up 16 points, or 0.4%, and that is helping with the overall mood in markets so far. In turn, the dollar is the laggard but : 0.7
    Inserted news: Empire break-up: Alibaba and the six units : One of the most well-known Chinese companies, Alibaba, is about to become six well-known Chinese companies. The e-commerce giant announced that it is going to split into six independent units soon – and its stock celebrated this fact with 14 : It has a bullish tone and is overall positive about Alibaba. I would rate it a 0.8.
    Inserted news: Eurozone March final consumer confidence -19.2 vs -19.2 prelim : Economic confidence falls in October as industrial confidence wanes. Services confidence also falls, though not as sharply. Consumer confidence had been improving for the past five months, but that improvement has now halted. : -0.1
    Inserted news: Russell 2000 Technical Analysis : On the daily chart below, we can see that the market got stuck in a range as soon as it bounced from the 1731 support. The uncertainty is high. On one hand the market is more optimistic as the banking crisis is fading, : The original can be found here: https://www.forexlive.com/news/!/ technical-analysis-eurusd-stuck-in-a-range-as-it-approaches-the-top-20200
    Inserted news: Saxony March CPI +8.3% vs +9.2% y/y prior : The monthly reading reflects a 0.9% increase in price pressures though the drop in headline annual inflation fits with what we have seen from the other state readings earlier. This should set up the national reading later to come in somewhere between 7.2% : 0.2
    Inserted news: Bitcoin is trying to break through the ceiling : The stock market's upbeat mood brought the price of Bitcoin back to the upper limit of the March trading range. In the low-liquid market early in the morning, Bitcoin picked up a wave of stops moving from $28.5K to $29 : 0.1
    Inserted news: XTIUSD Technical Analysis : The original can be found at https://www.forexlive.com/technical-analysis/!/oil-daily-chart-sellers-pile-in-after-breakout-of-range-4-hour-chart : 0.1
    Inserted news: Is Gold Still a “Reliable” Safe-haven Asset? : filed for bankruptcy. </p><p class="MsoNormal">Gold as a hedge against the European debt crisis</p><p class="MsoNormal">The European debt crisis started unfolding in early 2010. This time gold experienced a more profound : filed for bankruptcy. </p><p class="MsoNormal">The precious metal regained its bullish momentum on September 18, 2008, closing at $1,020 per ounce. Gold gained further traction as the Lehman Brothers’ bankruptcy spurred panic
    Inserted news: Bond yields nudge back a little higher as traders push and pull : At first glance, the inflation numbers in German states and Spain today might suggest that price pressures are cooling but I've mentioned countless times already that it comes with a very important caveat. The drop in headline annual inflation is largely to do with base : 0.5


