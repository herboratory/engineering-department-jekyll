---
layout: post
title: "如何在Telegram利用python-telegram-bot建立一個中醫經典古籍《素問》搜尋chatbot"
date: 2020-7-25
excerpt: "利用python-telegram-bot建立chatbot教學"
tags: [tutorial, python, telegram, chatbot]
feature: https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/76BB8BE7-2278-4F8A-9B8D-D630AA6E3C20.jpeg
comments: true
---

### 【前言】

Telegram最近在香港和台灣開始比較為人所知而使用。一方面是相對比較安全，而且有group chat、channel等功能；另一方面是，比起LINE，Telgram的chatbot的限制沒這麼多，而且是**免費！免費！免費！**（這是重點之重🤫）對於開發budget非常不多的人來說，Telegram是一個不錯的選項。

用Python寫Telegram chatbot最大的好處是有現成的package可用，例如[python-telegram-bot](https://python-telegram-bot.org)和[pyTelegram](https://github.com/eternnoir/pyTelegramBotAPI#telebot)，當然也能透過Flask、requests自行搗鼓出來。以套件來說，tutorial方面，不管是中、英文，套件以python-telegram-bot比較多，而且有專屬自己的、在Telegram討論group發問（不過感覺有時候不太friendly就是，而且問的人很多，有時候一不小心有人回答了對話也飄走了）；pyTelegram則在YouTube教學比較多。經過評估，最後決定用python-telegram-bot的套件。

寫完這個chatbot的tutorial，暫時不會再隨意寫程式，因為想要回歸中醫本行去開發別的東西，而且想要去乖乖上課improve CS知識方便之後寫web app。寫這篇收山之作，是因為之前寫算盤子君🤖️（@glochidion\_bot，大概暫時是唯一一個尋找和查詢有招收香港學生的台灣的大學系所和老師專長資料的Telegram chatbot）困難重重，一是不少python-telegram-bot的tutorial已經過期，二是碰壁試過撞牆多次後才求救叫天不應叫地不聞，有人聞了卻一句戳說是伸手黨，之前撞牆本來就煩躁，加上一個「伸手黨」把自己曾經努力過被抹殺，煩躁就如星火燎原那般燃燒起來，更TM的不爽了。既然如此，就決定再寫一個Telegram chatbot，專門給新手入門和伸手黨使用🤓，讓撞牆撞到了無生趣、非專業背景但想要入門的同好有個好開端。這個《素問》搜尋🔍chatbot是節取自徐長卿君🤖️（@cynanchum\_bot，大概算是首個中醫經典古籍搜尋chatbot，除了《素問》，還有《傷寒論》、《神農本草經》和《難經》）的《素問》搜尋部分。沒有中醫背景的人沒關係，就當作是設計一個搜尋機器人去看就好，因為框架都差不多。

至於看完這個tutorial會學到啥？

1. 如何從零開始設計chatbot - 從寫之前要注意的到寫程式到放到Google Cloud Platform的Compute Engine上跑
2. 如何用python-telegram-bot寫chatbot - 除了一般對話對應，還有如何加入inline keyboard（inline keyboard讓我撞牆很久簡直是崩潰邊緣，這部分簡直是血書...因為python-telegram-bot的demo寫的很莫名其妙根本不知道想要表達啥，其他tutorial要不過期要不就不work）

### 【開始寫之前 - 注意事項】

在開始實際寫之前，先說下一個Telegram chatbot設計需要注意的事項。

首先，Telegram的chatbot default是不能主動跟user有任何動作，即使曾經使用過也不行🙅‍♂️。

對於首次使用某個chatbot的user，第一個command給chatbot就是/start，這是每個bot設計的時候非常強烈建議要有（我沒試過設計沒有/start的，所以也不知道後果會怎樣，推測可能chatbot會不理你吧）。之後可以直接用其他command去跟chatbot溝通，也可以繼續用/start。

其次，根據個人需求，可以設計/help幫助指引user如何使用chatbot。這個設計對於使用有多功能的chatbot來說很重要。就如第一次去有目的的逛大的shopping mall，為了省事方便，都會找顧客服務中心問，同樣道理。

第三，非常強烈建議設計chatbot時要規範使用範圍。這是啥意思呢？意思是當用戶輸入一些用途以外的字眼，chatbot不會愣在哪不回應或甚至直接down機（user很會把機器玩壞滴～），而是有feedback給user，例如說“挖垮薄吼請重新輸入”之類的。

第四，在程式部分要有error log，這是方便開發時出現error會有指引告訴你哪裡出問題，不管是自己debug還是找別人求救時有個方向。

第五，建議開始動手寫chatbot之前先想好、設計好chatbot的流程。例如會想要chatbot做啥、達到「做啥」這個目的需要有什麼步驟和需要啥部件；仔細一點的話，可以順便想對話內容，例如想chatbot的語氣、回應方式之類的。

接下來會根據接下來要寫的《素問》搜尋chatbot，寫一個chatbot設計流程。

### 【開始寫之前 - 寫設計流程】

在開始寫chatbot之前先寫設計流程，除了可以幫助釐清自己的想法，也可以更清楚看到整體架構，從而看到整體架構有否缺陷。同時會更清楚自己寫的時候在寫什麼，不會出現東寫一塊、西加一塊。以下就開始寫個《素問》搜尋🔍chatbot的設計流程吧！

**\[設計流程\]**

這個chatbot可以幹嘛：讓user input keyword搜尋《素問》內容句子。

開發環境：Python 3.7

使用套件：python-telegram-bot

特別點：python-telegram-bot中，一個command和非command功能會以一個def對應一個dispatcher的方式才能運作

檔案架構： # 建議使用虛擬環境venv，獨立開個folder儲存運行。以下也會預設在獨立folder的venv進行。若不知道如何設置虛擬環境，可參考[這裡](https://docs.python.org/3/library/venv.html)。

- 主程式：cynanchum\_bot.py
- 存放搜尋用資料的資料夾：data #資料以JSON形式儲存

程式架構：

1. 需要import的package
2. 允許logging用以紀錄錯誤時哪裏出問題
3. Telegram chatbot的基礎設定，如token、bot、updater （這部分會建議大家用package之前看看demo）
4. 指令/start：用來給user首次啟動chatbot
5. 指令/about：解釋這個bot的背景，順便給herboratory打打廣告
6. 指令/help：指引用戶如何使用chatbot
7. 指令/suwen：搜尋《素問》的程式部分
8. getClickButtonData：在/about用到inline keyboard，而inline keyboard在user按下按鈕後，chatbot會根據inline keyboard的指令給予相對應的feedback。回應的setting就在這裡
9. reply\_handler：只要輸入非指令的字眼都會跑這部分回應user
10. error\_handler：出現error會跑這部分回應user
11. error：用來顯示error log。error\_handler是對user回應；error是給開發者知道出啥問題
12. main：所有def的dispatcher會放到這裡。python-telegram-bot的chatbot啟動待機的部分也會放到這裡。
13. 運行main

【以上】

有了設計流程，就知道接下來的工作內容 - 就是正式開始寫chatbot了。

### 【終於開始寫 - 起手式：先去找BotFather給自己的chatbot蛾子挖個蘿蔔坑】

既然要寫Telegram chatbot，當然得讓Telegram知道這個bot的存在。跟LINE不一樣的是，Telegram有個BotFather專門管理集中閣下的chatbot。當有了Telegram帳戶後，就能@BotFather去給chatbot挖蘿蔔坑。詳細步驟有前人已經寫得很好，就不在這裡覆述。大家可以根據[這裡](https://medium.com/@zaoldyeck/手把手教你怎麼打造-telegram-bot-a7b539c3402a)的步驟去註冊個坑，獲得token。token會在寫程式時會用到。

BotFather除了註冊chatbot，還能幫你的chatbot做設定，例如之後想要更改顯示名稱、加入/修改chatbot的關於、描述、圖片等。要設定很簡單，在@BotFather輸入/start就好。注意的是，例如徐長卿君🤖️的「@cynanchum\_bot」，決定了是不能更改的，但是「徐長卿君🤖️」是可以在BotFather更改。

### 【終於開始寫 - 程式部分】

程式部分根據設計流程部分，把相關的東西寫好即可。Telegram chatbot的command，寫完是可以獨立測試。建議每寫好一個command或功能部分先跑看看測試，以免之後一次過測試出現一大堆error還不知道該如何入手debug。

#### 【寫ing - import的啥東】

程式架構第一項就是import package。學過寫過Python都知道要import package。

這裡我們需要import用到的python-telegram-bot package

```python
import time
import json    #用來讀取搜尋資料檔
import os
import os.path
import logging    #顯示log
import telegram    #運行telegram用
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters, CallbackContext, ConversationHandler 
```

Updater是每個chatbot都會用到。如名字，對話會需要update，就是靠這個
CommandHandler就是用來處理輸入command用的部分
CallbackQueryHandler：在about用到inline keyboard時的按鈕指令部分就靠這個處理
MessageHandler：chatbot處理對話訊息的部分
Filters：在reply_handler用到，設定若非設定command會回覆用戶不知道說啥的訊息時的部分，配合MessageHandler用
CallbackContext：這個用來pass callback。這裡主要針對在error和error_handler時使用。但根據撞牆得知，這個版本v12會用到CallbackContext，但聽聞v13會改。所以就照樣先跟著用就好。
ConversationHandler：如其名，就是處理對話的部分

```python
from telegram import InlineKeyboardMarkup, InlineKeyboardButton
#InlineKeyboardMarkup, InlineKeyboardButton：建立inline keyboard必須用到的部分
```

#### 【寫ing - logging】

import完會用到的東西，就把顯示log的部分加進來。這部分基本上就是根據python-telegram-bot的用法照樣copy & paste過來就好。

```python
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)

logger = logging.getLogger(__name__)
```

這部分基本上就是根據python-telegram-bot的default用法照樣copy & paste過來就好。基本上[那面](https://github.com/python-telegram-bot/python-telegram-bot/tree/master/examples)每個demo都是這樣的用法。

#### 【寫ing - Telegram chatbot的基礎設定】

Telegram chatbot的基礎設定，包括chatbot的token - 也就是去找@BotFather挖蘿蔔坑獲得的一串。不記得的話就去@BotFather看看。

```python
# 在基礎設定，要創造Updater，將token pass給Updater。
# 注意的是，在v12中要加入“use_context = True”（之後版本不需要），用於有新訊息是回應。
# 此外TOKEN、bot和updater要放在def外面。若只放在main()會出現錯誤
TOKEN = "paste-your-token-here"
bot = telegram.Bot(token = TOKEN)
updater = Updater(TOKEN, use_context = True)
```

#### 【寫ing - 指令/start】

在python-telegram-bot，指令都會一個def對應一個後面會在main寫的dispatcher。這裡的/start command，預設user輸入/start後會回覆跟user打招呼，還有提供基本的「救命」指令，例如/help、/about。

```python
def start_handler(update, context: CallbackContext):

    # chatbot在接受用戶輸入/start後的output內容
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING) # 會顯示chatbot正在輸入中，增加對話真實感
    time.sleep(1) # 在顯示輸入中後停頓1秒，然後顯示下一句code的文字
    update.message.reply_text("Hello! 你好👋，{}！我是徐長卿君🤖".format(update.message.from_user.first_name)) # 給user的output
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("徐長卿君🤖️是個chatbot，能根據關鍵字搜尋《黃帝內經素問》的內容\n\n❓關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭，請輸入 /about") # 給user的output。output可以分開多次使用update.message.reply_text()。
```

update.message.reply\_text()是chatbot回復user的line，若想要分段顯示，可以重複使用。

在程式輸入/help、/about等command時，在Telegram的dialogue是會顯示成可以按的command。若相關command還沒建立或沒建立好而去按，可能會造成error。若只是測試顯示而不去按是沒問題的。

#### 【寫ing - main】

預設寫的順序main會在很後面，但為啥現在就要寫呢？因為是要測試用。

```python
def main():
    """啟動bot"""
    # 設定使用dispatcher，用來以後設定command和回覆用
    dp = updater.dispatcher
    dp.add_handler(CommandHandler("start", start_handler)) # 啟動chatbot
    #dp.add_handler(CommandHandler("help", help_handler)) # 顯示幫助的command
    #dp.add_handler(CommandHandler("about", about_handler)) # 顯示關於徐長卿君🤖️的command
    #dp.add_handler(CommandHandler("suwen", suwen_handler)) # 《素問》中文版搜尋功能
    #dp.add_handler(CallbackQueryHandler(getClickButtonData)) # 設定關於徐長卿君🤖️的按鈕連結
    #dp.add_handler(MessageHandler(Filters.text, reply_handler)) # 設定若非設定command會回覆用戶不知道說啥的訊息
    #dp.add_error_handler(error_handler) # 出現任何非以上能預設的error時會回覆用戶的訊息內容

    # 專門紀錄所有errors的handler，對應def error()
    #dp.add_error_handler(error)

    # 啟動Bot。bot程式與Telegram連結有兩種方式：polling和webhook。
    # 兩者的差異可以參考這篇reddit的解釋：https://www.reddit.com/r/TelegramBots/comments/525s40/q_polling_vs_webhook/。
    # 在python-telegram-bot裡面本身有built-in的webhook方法，但是在GCE中暫時還沒摸索到如何設定webhook，因此polling是最便捷的方法。
    updater.start_polling()

    # 就是讓程式一直跑。
    # 按照package的說法“start_polling() is non-blocking and will stop the bot gracefully.”。
    # 若要停止按Ctrl-C 就好
    updater.idle()

#運行main()，就會啟動bot。
if __name__ == '__main__':
    main()
```

由於根據大綱已經知道要寫那些command和部件，因此可以偷懶預先設定各個dispatcher，把暫時不用的先#起來。set dispatcher就是update.dispatcher.add\_handler()。而為了省事，update.dispatcher這裡就以dp代替，因此上面的code就變成dp.add\_handler()。

大家可能留意到，在add\_handler()裡面，若是對應command的話，CommandHandler("command", function\_name)的格式；若是等下會用到的inline keyboard發出的指令，就會用CallbackQueryHandler(function\_name)；若是處理command以外的用字回應，會使用MessageHandler(Filters.text, function\_name)；若是處理error的部分就用dp.add\_error\_handler(function\_name)（例如我這裡的function命名是error，就是dp.add\_error\_handler(error)）。這些dispatcher都是python-telegram-bot常用的。基本上根據格式用即可。

啟動運行chatbot，就是updater.start\_polling()。運行方法在code上面有解釋，這裡不做覆述。而updater.idle()這句是讓程式一直跑。

寫完這部分就可以測試跑。

就在terminal的虛擬環境下輸入以下command：

```python
python cynanchum_bot.py
```

然後程式就會開始跑。注意看看是否正常運行、有沒有error需要debug，出現error需要debug就先debug。然後就可以去Telegram跟自己的chatbot打招呼🙋‍♂️。第一次用chatbot要先加chatbot。當初徐長卿君🤖️的ID是cynanchum\_bot，所以就在搜尋輸入「cynanchum\_bot」，一切正常就會在搜尋結果顯示chatbot。加入chatbot後，就可以第一次開齋了。

輸入/start後，就會出現以下結果：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-16.13.58-1024x429.png)

就這樣，第一個command成功可用了！🤤🤪❤️

由於之前偷懶的關係，把main的dispatcher都預先寫好暫時#，所以之後每寫完一個function部分，就去main把相對應的dispatcher的#去掉，就可以進行測試。

#### 【寫ing - /about】

/about的部分，設計對話跟/start類似，code也是沿用/start的update.message.reply\_text()。這部分的重點在於如何設置inline keyboard。Inline keyboard的好處是限制用戶只能選擇提供的按鈕，而且用途廣泛，除了可以用在Telegram裡面，也能用在提供用戶連結。

```python
def about_handler(update, context: CallbackContext):

    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("感謝使用徐長卿君🤖️。徐長卿君🤖️是一個教學用的中醫搜尋chatbot，除了作為中醫古籍搜尋，另外最大目的是讓想要利用python-telegram-bot的新手有個最新的tutorial可供參考。主人曾經跟我說，在做上一個 - 也是第一個Telegram的chatbot算盤子君🤖️（一個台灣大學資訊的chatbot，有興趣可以去@glochidion_bot找算盤子君🤖️）經歷不太愉快。一方面是documentation和tutorial對於新手蠻unfriendly，而且不少已經開始有點過期，同時也沒有太多關於deploy的文章；另一方面有一次因為想要整理output的code一直不成功去求救卻被說是伸手黨，感覺自己努力過卻被當作白痴，太把自己的知識當作理所當然。氣不過，決定根據自己的專業建立一個中醫古籍搜尋器的Telegram chatbot，也藉此寫一篇tutorial，從建立到測試、測試到deploy，讓新手有個可以參考整個過程、實際應用的demo，同時也讓中醫人有多一個工具。") 
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("徐長卿君🤖️儲存的資料很多，所有資料都是主人親手校對整理，出錯在所難免，因此若想要報錯，又或者有任何疑問、建議，或者想透過徐長卿君🤖️宣傳、洽談合作，可以去扶疏堂研究所的Facebook page私訊聯繫，或者歡迎瀏覽扶疏堂研究所的網站看看其他項目和服務。")
    
    # Inline keyboard的部分
    reply_markup = InlineKeyboardMarkup([[
        InlineKeyboardButton("Facebook", url = "https://www.facebook.com/herboratory/ "),
        InlineKeyboardButton("Website", url = "https://herboratory.ai/")],
        [InlineKeyboardButton("關於徐長卿君🤖️ About Cynanchum kun🤖️", callback_data="about_me")]])

    bot.send_message(update.message.chat.id, "想了解什麼？若要報錯或反饋，歡迎透過Facebook或網站與扶疏堂研究所聯繫。", reply_to_message_id = update.message.message_id,
                     reply_markup = reply_markup)
```

Inline keyboard的格式如下：

```python
reply_markup_1 = InlineKeyboardMarkup([[
        InlineKeyboardButton("在Telegram顯示的項目名稱1號", url = "按鈕按下去想要連結到的網址"),
        InlineKeyboardButton("在Telegram顯示的項目名稱2號", url = "按鈕按下去想要連結到的網址")],
        [InlineKeyboardButton("在Telegram顯示的項目名稱3號", callback_data="設定callback_data的反應字")]])

    bot.send_message(update.message.chat.id, "在inline keyboard彈出來時chatbot在dialogue顯示的回應訊息", reply_to_message_id = update.message.message_id,
                     reply_markup = reply_markup_1)
```

首先，reply\_markup\_1是一個自定義命名，可以不叫reply\_markup\_1，叫reply\_markup也可，但記得改完名稱，在下面的reply\_markup\_1的位置也要寫上對應的名稱，不然inline keyboard無法正常運行。

接下來在InlineKeyboardMarkup裡面有很多\[ \]。在這裡，即使是一個按鈕，也需要用 \[ \]框住，就是這樣：\[InlineKeyboardButton("在Telegram顯示的項目名稱1號", url = "按鈕按下去想要連結到的網址")\]。若有多個，想要另開新一行顯示，就在\[ \]後面加上「,」，然後再開一個\[ \]框。以上的code在Telegram裡面會呈現倒品字形。

內容的話，若要顯示連結🔗，則用InlineKeyboardButton("在Telegram顯示的項目名稱1號", url = "按鈕按下去想要連結到的網址")。若要之後是要在Telegram內讓chatbot根據選項回應，則用InlineKeyboardButton("在Telegram顯示的項目名稱3號", callback\_data="設定callback\_data的反應字")。兩者差異就是一個是用url，一個是callback\_data。callback\_data所輸入的字眼會在之後的getClickButtonData的function中，若detect得到callback\_data裡面的字眼，就會根據設定顯示。而在這個chatbot，callback\_data設定是about\_me。

#### 【寫ing - getClickButtonData】

寫完/about的部分，不能立刻測試，因為inline keyboard還沒設定完成。剛剛說，/about的inline keyboard有個部分會在Telegram的dialogue顯示回應，這裡就是設定回應的部分。

```python
def getClickButtonData(update, context):
    """
    透過上方的about function取得了callback_data="about_me"，針對取得的參數值去判斷說要回覆給使用者什麼訊息
    取得到對應的callback_data後，去判斷說是否有符合，有符合就執行 update.callback_query.edit_message_text
    傳送你想傳送的訊息給使用者
    由於這裡不再是單純發訊息，而是再用callback_query的方法，發訊息時，chat_id = update.message.chat_id是不能用，要改成chat_id = update.callback_query.message.chat_id
    而偽裝輸入正在輸入中也要改成chat_id = update.callback_query.message.chat_id
    """
    
    if update.callback_query.data == "about_me":
        bot.send_chat_action(chat_id = update.callback_query.message.chat_id, action = telegram.ChatAction.TYPING)
        time.sleep(1)
        update.callback_query.edit_message_text("感謝使用徐長卿君🤖️。徐長卿君🤖️是一個教學用的中醫搜尋chatbot，除了作為中醫古籍搜尋，另外最大目的是讓想要利用python-telegram-bot的新手有個最新的tutorial可供參考。主人曾經跟我說，在做上一個 - 也是第一個Telegram的chatbot算盤子君🤖️（一個台灣大學資訊的chatbot，有興趣可以去@glochidion_bot找算盤子君🤖️）經歷不太愉快。一方面是documentation和tutorial對於新手蠻unfriendly，而且不少已經開始有點過期，同時也沒有太多關於deploy的文章；另一方面有一次因為想要整理output的code一直不成功去求救卻被說是伸手黨，感覺自己努力過卻被當作白痴，太把自己的知識當作理所當然。氣不過，決定根據自己的專業建立一個中醫古籍搜尋器的Telegram chatbot，也藉此寫一篇tutorial，從建立到測試、測試到deploy，讓新手有個可以參考整個過程、實際應用的demo，同時也讓中醫人有多一個工具。\n徐長卿君🤖️儲存的資料很多，所有資料都是主人親手校對整理，出錯在所難免，因此若想要報錯，又或者有任何疑問、建議，或者想透過徐長卿君🤖️宣傳、洽談合作，可以去扶疏堂研究所的Facebook page私訊聯繫，或者歡迎瀏覽扶疏堂研究所的網站看看其他項目和服務。")
```

如前述，之前callback\_data是設成about\_me，因此這個function最大的目的就是，若detect得到回傳“about\_me”，也就是「if update.callback\_query.data == "about\_me":」這句，就跑if下面的的東西。

注意的是，在這裡若要顯示訊息，chatbot的回應要改成update.callback\_query.edit\_message\_text，偽裝輸入正在輸入中也要改成update.callback\_query.message.chat\_id，不然會出現error。這是由於這裡是使用callback\_query方法，並非以message的方法傳送訊息。這裡之前讓我撞牆到懷疑人生，就是不知道要把message改成callback\_query，因此一直出現error。

這部分設定好，記得在main()裡對應的about\_handler和getClickButtonData的dispatcher把#去掉，就可以測試。

去虛擬環境再跑python 檔案，然後在Telegram輸入/about，會顯示這個樣子：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-16.57.12-971x1024.png)

按下「關於徐長卿君🤖️ About Cynanchum kun🤖️」，若getClickButtonData設定正確，會顯示一下畫面：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-16.59.16-1024x722.png)

