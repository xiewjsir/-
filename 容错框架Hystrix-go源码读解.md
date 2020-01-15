##### 定义
>下文是github上go-hystrix对与其功能的阐述，其脱胎于java版本的Hystrix库，主要目的是为了解决分布式系统中对于错误的保护，这一点从其熔断的定义也可以明白。

>Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.
>I think the Hystrix patterns of programmer-defined fallbacks and adaptive health monitoring are good for any distributed system. Go routines and channels are great concurrency primitives, but don't directly help our application stay available during failures.
>hystrix-go aims to allow Go programmers to easily build applications with similar execution semantics of the Java-based Hystrix library.

- command config定义
```
type CommandConfig struct {
    Timeout                int `json:"timeout"` // 超时时间定义
    MaxConcurrentRequests  int `json:"max_concurrent_requests"` // 最大并发请求数
    RequestVolumeThreshold int `json:"request_volume_threshold"` // 跳闸的最小请求数（不健康的断路器）
    SleepWindow            int `json:"sleep_window"` // 跳闸之后可以重试的时间
    ErrorPercentThreshold  int `json:"error_percent_threshold"` // 请求出错比
}
```

- 断路器的定义
```
// CircuitBreaker is created for each ExecutorPool to track whether requests
// should be attempted, or rejected if the Health of the circuit is too low.
type CircuitBreaker struct { // 断路器-定义
    Name                   string // 名字
    open                   bool   // 开启与否，关闭"open"=true，开启"open" = false
    forceOpen              bool   // 强制开启
    mutex                  *sync.RWMutex // 读写锁（unblock reading, block writer）
    openedOrLastTestedTime int64 // 断路器被打开或者最近一次尝试的时间，尝试指断路器打开之后，系统探测是否可以发送请求。
    executorPool *executorPool // 执行池
    metrics      *metricExchange // 监控断路器
}
var (
    circuitBreakersMutex *sync.RWMutex // 断路器锁
    circuitBreakers      map[string]*CircuitBreaker // 注册断路器，所有的断路器都保存在这里
)
func init() {
    circuitBreakersMutex = &sync.RWMutex{} // 初始化断路器锁
    circuitBreakers = make(map[string]*CircuitBreaker) // 初始化断路器
}
```

- 获取断路器
```
// GetCircuit returns the circuit for the given command and whether this call created it.
func GetCircuit(name string) (*CircuitBreaker, bool, error) {
    circuitBreakersMutex.RLock() 
    _, ok := circuitBreakers[name]
    if !ok {
        circuitBreakersMutex.RUnlock()
        circuitBreakersMutex.Lock()
        defer circuitBreakersMutex.Unlock() // 注意这里同时加了两次🔒且第二把锁是互斥锁，其中一个goroutine hold住并且赋值，锁释放。其他goroutine从内存中获取断路器
        if cb, ok := circuitBreakers[name]; ok { // double check，防止其他的goroutine修改了全局变量circuitBreakers
            return cb, false, nil
        }
        circuitBreakers[name] = newCircuitBreaker(name)
    } else {
        defer circuitBreakersMutex.RUnlock()
    }
    return circuitBreakers[name], !ok, nil
}
```

- 执行池
```
type executorPool struct { // 执行池
    Name    string  // 名字
    Metrics *poolMetrics // 执行池监控
    Max     int // 最大的并发请求数量
    Tickets chan *struct{} // 票证
}
// 开启一个新的执行池
func newExecutorPool(name string) *executorPool {
    p := &executorPool{}
    p.Name = name // 名字
    p.Metrics = newPoolMetrics(name) 
    p.Max = getSettings(name).MaxConcurrentRequests // 从配置中获取最大的并发请求数量，如果配置中没有，则从默认配置中获取
    p.Tickets = make(chan *struct{}, p.Max) // 初始化buffer chan
    for i := 0; i < p.Max; i++ {
        p.Tickets <- &struct{}{}
    }
    return p
}
func (p *executorPool) Return(ticket *struct{}) {
    if ticket == nil {
        return
    }
    p.Metrics.Updates <- poolMetricsUpdate{
        activeCount: p.ActiveCount(),
    }
    p.Tickets <- ticket
}
func (p *executorPool) ActiveCount() int {
    return p.Max - len(p.Tickets)
}
```

