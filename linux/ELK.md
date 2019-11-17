# ELK

## Elasticsearch 

### Install

```
# 下载
$ curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.16.tar.gz

# 解压 启动
$ tar -xvf elasticsearch-5.6.16.tar.gz
$ cd elasticsearch-5.6.16/bin
$ ./elasticsearch 


curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
'

```

``` shell
$ curl 'http://localhost:9200/?pretty'
```
### 集群内的原理

集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成

#### 集群健康

``` shell
# 集群健康
curl -X GET "localhost:9200/_cluster/health?pretty"
```  

``` json
{
  "cluster_name": "elasticsearch",
  "status": "yellow", 
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 3,
  "active_shards": 3,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3, 
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50
}
```

`"status": "yellow"`: `主` 分 片都正常运行（集群可以正常服务所有请求），但是 `副本` 分片没有全部处在正常状态  


`"unassigned_shards": 3` : 没有被分配到任何节点的副本数


### 文档

`文档` : 指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

#### 文档元数据

`_index` 文档位置  
`_type` 对象类别  
`_id`  文档唯一标识；自动生成的 `ID` 是 `URL-safe`、 基于 `Base64` 编码且长度为20个字符的 `GUID` 字符串。 这些 `GUID` 字符串由可修改的 `FlakeID` 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。

文档API结构:  
`/{_index}/{_type}/{_id}`

##### 创建文档

创建新的，不覆盖原有的文档
``` shell

# op_type 查询
$ curl -X PUT /website/blog/123?op_type=create -d '{ ... }'

# 请求末端 /_create
$ curl -X PUT /website/blog/123/_create -d '{}'


# 新建文档有冲突时响应体
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```

##### 删除文档

```
curl -X DELETE "localhost:9200/website/blog/123?pretty"
``` 
##### 处理冲突

1. 乐观并发控制
```
# create
$ curl -X PUT "localhost:9200/website/blog/1/_create?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}

# get
$ curl -X GET "localhost:9200/website/blog/1?pretty"
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "_seq_no": 3,
  "_primary_term": 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}

# update with if_seq_no and if_primary_term
$ curl -X PUT "localhost:9200/website/blog/1?if_seq_no=3&if_primary_term=1&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
'

# update again conflict
$ curl -X PUT "localhost:9200/website/blog/1?if_seq_no=3&if_primary_term=1&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
'
{
   "error": {
      "root_cause": [
         {
            "type": "version_conflict_engine_exception",
            "reason": "[1]: version conflict, required seqNo [3], primary term [1], current document has seqNo[4] and primary term [1]",
            "index_uuid": "asdfasdfa",
            "shard": "3",
            "index": "website"
         }
      ],
      "type": "version_conflict_engine_exception",
      "reason": "[blog][1]: version conflict, current [2], provided [1]",
      "index": "website",
      "shard": "3"
   },
   "status": 409
}

```

2. 外部系统版本控制 `version_type=external`

```
# version_type 创建
$ curl -X PUT "localhost:9200/website/blog/2?version=5&version_type=external&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
'

# version_type 更新
$ curl -X PUT "localhost:9200/website/blog/2?version=10&version_type=external&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}
'
```

##### 文档部分更新

文档只能被替换，<b>不能被修改</b>

`update` API使用 <em>检索-修改-重建索引</em>

```
curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
'

```

1. 脚本部分更新

`_source` 字段内容修改，在脚本中使用 `ctx._source`

```
# _source 中 views 增加1
$ curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
   "script" : "ctx._source.views+=1"
}
'

# _source 中添加某个字段值
$ curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
    "script": {
        "source": "ctx._source.tags.add(params.new_tag)",
        "lang": "painless",
        "params": {
            "new_tag": "search"
        }
    }
}
'

# ctx.op 删除
$ curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source":"ctx.op = ctx._source.views == params.count ? \u0027delete\u0027 : \u0027none\u0027",
        "lang": "painless",
        "params" : {
            "count": 1
        }
    }
}
'
```

2. 更新可能不存在文档

``` shell
$ curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
'
```

3. 更新和冲突 `retry_on_conflict`

``` shell
$ curl -X POST "localhost:9200/website/blog/1/_update?retry_on_conflict=5&pretty" -H 'Content-Type: application/json' -d'
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 0
   }
}
'

```

##### `multi-get` 获取多个文档

#### 批量操作

最佳点 ：通过批量索引典型文档，并不断增加批量大小进行尝试。个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。

``` shell
{ action: { metadata }}\n # 文档操作 `create` `index` `update` `delete`
{ request body        }\n # 
{ action: { metadata }}\n
{ request body        }\n
...
```


#### 添加索引

索引实际上是指向一个或者多个物理 `分片` 的 `逻辑命名空间`;

一个 `分片` 是一个底层的 `工作单元` ，它仅保存了 全部数据中的一部分;一个分片是一个 Lucene 的实例

主分片数量在索引创建时确定；  
副本分片数量可以在运行时调整


### 分布式文档存储

#### 文档路由到分片中

公式: `shard = hash(routing) % number_of_primary_shards`

`routing` 默认为 `ID`， 可以自定义

<b>注意:</b>创建索引的时候就确定好主分片的数量 并且永远不会改变这个数量：因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

### 分析与分析器
过程:
1. 文本分词为倒序索引词条
2. 词条标准格式化，提高可搜索性

