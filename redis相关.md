##### 设置密码
主要是两个参数：
- requirepass：外面服务、客户端来连接redis的密码。
- masterauth：redis从去连接redis主使用的密码。这个意思是说，如果你在主上设置了requirepass参数，你就需要再从上设置masterauth参数，并和主密码指定成一样的。这样从才能继续去同步主的数据。
- 并且，如果你是哨兵模式，还要在/etc/redis-sentinel.conf添加下面一行配置，这里lenovo20!&是那个配置的密码，你可以改成你的密码。否则，哨兵会认为redis主机失联。
  sentinel auth-pass mymaster abc123
  
方法一：在/etc/redis.conf添加下面2个字段，之后重启redis。两个redis主从密码要设置的一样。
```
requirepass "abc123"
masterauth "abc123"
```
方法二：
```
config set masterauth "abc123"
config set requirepass "abc123"
config rewrite
```