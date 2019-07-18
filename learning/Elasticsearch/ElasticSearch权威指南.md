# ElasticSearch 权威指南 - 中文版
**基于 Elasticsearch 2.x 版本**

## 搜索 - 请求头参数

### 空搜索
GET /_search - 没有指定任何查询的空搜索，返回集群所有索引下的所有文档

- hits：包含 total 字段来表示匹配到的文档总数，并且一个 hits 数组包含所查询结果的前十个文档
  - hits 数组：每个结果包含文档的 _index、_type、_id、_source 字段。source 字段包含整个文档内容
  - _score：每个结果都有一个 _score，衡量文档与查询的匹配程度。默认情况下，返回的文档按照 _score 降序排列
  - max_score：与查询匹配文档的 _score 的最大值
- took：执行整个搜索耗费了多少毫秒
- shareds：_shareds 告诉我们查询中参与的分片总数，以及这些分片成功了多少，失败了多少

GET /_search?timeout=10ms - timeout 设置查询超时时间
- timed_out：告诉我们查询是否超时。默认情况下，搜索请求不会超时。但如果低响应时间比完成查询结果更重要，可以指定 timeout 的值（默认：ms，可带单位：1ms、1s）

### 多索引、多类型
在一个或多个特殊的索引并且在一个或多个特殊的类型中进行索引。可以通过在 URL 中指定特殊的索引和类型达到这种效果

- /_search - 在所有索引中搜索所有的类型
- /xx/_search - 在 xx 索引中搜索所有文档
- /xx,yy/_search - 在 xx 和 yy 索引中搜索所有文档
- /x*,y*/_search - 在任何以 x 或 y 开头的索引中搜索所有文档
- /xx/aa/_search - 在 xx 索引中搜索 aa 类型
- /xx/aa,bb/_search - 在 xx 索引中搜索 aa 和 bb 类型
- /_all/aa,bb/_search - 在所有索引中搜索 aa 和 bb 类型

### 分页
和 SQL 使用 LIMIT 关键字返回单个 page 结果的方法相同，ElasticSearch 接受 from 和 size 参数

- size：显示应该返回的结果数量，默认是 10
- from：显示应该跳过的初始结果数量，默认是 0

例如：每页展示 5 条结果，可以用以下方式请求得到 1 到 3 页的结果  
1. GET /_search?size=5
2. GET /_search?size=5&from=5
3. GET /_search?size=5&from=10

**在分布式系统中深度分页**  
假如每页 10 条数据，我们请求第 1000 页，即：10001 - 10010 条数据。除所有分片都不得不产生前 10010 个结果以外，协调节点还要对全部汇总的结果进行排序，最后丢掉除 10001 - 10010 这 10 条数据以外的所有数据。在分布式系统中对结果排序的成本随分页深度成指数上升

### 带参数搜索
因查询字符串参数需要进行百分比编码（URL 编码）所以不推荐添加复杂查询参数，后面会详细讲解使用请求体查询

- _all 字段  
GET /_search?q=parameter - 返回包含 parameter 的所有文档  
当索引一个文档的时候，Elasticsearch 会取出所有字段拼接成一个大的字符串，作为 _all 字段进行索引  
**注：除非设置特定字段，否则查询字符串就使用 _all 字段进行搜索**

## 查询 - 请求体参数

### 空查询
返回所有索引的所有文档
```
GET /_search
{}
```

使用 from 和 size 参数分页
```
GET /_search
{
    "from": 30,
    "size": 10
}
```

**注：**  某些特定语言（如：JavaScript）的 HTTP 库是不允许 GET 请求带有请求体。而在 [RFC 7231](https://tools.ietf.org/html/rfc7231#page-24) - 一个专门负责处理 HTTP 语义和内容的文档。并没有规定一个带有请求体的 GET 请求应该如何处理。导致结果：有一些 HTTP 服务器允许，而有一些特别是用于缓存和代理的服务器则不允许

### 查询表达式
查询表达式(Query DSL)是一种非常灵活又富有表现力的 查询语言。 Elasticsearch 使用它可以以简单的 JSON 接口来展现 Lucene 功能的绝大部分。在你的应用中，你应该用它来编写你的查询语句。它可以使你的查询语句更灵活、更精确、易读和易调试

``` 查询语句传递给 query 参数
GET /_search
{
    "query": YOUR_QUERY_HERE
}
```

空查询
``` 带查询表达式的空查询
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

查询语句的结构
```
// 经典结构
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE, ...
    }
}

// 针对某字段
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE, ...
        }
    }
}
```

完整的查询请求 - 举例
```
GET /_search
{
    "query": {
        "match": {
            "argument": "value"
        }
    }
}

// 合并查询语句
GET /_search
{
    "bool": {
        "must": {"match": {"argument": "value"}},
        "must_not": {"match": {"argument": "value"}},
        "should": {"match": {"argument": "value"}},
        "filter": {"range": {"argument": {"gt": "number"}}}
    }
}
```

### 查询与过滤
Elasticsearch 使用的查询语言（DSL） 拥有一套查询组件，这些组件可以以无限组合的方式进行搭配。这套组件可以在以下两种情况下使用：过滤情况（filtering context）和查询情况（query context）

- filter context: 查询被设置成一个不评分或者过滤查询，结果只是简单的检查包含或者排除
- query context: 查询被设置成评分查询，结果会计算文档是否匹配、匹配度如何

### 常用查询介绍
- match_all: 查询匹配所有文档 - `{"match_all": {}}`
- match: 查询在任何字段上进行查询匹配 - `{"match": {"argument": "value"}}`
- multi_match: 在多字段上执行相同的 match 查询 - 
```
{
    "multi_match": {
        "filter": ["argument1", "argument2"...],
        "query": "text"
    }
}
```
- range: 查询在指定区间的数字或时间（判断符号：gt gte lt lte） - 
```
{
    "range": {
        "age": {
            "gte": number,
            "lt": number
        }
    }
}
```
- term: 用于精确值匹配，精确值包括：数字、时间、布尔值、not_analyzed 的字符串 - `{"term": {"argument": number}}`
- terms: 和 term 相同，但允许多值匹配，命中其中一个值即可 - `{"terms":"argument":["text1", "text2"...]}`
- exists: 

## 映射和分析



