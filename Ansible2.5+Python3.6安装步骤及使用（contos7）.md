安装基础工具
```
yum -y install net-tools vim wget

which git #查看是否安装了GIT,没有安装则安装，否则跳过
yum -y install git nss curl
```

关闭防火墙
```
firewall-cmd --state  #查看防火墙状态
systemctl stop firewalld.service  #关闭防火墙
systemctl stop firewalld.service  #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
永久关闭selinux
```
vim /etc/sysconfig/selinux
SELINUX=enforcing 改为 SELINUX=disabled
reboot   #重启服务
getenforce #查看是否关闭成功
```
安装Python3.6.5 + virtualenv
```
yum install -y gcc gcc-c++ ncurses ncurses-devel unzip zlib-devel zlib openssl-devel openssl libffi-devel
cd /tmp
wget http://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz
tar xf Python-3.6.5.tar.xz
cd Python-3.6.5
./configure --prefix=/usr/local --with-ensurepip=install --enable-shared LDFLAGS="-wl,-rpath /usr/local/lib" #报错无法解决 换为
./configure --prefix=/usr/local 
make && make install
which pip3.6
ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip
pip install virtualenv
useradd deploy && su - deploy
virtualenv -p /usr/local/bin/python3.6 .py3-a2.5-env
```
安装ansible
```
cd /home/deploy
git clone https://github.com/ansible/ansible.git
source /home/deploy/.py3-a2.5-env/bin/activate

pip install paramiko PyYAML jinja2 #安装ansible依赖包
mv ansible .py3-a2.5-env/
cd .py3-a2.5-env/ansible/

#在python3.6虚拟环境下加载ansible2.5
source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q 
ansible --version
```
免密码登录
```
ssh-keygen -t rsa
ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com #目标服务器 ~/.ssh和~/.ssh/authorized_keys
#如报ECDSA host key for ip has changed 错误，删除本地known_hosts里面的缓存信息即可。命令：
ssh-keygen -R "你的远程服务器ip地址或域名"  
```
测试
```
su - deploy
source .py3-a2.5-env/bin/activate
source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q
ansible-playbook --version
cd /home/deploy/test_playbooks #  创建以下目录结构和文件

├── deploy.yml #Playbook任务入口文件
├── inventory #server详细清单
│   └── testenv #具体清单和变量申明
└── roles #任务列表
    └── testbox #详细任务
        ├── files
        │   └── foo.sh
        ├── tasks
        │   └── main.yml #主任务文件
        └── templates
            └── nginx.conf.j2

ansible-playbook -i inventory/testenv ./deploy.yml
```

模块说明
```
#testenv 文件
[testservers] #server组列表
www.webserver.com #目标部署服务器

[testservers:vars] #server组列表参数
server_name=www.webserver.com #key/val参数
user=root #key/val参数
output=/root/test.txt #key/val参数
port=80 #key/val参数
user=deploy #key/val参数
worker_processes=4 #key/val参数
max_open_file=65505 #key/val参数
root=/www #key/val参数


#deploy.yml 文件
- hosts: "testservers" #server列表
  gather_facts: true #获取server基本信息
  remote_user: root #目标服务器系统用户
  roles:
    - testbox #进入roles/testbox任务目录，执行task任务


#main.yml 文件
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo' #在目录主机创建文件或目录，并赋予其系统权限
- name: copy a file
  copy: 'remote_src=no src=roles/testbox/files/foo.sh dest=/root/foo.sh mode=0644 force=yes' #实现Ansible服务端到目标主机的文件传送
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh' #获取远程文件状态信息
  register: script_stat #将stat结果赋值给 script_stat
- debug: msg="foo.sh exists" #打印语句到Ansible执行输出
  when: script_stat.stat.exists #跟Stat模块配合使用
- name: run the script
  command: 'sh /root/foo.sh' #用来执行Linux目标主机命令，区别为：Shell —— 会调用系统中的/bin/bash，这样就可以使用系统中的环境变量，例如重定向，管道符
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx.conf  #实现Ansible服务端到目标主机的jinja2模板传送
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest #Packaging模块 调用目标主机系统包管理工具（yum, apt）进行安装
- name: start nginx service
  service: name=nginx state=started #管理目标主机系统服务
```

##### 附Ubuntu安装Ansible
```
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
```