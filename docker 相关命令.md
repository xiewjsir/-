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

##### 文档
- []()