---
layout: post
title: "在Python使用scraparazzie尋找新聞並透過LINE Notify發送"
date: 2020-4-18
excerpt: "scraparazzie教學"
tags: [tutorial, python, google-news-feed, scraparazzie]
feature: http://herboratory.github.io/assets/img/AF592C18-5CBF-4C50-B5AC-866209F9E304.jpeg
comments: true
---

當初做[scraparazzie](https://herboratory.ai/portfolio/scraparazzie/)這個套件的目的，是希望可以抓自己有興趣的topic和keywords的新聞傳送到某個IM的bot，就能根據之後自己再設定隨時看到新聞，方便自己炒股和追蹤農業災害新聞（啊...因為想要炒commodity嘛🤫）。但我並不需要內文，只需要標題、新聞來源和發布時間就好，有興趣再click連結去看。偶然看到LINE Notify的[教學](https://bustlec.github.io/note/2018/07/10/line-notify-using-python/)，就拿來實作一下吧。

**開發環境：Python 3.7**

另外也需要LINE Notify的權杖，因此會需要有LINE帳號。接下來先step-by-step介紹申請LINE Notify權杖的部分。

### STEP 1:

- 至 [https://notify-bot.line.me/zh\_TW/](https://notify-bot.line.me/zh_TW/) 進行登入
- 點選右上方 帳號名稱選單中的「個人頁面」（My page）

![](assets/img/Screenshot-2020-04-18-at-11.08.42-1024x597.png)

![](assets/img/Screenshot-2020-04-18-at-11.10.52-1-1024x590.png)

- 點選【發行權杖】（Generate token）

![](assets/img/Screenshot-2020-04-18-at-11.11.40-1024x593.png)

- 輸入自訂「權杖名稱」, 這邊設定的名稱會在出現在提醒訊息的Title
- 選擇第一個【透過1對1聊天接受LINE Notify的通知】（1-on-1 chat with LINE Notify） ，然後按下「發行」（Generate token）

![](assets/img/Screenshot-2020-04-18-at-11.14.03-1024x593.png)

按下「發行」後，就會顯示權杖。**請將權杖內容「複製」並記下來**！程式與LINE Notify溝通就是靠這個權杖！

![](assets/img/Screenshot-2020-04-18-at-11.15.33-1024x593.png)

LINE Notify設定的部分大概到這裡結束。

### STEP 2:

- 建立 lineNotifyMessage.py 檔案
- 在token部分，記得將剛剛在LINE Notify記下的權杖貼上去

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

# 設定你想發的訊息或東西的部分｜Setting the items or message you want to send
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

由於想要用多個關鍵字找新聞，因此就直接寫個def和迴圈跑。location當然也可以設多個。若要找不同語言，強烈建議分開跑新聞，不然很容易會出現找不到結果的狀況。同時也不建議找太多篇新聞，因此max\_result別設太大。其他設定可以去scraparazzie的[github](https://github.com/herboratory/scraparazzie)看看。

### STEP 3:

- 測試執行如下：

```python
python lineNotifyMessage.py
```

- 然後就能收到在LINE Notify傳送來的新聞

![](assets/img/WhatsApp-Image-2020-04-18-at-17.05.08-576x1024.jpeg)

![](assets/img/WhatsApp-Image-2020-04-18-at-17.04.36-576x1024.jpeg)

![](assets/img/WhatsApp-Image-2020-04-18-at-17.04.37-576x1024.jpeg)

這個程式只是跑新聞出來而已。若想要變成定時，或者放在雲端跑的話，可以尋找相關的文章設定。

除了LINE Notify，也可以做成Slack bot，定時發新聞去workspace裡面。唔...想起以前曾經短暫時間在某機關工作，早上看到一堆人埋首在一疊報紙看報導剪報🤭，有這樣的程式還真方便多了😬
