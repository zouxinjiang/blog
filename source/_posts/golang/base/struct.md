---
title: struct 结构体
date: 2019-12-11 00:00:00

categories:
    - code

tags:
    - golang
---
# struct 结构体

## 方法
绑定了类型（接收者）的函数就叫方法，方法就是函数，跟普通函数没有区别，可以作为
参数，值，返回值等。

```go
type Vertex struct {
	X, Y float64
}
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

type Int int 
func (i Int)Show(){
    fmt.Println(i)
}
```
## 接收者
就是处于func和方法名之间的那个括号中的部分。
- 值接收者
    - 修改接收者的值只会影响在方法作用域内的接收者变量
- 指针接收者
    - 修改接收者的值不仅会影响在方法作用域内的接收者的值，
    还会影响到直接的接收者

把接收者当成函数的一个参数你就能明白他们之间的差别了。如果传的
是一个变量，则函数体内对变量进行任何修改都仅仅在函数体内有效，而
如果是传入的指针，那么修改变量值除了在函数体内有效，还会影响到指针
所指的那个变量的值。
- 什么时候用值接收者，什么时候用指针接收者呢？
    - 当方法内修改接收者值的时候要作用到该方法作用域外的地方或者
    结构体比较大的时候就要用指针接收者，结构体比较大的时候用指针
    接收者是为了效率，避免copy结构体带来的开销。

```go
type Abc struct{
	A int
    B int
    Res int
}
func (x Abc)Add() { // 值接收者
    x.Res = x.a+x.b
}

func (x *Abc)Sub(){ // 指针接收者
    x.Res = x.a - x.b
}

func FAdd(x Abc) {  // 传值函数
    x.Res = x.a + x.b
}

func FSub(x *Abc){ // 传址函数
    x.Res = x.a - x.b
}

func main() {
    var ma = Abc{
        A:2,
        B:1,
    }
    var fa = Abc{
         A:2,
         B:1,
    }
    FAdd(fa)
    fmt.Println("fadd:",fa.Res) // 0 因为是值，所以函数内修改
    // 不会影响到函数作用域外的地方
    FSub(&fa)
    fmt.Println("fsub:",fa.Res) // 1 因为是地址，所以函数内
    // 修改会影响到这个地方的fa
    ma.Add()
    fmt.Println("add:",ma.Res) // 0 值接收者，同FAdd
    ma.Sub()
    fmt.Println("sub",ma.Res) // 1 指针接收者，同FSub
}
```
虽然FAdd函数完全等价于Add方法，FSub函数完全等价于Sub方法，但是
从调用语法糖上就看出区别了，如果是函数，则需要区分到底是值还是指针
而接收者方式完全就是用 变量名.方法名 的方式进行，不需要区分值和指针
    
## 结构体内嵌
如果想让结构体A能够使用结构体B的东西，那么在没有说内嵌之前的做法是
在A内弄一个字段，该字段的类型就是B类型，然后结构体A就拥有了B结构体
的能力了。那么要用结构体B的能力，我们就需要 “A.B.b” 这种形式，go中
为了在使用时简洁，那么就期望使用的形式为 “A.b” ，所以内嵌就产生了，
内嵌仅仅就是解决形式上的问题，其他地方和在A里面弄一个字段为B类型的
方式是一模一样的。而内嵌会在结构体A上产生一个字段名叫B的字段，类型
就是B，当要对A进行初始化时，是要初始化内部B的。

1. 结构体内嵌接口
    ```go
    type IA interface {
    	Say()
    }
    
    type S struct {
    	IA
    }
    ```
2. 结构体内嵌结构体
    ```go
    type R struct {
        VR string
    }
    
    func (R) Abc() {}
    
    type S struct {
        R
    }
    ```
```go
// 作为字段
type A struct{
    VA1 int
    VA2 string
}

func (a A) ShowA1(){
    fmt.Println("A.VA1:",a.VA1)
}

type B struct {
    VB1 int
    VB2 string
    VA A
}

func (b B) ShowB1(){
    fmt.Println("B.VB1:",b.VB1)
}
// 使用
b := B{
    VA:A{}
}

b.ShowB1()
b.VA.ShowA1()
b.VA.VA1
b.VA.VA2

// 内嵌
type A struct{
    VA1 int
    VA2 string
}

func (a A) ShowA1(){
    fmt.Println("A.VA1:",a.VA1)
}

func (a A)ShowA2(){
	fmt.Println("A.VA2:",a.VA2)
}

type B struct {
    VB1 int
    VB2 string
    A
}

func (b B) ShowB1(){
    fmt.Println("B.VB1:",b.VB1)
}

func (b B)ShowA2() {
    fmt.Println("B.VA2:",b.VA2)
}

// 使用
b := B{
    A:A{}
}

b.ShowB1()
b.ShowA1()
b.VA.ShowA1()
b.VA1
b.VA.VA1
b.VA2
b.VA.VA2

b.ShowA2() // 会调 B最直接的那个ShowA2
// 如果b没有ShowA2()那个方法，则使用b.ShowA2()会调用A的ShowA2()
b.A.ShowA2() // 调的是A的ShowA2()

// 如果结构体A内嵌入了多个结构体，而这些结构体有相同的方法怎么办？ 
// 1 不要使用内嵌 2 重写那些重名的方法，在方法内指定你到底需要那个结构体的方法

```

   