# django 2.0 笔记

## 网站开发流程

1. 需求分析：了解网站的类型、具体功能、业务逻辑以及网站风格设计等，此外确定域名、网站空间、服务器以及网站备案
2. 规划静态内容：修正需求分析，根据用户的要求规划出网站内容板块草图
3. 设计阶段：根据网站草图，制作成效果图
4. 程序开发阶段：根据草图划分页面结构和设计，前端和后端可同时进行。前端根据效果图制作静态页面，后端根据页面结构和设计，设计数据库结构和开发网站后台
5. 测试和上线：本地搭建服务器，测试网站是否存在BUG，然后上传服务器进行上线
6. 维护推广：上线后根据实际情况完善网站的不足之处，定期修复和升级，保障网站运营顺畅，然后对网站进行推广宣传等

## django 建站基础

#### MTV 框架模式

* 模型（Model）：数据存取层，处理与数据相关的所有事务，例如：如何存取数据、验证数据有效性、包含那些行为以及数据之间的关系等
* 模板（Template）：表现层，提供了一个对设计者友好的语法用于渲染向用户呈现的信息。学习如何使用语法（面向设计者）以及如何扩展（面向程序员）
* 视图（View）：业务逻辑层，负责处理用户的请求并返回响应。存取模型及调取恰当模板的相关逻辑，模型与模板的桥梁

#### django 提供的功能（其他功能可通过第三方插件实现）

* 对象关系映射（Object Relational Mapping，ORM）：通过定义映射类来构建数据模型，将模型与关系数据库连接起来。方便对数据库的迁移，可适用多种数据库
* URL 设计：可设计任意形式的 URL，将真实地址进行隐藏等
* 模板系统：提供可扩展的模板语言，模板之间具有可继承可扩展
* 表单处理：自带各种表单模型，可继承扩展，具有有效性检验功能
* Cache 缓存：支持多种缓存方式
* 用户管理系统：提供用户认证、权限设置和用户组功能，功能扩展性强
* 国际化：内置国际化系统，方便开发出多种语言的网站
* admin 后台管理系统：内置 admin 后台管理组件，系统扩展性强

#### django 2.0 新特性

* 简化 URL 路由语法：使得 django.urls.path() 方法的语法更简单
* admin 后台管理系统：支持主流的移动设备并新增属性 ModelAdmin.autocomplete_fields 和方法 ModelAdmin.get_autocomplete_fields()
* 用户认证：PBKDF2 密码哈希默认迭代次数从 36000 增加到 100000
* Cache 缓存：cache.set_many() 返回一个列表，包含插入失败的键值对
* 通用视图：ContextMixin.extra_context 属性允许在 View.as_view() 中添加上下文
* Pagination 分页：增加 Paginator.get_page()，可以处理各种非法页面参数防止异常
* Templates 模板：提高 Engine.get_default() 在第三方模块的用途
* Validators 验证器：不允许 CharField 及其子类的表单输入为空
* File Storage 文件存储：File.opoen() 可以用于上下文管理器
* 连接 MySQL：由 mysqldb 模块改为 mysqlclient，使用差异不大
* Management Commands 管理命令：inspectdb 将 MySQL 的无符号整数视作 PositiveIntegerField 或者 PositiveSmallIntegerField 字段类型

#### 项目结构

* 项目文件说明（Project）
    * manage.py：命令行工具，用于和项目进行交互
    * __init__.py：初始化文件
    * settings.py：项目的配置文件
    * urls.py：项目的 URL 设置
    * wsgi.py：全称 Python web Server Gateway Interface，Python 服务器网关接口，是 Python 应用与Web服务器之间的接口，用于 Django 项目在服务器上的部署和上线

* 应用文件说明（APP）
    * migratons：用于数据库数据的迁移
    * __init__.py：初始化文件
    * admin.py：当前 APP 的后台管理系统
    * apps.py：当前 APP 的配置信息
    * models.py：定义映射类关联数据库，实现数据持久化
    * test.py：自动化测试模块
    * views.py：逻辑处理模块

## django 配置信息

**主要配置:项目路径、密钥配置、域名访问权限、APP 列表和中间件** [配置文件](settings.py) 

* 密钥配置 SECRET_KEY：用于用户密码、CSRF机制和会话 SESSION 等数据加密

* APP 列表 INSETALL_APPS：配置应用，将应用加载到项目中
    默认加载的应用：
        * admin：内置的后台管理系统
        * auth：内置的用户认证系统
        * contenttypes：记录项目中 model 元数据（ORM框架）
        * sessions：Session 会话功能
        * messages：消息提醒功能
        * staticfile：查找静态资源路径
    自己创建的应用添加到该列表，让项目能够加载管理该应用

