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

#### 过滤器

> 过滤器：用于对变量的内容进行处理（替换、反序、转义等），将数据格式进行转化，减少视图函数中的代码量。
格式：{{ variable | filter }} 模板引擎先解析过滤器 filter 处理变量 variable 然后将处理后的变量显示在网页上。
可多个过滤器联合使用，从左到右依次解析，中间使用管道 | 进行连接

**django 内置过滤器**

过滤器 | 使用形式 | 说明
----- | ----- | -----
add | {{ value \| add: "n" }} | 将 value 值加 n
addslashes | {{ value \| addslashes }} | 在 value 中的引号前加反斜线
capfirst | {{ value|capfirst }} | value 的第一个字符转化成大写形式
cut | {{ value \| cut:arg }} | 从 value 中删除所有 arg
date | {{ value \| date："D d M Y" }} | 将日期格式数据按照给定的格式输出
default | {{ value \| default："nothing" }} | 如果 value 的意义是 False，那么输出值为设置的默认值
default_if_none | {{ value \| defalut_if_none:"nothing" }} | 如果 value 的意义是 None，那么输出值为设置的默认值
dictsort | {{ value \| dictsort："name" }} | 将 value 列表中的元素（字典）按照关键字进行排序
dictsortreversed | {{ value \| dictsortreversed:"name" }} | 将 value 列表中的元素（字典）按照关键字进行反序排序
divisibleby | {{ value \| divisibleby:arg }} | 如果 value 能被 arg 整除，那么返回值为 True
escape | {{ value \| escape }} | 控制 HTML 转义，替换 value 中的某些 HTML 特殊字符
escapejs | {{ value \| escapejs }} | 替换 value 中的某些字符，适应 JavaScript 和 json 格式
filesizeformat | {{ value \| filesizeformat }} | 格式化 value，使其成为易读的文件大小格式
first | {{ value \| first }} | 返回列表中的第一个 Item
floatformat | {{ value \| floatformat }} 或 {{ value \| floatformat:arg}} | 对数据进行四舍五入处理，参数 arg 是保留的小数位
get_digit | {{ value \| get_digit:"arg" }} | 输出某一位上的数，个：1 十：2 百：3 千：4
iriencode | {{ value \| iriencode }} | 将 value 中的非 ASCII 字符转换成 URL 中适合的编码
join | {{ value \| join:"arg" }} | 指定字符串连接 value 列表
last | {{ value \| last }} | 返回列表中的最后一个 Item
length | {{ value \| length }} | 返回 value 的长度
length_is | {{ value \| length_is:"arg" }} | 如果 value 的长度等于 arg，返回 True
linebreaks | {{ value \| linebreaks }} | 将 value 中的 "\n" 用 <br> 替换，并使用 <p> 将 value 包围
linebreaksbr | {{ value \| linebreaksbr }} | 将 value 中的 "\n" 用 <br> 替换
linenumbers | {{ value \| linenumbers }} | 为显示的文本添加行数
ljust | {{ value \| ljust }} | 以左对齐方式显示 value
center | {{ value \| center }} | 以居中对齐方式显示 value
rjust | {{ value \| rjust }} | 以右对齐方式显示 value
lower | {{ value \| lower }} | 将 value 转换成小写
make_list | {{ value \| make_list }} | 将 value 转换成 list 列表
pluralize | {{ value \| pluralize }} 或 {{ value \| pluralize:"es" }} 或 {{ value \| pluralize:"y,ies" }} | 将 value 返回英文的复数形式
random | {{ value \| random }} | 从 value list 列表中返回一个随机的 Item
removetags | {{ value \| removetags:"tag1 tag2 tag3"}} | 删除 value 中 tag1、tag2、tag3 的标签
safe | {{ value \| safe }} | 关闭 HTML 转义，告诉 django 这段代码不需要转义， 针对字符串
safeseq | {{ value \| safeseq }} | 和 safe 相同，但 safeseq 针对多个字符串组成的 sequence
slice | {{ value \| slice:":n" }} | 截取前 n 个字符
slugify | {{ value \| slugify }} | 将 value 转换成 小写形式，同时删除所有的分单词字符，并将空格转换成横线
striptags | {{ value \| striptags }} | 删除 value 中的所有 HTML 标签
time | {{ value \| time:"H:i" }} 或 {{ value \| time }} | 格式化时间输出
truncatewords | {{ value \| truncatewords:n }} | 将 value 进行单词截取，参数 n 表示截取前 n 个单词，只可用于英文
upper | {{ value \| upper }} | 将 value 转换成大写 
urlize | {{ value \| urlize }} | 将字符串中的 URL 转换成可点击的形式
wordcount | {{ value \| wordcount }} | 返回字符串中的单词数目
wordwarp | {{ value \| wordwarp:n }} | 按照指定长度 n 分割字符串
timesince | {{ value \| timesince:arg }} | 返回参数 arg 到 value 的天数和小时数
timeuntil | {{ value \| timeuntil }} | 返回 value 距离当前日期的天数和小时数

