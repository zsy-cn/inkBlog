# 了解 Go 中并发的工作原理

你可能已对 Go 在并发方面的出色表现有所耳闻。 也许正是这一最突出的功能使 Go 如此受欢迎，让它成为了编写 Docker、Kubernetes 和 Terraform 等其他软件的理想之选。

在开始了解 Go 中并发的工作原理之前，你可能需要忘记从其他编程语言中已经了解的知识。 Go 使用的方法截然不同。

学习本模块时，你已具备了学习更高级主题（如并发）所需的知识。 尽管如此，我们还是要先说明为什么需要并发。 我们将逐一介绍不同的主题。

我们建议你实践所有的示例代码，直到你能很好地理解这些概念之后再继续。 正如你在前面的模块中所经历的那样，实践能帮助你更好地理解概念。

# 了解 goroutine（轻量线程）


并发是独立活动的组合，就像 Web 服务器虽然同时处理多个用户请求，但它是自主运行的。 并发在当今的许多程序中都存在。 Web 服务器就是一个例子，但你也能看到，在批量处理大量数据时也需要使用并发。

Go 有两种编写并发程序的样式。 一种是在其他语言中通过线程实现的传统样式。 在本模块中，你将了解 Go 的样式，其中值是在称为 goroutine 的独立活动之间传递的，以与进程进行通信。

如果这是你第一次学习并发，我们建议你多花一些时间来查看我们将要编写的每一段代码，以进行实践。

## Go 实现并发的方法

通常，编写并发程序时最大的问题是在进程之间共享数据。 Go 采用不同于其他编程语言的通信方式，因为 Go 是通过 channel  来回传递数据的。 这意味着只有一个活动 (goroutine) 有权访问数据，设计上不存在争用条件。 学完本模块中的 goroutine 和  channel 之后，你将更好地理解 Go 的并发方法。