如是者，/about的部分就做好了！😆😆😆

#### 【寫ing - /help】

寫/help的部分，程式跟/start非常類似。這裡注意的點就是設計對話方面，給user的指示要清晰簡單，不要拐彎抹角。

```python
def help_handler(update, context: CallbackContext):

    # chatbot在接受用戶輸入/start後的output內容
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("《🔍如何使用》\n搜尋《黃帝內經素問》（中文版）🔸\n輸入：「/suwen （搜尋關鍵字）」\n例如：/suwen 伏梁、/suwen 肺脈\n\n💬關於徐長卿君🤖️，或想要報錯和反饋💭，請輸入 /about") 
```

同樣的，寫完去main()把help對應的help\_handler的dispatcher的#去掉，再把程式跑起來後去Telegram測試，成功的話就會出現以下畫面：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-17.06.29-1024x415.png)

就這樣，/help command的部分就大功告成！🤪🤪🤪

#### 【寫ing - 《素問》搜尋指令/suwen】

寫這部分最大的挑戰是要把user輸入的關鍵字與command /suwen分開來。這部分寫法是源自[這裡](https://markteaching.com/telegram-bot-records/)。基本上搜尋command格式也是類似，就是/suwen 搜尋關鍵字。所以首先要先判斷輸入的搜尋command是否有搜尋關鍵字。而「/suwen 」（連空格）共7個字元，因此可以用if去判斷，若輸入字元小於等於7，也就是代表只有「/suwen 」時，就會告訴user輸入錯誤。當然有人會問，若字元小於7，已經不是/suwen的command，因此在接下來後面還會做雙重保險，若是輸入非command字眼，就會有回覆告訴user。

當判斷輸入的內容大於7個字元，就代表後面是有東西，然後就擷取keyword的部分，並確定是string，就開始搜尋並顯示結果；若不是string，就會回傳輸入方式有錯誤。這篇的重點主要在如何建立Telegram chatbot，因此搜尋的寫法不在這裡講解。

有個地方要注意，在Telegram，一個訊息是有字數限制，不能超過4096個字。所以在下面程式，結果顯示超過10項會分開message顯示。當然不代表顯示10筆就沒事，而是這是我的資料經過測試後比較理想的顯示方式。因此若打算顯示長篇大論的話就要注意字數限制。

```python
def suwen_handler(update, context: CallbackContext):
    if len(update.message.text) <= 7:  # 如果單純輸入 /suwen 會跟你說叫你該如何輸入，以及因為/suwen 的字元數量一定會小於7
        update.message.reply_text("你輸入方式有錯誤。\n請輸入：/suwen 搜尋關鍵字 \n 例如: /suwen 伏梁")
    else:
        keyword = update.message.text[7:] # 取得到 /suwen 後面的參數值
        if type(keyword) == str:
            # 搜尋keywords資料
            with open("data/suwen.json") as json_file: # 打開suwen.json
                data = json.load(json_file)

            search_result = []
            for items in data["items"]:
                article = items['article']
                for quotes in items["content"]:
                    if keyword in quotes:
                        search_result.append({"原句": quotes, "出處": article})

            if not search_result:
                bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
                time.sleep(1)
                update.message.reply_text("查無結果。🤔😐😔")

            else:
                text = "搜尋結果：\n"
                for i, quote_data in enumerate(search_result):
                    for key in quote_data:
                        text += "\n" + "{}：{}".format(key, quote_data[key])
                    text += "\n------"
                    if i % 10 == 9 or i == len(search_result) - 1:
                        bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
                        time.sleep(1)
                        update.message.reply_text(text)
                        text = "搜尋結果（續）：\n"

        else:
            # 輸入錯誤格式或有誤時，請重新輸入
            update.message.reply_text("你輸入方式有錯誤。\n請輸入：/suwen 搜尋關鍵字 \n 例如: /suwen 伏梁")
```

若搜尋部分沒問題，輸入/suwen 伏梁會出現以下結果：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-17.24.53-1024x985.png)

