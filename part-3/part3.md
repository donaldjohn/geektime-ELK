# 集群身份认证与用户鉴权
- 如何为集群启用X-Pack Security
- 如何为内置用户设置密码
- 设置 Kibana与ElasticSearch通信鉴权
- 使用安全API创建对特定索引具有有限访问权限的用户

This tutorial involves a single node cluster, but if you had multiple nodes, you would enable Elasticsearch security features on every node in the cluster and configure Transport Layer Security (TLS) for internode-communication, which is beyond the scope of this tutorial. By enabling single-node discovery, we are postponing the configuration of TLS. For example, add the following setting:

discovery.type: single-node

## 课程demo
```
#启动单节点
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true

#使用Curl访问ES，或者浏览器访问 “localhost:9200/_cat/nodes?pretty”。返回401错误
curl 'localhost:9200/_cat/nodes?pretty'

#运行密码设定的命令，设置ES内置用户及其初始密码。
bin/elasticsearch-setup-passwords interactive

curl -u elastic 'localhost:9200/_cat/nodes?pretty'


# 修改 kibana.yml
elasticsearch.username: "kibana"
elasticsearch.password: "changeme"

#启动。使用用户名，elastic，密码elastic
./bin/kibana


POST orders/_bulk
{"index":{}}
{"product" : "1","price" : 18,"payment" : "master","card" : "9876543210123456","name" : "jack"}
{"index":{}}
{"product" : "2","price" : 99,"payment" : "visa","card" : "1234567890123456","name" : "bob"}


#create a new role named read_only_orders, that satisfies the following criteria:
#The role has no cluster privileges
#The role only has access to indices that match the pattern sales_record
#The index privileges are read, and view_index_metadata


#create sales_user that satisfies the following criteria:
# Use your own email address
# Assign the user to two roles: read_only_orders and kibana_user


#验证读权限,可以执行
POST orders/_search
{}

#验证写权限,报错
POST orders/_bulk
{"index":{}}
{"product" : "1","price" : 18,"payment" : "master","card" : "9876543210123456","name" : "jack"}
{"index":{}}
{"product" : "2","price" : 99,"payment" : "visa","card" : "1234567890123456","name" : "bob"}


```

## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/configuring-security.html
#
## 课程demo

```
# 生成证书
# 为您的Elasticearch集群创建一个证书颁发机构。例如，使用elasticsearch-certutil ca命令：
bin/elasticsearch-certutil ca

#为群集中的每个节点生成证书和私钥。例如，使用elasticsearch-certutil cert 命令：
bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

#将证书拷贝到 config/certs目录下
elastic-certificates.p12


bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.transport.ssl.truststore.path=certs/elastic-certificates.p12

bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E http.port=9201 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.transport.ssl.truststore.path=certs/elastic-certificates.p12


#不提供证书的节点，无法加入
bin/elasticsearch -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E http.port=9202 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate


```


```
## elasticsearch.yml 配置

#xpack.security.transport.ssl.enabled: true
#xpack.security.transport.ssl.verification_mode: certificate

#xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
#xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12


```
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls.html


```
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.http.ssl.truststore.path: certs/elastic-certificates.p12

```
```


# ES 启用 https
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12

```

```
#Kibana 连接 ES https



# 为kibana生成pem
openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -out elastic-ca.pem


elasticsearch.hosts: ["https://localhost:9200"]
elasticsearch.ssl.certificateAuthorities: [ "/Users/yiruan/geektime/kibana-7.1.0/config/certs/elastic-ca.pem" ]
elasticsearch.ssl.verificationMode: certificate



# 为 Kibna 配置 HTTPS
# 生成后解压，包含了instance.crt 和 instance.key
bin/elasticsearch-certutil ca --pem

server.ssl.enabled: true
server.ssl.certificate: config/certs/instance.crt
server.ssl.key: config/certs/instance.key


```


