##### ubuntu安装

```
#方法一
- sudo apt-get update && apt-get install -y supervisor
#方法二
- sudo apt-get install python-pip
– sudo pip install supervisor
#方法三
– sudo easy_install supervisor
```
pip的安装请参考 [Python pip 安装使用教](https://www.runoob.com/w3cnote/python-pip-install-usage.html)

##### 配置 Supervisor
Supervisor 配置文件通常存放在 /etc/supervisor/conf.d 目录，在该目录下，可以创建多个配置文件指示 Supervisor 如何监视进程，例如，让我们创建一个开启并监视 queue:work 进程的 laravel-worker.conf 文件：
```
[program:omc-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/omc_backend/artisan queue:work --tries=3
autostart=true
autorestart=true
user=root
numprocs=3
redirect_stderr=true
stdout_logfile=/var/log/laravel-queue-work.log
```
在本例中，numprocs 指令让 Supervisor 运行 8 个 queue:work 进程并监视它们，如果失败的话自动重启。当然，你需要修改 queue:work sqs 的 command 指令来映射你的队列连接。

##### 启动 Supervisor
当成功创建配置文件后，需要刷新 Supervisor 的配置信息并使用如下命令启动进程:
```
supervisord #启动服务
sudo supervisorctl reread
sudo supervisorctl reload
sudo supervisorctl update
sudo supervisorctl start omc-worker:*
```
可以通过 Supervisor 官方文档获取更多信息。

##### 配置文件说明
```
[program:blog] #项目名
directory=/opt/bin #脚本目录
command=/usr/bin/python /opt/bin/test.py #脚本执行命令
autostart=true #supervisor启动的时候是否随着同时启动，默认True
#当程序exit的时候，这个program不会自动重启,默认unexpected
#设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的
autorestart=false
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=1
#日志输出 
stderr_logfile=/tmp/blog_stderr.log 
stdout_logfile=/tmp/blog_stdout.log 
#脚本运行的用户身份 
user = zhoujy 
#把 stderr 重定向到 stdout，默认 false
redirect_stderr = true
#stdout 日志文件大小，默认 50MB
stdout_logfile_maxbytes = 20M
#stdout 日志文件备份数
stdout_logfile_backups = 20

[program:zhoujy] #说明同上
directory=/opt/bin 
command=/usr/bin/python /opt/bin/zhoujy.py 
autostart=true 
autorestart=false 
stderr_logfile=/tmp/zhoujy_stderr.log 
stdout_logfile=/tmp/zhoujy_stdout.log 
#user = zhoujy  
```