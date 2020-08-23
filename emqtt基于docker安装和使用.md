###### 安装
- [官方安装教程](https://developer.emqx.io/docs/broker/v3/cn/install.html)
- 获取 docker 镜像(可跳过)
```
docker pull emqx/emqx:v3.1.0
```
- 启动 docker 容器
```
docker run -d --name emqx405 -v /tmp/emqx:/tmp/emqx -p 1883:1883 -p 8083:8083 -p 8883:8883 -p 8084:8084 -p 18083:18083 --network laradock_backend emqx/emqx:v4.0.5
```
- 进入容器控制台
```
sudo docker exec -it emqx405 /bin/sh
```

- 访问控制台,默认用户名：admin,密码：public
```
http://ip:18083
```

##### 使用
> [MQTT 客户端库](https://developer.emqx.io/sdk_tools?category=MQTT_Clients)
- PHP (laravel)，订阅无法收到消息，没找到原因，后改为golang实现
```
composer require bluerhinos/phpmqtt
```
订阅和发布示例，[官方DEMO](https://github.com/bluerhinos/phpMQTT/tree/master/examples)
```
#发布
Artisan::command('publish', function () {
    $server    = "192.168.1.68";
    $port      = 1883;
    $username  = "admin";
    $password  = "public";
    $client_id = "clientid183778760";
    $mqtt      = new \Bluerhinos\phpMQTT($server, $port, $client_id);
    if ($mqtt->connect(true, NULL, $username, $password)) {
        $mqtt->publish("helloworld/publishtest", "Hello World! at " . date("r"), 0);
        $mqtt->close();
        echo 'ok' . PHP_EOL;
    } else {
        echo "Time out!" . PHP_EOL;
    }
})->describe('emqtt publish');

#订阅
public function handle()
{
    $server    = "192.168.1.62";
    $port      = 1883;
    $username  = "admin";
    $password  = "public";
    $client_id = "clientid183778762"; // make sure this is unique for connecting to sever - you could use uniqid()
    $mqtt      = new \Bluerhinos\phpMQTT($server, $port, $client_id);

    if (!$mqtt->connect(true, NULL, $username, $password)) {
        exit(1);
    }

    $topics['helloworld/publishtest'] = array("qos" => 0, "function"=>[$this, "callback"]);
    $mqtt->subscribe($topics, 0);

    while ($mqtt->proc()) {

    }

    $mqtt->close();

    echo "all done!" . PHP_EOL;
}

public function callback($topic = null, $msg = null)
{
    echo "Msg Recieved: " . date("r") . "\n";
    echo "Topic: {$topic}\n\n";
    echo "\t$msg\n\n";
}
```

- golang
> [官方手册](https://godoc.org/github.com/eclipse/paho.mqtt.golang#ClientOptions),[源码](https://github.com/eclipse/paho.mqtt.golang/blob/master/cmd)
```
package main

import (
	"fmt"
	"log"
	"os"
	"time"
	"github.com/eclipse/paho.mqtt.golang"
)

var f mqtt.MessageHandler = func(client mqtt.Client, msg mqtt.Message) {
	fmt.Printf("TOPIC: %s\n", msg.Topic())
	fmt.Printf("MSG: %s\n", msg.Payload())
}

func main() {
	mqtt.DEBUG = log.New(os.Stdout, "", 0)
	mqtt.ERROR = log.New(os.Stdout, "", 0)

	opts := mqtt.NewClientOptions().AddBroker("tcp://192.168.1.68:1883").SetClientID("go-client-01")
	opts.SetUsername("admin").SetPassword("public")
	opts.SetKeepAlive(2 * time.Second)
	opts.SetDefaultPublishHandler(f)
	opts.SetPingTimeout(1 * time.Second)

	c := mqtt.NewClient(opts)
	if token := c.Connect(); token.Wait() && token.Error() != nil {
		panic(token.Error())
	}

	if token := c.Subscribe("helloworld/publishtest", 0, nil); token.Wait() && token.Error() != nil {
		fmt.Println(token.Error())
		os.Exit(1)
	}

	//for i := 0; i < 5; i++ {
	//	text := fmt.Sprintf("this is msg #%d!", i)
	//	token := c.Publish("helloworld/publishtest", 0, false, text)
	//	token.Wait()
	//}
	//
	//time.Sleep(6 * time.Second)
	//
	//if token := c.Unsubscribe("helloworld/publishtest"); token.Wait() && token.Error() != nil {
	//	fmt.Println(token.Error())
	//	os.Exit(1)
	//}
	//
	//c.Disconnect(250)
	//
	//time.Sleep(3 * time.Second)

	for {
		//Lazy...
		time.Sleep(500 * time.Millisecond)
	}
}
```

##### 源码安装
```
_build/emqx/rel/emqx/bin/emqx start
```
##### 认证设置
- 启用emqx_auth_username插件
```
vim etc/emqx.conf

allow_anonymous = true #默认开启匿名认证，任何客户端都可以连接mqtt服务器

#改为

allow_anonymous = false #关闭匿名认证
```
- 设置账号密码
```
vi etc/plugins/emqx_auth_username.conf

auth.user.1.username = admin  
auth.user.1.password = public  
auth.user.2.username = perfectlystyle@gmail.com
auth.user.2.password = 123456  
auth.user.3.username = name~!@#$%^&*()_+  
auth.user.3.password = pwsswd~!@#$%^&*()_+  
```

##### acl访问控制
```

```

##### 参考文档
- [使用golang开发mqtt服务压力测试工具](https://www.colabug.com/5646869.html)
- [How I can check if I lose connection with mqtt broker?](http://cn.voidcc.com/question/p-wrgqqnkn-ua.html)
- [centos 7 安装Erlang](https://www.cnblogs.com/song-wentao/p/11297966.html)
- [RabbitMQ的六种工作模式](https://www.cnblogs.com/Jeely/p/10784013.html)
- [rabbitmq官方的六种工作模式](https://blog.csdn.net/qq_33040219/article/details/82383127)
- [emq共享分组订阅问题](https://www.jianshu.com/p/d886afb416d3)
- [EMQ百万级MQTT消息服务(小技巧)](https://my.oschina.net/wenzhenxi/blog/1795750)
- [那些年用EMQ踩过的坑](https://www.freesion.com/article/135586096/)
- [EMQ 认证设置和acl访问控制](https://www.cnblogs.com/xiaoxianxianxian/p/13191345.html)