# 说明

本篇文章主要基于这篇博客:

> https://blog.go-zh.org/defer-panic-and-recover

# 被 `defer` 的函数的参数会立即求值

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

这里会输出 `0` 而不是 `1` 因为参数在 `defer` 声明的时候就被计算求值了.

# 多此执行 `defer` 函数将会放入后入先出队列

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```

最后会输出 `3210`.

# `defer` 函数可以修改命名返回值

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

最后会输出 `2`.

这段代码很有意思, 揭露了几个 go 语言特性:

1. 命名返回值相当于变量声明
2. `return` 语句相当于赋值语句
3. `defer` 将会在 `return` 后执行
4. `defer` 可以修改命名返回值从而改写 `return` 值

`defer` 这样设计的目的是为了方便处理错误机制, 在后面我们将会见到他, 即使如此我不认为这样的违反阅读顺序的设计是合理的.

# `defer` 结合 `panic` 和 `recover` 的例子

`panic` 一旦调用就会中止当前函数的执行, 确保当前函数内的 `defer` 执行完成后将自己控制权交由调用者, 重复这个过程一直到调用栈顶, 在这个过程中如果存在协程, 则会等待协程返回, 然后整个程序退出执行.

`recover` 只有在 `defer` 声明的函数中生效, 它的职责是拦截并且获取当前函数执行过程中抛出的 `panic`, 在正常执行中调用 `recover` 只会返回 `nil` 其他情况下返回值取决于 `panic` 传入值.

使用 `recover` 可以避免程序因为 `panic` 而退出执行.

```go
package main

import "fmt"

func main() {
	f()
	fmt.Println("Returned normally from f.")
}

func f() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered in f", r)
		}
	}()
	fmt.Println("Calling g.")
	g(0)
	fmt.Println("Returned normally from g.")
}

func g(i int) {
	if i > 3 {
		fmt.Println("Panicking!")
		panic(fmt.Sprintf("%v", i))
	}
	defer fmt.Println("Defer in g", i)
	fmt.Println("Printing in g", i)
	g(i + 1)
}
```

输出:

```
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.
```