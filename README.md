



```
sudo yum install python3
```

判斷 python 版本
```
> python --version          
Python 3.9.7
```

切換 python 版本
```
sudo rm /usr/bin/python
sudo ln -s /usr/bin/python3.6 /usr/bin/python
```

安裝 Django REST framework
```
pip install django
pip install djangorestframework
```

判斷 Django 版本
```
> python -m django --version
4.0.3
```

切換 Django 版本
```
pip3 install django==4.0.3
```


建立一個 Django項目
```
django-admin startproject DjangoDemo
```


再把 rest_framework 加到 DjangoDemo/settings.py 的 INSTALLED_APPS
```
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

新增一個叫做 user 的 app 來實現用戶管理的功能：
```
django-admin startapp user
```



再把 user 加到 DjangoDemo/settings.py 的 INSTALLED_APPS 就成功建立 user 這個 app 了
```
INSTALLED_APPS = [
    ...
    'user',
]
```



models 可以幫我們透過 ORM（Object-Relation Mapping）對資料庫進行增刪改查。

在這裏我們繼承 Django 的 Model 建立一個名為 User 的 class，並新增欄位 id、email、以及 name，因為不希望用戶註冊的 email 重複，所以設置 unique=True：

user/models.py
```
from django.db import models


class User(models.Model):
    id = models.AutoField(auto_created=True, primary_key=True)
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=100)
```



然後再執行 migrate 操作將改動寫入資料庫：

```
python manage.py migrate
```


user 目錄下面新增 serializers.py 做資料的處理
```
DjangoDemo/
├── db.sqlite3
├── DjangoDemo/ <略>
├── manage.py
└── user/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations/ <略>
    ├── models.py
    ├── tests.py
    ├── views.py
    └── serializers.py <-- 新增
```



在 user/serializers.py 貼上以下程式<br>
這樣 UserSerializer 會依照 User 的欄位（fields）進行資料處理、驗證以及轉換。因為所有欄位都要所以 fields 設為 __all__。
```
from user.models import User
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
     class Meta:
         model = User
         fields = '__all__'
```


寫完後就剩 view 還有 url 了！請將下面的程式貼到 user/views.py
```
from django.shortcuts import render
from django.http import JsonResponse
from django.db import transaction
from rest_framework.generics import GenericAPIView
from user.serializers import UserSerializer 
from user.models import User

class UsersView(GenericAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

    def get(self, request, *args, **krgs):
        users = self.get_queryset()
        serializer = self.serializer_class(users, many=True)
        data = serializer.data
        return JsonResponse(data, safe=False)

    def post(self, request, *args, **krgs):
        data = request.data
        try:
            serializer = self.serializer_class(data=data)
            serializer.is_valid(raise_exception=True)
            with transaction.atomic():
                serializer.save()
            data = serializer.data
        except Exception as e:
            data = {'error': str(e)}
        return JsonResponse(data)
```


寫完 view 後我們需要將新增 UsersView 對應的 URL，常見的做法是在 user 目錄下新增 url.py 檔案或是直接加在 project 下面的 url.py。如果 URL 不多的話可以直接加在 project 下面，因為我們只有一個 app，所以就直接修改 DjangoDemo/urls.py：

```
from user import views as user_view
urlpatterns = [
    ...
    path('users/', user_view.UsersView.as_view()),
]
```


## 用 Django REST framework swagger 套件 drf_yasg 生成 Swagger UI
drf-yasg 是一個可以在 Django REST framework 自動生成 Swagger 的套件。我們先用 pip 安裝：
```
pip install -U drf-yasg
```

再將 drf_yasg 加到的 DjangoDemo/settings.py：
```
INSTALLED_APPS = [
   ...
   'drf_yasg',
]
```

修改 DjangoDemo/urls.py 新增 drf_yasg 的 URL（建議修改 openapi.Info 的 title、contact 等資訊）：

```
...
# from django.conf.urls import url
from django.urls import re_path as url
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from user import views as user_view
...
schema_view = get_schema_view(
   openapi.Info(
      title="Snippets API",
      default_version='v1',
      description="Test description",
      terms_of_service="https://www.google.com/policies/terms/",
      contact=openapi.Contact(email="contact@snippets.local"),
      license=openapi.License(name="BSD License"),
   ),
   public=True,
   permission_classes=(permissions.AllowAny,),
)
urlpatterns += [
   url(r'^swagger(?P<format>\.json|\.yaml)$', schema_view.without_ui(cache_timeout=0), name='schema-json'),
   url(r'^swagger/$', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
   url(r'^redoc/$', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
   ...
]
```


完成後透過指令啟動 server：
```
python manage.py runserver
```

`NameError: name 'url' is not defined` 參考:
<https://stackoverflow.com/questions/70319606/importerror-cannot-import-name-url-from-django-conf-urls-after-upgrading-to>



https://stackoverflow.com/questions/25771755/django-operationalerror-no-such-table

`sqlite3.OperationalError: no such table: user_user` 參考:
```
python manage.py migrate --run-syncdb

python manage.py makemigrations user
python manage.py migrate user
```



docker run -itd centos

docker run -itd -p 8000:8000 python:3

docker exec -it <CONTAINER ID> bash


pip install django
pip install djangorestframework
pip install -U drf-yasg

git clone https://github.com/rockexe0000/Django-restful

cd Django-restful/demo

python manage.py runserver




http://127.0.0.1:8000/swagger/





















































refer:

<https://www.djangoproject.com/start/>

<https://www.django-rest-framework.org/tutorial/quickstart/>

https://github.com/twtrubiks/django-rest-framework-tutorial

<https://ithelp.ithome.com.tw/articles/10218874>

