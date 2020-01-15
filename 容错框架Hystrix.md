##### 限流器
> 限流器，从字面上理解就是用来限制流量，有时候流量突增(可预期的比如“双11”，不可预期的微博的热门话题等)，会将后端服务压垮，甚至直接宕机，使用限流器能限制访问后端的流量，起到一个保护作用，被限制的流量，可以根据具体的业务逻辑去处理，直接返回错误或者返回默认值等等
golang 提供了拓展库(golang.org/x/time/rate)提供了限流器组件，用法上也很简单直观，通过下面这段代码就可以创建一个限流器
```
// 每 800ms 产生 1 个 token，最多缓存 1 个 token，如果缓存满了，新的 token 会被丢弃
limiter := rate.NewLimiter(rate.Every(time.Duration(800)*time.Millisecond), 1)
```
限流器提供三种使用方式，Allow, Wait, Reserve
- Allow: 返回是否有 token，没有 token 返回 false，或者消耗 1 个 token 返回 true
- Wait: 阻塞等待，知道取到 1 个 token
- Reserve: 返回 token 信息，Allow 其实相当于 Reserve().OK，此外还会返回需要等待多久才有新的 token

一般使用 Wait 的场景会比较多一些
```
if err := limiter.Wait(context.Background()); err != nil {
    panic(err)
}

// do you business logic
```
##### 熔断器
和限流器对依赖服务的保护机制不一样，熔断器是当依赖的服务已经出现故障时，为了保证自身服务的正常运行不再访问依赖的服务，防止雪崩效应

熔断器有三种状态：
- 关闭状态：服务正常，并维护一个失败率统计，当失败率达到阀值时，转到开启状态
- 开启状态：服务异常，调用 fallback 函数，一段时间之后，进入半开启状态
- 半开启装态：尝试恢复服务，失败率高于阀值，进入开启状态，低于阀值，进入关闭状态
github.com/afex/hystrix-go，提供了 go 熔断器实现，使用上面也很方便，首先创建一个熔断器