* 静态资源

``` 静态资源
# Static files (css, JavaScript, Images) 将静态文件放在 static 中，static 放在 APP 应用里

STATIC_URL = '/static/'

# 设置根目录的静态资源文件夹 static

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)

# 收集整个项目的静态文件，并放在一个新的文件夹中

STATIC_ROOT = os.path.join(BASE_DIR, "static")
···

* 数据库配置 DATABASES：多数据库配置模板
··· 默认使用 default 数据库
DATABASES = {
	# sqlite3 数据库
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
    # MySQL 数据库
    'mysql': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': "数据库名",
        "USER": "用户名",
        "PASSWORD": "密码",
        "HOST": "IP地址",
        "PORT": "端口",
    }
}
```

* 中间件 MIDDLEWARE：处理 request 和 response 的钩子

    * SecurityMiddleware：内置的安全机制，保护用户与网站的的通信安全
    * SessionMiddleware：会话 session 功能
    * CommonMiddleware：处理请求信息，规范化请求内容
    * CsrfViewMiddleware：开启 CSRF 防护功能
    * AuthenticationMiddleware：开启内置的用户认证系统
    * MessageMiddleware：开启内置信息提示功能
    * XFrameOptionsMiddleware：防止恶意程序点击劫持

    **注意：中间件的设置顺序是固定的，顺序的更改可能导致程序的异常**

## 编写 URL 规则

**注意：在 APP 应用中添加 urls.py 将其应用的 URL 写在本 urls.py 中，将 urls.py 导入到项目的 urls.py 中，防止应用过多，导致 URL 混乱**

#### django.urls.path 和 django.urls.re_path 简介

path('url 地址', 映射的 View 函数, {'键值对': '额外参数'}, name='Template 中的重命名')

url 地址中可带参数

    * 字符类型：匹配任何非空字符串，但不含斜杠，默认类型。格式：<变量名> / <str:变量名>
    * 整型：匹配 0 和正整数。格式：<int:变量名>
    * slug：可理解为注释、后缀、附属、简称等，常用于作为 URL 的解释性字符，匹配任何的 ASCII 字符，使 URL 更加清晰。格式：<slug:变量名>
    * uuid：匹配 uuid 格式的对象，规定使用破折号，字母小写。格式：<uuid:变量名>

注：变量名用在 View 中接受的参数，必须一一对应，不然抛出参数不符合的异常

re_path('(?P<变量名>[正则表达式])/', 映射的 View 函数, name='Template 中的重命名')

    * ?P 是固定格式
    * <变量名> 正则表达式的变量，用于向 View 参数传递
    * [] 正则表达式

name='' 在 Template 中动态添加 URL，使其方便修改 URL

额外参数：只能以字典形式表示，键-参数名、值-参数值。参数只能在 View 视图中读取和使用

## 视图

#### 视图函数返回的相应类型

响应类型 | 说明
-------- | ---------
HttpResponse('内容') | HTTP 状态码 200，请求已成功被服务器接收
HttpResponseRedirect('URL/') | HTTP 状态码 302，重定向 URL
HttpResponseBadRequest('BadRequest') | HTTP 状态码 400，访问的页面不存在或者请求错误
HttpResponseNotFound('NotFound') | HTTP 状态码 404，网页不存在或网页的 URL 失效
HttpResponseForbidden('NotFound') | HTTP 状态码 403，没有访问权限
HttpResponseNotAllowed('NotAllowedGet') | HTTP 状态码 405，不允许使用该请求方式
HttpResponseServerError('ServerError') | HTTP 状态码 500， 服务器内容错误

django.shortcuts 模块将上述的 django.http 响应类型进行了封装。如：render()、redirect()等

#### 请求信息 request 常用属性

属性 | 说明 | 实例
----- | ----- | -----
COOKIES | 获取客户端（浏览器）Cookie 信息 | data = request.COOKIES
FILES | 字典对象，包含所有上传文件 | file = request.FILES
GET | 获取 GET 请求的请求参数，以字典形式存储 | request.GET.get('name')
META | 获取客户端的请求头信息，以字典形式存储 | request.META.get('REMOTE_ADDR') // 获取IP地址
POST | 获取 POST 请求的请求参数 | reques.POST.get('name')
method | 获取该请求的方式（GET/POST等） | data = request.method
path | 获取当前请求的 URL 地址 | path = request.path
user | 获取当前请求的用户信息 | name = request.user.username
body | 获取请求的主体内容 | data = request.body.decode()