若能正確顯示搜尋結果，就代表/suwen command的部分大功告成！🙃

#### 【寫ing - 防止chatbot被玩壞的reply\_handler】

chatbot寫到這裡，主要功能部分就算是完成了。接下來要採取預防措施防止chatbot蛾子被user玩壞。

預防措施可以分成兩個部分，一個是接下來要做的reply\_handler，一個是這個之後的error\_handler和error的部分。reply\_handler基本上最大預防被user玩壞的關卡 - 因為預設只要user不是輸入正確的command，不管是否故意，反正就直接會顯示類似「挖垮薄」的訊息提醒user。因此邏輯很簡單，只要不是以上的command，就跑這一部分。

```python
def reply_handler(update, context: CallbackContext):
    """Reply message."""
    text = update.message.text
    if (text != "/start") or (text != "/help") or (text != "/about") or (text != "/suwen"):
        bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
        time.sleep(1)
        update.message.reply_text("對不起，徐長卿君🤖不能理解你說啥。🤔\n\n關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭的聯繫方式，請輸入 /about")
```

若一切正確，測試時亂打一通，就會顯示以下結果：

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-17.38.13-1024x361.png)

#### 【寫ing - error\_handler & error】

error\_handler和error的部分，最大的目的是防止不可預知的down機情形。算是有點類似有黑盒子的汽車開在懸崖峭壁失控，只能拖著懸崖靠摩擦力減速，過程有可能導致chatbot down機也有可能不會，但至少讓user知道chatbot有點問題，也給開發者留下尾巴知道發生啥事。一般來說，經過粗魯測試後沒被玩壞或儲存地方沒啥意外的話理論上不會出現啥問題，但以防萬一，還是先做好準備。

