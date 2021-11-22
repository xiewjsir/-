##### 安装基础工具
```
yum -y update
yum -y install g++ gcc gcc-c++ make kernel-*    #主要是在安装增强工具提示没有安装这些软件
yum -y install bzip2*   #增强工具用的是bzip2压缩
```

##### 安装增强功能
```
#virtualbox 设备->安装增强功能 无反应 (虽无反应，但手动安装前，这一步必须有) 
sudo mkdir --p /media/cdrom
sudo mount -t auto /dev/cdrom /media/cdrom/
cd /media/cdrom/
sudo sh VBoxLinuxAdditions.run
```
##### 虚拟机安laradock 相关操作
```
#创建用户组
groupadd -g 993 vboxsf  (alpine:addgroup -g 993 -S vboxsf)
gpasswd -a www-data vboxsf (alpine:addgroup -g 993 -S vboxsf)
```

