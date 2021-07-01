---
layout: post
title: "利用Python建立text-to-speech的browser web app"
date: 2021-7-1
excerpt: "以django與Google text-to-speech (TTS)服務建立的project"
tags: [tutorial, python, tts, django]
feature: https://raw.githubusercontent.com/herboratory/engineering-department/master/assets/img/993C7260-F9A8-4B1D-B86A-11C737F39862.jpeg
comments: true
---
## 【這個程式的緣起】
之前父親節，很多人都會費盡心思準備父親節禮物，當然我也不例外。可是我是窮L，也不知道該買啥 - 老父是個學習宅，以前連我老母也是個宅 - 我肯定是他們親生的，因為我也是個宅。碰巧最近老父又迷上學習語言，而之前我因為上剪片workshop要繳交作業，為了不出鏡不出聲，於是用Python寫了個利用Google的text-to-speech（TTS）服務的程式把講稿generate成MP3，我爸就對這個程式很有興趣。但是因為是一堆code，估計也有點嚇怕了他。於是趁著父親節，就把這個程式加工，用django搭建個web app，讓老爸可以按一按就能有個MP3檔案。雖然之前跟著Mozilla的tutorial做了個個人圖書管理程式，但對django還是不算熟悉，因此建立這個TTS app也經歷不少鬼打牆的時光。所以一邊做一邊記錄在案，下面的內容就是做的時候的筆記。

不囉嗦！開動唄！

## 【環境】
OS：MacOS Big Sur 11.0.1
環境：Python 3.7

## 【部署虛擬環境】
project獨立開個虛擬環境來做，方便管理雜七雜八的packages。首先先自己找個自己爽的地方弄個folder將之後的東西集中處理，然後在Terminal輸入以下的code：

```
python3 -m venv playground
source playground/bin/activate
pip install --upgrade pip
```

## 【安裝需要的packages】
接下來安裝需要的packages
```Python
pip install django
pip install --upgrade google-cloud-texttospeech # Google的text-to-speech服務的Python package
```

## 【建立project】
安裝需要的package後，就開始利用django建立project：
```Python
django-admin startproject tts_converter
```
## 【setting.py調整】
打開tts_converter/tts_converter/settings.py。