可以使用下面的标语来概括 Go 的方法：“不是通过共享内存通信，而是通过通信共享内存。” 你可以通过 [Go 博客中关于通过通信共享内存的文章](https://blog.golang.org/codelab-share#:~:text=Hoare's Communicating Sequential Processes?azure-portal=true.)来了解更多信息，但我们将在下面的部分继续讨论这个问题。

如前所述，Go 还提供低级别的并发基元。 但在本模块中，我们将只介绍 Go 实现并发的惯用方法。

让我们从探索 goroutine 开始。

## Goroutine

goroutine 是轻量线程中的并发活动，而不是在操作系统中进行的传统活动。 假设你有一个写入输出的程序和另一个计算两个数字相加的函数。 一个并发程序可以有数个 goroutine 同时调用这两个函数。

我们可以说，程序执行的第一个 goroutine 是 `main()` 函数。 如果要创建其他 goroutine，则必须在调用该函数之前使用 `go` 关键字，如下所示：

Go

```go
func main(){
    login()
    go launch()
}
```

你还会发现，许多程序喜欢使用匿名函数来创建 goroutine，如下所示：

Go

```go
func main(){
    login()
    go func() {
        launch()
    }()
}
```

为了查看其运行情况，让我们编写一个简单的并发程序。

## 编写并发程序

由于我们只想将重点放在并发部分，因此我们使用现有程序来检查 API 终结点是否响应。 代码如下：

Go

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func main() {
    start := time.Now()

    apis := []string{
        "https://management.azure.com",
        "https://dev.azure.com",
        "https://api.github.com",
        "https://outlook.office.com/",
        "https://api.somewhereintheinternet.com/",
        "https://graph.microsoft.com",
    }

    for _, api := range apis {
        _, err := http.Get(api)
        if err != nil {
            fmt.Printf("ERROR: %s is down!\n", api)
            continue
        }

        fmt.Printf("SUCCESS: %s is up and running!\n", api)
    }

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}
```

运行前面的代码时，将看到以下输出：

输出

```output
SUCCESS: https://management.azure.com is up and running!
SUCCESS: https://dev.azure.com is up and running!
SUCCESS: https://api.github.com is up and running!
SUCCESS: https://outlook.office.com/ is up and running!
ERROR: https://api.somewhereintheinternet.com/ is down!
SUCCESS: https://graph.microsoft.com is up and running!
Done! It took 1.658436834 seconds!
```

这里没有什么特别之处，但我们可以做得更好。 或许我们可以同时检查所有站点？ 此程序可以在 500 毫秒的时间内完成，不需要耗费将近两秒。

请注意，我们需要并发运行的代码部分是向站点进行 HTTP 调用的部分。 换句话说，我们需要为程序要检查的每个 API 创建一个 goroutine。

为了创建 goroutine，我们需要在调用函数前使用 `go` 关键字。 但我们在这里没有函数。 让我们重构该代码并创建一个新函数，如下所示：

Go

```go
func checkAPI(api string) {
    _, err := http.Get(api)
    if err != nil {
        fmt.Printf("ERROR: %s is down!\n", api)
        return
    }

    fmt.Printf("SUCCESS: %s is up and running!\n", api)
}
```

注意，我们不再需要 `continue` 关键字，因为我们不在 for 循环中。 要停止函数的执行流，只需使用 `return` 关键字。 现在，我们需要修改 `main()` 函数中的代码，为每个 API 创建一个 goroutine，如下所示：

Go

```go
for _, api := range apis {
    go checkAPI(api)
}
```

重新运行程序，看看发生了什么。

看起来程序不再检查 API 了，对吗？ 你可能会看到如下输出：

输出

```output
Done! It took 1.506e-05 seconds!
```

速度可真快！ 发生了什么情况？ 请注意，你看到最后一条消息显示程序已完成。 这是因为 Go 为循环中的每个站点都创建了一个 goroutine，并立即转到下一行。

即使看起来 `checkAPI` 函数没有运行，它实际上是在运行。 它只是没有时间完成。 请注意，如果在循环之后添加一个睡眠计时器会发生什么，如下所示：

Go

```go
for _, api := range apis {
    go checkAPI(api)
}

time.Sleep(3 * time.Second)
```

现在，重新运行程序时，可能会看到如下所示的输出：

输出

```output
ERROR: https://api.somewhereintheinternet.com/ is down!SUCCESS: https://api.github.com is up and running!SUCCESS: https://management.azure.com is up and running!SUCCESS: https://dev.azure.com is up and running!SUCCESS: https://outlook.office.com/ is up and running!SUCCESS: https://graph.microsoft.com is up and running!Done! It took 3.002114573 seconds!
```

看起来似乎起作用了，对吧？ 不完全如此。 如果你想在列表中添加一个新站点呢？ 也许三秒钟是不够的。 你怎么知道？ 你无法管理。 必须有更好的方法，这就是我们在下一节讨论 channel 时要涉及的内容。


# 将 channel 用作通信机制

​				 				已完成 			 			100 XP 		

- 8 分钟

Go 中的 channel 是 goroutine 之间的通信机制。 这就是为什么我们之前说过 Go  实现并发的方式是：“不是通过共享内存通信，而是通过通信共享内存。” 当你需要将值从一个 goroutine 发送到另一个 goroutine  时，将使用 channel。 让我们看看它们的工作原理，以及如何开始使用它们来编写并发 Go 程序。

## Channel 语法

由于 channel 是发送和接收数据的通信机制，因此它也有类型之分。 这意味着你只能发送 channel 支持的数据类型。 除使用关键字 `chan` 作为 channel 的数据类型外，还需指定将通过 channel 传递的数据类型，如 `int` 类型。

每次声明一个 channel 或希望在函数中指定一个 channel 作为参数时，都需要使用 `chan <type>`，如 `chan int`。 要创建 channel，需使用内置的 `make()` 函数，如下所示：

Go

```go
ch := make(chan int)
```

一个 channel 可以执行两项操作：发送数据和接收数据。 若要指定 channel 具有的操作类型，需要使用 channel 运算符 `<-`。 此外，在 channel 中发送数据和接收数据属于阻止操作。 你一会儿就会明白为何如此。

如果希望 channel 仅发送数据，则必须在 channel 之后使用 `<-` 运算符。 如果希望 channel 接收数据，则必须在 channel 之前使用 `<-` 运算符，如下所示：

Go

```go
ch <- x // sends (or write) x through channel ch
x = <-ch // x receives (or reads) data sent to the channel ch
<-ch // receives data, but the result is discarded
```

可在 channel 中执行的另一项操作是关闭 channel。 若要关闭 channel，需使用内置的 `close()` 函数，如下所示：

Go

```go
close(ch)
```

关闭 channel 时，你希望数据将不再在该 channel 中发送。 如果试图将数据发送到已关闭的 channel，则程序将发生严重错误。 如果试图通过已关闭的 channel 接收数据，则可以读取发送的所有数据。 随后的每次“读取”都将返回一个零值。

让我们回到之前创建的程序，然后使用 channel 来删除睡眠功能并稍做清理。 首先，让我们在 `main` 函数中创建一个字符串 channel，如下所示：

Go

```go
ch := make(chan string)
```

接下来，删除睡眠行 `time.Sleep(3 * time.Second)`。

现在，我们可以使用 channel 在 goroutine 之间进行通信。 让我们重构代码并通过 channel 发送该消息，而不是在 `checkAPI` 函数中打印结果。 要使用该函数中的 channel，需要添加 channel 作为参数。 `checkAPI` 函数应如下所示：

Go

```go
func checkAPI(api string, ch chan string) {
    _, err := http.Get(api)
    if err != nil {
        ch <- fmt.Sprintf("ERROR: %s is down!\n", api)
        return
    }

    ch <- fmt.Sprintf("SUCCESS: %s is up and running!\n", api)
}
```

请注意，我们必须使用 `fmt.Sprintf` 函数，因为我们不希望从此处打印出任何文本。 直接发送格式化文本。 另请注意，我们在 channel 变量之后使用 `<-` 运算符来发送数据。

现在，你需要更改 `main` 函数以发送 channel 变量并接收要打印的数据，如下所示：

Go

```go
ch := make(chan string)

for _, api := range apis {
    go checkAPI(api, ch)
}

fmt.Print(<-ch)
```

请注意，我们在 channel 之前使用 `<-` 运算符来表明我们想要从 channel 读取数据。

重新运行程序时，会看到如下所示的输出：

输出

```output
ERROR: https://api.somewhereintheinternet.com/ is down!

Done! It took 0.007401217 seconds!
```

至少它不用调用睡眠函数就可以工作，对吧？ 但它仍然没有达到我们的目的。 我们只看到其中一个 goroutine 的输出，而我们共创建了五个 goroutine。 在下一节中，我们来看看这个程序为什么是这样工作的。

## 无缓冲 channel

使用 `make()` 函数创建 channel 时，会创建一个无缓冲 channel，这是默认行为。 无缓冲 channel 会阻止发送操作，直到有人准备好接收数据。 这就是为什么我们之前说发送和接收都属于阻止操作。 这也是上一节中的程序在收到第一条消息后立即停止的原因。

我们可以说 `fmt.Print(<-ch)` 会阻止程序，因为它从 channel 读取，并等待一些数据到达。 一旦有任何数据到达，它就会继续下一行，然后程序完成。

其他 goroutine 发生了什么？ 它们仍在运行，但都没有在侦听。 而且，由于程序提前完成，一些 goroutine 无法发送数据。 为了证明这一点，让我们添加另一个 `fmt.Print(<-ch)`，如下所示：

Go

```go
ch := make(chan string)

for _, api := range apis {
    go checkAPI(api, ch)
}

fmt.Print(<-ch)
fmt.Print(<-ch)
```

重新运行程序时，会看到如下所示的输出：

输出

```output
ERROR: https://api.somewhereintheinternet.com/ is down!SUCCESS: https://api.github.com is up and running!Done! It took 0.263611711 seconds!
```

请注意，现在你会看到两个 API 的输出。 如果继续添加更多 `fmt.Print(<-ch)` 行，你最终将会读取发送到 channel 的所有数据。 但是如果你试图读取更多数据，而没有 goroutine 再发送数据，会发生什么呢？ 例如：

Go

```go
ch := make(chan string)for _, api := range apis {    go checkAPI(api, ch)}fmt.Print(<-ch)fmt.Print(<-ch)fmt.Print(<-ch)fmt.Print(<-ch)fmt.Print(<-ch)fmt.Print(<-ch)fmt.Print(<-ch)
```

重新运行程序时，会看到如下所示的输出：

输出

```output
ERROR: https://api.somewhereintheinternet.com/ is down!SUCCESS: https://api.github.com is up and running!SUCCESS: https://management.azure.com is up and running!SUCCESS: https://graph.microsoft.com is up and running!SUCCESS: https://outlook.office.com/ is up and running!SUCCESS: https://dev.azure.com is up and running!
```

它在运行，但程序未完成。 最后一个打印行阻止了程序，因为它需要接收数据。 必须使用类似 `Ctrl+C` 的命令关闭程序。

这只是证明了读取数据和接收数据都属于阻止操作。 要解决此问题，只需更改循环的代码，然后只接收确定要发送的数据，如下所示：

Go

```go
for i := 0; i < len(apis); i++ {
    fmt.Print(<-ch)
}
```

以下是程序的最终版本，以防你的版本出错：

Go

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func main() {
    start := time.Now()

    apis := []string{
        "https://management.azure.com",
        "https://dev.azure.com",
        "https://api.github.com",
        "https://outlook.office.com/",
        "https://api.somewhereintheinternet.com/",
        "https://graph.microsoft.com",
    }

    ch := make(chan string)

    for _, api := range apis {
        go checkAPI(api, ch)
    }

    for i := 0; i < len(apis); i++ {
        fmt.Print(<-ch)
    }

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}

func checkAPI(api string, ch chan string) {
    _, err := http.Get(api)
    if err != nil {
        ch <- fmt.Sprintf("ERROR: %s is down!\n", api)
        return
    }

    ch <- fmt.Sprintf("SUCCESS: %s is up and running!\n", api)
}
```

重新运行程序时，会看到如下所示的输出：

输出

```output
ERROR: https://api.somewhereintheinternet.com/ is down!
SUCCESS: https://api.github.com is up and running!
SUCCESS: https://management.azure.com is up and running!
SUCCESS: https://dev.azure.com is up and running!
SUCCESS: https://graph.microsoft.com is up and running!
SUCCESS: https://outlook.office.com/ is up and running!
Done! It took 0.602099714 seconds!
```

程序正在执行应执行的操作。 你不再使用休眠函数，而是使用 channel。 另请注意，在不使用并发时，程序需要约 600 毫秒即可完成，而不会耗费近 2 秒。

最后，我们可以说，无缓冲 channel 在同步发送和接收操作。 即使使用并发，通信也是同步的。

# 了解有缓冲 channel

​				 				已完成 			 			100 XP 		

- 9 分钟

正如你所了解的，默认情况下 channel 是无缓冲行为。 这意味着只有存在接收操作时，它们才接受发送操作。 否则，程序将永久被阻止等待。

有时需要在 goroutine 之间进行此类同步。 但是，有时你可能只需要实现并发，而不需要限制 goroutine 之间的通信方式。

有缓冲 channel 在不阻止程序的情况下发送和接收数据，因为有缓冲 channel 的行为类似于队列。 创建 channel 时，可以限制此队列的大小，如下所示：

Go

```go
ch := make(chan string, 10)
```

每次向 channel 发送数据时，都会将元素添加到队列中。 然后，接收操作将从队列中删除该元素。 当 channel 已满时，任何发送操作都将等待，直到有空间保存数据。 相反，如果 channel 是空的且存在读取操作，程序则会被阻止，直到有数据要读取。

下面是一个理解有缓冲 channel 的简单示例：

Go

```go
package main

import (
    "fmt"
)

func send(ch chan string, message string) {
    ch <- message
}

func main() {
    size := 4
    ch := make(chan string, size)
    send(ch, "one")
    send(ch, "two")
    send(ch, "three")
    send(ch, "four")
    fmt.Println("All data sent to the channel ...")

    for i := 0; i < size; i++ {
        fmt.Println(<-ch)
    }

    fmt.Println("Done!")
}
```

运行程序时，将看到以下输出：

输出

```output
All data sent to the channel ...
one
two
three
four
Done!
```

你可能会说我们在这里没有做任何不同的操作，你是对的。 但是让我们看看当你将 `size` 变量更改为一个更小的数字（你甚至可以尝试使用一个更大的数字）时会发生什么情况，如下所示：

Go

```go
size := 2
```

重新运行程序时，将看到以下错误：

输出

```output
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.send(...)
        /Users/developer/go/src/concurrency/main.go:8
main.main()
        /Users/developer/go/src/concurrency/main.go:16 +0xf3
exit status 2
```

出现此错误是因为对 `send` 函数的调用是连续的。 你不是在创建新的 goroutine。 因此，没有任何要排队的操作。

channel 与 goroutine 有着紧密的联系。 如果没有另一个 goroutine 从 channel 接收数据，则整个程序可能会永久处于被阻止状态。 正如你所见，这种情况确实会发生。

现在让我们进行一些有趣的实践！ 我们将为最后两次调用创建一个 goroutine （前两次调用被正确地放入缓冲区），并运行 for 循环四次。 代码如下：

Go

```go
func main() {
    size := 2
    ch := make(chan string, size)
    send(ch, "one")
    send(ch, "two")
    go send(ch, "three")
    go send(ch, "four")
    fmt.Println("All data sent to the channel ...")

    for i := 0; i < 4; i++ {
        fmt.Println(<-ch)
    }

    fmt.Println("Done!")
}
```

运行程序时，它按预期工作。 我们建议在使用 channel 时始终使用 goroutine。

为了对创建的有缓冲 channel 包含的元素比需要的更多的情况进行测试，让我们使用之前用于检查 API 的示例，并创建大小为 10 的有缓冲 channel：

Go

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func main() {
    start := time.Now()

    apis := []string{
        "https://management.azure.com",
        "https://dev.azure.com",
        "https://api.github.com",
        "https://outlook.office.com/",
        "https://api.somewhereintheinternet.com/",
        "https://graph.microsoft.com",
    }

    ch := make(chan string, 10)

    for _, api := range apis {
        go checkAPI(api, ch)
    }

    for i := 0; i < len(apis); i++ {
        fmt.Print(<-ch)
    }

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}

func checkAPI(api string, ch chan string) {
    _, err := http.Get(api)
    if err != nil {
        ch <- fmt.Sprintf("ERROR: %s is down!\n", api)
        return
    }

    ch <- fmt.Sprintf("SUCCESS: %s is up and running!\n", api)
}
```

运行程序时，将看到与以前相同的输出。 可以更改 channel 的大小，用更小或更大的数字进行测试，程序仍能正常运行。

## 无缓冲 channel 与有缓冲 channel

现在，你可能想知道何时使用这两种类型。 这完全取决于你希望 goroutine 之间的通信如何进行。 无缓冲 channel 同步通信。 它们保证每次发送数据时，程序都会被阻止，直到有人从 channel 中读取数据。

相反，有缓冲 channel 将发送和接收操作解耦。 它们不会阻止程序，但你必须小心使用，因为可能最终会导致死锁（如前文所述）。  使用无缓冲 channel 时，可以控制可并发运行的 goroutine 的数量。 例如，你可能要对 API  进行调用，并且想要控制每秒执行的调用次数。 否则，你可能会被阻止。

## Channel 方向

Go 中 channel 的一个有趣特性是，在使用 channel 作为函数的参数时，可以指定 channel 是要发送数据还是接收数据。 随着程序的增长，可能会使用大量的函数，这时候，最好记录每个 channel 的意图，以便正确使用它们。 或者，你要编写一个库，并希望将  channel 公开为只读，以保持数据一致性。

要定义 channel 的方向，可以使用与读取或接收数据时类似的方式进行定义。 但是你在函数参数中声明 channel 时执行此操作。 将 channel 类型定义为函数中的参数的语法如下所示：

Go

```go
chan<- int // it's a channel to only send data
<-chan int // it's a channel to only receive data
```

通过仅接收的 channel 发送数据时，在编译程序时会出现错误。

让我们使用以下程序作为两个函数的示例，一个函数用于读取数据，另一个函数用于发送数据：

Go

```go
package mainimport "fmt"func send(ch chan<- string, message string) {    fmt.Printf("Sending: %#v\n", message)    ch <- message}func read(ch <-chan string) {    fmt.Printf("Receiving: %#v\n", <-ch)}func main() {    ch := make(chan string, 1)    send(ch, "Hello World!")    read(ch)}
```

运行程序时，将看到以下输出：

输出

```output
Sending: "Hello World!"Receiving: "Hello World!"
```

程序阐明每个函数中每个 channel 的意图。 如果试图使用一个 channel 在一个仅用于接收数据的 channel 中发送数据，将会出现编译错误。 例如，尝试执行如下所示的操作：

Go

```go
func read(ch <-chan string) {
    fmt.Printf("Receiving: %#v\n", <-ch
    ch <- "Bye!"
}
```

运行程序时，将看到以下错误：

输出

```output
# command-line-arguments
./main.go:12:5: invalid operation: ch <- "Bye!" (send to receive-only type <-chan string)
```

编译错误总比误用 channel 好。

## 多路复用

最后，让我们讨论一个关于如何在使用 `select` 关键字的同时与多个 channel 交互的简短主题。 有时，在使用多个 channel 时，需要等待事件发生。 例如，当程序正在处理的数据中出现异常时，可以包含一些逻辑来取消操作。

`select` 语句的工作方式类似于 `switch` 语句，但它适用于 channel。 它会阻止程序的执行，直到它收到要处理的事件。 如果它收到多个事件，则会随机选择一个。

`select` 语句的一个重要方面是，它在处理事件后完成执行。 如果要等待更多事件发生，则可能需要使用循环。

让我们使用以下程序来看看 `select` 的运行情况：

Go

```go
package main

import (
    "fmt"
    "time"
)

func process(ch chan string) {
    time.Sleep(3 * time.Second)
    ch <- "Done processing!"
}

func replicate(ch chan string) {
    time.Sleep(1 * time.Second)
    ch <- "Done replicating!"
}

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    go process(ch1)
    go replicate(ch2)

    for i := 0; i < 2; i++ {
        select {
        case process := <-ch1:
            fmt.Println(process)
        case replicate := <-ch2:
            fmt.Println(replicate)
        }
    }
}
```

运行程序时，将看到以下输出：

输出

```output
Done replicating!
Done processing!
```

请注意，`replicate` 函数先完成。 这就是你在终端中先看到其输出的原因。 main 函数存在一个循环，因为 `select` 语句在收到事件后立即结束，但我们仍在等待 `process` 函数完成。

对于这个挑战，你需要通过提高现有程序的运行速度来改进它。 尝试自己编写程序，即使你不得不回头查看你以前用于练习的示例也要尝试。 然后，将你的解决方案与下一节中的解决方案进行比较。

Go 中的并发是一个复杂的问题，在实践中你会更好地理解它。 这一挑战只是你可以用来实践的一个建议。

祝好运！

## 利用并发方法更快地计算斐波纳契数

使用以下程序按顺序计算斐波纳契数：

Go

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func fib(number float64) float64 {
    x, y := 1.0, 1.0
    for i := 0; i < int(number); i++ {
        x, y = y, x+y
    }

    r := rand.Intn(3)
    time.Sleep(time.Duration(r) * time.Second)

    return x
}

func main() {
    start := time.Now()

    for i := 1; i < 15; i++ {
        n := fib(float64(i))
    fmt.Printf("Fib(%v): %v\n", i, n)
    }

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}
```

你需要根据现有代码构建两个程序：

- 实现并发的改进版本。 完成此操作需要几秒钟的时间（不超过 15 秒），就像现在这样。 应使用有缓冲 channel。

- 编写一个新版本以计算斐波纳契数，直到用户使用 `fmt.Scanf()` 函数在终端中输入 `quit`。 如果用户按 Enter，则应计算新的斐波纳契数。 换句话说，你将不再有从 1 到 10 的循环。

  使用两个无缓冲 channel：一个用于计算斐波纳契数，另一个用于等待用户的“退出”消息。 你需要使用 `select` 语句。

下面是与程序进行交互的示例：

输出

```output
1

1

2

3

5

8

13
quit
Done calculating Fibonacci!
Done! It took 12.043196415 seconds!
```

## 利用并发方法更快地计算斐波纳契数

实现并发并使程序的运行速度更快的改进版本如下所示：

Go

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func fib(number float64, ch chan string) {
    x, y := 1.0, 1.0
    for i := 0; i < int(number); i++ {
        x, y = y, x+y
    }

    r := rand.Intn(3)
    time.Sleep(time.Duration(r) * time.Second)

    ch <- fmt.Sprintf("Fib(%v): %v\n", number, x)
}

func main() {
    start := time.Now()

    size := 15
    ch := make(chan string, size)

    for i := 0; i < size; i++ {
        go fib(float64(i), ch)
    }

    for i := 0; i < size; i++ {
        fmt.Printf(<-ch)
    }

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}
```

使用两个无缓冲 channel 的程序的第二个版本如下所示：

Go

```go
package main

import (
    "fmt"
    "time"
)

var quit = make(chan bool)

func fib(c chan int) {
    x, y := 1, 1

    for {
        select {
            case c <- x:
                x, y = y, x+y
            case <-quit:
                fmt.Println("Done calculating Fibonacci!")
            return
        }
    }
}

func main() {
    start := time.Now()

    command := ""
    data := make(chan int)

    go fib(data)

    for {
        num := <-data
        fmt.Println(num)
        fmt.Scanf("%s", &command)
        if command == "quit" {
            quit <- true
            break
        }
    }

    time.Sleep(1 * time.Second)

    elapsed := time.Since(start)
    fmt.Printf("Done! It took %v seconds!\n", elapsed.Seconds())
}
```

