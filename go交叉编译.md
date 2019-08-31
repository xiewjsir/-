##### 安装 MinGW
[下载地址](https://osdn.net/projects/mingw/releases/)

将XXX\MinGW\bin加入环境变量

##### 编写make.bat脚本
```
set GOPATH=D:\BaseOsGo
set GOARCH=386
set GOOS=linux
go build
pause
```