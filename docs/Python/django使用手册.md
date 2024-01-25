# django使用手册

[toc]

## 创建Django项目

1. 打开Pycharm，Pycharm可以直接启动Django项目。

2. 也可以手动安装django依赖，并初始化Django项目：`django-admin startproject mysite`。其中mysite是项目名。

4. 我们将`manage.py`所在的目录称为**根目录**，在根目录下执行启动web服务。

   ```bash
   # 方式1.直接运行项目，默认8000端口
   python manage.py runserver
   # 方式2.监听某个端口
   python manage.py runserver 0.0.0.0:8000
   ```

5. 根目录下执行：`python manage.py startapp polls`。这里我们创建了一个名为polls的应用。

6. 将新创建的polls应用加载到`/mysite/mysite/settings.py`文件中。

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


6. 项目结构：

   ```
   mysite		# 根路径执行django-admin startproject mysite，创建此项目
   ----mysite	# 主应用
   ----polls	# 自定义应用
   ```



**项目**包含**应用**。比如，图书管理系统是一个项目。借书是一个应用，还书是另一个应用。



当然，我们也可以不用创建新应用，而是直接使用初始化的主应用，此时一定要在主应用目录下创建一个`migrations`文件，例如：`mysite/mysite/migrations`





## 编写Django应用

1. 编写polls应用的视图文件`/mysite/polls/views.py`。类似Spring Boot的controller文件。

   ```python
   from django.http import HttpResponse
   
   
   def index(request):
       return HttpResponse("Hello, world. You're at the polls index.")
   ```
   
2. 配置polls应用的URL映射关系，编写文件`/mysite/polls/urls.py`。

   ```python
   from django.urls import path
   from . import views
   
   urlpatterns = [
       path("", views.index, name="index"), # name属性特别适用于模板。对于前后端分离的项目来说，意义不大，可以不用设置name属性。
   ]
   ```
   
3. 编写mysite项目的URL映射关系，编写文件`/pro/mysite/mysite/urls.py`。

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

2. 主应用`/mysite/settings.py`中配置数据库。

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

3. 确保当前应用被注册：进入主应用的`/mysite/settings.py`，找到`INSTALLED_APPS`，注册自己的应用。

   ```python
   INSTALLED_APPS = [
       # 自己配置
       'polls'
   ]
   ```

4. 迁移数据库，进入根目录，执行：`python manage.py migrate`。此时可以看到数据库中生成了很多数据表。

5. 我们可以在自己的应用中创建Model，并同步到数据库。简单来说，就是在`/polls/models.py`文件中创建一个class，然后数据库中自动生成对应的数据表。

6. 编写文件`/polls/models.py`。

   ```python
   import decimal
   from datetime import datetime
   from django.db import models
   
   
   # 字段设置建议：建议所有值都设定默认值。
   class Book(models.Model):
       title: str = models.CharField(max_length=100)
       author: str = models.CharField(max_length=100)
       publication_date: datetime = models.DateTimeField(default=datetime.now())
       # 整数数位+小数数位 <= 8  两位小数  默认是0.00
       price: decimal = models.DecimalField(max_digits=8, decimal_places=2, default=0.00)
       code: int = models.IntegerField(default=0)
   
       # 不能直接使用__dict__ 因为__dict__中包含很多父类的字段，在使用update_or_create等方法时会报错
       def get_dict(self):
           result_dict = {
               'title': self.title,
               'author': self.author,
               'publication_date': self.publication_date,
               'price': self.price,
               'code': self.code
           }
           return result_dict
   
       # 假定需要对Book数组自定义排序，可以重写__lt__函数，然后可以直接sort这个数组: book_arr.sort()
       def __lt__(self, other):
           if isinstance(other, Book):
               if self.title != other.title:
                   return self.title < other.title
               if self.author != other.author:
                   return self.author < other.author
               return self.publication_date < other.publication_date
           return NotImplemented
   
       # 如果想对Book数组去重，则需要重写__hash__函数和__eq__函数。去重时只需要: book_arr = list(set(book_arr))
       def __hash__(self):
           return hash((self.title, self.author))
   
       def __eq__(self, other):
           if isinstance(other, Book):
               return (self.title == other.title
                       and self.author == other.author)
           return False
   
       class Meta:
           db_table = 'xnxy_book'  # 自定义数据表名
   
   ```
   
