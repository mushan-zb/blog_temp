## 概述
对于 python web 程序一般会分为两个部分：服务程序和应用程序
* 服务程序负责对 socket 服务器进行封装，在请求到来时对请求的数据进行整理
* 应用程序负责具体的逻辑处理。web 框架是为了方便应用程序开发而设计的

python 服务器网关接口（python web server gateway interface 缩写 WSGI）是 python 应用程序或者框架和 web 服务器之间的一种接口。WSGI 并没有官方实现，而是更像一种协议，只要遵照这些协议就可以在任何服务器上运行

## Django 请求响应的过程
1. 浏览器用户使用 url 发出一个请求
    * WEB 应用启动时会生成一个 WSGIHandler 实例
    * 该实例会在第一次请求时调用 load_middleware
    * 在执行 WSGIHandler 时会按照 settings.py 中的 MIDDLEWARE_CLASSES 顺序加载中间件
    * 一个 middleware 包括请求响应的四个阶段 request、view、response、exception，对应方法：process_request、process_view、process_response、process_exception
2. 构造 WSGIRequest
    * 根据 HTTP 请求构造 WSGIRquest 请求
3. request 的 Middleware 中间件进行处理
    * 对 request 请求进行预处理，此处可以限制爬虫、特定信息访问失效，返回 response 响应
4. URLConf 通过 url.py 文件将 request 请求的 URL 传递给相应的视图
    * 该过程会创建 django.core.urlresolvers.RegexURLResolver 的一个实例
    * ROOT_URLCONF 的配置产生的每一个在 urlpatterns 列表中
    * 如果没有找到匹配的条目，resolver 会产生 django.core.urlresolvers.Resolver404 异常，它是 django.http.Http404 子类。返回 response 响应
5. 调用 View 视图中的相应方式处理 request 请求
    * 处理 request 请求的内容，部分数据向 Model 模型请求
    * 将处理好的数据返回
    * 非必需：将数据返回给 Template 进行渲染
6. Model 模型将 View 请求的数据进行查询并返回
    * 对象关系映射(ORM)定义数据字段，连接数据库，进行创建、增删改查
7. Template 模板渲染输出
    * 渲染后的 HTML 页面返回给 View 视图，返回 response 响应
8. HTTPResponse 被发送到 Response Middlewares 进行处理后返回给浏览器用户
    * Response Middleware 可以对返回的 HTTPResponse 进行处理
