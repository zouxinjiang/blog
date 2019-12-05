---
title: goroutine  协程
date: 2019-12-13 00:00:00

categories:
    - code

tags:
    - golang
---
# go routine  协程
goroutine 是go中类似线程的一种轻量级可执行单元。是go中做并发处理
的一种强大的数据结构。go将goroutine作为自己的一部分，所以go中开启
并发代码非常简单，只需要一个go 关键字即可。
```go
func main() {
    
    go func() {
    	fmt.Println("子程序")
    }
}
```

## chan 信道
chan用于goroutine之间的数据通信，同步控制等。信道是带有类型的。
chan的零值为nil
```go
var ch chan int
var ch = make(chan int)
var ch = make(chan int,1) // 创建能缓存一个数据的信道
var ch chan<- int // 只能写的信道
var ch <-chan int // 只能读的信道
```
### 使用
```go
a := <-ch  //读取数据，如果没有数组则阻塞
a,ok := <-ch // 如果ch被close掉的话,则ok=false
ch <- a // 写入数据，如果ch没有缓存功能且没有消费者读取
// 或者缓存已满，则阻塞

close(ch)  // 关闭信道
for i := range ch {  // 读取数据，直到ch被close，否则会阻塞
}
// 阻塞式
select {
    case <-ch:
        // todo ch数据可读则执行
    case <- otherCh:
        // todo  otherCh数据可读则执行
}
// 非阻塞式
select {
    case <-ch:
        // todo ch数据可读则执行
    case <- otherCh:
        // todo  otherCh数据可读则执行
    default: // 如果select没有default分支，则阻塞式读取
        // todo  没有信道可读
}

```
如果对一个nil的信道写入数据，会死锁。如果对一个close掉的chan进行
写入会panic.

## sync.Mutex 互斥
用于goroutine数据间互斥控制。共有两个方法
- Lock
- Unlock

sync.RWMutex是带有读写互斥性质的
- RLock
- RUnlock
- Lock
- Unlock

```go
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```