**django自定义过滤器**

* 在项目或应用中创建 templatetags 文件夹（必须为该名）
* 若在项目中自定义过滤器则需要在配置文件 settings.py 的 INSTALLED_APPS 里添加该文件夹，应用中不需要单独添加
``` 创建自定义过滤器代码格式
from.django import template
register = template.Library()

@register.filter
def myfilter(value, args):
    pass
```
* 引用自定义过滤器 {{ value \| myfilter }} 或 {{ value \| myfilter:"arg" }}

## 模型与数据库

### django 使用 ORM 框架给常用的数据库提供了统一的调用 API

**创建的数据库必须继承 models.Model 类**

#### 数据表常用字段类型说明

表字段 | 说明
----- | -----
models.AutoField | 默认会生成一个名为 id 的字段并为 int 类型
models.CharField | 字符串类型
models.BooleanField | 布尔类型
models.ComaSeparatedIntegerField | 用逗号分割的整数类型
models.DateField | 日期（Date）类型
models.DateTimefield | 日期（DateTime）类型
models.Decimal | 十进制小数类型
models.EmailField | 字符串类型（邮箱正则表达式检验）
models.FloatField | 浮点数类型
models.IntegerField | 整数类型
models.BigIntergerField | 长整数类型
models.IPAddressField | 字符串类型（IPv4 正则表达式检验）
models.GenericIPAddressField | 字符串类型，参数 protocol 可以是：both、IPv4、IPv6，检验 IP 地址
models.NullBooleanfield | 允许为空值的布尔类型
models.PositiveIntegerField | 正整数类型
models.PositiveSmallIntegerField | 小正整数类型
models.SlugField | 包含字母、数字、下划线和连字符的字符串
models.SmallIntegerField | 小整数类型
models.TextField | 长文本类型
models.TimeField | 时间类型 显示时分秒 HH:MM[:ss[.uuuuuu]]
models.URLField | 字符串类型，地址为正则表达式
models.BinaryField | 二进制数据类型

#### 数据表字段常用参数

参数 | 说明
----- | -----
Null | 默认为 False 若为 True 表示字段可以为空
Blank | 默认为 False 若为 True 表示在 Admin 站点管理中添加数据是允许设置空值
Default | 设置默认值
primary_key | 若为 True 表示将该字段设置为主键
db_column | 设置数据库中的字段名称
Unique | 默认为 False 若为 True 表示字段设置成唯一属性
db_index | 若为 True 表示为该字段添加数据库索引
verbose_name | 在 Admin 站点管理设置字段显示的名称
related_name | 关联对象反向引用描述符，用于多表查询，可解决一个数据表有两个外键同时指向另一个数据表而出现重名的问题

### 数据库提供的关系
* 一对一关系，外键可在任意一端
* 一对多关系，外键必须在多端
* 多对多关系，外键在两端，每一端都需要有外键

