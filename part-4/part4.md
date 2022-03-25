# Logstash 入门及架构介绍
## 课程demo
```


# 一个 Demo， demo 运行
sudo bin/logstash -f logstash-filter.conf

# demo数据
127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] "GET /xampp/status.php HTTP/1.1" 200 3891 "http://cadenza/xampp/navi.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0"


# codec demo


sudo bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=> rubydebug}}"
sudo bin/logstash -e "input{stdin{codec=>json}}output{stdout{codec=> rubydebug}}"
sudo bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=> dots}}"


sudo bin/logstash -f multiline-exception.conf

# 多行数据，异常
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)



# 一个实例
https://github.com/onebirdrocks/geektime-ELK/blob/master/part-1/2.4-Logstash%E5%AE%89%E8%A3%85%E4%B8%8E%E5%AF%BC%E5%85%A5%E6%95%B0%E6%8D%AE/movielens/logstash.conf

```
# Logstash插件及文档介绍
##课程demo
```
https://spring.io/guides/gs/accessing-data-mysql/
create database db_example;
use db_example;
show tables;
drop table user;
select * from user;



# 新增用户
curl localhost:8080/demo/add -d name=Mike -d email=mike@xyz.com -d tags=Elasticsearch,IntelliJ
curl localhost:8080/demo/add -d name=Jack -d email=jack@xyz.com -d tags=Mysql,IntelliJ
curl localhost:8080/demo/add -d name=Bob -d email=bob@xyz.com -d tags=Mysql,IntelliJ

#查看所有的用户
curl 'localhost:8080/demo/all'

# 更新用户
curl -X PUT localhost:8080/demo/update -d id=16 -d name=Bob2 -d email=bob2@xyz.com -d tags=Mysql,IntelliJ

# 删除用户
curl -X DELETE localhost:8080/demo/delete -d id=15



mysql-demo.conf

# 创建 alias，只显示没有被标记 deleted的用户
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "users",
        "alias": "view_users",
         "filter" : { "term" : { "is_deleted" : false } }
      }
    }
  ]
}

# 通过 Alias查询，查不到被标记成 deleted的用户
POST view_users/_search
{

}


POST view_users/_search
{
  "query": {
    "term": {
      "name.keyword": {
        "value": "Jack"
      }
    }
  }
}

POST users/_search
{
  "query": {
    "term": {
      "name.keyword": {
        "value": "Jack"
      }
    }
  }
}

```
# Beats介绍
## 课程demo
```

##
# 查看 packetbeat 模块
# 设置 packetbeat 的mysql 模块
# 启动运行
#
./metricbeat modules list
./metricbeat modules enable mysql
./metricbeat setup --dashboards

# 安装mysql
create database db_example
use db_example;
show tables;
select * from user

curl localhost:8080/demo/add -d name=Mike -d email=mike@xyz.com -d tags=Elasticsearch,IntelliJ
curl localhost:8080/demo/add -d name=Jack -d email=jack@xyz.com -d tags=Mysql,IntelliJ
curl localhost:8080/demo/add -d name=Bob -d email=bob@xyz.com -d tags=Mysql,IntelliJ

curl 'localhost:8080/demo/all'


# 配置 packetbeat
# 启动
修改 packetbeat，打开 http 5601 9200 和 mysql 3306监控

sudo chown root packetbeat.yml
sudo ./packetbeat setup --dashboards
sudo ./packetbeat


# 查看所有 Filebeat 模块
# 查看所有的modules
./filebeat modules list

#
./filebeat modules enable mysql

```

## 课程demo
```

localhost:5601/status




PUT /logstash-2015.05.18
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}



PUT /logstash-2015.05.19
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}


PUT /logstash-2015.05.20
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}

# For Mac & Windows
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl

curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json


#For Windows
Invoke-RestMethod "http://localhost:9200/_bulk?pretty" -Method Post -ContentType 'application/x-ndjson' -InFile "logs.jsonl"



GET /_cat/indices?v

```
# 使用Kibana Discovery 探索数据
## 课程demo
```

设置时间过滤器
搜索你的数据
根据字段进行过滤
查看文档数据
查看文档上下文
暂时字段数据统计
Save Query


```
# 基本可视化组件介绍
# 课程demo
```

PUT /shakespeare
{
  "mappings": {
    "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
    }
  }
}


# For Mac & Windows
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json

#For Windows
Invoke-RestMethod "http://localhost:9200/bank/account/_bulk?pretty" -Method Post -ContentType 'application/x-ndjson' -InFile "accounts.json"
Invoke-RestMethod "http://localhost:9200/shakespeare/_bulk?pretty" -Method Post -ContentType 'application/x-ndjson' -InFile "shakespeare.json"


```
# 构建Dashboard
## 课程demo
```
- 创建仪表潘
- 加载仪表盘
- 共享仪表盘

```
# 用 ELK 来做日志管理
## 课程demo
```
./filebeat modules list
./filebeat modules enable system
./filebeat modules enable elasticsearch


## 进 modules.d 编辑相应的文件，修改log路径

./filebeat setup –dashboards

./filebeat export template | more

./filebeat -e

```POST elasticoffee/_search
{
  "size": 0, 
  "aggs": {
    "by": {
      "terms": {
        "field": "beverage.keyword",
        "size": 10
      }
    }
  }
}