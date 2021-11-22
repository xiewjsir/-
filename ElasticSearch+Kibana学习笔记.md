##### 安装
```
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag

#或laradock
docker-composer up -d elasticsearch kibana
```

##### laravel
- 安装ES
```
composer require elasticsearch/elasticsearch
```
- 环境配置 .env
```
ELASTIC_HOST=192.168.20.129:9200 # 这里是你的 ElasticSearch 服务器 IP 及端口号
ELASTIC_LOG_INDEX=bf_log # ElasticSearch 索引
ELASTIC_LOG_TYPE=log # ElasticSearch 类型
```
- ES配置文件/config/elasticsearch.php
```
return [
    'hosts' => [
        env('ELASTIC_HOST')
    ],
    'log_index' => env('ELASTIC_LOG_INDEX'),
    'log_type' => env('ELASTIC_LOG_TYPE'),
];

```

```
composer require monolog/monolog
```

- CURL
```
#查询
GET /logstash/_search
{
  "query": {
    "range" : {
        "time" : {
           "lt" : "2020-05-06"
        }
    }
  }
}

#删除
POST /logstash/_delete_by_query?conflicts=proceed&pretty
{
  "query": {
    "range" : {
        "time" : {
           "lte" : "2020-04-29"
        }
    }
  }
}

#查看所以索引
get _cat/indices


#查看当前节点的所有 Index
curl -X GET 'http://localhost:9200/_cat/indices?v'

#列出每个 Index 所包含的 Type。
curl 'localhost:9200/_mapping?pretty=true'

#新建一个名叫weather的 Index
curl -X PUT 'localhost:9200/weather'

#删除Index
curl -X DELETE 'localhost:9200/weather'

```

```
curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}'
```
上面代码中，首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段。
- user
- title
- desc
这三个字段都是中文，而且类型都是文本（text），所以需要指定中文分词器，不能使用默认的英文分词器。
Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。
```
"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
```
上面代码中，analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。



##### 问题
- max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
#切换到root用户
#执行命令：
sysctl -w vm.max_map_count=262144

#查看结果：
sysctl -a|grep vm.max_map_count

#显示：
vm.max_map_count = 262144 

#上述方法修改之后，如果重启虚拟机将失效，所以：
#解决办法：在/etc/sysctl.conf文件最后添加一行

vm.max_map_count=262144

#即可永久修改
```

##### 参考文档
- [全文搜索引擎 Elasticsearch 入门教程](https://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
- [elasticsearch/elasticsearch](https://packagist.org/packages/elasticsearch/elasticsearch)
- [为 Laravel 适配 ElasticSearch 日志驱动 ***](https://my.oschina.net/zobeen/blog/2250157)
- [Kibana（一张图片胜过千万行日志）](https://www.cnblogs.com/cjsblog/p/9476813.html)
- [Elasticsearch-PHP 中文文档](https://learnku.com/docs/elasticsearch-php/6.0)
- [elasticsearch启动时遇到的错误](https://www.cnblogs.com/yidiandhappy/p/7714489.html)
- [Elasticsearch-PHP 中文文档](https://learnku.com/index.php/docs/elasticsearch-php/6.0/search-operations/2009)