### 数据读写

#### 插入数据

**方法一**
* 实例化 Model 模型 model = XxxModel()
* 模型对象赋值 model.xxx = xxx
* 数据插入数据库 model.save()

**方法二**
* XxxModel.object.create(属性赋值（字段名=值）)

**方法三**
* 实例化 Model 模型 model = XxxModel(属性赋值（字段名=值）)
* 数据插入数据库 model.save()

#### 查询数据

* 查询单条数据 XxxModel.object.get(属性（字段名=值）) 查询字段必须是主键或唯一约束的字段，并且查询数据必须存在，否则报错
* 查询多条数据 XxxModel.object.filter(属性（字段名=值）) 查询字段任意，若不存在数据，返回空列表
* 查询全部数据 XxxModel.object.all() 返回整表信息，大数据时不建议使用，建议在后面添加列表截取 [:] 后台转化成 limit 去查询
* 查询某个字段 XxxModel.object.values('字段名') 以列表形式返回，每个数据时一个字典（字段名:值）
* 查询某个字段 XxxModel.object.values_list('字段名') 以列表形式返回，每个数据时一个元组（值）

**查询中的特殊条件**
* 查询在 filter 中多个条件同时满足的数据 即 and 直接在 filter(属性1，属性2) 使用逗号（，）分割
* 查询在 filter 中多个条件满足其中一条即可的数据 即 or 需要引入 Q() 函数 filter(Q(属性1)|Q(属性2)) 导入Q函数包 from django.db.models import Q
* count() 计数函数，计算 filter().count() 返回的数据条数
* distinct() 去重函数，去除 values(字段名).filter().distinct() 返回的重复数据
* order_by() 排序函数，根据 order_by(字段名) 进行排序 -字段名 降序 默认升序
* annotate() 分组函数，类似 SQL 中的 GROUP BY
* aggregate() 计算函数，类似 SQL 中的 COUNT()

**匹配符的使用以及说明**

匹配符 | 使用 | 说明
----- | ----- | -----
\_\_exact | filter(字段名\_\_exact=条件) | 精准等于，如 SQL 的 like 条件
\_\_iexact | filter(字段名\_\_iexact=条件) | 精准等于并忽略大小写
\_\_contains | filter(字段名\_\_contains=条件) | 模糊匹配 如 SQL 的 like %条件%
\_\_icontains | filter(字段名\_\_icontains=条件) | 模糊匹配并忽略大小写
\_\_gt | filter(字段名\_\_gt=条件) | 大于条件
\_\_gte | filter(字段名\_\_gte=条件) | 大于等于条件
\_\_lt | filter(字段名\_\_lt=条件) | 小于条件
\_\_lte | filter(字段名\_\_lte=条件) | 小于等于条件
\_\_in | filter(字段名\_\_in=[条件1, 条件2]) | 判断是否在列表中
\_\_startswith | filter(字段名\_\_startswith=条件) | 以条件开头
\_\_istartswith | filter(字段名\_\_istartswith=条件) | 以条件开头并忽略大小写
\_\_endswith | filter(字段名\_\_endswith=条件) | 以条件结尾
\_\_iendswith | filter(字段名\_\_iendswith=条件) | 以条件结尾并忽略大小写
\_\_range | filter(字段名\_\_range=条件) | 在条件范围内
\_\_year | filter(字段名\_\_year=条件) | 日期字段的年份
\_\_month | filter(字段名\_\_month=条件) | 日期字段的月份
\_\_day | filter(字段名\_\_day=条件) | 日期字段的天数
\_\_isnull | filter(字段名\_\_isnull=True/False) | 判断是否为空

**多表查询**

