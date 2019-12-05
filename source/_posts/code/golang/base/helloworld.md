---
title: hello world
date: 2019-12-04 00:00:00

categories:
    - code

tags:
    - golang
---
# hello world
我们好像学习编程语言的第一个程序都是hello world，那我们看看go语言的hello world呢
```go
package main
import (
"fmt"
)

func main() {
    fmt.Println("hello world")
}
```
## 结构
- package main 包名
- import xxx 此包中要用到的其他包
- func main()  main函数，程序入口
