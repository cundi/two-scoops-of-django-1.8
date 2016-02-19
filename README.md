# two-scoops-of-django-1.8
第三本  

# 第九章－ 函数视图最佳实践
****************
在Django中，项目开始时，函数视图被全世界的开发者广泛使用。

## 9.1 函数视图的优点
函数视图的简单性带来的是代码带重复使用上的损失：函数视图没有和类视图那样能够从超类继承的能力。


## 9.2 传递HttpRequest对象
当我们想在视图中重复使用代码时，而不是将代码绑定到中间件或者上下文处理器这样的全局行为。就像本书开始所介绍的那样，我们建议创建可以用到整个项目中的实用函数。  

对很多的实用函数而言，我们要接受一个属性或者是一个来自django.http.HttpRequest(或者简写为HttpRequest)对象的属性，然后获取数据或者执行行为。我们发现的是通过将request自身作为一个基本的参数，我们在更多的方法上便拥有了简单一些的参数。这就意味着，更少的在管理函数／方法参数上的认知负担：只需传递HttpRequest即可！  

*Example 9.1*  

```python
# sprinkles/utils.py

from django.core.exceptions import PermissionDenied


def check_sprinkle_rights(request):
    if request.user.can_sprinkle or request.user.is_staff:
        return request

    # Return a HTTP 403 back to the user 将HTTP状态码403返回给用户
    raise PermissionDenied
```

函数check_sprinkle_rights()对用户的权限作出了快速检查，抛出异常django.core.exception.PerssionDenied，此异常触发了一个我们在章节29.4.3中所描述的自定义HTTP 403视图。  

你也注意到了，我们返回的是HttpRequest对象，而不是任意一个值，或者None对象。我们之所以这样做，是因为Python是一种动态类型的语言，我们可以附加属性到HttpRequest。例如：  

Example 9.2  

```python
# sprinkle/utils.py

from django.core.exceptions import PermissionDenied


def check_sprinkles(request):
    if request.user.can_sprinkle or request.user.is_staff:
        # 通过此处添加的值，我们展示的模板可以更加通用。
        # 我们不需要使用{% if request.user.can_sprinkle or request.user.is_staff %}
        # 而是只使用{% if request.user.can_sprinkle %}就好。
        request.can_sprinkle = True
        return request

    # 将HTTP 403 返回给用户
    raise PermissionDenied
```

还有另外原因，我们会简短截说。这期间，让我们使用下面的代码来说明问题：  

Example 9.3  

```python
# sprinkle/views.py

from django.shortcuts import get_object_or_404
from django.shortcuts import render

from .utils import check_sprinkles
from .mdoels import Sprinkle


def sprinkle_list(request):
    """标准列表视图"""

    request = check_sprinkles(request)


    return render(request,
        "sprinkles/sprinkle_list.html",
        {"sprinkles": Sprinkle.objects.all()})


def sprinkle_detail(request, pk):
    """标准详细视图"""


```

## 9.3 甜蜜的装饰器
就这次来说，我们不谈冰激凌，全涌来说代码！在计算机科用语中，语法糖是一个种为了使代码易于阅读或者表达而添加到编程语言的语法。在Python中，装饰器一个由人为加入的必不可少的功能，这只是为了让在人类在阅读代码时，让代码看上去显得整洁和甜蜜。是的，装饰器是一道甜品。   

我们综合了简单函数和装饰器语法糖的力量，我们得到了，便于重复使用的能用到很多场合的工具django.contrib.auth.decorators.login_required装饰器。  

这里是一个简单的使用在函数视图中的装饰器模板的例子；  

Example 9.5  

```python
# simple decorator template
import functools


def decorator(view_func):
    @functools.wraps(view_func)
    def new_view_func(request, *args, **kwargs):
        # 此处可以修改request对象（HttpRequest）。
        response = view_func(request, *args, **kwargs)
        # 此处可以修改response对象（HttpReponse）。
        return response
    return new_view_func
```

这样做或许难以理解，所以我们会一步步的来详细说明，代码中的注释很明确地声明了我们正在干的事情。首先，我们来修改前面那个例子的装饰器模板以配合我们自身的需要：  

Example 9.6  

```python
```
