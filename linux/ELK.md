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

```shell
curl -X GET "localhost:9200/_cluster/health?pretty"
```

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
