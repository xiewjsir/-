##### 参考文档
https://www.cnblogs.com/wsy1030/p/9228488.html

##### 相关配置
- 修改gitlab配置
```
docker exec -it wayne_gitlab_1 vim /etc/gitlab/gitlab.rb

gitlab_rails['gitlab_shell_ssh_port'] = xxxx #端口
external_url "http://127.0.0.1"  #IP 或域名 不含端口
#配置邮箱
Email Settings
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '183778760@qq.com'
gitlab_rails['gitlab_email_display_name'] = 'Xiewj'
gitlab_rails['gitlab_email_reply_to'] = '183778760@qq.com'
gitlab_rails['gitlab_email_subject_suffix'] = ''

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "183778760@qq.com"
gitlab_rails['smtp_password'] = "授权码"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

- 配置SSH，并开启 root 远程登录
```
vim /etc/ssh/sshd_config
PermitRootLogin yes

#connection reset by (server_ip_address) port 22
rm /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server

service ssh restart #可以尝试连续多重启几次
```

- 开启qq邮箱的POP3/SMTP服务并保存好授权码
1) qq邮箱的设置 -> 账户
2) 测试（可略）
gitlab-rails console Notify.test_email("183778760@qq.com",“title”,“gitlab”).deliver_now

- 测试ssh
```
ssh -vT 127.0.0.1
```
- 主页-系统管理-插件管理
安装几个插件 ，直接搜索就可以
```
ssh               						 #执行远程脚本
gitlab           						 #集成gitlab用
Build Authorization Token Root 	         #构建授权token
Gitlab hook						         #钩子插件
```

- jenkins服务器 生成sshkey: ssh-keygen -t rsa -C "183778760@qq.com"
    - 将公钥加入gitlab:略
    - 私钥加入jenkins:Jenkins->凭据->系统->全局凭据 (unrestricted)->添加凭据

- jenkins->全局安全配置
授权策略: 置为任何用户可以做任何事(没有任何限制)
跨站请求伪造保护：去掉勾选 

- gitlab->管理中心->设置->外发请求
允许钩子和服务访问本地网络:勾选

- 创建一个gitlab项目和jenkins项目
    - jenkins项目配置->构建触发器->触发远程构建->身份验证令牌。拿到 构建链接：JENKINS_URL/job/jenkins-gitlab/build?token=TOKEN_NAME。
    - 在gitlab项目->设置->导入所有仓库，配置刚拿到地构建链接和安全令牌。


##### Jenkins切换root用户
```
docker exec --user root -it jengit_jenkins_1 bash
```


