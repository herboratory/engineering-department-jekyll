---
layout: post
title:  "Scraparazzie 發佈｜Release"
date:   2020-04-15
excerpt: "能利用關鍵字或選擇特定topic搜尋Google News feeds的Python package | Python package for searching specific topic or keywords of Google News feeds"
project: true
tag:
- scraparazzie 
- python
- pypi
- google_news_feeds
- package
- pip
comments: false
---
#### 專案資料：
scraparazzie是一個能利用關鍵字或選擇特定topic搜尋Google News feeds的Python package。

大概花了四天時間時間做了scraparazzie，一個能搜尋Google News feeds的套件。這個套件是以gnewsclient為框架，進行修改。在原有基礎上增建功能。

#### gnewsclient原有功能：
1. 能設定語言、地區、topic和output結果項數；
2. 能顯示新聞圖片連結和影片資料；

#### scraparazzie所修改功能：
1. output格式增加發佈日期時間；
2. 除了原有topic搜尋，增加關鍵字搜尋（但只限於topic或關鍵字二選一搜尋，並以關鍵字為優先搜尋項目）；
3. 去除顯示新聞圖片連結和影片資料
4. 搜尋結果從最新到最舊列出；
5. 匯出list功能

當初寫這個套件是因為個人需求，需要一天固定時間搜尋世界各地媒體發新聞標題、發布來源、發佈日期時間等資訊給我再作處理。之前找過不少教學，通常用是用newspaper3k或者自行用beautifulsoup自己寫出來。雖然也有Google News的，但是功能不多，這個套件有什麼特別呢？主要是Python能用的Google News套件不多，github上相關的大概有google-news-scraper、GoogleNews、google_news、gnp和gnewsclient。基本上比較能用、比較符合我的需求只有gnewsclient，但偏偏沒有顯示發佈日期時間功能。本來直接想在package上改來用，後來發現不好改，而且有其他功能想要加進去。把心一橫就把package的框架保留，裡面爬新聞、output新聞部分另外處理。搗鼓了好幾天、撞牆了好幾天，經過一堆大神協助，今天終於進入尾聲。收尾前，看到gnewsclient有人問能否加入搜尋新聞功能，看了下不難，就加入關鍵字搜尋功能。然後就把給打包成package，發出人生第一個python package。（感覺是否應該慶祝一下？畢竟作為個非專業的哎呀碼農...😬）發出後才想到想要把結果output成list方便後續處理，然後又搗鼓了一下，然後就已經是1.2.4了。

v1.2.4為相對穩定版本，在更新成v1.2.4過程再測試遇到的情形都有處理，因此出現error的情形應該會減少。若有任何問題，請到Github那邊issue吧。

#### Project Information:
scraparazzie is a Python package for searching specific topic or keywords of Google News feeds.
It had been spending for around 4 days to deal with scraparazzie, a python package for searching Google News feeds. This package is based on gnewsclient as framework to modify and enhance features.

#### Features that gnewsclient have:
1. Available to set language, location, topic and number of output items for searching;
2. Available to show image and video (if there is) link of the news

#### Features that scraparazzie have:
1. Available to show publish datetime;
2. Except topic, keywords searching is included (either topic or keyword searching, and keyword searching is the top priority searching);
3. Take away the feature to show image and video (if there is) link of the news;
4. Result shows from latest to oldest;
5. Export as list

The main reason I decided to deal with this project is personal use - I need a something that can help me to collect latest news with title, source, link and publish date at fixed time for further application. Of course I searched tons of tutorial, mostly suggesting to do it through newspaper3k or beautifulsoup. There are some package and code for Google News on github, e.g. google-news-scraper, GoogleNews, google_news, gnp and gnewsclient. However, there are not many features and I can't find a package that fits all my requirement. gnewsclient is the best one I can find, but still I can't get publish datetime. At the beginning I wanted to modify directly from the package, and I realised it's not an easy job; furthermore, I might want to plug some more features. In the end I just left the framework of gnewsclient and reformed the news processing part. With tons of trial & error and precious help from excellent programmers, the package works as I want...and yes, I can temporary take a break from here. Before get things done, there was a post few years ago at gnewsclient made a request to put keyword searching feature but no feedback. It didn't look difficult and didn't take much effort to modify the code. After that, I packed the package - my very first Python package EVER in my life (sounds like I should celebrate? As a non-professional coder...😬). It was kind of silly that I realised I should have put the export-as-list feature and debug few tiny error of the Readme.md, so kind of taking me some time to deal with it - and become v1.2.4.
v1.2.4 is the relatively stable version. I also debug some situation for newly installation that may face, there should be less error. If there is question or enquiry, bugs reporting and things, please issue at Github.

<center>![Current Mood](https://github.com/herboratory/engineering_department/raw/master/assets/img/ea67a62202f703a351e20354be1d7876-300x262.jpg)</center>