7. 迁移数据：

   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

8. 备注：

   **强烈建议遵循迁移工具修改数据表！**

   **不要直接在数据库中修改/删除数据表！**这样可能导致迁移记录更新失败。可以删除makemigrations表中的迁移记录解决该问题。



## SQL增删改查

> 增

```python
# 方法一
book: Book = Book(title='C++')
book.save()  # 保存失败，因为code属性没有默认值，不能为空

book: Book = Book(title='C++', code=1)  # 即便没有指定author，但是也会为author分配一个空字符串
book.save()  # 保存成功，注意save函数没有返回值


# 方法二 (建议)
inserted_book = Book.objects.create(title='C++', author='晓龙')  # 会返回插入到数据库中的值
print(inserted_book)

# 方法三 (更建议)
book = Book()
# ... 假定我们已经初始化了book对象
inserted_book = Book.objects.create(**book.get_dict())  # 这种方式可以直接将对象插入到数据库中
print(inserted_book)
```



> 批量增

```python
course_arr = [Course(), Course(), Course()]
Book.objects.bulk_create(course_arr)
```



> 不存在就【增】，存在就【改】

假定数据表中的数据如下：

| id   | title | author | price | code | publication_date           |
| ---- | ----- | ------ | ----- | ---- | -------------------------- |
| 1    | Java  | 一龙   | 1.00  | 1    | 2023-10-08 00:00:00.000000 |
| 2    | C++   | 二龙   | 2.00  | 2    | 2023-10-08 00:00:00.000000 |
| 3    | C++   | 三龙   | 3.00  | 3    | 2023-10-08 00:00:00.000000 |

```python
# defaults中时修改后的字段值，defaults中没有涉及到的字段保持原值。非defaults值就是查询条件。
# 两个返回值：第1个是修改/插入的对象的结构；第2个用于判断是否是新建的对象
# 确保只能查询出1个值
created_book, is_created = Book.objects.update_or_create(title='Java', defaults={'author': '大龙'})
if is_created:
    print('创建对象')
else:
    print('修改对象')
print(created_book)
'''
打印结果:
修改对象
{title:Java,author:大龙,publication_date:2023-10-08 00:00:00+00:00,price:1.00,code:1}
'''
```



```python
# 报错，因为存在两个满足条件的值
created_book, is_created = Book.objects.update_or_create(title='C++', defaults={'author': '大龙'})
```



```python
# 插入成功
created_book, is_created = Book.objects.update_or_create(title='Python', defaults={'author': '大龙', 'code': 3})
```



> 删

```python
Book.objects.filter(title='C++', author='晓龙').delete()
```



> 批量删除

这里的批量删除是指删除多个行记录，并且这些行记录的值各不相同。

```python
objects = [
    {'title': 'C++', 'author': '二龙'},
    {'title': 'Python', 'author': 'x龙'},
    {'title': 'PHP', 'author': '天龙'}
]
query_conditions = []
for obj in objects:
    query_conditions.append(Q(title=obj['title']) & Q(author=obj['author']))

final_query = reduce(lambda x, y: x | y, query_conditions)

result_arr = Book.objects.filter(final_query)
result_arr.delete()
```





> 改

```python
Book.objects.filter(title='C++').update(author='晓龙')
```



> 查

```python
from django.db.models import Q

book_query_set = Book.objects.filter(Q(title='C++') & Q(author='二龙'))
book_arr = [book for book in book_query_set]  # book_arr中的元素类型是Book
print(book_arr)
```





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
