##### sync.Mutex
- 在Lock()和Unlock()之间的代码段称为资源的临界区(critical section)，在这一区间内的代码是严格被Lock()保护的，是线程安全的，任何一个时间点都只能有一个goroutine执行这段区间的代码。
- Lock()保证了在某一时间点只有其中一个goroutine能访问其中一个critical section。当释放了一个critical section，其它的Lock()将争夺互斥锁，也就是所谓的竞争现象(race condition)。因为竞争的存在，这42个critical section被访问的顺序是随机的，完全无法保证哪个critical section先被访问。
##### sync.RWMutex
- RWMutex是基于Mutex的，在Mutex的基础之上增加了读、写的信号量，并使用了类似引用计数的读锁数量
- 读锁与读锁兼容，读锁与写锁互斥，写锁与写锁互斥，只有在锁释放后才可以继续申请互斥的锁：
    - 可以同时申请多个读锁
    - 有读锁时申请写锁将阻塞，有写锁时申请读锁将阻塞
    - 只要有写锁，后续申请读锁和写锁都将阻塞
```
func (rw *RWMutex) Lock()
func (rw *RWMutex) RLock()
func (rw *RWMutex) RLocker() Locker
func (rw *RWMutex) RUnlock()
func (rw *RWMutex) Unlock()
```
其中：
- Lock()和Unlock()用于申请和释放写锁
- RLock()和RUnlock()用于申请和释放读锁
- 一次RUnlock()操作只是对读锁数量减1，即减少一次读锁的引用计数
- 如果不存在写锁，则Unlock()引发panic，如果不存在读锁，则RUnlock()引发panic
RLocker()用于返回一个实现了Lock()和Unlock()方法的Locker接口

此外，无论是Mutex还是RWMutex都不会和goroutine进行关联，这意味着它们的锁申请行为可以在一个goroutine中操作，释放锁行为可以在另一个goroutine中操作。
由于RLock()和Lock()都能保证数据不被其它goroutine修改，所以在RLock()与RUnlock()之间的，以及Lock()与Unlock()之间的代码区都是critical section。
##### 参考文档
- [Go基础系列：互斥锁Mutex和读写锁RWMutex用法详述](https://www.cnblogs.com/f-ck-need-u/p/9998729.html)