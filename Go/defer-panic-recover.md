# defer-panic-recover
## defer
defer 语句会将函数推入到一个列表中。同时列表中的函数会在 return 语句执行后被调用。 defer 常常会被用来简化资源清理释放之类的操作。
### defer的特性
* 函数参数值由 defer 语句**调用时确定**
	```go
	func a() {
		i := 0
		defer fmt.Println(i)
		i++
		return
	}
	```
	调用后输出为
	```
	0
	```
* deferred 的函数将会在 return 语句之后按照**先进后出**的次序执行，即LIFO
	```go
	func b() {
		for i := 0; i < 4; i++ {
			defer fmt.Print(i)
		}
	}
	```
	调用后输出为
	```
	3210
	```
* deferred 函数还可以**读取return返回值**并**改变其值**
	```go
	func c() (i int) {
		defer func() { i++ }()//返回值自增
		return 1
	}
	func main() {
		fmt.Printf("%v", c())
	}
	```
	输出为
	```
	2
	```
## panic
panic 是 go 的内置函数，它可以终止程序的正常执行流程并发出 panic （类似其他语言的 exception ）。

比如当函数F调用 panic ， f 的执行将被终止，然后 **defer 的函数正常执行完后**返回给调用者。  
这个流程会栈的调用次序不断向上抛出 panic ，直到返回到 goroutine 栈顶，此时，程序将会崩溃退出。

panic 可以通过**直接调用** panic 产生。同时也可能由**运行时的错误**所产生，例如数组越界访问。
```go
package main

import "fmt"

func main() {
	defer func() { fmt.Println("111") }()
	defer func() { fmt.Println("222") }()
	defer func() { fmt.Println("333") }()
	panic("test")
}
```
结果为
```
333
222
111
panic: test

goroutine 1 [running]:
main.main()
        d:/code/note/test/go/main.go:9 +0x6b
exit status 2
```
## recover
recover 是 go 语言的内置函数，它的主要作用是可以从 panic 的重新夺回 goroutine 的控制权。

Recover **必须通过 defer 来运行**。在正常的执行流程中，调用recover 将会返回 nil 且没有什么其他的影响。但是如果当前的goroutine 产生了 panic ， recover 将会捕获到 panic 抛出的信息，同时恢复其正常的执行流程。

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
函数 g 接收参数 i ，如果 i 大于 3 就会产生 panic ，否则调用 `g(i+1)` 。而函数 f 通过 defer 匿名函数来执行 recover 并打印出捕获到的 panic 信息（如 r 不等于 `nil` ）。
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
假如我们移除 f 中的 recover ， panic 就不会被恢复并将到传送到 goroutine 栈顶，从而终止程序运行。如此输出如下：
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
panic: 4

goroutine 1 [running]:
main.g(0x4)
        d:/code/note/test/go/main.go:24 +0x1ec
main.g(0x3)
        d:/code/note/test/go/main.go:28 +0x136
main.g(0x2)
        d:/code/note/test/go/main.go:28 +0x136
main.g(0x1)
        d:/code/note/test/go/main.go:28 +0x136
main.g(0x0)
        d:/code/note/test/go/main.go:28 +0x136
main.f()
        d:/code/note/test/go/main.go:17 +0x5d
main.main()
        d:/code/note/test/go/main.go:6 +0x19
exit status 2
```