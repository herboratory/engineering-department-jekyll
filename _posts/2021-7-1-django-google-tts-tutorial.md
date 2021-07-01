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
