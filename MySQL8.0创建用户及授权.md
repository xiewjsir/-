##### 创建远程链接账号
```
create user '用户名'@'%'  IDENTIFIED  by '密码';
```
##### 修改账号密码加密方式
```
alter user '用户名'@'%' identified with mysql_native_password by '密码';
```
##### 账号授权
```
GRANT ALL ON  `库名`.* TO '用户名'@'%';
```
##### 刷新权限
```
flush privileges;
```