##### 参考文档
- [微服务组件之限流器与熔断器](https://studygolang.com/articles/13254)

```
package main

import (
	"fmt"
	"time"

	"github.com/afex/hystrix-go/hystrix"
	"github.com/grpc-ecosystem/go-grpc-middleware/retry"
	addservice "github.com/hatlonely/hellogolang/sample/addservice/api"
	"github.com/hatlonely/hellogolang/sample/addservice/internal/grpclb"
	"golang.org/x/net/context"
	"golang.org/x/time/rate"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
)

func main() {
	conn, err := grpc.Dial(
		"",
		grpc.WithInsecure(),
		// 开启 grpc 中间件的重试功能
		grpc.WithUnaryInterceptor(
			grpc_retry.UnaryClientInterceptor(
				grpc_retry.WithBackoff(grpc_retry.BackoffLinear(time.Duration(1)*time.Millisecond)), // 重试间隔时间
				grpc_retry.WithMax(3),                                             // 重试次数
				grpc_retry.WithPerRetryTimeout(time.Duration(5)*time.Millisecond), // 重试时间
				// 返回码为如下值时重试
				grpc_retry.WithCodes(codes.ResourceExhausted, codes.Unavailable, codes.DeadlineExceeded),
			),
		),
		// 负载均衡，使用 consul 作服务发现
		grpc.WithBalancer(grpc.RoundRobin(grpclb.NewConsulResolver(
			"127.0.0.1:8500", "grpc.health.v1.add",
		))),
		// grpc.WithBalancer(grpc.RoundRobin(grpclb.NewPseudoResolver(
		// 	[]string{
		// 		"127.0.0.1:3000",
		// 		"127.0.0.1:3001",
		// 		"127.0.0.1:3002",
		// 	},
		// ))),
	)
	if err != nil {
		fmt.Printf("dial failed. err: [%v]\n", err)
		return
	}
	defer conn.Close()

	client := addservice.NewAddServiceClient(conn)

	// 限流器
	// 每 800ms 产生 1 个 token，最多缓存 1 个 token，如果缓存满了，新的 token 会被丢弃
	limiter := rate.NewLimiter(rate.Every(time.Duration(800)*time.Millisecond), 1)

	// 熔断器
	hystrix.ConfigureCommand(
		"addservice", // 熔断器名字，可以用服务名称命名，一个名字对应一个熔断器，对应一份熔断策略
		hystrix.CommandConfig{
			Timeout:                100,  // 超时时间 100ms
			MaxConcurrentRequests:  2,    // 最大并发数，超过并发返回错误
			RequestVolumeThreshold: 4,    // 请求数量的阀值，用这些数量的请求来计算阀值
			ErrorPercentThreshold:  25,   // 错误率阀值，达到阀值，启动熔断，25%
			SleepWindow:            1000, // 熔断尝试恢复时间，1000ms
		},
	)

	for i := int64(1); i < 10; i++ {
		for j := int64(1); j < 10; j++ {
			// 限流
			if err := limiter.Wait(context.Background()); err != nil {
				panic(err)
			}

			{
				// 熔断，阻塞方式调用
				var res *addservice.AddResponse
				req := &addservice.AddRequest{A: i, B: j}
				err := hystrix.Do("addservice", func() error {
					// 正常业务逻辑，一般是访问其他资源
					var err error
					// 设置总体超时时间 10 ms 超时
					ctx, cancel := context.WithTimeout(context.Background(), time.Duration(50*time.Millisecond))
					defer cancel()
					res, err = client.Add(
						ctx, req,
						// 这里可以再次设置重试次数，重试时间，重试返回码
						grpc_retry.WithMax(3),
						grpc_retry.WithPerRetryTimeout(time.Duration(5)*time.Millisecond),
						grpc_retry.WithCodes(codes.DeadlineExceeded),
					)
					return err
				}, func(err error) error {
					// 失败处理逻辑，访问其他资源失败时，或者处于熔断开启状态时，会调用这段逻辑
					// 可以简单构造一个response返回，也可以有一定的策略，比如访问备份资源
					// 也可以直接返回 err，这样不用和远端失败的资源通信，防止雪崩
					// 这里因为我们的场景太简单，所以我们可以在本地在作一个加法就可以了
					fmt.Println(err)
					res = &addservice.AddResponse{V: req.A + req.B}
					return nil
				})

				// 事实上这个断言永远为假，因为错误会触发熔断调用 fallback，而 fallback 函数返回 nil
				if err != nil {
					fmt.Printf("client add failed. err: [%v]\n", err)
				}

				fmt.Println(req, res)
			}

			{
				// 熔断，非阻塞方式调用
				var res1 *addservice.AddResponse
				var res2 *addservice.AddResponse
				req := &addservice.AddRequest{A: i, B: j}
				success := make(chan struct{}, 2)

				// 无 fallback 处理
				errc1 := hystrix.Go("addservice", func() error {
					var err error
					ctx, cancel := context.WithTimeout(context.Background(), time.Duration(50*time.Millisecond))
					defer cancel()
					res1, err = client.Add(ctx, req)
					if err == nil {
						success <- struct{}{}
					}
					return err
				}, nil)

				// 有 fallback 处理
				errc2 := hystrix.Go("addservice", func() error {
					var err error
					ctx, cancel := context.WithTimeout(context.Background(), time.Duration(50*time.Millisecond))
					defer cancel()
					res2, err = client.Add(ctx, req)
					if err == nil {
						success <- struct{}{}
					}
					return err
				}, func(err error) error {
					fmt.Println(err)
					res2 = &addservice.AddResponse{V: req.A + req.B}
					success <- struct{}{}
					return nil
				})

				for i := 0; i < 2; i++ {
					select {
					case <-success:
						fmt.Println("success", i)
					case err := <-errc1:
						fmt.Println("err1:", err)
					case err := <-errc2:
						// 这个分支永远不会走到，因为熔断机制里面永远不会返回错误
						fmt.Println("err2:", err)
					}
				}

				fmt.Println(req, res1, res2)
			}
		}
	}
}
```