* 正向查询：如果查询对象的主体和要查询的数据是同一模型，但条件包含外键条件，则称为正向查询，执行一次 SQL 语句 如：Model.object.filter(other__id=num)
* 反向查询：如果查询对象的主体和要查询的数据不是同一模型 则称为反向查询，执行两次 SQL 语句  如：model[num].other__set.values(字段名)
* select_related 方法：实现只执行一次 SQL 进行反向查询，可以实现多表联合查询 如：Model.object.select_related('外键字段名__其他表外键字段名').values(字段名)

#### 更新数据

**方法一**
* 获取需要更新数据的对象 model = XxxModel.object.get(属性（字段名=值）)
* 数据更新至数据库 model.save()

**方法二**
* 更新单条数据 XxxModel.object.get(属性（字段名=值）).update(修改属性赋值（字段名=值）)
* 更新多条数据 XxxModel.object.filter(属性（字段名=值）).update(修改属性赋值（字段名=值）)
* 更新全部数据 XxxModel.object.update(修改属性赋值（字段名=值）)

#### 删除数据

* 删除单条数据 XxxModel.object.get(属性（字段名=值）).delete()
* 删除多条数据 XxxModel.object.filter(属性（字段名=值）).delete()
* 删除全部数据 XxxModel.object.all().delete()

## 表单和模型

**django 提供了表单模型**
由 Form 类实现，主要有：django.forms.Form 和 django.forms.ModelForm 实现

#### django.forms.Form 表单

##### django 内置的表单字段

字段 | 说明
----- | -----
BooleanField | 复选框，如果字段带有 required=True，复选框默认被选定
CharField | 文本框，参数 max_length 和 min_length 分别设置输入长度
ChoiceField | 下拉框，参数 choices 设置数据内容
TypeChoiceField | 下拉框，比 ChoiceField 多出参数 coerce 数据类型强制转换 和 empty_value 空值
DateField | 文本框，具有验证日期格式的功能，参数 input_formats 设置日期格式
EmailField | 文本框，具有验证邮箱格式的功能，可选参数 max_length 和 min_length
FileField | 文件上传功能，参数 max_length 文件名最大长度 和 allow_empty_file 文件内容是否为空
FilePathField | 在特定的目录选择并上传文件 必要参数 path 可选参数 recursive match allow_files allow_folders
FloatField | 验证数据是否为浮点数
ImageField | 验证文件是否为 Pillow 库可识别的图像格式
IntegerField | 验证数据是否为整数
GenericIPAddressField | 验证数据是否为有效 IP 地址
SlugField | 验证数据是否只包括字母、数字、下划线和连字符
TimeField | 验证数据是否为 datetime.time 或指定的特定时间格式的字符串
URLField | 验证数据是否为有效的 URL 地址

##### 表单字段通用参数
参数 | 说明
----- | -----
Required | 输入数据是否为空，默认值是空
Widget | 设置 HTML 控件的样式
Label | 用于生成 Label 标签或显示内容
Initial | 设置初始值
help_text | 设置帮助提示信息
error_messages | 设置错误信息，以字典格式表示
show_hidden_initial | 值为 True/False 是否在当前插件后再添加一个隐藏且具有默认值的插件（用于检验两次输入是否一致）
Validators | 自定义数据验证规则，以列表格式表示，列表元素为函数名
Localize | 值为 True/False 是否支持本地化
Disabled | 值为 True/False 是否可编辑
label_suffix | Label 内容的后缀，在 Label 后添加内容

#### django.forms.ModelForm 模型表单

* 添加模型外的表单字段是在模型已有字段下添加额外的表单字段
* 模型与表单设置是将模型的字段转换成表单字段，由 Meta 类的属性实现两者的字段转换
* 自定义表单字段 weight 的数据清洗函数只适用于字段 weight 的数据清洗

**Meta 类的属性及说明**

