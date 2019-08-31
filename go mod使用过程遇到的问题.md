###### 问题一
golang.org 这个网站打不开，设置https://goproxy.io无效的情况下，可以通过替换github地址来下载mod

```
#解决方法
replace  (
    golang.org/x/text => github.com/golang/text latest
    golang.org/x/net => github.com/golang/net latest
    golang.org/x/crypto => github.com/golang/crypto latest
    golang.org/x/tools => github.com/golang/tools latest
    golang.org/x/sync => github.com/golang/sync latest
    golang.org/x/sys => github.com/golang/sys latest
    cloud.google.com/go => github.com/googleapis/google-cloud-go latest
    google.golang.org/genproto => github.com/google/go-genproto latest
    golang.org/x/exp => github.com/golang/exp latest
    golang.org/x/time => github.com/golang/time latest
    golang.org/x/oauth2 => github.com/golang/oauth2 latest
    golang.org/x/lint => github.com/golang/lint latest
    google.golang.org/grpc => github.com/grpc/grpc-go latest
    google.golang.org/api => github.com/googleapis/google-api-go-client latest
    google.golang.org/appengine => github.com/golang/appengine latest
    golang.org/x/mobile => github.com/golang/mobile latest
    golang.org/x/image => github.com/golang/image latest
    
    k8s.io/utils => github.com/kubernetes/utils  latest
    k8s.io/client-go => github.com/kubernetes/client-go  latest
    k8s.io/klog => github.com/kubernetes/klog  latest
    k8s.io/api => github.com/kubernetes/api  latest
    k8s.io/apimachinery => github.com/kubernetes/apimachinery  latest
)
```

##### 问题二
```
build command-line-arguments: cannot load github.com/hashicorp/consul/api: ambiguous import: found github.com/hashicorp/consul/api in multiple modules:
        github.com/hashicorp/consul v1.4.2 (D:\BaseOsGo\pkg\mod\github.com\hashicorp\consul@v1.4.2\api)
        github.com/hashicorp/consul/api v1.1.0 (D:\BaseOsGo\pkg\mod\github.com\hashicorp\consul\api@v1.1.0)

#解决方法
replace github.com/golang/lint => golang.org/x/lint v0.0.0-20190409202823-959b441ac422

replace github.com/testcontainers/testcontainer-go => github.com/testcontainers/testcontainers-go v0.0.5

replace github.com/hashicorp/consul => github.com/hashicorp/consul v1.6.0
```

##### 参考文档
- [goLang开发环境配置：go mod使用](https://studygolang.com/articles/20805)