FILES：该字典有三个键：filename 为上传文件的文件名；content-type 为上传文件的类型；content 为上传文件的原始内容
POST：content-type: application/x-www-form-urlencoded 可获取信息，若为 application/json 则需要在body中获取信息

#### 通用视图类

[ListView]: https://docs.djangoproject.com/zh-hans/2.1/ref/class-based-views/generic-display/#django.views.generic.list.ListView "ListView 官方文档"
[DetailView]: https://docs.djangoproject.com/zh-hans/2.1/ref/class-based-views/generic-display/#django.views.generic.detail.DetailView "DetailView 官方文档"

* ListView：将数据库的数据传递给 HTML 模板，通常返回列表形式的多条数据 [ListView]
* DetailView：将数据库的数据传递给 HTML 模板，通常返回一条详细数据 [DetailView]

[通用视图](https://docs.djangoproject.com/zh-hans/2.1/ref/class-based-views/base/)官方文档

## 模板

### 标签 {% 标签名 %} 变量 {{ 变量名 }}

#### django 常用的内置标签

标签 | 描述
----- | -----
{% for %} | 循环标签
{% if %} | 判断标签
{% csrf_token %} | 生成 csrf_token 标签，用于防护跨站请求伪造攻击
{% url %} | 引用相对 URL 地址
{% with %} | 变量重命名
{% load %} | 加载导入 django 的标签库
{% static %} | 读取静态文件配置
{% extend xxx %} | 继承 xxx 模板
{% block xxx %} | 父类模板定义子类可重写模块，重写父类模板的代码

#### for 标签中自带的一些常用变量

变量 | 描述
----- | -----
forloop.counter | 获取当前索引值，从 1 开始计算
forloop.counter0 | 获取当前索引值，从 0 开始计算
forloop.revcounter | 递减获取索引值，减至 1 的位置
forloop.revcounter0 | 递减获取索引值，减至 0 的位置
forloop.first | 当前遍历的元素是第一项时为真
forloop.last | 当前遍历的元素是最后一项时为真
forloop.parentloop | 在嵌套循环中，获取上层循环的 forloop 对象

#### 自定义过滤器

> 自定义过滤器：用于对变量的内容进行处理（替换、反序、转义等），将数据格式进行转化，减少视图函数中的代码量。
格式：{{ variable | filter }} 模板引擎先解析过滤器 filter 处理变量 variable 然后将处理后的变量显示在网页上。
可多个过滤器联合使用，从左到右依次解析，中间使用管道 | 进行连接

**django 内置过滤器**

过滤器 | 使用形式 | 说明
----- | ----- | -----
add | {{ value|add: "2" }} | 将 value 值加 2
addslashes | {{ value|addslashes }} | 在 value 中的引号前加反斜线
capfirst | {{ value|capfirst }} | value 的第一个字符转化成大写形式
cut | {{ value|cut:arg }} | 从 value 中删除所有 arg
date | {{ value|date："D d M Y" }} | 将日期格式数据按照给定的格式输出
default | {{ value|default："nothing" }} | 如果 value 的意义是 False，那么输出值为设置的默认值
default_if_none | {{ value|defalut_if_none:"nothing" }} | 如果 value 的意义是 None，那么输出值为设置的默认值
dictsort | {{ value|dictsort："name" }} | 将 value 列表中的元素（字典）按照关键字进行排序
dictsortreversed | {{ value|dictsortreversed:"name" }} | 将 value 列表中的元素（字典）按照关键字进行反序排序
divisibleby | {{ value|divisibleby:arg }} | 如果 value 能被 arg 整除，那么返回值为 True
escape | {{ value|escape }} | 控制 HTML 转义，替换 value 中的某些 HTML 特殊字符
escapejs | {{ value|escapejs }} | 替换 value 中的某些字符，适应 JavaScript 和 json 格式
filesizeformat | {{ value|filesizeformat }} | 格式化 value，使其成为易读的文件大小格式
first | {{ value|first }} | 返回列表中的第一个 Item
floatformat | {{ value|floatformat }} 或 {{ value|floatformat:arg}} | 对数据进行四舍五入处理，参数 arg 是保留的小数位
get_digit | {{ value|get_digit:"arg" }} | 输出某一位上的数，个：1 十：2 百：3 千：4
iriencode | {{ value|iriencode }} | 将 value 中的非 ASCII 字符转换成 URL 中适合的编码
join | {{ value|join:"arg" }} | 指定字符串连接 value 列表
last | {{ value|last }} | 返回列表中的最后一个 Item
length | {{ value|length }} | 返回 value 的长度

