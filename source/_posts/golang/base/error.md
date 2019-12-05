---
title: error 错误
date: 2019-12-14 00:00:00

categories:
    - code

tags:
    - golang
---
# error 错误
go中的错误error是一个接口类型,只要实现了 func Error() string 方法
就能够当成错误值。

## panic
- panic(v interface{}) 直接使程序崩溃，并打出堆栈信息，这个一般在
程序中比较少用，除非你确定要使程序崩溃

```go
panic(err)
```
## recover
- recover() v interface{} 连同defer一起使用，函数捕获程序中的panic，并使panic不会引起
程序崩溃，而是将panic的内容作为recover的返回值。这个在http server中
很常见，目的就是防止程序错误导致服务崩溃。
```go
func main() {
    defer func (){
        if v := recover(); v := nil {
            // todo  程序panic处理逻辑
        }   
    }
}
```
