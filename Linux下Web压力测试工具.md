##### 前言
这里列举了常见的压力测试工具
1. webbench  测试结果莫名其妙
2. http_load 还行
3. siege 支持https 可以一试
4. ab 略
5. tsung 推荐(可用于复杂系统)

##### webbench
- 这个使用就最为简单的，比Apache自带的ab压力测试要好使一点
- [官网](http://home.tiscali.cz/~cz210552/webbench.html)
- 下载 webbench-1.5.tar.gz
```
wget http://home.tiscali.cz/~cz210552/distfiles/webbench-1.5.tar.gz
```
- 安装
```
tar -xzvf webbench-1.5.tar.gz
cd webbench-1.5
mkdir /usr/local/man
make install clean
```
- 常用参数
    - -t 测试时间
    - -c 并发连接数
- 粟子
```
#模拟1000并发数，测试时间300秒。
webbench -c 100 -t 30 http://api.perfectstyle.cn/v1/es/index

#Benchmarking: GET http://api.perfectstyle.cn/v1/es/index
#100 clients, running 30 sec.

#Speed=172 pages/min, 447419 bytes/sec.
#Requests: 86 susceed, 0 failed.
#分析：每秒钟响应请求数：172 pages/min，每秒钟传输数据量447419 bytes/sec,请求成功数 86，失败0.

w和uptime命令查看负载
#04:03:58 up 10 days, 13:19, 1 user, load average: 0.54, 0.40, 0.20
#1、当前时间 04:03:58
#2、系统已运行的时间 10 days, 13:19
#3、当前在线用户 1 user
#4、平均负载：0.54, 0.40, 0.20，最近1分钟、5分钟、15分钟系统的负载
```

##### Http_load
- 单一进程运行，以并行复用的方式运行，用以测试web服务器的吞吐量与负载。一般简单的压力测试可以使用它
- 下载
```
wget http://www.acme.com/software/http_load/http_load-09Mar2016.tar.gz
```
- 安装
```
tar zxvf http_load-09Mar2016.tar.gz && cd http_load-09Mar2016
make && make install
```
- 常用参数
    - -p 并发访问进程数
    - -f 总的访问次数
    - -r 每秒的访问频率
    - -s 总的访问时间
- 粟子
```
#总访问1000次，并发100,url.txt的内容是域名（http://omc.worthcloud.lo/v1/auth/sendcode）
http_load -p 100 -f 1000 url.txt
#结果
1000 fetches, 100 max parallel, 1.22121e+06 bytes, in 183.821 seconds
1221.21 mean bytes/connection
5.44007 fetches/sec, 6643.48 bytes/sec
msecs/connect: 0.149722 mean, 0.833 max, 0.037 min
msecs/first-response: 17650 mean, 24800.8 max, 1113.99 min
766 bad byte counts
HTTP response codes:
  code 200 -- 234
  code 429 -- 766

#分析
1．1000 fetches, 100 max parallel, 1.22121e+06 bytes, in 183.821 seconds
说明在上面的测试中运行了1000个请求，最大的并发进程数是100，总计传输的数据是1.22121e+06bytes，运行的时间是183.821秒
2．1221.21 mean bytes/connection
说明每一连接平均传输的数据量1.22121e+06/1000=1221.21
3．5.44007 fetches/sec, 6643.48 bytes/sec
说明每秒的响应请求为5.44007，每秒传递的数据为6643.48 bytes/sec
4．msecs/connect: 0.149722 mean, 0.833 max, 0.037 min
说明每连接的平均响应时间是0.149722 msecs，最大的响应时间0.833 msecs，最小的响应时间0.037 msecs
5．msecs/first-response: 17650 mean, 24800.8 max, 1113.99 min
6、HTTP response codes:
    code 200 -- 234
    code 429 -- 766
说明打开响应页面的类型，如果403的类型过多，那可能要注意是否系统遇到了瓶颈。429 意为请求太多
特殊说明：这里，我们一般会关注到的指标是fetches/sec、msecs/connect
他们分别对应的常用性能指标参数
Qpt-每秒响应用户数和response time，每连接响应用户时间。
测试的结果主要也是看这两个值。当然仅有这两个指标并不能完成对性能的分析，我们还需要对服务器的cpu、men进行分析，才能得出结论

#持续300秒，每秒100次访问
http_load -r 100 -s 300 url.txt
```

##### Siege
- 多用户的并发访问，记录每个用户所有请求过程的相应时间，并在一定数量的并发访问下重复进行
- 下载
```
wget http://download.joedog.org/siege/siege-latest.tar.gz
```
- 安装
```
tar zxvf siege-latest.tar.gz && cd siege-4.0.4
./configure
make && make install
siege.config
```
##### 常用参数
- -i 多个url，随机访问
- -c 并发用户数
- -r 重复次数
- -t 测试时间
- -l 输出测试日志
##### 粟子
```
siege -c 200 -t 10 http://omc.worthcloud.lo/v1/auth/sendcode
并发数200，持续时间10秒。
siege -c 50 -r 100 http://omc.worthcloud.lo/v1/auth/sendcode
50个用户（每次并发量，注意不是每秒并发量） 重复100次 共产生 50 * 100 = 5000个请求
siege -c 50 -r 100 http://www.imdst.com/?name=test
50个用户 重复100次 发送GET参数
siege -c 50 -r 100 "http://www.imdst.com POST name=test" 50个用户 重复100次 发送POST参数 (注意引号)
```
##### 结果说明
```
Transactions:		        3273 hits       //完成3273次处理
Availability:		       92.96 %          //92.96 % 成功率
Elapsed time:		      599.24 secs       //总共使用时间
Data transferred:	        3.97 MB         //共数据传输 3.97 MB
Response time:		       35.51 secs       //响应时间，显示网络连接的速度 
Transaction rate:	        5.46 trans/sec  //平均每秒完成 5.46 次处理
Throughput:		        0.01 MB/sec         //平均每秒传送数据
Concurrency:		      193.97            //实际最高并发连接数 
Successful transactions:         681        //成功处理次数
Failed transactions:	         248        //失败处理次数
Longest transaction:	      133.77        //每次传输所花最长时间
Shortest transaction:	        0.98        //每次传输所花最短时间
```

##### Tsung
- 开源的多协议分布式负载测试工具，基于erlang，使用perl+gnuplot生成图表，它能用来压力测试HTTP, WebDAV, SOAP, PostgreSQL, MySQL, LDAP 和 Jabber/XMPP的服务器。它可以分布在多个客户机，并能够模拟成千上万的虚拟用户数并发。
- 下载安装
```
yum -y install perl perl-RRD-Simple.noarch perl-Log-Log4perl-RRDs.noarch gnuplot perl-Template-Toolkit ncurses-devel
#安装erlang
wget http://erlang.org/download/otp_src_22.1.tar.gz
tar -xvf otp_src_22.1.tar.gz && cd otp_src_22.1
./configure --prefix=/usr/local/erlang
make && make install
vim ~/.bash_profile

ER_LANG=/usr/local/erlang
export PATH=$PATH:/usr/local/go/bin:/root/go/bin:$ER_LANG/bin

source ~/.bash_profile

#验证是否安装成功
#执行 【erl】,进入编辑器；
#成功后执行【halt().】 退出编辑器（最后的点别忘记）

#开始安装tsung
git clone https://github.com/processone/tsung.git
cd tsung
./configure --prefix=/usr/local/tsung
make && make install

vim ~/.bash_profile
export PATH=$PATH:/usr/local/go/bin:/root/go/bin:$ER_LANG/bin:/usr/local/tsung/bin
source ~/.bash_profile
```
- 配置
在/usr/share/doc/tsung/examples/目录有多个例子供参考，我们这里主要是使用http_simple.xml进行修改。 Tsung默认配置文件路径为：~/.tsung/
```
mkdir ~/.tsung/  
cp /data/tsung-1.7.0/http_simple.xml ~/.tsung/tsung.xml
```
- 配置文件参考
```
<?xml version="1.0"?>
<!DOCTYPE tsung SYSTEM "/usr/local/erlang/share/tsung/tsung-1.0.dtd">
<tsung loglevel="notice" version="1.0">

  <!-- Client side setup -->
  <clients>
    <client host="localhost" use_controller_vm="true"/>
  </clients>

  <!-- Server side setup -->
<servers>
  <server host="omc.worthcloud.lo" port="80" type="tcp"></server>
</servers>

  <!-- to start os monitoring (cpu, network, memory). Use an erlang
  agent on the remote machine or SNMP. erlang is the default -->
  <monitoring>
    <monitor host="myserver" type="snmp"></monitor>
  </monitoring>

  <load>
  <!-- several arrival phases can be set: for each phase, you can set
  the mean inter-arrival time between new clients and the phase
  duration -->
   <arrivalphase phase="1" duration="10" unit="minute">
     <users interarrival="1" unit="second"></users>
   </arrivalphase>
    <arrivalphase phase="2" duration="10" unit="minute">
      <users maxnumber="600" arrivalrate="10" unit="second"></users>
    </arrivalphase>
  </load>

  <options>
   <option type="ts_http" name="user_agent">
    <user_agent probability="80">Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.7.8) Gecko/20050513 Galeon/1.3.21</user_agent>
    <user_agent probability="20">Mozilla/5.0 (Windows; U; Windows NT 5.2; fr-FR; rv:1.7.8) Gecko/20050511 Firefox/1.0.4</user_agent>
   </option>
  </options>

  <!-- start a session for a http user. the probability is the
  frequency of this type os session. The sum of all session's
  probabilities must be 100 -->

 <sessions>
  <session name="http-example" probability="100" type="ts_http">

    <!-- full url with server name, this overrides the "server" config value -->

    <request> <http url="/" method="GET" version="1.1"></http> </request>
    <request> <http url="/images/accueil1.gif" method="GET" version="1.1" if_modified_since="Fri, 14 Nov 2003 02:43:31 GMT"></http> </request>
    <request> <http url="/images/accueil2.gif" method="GET" version="1.1" if_modified_since="Fri, 14 Nov 2003 02:43:31 GMT"></http> </request>
    <request> <http url="/images/accueil3.gif" method="GET" version="1.1" if_modified_since="Fri, 14 Nov 2003 02:43:31 GMT"></http> </request>

    <thinktime value="20" random="true"></thinktime>

    <request> <http url="/v1/auth/sendcode" method="GET" version="1.1" ></http> </request>

  </session>
 </sessions>
</tsung>

```
- 配置文件说明 [官方用户手册](http://tsung.erlang-projects.org/user_manual/)：
```
clients   客户端，如果是多台客户端同时压测的话，可以配置多个client的IP地址  
servers   被压测服务器配置，主要是填写IP和端口  
load      配置压测信息，如：10分钟之内，不限制并发用户以每2秒的速度并发请求。
          interarrival方式：phase 阶段  duration持续时间  interarrival 多少秒增加一个用户  
          arrivalrate方式： maxnumber最大用户数  arrivalrate 每秒增加用户数
sessions  被压测服务器的具体配置信息，这个可以指定配置的URL，由于我这里有token等相关的变量，因此需要采用文件的形式，否则会提示配置文件错误。

```
- 粟子
```
tsung start 或 tsung -f ~/.tsung/tsung.xml start
#Creating Tsung log directory /root/.tsung/log
#Starting Tsung
#Log directory is: /root/.tsung/log/20191018-1106

/usr/local/tsung/lib/tsung/bin/tsung_stats.pl
```

##### ab
```
yum -y install httpd-tools
```
- 参数说明
```
-n	即requests，用于指定压力测试总共的执行次数。
-c	即concurrency，用于指定的并发数。
-t	即timelimit，等待响应的最大时间(单位：秒)。
-b	即windowsize，TCP发送/接收的缓冲大小(单位：字节)。
-p	即postfile，发送POST请求时需要上传的文件，此外还必须设置-T参数。
-u	即putfile，发送PUT请求时需要上传的文件，此外还必须设置-T参数。
-T	即content-type，用于设置Content-Type请求头信息，例如：application/x-www-form-urlencoded，默认值为text/plain。
-v	即verbosity，指定打印帮助信息的冗余级别。
-w	以HTML表格形式打印结果。
-i	使用HEAD请求代替GET请求。
-x	插入字符串作为table标签的属性。
-y	插入字符串作为tr标签的属性。
-z	插入字符串作为td标签的属性。
-C	添加cookie信息，例如："Apache=1234"(可以重复该参数选项以添加多个)。
-H	添加任意的请求头，例如："Accept-Encoding: gzip"，请求头将会添加在现有的多个请求头之后(可以重复该参数选项以添加多个)。
-A	添加一个基本的网络认证信息，用户名和密码之间用英文冒号隔开。
-P	添加一个基本的代理认证信息，用户名和密码之间用英文冒号隔开。
-X	指定使用的和端口号，例如:"126.10.10.3:88"。
-V	打印版本号并退出。
-k	使用HTTP的KeepAlive特性。
-d	不显示百分比。
-S	不显示预估和警告信息。
-g	输出结果信息到gnuplot格式的文件中。
-e	输出结果信息到CSV格式的文件中。
-r	指定接收到错误信息时不退出程序。
-h	显示用法信息，其实就是ab -help。
```
- 粟子
```
ab -n 100 -c 10 swoole.perfectstyle.cn/admin/login

This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking swoole.perfectstyle.cn (be patient).....done


Server Software:        nginx (服务器软件名称及版本信息)
Server Hostname:        swoole.perfectstyle.cn (服务器主机名)
Server Port:            80 (服务器端口)

Document Path:          /admin/login (供测试的URL路径)
Document Length:        6204 bytes (供测试的URL返回的文档大小)

Concurrency Level:      10 (并发数)
Time taken for tests:   0.897 seconds (压力测试消耗的总时间)
Complete requests:      100 (总次数)
Failed requests:        0 (失败的请求数)
Write errors:           0 (网络连接写入错误数)
Total transferred:      676130 bytes (传输的总数据量)
HTML transferred:       620400 bytes (HTML文档的总数据量)
Requests per second:    111.48 [#/sec] (mean) (平均每秒的请求数) 这个是非常重要的参数数值，服务器的吞吐量
Time per request:       89.699 [ms] (mean) (所有并发用户(这里是1000)都请求一次的平均时间，这里将一次10个并发请求看成一个整体)
Time per request:       8.970 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)
Transfer rate:          736.11 [Kbytes/sec] received 每秒获取的数据长度 (传输速率，单位：KB/s)

网络上消耗的时间的分解：
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       0
Processing:    11   75 133.3     37     759
Waiting:       11   74 133.5     36     759
Total:         11   75 133.3     37     759

Percentage of the requests served within a certain time (ms)
  50%     37 50%的请求在37ms内返回 
  66%     44 66%的请求在44ms内返回
  75%     52
  80%     60
  90%    131
  95%    310
  98%    741
  99%    759
 100%    759 (longest request)
```
##### 参考文档
- [Linux下Web压力测试工具](https://blog.imdst.com/linuxxia-webya-li-ce-shi-gong-ju/)
- [linux下uptime命令详解](https://www.cnblogs.com/ultranms/p/9253217.html)
- [http_load使用详解](https://www.jianshu.com/p/c869f96ed929)
- [Siege高性能压测工具](https://www.jianshu.com/p/74c465ff136f)
- [压力测试工具tsung初体验](https://codezye.com/2015/12/28/%E5%8E%8B%E5%8A%9B%E6%B5%8B%E8%AF%95%E5%B7%A5%E5%85%B7tsung%E7%94%A8%E6%B3%95%E7%AE%80%E4%BB%8B/)
- [Tsung测试统计报告说明](https://blog.csdn.net/u010419967/article/details/37516019)
- [tsung压力测试——tcp测试tsung.xml配置模版说明](https://www.cnblogs.com/lemon-flm/p/7885509.html)