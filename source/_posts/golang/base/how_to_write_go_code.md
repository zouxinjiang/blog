---
title: How to write go code
date: 2019-12-04 00:00:00

categories:
    - code

tags:
    - golang
---
# How to write go code

## 环境变量
- GOROOT 指定go语言安装的位置
- GOPATH 指定go代码所在位置
- GOBIN 指定go install的程序所在的位置

我们在程序中import包时的路径就是 $GOROOT/xxx 或 $GOPATH/xxx的包
还有一个特殊的约定，当你import包假设为 github.com/a/b 那么优先
使用的是 vendor/目录下的东西，然后搜索不到则到GOPATH下搜索，最后
再到GOROOT下搜索。

## 编写代码
在$GOPATH/src目录下创建一个你自己的包，然后写你的代码即可。

## 编译
使用 go build xxx(你包名文件夹名) 然后就会在当前目录得到编译后的
文件。也可以使用 go install xxx ，这样编译结果就会在GOBIN指定的
那个位置。


