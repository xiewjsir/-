##### 查看Linux内核版本命令

- 方法一
```
cat /proc/version
```
- 方法二
```
uname -a
```

##### 查看Linux系统版本的命令
- 方法一 (这种方法只适合Redhat系的Linux)
```
cat /etc/redhat-release
```

##### 查看CPU内核等信息
```
cat /proc/cpuinfo

#查看每个bai物理CPU内核个数
grep "cpu cores" /proc/cpuinfo|uniq

#查看逻辑CPU个数
cat /proc/cpuinfo |grep "processor"|sort -u|wc -l
```

##### 查看linux系统内存使用量和交换区使用量
```
free -mh
```

##### 查看linux系统各分区的使用情况
```
df -h
```

####参考
- [Linux centos7 日常运维——使用w查看系统负载、vmstat命令、top命令、sar命令、nload命令](http://www.mamicode.com/info-detail-2282566.html)