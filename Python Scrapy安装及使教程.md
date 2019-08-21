##### 安装
- windows10（64位的操作系统，不知道为什么装成32位的python了）
```
#下载Python(https://www.python.org/ftp/python/3.7.3/python-3.7.3.exe)

#下载对应版本Twisted(https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted)
pip install Twisted‑19.2.0‑cp37‑cp37m‑win32.whl  #cp之后数字为对应Python版本号

#安装pywin（https://github.com/mhammond/pywin32/releases/download/b224/pywin32-224.win-amd32-py3.7.exe）

#安装Scrapy
pip install Scrapy

#安装mysqlclient(https://www.lfd.uci.edu/~gohlke/pythonlibs/)
pip install mysqlclient‑1.4.2‑cp37‑cp37m‑win32.whl

#安装mysql相关模块
pip install pymysql sqlalchemy
pip install mysql-connector-python mysql-connector-python

##### 新建项目
```
pwd 
#/var/www/learns/python/learnspider/ixiupet

scrapy startproject ixiupet

#生成目录结构如下
.
├── ixiupet #项目的Python模块，将会从这里引用代码
│    ├── __init__.py
│    ├── items.py #项目的目标文件
│    ├── middlewares.py
│    ├── pipelines.py #项目的管道文件
│    ├── __pycache__
│    ├── settings.py #项目的设置文件
│    └── spiders #存储爬虫代码目录
│        ├── __init__.py
│        └── __pycache__
└── scrapy.cfg #项目的配置文件

```

##### 参考文档
- [Scrapy 入门教程](https://www.runoob.com/w3cnote/scrapy-detail.html)