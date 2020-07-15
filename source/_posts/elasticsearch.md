---
title: Elasticsearch基本操作
date: 2019-03-11 10:12:41
tags: [elasticsearch,搜索引擎]
---

### 基本操作elasticsearch v6.8.7

```
创建索引
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

获取索引数据
```
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

批量创建索引 5MB~15MB, 1,000~5,000条记录为宜
下载[accounts.json](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true) 文件，
```
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}


curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
```
查看索引索引情况
```
curl "localhost:9200/_cat/indices?v"
```

搜索
```
curl -X POST "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
'
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```

查看索引mapping情况（索引中各字段的映射定义）
```
curl -X GET "localhost:9200/bank/_mapping?pretty"
```

聚合查询 [Refer](https://www.cnblogs.com/duanxz/p/6528161.html)

记得使用state.keyword，使用完整keyword，其中size=0 表示不需要返回参与查询的文档
```
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
{
  "size": 0,
  "aggs": {
    "return_expires_in": {
      "sum": {
        "field": "expires_in"
      }
    }
  }
}'
{
  "size": 0,
  "aggs": {
    "return_min_expires_in": {
      "min": {
        "field": "expires_in"
      }
    }
  }
}'
{
  "size": 0,
  "aggs": {
    "return_max_expires_in": {
      "max": {
        "field": "expires_in"
      }
    }
  }
}'
{
  "size": 0,
  "aggs": {
    "return_avg_expires_in": {
      "avg": {
        "field": "expires_in"
      }
    }
  }
}'
```

索引自动创建
添加索引数据时，索引mapping会自己创建
```
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" 
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
```

## 实操

[es之零停机重新索引数据](https://www.cnblogs.com/niutao/p/10908988.html)
生产环境的索引一定要记得创建alias，不然后面就等着哭吧！
以下所有操作都是基于一个前提：在建原始索引的时候，给原始索引创建了别名

```
PUT /my_index_v1         //创建索引 my_index_v1
PUT /my_index_v1/_alias/my_index       //设置 my_index为 my_index_v1

创建mapping
1. 原始的索引bank,类型：account,mapping如下
{
    "settings": {
        "number_of_shards": 5
    },
    "mappings": {
        "account": {
            "properties": {
                "content": {
                	"type" : "text",        
					"fields" : {            
					  "keyword" : {         
					    "type" : "keyword", 
					    "ignore_above" : 256
					  }                     
					}                       
                },
                "content2": {
                	"type" : "text"                
                },
                "age": {
                    "type": "long"
                }
            }
        }
    }
}
```
2. 新建一个空的索引bak_bak，类型：account,分片20,age字段由long改成了string类型，具有最新的、正确的配置
```
{
    "settings": {
        "number_of_shards": 6
    },
    "mappings": {
        "account": {
            "properties": {
                "content": {
                	"type" : "text",        
					"fields" : {            
					  "keyword" : {         
					    "type" : "keyword", 
					    "ignore_above" : 256
					  }                     
					}                       
                },
                "content2": {
                	"type" : "text"                
                },
                "age": {
                    "type": "text"
                }
            }
        }
    }
}
```
3. 设置别名
```
POST /_aliases
{
    "actions": [
        { "add": { "index": "articles1", "alias": "my_index" }},
        { "add": { "index": "articles2", "alias": "my_index" }}
    ]
}

PUT /articles2         //创建索引 articles2
PUT /articles2/_alias/my_index       //设置 my_index为 articles2
```
4. 查询当前别名下的所有索引：
```
GET /*/_alias/my_index
```
5. 数据重新索引
```
POST _reindex
{
  "source": {
    "index": "articles1"
  },
  "dest": {
    "index": "articles2"
  }
}
```
查看数据是否进入新的索引
```
GET articles2/article/1
```
6. 接下来修改alias别名的指向（如果你之前没有用alias来改mapping,纳尼就等着哭吧）
```
curl -XPOST localhost:8305/_aliases -d '
{
    "actions": [
        { "remove": {
            "alias": "my_index",
            "index": "articles1"
        }},
        { "add": {
            "alias": "my_index",
            "index": "articles2"
        }}
    ]
}
```