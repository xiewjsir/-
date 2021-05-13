-安装docker
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
```

- 查看境像IP地址
```
docker inspect id | grep IPAddress | cut -d '"' -f 4
```
- 构建容器
```
docker run -itd --name=ubuntu_server -p 8084:80 -v D:/www:/var/www ubuntu:latest /bin/bash
```
- 删除未使用的映像
```
docker image prune -a -f
```
- 删除未运行的容器
```
docker container prune -f
```

-安装docker-compose
```
sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

-  Couldn't connect to Docker daemon at http+docker://localhost - is it running?
```
#如果还没有 docker group 就添加一个：

sudo groupadd docker
#将用户加入该 group 内。然后退出并重新登录就生效啦。

sudo gpasswd -a ${USER} docker
#重启 docker 服务

sudo service docker restart
#切换当前会话到新 group 或者重启 X 会话

newgrp - docker
```

```
//构建镜像 emq-iot:latest
docker build -t emq-iot .
或 docker build -t emq-iot -f 路径 .

//用构建的镜像创建容器 emq-iot （-v 本地目录:容器目录  或 -v 容器目录）
docker run -it --name=emq-iot emq-iot:latest

//登录 harbor镜像仓库
docker login https://harbor.smarlife.cn

//打标签 仓库地址/项目/镜像
docker tag emq-iot harbor.smarlife.cn/library/emq-iot

//推送镜像至仓库
docker push harbor.smarlife.cn/library/emq-iot

```

##### Dockerfiler如何使用多个启动命令entrypoint
```
两个办法，一个是CMD不用中括号框起来，将命令用"&&"符号链接：

# 用nohup框起来，不然npm start执行了之后不会执行后面的
CMD nohup sh -c 'npm start && node ./server/server.js'
另一个方法是不用CMD，用ENTRYPOINT命令，指定一个执行的shell脚本，然后在entrypoint.sh文件中写上要执行的命令：

ENTRYPOINT ["./entrypoint.sh"]
entrypoint.sh文件如下：

// entrypoint.sh
nohup npm start &
nohup node ./server/server.js &




实际示例：

EXPOSE 8002
EXPOSE 9999
 
ENTRYPOINT cnpm i  && npm run ci  &&  pm2 start google-chrome   --interpreter none   --   --headless   --disable-gpu   --disable-translate   --disable-extensions   --disable-background-networking   --safebrowsing-disable-auto-update   --disable-sync   --metrics-recording-only   --disable-default-apps   --no-first-run   --mute-audio   --hide-scrollbars   --no-sandbox  --remote-debugging-port=9999   && tail -f /root/logs/master-stdout.log 
```

```
docker build -t emq-iot .
docker run -it --name="emq-iot3" -v /var/www/iot/runtime:/app/emq-iot/runtime -v /var/logs/supervisor:/var/log/supervisor -v /var/www/iot/config/pro:/app/emq-iot/config/pro -p 8003:80 -p 4443:443 -p 9503:9508  emq-iot

docker build -t emq-iot:v2 .

docker tag emq-iot harbor.smarlife.cn/iot/emq-iot

docker push harbor.smarlife.cn/iot/emq-iot


supervisorctl -c /etc/supervisor/supervisord.conf #查看运行状态
supervisorctl -c /etc/supervisor/supervisord.conf restart iot-handle:iot-handle_00 #重启进程 

docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }')    //删除容器
docker rmi $(docker images | grep "none" | awk '{print $3}')    //删除镜像

```

##### 文档
- [Dockerfile文件详解](https://www.cnblogs.com/panwenbin-logs/p/8007348.html)
- [Dockerfile 中的 CMD 与 ENTRYPOINT](https://www.cnblogs.com/sparkdev/p/8461576.html)