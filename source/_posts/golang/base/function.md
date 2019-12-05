---
title: function 函数
date: 2019-12-10 00:00:00

categories:
    - code

tags:
    - golang
---
# function 函数

## 结构
```go
func 函数名(参数) 返回值 {
    函数体
}
```
## 特点
1. 多返回值
2. 返回值可命名
2. 函数使一等公民，完全当成变量使用

## 示例
```go
func add(a int,b int) int {
    return a+b
}

func add(a,b int) int {
    return a+b
}

func add(a,b int) (c int) {
    return a+b
    //c = a+b
    //return c
    //return 
}

func swap(a,b int) (int,int) {
    return b,a
}

// 一等公民， 函数可以当作变量作为参数，字段，返回值
func sum(a,b int){
    fmt.Println(a+b)
}
func do(a,b int,fn func(int,int))

func main() {
    do(1,2,sum)
}
```