# 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls.html#tls-http
# 常见的集群部署方式
## 相关阅读
# Hot & Warm 架构与 Shard Filtering
## 课程代码
```
# 标记一个 Hot 节点
bin/elasticsearch  -E node.name=hotnode -E cluster.name=geektime -E path.data=hot_data -E node.attr.my_node_type=hot

# 标记一个 warm 节点
bin/elasticsearch  -E node.name=warmnode -E cluster.name=geektime -E path.data=warm_data -E node.attr.my_node_type=warm

# 查看节点
GET /_cat/nodeattrs?v

# 配置到 Hot节点
PUT logs-2019-06-27
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":0,
    "index.routing.allocation.require.my_node_type":"hot"
  }
}



PUT my_index1/_doc/1
{
  "key":"value"
}



GET _cat/shards?v


# 配置到 warm 节点
PUT PUT logs-2019-06-27/_settings
{  
  "index.routing.allocation.require.my_node_type":"warm"
}


# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1

# 标记一个 rack 2
bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack2

PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id"
  }
}

PUT my_index1
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1
  }
}

PUT my_index1/_doc/1
{
  "key":"value"
}


GET _cat/shards?v
DELETE my_index1/_doc/1



# Fore awareness
# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1

# 标记一个 rack 2
bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack1


PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id",
    "cluster.routing.allocation.awareness.force.my_rack_id.values": "rack1,rack2"
  }
}
GET _cluster/settings

# 集群黄色
GET _cluster/health

# 副本无法分配
GET _cat/shards?v


GET _cluster/allocation/explain?pretty
```
## 相关阅读
- https://www.elastic.co/cn/blog/sizing-hot-warm-architectures-for-logging-and-metrics-in-the-elasticsearch-service-on-elastic-cloud
- https://www.elastic.co/cn/blog/deploying-a-hot-warm-logging-cluster-on-the-elasticsearch-service
# 如何对集群进行容量规划
## 代码Demo

