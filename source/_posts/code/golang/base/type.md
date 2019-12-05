---
title: type 类型
date: 2019-12-06 00:00:00

categories:
    - code

tags:
    - golang
---
# type 类型

## 基本类型
```go
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128
```

### 零值
```text
数值类型为 0，
布尔类型为 false，
字符串为 ""（空字符串）。
```

### 类型转化
T(v) 将v转化成T类型

## 指针
go语言的指针没有什么太难的，指针操作只有两个 & 取地址 * 取值

## 复合类型

### 结构体 struct 
go的结构体是C语言的结构体的扩展。详细请参考struct章节
```go
type 名字 struct{
    字段名列表 字段类型  `tag`
    ...
}
```
#### 特点

1. 能够绑定方法
2. 指针和值 变量调用方法时都是使用 . ；不像C++使用 . 和 -> 两种方式
3. 支持命名嵌入和匿名嵌入(类似面向对象的继承)
4. 结构体初始化时可以使用 name:value的方式指定初始化的字段

### 数组 很少用
go语言的数组大小是固定的，不可变长度，所以很少用，用得最多的是切片
```go
var arr [2]int
var arr = [2]int{}
var aarr [2][3]int // 两行三列

[2]int 和 [3]int 是两种完全不同的类型
```
#### 使用
使用方式基本和C的数组一样

### 切片
切片是对数组的引用，切片提供了动态变化大小、比数组更加灵活。
[]T 直接就表示T类型的切片。切片不会存储任何数据，数据都依赖
于底层数组，如果切片直接引用的你指定的数组，那么你指定的那个
数组就是切片依赖的底层数组，如果你直接声明并初始化切片，那么
底层数组是系统生成的，你只是看不到而已。切片的零值为nil，没有
底层数组

```go
var s []int
var s = []int{}
var s = make([]int,0,10) // 第二个参数是len，第三个参数是cap
// 引用数组的一部分
var a = [2]int{1,2}
var s = a[:]
```

#### 使用
切片的使用方式是数组使用当时的超集。除此之外还有，
```go
var arr = [100]int{}

var s = arr[1:20] // [1,20)
var s = arr[:10] // [0,10)
var s = arr[10:] // [10,100]
var s = arr[:] // [0,100]

len(s)  // 获取切片数据长度
cap(s)  // 获取切片分配的存储空间的长度，切片是变长的，
 // 变长是要重新申请内存的，所以有一定的增长机制，
// 不需要每次增长,都去申请内存，所以cap是代表当前一共申请了
// 多少个存储单元

s = append(s,1,2,3) // 向切片追加元素

// for遍历切片 i 索引 v 值
for i,v := range s {}
for i := range s {}
for _,v := range s{}
```

### map 映射
go中的键值对结构，零值为nil，nil类型的map没有键也不能添加键

```go
var m map[string]int
var m = map[string]int{}
var m = make(map[string]int)
```

#### 使用
- 添加键
    ```go
    m[key] = value
    ```
- 获取
    ```go
    a := m[key]
    a,ok := m[key]  // ok布尔值，值为该map中是否含key这个键
    ```
- 删除
    ```go
    delete(m,key)
    ```
- 键数量
    ```go
    len(m)
    ```
### 函数值
在go中，函数也是值，所以函数能够放在任何值能放的地方。

#### 闭包
带有环境的函数值就是闭包。它可能引用了函数外的变量，所以它需要依赖
它所处的环境，将环境和函数一起打包成一个值，就成了闭包。
```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
    // return的这个函数引用了sum这个变量
    //  而sum不属于return这个函数，所以return的这个函数
    // 依赖了adder的环境(sum)，那个return的这个函数和sum一起
    // 被打包返回了
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