error\_handler()的部分很簡單，就是有error時有個feedback給user。在function的括號要加入bot和error，才會知道是針對error而設。

```python
def error_handler(bot, update, error, context: CallbackContext):
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("對不起，徐長卿君🤖不能理解你說啥。🤔\n\n關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭的聯繫方式，請輸入 /about")
```

error()部分，就是在console顯示log而已。這部分針對尤其是早期掛本機時公開測試時，若中間有啥error時可以滾回去log看error是啥。若是已經deploy在雲端的話作用不大。當然可以修改下面的code另外儲存log，那就可以隨時可以翻看。

```python
def error(update, context):
    """紀錄Updates時出現的errors。出現error時console就會print出下面logger.warning的內容"""
    logger.warning('Update "%s" caused error "%s"', update, context.error)
```

#### 【整個chatbot的code】

整個chatbot寫完後，程式碼如下：

```python
import time
import json
import os
import os.path
import logging
import telegram
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters, CallbackContext, ConversationHandler
from telegram import InlineKeyboardMarkup, InlineKeyboardButton

# 允許 logging。當出現error時能知道哪裡出了問題。
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)

logger = logging.getLogger(__name__)

# 創造Updater，將token pass給Updater。
# 在v12中要加入“use_context = True”（之後版本不需要），用於有新訊息是回應。
# TOKEN、bot和updater要放在def外面。若只放在main()會出現錯誤
TOKEN = "paste-your-token-here"
bot = telegram.Bot(token = TOKEN)
updater = Updater(TOKEN, use_context = True)

def start_handler(update, context: CallbackContext):

    # chatbot在接受用戶輸入/start後的output內容
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING) # 會顯示chatbot正在輸入中，增加對話真實感
    time.sleep(1) # 在顯示輸入中後停頓1秒，然後顯示下一句code的文字
    update.message.reply_text("Hello! 你好👋，{}！我是徐長卿君🤖".format(update.message.from_user.first_name)) # 給user的output
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("徐長卿君🤖️是個chatbot，能根據關鍵字搜尋《黃帝內經素問》的內容\n\n❓關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭，請輸入 /about") # 給user的output。output可以分開多次使用update.message.reply_text()。
    
def help_handler(update, context: CallbackContext):

    # chatbot在接受用戶輸入/start後的output內容
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("《🔍如何使用》\n搜尋《黃帝內經素問》（中文版）🔸\n輸入：「/suwen （搜尋關鍵字）」\n例如：/suwen 伏梁、/suwen 肺脈\n\n💬關於徐長卿君🤖️，或想要報錯和反饋💭，請輸入 /about") 
    
def about_handler(update, context: CallbackContext):

    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("感謝使用徐長卿君🤖️。徐長卿君🤖️是一個教學用的中醫搜尋chatbot，除了作為中醫古籍搜尋，另外最大目的是讓想要利用python-telegram-bot的新手有個最新的tutorial可供參考。主人曾經跟我說，在做上一個 - 也是第一個Telegram的chatbot算盤子君🤖️（一個台灣大學資訊的chatbot，有興趣可以去@glochidion_bot找算盤子君🤖️）經歷不太愉快。一方面是documentation和tutorial對於新手蠻unfriendly，而且不少已經開始有點過期，同時也沒有太多關於deploy的文章；另一方面有一次因為想要整理output的code一直不成功去求救卻被說是伸手黨，感覺自己努力過卻被當作白痴，太把自己的知識當作理所當然。氣不過，決定根據自己的專業建立一個中醫古籍搜尋器的Telegram chatbot，也藉此寫一篇tutorial，從建立到測試、測試到deploy，讓新手有個可以參考整個過程、實際應用的demo，同時也讓中醫人有多一個工具。") 
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("徐長卿君🤖️儲存的資料很多，所有資料都是主人親手校對整理，出錯在所難免，因此若想要報錯，又或者有任何疑問、建議，或者想透過徐長卿君🤖️宣傳、洽談合作，可以去扶疏堂研究所的Facebook page私訊聯繫，或者歡迎瀏覽扶疏堂研究所的網站看看其他項目和服務。")
    
    reply_markup = InlineKeyboardMarkup([[
        InlineKeyboardButton("Facebook", url = "https://www.facebook.com/herboratory/ "),
        InlineKeyboardButton("Website", url = "https://herboratory.ai/")],
        [InlineKeyboardButton("關於徐長卿君🤖️ About Cynanchum kun🤖️", callback_data="about_me")]])

    bot.send_message(update.message.chat.id, "想了解什麼？若要報錯或反饋，歡迎透過Facebook或網站與扶疏堂研究所聯繫。", reply_to_message_id = update.message.message_id,
                     reply_markup = reply_markup)
    # chatbot在接受用戶輸入/start後的output內容

def suwen_handler(update, context: CallbackContext):
    if len(update.message.text) <= 7:  # 如果單純輸入 /suwen 會跟你說叫你該如何輸入，以及因為/suwen 的字元數量一定會小於7
        update.message.reply_text("你輸入方式有錯誤。\n請輸入：/suwen 搜尋關鍵字 \n 例如: /suwen 伏梁")
    else:
        keyword = update.message.text[7:] # 取得到 /location 後面的參數值
        if type(keyword) == str:
            # 搜尋keywords資料
            with open("data/suwen.json") as json_file: # 打開suwen.json
                data = json.load(json_file)

            search_result = []
            for items in data["items"]:
                article = items['article']
                for quotes in items["content"]:
                    if keyword in quotes:
                        search_result.append({"原句": quotes, "出處": article})

            if not search_result:
                bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
                time.sleep(1)
                update.message.reply_text("查無結果。🤔😐😔")

            else:
                text = "搜尋結果：\n"
                for i, quote_data in enumerate(search_result):
                    for key in quote_data:
                        text += "\n" + "{}：{}".format(key, quote_data[key])
                    text += "\n------"
                    if i % 10 == 9 or i == len(search_result) - 1:
                        bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
                        time.sleep(1)
                        update.message.reply_text(text)
                        text = "搜尋結果（續）：\n"

        else:
            # 輸入錯誤格式或有誤時，請重新輸入
            update.message.reply_text("你輸入方式有錯誤。\n請輸入：/suwen 搜尋關鍵字 \n 例如: /suwen 伏梁")

def getClickButtonData(update, context):
    """
    透過上方的about function取得了callback_data="about_me"，針對取得的參數值去判斷說要回覆給使用者什麼訊息
    取得到對應的callback_data後，去判斷說是否有符合，有符合就執行 update.callback_query.edit_message_text
    傳送你想傳送的訊息給使用者
    由於這裡不再是單純發訊息，而是再用callback_query的方法，發訊息時，chat_id = update.message.chat_id是不能用，要改成chat_id = update.callback_query.message.chat_id
    而
    而偽裝輸入正在輸入中也要改成chat_id = update.callback_query.message.chat_id
    """
    
    if update.callback_query.data == "about_me":
        bot.send_chat_action(chat_id = update.callback_query.message.chat_id, action = telegram.ChatAction.TYPING)
        time.sleep(1)
        update.callback_query.edit_message_text("感謝使用徐長卿君🤖️。徐長卿君🤖️是一個教學用的中醫搜尋chatbot，除了作為中醫古籍搜尋，另外最大目的是讓想要利用python-telegram-bot的新手有個最新的tutorial可供參考。主人曾經跟我說，在做上一個 - 也是第一個Telegram的chatbot算盤子君🤖️（一個台灣大學資訊的chatbot，有興趣可以去@glochidion_bot找算盤子君🤖️）經歷不太愉快。一方面是documentation和tutorial對於新手蠻unfriendly，而且不少已經開始有點過期，同時也沒有太多關於deploy的文章；另一方面有一次因為想要整理output的code一直不成功去求救卻被說是伸手黨，感覺自己努力過卻被當作白痴，太把自己的知識當作理所當然。氣不過，決定根據自己的專業建立一個中醫古籍搜尋器的Telegram chatbot，也藉此寫一篇tutorial，從建立到測試、測試到deploy，讓新手有個可以參考整個過程、實際應用的demo，同時也讓中醫人有多一個工具。\n徐長卿君🤖️儲存的資料很多，所有資料都是主人親手校對整理，出錯在所難免，因此若想要報錯，又或者有任何疑問、建議，或者想透過徐長卿君🤖️宣傳、洽談合作，可以去扶疏堂研究所的Facebook page私訊聯繫，或者歡迎瀏覽扶疏堂研究所的網站看看其他項目和服務。")

def reply_handler(update, context: CallbackContext):
    """Reply message."""
    text = update.message.text
    if (text != "/start") or (text != "/help") or (text != "/about") or (text != "/suwen"):
        bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
        time.sleep(1)
        update.message.reply_text("對不起，徐長卿君🤖不能理解你說啥。🤔\n\n關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭的聯繫方式，請輸入 /about")

def error_handler(bot, update, error, context: CallbackContext):
    bot.send_chat_action(chat_id = update.message.chat_id, action = telegram.ChatAction.TYPING)
    time.sleep(1)
    update.message.reply_text("對不起，徐長卿君🤖不能理解你說啥。🤔\n\n關於指令使用方法，請輸入 /help \n💬關於徐長卿君🤖️，或想要報錯和反饋💭的聯繫方式，請輸入 /about")

def error(update, context):
    """紀錄Updates時出現的errors。出現error時console就會print出下面logger.warning的內容"""
    logger.warning('Update "%s" caused error "%s"', update, context.error)

def main():
    """啟動bot"""
    # 設定使用dispatcher，用來以後設定command和回覆用
    dp = updater.dispatcher
    dp.add_handler(CommandHandler("start", start_handler)) # 啟動chatbot
    dp.add_handler(CommandHandler("help", help_handler)) # 顯示幫助的command
    dp.add_handler(CommandHandler("about", about_handler)) # 顯示關於徐長卿君🤖️的command
    dp.add_handler(CommandHandler("suwen", suwen_handler)) # 《素問》中文版搜尋功能
    dp.add_handler(CallbackQueryHandler(getClickButtonData)) # 設定關於徐長卿君🤖️的按鈕連結
    dp.add_handler(MessageHandler(Filters.text, reply_handler)) # 設定若非設定command會回覆用戶不知道說啥的訊息
    dp.add_error_handler(error_handler) # 出現任何非以上能預設的error時會回覆用戶的訊息內容

    # 專門紀錄所有errors的handler，對應def error()
    dp.add_error_handler(error)

    # 啟動Bot。bot程式與Telegram連結有兩種方式：polling和webhook。
    # 兩者的差異可以參考這篇reddit的解釋：https://www.reddit.com/r/TelegramBots/comments/525s40/q_polling_vs_webhook/。
    # 在python-telegram-bot裡面本身有built-in的webhook方法，但是在GCE中暫時還沒摸索到如何設定webhook，因此polling是最便捷的方法。
    updater.start_polling()

    # 就是讓程式一直跑。
    # 按照package的說法“start_polling() is non-blocking and will stop the bot gracefully.”。
    # 若要停止按Ctrl-C 就好
    updater.idle()

#運行main()，就會啟動bot。
if __name__ == '__main__':
    main()
```

