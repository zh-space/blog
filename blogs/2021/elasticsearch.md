---
title: elasticsearch study
date: 2021-11-22 
tags:
 - elasticsearch
---
## 学习日志
## 下载elasticsearch
进入到elasticsearch的学习
首先在windows上安装elasticsearch，重要的一点是elasticsearch需要java环境才能运行，所以Java环境需要配置，且elasticsearch的版本何java的版本是有对应关系的，版本关系不对应是不能运行的，对应关系自行百度。
## elasticsearch.yml的配置
下载好elasticsearch之后我们要配置让他能让所有端口访问
打开config目录下的elasticsearch.yml文件添加代码
```yaml
http.host: 0.0.0.0
```
大功告成
## elasticsearch的运行内存配置
elasticsearch下载好之后点击bin目录下的elasticsearch.bat运行，因为elasticsearch默认占用内存是1G，所以运行elasticsearch之后你的电脑内存会被占满。
所以这个时候我们需要到配置文件中去配置elasticsearch的运行内存
在网上找了办法但是都没有很明确的解决方法，最后我自己在配置文件中找到了修改的关键在config目录下有一个jvm.options文件，用记事本打开往下翻有两行被注释了的代码
```yaml
#-Xms2g
#-Xmx2g
```
这个应该就是设置elasticsearch运行内存的代码
修改为：
```yaml
-Xms64m
-Xmx128m
```
我设置的是最小64M最大128M，你们可以根据自己的需要设置
## 安装elasticsearch可视化kibana
百度kibana下载即可，虽然只有300多M，但是就是解压需要很长时间
运行kibana需要先运行elasticsearch
## elasticsearch初步检索
### 1._cat
```
GET /_cat/nodes:  查看所有节点
GET /_cat/health:  查看es健康状况
GET /_cat/master:  查看主节点
GET /_cat/indices:  查看所有索引  show database; 
```
### 2.索引一个文档（保存）
保存一个数据，保存在那个索引下的那个类型下（mysql来说就是保存在那个数据库下的那一张表下），指定用那个唯一标识
```json
在customer索引下的external类型下保存1号数据为
PUT customer/external/1; 
{
    "name":"John Doe"
}
```
PUT和POST都可以，
POST新增。如果不指定id，会自动生成id，指定id就会修改这个数据，并新增版本号

PUT可以新增可以修改。PUT必须指定id；由于PUT需要指定id，我们一般用来做修改操作，不指定id会报错
### 3.查询文档
```
GET customer/external/1; 
```
查询在customer索引下的external类型下id为1的内容
![Image text](https://i.loli.net/2021/11/23/hKf1RxLPiCreV2F.png)
### 4.更新文档
```json
POST customer/external/1/_update  会对比原来的数据，如果没有改变就什么都不做，版本号和序列号也不发生改变
{
    "docs":{
        "name":"john Doew"
    }
}
```
或者
```json
POST customer/external/1  不会对比原来的数据，版本号和序列号会不断的更新
{
    "name":"john Doew2"
}
```
或者
```json
PUT customer/external/1  不会对比原来的数据，版本号和序列号会不断的更新
{
    "name":"john Doew3"
}
```
### 5.删除文档&索引  
没有类型删除
DELETE customer/external/1 删除文档
DELETE customer  删除索引
### 6.bulk批量API
```json
POST /customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"ZH"}
{"index":{"_id":"2"}}
{"name":"HL"}
```
批量插入每个都是独立的，一个插入失败不会影响其他的插入
![Image text](https://i.loli.net/2021/11/23/jW4qJlQuaEbhNSY.png)
### 7.查询
```json
GET /bank/account/_search?q=*&sort=account_number:asc
GET /bank/account/_search
{
  "query": {
    "match_all": {}
  },
  "sort":[
    {
      "account_number":"asc"
    },
    {
      "balance":"desc"
    }
    ]
}
GET /bank/_search
{
  "query": {
    "match_all": {}   
  },
  "sort": [    排序
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 5,  分页
  "size": 5, 
  "_source": ["age","gender"]   段返回指定字
}
GET /bank/_search
{
  "query": {
    "match": {      match查询条件，是分词查询 如果不想分词查询只想查固定的用match_phrase
      "age": "22"   22可以是数字(22)也可以是字符串("22")
    }
  }
}
多字段匹配
GET bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill Lopezo",
      "fields": ["address","city"]
    }
  }
}
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "gender": "m"
        }},
        {"match": {
          "address": "mill"
        }}
      ],
      "must_not": [
        {"match": {
          "age": "18"
        }}
      ],
      "should": [
        {"match": {
          "lastname": "Wallace"
        }}
      ],
      "filter": [
        {"range": {
          "age": {
            "gte": 18,
            "lte": 30
          }
        }}
      ]
    }
  }
}
文本字段用match查询，非文本字段用term查询
GET bank/_search
{
  "query": {
    "term": {
      "age": "28"
    }
  }
}
分组
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  }, 
  "aggs": {
    "ageAgg": {    
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg":{
      "avg": {
        "field": "balance"
      }
    }
  }
}
##搜索address中包含mill的所有人的年龄分布以及平均年龄
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  }, 
  "aggs": {
    "ageAgg": {    
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg":{
      "avg": {
        "field": "balance"
      }
    }
  },
  "size": 0
}
###按照年龄聚合，并且请求这些年俩年龄段的这些人的平均薪资
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "ageAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
##查出所有年龄分布，并且这些年龄段中M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 100
      },
      "aggs": {
        "genderAgg": {
          "terms": {
            "field": "gender.keyword",
            "size": 10
          },
          "aggs": {
            "balanceAvg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        "ageBalance":{
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
返回映射
GET bank/_mapping
PUT /my_index
{
  "mappings": {
    "properties": {
      "age":{"type":"integer"},
      "email":{"type":"keyword"},
      "name":{"type":"text"}
    }
  }
}
新增映射
PUT /my_index/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false        是否可被索引
    }
  }
}
想要更新索引只能数据迁移
GET /newbank/_mapping
PUT /newbank
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "email": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "text"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "state": {
        "type": "keyword"
      }
    }
  }
}
##数据迁移
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newbank"
  }
}
GET /newbank/_search
```
全文检索按照评分进行排序，会对检索条件进行分词匹配
更多请参照官方文档