属性 | 说明
----- | -----
Model | 必要属性，用于绑定 Model 对象
Field | 必要属性，设置模型内哪些字段转换成表单字段。属性值为 '\_\_all\_\_' 代表整个模型的字段，挑选多个使用列表或元组
Exclude | 可选属性，与 Field 相反，禁用模型内那些字段转换成表单字段
Labels | 可选属性，设置表单字段里的参数 label 以字典形式表示
Widgets | 可选属性，设置表单字段里的参数 widget
field_classes | 可选属性，将模型字段重新定义成表单字段类型
help_texts | 可选属性，设置表单字段里的参数 help_text
error_messages | 可选属性，设置表单字段里的参数 error_messages

##### 自定义清洗函数

> 自定义表单字段的数据清洗函数名必须是 “clean_字段名” 的格式作为函数名，而且必须把清洗后的数据 return 返回。如果在函数中设置了 ValidationError 异常抛出，
那么函数可视为带有数据验证的清洗函数。在函数中使用 self.clean_data['字段名'] 获取数据。注意：在视图函数中使用时使用 clean_data 方法自动调用对用字段名的清洗函数


## Admin 后台系统

* 启动 Admin 后台系统应用 默认启动状态 名称 django.contrib.admin
* 创建超级用户，用于管理 执行 python manage.py createsuperuser 填写 username Email password
* 在浏览器输入 IP:port/admin 即可登录后台系统
* 对 Admin 后台系统的设置在各个应用中的 admin.py 文件中更改设置
* admin.site.site_title 和 admin.site.site_header 修改后台系统名称

**Admin 的常用基本属性**

属性 | 说明
----- | -----
list_display | 设置模型字段，用于 Admin 后台数据的表头显示
search_fields | 设置可搜索字段，在后台生成搜索框。如有外键应使用双下划线连接字段
list_fielder | 设置过滤器，在后台右侧生成导航栏。如有外键应使用双下划线连接字段
ordering | 设置排序方式
date_hierarchy | 设置字段的时间选择器，字段必须为时间格式才能使用
fields | 在添加数据时，设置可添加数据的字段
readonly_fields | 设置可读字段，在修改或新增数据时使其无法设置

#### Admin 的二次开发

**get_readonly_fields 函数 在 admin.py 中重写**

```实现不同用户角色决定字段的可读性
# 重写 get_readonly_fields 函数，设置超级用户和普通用户不同的权限
def get_readonly_fields(self, request, obj=None):
    # 判断是否是超级用户
    if request.user.is_superuser:
        self.readonly_fields = []
    return self.readonly_fields
```

**设置字段格式 在 models.py 中自定义函数**

```实现不同产品颜色不同
# 自定义函数，设置字体颜色。函数名自定义
def color_type(self):
    if '值' in self.字段名:
        color = '颜色'
    elif '值' in self.字段名:
        color = '颜色'
    else:
        color = '颜色'
    return format_html(
            '<span> style="color:{};">{}</span>',
            color,
            self.字段名,
        )

color_type.short_description = '带颜色的某字段'
```

在 admin.py 中的属性 list_display 中添加自定义字段 color_type

**get_queryset 函数 在 admin.py 中重写**

```实现不同用户角色设置数据的访问权限
# 重写 get_queryset 函数，根据当前用户名设置数据访问权限
def get_queryset(self, request):
    queryset = super().get_queryset(request)
    if request.user.is_superuser:
        return queryset
    else:
        return queryset.filter(条件)
```

**formfield_for_foreignkey 函数 在 admin.py 中重写**

```实现对外键可选值的过滤功能
# 新增或修改数据时，设置外键可选值
def formfield_for_foreignkey(self, db_field, request, **kwargs):
    # db_field 是模型的外键对象，一个模型可以有多个外键，所以需要先对外键名进行判断
    if db_field.name == '外键名':
        if not request.user.is_superuser:
            kwargs['queryset'] = 外键所对用的模型 Model.objects.filter(条件)
    return super(admin.ModelAdmin, self).formfield_for_foreignkey(db_field, request, **kwargs)
```

**save_model 函数 在 admin.py 中重写**

