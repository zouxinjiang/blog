---
title: package 包
date: 2019-12-05 00:00:00

categories:
    - code

tags:
    - golang
---
# package 包
每个go程序都是由一些列的包构成，之前的hello world使用了fmt包，
而自身则是在main包下。程序入口函数main，必须在main包下。

## import 导包
1. 每行一个import
    ```go
    import "fmt"
    import "io"
    ```
2. 括号包含，只要一个import
    ```go
   import (   
       "fmt"
       "io"
   )
    ```
3. 导入包时指定包别名
```go
import (
 myfmt "fmt"
 _ "math"
 . "log"
)
```
- myfmt "fmt" 给fmt包起别名为myfmt，则代码中使用时应该使用 myfmt.xxx 
引用包内可导出对象
- _ "math" 导入了math包，但是代码内没有使用math包内的任何对象，1. go编译时对没有使用的包，
变量等都会报错。 2. 我们需要导入math包使math包的init函数被执行
- . "log" 这种方式很少用，这种方式使用包内对象时就可以直接使用 对象名 
而不是 包名.对象名 这种方式

## 使用包
使用时用 "包名.使用的对象"  这种方式即可。示例如下
- 使用包内变量  math.Pi
- 使用包内函数 rand.Seed(xx)
- 使用包内类型 io.Reader
- ...

## 可导出性
导出： 在包内外可见
不可导出：仅仅在包内可见
在go语言中使用大小写作为可导出性的规范，不像其他语言使用关键字作为
可导出性的约束。
- 大写开头的均可导出
- 小写开头的则不可导出
