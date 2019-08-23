- ##### 安装
  [官方安装教程](https://www.rabbitmq.com/install-debian.html)
  ```
  #5672:默认的客户端连接的端口
  #15672：默认的web管理界面的端口
  
  docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management
  
  #访问管理系统，用户/密码：guest/guest
  http://host-ip:15672
  
  #集群安装
  docker network create --driver bridge rabbitmq_net_1
  docker inspect rabbitmq_net_1
  
  docker run -d --hostname rabbit1 --name myrabbit1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management
  
  docker exec -it myrabbit1 bash
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl start_app
  exit
  
  docker run -d --hostname rabbit2 --name myrabbit2 -p 5673:5672 --link myrabbit1:rabbit1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management
  
  docker exec -it myrabbit2 bash
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl join_cluster --ram rabbit@rabbit1 #参数“--ram”表示设置为内存节点，忽略次参数默认为磁盘节点。
  rabbitmqctl start_app
  exit
  
  docker run -d --hostname rabbit3 --name myrabbit3 -p 5674:5672 --link myrabbit1:rabbit1 --link myrabbit2:rabbit2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:3-management
  
  docker exec -it myrabbit3 bash
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl join_cluster --ram rabbit@rabbit1
  rabbitmqctl start_app
  exit
  
  ```
  ##### 核心概念
  - broker：简单来说就是消息队列服务器实体。
  - vhost：虚拟主机，一个 broker 里可以开设多个vhost，用作不同用户的权限分离。
  - exchange：消息交换机，它指定消息按什么规则路由到哪个队列。
  - queue：消息队列载体，每个消息都会被投入到一个或多个队列。
  - binding：绑定，它的作用就是把 exchange 和 queue 按照路由规则绑定起来。
  - routing key：路由关键字，exchange 根据这个关键字进行消息投递。
  - producer：消息生产者，就是投递消息的程序。
  - consumer：消息消费者，就是接受消息的程序。
  - channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。
  - exchange type：包含四种消息交换机类型：
      - fanout（复制分发路由）把所有发送到该 exchange的消息路由到所有与它绑定的 queue 中。
      - direct（直接转发路由）把消息路由到那些binding key与 routing key 完全匹配的 queue 中。
      - topic（通配路由）将消息路由到binding key 与 routing key 相匹配的 queue 中。
      - headers 不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配。

  ##### 持久化
  RabbitMQ要实现发布订阅持久化，按照消息的传输流程，可以分成三类：

  - Exchange 持久化：如果不设定Exchange持久化，那么在RabbitMQ由于某些异常等原因重启之后，Exchange会丢失。Exchange丢失， 会影响发送端发送消息到RabbitMQ。
  ```
  #exchange持久化,在声明时指定durable => true
  $channel.exchange_declare(
          $exchange,
          $type,
          $passive = false,
          $durable = false,
          $auto_delete = true,
          $internal = false,
          $nowait = false,
          $arguments = array(),
          $ticket = null
      ) ;//声明消息交换机，且为可持久化的
  ```
  - Queue持久化：发送端将消息发送至Exchange，Exchange将消息转发至关联的Queue。如果Queue不设置持久化，那么在RabbitMQ重启之后，Queue信息会丢失。导致消息发送至Exchange，但Exchange不知道需要将该消息发送至哪些具体的队列。
  ```
  #queue持久化,在声明时指定durable => true
  $channel.queue_declare(
          $queue = '',
          $passive = false,
          $durable = false,
          $exclusive = false,
          $auto_delete = true,
          $nowait = false,
          $arguments = array(),
          $ticket = null
      )
  ```
  - Message持久化：发送端将消息发送至Exchange，Exchange将消息转发至关联的Queue，消息存储于具体的Queue中。如果RabbitMQ重启之后，由于Message未设置持久化，那么消息会在重启之后丢失。
  ```
  #消息持久化,在投递时指定$msg.delivery_mode => 2(1是非持久化).
  $channel->basic_publish(
          $msg,
          $exchange = '',
          $routing_key = '',
          $mandatory = false,
          $immediate = false,
          $ticket = null
      )
  ```

  为了保证发布订阅的持久化，必须设置Exchange、Queue、Message的持久化，才可以保证消息最终不会丢失
  ##### 应用
  [官方应用教程](https://www.rabbitmq.com/getstarted.html)
  - 简单模式
  ```
  $connection = new AMQPStreamConnection('192.168.1.73', 5672, 'guest', 'guest');
  $channel    = $connection->channel();
  $channel->queue_declare('hello', false, false, false, false);
  
  if ('send' == $this->option('act')) {
      $msg = new AMQPMessage('Hello World!');
      $channel->basic_publish($msg, '', 'hello');
  
      $this->info("[x] Sent 'Hello World!'");
  } else {
      $this->info("[*] Waiting for messages. To exit press CTRL+C");
  
      $callback = function ($msg) {
          $this->info("[x] Received {$msg->body}");
      };
  
      $channel->basic_consume('hello', '', false, true, false, false, $callback);
  
      while (count($channel->callbacks)) {
          $channel->wait();
      }
  }
  
  $channel->close();
  $connection->close();
  ```
  - 任务队列
  ```
  $connection = new AMQPStreamConnection('192.168.1.73', 5672, 'guest', 'guest');
  $channel    = $connection->channel();
  
  $channel->queue_declare('task_queue', false, true, false, false);
  
  if ('send' == $this->option('act')) {
      $data = $this->option('msg');
      if (empty($data)) {
          $data = "Hello World!";
      }
      $msg = new AMQPMessage(
          $data,
          array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT) #不同之处
      );
  
      $channel->basic_publish($msg, '', 'task_queue');
  
      echo ' [x] Sent ', $data, "\n";
  } else {
      echo " [*] Waiting for messages. To exit press CTRL+C\n";
  
      $callback = function ($msg) {
          echo ' [x] Received ', $msg->body, "\n";
          sleep(substr_count($msg->body, '.'));
          echo " [x] Done\n";
          $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
      };
  
      $channel->basic_qos(null, 1, null); #不同之处
      $channel->basic_consume('task_queue', '', false, false, false, false, $callback);
  
      while (count($channel->callbacks)) {
          $channel->wait();
      }
  }
  
  $channel->close();
  $connection->close();
  ```
  - 发布订阅
  ```
  $connection = new AMQPStreamConnection('192.168.1.73', 5672, 'guest', 'guest');
  $channel    = $connection->channel();
  
  $channel->exchange_declare('logs', 'fanout', false, false, false);
  
  if ('send' == $this->option('act')) {
      $data = $this->option('msg');
      if (empty($data)) {
          $data = "info: Hello World!";
      }
      $msg = new AMQPMessage($data);
  
      $channel->basic_publish($msg, 'logs');
  
      echo ' [x] Sent ', $data, "\n";
  } else {
      list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);
  
      $channel->queue_bind($queue_name, 'logs');
  
      echo " [*] Waiting for logs. To exit press CTRL+C\n";
  
      $callback = function ($msg) {
          echo ' [x] ', $msg->body, "\n";
      };
  
      $channel->basic_consume($queue_name, '', false, true, false, false, $callback);
  
      while (count($channel->callbacks)) {
          $channel->wait();
      }
  }
  
  $channel->close();
  $connection->close();
  ```
  - 路由模式
  ```
  $connection = new AMQPStreamConnection('192.168.1.73', 5672, 'guest', 'guest');
  $channel    = $connection->channel();
  
  $channel->exchange_declare('direct_logs', 'direct', false, false, false);
  
  if ('send' == $this->option('act')) {
  
      $severity = $this->option('severity') ? $this->option('severity') : 'info';
      $data = $this->option('msg');
      if (empty($data)) {
          $data = "Hello World!";
      }
  
      $msg = new AMQPMessage($data);
      $channel->basic_publish($msg, 'direct_logs', $severity);
      echo ' [x] Sent ', $severity, ':', $data, "\n";
  } else {
      list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);
  
      $severities = ['warning','error'];
      foreach ($severities as $severity) {
          $channel->queue_bind($queue_name, 'direct_logs', $severity);
      }
  
      echo " [*] Waiting for logs. To exit press CTRL+C\n";
  
      $callback = function ($msg) {
          echo ' [x] ', $msg->delivery_info['routing_key'], ':', $msg->body, "\n";
      };
  
      $channel->basic_consume($queue_name, '', false, true, false, false, $callback);
  
      while (count($channel->callbacks)) {
          $channel->wait();
      }
  }
  
  $channel->close();
  $connection->close();
  ```
  ```
  php artisan rabbit:routing --act=rec
  php artisan rabbit:routing --act=send --severity=error --msg='test pub sub'
  ```
  - 主题模式
  > topic 模式可以理解为主题模式，当 key 包涵某个主题时，即可进入该主题的队列。topic 模式的 key 必须具有固定的格式：以 . 作为间隔的一串单词；比如：quick.orange.rabbit，key 最多不能超过 255byte。
  > 交换机和队列的key可以以类似正则表达式的方式存在，有两种语法


      - "*" 可以替代一个单词
      - "#" 可以替代 0 个或多个单词


  ​    
  ```
  $connection = new AMQPStreamConnection('192.168.1.73', 5672, 'guest', 'guest');
  $channel    = $connection->channel();
  
  $channel->exchange_declare('topic_logs', 'topic', false, false, false);
  
  if ('send' == $this->option('act')) {
      $routing_key = $this->option('routing_key');
  
      $data = $this->option('msg');
      if (empty($data)) {
          $data = "Hello World!";
      }
  
      $msg = new AMQPMessage($data);
  
      $channel->basic_publish($msg, 'topic_logs', $routing_key);
  
      echo ' [x] Sent ', $routing_key, ':', $data, "\n";
  } else {
      list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);
  
      $binding_keys = ['#','ern.*','*.critical'];
      foreach ($binding_keys as $binding_key) {
          $channel->queue_bind($queue_name, 'topic_logs', $binding_key);
      }
  
      echo " [*] Waiting for logs. To exit press CTRL+C\n";
  
      $callback = function ($msg) {
          echo ' [x] ', $msg->delivery_info['routing_key'], ':', $msg->body, "\n";
      };
  
      $channel->basic_consume($queue_name, '', false, true, false, false, $callback);
  
      while (count($channel->callbacks)) {
          $channel->wait();
      }
  }
  
  $channel->close();
  $connection->close();
  ```
  ```
  php artisan rabbit:topic --act=rec
  php artisan rabbit:topic --act=send --routing_key=kern.critical --msg='test pub sub'
  ```

  ##### 参考文档

  - [官方教程](https://www.rabbitmq.com/getstarted.html)

  - [hub.docker.com](https://hub.docker.com/_/rabbitmq?tab=description)
  - [RabbitMQ教程](https://blog.csdn.net/hellozpc/article/details/81436980)
  - [RabbitMQ 发布订阅持久化及持久化方式](https://www.cnblogs.com/jiagoushi/p/8678871.html)
  - [RabbitMQ的应用场景以及基本原理介绍](https://blog.csdn.net/whoamiyang/article/details/54954780)
  - [RabbitMQ发布订阅实战-实现延时重试队列](https://www.cnblogs.com/itrena/p/9044097.html)
  - [RabbitMQ系列（五）使用Docker部署RabbitMQ集群](https://www.cnblogs.com/vipstone/p/9362388.html)