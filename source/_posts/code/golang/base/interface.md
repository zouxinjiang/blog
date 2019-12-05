---
title: interface 接口
date: 2019-12-12 00:00:00

categories:
    - code

tags:
    - golang
---
# interface 接口
跟其他语言一样，go也有接口类型。接口是一系列函数定义的集合。不过
go语言和其他语言的接口有些许区别，特别是实现上来说。其他语言实现
一个接口需要显示的在语法层面上进行说明 比如java就要用 implements 
关键字，而go语言则没有，只需要你有接口中的所有方法就实现了该接口
所以go中的接口实现隐式的。
- 接口的零值是 nil
- 接口也是值
- 接口的底层实现可以看成是(value,type)  存储了该接口类型存储的类型
以及值
- 空接口 interfaces{} 没有定义任何方法，所以它能存储任意值

<b style="color:red">注意：存储了nil值类型的接口并不为nil;接口实现者
如果有指针接收者的方法，则将该类型的值赋值给接口类型的变量时，只能
赋值为该类型的值的地址</b>

```go
type MyItf interface{
    SaySomething(data string)
    Show()
}

type MyInt int
func (*MyInt)SaySomething(data string){
    //todo
}

func(MyInt)Show(){
// todo
}

var itf MyItf
var i = MyInt(2)
itf = &i // 此处只能是&i,因为MyInt存在指针接收者类型的方法
```

## 类型断言
go是静态语言，有严格的类型检查，所以将值存入接口类型后，当我们要使用的
时候就会需要把它转化成原来的类型，此时就需要类型断言了，这看起来
有点像类型转换，但是不是，类型断言只是把接口存储的值提取出来，并
没有做类型转换，如果断言指定的类型和接口中存储的类型不一致，那么
就会panic。接口类型你可以当成(value,type)来看待，所以类型断言实际
就是提取出value的过程，如果你指定的类型和接口中存储的type不兼容
那么就会panic。
- 兼容。 这里说的兼容只存在于要么你指定的类型和接口中type一模一样
要么你指定的类型是另一个接口，且接口中的type类型实现了你指定的那个接口
```go
v := t.(T)  // 将接口t断言成T类型
v,ok := t.(T) // 避免panic的断言，断言失败 ok=false，不会panic

switch v := t.(type) {  // t.(type)这种只能用于switch case情况
case T:
// todo  v类型为T
default:
// todo 没有匹配，v类型还是t
}
```

## 内嵌
接口a可以内嵌接口b,内嵌后的a相当于继承了接口b的所有方法。
```go
type B interface{
	SayB()
}
type A interface{
    B       // 内嵌，相当于A也拥有了SayB()方法
    SayA()
}
```

## 特点
1. 接口类型能够更加灵活的实现编程
2. 解耦程序依赖
