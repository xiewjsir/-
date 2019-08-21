##### 环境介绍：
```
系统版本：Centos7.6
内核：3.10.0-957.12.1.el7.x86_64

Keepalived保证nginx服务器高可用
Haproxy实现nginx服务器的负载均衡

192.168.1.68        nginx1
192.168.1.69        nginx2
192.168.1.66        Keepalived + Haproxy1 #(lb1)
192.168.1.77        Keepalived + Haproxy2 #(lb2)
192.168.1.88        VIP
```

##### 一、准备工作
>为方便操作，所有操作均以root用户执行
- 关闭selinux和防火墙
```
sed -ri 's#(SELINUX=).*#\1disabled#' /etc/selinux/config
setenforce 0

systemctl disable firewalld
systemctl stop firewalld
```
- 关闭swap
```
swapoff -a
```
##### 配置keepalived
- 在lb1机器上配置
```
yum install -y keepalived

cat << EOF > /etc/keepalived/keepalived.conf
! Configuration File for keepalived
    global_defs {
        notification_email {
            root@localhost      #发送邮箱
        }
        notification_email_from keepalived@localhost    #邮箱地址   
        smtp_server 127.0.0.1   #邮件服务器地址
        smtp_connect_timeout 30 
        router_id node1         #主机名，每个节点不同即可
        vrrp_mcast_group4 224.0.100.100    #组播地址
    }       
        
vrrp_instance VI_1 {
    state MASTER        #在另一个节点上为BACKUP
    interface enp0s3      #IP地址漂移到的网卡(根据自已系统填写)
    virtual_router_id 6 #多个节点必须相同
    priority 100        #优先级，备用节点的值必须低于主节点的值
    advert_int 1        #通告间隔1秒
    authentication {
        auth_type PASS      #预共享密钥认证
        auth_pass 571f97b2  #密钥
    }
    virtual_ipaddress {
        192.168.1.88/24    #VIP地址
    }
}	
EOF

systemctl enable keepalived
systemctl start keepalived
ip addr show #查看VIP是否生效
```
- 在lb2主机配置
```
yum install -y keepalived

cat << EOF > /etc/keepalived/keepalived.conf
! Configuration File for keepalived
    global_defs {
        notification_email {
            root@localhost      #发送邮箱
        }
        notification_email_from keepalived@localhost    #邮箱地址   
        smtp_server 127.0.0.1   #邮件服务器地址
        smtp_connect_timeout 30 
        router_id node2         #主机名，每个节点不同即可
        vrrp_mcast_group4 224.0.100.100  #组播地址
    }       
        
vrrp_instance VI_1 {
    state BACKUP        #在另一个节点上为MASTER
    interface enp0s3      #IP地址漂移到的网卡
    virtual_router_id 6 #多个节点必须相同
    priority 80        #优先级，备用节点的值必须低于主节点的值
    advert_int 1        #通告间隔1秒
    authentication {
        auth_type PASS      #预共享密钥认证
        auth_pass 571f97b2  #密钥
    }
    virtual_ipaddress {
        192.168.1.88/24    #漂移过来的IP地址
    }
}	
EOF

systemctl enable keepalived
systemctl start keepalived
#测试（如果出现主备同时占用VIP地址的情况，可以关闭防火墙试试）
tcpdump -nn host 224.0.100.100 #组播地组

```

##### 配置Haproxy
- 在lb1主机上
```
yum install  -y haproxy

cat << EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m

frontend kubernetes
    bind *:80
    mode tcp
    default_backend kubernetes-master

backend kubernetes-master
    balance roundrobin
    server master1  192.168.1.68:80 check maxconn 2000
    server master2  192.168.1.69:8088 check maxconn 2000
EOF

systemctl enable haproxy
systemctl start haproxy
```
- 在lb2主机上
```
yum install  -y haproxy

cat << EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m

frontend kubernetes
    bind *:80
    mode tcp
    default_backend kubernetes-master

backend kubernetes-master
    balance roundrobin
    server master1  192.168.1.68:80 check maxconn 2000
    server master2  192.168.1.69:8088 check maxconn 2000
EOF

systemctl enable haproxy
systemctl start haproxy
```

##### ningx1
```
docker run -itd --name=nginx1 -p 80:80 -p 443:443 -v /tmp/nginx1:/usr/share/nginx/html nginx

cat << EOF > /tmp/nginx1/index.html
host1
EOF
```

##### ningx2
```
docker run -itd --name=nginx2 -p 8088:80 -p 8443:443 -v /tmp/nginx2:/usr/share/nginx/html nginx

cat << EOF > /tmp/nginx2/index.html
host2
EOF
```

##### 测试
```
http://192.168.1.88
```

