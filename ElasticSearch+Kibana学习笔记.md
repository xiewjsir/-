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

##### 参考文档
- [elasticsearch/elasticsearch](https://packagist.org/packages/elasticsearch/elasticsearch)
- [为 Laravel 适配 ElasticSearch 日志驱动 ***](https://my.oschina.net/zobeen/blog/2250157)
- [Kibana（一张图片胜过千万行日志）](https://www.cnblogs.com/cjsblog/p/9476813.html)
- [Elasticsearch-PHP 中文文档](https://learnku.com/docs/elasticsearch-php/6.0)