### 1. 配置資料庫
Django default用SQLite。在這個項目裡，基本上不太會需要用到很多存取行為，頂多就是有session，因此我們也使用SQLite。所以在這裏不需要額外的配置工作。若想要用其他的database，可參考[這裏](https://docs.djangoproject.com/en/3.2/intro/tutorial02/)
```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

### 2. 其他設置
```Python
LANGUAGE_CODE = 'en-gb'  # 語言
TIME_ZONE = 'Asia/Hong_Kong'  # 時區

### 3. 在Terminal輸入下面command測試是否成功建立：
```Python
python3 manage.py runserver
```

## 【建立app】
測試後若能順利顯示，就在project裡建立app。在Terminal輸入以下指令：
```Python
python3 manage.py startapp converter
```

## 【調試app】
### 【app基礎設定】
打開converter/view.py然後加入以下的code：
```Python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```
然後在converter的資料夾建立urls.py，並加入以下的code：
```Python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('converter/', include('converter.urls')), # 這一行用來增加連結"converter"
    path('admin/', admin.site.urls),
]
```
回去Terminal測試app是否成功建立、converter的連結是否成功建立：
```Python
python3 manage.py runserver
```
然後在瀏覽器輸入http://127.0.0.1:8000/converter ，若成功的話會看到“Hello, world.” - 這是之前在converter/view.py輸入的內容。

### 【migrate去建立需要的資料庫表格】
migrate的指令會看INSTALLED_APPS並且根據setting.py建立所有需要的資料庫表格。在Terminal輸入下面指令：
```Python
python manage.py migrate
```

### 【建立與啟動模型Models】
打開models.py，定義會儲存的資料。轉換器輸入有欲轉換的文字和語言，因此這裡有兩個項目：
```Python
class InputContent(models.Model):
    text_content = models.TextField()
    language = models.CharField(max_length=200)
```
然後去setting.py，裡面找INSTALLED_APPS的section，會看到這些：
```Python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

將這個部分修改成這樣：

```Python
INSTALLED_APPS = [
    'converter.apps.ConvertersConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
```converter.apps.ConvertersConfig```裡面的```ConverterConfig```的Converter必須大寫。若是用其他名字作為app的名稱，格式就是```appname.apps.AppnameConfig```。

將models.py儲存好，在terminal輸入套用migrations：

```Python
python manage.py makemigrations converter
python manage.py sqlmigrate converter 0001
python manage.py migrate
```

## 【將HTML頁面與django連結】
首先需要有個index.html。
如前述，用戶輸入的資料有兩個項目：欲轉換的文字和語言，因此頁面也是有對應的輸入欄：

```
<html>
    <head>
      <title>Text-to-Speech Converter｜文字-語音轉換器</title>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <link href='https://fonts.googleapis.com/css?family=Nunito:400,300' rel='stylesheet' type='text/css'>
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
      {% load static %}
      <link rel="stylesheet" href="{% static 'css/styles.css' %}">
    </head>
    <body>
      <div class="row">
        <div class="col-md-12">
          <form action="" method="POST">
            {% csrf_token %}
            <h2> Text-to-Speech Converter<br>文字-語音轉換器</h2>
            <fieldset>
              <label for="text-label-text-content">Text Content｜文字內容：</label>
              <textarea id="text_content" name="text_content"></textarea>
              <label for="language_select">Language｜語言：</label>
              <select id="language" name="language">
                <optgroup label="Select Language｜選擇語言">
                  <option value="cantonese">Cantonese｜廣東話</option>
                  <option value="mandarin">Mandarin｜國語</option>
                  <option value="putonghua">Putonghua｜普通話</option>
                  <option value="english">English｜英文</option>
                  <option value="french">French｜法文</option>
                  <option value="japanese">Japanese｜日文</option>
                  <option value="vietnamese">Vietnamese｜越南文</option>
                </optgroup>
              </select>
            </fieldset>
            <p>Click "Transform｜轉換" to transform and download the audio.</p>
            <p>按「Transform｜轉換」轉換並下載語音檔。</p>
            <center><button type="submit">Transform｜轉換</button></center>
          </form>
        </div>
      </div>
    </body>
</html>
```

若要美化可以做個CSS檔。而上面的的HTML code已經有加入CSS的部分，位置在```<head>...</head>```的最後兩行。CSS就懶得在這裡po，大家可以弄個CSS自行美化一下。

這個app建立了index.html和styles.css，要注意這兩個檔案放置的位置。HTML檔的部分，在converter建立一個名為「templates」的資料夾，並將index.html放進去。CSS檔的部分，則在converter建立一個名為「static」的資料夾，裡面再建立一個名為「css」的資料夾，然後把styles.css放進去。若只有CSS的話，建立「css」資料夾就不是必要；但若網站有用到javascript，為了方便管理，各自開個資料夾存放還是有必要。

接下來打開converter/view.py，會看到之前輸入的code：
```Python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world.")
```

現在要把view.py這邊的index與index.html連結，因此return的部分需要修改成如下：
```Python
def index(request):
    return render(request, 'index.html')
```

儲存好view.py，然後打開converter/urls.py，確定code是否長成這個樣子：
```Python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

在view.py裡，def能否不用index？可以的。只要在app的urls.py裡，```urlpatterns```的```views.index```的「index」跟view.py的def名稱一樣就可以。

但是呢，我們還需要把form裡面的文字和選擇語言load進來，並且存到session，因此view.py的程式加入並改成這樣：
```Python
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.

def index(request):
    text_content = request.POST.get('text_content')
    language = request.POST.get('language')
    request.session['current_content'] = text_content
    request.session['current_language'] = language
    print(request.POST.get)  #測試是否有get到內容
    print(request.POST.get('text_content'))  #測試是否有get到text_content的內容
    print(request.POST.get('language'))  #測試是否有get到language的內容
    print("the content in session is: " + str(request.session.get('current_content')))  #測試是否有get到session版本的text_content的內容
    if text_content:
        print("it works.")  #測試是否有運作
        print("the text content is " + text_content + ", and the language is " + language + ".")
    return render(request, 'index.html')
```

若在terminal有出現自己在browser填寫測試用的內容，就代表正常運作了。

下一步，就是要把文字轉換成語音的部分搗鼓出來。程式部分基本上參考[Google Cloud Text-to-Speech AI API in Python - Getting Started (Part 1)](https://www.youtube.com/watch?v=N2F7VgDn3BQ)和[Google Cloud Text-to-Speech AI API in Python - Creating a Python Program (Part 2)](https://www.youtube.com/watch?v=ZXnPMzmrmIY)，用Google的text-to-speech服務處理。如何申請服務也請參考影片內容，這裡就不在複述。

根據之前在view.py檔案內的code改成這個樣子（code的解釋在裡面）：
```Python
# import django所需要的東西
from django.shortcuts import render
from django.http import HttpResponse

# import text-to-speech所需要的東西
import os
from google.cloud import texttospeech
from google.cloud import texttospeech_v1
from google.cloud.texttospeech_v1.types.cloud_tts import SsmlVoiceGender
import tempfile
import uuid

# Create your views here.

def index(request):
    # 用POST的方法在網頁取得文字內容和語言選擇的資料
    text_content = request.POST.get('text_content')
    language = request.POST.get('language')
    request.session['current_content'] = text_content
    request.session['current_language'] = language

    # 測試是否透過POST獲得文字內容和語言選項並存到session
    print(request.POST.get)  #測試是否有get到內容
    print(request.POST.get('text_content'))  #測試是否有get到text_content的內容
    print(request.POST.get('language'))  #測試是否有get到language的內容
    print("the content in session is: " + str(request.session.get('current_content')))  #測試是否有get到session版本的text_content的內容
    if text_content: # 若文字內容有東西的話就跑下面的東西
        print("it works.")  #測試是否有文字內容的部分是否有運作
        print("the text content is " + text_content + ", and the language is " + language + ".")

        # 下面開始就是文字轉成語音的code的部分
        os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "herboratory-tiny-applications-03173a1733bf.json" # load入G大神的crediential，這個請根據參考影片那邊申請獲得

        client = texttospeech_v1.TextToSpeechClient() #call text-to-speech (TTS)的服務

        #把可選擇的語言將對應Google的語言碼和聲音碼做成dictionary
        language_code_list = {'cantonese': 'yue-HK', 'english': 'en-GB', 'french': 'fr-FR', 'japanese': 'ja-JP', 'mandarin': 'cmn-TW', 'putonghua': 'cmn-CN', 'vietnamese': 'vi-VN'}
        voice_list = {'cantonese': 'yue-HK-Standard-B', 'english': 'en-GB-Wavenet-B', 'french': 'fr-FR-Wavenet-B', 'japanese': 'ja-JP-Wavenet-B', 'mandarin': 'cmn-TW-Wavenet-B', 'putonghua': 'cmn-CN-Wavenet-B', 'vietnamese': 'vi-VN-Wavenet-B'}

        synthesis_input = texttospeech_v1.SynthesisInput(text = text_content) # 合成輸入文字
        
        # load入選擇的語言，並把語言碼和聲音碼的資料給TTS服務設定聲音
        voice = texttospeech_v1.VoiceSelectionParams(
            name = voice_list[language],
            language_code = language_code_list[language]
        )

        # 把聲音設成MP3
        audio_config = texttospeech_v1.AudioConfig(
            audio_encoding = texttospeech_v1.AudioEncoding.MP3
        )

        # 合成檔案
        response = client.synthesize_speech(
            input = synthesis_input,
            voice = voice,
            audio_config = audio_config
        )
        
        # export成MP3到“output”的folder，並且MP3檔名用uuid命名
        export_name = "output/{name}.mp3".format(name = str(uuid.uuid4()))
        with open(export_name, "wb") as output:
            output.write(response.audio_content)

    return render(request, 'index.html')
```

到此，程式的部分基本上算是完成了。由於這個web app是會在本機跑，不deploy到cloud，為了方便起見，還是弄個shell檔案把前期基本的東西先處理好。

## 【結語】
以上為利用django與Google的text-to-speech service建立web app的做法。過去看了很多django的教學，一人一種做法。的確，用django建立web app是有個框架，但並沒有單一固定的做法，即時同個問題也有不同的實現方法，因此可能大家入門時會感覺比較困惑。這篇tutorial可能對於完全零概念來說不容易懂，裡面有很多瑣碎的點。但算是提供了一個相對簡單且具有應用功能的建立框架給大家參考。
