+++
title = "defer、panic与recover"
description = "go 既拥有 if，for，switch，goto 这四种常见的控制语句，也拥有在一个单独的 goroutine 中运行代码的 go 语句，这篇文章主要谈论人们较少提及的 defer、panic、recover 语句。"
date = 2022-08-22
[taxonomies]
tags= ["defer", "panic", "GoLang"]
+++

**defer 语句**会把一个函数调用放置到一个列表中，当调用 `defer` 语句的函数返回时会依次调用该列表中定义的所有函数。`defer` 通常会调用执行各种清理动作的简单函数。比如说下面这个例子，打开两个文件，并将内容从一个文件复制到另一个文件中：

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

上面的程序看起来是正常的，但是如果 `os.Create(dstName)` 执行失败，程序将直接返回，从而导致 src 文件句柄没有及时关闭。一种比较简单的解决方式是将 src.Close() 方法在第二个 return 语句执行之前手动调用一次，但在函数逻辑很复杂的情况下可能会很容易被忽视，相反通过 defer 语句我们可以确保文件总是被及时关闭，上面的程序重构后如下：

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

defer 语句确保了无论在函数中有多少的 return 语句我们都可以在打开文件之后立刻通过 defer 语句调用 close 方法关闭文件，以确保文件最终会及时关闭。defer 语句的行为是非常简单明了的，下面是几个简单的规则：

- 1、defer 语句调用的函数参数值是基于调用 defer 语句时的对应变量值来确定的。比如在下面这个例子中表达式 "i" 的值在 defer fmt.Println 的时候就已经被赋值，在函数返回的时候 defer 语句将打印出 "0"。

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
}
```

- 2、在函数返回的时候 defer 列表中定义的函数将以后进先出的次序执行。如下面的程序输出为 "3210" :

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```

- 3、defer 可以读取和更新函数返回值中的命名返回值。如下面的函数返回值为 2:

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```

**Panic** 是一个阻止函数继续执行并向上抛出异常的内置函数。当函数 F 发生 panic 时，函数 F 的执行会停止，但其中定义的 defer 函数会正常执行。而此时对于函数 F 的调用者，F 的行为就像是手动调用了 panic，之后该程序会根据栈信息不停的向上返回，直到在这个 goroutine 中所有的程序都返回，这意味着该程序已经崩溃。Panics 既能通过手动调用 panic 语句调用也可能会发生在程序运行时，比如说数组越界。

**Recover** 是一个可以用来恢复发生 panic 的 goroutine 的内置函数，并且仅仅只能用在 defer 调用的函数里面。如果程序执行正常，recover() 函数将返回 nil 值，并且不会对程序造成影响；如果程序发生了 panic, 那么 recover() 将会捕获传递给 panic 的信息，并且恢复程序正常执行。

下面的程序演示了 panic 和 defer 的工作机制：

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

上面的例子中函数 g 接受参数 i，如果参数 i 的值大于 3 程序会发生 panic，否则它会用 `i + 1` 的值作为参数，递归的调用函数 g。函数 f 使用 defer 函数借助 recover 捕捉 panic 错误，如果 recover 返回值为非 nil 值，则打印出捕获到的值。在继续阅读之前，试着想象一下这个程序的输出可能是什么。

程序将会输出：

```bash
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

如果将上面函数 f 中的 defer 函数移除，那么 panic 异常最终将会抛出到 goroutine 调用栈的顶部，然后终止程序。之后程序输出将如下所示：

```bash
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
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
```

Go 标准库中的 [json package](https://pkg.go.dev/encoding/json) 包，包含 panic 和 recover 的实际例子。它使用一组递归函数对接口进行编码，如果在遍历值时发生错误，panic 会被调用然后返回到堆栈的顶层，然后使用 recover 捕获错误或者返回一个合适的错误值（具体见 [encode.go](https://go.dev/src/encoding/json/encode.go) 中 encodeState 类型的 'error' 和 'marshal' 方法）。

Go 标准库中的约定是在包内部使用 panic，而它的外部 API 仍然提供明确的错误返回值。

其他一些使用 defer 的场景包括释放一个 mutex、打印出 footer 等：

```go
// releasing a mutex
mu.Lock()
defer mu.Unlock()
// printing a footer
printHeader()
defer printFooter()
```

总之，defer 语句为程序的控制流提供了不寻常和强有力的机制，可以使用它来模拟其他编程语言中需要借助专用结构才能实现的功能。
