##### 背景
>Go项目有很多命令，且带有参数，可能我们平时开发的时候最常用的就是 go build 、go run 但是当我们使用其他命令时，通常就会携带一堆参数， 比如 go test -v -count=1 - ... 这个时候一个构建脚本可以解决问题，一段Makefile的规则也能解决问题。我个人更加倾向Makefile+script去解决复杂的事情。

##### Makefile文件格式
一个简单的Makefile应该是如下规则格式
```
target ... : prerequisites ...
    recipe
    ...
    ...

// 直观一点的 - tab很重要
<target> : <prerequisites> 
[tab]  <commands>
```
执行目标，处理先决条件，然后做剩下的事情
```
install:
    echo installing

build: install
    go build -o main main.go
```
当执行 make build 其实就是执行了 install-build 下面的命令

##### 常用符号解释
- target: 即自定义的自己想要执行的命令
- prerequisites: 先决条件，执行在target之前，多个用空格隔开
- tab: 每一个command前面一定要有制表符，用于标示这是命令
- commands: 具体执行的命令。
- .PHONY 伪指令，主要作用是命名避免与同名文件冲突和提高性能，加上后此参数，当存在冲突文件，也会执行 target, 否则会报错。
- 不带执行参数，默认执行第一个target
- 在command前加上@，禁止打印出command执行语句，只显示执行结果
- ${val}表示变量，与shell一样使用
- 允许使用通配符
- ‘#’表示注释

##### 简单示例
```
# 二进制文件名
PROJECT="main"
# 启动文件PATH
MAIN_PATH="main.go"
VERSION="v0.0.1"
DATE= `date +%FT%T%z`

# 可以执行函数，但是函数不能存在 command
ifeq (${VERSION}, "v0.0.1") 
    VERSION=VERSION = "v0.0.1"
endif

version:
    @echo ${VERSION}

# .PHONY 有 build 文件，不影响 build 命令执行
.PHONY: build
build:
    @echo version: ${VERSION} date: ${DATE} os: Mac OS
    @go  build -o ${PROJECT} ${MAIN_PATH}

install:
    @echo download package
    @go mod download

# 交叉编译运行在linux系统环境
build-linux:
    @echo version: ${VERSION} date: ${DATE} os: linux-centOS
    @GOOS=linux go build -o ${PROJECT} ${MAIN_PATH}

run:   build
    @./${PROJECT}

clean:
    rm -rf ./log
```

##### 更多用法请参考文档
###### 注意事项
- *** missing separator. Stop. 错误解决方案：
> Makefile 文件，执行命令前面需要用 Tab 制表符标示，空格等无效。如果使用编辑器，注意编辑器设置 Tab 按键的解析，是制表符还是空格。

##### 文档
- [原文](https://studygolang.com/articles/20704)