```实现对新增或修改时对数据的处理
# 修改保存方法
def save_model(self, request, obj, form, change):
    if change:
        # 获取当前用户名
        user = request.user
        # 使用模型获取数据， pk 代表默认主键字段
        name = self.model.object.get(pk=obj.pk).name
        # 使用表单获取数据
        data = form.clean_data['data']
        # 写入日志文件
        with open('路径', 'a') as file:
            file.write(数据)
    # 使用 super 可使自定义 save_model 既保留父类已有功能又可以添加自定义功能
    super().save_model(request, obj, form, change)
```

**自定义 Admin 模板**

Admin 后台系统的 HTML 模板是由 Django 提供的，可以在安装目录中找到 \Lib\site-packages\django\contrib\admin\templates\admin

利用模板继承方式修改后台模板

**在模板文件夹 templates 下创建 文件夹 admin 和 文件夹 App项目名**

* 文件夹 admin 代表该文件里的模板用于 Admin 后台管理系统，而且文件夹必须命名为 admin
* 文件夹 App项目名 代表项目的 App，文件夹命名必须与 App 的命名一致。文件夹中的继承模板，只适用于该应用自身
* 如果将继承模板放在 admin 文件夹下，说明文件将适用于当前项目的所有应用
* 自定义模板名必须和源模板名一致
* 源模板导入的标签在自定义模板中也需要导入
* 通过 block 标签实现源模板模板的代码重写。在源模板中，已经使用 block 标签将不同的功能进行了区分，可以直接对某个功能进行自定义开发

## Auth 认证系统

完善的用户管理系统 提供三大部分：用户信息、用户权限、和用户组 在数据库中分别对应数据表：auth_user、auth_permission 和 auth_group

在 setting.py 文件中启动应用 django.contrib.auth 和中间插件 django.contrib.auth.middleware.AuthenticationMiddleware 默认已经启用

#### 常用的方式为在项目下新建一个应用 user 在该应用里实现用户的各种操作 如：注册、登录、修改密码、注销等

* 在 views.py 中调用 django.contrib.auth.models 中的模型对 auth_user、auth_permission 和 auth_group 进行数据查询更改
* 验证账号密码是否正确需要调用 django.contrib.auth.authenticate(username=username, password=password) 函数
* 通过 authenticate 函数的返回值 authenticate.is_active 判断账号是否被激活
* 登录需要在验证正确后调用 django.contrib.auth.login(request, authenticate) 函数 authenticate 是 authenticate 函数的返回值
* 注册使用 User.objects.create_user(username=username, password=password) 插入数据
* 修改密码使用 authenticate 函数返回值 authenticate.set_password(password)
* set_password 是在 make_password 函数之上封装而成，make_password 可单独使用，主要用于加密数据 make_password(password, None, 加密算法)
* check_password 可以判断加密前后的数据是否一致 check_password(加密前数据, 加密后数据) 返回值为 True/False 若加密前数据经加密后为加密后数据则返回 True
* 注销调用 logout(request) 函数

#### 找回密码——邮件找回

在 setting.py 文件中配置一下信息启用邮箱
```启用邮箱配置
# 设置连接方式为 SSL
EMAIL_USER_SSL = True
# 设置邮件服务器域名/IP
EMAIL_HOST = '邮件服务器'
# 设置邮件服务器端口 SMTP 常用端口号为 465/587
EMAIL_PORT = 端口号
# 设置发送邮件的账号
EMAIL_HOST_USER = '邮箱账号'
# 设置 SMTP 服务授权码
EMAIL_HOST_PASSWORD = '授权码'
# 设置默认邮箱账号
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```

**邮件发送的三种方式**

1. send_mail 每次发送邮件都会建立一个新的连接，发送多封邮件则会建立多个连接
2. send_mass_mail 建立单个连接发送多封邮件，适合群发邮件
3. EmailMultiAlternatives 可以设置邮件正文内容为 HTML 格式，也可以添加附件，满足多方面开发需求

## 扩展 User 模型

