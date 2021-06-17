---
layout: post
title: "Rasa搗鼓筆記 - 安裝"
date: 2021-6-17
excerpt: "Rasa安裝教學與詞量源雜七雜八的選取的心得"
tags: [tutorial, python, rasa, chatbot]
feature: https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/FBD52907-3952-42C0-888B-C9A3BC8324E3.jpeg
comments: true
---
好幾年前...大概是前年吧，想要做智能一點的chatbot，用過chatterbot效果不算好；想要用Rasa，大概太老了連安裝都安裝不了，搞了好幾天最後放棄了。由於最近還是皮癢加上個人健康需求還是想要做個比較聰明點的chatbot，最後還是忍不住想要再搗鼓Rasa。因此就試看看唄。經過一天收集閱讀消化資料，找了一堆tutorials，以中文為語言的的基本上內容離不開哪些，看完還是覺得知識和操作很零碎，也不想只是純粹跟tutorial去做，因此稍微找了些資料才去試著安裝，這次終於成功了。下面的步驟是經過測試OK的。

開始講安裝前，快速飄過介紹下Rasa唄。

Rasa是一個open source的machine learning框架，用於構建上下文AI assistant和chatbot。Rasa有兩個主要模塊：<br>
NLU：實現意圖識別和槽值提取，它把用戶的輸入轉換為結構化數據<br>
Core：是一個對話管理平台，用於預測、決定下一步做什麼

還有一個是Rasa X。Rasa X是一個可幫助構建、改進和部署由Rasa框架提供支持的AI assistants的工具。

## 【安裝流程】

### Environment:
建議Python 3.6 - 3.8之間，不support 3.9，尤其打算使用Rasa X。安裝時我用Python 3.7，OS環境是MacOS Big Sur 11.0.1。

### Virtual Environment Setting:
```python
python3 -m venv whatever-name
```

### Upgrade Package:
```python
pip install --upgrade pip
```

### Before Installing:
在Rasa NLU模塊中，提供了一種名為Pipline(管道)配置方式，傳入的消息(Message)通過管道(Pipline)中一系列組件(Component)處理後得到最終的模型。

![](https://rasa.com/docs/rasa/ideal-img/component-lifecycle-img.0111328.1202.png)
<p align="center">上圖參考自Rasa: https://rasa.com/docs/rasa/tuning-your-model</p><br>

管道由多個組件構成，每個組件有各自的功能，比如實體提取、意圖分類、響應選擇、預處理等，這些組件在管道中一個接著一個的執行，每個組件處理輸入並創建輸出，並且輸出可以被該組件之後管道中任何組件使用。當然，有些組件只生成管道中其他組件使用的信息，有些組件生成Output屬性，這些Output屬性將在處理完成後返回。因此選擇components要根據目標需求而定。[這裡](https://jiangdg.blog.csdn.net/article/details/104530994)或[這裏](https://zhuanlan.zhihu.com/p/83566179)有一些components的介紹可以參考。

這些components裡面，其中一個比較重要的是詞向量源、model和分詞。Rasa常用的詞向量源與model有MITIE和spaCy，基本上大部分的tutorial都會走著兩個路線。以中文來說，找到的tutorial都是以MITIE+jieba為主，這是由於spaCy當時不支援中文。spaCy去年開始支援中文，因此可以多一個選項。
至於兩者有什麼差別，簡單來說，如[《ChatBots vs Reality: how to build an efficient chatbot, with wise usage of NLP》](https://towardsdatascience.com/chatbots-vs-reality-how-to-build-an-efficient-chatbot-with-wise-usage-of-nlp-77f41949bf08)說，「Mitie and Spacy are very different libraries from each other: the first oneuses more general-purpose language models, and therefore very slow to train, while Spacy uses more task specific models, and is very fast to train.」。更仔細的話，可以參考以下文章：<br>
RASA NLU语言模型：[https://zhuanlan.zhihu.com/p/331853394](https://zhuanlan.zhihu.com/p/331853394)<br>
Build a Multilingual Chatbot with Rasa and Heroku – Introduction：[https://www.langnerd.com/multilingual-chatbot-with-rasa-and-heroku-part-1/](https://www.langnerd.com/multilingual-chatbot-with-rasa-and-heroku-part-1/)

其他關於components可供參考的文章：<br>
Our experience building chatbots with Rasa — Tuning the NLU pipeline：[https://chatbotslife.com/our-experience-building-chatbots-with-rasa-tuning-the-nlu-pipeline-74a80cd565b8](https://chatbotslife.com/our-experience-building-chatbots-with-rasa-tuning-the-nlu-pipeline-74a80cd565b8)<br>
Rasa简介：[https://zhuanlan.zhihu.com/p/342671832](https://zhuanlan.zhihu.com/p/342671832)

### Installation:
如前述，支援中文的詞向量源與model現在有兩個：MITIE+jieba和spaCy

#### 若用MITIE+jieba安裝如下：
安裝rasa：
```python
pip install git+https://github.com/mit-nlp/MITIE.git
pip install rasa[mitie]
```

安装結巴分詞：
```python
pip install jieba
```

另外還要準備好中文詞向量模型total_word_feature_extractor_zh.dat（密碼：p4vx）[https://pan.baidu.com/s/1kNENvlHLYWZIddmtWJ7Pdg](https://pan.baidu.com/s/1kNENvlHLYWZIddmtWJ7Pdg)，這裡需要將該模型copy到創建的python項目data目錄下（可任意位置），後面訓練NLU模型時會用到。

#### 若用spaCy安裝如下：
```python
pip install https://github.com/howl-anderson/Chinese_models_for_SpaCy/releases/download/v2.0.3/zh_core_web_sm-2.0.3.tar.gz
./spacy_model_link.bash
pip install rasa[spacy]
```

#### 或者全部安裝：
```python
pip install rasa[full]
```

#### 安裝rasa X：
```python
pip install --use-deprecated=legacy-resolver rasa-x --extra-index-url https://pypi.rasa.com/simple
https://rasa.com/docs/rasa-x/installation-and-setup/install/local-mode
```

到這裡總算安裝成功。接下來就要啟動和setting。雖然有想寫成個系列教學文，不過實在沒啥力氣，所以還是別期望太高了。
