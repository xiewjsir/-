##### 安装xhprof
```
git clone https://github.com/longxinH/xhprof.git ./xhprof

cd xhprof/extension/

/path/to/php7/bin/phpize #更换为自已对应的路径,用whereis php可以查看 /usr/local/bin/phpize

./configure --with-php-config=/usr/local/bin/php-config

make && sudo make install
```
##### 添加xhprof.so扩展
```
vim /usr/local/etc/php/php.ini

[Xhprof]
extension=xhprof.so
xhprof.output_dir=/var/www/xhprof/logs #(自定义)

#或laradock
docker-php-ext-enable xhprof

#重启PHP-FPM
```
XHPROF_FLAGS_MEMORY:统计内存占用
XHPROF_FLAGS_CPU:统计cpu占用
XHPROF_FLAGS_NO_BUILTINS:不统计内置函数, 此次输出可以看到已经忽略了我们拓展中的函数
我们发现统计内存占用的字段, 有两个输出 mu 和 pmu , mu 表示使用的内存(bytes), pmu 表示使用的内存峰值(bytes)
```
xhprof_enable();
......
$xhprof_data = \xhprof_disable();
print_r($xhprof_data);
exit;

#ct 表示 当前这个函数调用的次数,此案例都是1次
#wt 表示 函数执行时间的耗时，单位为微秒
#默认获取的信息并不是很多, 比如我们经常还要关心占用的内存、cpu等指标.
xhprof_enable(XHPROF_FLAGS_MEMORY+XHPROF_FLAGS_CPU+XHPROF_FLAGS_NO_BUILTINS);
```
```
xhprof_enable();
......
$xhprof_data = \xhprof_disable();

include_once  '/var/www/xhprof/xhprof_lib/utils/xhprof_lib.php';
include_once  '/var/www/xhprof/xhprof_lib/utils/xhprof_runs.php';
$xhprof_runs = new \XHProfRuns_Default();
$run_id = $xhprof_runs->save_run($xhprof_data, 'your_project');
echo $run_id;exit;

```

通过指向/var/www/xhprof/xhprof_html的或名查看统计图表

字段名|含义
|:--|:---|
|Calls|调用次数|
|Incl. Wall Time|调用的包括子函数所有花费时间，以微秒算|
|Excl. Wall Time|函数执行本身花费的时间，不包括子树执行时间,以微秒算|
|Incl. CPU|调用的包括子函数所有花费的cpu时间|
|Excl. CPU|函数执行本身花费的cpu时间，不包括子树执行时间,以微秒算|
|Incl.MemUse|包括子函数执行使用的内存, 以字节算|
|Excl.MemUse|函数执行本身内存,以字节算|
|Incl.PeakMemUse|Incl.MemUse的峰值|
|Excl.PeakMemUse|Excl.MemUse的峰值|

##### 安装图形工具graphviz ，安装失败，错误 autoreconf: /usr/bin/autoconf failed with exit status: 1 无法解决
下载地址[官方最新](https://graphviz.gitlab.io/_pages/Download/Download_source.html)
，这次安装版本[2.24.0](http://lcginfo.cern.ch/pkgver/graphviz/2.24.0/)
```
sudo apt-get install libtool 
sudo apt-get install libsysfs-dev

#解压(下载下来的文件是graphviz-2.24.0.tar.gz解压报错，重命名graphviz-2.24.0.tar后解压成功)
tar -xvf graphviz-2.24.0.tar
cd graphviz-2.24.0
./autogen.sh
./configure
make
make install
```

##### 问题
- Can't exec "aclocal": No such file or directory at /usr/local/share/autoconf/Autom4te/FileUtils.pm line 326.
```
#解决办法，下载安装automake
```


##### 参考文档
- [php7中使用 xhprof 分析](https://juejin.im/post/5cbdc51fe51d456e6d1334aa)
- [PHP 性能分析第一篇: Xhprof & Xhgui 介绍](http://blog.oneapm.com/apm-tech/235.html)
- [PHP 性能分析第二篇: Xhgui In-Depth](http://blog.oneapm.com/apm-tech/219.html)
- [PHP 性能分析第三篇: 性能调优实战](http://blog.oneapm.com/apm-tech/216.html)
- [PHP 性能分析工具xhgui+tideways实践](https://tyloafer.github.io/posts/7748/)