### 【GCP上☁️deploy】

到這裡，估計chatbot應該是本機運行過、本機也粗魯玩過。要上雲端，若沒有附件（例如不會如這個chatbot需要連帶個JSON檔案的話）或者把附件安置好在某個☁️地方，或者本身連database連結去用的話，可以用Google Function。這裡會用一個「切菜焉用牛刀」的方法，就是在Google Cloud Platform（GCP）開一個虛擬機器（virtual machine），然後把folder的東西整齊放到虛擬機器運行就行。

這裡假設已經有GCP帳戶。沒有的話，先弄個GCP的帳戶，新用戶可以有12個月或300鎂額度，詳細可看[這裡](https://cloud.google.com/free/docs/gcp-free-tier?hl=zh-TW)介紹。有GCP帳戶後，登陸到GCP，然後開個新project。輸入project name，然後按CREATE。然後等GCP運作一下，把東西搭建好。就去開虛擬機器。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-18.20.29-1024x666.png)

然後進入project，按下左上角三條線，就會出現menu如下。在Compute Machine選VM instances。等一切setting完成後，

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-18.28.24-646x1024.png)

等settle down後，在VM instances按Create。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-18.32.29-1024x539.png)

進去後，會設定虛擬機器的配備。在這裡可以幫機器命名。Region的話一般選擇asia-east1 (Taiwan)，Zone選擇asia-east1-b。現在用香港的話還是謝謝不送了。不過之前無論是在GCP setup網站還是安置算盤子君🤖️，都是選Taiwan。Machine configuration的話，由於不期望流量很大，所以Series選第一代的N1就好，machine type就選個就算有點流量也沒啥大問題的g1-small (1 vPCU, 1.7 GB memory)。若只是玩玩而已，可以選更小的mirco。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-19.57.55-868x1024.png)

