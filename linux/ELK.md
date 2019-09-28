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
