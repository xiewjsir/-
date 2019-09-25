# jaeger 使用初探

#### Jaeger（耶格）组件

- **Agent**

  Agent是一个网络守护进程，监听通过UDP发送过来的Span，它会将其批量发送给collector。按照设计，Agent要被部署到所有主机上，作为基础设施。Agent将collector和客户端之间的路由与发现机制抽象了出来。

- **Collector**

  Collector从Jaeger Agent接收Trace，并通过一个处理管道对其进行处理。目前的管道会校验Trace、建立索引、执行转换并最终进行存储。存储是一个可插入的组件，现在支持Cassandra和elasticsearch。

- **Query**

  Query服务会从存储中检索Trace并通过UI界面进行展现，该UI界面通过React技术实现，其页面UI如下图所示，展现了一条Trace的详细信息。

- **存储**

  jaeger采集到的数据必须存储到某个存储引擎，目前支持Cassandra和elasticsearch
  
  

#### 安装

```
#docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one

#添加网络
docker network create go-micro-net

#docker + elasticsearch安装 (elasticsearch:latest 报错，必须指定版本号,最新版是7.3,使用时jaeger报错)
docker run -d --name elasticsearch --net go-micro-net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:5.6
  
#docker + collector安装
docker run -d --name jaeger-collector --restart=always --net go-micro-net -e SPAN_STORAGE_TYPE=elasticsearch -e ES_SERVER_URLS=http://elasticsearch:9200 -e ES_USERNAME=elastic -p 14267:14267 -p 14268:14268 -p 9411:9411 jaegertracing/jaeger-collector

#docker + query安装,注意，ES_USERNAME、ES_PASSWORD这两个环境变量，当你的elasticsearch未设置账号密码时，你可以不填，也可以填上默认值，elasticsearch的默认ES_USERNAME=elastic，ES_PASSWORD=changeme

docker run -d --name jaeger-query --restart=always --net go-micro-net -e SPAN_STORAGE_TYPE=elasticsearch -e ES_SERVER_URLS=http://elasticsearch:9200 -e ES_USERNAME=elastic -e ES_PASSWORD=你的密码 -p 16686:16686/tcp jaegertracing/jaeger-query

#访问地址
host-ip::16686

#docker + agent安装
docker run   -d  --name jaeger-agent --restart=always --net go-micro-net -p 5775:5775/udp   -p 6831:6831/udp   -p 6832:6832/udp   -p 5778:5778/tcp   jaegertracing/jaeger-agent   /go/bin/agent-linux --collector.host-port=collector ip:14267
```



#### 参考文档

- [jaeger 使用初探](https://www.cnblogs.com/chopper-poet/p/10743141.html)
- [使用elasticsearch作为存储引擎部署jaeger](https://my.oschina.net/u/2548090/blog/1821372)
- [Opentracing官方文档](https://github.com/opentracing/specification)
- [OpenTracing语义规范(中文版)](https://segmentfault.com/a/1190000008895129)
- [OpenTracing语义惯例](https://opentracing-contrib.github.io/opentracing-specification-zh/semantic_conventions.html)
- [opentracing文档中文版 ( 翻译 ) 吴晟](https://wu-sheng.gitbooks.io/opentracing-io/content/)
- [elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/docker.html)