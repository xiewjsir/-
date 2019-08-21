- 查看境像IP地址
```
docker inspect id | grep IPAddress | cut -d '"' -f 4
```
- 构建容器
```
docker run -itd --name=ubuntu_server -p 8084:80 -v D:/www:/var/www ubuntu:latest /bin/bash
```