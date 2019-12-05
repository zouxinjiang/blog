---
title: 流程控制
date: 2019-12-09 00:00:00

categories:
    - code

tags:
    - golang
---
# 流程控制

## for循环
go 只有for循环，没有其他类型的循环
```go
// 死循环
for {
}

// 类c for循环
for i:=0;i<10;i++ {
}

for ;i<10; {
}

// while循环
for i < 10 {
    i++
}
```

- break 跳出当前循环
- continue 结束此次循环进入下一次循环

## if
```go
if 条件 {
}

if 语句; 条件 {
}

if 条件 { 
}else{
}

if 条件 {
} else if 条件 {
} else{
}
```

## switch
switch相对于C中的switch语法糖好用得多。而且go的每个case后相当于
有一个break，所以switch只会有一个case执行到，如果想让从第一个符合
条件的case后的case也执行，则需要关键字 fallthrough.
```go
switch a {
case 1:
    //todo
}

switch a {
case 1:
    //todo
default:
    //todo
}

switch a {
case 1,2,3:
    //todo
default:
    //todo
}

// 当成if else if来用
switch {
case a = 1:
    // todo
default:
    // todo
}
```

## defer
延迟执行，defer的动作会延迟到函数返回时被执行。在一个函数中
多次使用defer，遇到一个defer则将其压入栈中，直到函数return，
然后再从栈中依次执行。由于使用的是栈，所以先defer的要比后defer
的后执行。

```go
func main(){
    defer fmt.Println("a")
    defer fmt.Println("b")
}
// 最后输出为 b，a
```

