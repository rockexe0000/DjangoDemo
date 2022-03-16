# 用 Django REST Framework 撰寫 RESTful API 並生成 Swagger 文檔（上） — 用Django REST Framework 撰寫 RESTful API

> 簡介 Swagger、Django REST Framework 以及 drf-yasg

簡介 Swagger、Django REST Framework 以及 drf-yasg

適用讀者：對 Django 有基本認識想要用 Django REST Framework 撰寫 RESTful API 並生成 Swagger 文檔。如果沒有聽過 Django，建議先閱讀[官方文檔](https://www.djangoproject.com/)。

本系列分成上下兩篇，本篇介紹 Swagger 以及如何用 Django REST Framework 撰寫 RESTful API，下篇則介紹[如何用 drf\_yasg 生成 Swagger 文檔。](https://medium.com/@zoejoyuliao/%E7%94%A8-django-rest-framework-%E6%92%B0%E5%AF%AB-restful-api-%E4%B8%A6%E7%94%9F%E6%88%90-swagger-%E6%96%87%E6%AA%94-%E4%B8%8B-%E7%94%9F%E6%88%90-swagger-%E6%96%87%E6%AA%94-60c45e04afa8)

後端工程師在提供 [RESTful API](https://searchapparchitecture.techtarget.com/definition/RESTful-API) 給前端工程師或客戶時，常常會需要說明API 的路徑、調用方式、以及回傳的 response 等工作。如果用口頭或是 World 等文書軟體溝通的話，常會遇到如何規範文件格式、文檔與程式不一致的問題。

因此，常見的方式是依照 Linux 基金會主持設立的 [**OpenAPI 規範（OpenAPI Specification, OAS）**](https://swagger.io/specification/)，來生成描述 API 的文件。而產生文檔的工具則是用由 Reverb technologies 開發並於 2015 年將貢獻給 OpenAPI 基金會的 [Swagger](https://swagger.io/)。

Swagger 是一種和編程語言無關，圍繞 OpenAPI 的一組開源工具。當你寫完 RESTful API 時，你可以用 Swagger-Editor 來編寫 API 文檔（ yaml 或 json ）或用相關套件幫你自動解析，然後通過 Swagger-UI 來渲染出如下的介面。透過這樣，前端或客戶可以很清楚的知道目前有哪些 API、請求的方式、需要傳的參數以及回傳的結果。如果不太清楚，可以玩玩看 Swagger 官網提供的 [demo 網站](https://petstore.swagger.io/?_ga=2.168783145.904299940.1589011534-393926135.1575990212)。

![](https://miro.medium.com/max/1400/0*d6gMyFfIrrEXMUs0.png)

image source: [https://github.com/axnsan12/drf-yasg](https://github.com/axnsan12/drf-yasg)

python 有許多網站框架，常見的是功能全面的 Django、以及輕量級的 flask 。Django 可以用來撰寫 RESTful API，但如果想要更多功能的話，可以參考本篇說明使用 Django REST framework 撰寫 API 並搭配 Swagger 套件 drf-yasg 來生成Swagger 文檔。

1\. 安裝 Django REST framework
----------------------------

[Django REST framework](https://www.django-rest-framework.org/) 是一個用來撰寫 RESTful API 好用的框架，在使用之前需要先安裝 [Django](https://www.djangoproject.com/) 以及 Django REST framework：

pip install django  
pip install djangorestframework  
pip install markdown       # （可不安裝）  
pip install django-filter  # （可不安裝）

安裝完後我們就要生成一個 Django project。再用 Django 建置網站的時候我們會生成一個 project（**專案**），以及數個 app（應用程式）。project 負責管理所有的 app，而 app 則是幫我們實現網站的基本功能。比方說我想練習做一個 Facebook ，可以將 project 取名 Facebook，然後建立數個 app，如管理用戶的 user、管理社團的 group 等 app 來幫我們實現網站需要的功能。

生成 project 的方式只要執行 Django 以下的指令即可：

django-admin startproject <your project name>

在本篇中的範例 project 為 demo 所以透過以下指令生成：

django-admin startproject demo

執行完後可發現 django 幫你生成了基本的 project：

demo/  
├── db.sqlite3  
├── demo/  
│   ├── \_\_init\_\_.py  
│   ├── settings.py <-- 等等要加 app  
│   ├── urls.py  
│   └── wsgi.py  
└── manage.py

再把 rest\_framework 加到 demo/settings.py 的 INSTALLED\_APPS

INSTALLED\_APPS = \[  
    ...  
    'rest\_framework',  
\]

初始化 project 後，我們需要新增 app 來幫我們實現網站的基本功能，所以我們先新增一個叫做 user 的 app 來實現用戶管理的功能：

django-admin startapp user

執行完後可以看到生成 user 囉：

demo/  
├── db.sqlite3  
├── demo/ <略>  
├── manage.py  
└── user/ <-- 剛剛新增的 app  
    ├── \_\_init\_\_.py  
    ├── admin.py  
    ├── apps.py  
    ├── migrations/ <略>  
    ├── models.py  
    ├── tests.py  
    └── views.py

再把 user 加到 _demo/settings.py_ 的 INSTALLED\_APPS 就成功建立 user 這個 app 了！

INSTALLED\_APPS = \[  
    ...  
    'user',  
\]

接下來因為 Django 採用 [MTV](https://medium.com/@zoejoyuliao/%E6%AF%94%E8%BC%83-mvc-%E8%88%87-django-%E7%9A%84-mtv-6c93ea9484fc) 框架，在初始化 app 後我們要開始撰寫 models、serializers、views 還有 urls 才能生成 RESTful API。

2\. 寫 models
------------

models 可以幫我們透過 [ORM（Object-Relation Mapping）](https://www.itread01.com/content/1544005142.html)對資料庫進行增刪改查。也就是說我們不需要撰寫 sql 語法，就可以以操作 object 的方式與資料庫交互，即使更換 settings.py 的資料庫設定也不影響程式。

在這裏我們繼承 Django 的 Model 建立一個名為 User 的 class，並新增欄位 id、email、以及 name，因為不希望用戶註冊的 email 重複，所以設置 unique=True：

from django.db import models

class User(models.Model):  
    id = models.AutoField(auto\_created=True, primary\_key=True)  
    email = models.EmailField(unique=True)  
    name = models.CharField(max\_length=100)

執行 makemigrations 在 migrations 目錄下生成紀錄 models 變動的 .py 檔：

python manage.py makemigrations

就會看到建立成功的提示：

![](https://miro.medium.com/max/1090/1*BVCn_pGyvQ_VfV8Fpbs13g.png)

makemigrations

然後再執行 migrate 操作將改動寫入資料庫：

python manage.py migrate

![](https://miro.medium.com/max/1400/1*Q4VZTTUPFFk-FrkCQL_Ftg.png)

migrate

可以看到我們在資料庫成功建立 User 的表囉！（Django 默認用 SQLite）

3\. 寫 serializers
-----------------

寫完 models 後，我們需要再 user 目錄下面新增 serializers.py 做資料的處理、驗證、以及資料的轉換，以序列化生成我們需要的 json 或 xml 格式：

demo/  
├── db.sqlite3  
├── demo/ <略>  
├── manage.py  
└── user/  
    ├── \_\_init\_\_.py  
    ├── admin.py  
    ├── apps.py  
    ├── migrations/ <略>  
    ├── models.py  
    ├── tests.py  
    ├── views.py  
    └── serializers.py <-- 新增

在 _user/serializers.py_ 貼上以下程式：

from user.models import User  
from rest\_framework import serializers  
class UserSerializer(serializers.ModelSerializer):  
     class Meta:  
         model = User  
         fields = '\_\_all\_\_'

這樣 UserSerializer 會依照 User 的欄位（fields）進行資料處理、驗證以及轉換。因為所有欄位都要所以 fields 設為 \_\_all\_\_。

4\. 寫 views
-----------

寫完後就剩 view 還有 url 了！請將下面的程式貼到 _user/views.py_

from django.http import JsonResponse  
from django.db import transaction  
from rest\_framework.generics import GenericAPIViewfrom user.serializers import UserSerializer   
from user.models import User  
class UsersView(GenericAPIView): queryset = User.objects.all()  
    serializer\_class = UserSerializer def get(self, request, \*args, \*\*krgs):  
        users = self.get\_queryset()  
        serializer = self.serializer\_class(users, many=True)  
        data = serializer.data  
        return JsonResponse(data, safe=False) def post(self, request, \*args, \*\*krgs):  
        data = request.data  
        try:  
            serializer = self.serializer\_class(data=data)  
            serializer.is\_valid(raise\_exception=True)  
            with transaction.atomic():  
                serializer.save()  
            data = serializer.data  
        except Exception as e:  
            data = {'error': str(e)}  
        return JsonResponse(data)

Django 的 view 有兩種形式，一種是 functional based 一種是 class based，Django REST framework 採用的是 class based，如果不了解之間的差異可以參考 [Django : Class Based Views vs Function Based View](https://medium.com/@ksarthak4ever/django-class-based-views-vs-function-based-view-e74b47b2e41b)。上面的程式是繼承 Django REST framework 的 [GenericAPIView](https://www.itread01.com/content/1546925049.html) 實現 UsersView 的 get（獲得 user 列表） 以及 post （創建 user）的方法。

5\. 寫 urls
----------

寫完 view 後我們需要將新增 UsersView 對應的 URL，常見的做法是在 user 目錄下新增 url.py 檔案或是直接加在 project 下面的 url.py。如果 URL 不多的話可以直接加在 project 下面，因為我們只有一個 app，所以就直接修改 _demo/urls.py_：

from user import views as user\_viewurlpatterns = \[  
    ...  
    path('users/', user\_view.UsersView.as\_view()),  
\]

這樣就寫完基本的 API 囉！接下來要安裝 drf-yasg 來生成 swagger，可以參考下篇：


[Source](https://zoejoyuliao.medium.com/%E7%94%A8-django-rest-framework-%E6%92%B0%E5%AF%AB-restful-api-%E4%B8%A6%E7%94%9F%E6%88%90-swagger-%E6%96%87%E6%AA%94-7cbef7c8e8d6)