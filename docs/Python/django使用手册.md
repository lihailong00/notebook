# django使用手册

[toc]

## 创建Django项目

1. 打开Pycharm，初始化项目`/pro`。

2. 安装django依赖。

3. `/pro`路径下，执行：`django-admin startproject mysite`。其中mysite是项目名。

4. 进入`/pro/mysite`，启动项目。

   ```bash
   # 方式1.直接运行项目，默认8000端口
   python manage.py runserver
   # 方式2.监听某个端口
   python manage.py runserver 0.0.0.0:8000
   ```

5. 创建一个应用，`/pro/mysite`路径下执行：`python manage.py startapp polls`。这里我们创建了一个名为polls的应用。

6. 将新创建的polls应用加载到`/pro/mysite/mysite/settings.py`文件中。

   ```python
   INSTALLED_APPS = [
       'django.contrib.admin',
       'django.contrib.auth',
       'django.contrib.contenttypes',
       'django.contrib.sessions',
       'django.contrib.messages',
       'django.contrib.staticfiles',
       # 下面是自己创建的应用
       'polls'
   ]
   ```

   



项目结构：

```
django-pro		# Python根路径
----mysite		# 根路径执行django-admin startproject mysite，创建此项目
--------mysite	# 根路径创建项目后，同时也会创建这个 主应用。
--------polls	# 自定义创建的 应用
```



项目包含应用。比如，图书管理系统是一个项目。借书是一个应用，还书是另一个应用。





## 编写Django应用

1. 编写polls应用的视图文件，类似Spring Boot的controller文件。

   编写文件`/pro/mysite/polls/urls.py`

   ```python
   from django.http import HttpResponse
   
   
   def index(request):
       return HttpResponse("Hello, world. You're at the polls index.")
   ```

2. 配置polls应用的URL映射关系。

   编写文件`/pro/mysite/polls/urls.py`

   ```python
   from django.urls import path
   from . import views
   
   urlpatterns = [
       path("", views.index, name="index"), # name属性特别适用于模板。对于前后端分离的项目来说，意义不大，可以不用设置name属性。
   ]
   ```

3. 编写mysite项目的URL映射关系。

   编写文件`/pro/mysite/mysite/urls.py`

   ```python
   from django.contrib import admin
   from django.urls import path, include
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('polls/', include('polls.urls'))  # 我添加的
   ]
   ```

4. 访问`http://localhost:8000/polls`。





## 连接数据库

Django会默认使用SQLite数据库。但是项目中通常使用MySQL数据库，因此需要如下配置。

1. 安装MySQL驱动程序：`pip install mysqlclient`。

2. `/pro/mysite/mysite/settings.py`中配置数据库。

   ```python
   ''' 默认数据库配置
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.sqlite3',
           'NAME': BASE_DIR / 'db.sqlite3',
       }
   }
   '''
   
   # Mysql 数据库配置
   DATABASES = {
       'default': {
           "ENGINE": "django.db.backends.mysql", # 选择MySQL引擎
           "NAME": "django_db",  # 数据库名
           "USER": "root",
           "PASSWORD": "root",
           "HOST": "127.0.0.1",
           "PORT": "3306",
       }
   }
   ```

3. 迁移数据库，进入`/pro/mysite`，执行：`python manage.py migrate`。此时可以看到数据库中生成了很多数据表。

4. 我们可以在自己的应用中创建Model，并同步到数据库。简单来说，就是在`/pro/polls/models.py`文件中创建一个class，然后数据库中自动生成对应的数据表。

5. 编写文件`/pro/polls/models.py`。

   ```python
   from django.db import models
   
   
   class Book(models.Model):
       title = models.CharField(max_length=100)
       author = models.CharField(max_length=100)
       publication_date = models.DateField()
       # 整数数位+小数数位 <= 8  两位小数  默认是0.00
       price = models.DecimalField(max_digits=8, decimal_places=2, default=0.00)
       code = models.IntegerField(default=0)
   
       # 一定要写这行！
       objects = models.Manager()
   
       def __str__(self):
           return 'title=' + self.title.__str__() + \
               '\nauthor=' + self.author.__str__() + \
               'publication_date=' + self.publication_date.__str__() + \
               '\nprice=' + self.price.__str__() + \
               '\ncode=' + self.code.__str__()
       
   ```

6. 迁移数据：

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

7. 备注：

   **强烈建议遵循迁移工具修改数据表！**

   **不要直接在数据库中修改/删除数据表！**这样可能导致迁移记录更新失败。可以删除makemigrations表中的迁移记录解决该问题。



