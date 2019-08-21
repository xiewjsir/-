##### 安装
> [官方安装教程](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)
```
wget -P /tmp/harbor https://storage.googleapis.com/harbor-releases/release-1.8.0/harbor-online-installer-v1.8.0.tgz

cd /tmp/harbor
tar zxf harbor-online-installer-v1.8.0.tgz -C /usr/local/
cd /usr/local/harbor/

#调整配置文件，主要修改域名和端口号
vim harbor.yml

#安装，harbor通过docker-compose安装和管理
./install.sh

#安装之后的目录结构
.
├── common
│   └── config
│       ├── core
│       │   ├── app.conf
│       │   ├── certificates
│       │   └── env
│       ├── db
│       │   └── env
│       ├── jobservice
│       │   ├── config.yml
│       │   └── env
│       ├── log
│       │   └── logrotate.conf
│       ├── nginx
│       │   ├── client_body_temp
│       │   ├── conf.d
│       │   ├── fastcgi_temp
│       │   ├── nginx.conf
│       │   ├── proxy_temp
│       │   ├── scgi_temp
│       │   └── uwsgi_temp
│       ├── registry
│       │   ├── config.yml
│       │   └── root.crt
│       └── registryctl
│           ├── config.yml
│           └── env
├── docker-compose.yml
├── harbor.yml
├── install.sh
├── LICENSE
└── prepare

```
安装之后查看运行情况
```
[root@localhost harbor]# docker-compose ps
      Name                     Command                  State                 Ports          
---------------------------------------------------------------------------------------------
harbor-core         /harbor/start.sh                 Up (healthy)                            
harbor-db           /entrypoint.sh postgres          Up (healthy)   5432/tcp                 
harbor-jobservice   /harbor/start.sh                 Up                                      
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp
harbor-portal       nginx -g daemon off;             Up (healthy)   80/tcp                   
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:8088->80/tcp     
redis               docker-entrypoint.sh redis ...   Up             6379/tcp                 
registry            /entrypoint.sh /etc/regist ...   Up (healthy)   5000/tcp                 
registryctl         /harbor/start.sh                 Up (healthy)  
```
配置/etc/hosts
```
.....
127.0.0.1 harbor.wayne.com
```
访问（域名自行更改，默认80）
```
http://harbor.wayne.com:8088
```
默认账号:admin,密码：Harbor12345 可以在harbor.yml文件中查看到

##### 使用

```
docker login http://harbor.wayne.com:8088

docker images
nginx                              alpine                     dd025cdfe837        4 weeks ago         16.1MB

docker tag nginx:alpine harbor.wayne.com:8088/webserver/nginx  #webserver为项目名称
nginx                                   alpine                     dd025cdfe837        4 weeks ago         16.1MB
harbor.wayne.com:8088/webserver/nginx   latest                     dd025cdfe837        4 weeks ago         16.1MB

docker push  harbor.wayne.com:8088/webserver/nginx
docker rmi  harbor.wayne.com:8088/webserver/nginx
docker pull harbor.wayne.com:8088/webserver/nginx:latest
```
登录时报错：
Error response from daemon: Get https://192.168.1.67/v2/: dial tcp 192.168.1.67:443: connect: connection refused
解决这个问题，重启容器即可
```
docker-compose restart
```