Boot disk的話改成Debian 10，Identity and API access不需要更動。Firewall勾選Allow HTTP traffic和Allow HTTPS traffic。然後按Create。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-19.59.44-773x1024.png)

等機器設定好之後，就會進入以下畫面。機器會配備個Internal IP和External IP（這裡我遮蔽了）。接下來就是利用SSH把該upload的東西給upload上去。

在Connect下面有個SSH，按下隔壁的箭頭，會有幾個選項，就選第一個「Open in browser window」。之後會有個popup window跳出來，沒有的話看看browser有沒有阻擋popup設定。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-18.42.03-1024x540.png)

進去之後，首先建立個folder，我就命名cynanchum\_bot，這個folder之後就是儲存chatbot的地方。command如下：

```
mkdir cynanchum_bot
```

然後進入folder：

```
cd cynanchum_bot
```

先更新虛擬機器的東西：

```
apt-get update
```

檢查一下Python版本：

```
python --version
```

若Python版本不是3+，或顯示“-bash: python: command not found”，就先看看有沒有內置Python 3:

```
ls /usr/bin/python*
```

沒有的話，就根據以下command安裝Python3。有的話這步就skip。

```
sudo apt install python3 python3-dev python3-venv
```

然後更改Python版本到3.x（這裡我用3）：