分词器:
1. 字符过滤器
2. 分词器
3. Token过滤器


``` shell
# 测试分析器
$ curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'
{
  "analyzer": "standard",
  "text": "Text to analyze"
}
'

```

#### 映射

索引中每个文档都有 <b>类型</b> 。每种类型都有它自己的 <b>映射</b> ，或者模式定义。映射定义了类型中的域，每个域的数据类型，以及Elasticsearch如何处理这些域。映射也用于配置与类型有关的元数据。

``` shell
# 查看映射
$ curl -X GET "localhost:9200/gb/_mapping/tweet?pretty"
```

映射只能新增，不可以修改  
更新一个映射来添加一个新域，但不能将一个存在的域从 `analyzed` 改为 `not_analyzed` 


```
curl -X PUT "localhost:9200/gb?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "tweet": {
      "properties": {
        "tweet": { "type": "string","analyzer": "english"},
        "date": { "type": "date" },
        "name": { "type": "string" },
        "user_id": {"type": "long"}
      }
    }
  }
}
'

```

#### 查询表达式

##### 查询语句结构

```
# 典型
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}

# 特定字段
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}

# 合并查询语句
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}

```

###### 验证查询 `validate-query`
``` shell
$ curl -X GET "localhost:9200/gb/tweet/_validate/query?explain&pretty" -H 'Content-Type: application/json' -d'
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
'

```

#### 一个字段多个映射

```
# 单映射
"tweet": {
    "type":     "string",
    "analyzer": "english"
}

# 多映射
"tweet": { 
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```

#### 相关性
评分的计算方式取决于查询类型 不同的查询语句用于不同的目的： `fuzzy` 查询会计算与关键词的拼写相似程度，`terms` 查询会计算 找到的内容与关键词组成部分匹配的百分比，但是通常我们说的 __relevance__ 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。

Elasticsearch 相似度算法 被定义为检索词频率/反向文档频率， _TF_/_IDF_:
1. __检索词频率__
2. __反向文档频率__  
3. __字段长度准则__

### 分布式检索

文档的唯一性由 `_index`, `_type`, 和 `routing` values （通常默认是该文档的 `_id` ）的组合来确定。

搜索被执行成一个两阶段过程，我们称之为 `query` then `fetch`;

数据量大可以考虑使用 `scroll`

### 索引

1. `number_of_shards` 每个索引的主分片数,创建后不可修改  
2. `number_of_replicas` 每个主分片的副本数，默认值是 `1`;可以随时修改  
3. `analysis`分析器

##### 自定义分析器

```
# 创建
$ curl -X PUT "localhost:9200/my_index?pretty" -H 'Content-Type: application/json' -d'
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
'

# 使用
$ curl -X PUT "localhost:9200/my_index/_mapping/my_type?pretty" -H 'Content-Type: application/json' -d'
{
    "properties": {
        "title": {
            "type":      "string",
            "analyzer":  "my_analyzer"
        }
    }
}
'

```

##### 索引别名重新索引
```
# alias
## 创建新索引
curl -X PUT "localhost:9200/my_index_v1?pretty"
## 设置别名
curl -X PUT "localhost:9200/my_index_v1/_alias/my_index?pretty"

# _aliases
## 重建索引并删除旧索引
$ curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
'
```

### 分片原理


## Logstash 

## Kibana

### Install

`/status` - Kibana服务器状态  

#### 动态映射

``` 
PUT .kibana
{
  "index.mapper.dynamic": true
}
```

### X-Pack 安全模块

java自签名证书文件格式: `PEM`

```
keytool -import -v -trustcacerts -alias IdenTrust -keypass yourpassword -file dst_root_ca_x3.pem -keystore cacerts.jks -storepass yourpassword
```

#### 多Elasticsearch 节点间负载均衡

1. Kibana机器安装Elasticsearch
2. 配置节点为协调节点(Coordinating only node).`elasticsearch.yml` 中，设置如下:  
``` yaml
node.master: false
node.data: false
node.ingest: false
```
3. 设置客户端节点接入Elasticsearch 集群。配置文件 `elasticsearch.yml` 配置如下:  
```yml
cluster.name: "my_cluster"
```
4. 检查 `elasticsearch.yml` 中的 `transport` 和 `HTTP` 主机配置： `network.host` 和 `transport.host` 。`transport.host` 需要为集群中其它成员网络可达， `network.host` 是访问 `Kibana` 的 `HTTP` 网络连接(默认为 localhost:9200 )。
``` yaml
network.host: localhost
http.port: 9200

# by default transport.host refers to network.host
transport.host: <external ip>
transport.tcp.port: 9300 - 9400
```
5. 请确认 `Kibana` 设置为指向本地客户端节点。在配置文件 `kibana.yml` 中，`elasticsearch.url` 应设为 `localhost:9200`
```yaml
# The Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://localhost:9200"
```

### 基础入门

``` shell
$ cd ~/kibana

$ curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json

$ curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json

$ curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl
 
```

`GET /_cat/indices?v` 查看数据加载是否成功

##### 创建索引模式  
Logstash 数据集会包含时间序列数据，所以点击 `Add New` 来定义这个数据集的索引，确保 `Index contains time-based events` 被选中，并从 `Time-field name` 下拉列表中选择 `@timestamp` 字段。
