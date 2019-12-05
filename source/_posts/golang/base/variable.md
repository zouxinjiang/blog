---
title: 变量，常量
date: 2019-12-07 00:00:00

categories:
    - code

tags:
    - golang
---
# 变量，常量

## var 变量
### 声明
语法糖：
```go
var a int
var a = 1
var a int = 1
var x,y int
var x,y int = 0,1
var(
    b = 1
    c int = 2
    d int
    e,f string
)
// 局部变量还有一种声明方式 := 使用这种方式则左表达式
// 必须有至少一个变量是未声明的
a := 1
a,b,c := 1,'e',"sfdsafd"
a,b,c := 返回三个值的函数调用
```
1. 全局变量
    - 包内全局变量。在包级别直接使用var定义的变量
    - 不存在包外全局变量，因为go的所有东西都定义在包内
    ```go
    package main
    var (
        a = 1
    )
    
    func main(){
    }
    ```
2. 局部变量
    - 在代码区域内声明的变量，只在某区域内有效
    ```go
    package main
    import (
    "fmt"
    )
    func main() {
       var a = 1 // 在main函数内有效
       for {
           var b = 2 // 只在for循环内有效
           fmt.Println(b)
           break
       }
    fmt.Println(a)       
   }
    ```

### 特点

1. 类型放在名字后面
2. 类型可推导
3. 类型相同的可放在一起简写
4. 使用 _ 忽略掉不需要的变量

## const 常量

```go
const a = 1
const a int = 2
const (
    a = 1
    b = 2
)
const (
    a = iota  //0
    b // 1
    c // 2
)

const (
    a = iota // 0
    b // 1
    c // 2
    d = iota+1000 // 1000
    e // 1001
    f // 1002
    x = 1 << iota // 1
    y // 2
    z // 4
)
```
