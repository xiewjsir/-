#### 1、GO111MODULE环境变量设置(1.11以上版本可以略过)
***

*GO111MODULE=on*

###### linux
*执行*
```
vim /etc/profile
export GO111MODULE=on 
```
###### window
*PowerShell中执行*
```
$env:GO111MODULE = on
```

保存，执行命令
```
source /etc/profile
```

#### 2、GOPROXY 环境变量设置
***

###### linux

```
export GOPROXY=https://goproxy.io
```
###### window
*PowerShell中执行*
```
$env:GOPROXY = "https://goproxy.io"
```

###### go mod 命令

```
download    download modules to local cache (下载依赖的module到本地cache))
edit        edit go.mod from tools or scripts (编辑go.mod文件)
graph       print module requirement graph (打印模块依赖图))
init        initialize new module in current directory (再当前文件夹下初始化一个新的module, 创建go.mod文件))
tidy        add missing and remove unused modules (增加丢失的module，去掉未用的module)
vendor      make vendored copy of dependencies (将依赖复制到vendor下)
verify      verify dependencies have expected content (校验依赖)
why         explain why packages or modules are needed (解释为什么需要依赖)
```

###### 项目初始化
*项目根目录执行*
```
go mod init module名
```

###### 问题
goland无法识别go mod包管理工具
```
//在goland的setting里设置启用Go Modules
goland Preference->Go->Go Modules(vgo) -> Enable Go Modules(vgo)intergration
```

###### 其它参考
- [跳出Go module的泥潭](https://colobu.com/2018/08/27/learn-go-module/)
- [outside GOPATH, no import comments](https://blog.csdn.net/weixin_36920975/article/details/89916993) 