1. 代理模型：这是一种模型继承，无须在数据库中创建新数据表，一般用于改变现有模型的行为方式，如增加新方法函数等，并不影响现有数据库结构。当不需要在数据库中存储额外信息，而需要增加操作方法或更改模型的查询管理方式时，适合使用代理模型扩展 User 模型
2. Profile 扩展模型 User：当存储的信息与模型 User 相关，而且并不改变模型 User 原有的认证方式时，课定义新的模型 MyUser，并设置某个字段为 OneToOneField，这样能与模型 User 形成一对一关联，该方法称为用户配置（User Profile）
3. AbstractBaseUser 扩展模型 User：当模型 User 的内置方法并不符合开发需求时，可使用该方法对模型 User 重新自定义设计，该方法对模型 User 和数据库架构影响很大
4. AbstractUser 扩展模型 User：如果模型 User 内置的方法符合开发需求，在不改变函数方法的情况下，添加模型 User 的额外字段，可通过使用 AbstractUser 定义的模型会替换原有的模型 User 方法实现

**一般新建一个 user 应用 专门用于用户管理。在其 models.py 中写如下代码进行扩展**

```
from django.db import models
from django.contrib.auth.models import AbstractUser

# AbstractUser 方式扩展 User 模型
class MyUser(AbstractUser):
    qq = models.CharField('QQ 号码', max_length=20)
    weChat = models.CharField('微信账号', max_length=50)
    mobile = models.CharField('手机号', max_length=11)

    # 设置返回值
    def __str__(self):
        return self.username
```

模型 MyUser 继承自 AbstractUser 类，AbstractUser 类已有内置模型 User 的字段属性，因此在数据迁移之前需要在 settings.py 中配置相关信息 配置信息：AUTH_USER_MODEL = 'user.MyUser' 该配置信息会将默认的内置模型 User 替换成 MyUser 模型，若没有配置该信息，则会在数据库中创建两个数据表 auth_user 和 user_myuser 默认使用 auth_user

#### 使用模型 user.MyUser 和内置模型 User 的区别

* 在启用 Admin 后台系统后，在认证和授权里用户已经被移除。需要手动将 MyUser 添加到后台


```
from django..contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.utils.translation import gettext_lazy as _

from .models import MyUser

@admin.register(MyUser)
class MyUserAdmin(UserAdmin):
    list_display = ['username', 'first_name', 'last_name', 'email', 'mobile', 'qq', 'weChat']
    # 将 UserAdmin.fieldsets 转化成列表
    fieldsets = list(UserAdmin.fieldsets)
    # 重写UserAdmin.fieldsets，在个人信息中添加 mobile、qq、weChat 等信息
    fieldsets[1] = (_('Personal info'), {'fields': ('first_name', 'last_name', 'email', 'mobile', 'qq', 'weChat')})
```

* User 模型定义的表单类，可以在 MyUser 模型中继承

表单类 | 表单字段 | 说明
----- | ----- | -----
UserCreationForm | username, password1, password2 | 创建新的用户的信息
UserChangeForm | password, 模型 User 的全部字段 | 修改已有的用户信息
AuthenticationForm | username, password | 用户登录时所触发的认证功能
PasswordResetForm | email | 将重置密码通过发送邮箱方式实现密码找回
SetPasswordForm | password1, password2 | 修改或新增用户密码，设置密码时，无须对旧密码验证
PasswordChangeForm | old_password, new_password1, new_password2 | 继承 SetPasswordForm，修改密码前需要对旧密码进行验证
AdminPasswordChangeForm | passwor1, password2 | 用于 Admin 后台修改用户密码

**继承样例：在 user 应用中新建 form.py**

```
from django.contrib.auth.forms import UserCreationForm

from .models import MyUser

class MyUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = MyUser
        # 在注册信息中添加邮箱、手机号、微信号和 QQ 号
        fields = UserCreationForm.Meta.fields + ('email', 'mobile', 'weChat', 'qq')
```

## 设置用户权限