- 执行函数入口GO，传递进去自定义的run，fallback函数，这个函数会封装ctx，然后继续调用GoC。
```
func Go(name string, run runFunc, fallback fallbackFunc) chan error {
    runC := func(ctx context.Context) error { // 匿名run带ctx
        return run()
    }
    var fallbackC fallbackFuncC
    if fallback != nil {
        fallbackC = func(ctx context.Context, err error) error { // 匿名fallback带ctx
            return fallback(err)
        }
    }
    return GoC(context.Background(), name, runC, fallbackC)
}
```

- 主要的执行逻辑GoC,提供了异步的执行。
```
func GoC(ctx context.Context, name string, run runFuncC, fallback fallbackFuncC) chan error {
    cmd := &command{ // command执行者
        run:      run, // run
        fallback: fallback, // fallback
        start:    time.Now(), // 开始时间
        errChan:  make(chan error, 1), // 错误
        finished: make(chan bool, 1), // 是否完成
    }
    // dont have methods with explicit params and returns
    // let data come in and out naturally, like with any closure
    // explicit error return to give place for us to kill switch the operation (fallback)
    circuit, _, err := GetCircuit(name) // 获取断路器
    if err != nil {
        cmd.errChan <- err
        return cmd.errChan
    }
    cmd.circuit = circuit
    ticketCond := sync.NewCond(cmd) // cond条件，具体使用见下文参考
    ticketChecked := false
    returnTicket := func() { // 
        cmd.Lock()
        // Avoid releasing before a ticket is acquired.
        for !ticketChecked {
            ticketCond.Wait() // 相当于select{}
        }
        cmd.circuit.executorPool.Return(cmd.ticket) // 将ticket放回池子中
        cmd.Unlock()
    }
    returnOnce := &sync.Once{} // 确保被multi goroutine执行一次
    reportAllEvent := func() { // events采集，后续dashboard使用
        err := cmd.circuit.ReportEvent(cmd.events, cmd.start, cmd.runDuration)
        if err != nil {
            log.Printf(err.Error())
        }
    }
    // g1, 检测断路器不允许通过，尝试fallback，将中途遇到的event上报。
    go func() {
        defer func() { cmd.finished <- true }()
        if !cmd.circuit.AllowRequest() {
            cmd.Lock()
            // It's safe for another goroutine to go ahead releasing a nil ticket.
            ticketChecked = true
            ticketCond.Signal()
            cmd.Unlock()
            returnOnce.Do(func() {
                returnTicket()
                cmd.errorWithFallback(ctx, ErrCircuitOpen)
                reportAllEvent()
            })
            return
        }
        cmd.Lock()
        select {
        case cmd.ticket = <-circuit.executorPool.Tickets: // 从池子中取出ticket
            ticketChecked = true
            ticketCond.Signal() // 通知cond.Wait()
            cmd.Unlock()
        default:
            ticketChecked = true
            ticketCond.Signal()
            cmd.Unlock()
            returnOnce.Do(func() {
                returnTicket()
                cmd.errorWithFallback(ctx, ErrMaxConcurrency) // 并发过高
                reportAllEvent()
            })
            return
        }
        runStart := time.Now()
        runErr := run(ctx)
        returnOnce.Do(func() {
            defer reportAllEvent()
            cmd.runDuration = time.Since(runStart) // 运行时间
            returnTicket() // 把ticket返回去
            if runErr != nil {
                cmd.errorWithFallback(ctx, runErr) // fall back！
                return
            }
            cmd.reportEvent("success") // 执行成功
        })
    }()
    go func() {
        timer := time.NewTimer(getSettings(name).Timeout)
        defer timer.Stop()
        select {
        case <-cmd.finished: // 结束select
            // returnOnce has been executed in another goroutine
        case <-ctx.Done(): // 处理ctx错误
            returnOnce.Do(func() {
                returnTicket()
                cmd.errorWithFallback(ctx, ctx.Err())
                reportAllEvent()
            })
            return
        case <-timer.C: // 处理超时
            returnOnce.Do(func() {
                returnTicket()
                cmd.errorWithFallback(ctx, ErrTimeout)
                reportAllEvent()
            })
            return
        }
    }()
    return cmd.errChan // 错误返回至上层
}
```

- 同步的执行方式，如果需要。
```
func Do(name string, run runFunc, fallback fallbackFunc) error {
    runC := func(ctx context.Context) error {
        return run()
    }
    var fallbackC fallbackFuncC
    if fallback != nil {
        fallbackC = func(ctx context.Context, err error) error {
            return fallback(err)
        }
    }
    return DoC(context.Background(), name, runC, fallbackC) // 具体逻辑如上GoC
}
```

##### 原文
- [go-hystrix 源码读解](https://www.zybuluo.com/aliasliyu4/note/1248898)
