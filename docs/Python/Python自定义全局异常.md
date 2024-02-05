# Python自定义全局异常

## 需求

我们希望在一个controller层的函数func上标注一个装饰器，当func中抛出（raise）异常后，统一返回错误信息。



## 代码

> 自定义异常装饰器

```Python
from django.http import JsonResponse


def error_resp(f):
    def wrapper(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except Exception as e:
            return JsonResponse({  # 这里我们可以自定义返回异常信息
                'success': False,
                'msg': e.__str__()
            }, json_dumps_params={'ensure_ascii': False})

    return wrapper
```

> 使用

```Python
@error_resp  # 外层装饰器
@record_exception_log  # 内层装饰器（先不管它的作用）
def first_login(request):
    pass
```

【注意】一定要确保`error_resp`在最外层，并且保证内存装饰器不能捕获异常（或者内存装饰器捕获完异常后还要抛出异常）