##### Haproxy配置文件详解
```
global # 全局参数的设置
    log         127.0.0.1 local2                #log语法：log <address_1>[max_level_1] # 全局的日志配置，使用log关键字，指定使用127.0.0.1上的syslog服务中的local0日志设备，记录日志等级为info的日志
    chroot      /var/lib/haproxy                #改变当前工作目录
    pidfile     /var/run/haproxy.pid            #当前进程id文件
    maxconn     4000                            #最大连接数
    user        haproxy                         #所属用户
    group       haproxy                         #所属组
    daemon                                      #以守护进程方式运行haproxy
    stats socket /var/lib/haproxy/stats
defaults                                        #默认
    mode        http                            #默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
    log         global                          #应用全局的日志配置
    option      httplog                         #启用日志记录HTTP请求，默认haproxy日志记录是不记录HTTP请求日志
    option      dontlognull                     #启用该项，日志中将不会记录空连接。所谓空连接就是在上游的负载均衡器或者监控系统为了探测该服务是否存活可用时，需要定期的连接或者获取某一固定的组件或页面，或者探测扫描端口是否在监听或开放等动作被称为空连接；官方文档中标注，如果该服务上游没有其他的负载均衡器的话，建议不要使用该参数，因为互联网上的恶意扫描或其他动作就不会被记录下来
    option      http-server-close               #每次请求完毕后主动关闭http通道
    option      forwardfor       except 127.0.0.0/8             #如果服务器上的应用程序想记录发起请求的客户端的IP地址，需要在HAProxy上 配置此选项， 这样 HAProxy会把客户端的IP信息发送给服务器，在HTTP请求中添加"X-Forwarded-For"字段。 启用  X-Forwarded-For，在requests头部插入客户端IP发送给后端的server，使后端server获取到客户端的真实IP。 
    option      redispatch                      #当使用了cookie时，haproxy将会将其请求的后端服务器的serverID插入到cookie中，以保证会话的SESSION持久性；而此时，如果后端的服务器宕掉了， 但是客户端的cookie是不会刷新的，如果设置此参数，将会将客户的请求强制定向到另外一个后端server上，以保证服务的正常。
    retries     3                               #定义连接后端服务器的失败重连次数，连接失败次数超过此值后将会将对应后端服务器标记为不可用
    timeout http-request    10s                 #http请求超时时间
    timeout queue           1m                  #一个请求在队列里的超时时间
    timeout connect         10s                 #连接超时
    timeout client          1m                  #客户端超时
    timeout server          1m                  #服务器端超时
    timeout http-keep-alive 10s                                                             #设置http-keep-alive的超时时间
    timeout check           10s                                                             #检测超时
    maxconn                 3000                                                            #每个进程可用的最大连接数
frontend  main *:80                                                                         #监听地址为80
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js
    use_backend static          if url_static
    default_backend             my_webserver                                                #定义一个名为my_webserver前端部分。此处将对于的请求转发给后端
backend static                                                                              #使用了静态动态分离（如果url_path匹配 .jpg .gif .png .css .js静态文件则访问此后端）
    balance     roundrobin                                                                  #负载均衡算法（#banlance roundrobin 轮询，balance source 保存session值，支持static-rr，leastconn，first，uri等参数）
    server      static 127.0.0.1:80 check                                                   #静态文件部署在本机（也可以部署在其他机器或者squid缓存服务器）
backend my_webserver                                                                        #定义一个名为my_webserver后端部分。PS：此处my_webserver只是一个自定义名字而已，但是需要与frontend里面配置项default_backend 值相一致
    balance     roundrobin                                                                  #负载均衡算法
    server  web01 172.31.2.33:80  check inter 2000 fall 3 weight 30                         #定义的多个后端
    server  web02 172.31.2.34:80  check inter 2000 fall 3 weight 30                         #定义的多个后端
    server  web03 172.31.2.35:80  check inter 2000 fall 3 weight 30                         #定义的多个后端
```

##### Haproxy有8种负载均衡算法（balance）
```
roundrobin          # 轮询，软负载均衡基本都具备这种算法
static-rr           # 根据权重，建议使用
leastconn           # 最少连接者先处理，建议使用
source              # 根据请求源IP，建议使用
uri                 # 根据请求的URI
url_param，         # 根据请求的URl参数'balance url_param' requires an URL parameter name
hdr(name)           # 根据HTTP请求头来锁定每一次HTTP请求
rdp-cookie(name)    # 根据据cookie(name)来锁定并哈希每一次TCP请求
```

##### Haproxy Session会话共享 （未实践，供参考）
- 用户IP 识别：haroxy 将用户IP经过hash计算后 指定到固定的真实服务器上（类似于nginx 的IP hash 指令）。
```
#配置指令
balance source
```
- Cookie 识别：haproxy 将WEB服务端发送给客户端的cookie中插入(或添加加前缀)haproxy定义的后端的服务器COOKIE ID。
```
#配置指令
cookie SESSION_COOKIE insert indirect nocache     #用firebug可以观察到用户的请求头的cookie里 有类似” Cookie jsessionid=0bc588656ca05ecf7588c65f9be214f5; SESSION_COOKIE=app1” SESSION_COOKIE=app1就是haproxy添加的内容
```
- Session 识别：haproxy 将后端服务器产生的session和后端服务器标识存在haproxy中的一张表里。客户端请求时先查询这张表。
```
#配置指令
appsession JSESSIONID len 64 timeout 5h request-learn
```

##### 参考文档
- [虚拟IP和IP漂移](https://xiaobaoqiu.github.io/blog/2015/04/02/xu-ni-iphe-ippiao-yi/)
- [keepalived主备模式同时都有VIP问题](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=4174822&page=1)
- [keepalived实现双机热备](https://www.cnblogs.com/jefflee168/p/7442127.html)
- [kubeadm安装kubernetes v1.11.3 HA多主高可用并启用ipvs](https://blog.51cto.com/bigboss/2174899)
- [HAProxy 调度算法](https://www.jianshu.com/p/aa9ac2f7ceec)
- [Haproxy 8种算法+Session共享](https://blog.csdn.net/qq_25072517/article/details/88316621)
