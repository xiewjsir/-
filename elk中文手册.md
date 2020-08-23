###### 查看所有索引
```
curl -X GET -u elastic:123456 'localhost:9200/_cat/indices' -H 'Content-Type: application/json'
```

###### 清理缓存
```
curl -u elastic:123456  -H "Content-Type:application/json" -XPOST 'http://localhost:9200/索引名/_cache/clear' -d '{ "fielddata": "true" }'```

curl -u elastic:123456  -H "Content-Type:application/json" -XPOST 'http://localhost:9200/_cache/clear' -d '{ "fielddata": "true" }'
```
##### 更改配置
```
curl -u elastic:123456 -XPUT localhost:9200/_cluster/settings -H "Content-Type:application/json" -d '{
  "persistent" : {
    "indices.breaker.request.limit" : "40%"
  }
}'

curl -u elastic:123456 -XPUT localhost:9200/_cluster/settings -H "Content-Type:application/json" -d '{
  "persistent" : {
    "indices.breaker.fielddata.limit" : "40%"
  }
}'
```

###### 其它
```
curl -X GET -u elastic:123456 'localhost:9200/_count?pretty' -H 'Content-Type: application/json' -d '{"query":{"match_all":{}}}'

curl -u elastic:123456 -XPOST "localhost:9200/Indexname/_delete_by_query" -H 'Content-Type: application/json' -d'        
{
  "query": {
    "range": {
      "@timestamp": {
"lt":"2019-04-10T13:52:01.000Z"
      }
    }
  }
}'

#删除
POST /logstash/_delete_by_query?conflicts=proceed&pretty
{
  "query": {
    "range" : {
        "time" : {
           "lte" : "2020-05-30"
        }
    }
  }
}

```

##### 文档
- [ES权威指南](https://es.xiaoleilu.com/010_Intro/15_API.html)
- [ELK中文手册](https://www.kancloud.cn/hanxt/elk/155902)
- [定时删除elasticsearch指定时间之前的数据（delete同步问题）](https://blog.csdn.net/qq_43130832/article/details/88972538)
- [全文搜索引擎 Elasticsearch 入门教程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)