## SQL增删改查





## 获取请求参数







参考acwing教程。

django-admin --version 查看django版本

django-admin startproject `acapp`

目录结构：

```
acapp/
|-- acapp
|   |-- __init__.py
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```



启动服务：

python3 manage.py runserver 0.0.0.0:8000

windows上把python3 改成 python



此时不能正常访问，需在`acapp/acapp/settings.py`中配置ALLOWED_HOSTS。可以输入`ag ALLOWD_HOSTS`查看关键字在哪里。



运行成功！

添加.gitignore，填写`**/__pychach__`，`*.swp`



python3 manage.py startapp game



创建管理员

python3 manage.py createsuperuser



进入game目录

touch urls.py

mkdir templates

```
modules views urls可以只是一个文件
templates必须是文件夹
modules 存数据（class）
views 存视图（函数逻辑）
urls 路由
```

此时目录结构是：

```
.
|-- acapp
|   |-- __init__.py
|   |-- __pycache__
|   |   |-- __init__.cpython-38.pyc
|   |   |-- settings.cpython-38.pyc
|   |   |-- urls.cpython-38.pyc
|   |   `-- wsgi.cpython-38.pyc
|   |-- asgi.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
|-- db.sqlite3
|-- game
|   |-- __init__.py
|   |-- admin.py
|   |-- apps.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- templates
|   |-- tests.py
|   |-- urls.py
|   `-- views.py
`-- manage.py
```



编辑game中的views.py文件。

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("我的第一个django网页")

def play(request):
    return HttpResponse("游戏界面")
```

编辑game中的urls文件：

```python
from django.urls import path
from game.views import index, play


urlpatterns = [
    path('game/', index, name="index"),
    path('play/', play, name='play')
]
```



将game中的urls引入到总的urls中（acapp中）

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('', include('game.urls')),
    path('admin/', admin.site.urls),
]
```





第二节课

django项目结构

```
templates目录: 管理html文件
urls目录: 路由配置文件
models目录:
views目录: http函数
static目录：存放静态文件，包括html，css，js，png...
consumers目录: 管理websocket函数
```



删掉之前的urls, models, views文件，建立对应的3个目录，分别创建`__init__.py`文件

重新配置acapp中的urls。



在acapp的settings.py中设置时区，将`TIME_ZONE`的值改为`Asia/Shanghai`



将之前创建的game加入acapp中：

```
game目录下找到apps.py，复制类名GameConfig。在acapp目录下的settings.py文件中的INSTALLED_APPS中加上game.apps.GameConfig，并且在STATIC_URL处加上这些代码：

# 习惯在static中存开发者文件
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
# 习惯在media中存放用户文件
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
STATIC_URL = '/static/'
```



然后再game/static 目录下创建如下目录

```
static
|-- audio
|-- css
|-- image
|   |-- menu
|   |-- playground
|   `-- settings
`-- js
```

使用wget将背景图片下载到服务器中。



编写脚本，将所有js文件放在某个目录。

在`/acapp/scripts/`下创建文件compress_game_js.sh，输入以下内容：

```bash
#! /bin/bash
  
JS_PATH=/home/lhl/acapp/game/static/js/
JS_PATH_DIST=${JS_PATH}dist/
JS_PATH_SRC=${JS_PATH}src/

find $JS_PATH_SRC -type f -name '*.js' | sort | xargs cat > ${JS_PATH_DIST}game.js
```

并在JS_PATH下创建dist和src目录。



acapp/game/template中添加文件夹multiends，并创建web.html文件，编写如下内容：

```html
{ load static %}
<head>
    <link rel="stylesheet" href="https://cdn.acwing.com/static/jquery-ui-dist/jquery-ui.min.css" />
    <script src="https://cdn.acwing.com/static/jquery/js/jquery-3.3.1.min.js"></script>
    <link rel="stylesheet" href="{% static 'css/game.css' %}" />
    <script src="{% static 'js/dist/game.js' %}"></script>
</head>

<body style="margin: 0">
    <div id="ac_game_12345678"></div>
    <script>
        $(document).ready(function() {
            let ac_game = new AcGame();
        })
    </script>
</body>
```





请求：

```python
def verify(request):
    print('request.method=' + str(request.method))
    # 前端访问：http://localhost:8000/verify?username=lhl&password=123
    print('request.GET=' + str(request.GET))
    print('request.POST=' + str(request.POST))
```



## B站

pip安装：mysqlclient
