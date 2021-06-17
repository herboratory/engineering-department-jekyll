---
layout: post
title: "Using scraparazzie to search news and send through LINE Notify in Python"
date: 2020-4-18
excerpt: "scraparazzie教學"
tags: [tutorial, python, google-news-feed, scraparazzie]
feature: https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/AF592C18-5CBF-4C50-B5AC-866209F9E304.jpeg
comments: true
---

When the idea came up to make the package [scraparazzie](https://herboratory.github.io/engineering-department/scraparazzie-release/), the main purpose is to grab news by topics and keywords. Since I don't really need the news content but title, source and publish date, so that I can receive news with further setting and send the results to an IM bot. That's why I made scraprazzie. And, yes, mainly for stock market and agriculture news (eh...target in commodities🤫) Somehow I found a [tutorial](https://bustlec.github.io/note/2018/07/10/line-notify-using-python/) about application in LINE Notify, so let's do it!

**Environment: Python 3.7**

We also need to get the token of LINE Notify, so we need a LINE account and app. Below we will go step-by-step to apply the LINE Notify token.

### STEP 1:

- head to [https://notify-bot.line.me/](https://notify-bot.line.me/) and login
- Click the arrow next to you name at top right > My page

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-04-18-at-11.08.42-1024x597.png)

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-04-18-at-11.10.52-1-1024x590.png)

- Click 【Generate token】

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-04-18-at-11.11.40-1024x593.png)

- Input your token name, this will show at as your notification title.
- Select the first one【1-on-1 chat with LINE Notify】, then click 【Generate token】.

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-04-18-at-11.14.03-1024x593.png)

After clicking 【Generate token】, the token will show. **Please copy and keep the token**! The program need to use this token to communicate with LINE Notify!

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-04-18-at-11.15.33-1024x593.png)

Setting of LINE Notify is done here.

### STEP 2:

- Build the lineNotifyMessage.py
- At the "token", please paste the token you just got from LINE Notify.

```python
from scraparazzie import scraparazzie
import requests
import time
from random import randrange

# LINE Notify的部分｜LINE Notify setting

token = "paste_your_token_here"    # 從LINE Notify獲得的權杖｜Token from LINE Notify
def lineNotifyMessage(token, msg):    # LINE Notify傳訊息的設定｜Message sending setting of LINE Notify
    headers = {
        "Authorization": "Bearer " + token, 
        "Content-Type" : "application/x-www-form-urlencoded"
    }

    payload = {'message': msg}
    r = requests.post("https://notify-api.line.me/api/notify", headers = headers, params = payload)
    return r.status_code
```

# 設定你想發的訊息或東西的部分｜Setting the items or message you want to send
```python
keywords = ["locust infest plague", "Chinese medicine"]
locations = ["Canada"]
chooen_language = ["English"]

def searching_topic(locations, keywords, languages):
    for location in locations:
        location = "{}".format(location)
        for topic in keywords:
            items = "{}".format(topic)
            for language in languages:
                language = "{}".format(language)
                print("Items in progress: " + location + " - " + topic + " - " + language)
                client = scraparazzie.NewsClient(language = language, location = location, query = items, max_results = 1) # 利用scraparazie尋找詢問的關鍵字｜Searching the keywords through scraparazzie
                client.print_news()
                output = client.export_news() # 輸出結果｜Output searching result
                for export_items in output:
                    message = "\n" + "處理關鍵字｜Processing keywords: " + items + "\n" + "標題｜Title: " + export_items['title'] + "\n" + "新聞來源｜Source: " + export_items['source'] + "\n" + "連結｜Link: " + export_items['link'] + "\n" + "發布時間｜Publish date: " + export_items['publish_date']
                    lineNotifyMessage(token, message) # 運行lineNotifyMessage()｜Run lineNotifyMessage()
                sleep_time = randrange (1, 10)
                time.sleep(sleep_time)

searching_topic(locations, keywords, chooen_language)
```

Since I want to search more than one set of keywords, so I just write a function with loops to run. You can also set multiple locations. But a warm reminder that if you want to search different languages, strongly recommend to run separately, or else it may cause empty result. Also, I don't recommend to search huge amount of news posts, so don't set the max\_result too large. For other setting, please feel free to check out [github](https://github.com/herboratory/scraparazzie) of scraparazzie.

### STEP 3:

- Run the code

```python
python lineNotifyMessage.py
```

- Then you can receive news at LINE Notify.

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/WhatsApp-Image-2020-04-18-at-17.05.08-576x1024.jpeg)

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/WhatsApp-Image-2020-04-18-at-17.04.36-576x1024.jpeg)

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/WhatsApp-Image-2020-04-18-at-17.04.37-576x1024.jpeg)

This programe only get the news and send to LINE Notify. If you want to set the program runs with schedule, or run on cloud, just Google and look for the setup.

Except LINE Notify, it can be use as Slack bot to send news to workspace. Hmm...this reminds me I was working at a department quite some years ago for a very short while, you know, in the morning (even earlier than I arrived - I most arrived top-3 earliest of the whole office), I always saw another team to dig into newspaper and newspaper cutting.🤭 Going through programming is much more convenient. 😬