```
alias python='/usr/bin/python3'
```

然後刷一下.bashrc檔：

```
. ~/.bashrc
```

然後再檢查Python版本，應該會改成3.5（或自行設定的版本）

```
python --version
```

版本改好之後，或者本身有Python3，先安裝python3-venv：

```
sudo apt-get install python3-venv
```

然後就可以建立Python的虛擬環境。由於本來已經在要放置徐長卿君🤖的cynanchum\_bot.py和資料的資料夾，所以就直接現成設虛擬環境的名稱就好。這裡名稱為environment：

```
python3 -m venv environment
```

輸入“ls”，就會看到裡面有個environment的folder。

接下來就是把上傳的東西上傳。我懶得設啥，東西也不多，就每個拉上去就算了。在右上角會看到一個⚙️ icon，按下去，就會有個menu。選Upload file。就可以把檔案upload上去。

![](https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/Screenshot-2020-07-25-at-19.11.12.png)

然後輸入“cd”按Enter，輸入”ls”確定cynanchum\_bot.py成功upload。然後利用以下command把主程式搬到folder裡：

```
mv cynanchum_bot.py cynanchun_bot
```

這裡要說一件滿無聊兼白痴的事，用Upload file，是會把檔案上傳到最頂端的位置，因此要搬運的話可以輸入"cd"然後按Enter，然後輸入以下command把搜尋用的JSON檔搬去data的folder裡（由於正式版本的徐長卿君機器人除了素問資料，還有其他的資料；而除了cynanchum\_bot.py之外都是JSON檔，所以可以用這個方案）：