```
PUT logs_2019-06-27
PUT logs_2019-06-26


POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "logs_2019-06-27",
        "alias": "logs_write"
      }
    },
    {
      "remove": {
        "index": "logs_2019-06-26",
        "alias": "logs_write"
      }
    }
  ]
}


# POST /<logs-{now/d}/_search
POST /%3Clogs-%7Bnow%2Fd%7D%3E/_search

# POST /<logs-{now/w}/_search
POST /%3Clogs-%7Bnow%2Fw%7D%3E/_search

```
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/guide/current/capacity-planning.html
- https://yq.aliyun.com/articles/670118
# 分片设计及管理
##课程Demo
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/cluster-reroute.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/indices-forcemerge.html
- https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-total-shards.html
# 在私有云上管理与部署 Elasticsearch 集群
## 相关阅读
- https://www.elastic.co/cn/blog/introducing-elastic-cloud-on-kubernetes-the-elasticsearch-operator-and-beyond?elektra=products&storm=sub1
- https://www.elastic.co/blog/introducing-elastic-cloud-on-kubernetes-the-elasticsearch-operator-and-beyond
- https://github.com/operator-framework
- https://github.com/upmc-enterprises/elasticsearch-operator
# 在公有云上管理与部署 Elasticsearch 集群
## 相关阅读
- https://data.aliyun.com/product/elasticsearch
- https://www.elastic.co/cn/blog/elasticsearch-service-on-elastic-cloud-introduces-new-pricing-with-reduced-costs
# 生产环境查用配置与线上清单
## 相关阅读
# 监控 Elasticsearch 集群
## 课程demo
```
# Node Stats：
GET _nodes/stats

#Cluster Stats:
GET _cluster/stats

#Index Stats:
GET kibana_sample_data_ecommerce/_stats

#Pending Cluster Tasks API:
GET _cluster/pending_tasks

# 查看所有的 tasks，也支持 cancel task
GET _tasks


GET _nodes/thread_pool
GET _nodes/stats/thread_pool
GET _cat/thread_pool?v
GET _nodes/hot_threads
GET _nodes/stats/thread_pool


# 设置 Index Slowlogs
# the first 1000 characters of the doc's source will be logged
PUT my_index/_settings
{
  "index.indexing.slowlog":{
    "threshold.index":{
      "warn":"10s",
      "info": "4s",
      "debug":"2s",
      "trace":"0s"
    },
    "level":"trace",
    "source":1000  
  }
}

# 设置查询
DELETE my_index
//"0" logs all queries
PUT my_index/
{
  "settings": {
    "index.search.slowlog.threshold": {
      "query.warn": "10s",
      "query.info": "3s",
      "query.debug": "2s",
      "query.trace": "0s",
      "fetch.warn": "1s",
      "fetch.info": "600ms",
      "fetch.debug": "400ms",
      "fetch.trace": "0s"
    }
  }
}

GET my_index


```
## 相关阅读
# 诊断集群的潜在问题
## 相关阅读
- https://elasticsearch.cn/slides/162
- https://yq.aliyun.com/articles/657712
- https://yq.aliyun.com/articles/657108
- https://help.aliyun.com/document_detail/90391.html
# 集群健康与问题排查
## 课程demo
```
#案例1
DELETE mytest
PUT mytest
{
  "settings":{
    "number_of_shards":3,
    "number_of_replicas":0,
    "index.routing.allocation.require.box_type":"hott"
  }
}





# 检查集群状态，查看是否有节点丢失，有多少分片无法分配
GET /_cluster/health/

# 查看索引级别,找到红色的索引
GET /_cluster/health?level=indices


#查看索引的分片
GET _cluster/health?level=shards

# Explain 变红的原因
GET /_cluster/allocation/explain

GET /_cat/shards/mytest
GET _cat/nodeattrs

DELETE mytest
GET /_cluster/health/

PUT mytest
{
  "settings":{
    "number_of_shards":3,
    "number_of_replicas":0,
    "index.routing.allocation.require.box_type":"hot"
  }
}

GET /_cluster/health/

#案例2, Explain 看 hot 上的 explain
DELETE mytest
PUT mytest
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1,
    "index.routing.allocation.require.box_type":"hot"
  }
}

GET _cluster/health
GET _cat/shards/mytest
GET /_cluster/allocation/explain

PUT mytest/_settings
{
    "number_of_replicas": 0
}

```
## 相关阅读
# 提升集群写性能
## 课程demo
```
DELETE myindex
PUT myindex
{
  "settings": {
    "index": {
      "refresh_interval": "30s",
      "number_of_shards": "2"
    },
    "routing": {
      "allocation": {
        "total_shards_per_node": "3"
      }
    },
    "translog": {
      "sync_interval": "30s",
      "durability": "async"
    },
    "number_of_replicas": 0
  },
  "mappings": {
    "dynamic": false,
    "properties": {}
  }
}
```
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html
# 提升集群读性能
## 课程demo
```
PUT blogs/_doc/1
{
  "title":"elasticsearch"
}
GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "title": "elasticsearch"
        }}
      ],
      
      "filter": {
        "script": {
          "script": {
            "source": "doc['title.keyword'].value.length()>5"
          }
        }
      }
    }
  }
}


GET blogs/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "elasticsearch"}},
        {
          "range": {
            "publish_date": {
              "gte": 2017,
              "lte": 2019
            }
          }
        }
      ]
    }
  }
}
```
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html
# 集群压力测试
## 课程demo
```
pip3 install esrally

esrally configure

# 只测试 1000条数据
esrally --distribution-version=7.1.0 --test-mode

# 测试完整数据
esrally --distribution-version=7.1.0

```
# 相关阅读
- https://github.com/elastic/rally
- https://github.com/elastic/rally-tracks
- https://logz.io/blog/rally/
# 缓存及使用Circuit Breaker限制内存使用
## 课程demo
```
GET _cat/nodes?v

GET _nodes/stats/indices?pretty

GET _cat/nodes?v&h=name,queryCacheMemory,queryCacheEvictions,requestCacheMemory,requestCacheHitCount,request_cache.miss_count

GET _cat/nodes?h=name,port,segments.memory,segments.index_writer_memory,fielddata.memory_size,query_cache.memory_size,request_cache.memory_size&v


PUT /_cluster/settings
{
    "persistent" : {
       "indices.breaker.request.limit" : "90%"
    }
}

```
## 补充阅读
- https://www.elastic.co/blog/improving-node-resiliency-with-the-real-memory-circuit-breaker
# 一些运维相关的建议
## 课程demo
```
# 移动一个分片从一个节点到另外一个节点

POST _cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "index_name",
        "shard": 0,
        "from_node": "node_name_1",
        "to_node": "node_name_2"
      }
    }
  ]
}


# Fore the allocation of an unassinged shard with a reason

POST _cluster/reroute?explain
{
  "commands": [
    {
      "allocate": {
        "index": "index_name",
        "shard": 0,
        "node": "nodename"
      }
    }
  ]
}


# remove the nodes from cluster 
PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.exclude._ip":"the_IP_of_your_node"
  }
}

# Force a synced flush
POST _flush/synced


# change the number of moving shards to balance the cluster
PUT /_cluster/settings
{
  "transient": {"cluster.routing.allocation.cluster_concurrent_rebalance":2}
}

# change the number of shards being recovered simultanceously per node
PUT _cluster/settings
{
  "transient": {"cluster.routing.allocation.node_concurrent_recoveries":5}
}

# Change the recovery speed
PUT /_cluster/settings
{
  "transient": {"indices.recovery.max_bytes_per_sec": "80mb"}
}

# Change the number of concurrent streams for a recovery on a single node
PUT _cluster/settings
{
  "transient": {"indices.recovery.concurrent_streams":6}
}


# Change the sinze of the search queue
PUT _cluster/settings
{
  "transient": {
    "threadpool.search.queue_size":2000
  }
}

# Clear the cache on a node
POST _cache/clear


#Adjust the circuit breakers
PUT _cluster/settings
{
  "persistent": {
    "indices.breaker.total.limit":"40%"
  }
}


```# 使用 shrink与rolloverAPI有效的管理索引
#课程demo
```


# 打开关闭索引
DELETE test
#查看索引是否存在
HEAD test

PUT test/_doc/1
{
  "key":"value"
}

#关闭索引
POST /test/_close
#索引存在
HEAD test
# 无法查询
POST test/_count

#打开索引
POST /test/_open
POST test/_search
{
  "query": {
    "match_all": {}
  }
}
POST test/_count


# 在一个 hot-warm-cold的集群上进行测试
GET _cat/nodes
GET _cat/nodeattrs

DELETE my_source_index
DELETE my_target_index
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

# 分片数3，会失败
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 3,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}



# 报错，因为没有置成 readonly
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

#将 my_source_index 设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}

# 报错，必须都在一个节点
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

DELETE my_source_index
## 确保分片都在 hot
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0,
   "index.routing.allocation.include.box_type":"hot"
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}


GET _cat/shards/my_target_index

# My target_index状态为也只读
PUT my_target_index/_doc/1
{
  "key":"value"
}



# Split Index
DELETE my_source_index
DELETE my_target_index

PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

# 必须是倍数
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 10
  }
}

# 必须是只读
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 8
  }
}


#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


POST my_source_index/_split/my_target_index
{
  "settings": {
    "index.number_of_shards": 8,
    "index.number_of_replicas":0
  }
}

GET _cat/shards/my_target_index



# write block
PUT my_target_index/_doc/1
{
  "key":"value"
}



#Rollover API
DELETE nginx-logs*
# 不设定 is_write_true
# 名字符合命名规范
PUT /nginx-logs-000001
{
  "aliases": {
    "nginx_logs_write": {}
  }
}

# 多次写入文档
POST nginx_logs_write/_doc
{
  "log":"something"
}


POST /nginx_logs_write/_rollover
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  5,
    "max_size":  "5gb"
  }
}

GET /nginx_logs_write/_count
# 查看 Alias信息
GET /nginx_logs_write


DELETE apache-logs*


# 设置 is_write_index
PUT apache-logs1
{
  "aliases": {
    "apache_logs": {
      "is_write_index":true
    }
  }
}
POST apache_logs/_count

POST apache_logs/_doc
{
  "key":"value"
}

# 需要指定 target 的名字
POST /apache_logs/_rollover/apache-logs8xxxx
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  1,
    "max_size":  "5gb"
  }
}


# 查看 Alias信息
GET /apache_logs


```
## 相关阅读
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/indices-shrink-index.html
- https://www.elastic.co/guide/en/elasticsearch/reference/7.1/indices-rollover-index.html
# 索引全生命周期管理及工具介绍
## 课程demo
```

# 运行三个节点，分片 将box_type设置成 hot，warm和cold
# 具体参考 github下，docker-hot-warm-cold 下的docker-compose 文件



DELETE *



# 设置 1秒刷新1次，生产环境10分种刷新一次
PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval":"1s"
  }
}

# 设置 Policy
PUT /_ilm/policy/log_ilm_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 5
          }
        }
      },
      "warm": {
        "min_age": "10s",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "warm"
            }
          }
        }
      },
      "cold": {
        "min_age": "15s",
        "actions": {
          "allocate": {
            "include": {
              "box_type": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "20s",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}



# 设置索引模版
PUT /_template/log_ilm_template
{
  "index_patterns" : [
      "ilm_index-*"
    ],
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "log_ilm_policy",
          "rollover_alias" : "ilm_alias"
        },
        "routing" : {
          "allocation" : {
            "include" : {
              "box_type" : "hot"
            }
          }
        },
        "number_of_shards" : "1",
        "number_of_replicas" : "0"
      }
    },
    "mappings" : { },
    "aliases" : { }
}



#创建索引
PUT ilm_index-000001
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "index.lifecycle.name": "log_ilm_policy",
    "index.lifecycle.rollover_alias": "ilm_alias",
    "index.routing.allocation.include.box_type":"hot"
  },
  "aliases": {
    "ilm_alias": {
      "is_write_index": true
    }
  }
}

# 对 Alias写入文档
POST  ilm_alias/_doc
{
  "dfd":"dfdsf"
}

```
## 相关阅读
## 课程demo
```
{
"template": "logs-*",
"settings": {
"index.indexing.slowlog.threshold.index.debug": "2s",
"index.indexing.slowlog.threshold.index.info": "5s",
"index.indexing.slowlog.threshold.index.trace": "500ms",
"index.indexing.slowlog.threshold.index.warn": "10s",
"index.merge.policy.max_merged_segment": "2gb",
"index.merge.policy.segments_per_tier": "24",
"index.number_of_replicas": "1",
"index.number_of_shards": "12",
"index.optimize_auto_generated_id": "true",
"index.refresh_interval": "600s",
"index.routing.allocation.total_shards_per_node": "-1",
"index.search.slowlog.threshold.fetch.debug": "500ms",
"index.search.slowlog.threshold.fetch.info": "800ms",
"index.search.slowlog.threshold.fetch.trace": "200ms",
"index.search.slowlog.threshold.fetch.warn": "1s",
"index.search.slowlog.threshold.query.debug": "2s",
"index.search.slowlog.threshold.query.info": "5s",
"index.search.slowlog.threshold.query.trace": "500ms",
"index.search.slowlog.threshold.query.warn": "10s",
"index.translog.durability": "async",
"index.translog.flush_threshold_size": "5000mb",
"index.translog.sync_interval": "120s",
"index.unassigned.node_left.delayed_timeout": "7200m"
},
"mappings": {
"_default_": {
"_all": {
"store": "false"
}
},
"typename": {
"dynamic": false,
"properties": {
"full_name": {
"type": "text"
}
}
}
}
}
```
