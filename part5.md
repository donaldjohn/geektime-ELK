# 集群Backup & Restore
## 课程demo
```

#在 elasticsearch.yml 加入相关的配置
path.repo: ["/Users/yiruan/geektime/mount/my_backup"]

#创建一个 repositoty
PUT /_snapshot/my_fs_backup
{
    "type": "fs",
    "settings": {
        "location": "/Users/yiruan/geektime/mount/my_backup",
        "compress": true
    }
}

# 创建一个snapshot
PUT /_snapshot/my_fs_backup/snapshot_1?wait_for_completion=true

DELETE test
PUT test/_doc/1
{
  "key":"value1"
}


#指定索引创建快照
PUT /_snapshot/my_fs_backup/snapshot_2?wait_for_completion=true
{
  "indices": "test",
  "ignore_unavailable": true,
  "include_global_state": false,
  "metadata": {
    "taken_by": "yiming",
    "taken_because": "backup before upgrading"
  }
}

#查看所有的快照
GET /_snapshot/my_fs_backup/_all

# 删除快照
DELETE /_snapshot/my_fs_backup/snapshot_2


POST /_snapshot/my_fs_backup/snapshot_1/_restore
{
  
}

# 指定索引进行 restore
POST /_snapshot/my_fs_backup/snapshot_1/_restore
{
  "indices": "test",
  "index_settings": {
    "index.number_of_replicas": 5
  },
  "ignore_index_settings": [
    "index.refresh_interval"
  ]
}

DELETE test

# 删除快照
DELETE /_snapshot/my_fs_backup



```# 用 Java 和 Elasticsearch 开发应用
# 课程demo
```
```
## 相关资料
- https://spring.io/projects/spring-data-elasticsearch#overview
# 电影搜索服务# 电影搜索服务
## 课程demo
```
安装配置
app-search.yml:
allow_es_settings_modification: true

#Allow Elasticsearch to create indexes automatically: Add the following line #within either Elasticsearch cluster settings or elasticsearch.yml:

action.auto_create_index: ".app-search-*-logs-*,-.app-search-*,+*"




准备数据，使用python 2.7
APP_SEARCH_NAME=tmdb APP_SEARCH_PWD=private-dtcda1pdruoq2hvwqe8rhz1x python ./ingest_tmdb_to_appserarch.py

# 使用最新版本的 node
nvm list
nvm use default

```

## 补充阅读
- APP Search Release - https://www.elastic.co/blog/elastic-app-search-is-now-generally-available
- Search UI - https://www.elastic.co/blog/search-ui-1-0-0-released?baymax=web&elektra=search-ui-webinar
# stackoverflow 用户调查问卷分析
## 课程demo
```
sudo bin/logstash -f ./logstash-stackoverflow-survey.conf


PUT final-stackoverflow-survey
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  },
  "settings": {
    "number_of_replicas": 0
  }
}



数字相关
YearsCode
WorkWeekHrs
Age
Age1stCode 16
YearsCodePro



PUT _ingest/pipeline/stackoverflow_pipeline
{
  "description": "Pipeline for stackoverflow survey",
  "processors": [
    {
      "split": {
        "field": "DatabaseDesireNextYear",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "DatabaseWorkedWith",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "DevEnviron",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "LanguageWorkedWith",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "MiscTechDesireNextYear",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "PlatformWorkedWith",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "PlatformDesireNextYear",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "WebFrameWorkedWith",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "WebFrameDesireNextYear",
        "separator": ";"
      }
    },
    {
      "split": {
        "field": "Containers",
        "separator": ";"
      }
    },
    {
      "script": {
        "source": """
          try{
	          ctx.YearsCode = Integer.parseInt(ctx.YearsCode);
          	}catch(Exception e){
          		ctx.YearsCode = 0;
          	}
"""
      }
    },
    {
      "script": {
        "source": """
          try{
	          ctx.WorkWeekHrs = Integer.parseInt(ctx.WorkWeekHrs);
          	}catch(Exception e){
          		ctx.WorkWeekHrs = 0;
          	}
"""
      }
    },
    {
      "script": {
        "source": """
          try{
	          ctx.Age = Integer.parseInt(ctx.Age);
          	}catch(Exception e){
          		ctx.Age = 0;
          	}
"""
      }
    },
    {
      "script": {
        "source": """
          try{
	          ctx.Age1stCode = Integer.parseInt(ctx.Age1stCode);
          	}catch(Exception e){
          		ctx.Age1stCode = 0;
          	}
"""
      }
    },
    {
      "script": {
        "source": """
          try{
	          ctx.YearsCodePro = Integer.parseInt(ctx.YearsCodePro);
          	}catch(Exception e){
          		ctx.YearsCodePro = 0;
          	}
"""
      }
    }
  ]
}



POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "stackoverflow-survey-raw"
  },
  "dest": {
    "index": "final-stackoverflow-survey",
    "pipeline": "stackoverflow_pipeline"
  }
}

GET final-stackoverflow-survey/_mapping

```
## 参考链接
http://stackoverflow.com/research/ 