```
mv *.json ~/cynanchum_bot/data
```

然後就可以利用“cd”回去最上層，然後“cd cynanchum\_bot”去儲存的folder位置，進入虛擬環境跑chatbot：

```
source environment/bin/activate
```

進去虛擬環境後，別太興奮直接跑程式，因為還要安裝python-telegram-bot到GCP的虛擬機器：

```
pip3 install wheel python-telegram-bot
```

安裝完，就可以試跑了：

```
python3 cynanchum_bot.py
```

成功的話，就跟之前本機測試一樣，在Telegram輸入東西後，chatbot會有回應。可是若把SSH關掉，程式就會停止運作。所以若要讓程式即使在SSH關掉後，依然能在背景繼續運作，就需要輸入以下command：

```
nohup python cynanchum_bot.py &
```

然後就能在關掉SSH連線後還能在背景繼續運作。

### 【後記】

花了快十二小時，終於寫完這個tutorial。希望這篇tutorial能幫助想要寫chatbot的非相關專業背景同好有個比較全面的起始點。作為chatbot平台，Telegram的免費的確是個賣點，而且從資安角度整體算是暫時相對比較安全（當然要追求安全，要不就用Signal，要不完全不用）。Telegram除了作為IM和建立chatbot的平台，還能建立群組、channel等，功能上也蠻多樣化。

能完成算盤子君🤖️和徐長卿君🤖️，很感謝在Stackoverflow和Python Taiwan的幫助過我的大神，也很感謝中間協助過我的朋友們。

這個tutorial完成，我也放下心頭大石，暫時不會亂手滑寫程式。
