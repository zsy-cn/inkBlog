# 在 Go 中实现错误处理和日志记录

有时，你所编写的程序的行为不符合预期。 有时存在一些你无法控制的外部因素，例如其他进程阻止了文件，或者尝试访问不再可用的内存地址。 失败只是程序可能具有的另一种行为。 如果能预见这些失败，就能在它们出现时解决问题。

正如你所了解的那样，Go 的异常处理方法与其他语言不同，其错误处理过程也是如此。 在 Go 中，可能失败的函数应始终返回一个额外值，以便你能够成功预测和管理失败。 例如，你可以运行默认行为并记录尽可能多的信息以再现并修复问题。

在本模块中，你将学习 Go 的日志记录和错误处理方法。

# 了解如何在 Go 中处理错误

编写程序时，需要考虑程序失败的各种方式，并且需要管理失败。 无需让用户看到冗长而混乱的堆栈跟踪错误。 让他们看到有关错误的有意义的信息更好。 正如你所看到的，Go 具有 `panic` 和 `recover` 之类的内置函数来管理程序中的异常或意外行为。 但错误是已知的失败，你的程序应该可以处理它们。

Go 的错误处理方法只是一种只需要 `if` 和 `return` 语句的控制流机制。 例如，在调用函数以从 `employee` 对象获取信息时，可能需要了解该员工是否存在。 Go 处理此类预期错误的一贯方法如下所示：

```go
employee, err := getInformation(1000)
if err != nil {
    // Something is wrong. Do something.
}
```

注意 `getInformation` 函数返回了 `employee` 结构，还返回了错误作为第二个值。 该错误可能为 `nil`。 如果错误为 `nil`，则表示成功。 如果错误不是 `nil`，则表示失败。 非 `nil` 错误附带一条错误消息，你可以打印该错误消息，也可以记录该消息（更可取）。 这是在 Go 中处理错误的方式。 下一部分将介绍一些其他策略。

你可能会注意到，Go 中的错误处理要求你更加关注如何报告和处理错误。 这正是问题的关键。 让我们看一些其他示例，以帮助你更好地了解 Go 的错误处理方法。

我们将使用用于结构的代码片段来练习各种错误处理策略：


```go
package main

import (
    "fmt"
    "os"
)

type Employee struct {
    ID        int
    FirstName string
    LastName  string
    Address   string
}

func main() {
    employee, err := getInformation(1001)
    if err != nil {
        // Something is wrong. Do something.
    } else {
        fmt.Print(employee)
    }
}

func getInformation(id int) (*Employee, error) {
    employee, err := apiCallEmployee(1000)
    return employee, err
}

func apiCallEmployee(id int) (*Employee, error) {
    employee := Employee{LastName: "Doe", FirstName: "John"}
    return &employee, nil
}
```

从现在开始，我们将重点介绍如何修改 `getInformation`、`apiCallEmployee` 和 `main` 函数，以展示如何处理错误。

## 错误处理策略

当函数返回错误时，该错误通常是最后一个返回值。 正如上一部分所介绍的那样，调用方负责检查是否存在错误并处理错误。 因此，一个常见策略是继续使用该模式在子例程中传播错误。 例如，子例程（如上一示例中的 `getInformation`）可能会将错误返回给调用方，而不执行其他任何操作，如下所示：


```go
func getInformation(id int) (*Employee, error) {
    employee, err := apiCallEmployee(1000)
    if err != nil {
        return nil, err // Simply return the error to the caller.
    }
    return employee, nil
}
```

你可能还需要在传播错误之前添加更多信息。 为此，可以使用 `fmt.Errorf()` 函数，该函数与我们之前看到的函数类似，但它返回一个错误。 例如，你可以向错误添加更多上下文，但仍返回原始错误，如下所示：

```go
func getInformation(id int) (*Employee, error) {
    employee, err := apiCallEmployee(1000)
    if err != nil {
        return nil, fmt.Errorf("Got an error when getting the employee information: %v", err)
    }
    return employee, nil
}
```

另一种策略是在错误为暂时性错误时运行重试逻辑。 例如，可以使用重试策略调用函数三次并等待两秒钟，如下所示：

```go
func getInformation(id int) (*Employee, error) {
    for tries := 0; tries < 3; tries++ {
        employee, err := apiCallEmployee(1000)
        if err == nil {
            return employee, nil
        }

        fmt.Println("Server is not responding, retrying ...")
        time.Sleep(time.Second * 2)
    }

    return nil, fmt.Errorf("server has failed to respond to get the employee information")
}
```

最后，可以记录错误并对最终用户隐藏任何实现详细信息，而不是将错误打印到控制台。 我们将在下一模块介绍日志记录。 现在，让我们看看如何创建和使用自定义错误。

## 创建可重用的错误

有时错误消息数会增加，你需要维持秩序。 或者，你可能需要为要重用的常见错误消息创建一个库。 在 Go 中，你可以使用 `errors.New()` 函数创建错误并在若干部分中重复使用这些错误，如下所示：


```go
var ErrNotFound = errors.New("Employee not found!")

func getInformation(id int) (*Employee, error) {
    if id != 1001 {
        return nil, ErrNotFound
    }

    employee := Employee{LastName: "Doe", FirstName: "John"}
    return &employee, nil
}
```

`getInformation` 函数的代码外观更优美，而且如果需要更改错误消息，只需在一个位置更改即可。 另请注意，惯例是为错误变量添加 `Err` 前缀。

最后，如果你具有错误变量，则在处理调用方函数中的错误时可以更具体。 `errors.Is()` 函数允许你比较获得的错误的类型，如下所示：

```go
employee, err := getInformation(1000)
if errors.Is(err, ErrNotFound) {
    fmt.Printf("NOT FOUND: %v\n", err)
} else {
    fmt.Print(employee)
}
```

## 用于错误处理的推荐做法

在 Go 中处理错误时，请记住下面一些推荐做法：

- 始终检查是否存在错误，即使预期不存在。 然后正确处理它们，以免向最终用户公开不必要的信息。
- 在错误消息中包含一个前缀，以便了解错误的来源。 例如，可以包含包和函数的名称。
- 创建尽可能多的可重用错误变量。
- 了解使用返回错误和 panic 之间的差异。 不能执行其他操作时再使用 panic。 例如，如果某个依赖项未准备就绪，则程序运行无意义（除非你想要运行默认行为）。
- 在记录错误时记录尽可能多的详细信息（我们将在下一部分介绍记录方法），并打印出最终用户能够理解的错误。

# 了解如何在 Go 中记录


日志在程序中发挥着重要作用，因为它们是在出现问题时你可以检查的信息源。  通常，发生错误时，最终用户只会看到一条消息，指示程序出现问题。 从开发人员的角度来看，我们需要简单错误消息以外的更多信息。  这主要是因为我们想要再现该问题以编写适当的修补程序。 在本模块中，你将学习日志记录在 Go 中的工作原理。 你还将学习一些应始终实现的做法。

## `log` 包

对于初学者，Go 提供了一个用于处理日志的简单标准包。 可以像使用 `fmt` 包一样使用此包。 该标准包不提供日志级别，且不允许为每个包配置单独的记录器。 如果需要编写更复杂的日志记录配置，可以使用记录框架执行此操作。 稍后我们将介绍记录框架。

下面是使用日志的最简单方法：


```go
import (
    "log"
)

func main() {
    log.Print("Hey, I'm a log!")
}
```

运行前面的代码时，将看到以下输出：

输出

```output
2020/12/19 13:39:17 Hey, I'm a log!
```

默认情况下，`log.Print()` 函数将日期和时间添加为日志消息的前缀。 你可以通过使用 `fmt.Print()` 获得相同的行为，但使用 `log` 包还能执行其他操作，例如将日志发送到文件。 稍后我们将详细介绍 `log` 包功能。

你可以使用 `log.Fatal()` 函数记录错误并结束程序，就像使用 `os.Exit(1)` 一样。 使用以下代码片段试一试：


```go
package main

import (
    "fmt"
    "log"
)

func main() {
    log.Fatal("Hey, I'm an error log!")
    fmt.Print("Can you see me?")
}
```

运行前面的代码时，将看到以下输出：

输出

```output
2020/12/19 13:53:19  Hey, I'm an error log!
exit status 1
```

注意最后一行 `fmt.Print("Can you see me?")` 未运行。 这是因为 `log.Fatal()` 函数调用停止了该程序。 在使用 `log.Panic()` 函数时会出现类似行为，该函数也调用 `panic()` 函数，如下所示：

```go
package main

import (
    "fmt"
    "log"
)

func main() {
    log.Panic("Hey, I'm an error log!")
    fmt.Print("Can you see me?")
}
```

运行前面的代码时，将看到以下输出：

输出

```output
2020/12/19 13:53:19  Hey, I'm an error log!
panic: Hey, I'm an error log!

goroutine 1 [running]:
log.Panic(0xc000060f58, 0x1, 0x1)
        /usr/local/Cellar/go/1.15.5/libexec/src/log/log.go:351 +0xae
main.main()
        /Users/christian/go/src/helloworld/logs.go:9 +0x65
exit status 2
```

你仍获得日志消息，但现在还会获得错误堆栈跟踪。

另一重要函数是 `log.SetPrefix()`。 可使用它向程序的日志消息添加前缀。 例如，可以使用以下代码片段：


```go
package main

import (
    "log"
)

func main() {
    log.SetPrefix("main(): ")
    log.Print("Hey, I'm a log!")
    log.Fatal("Hey, I'm an error log!")
}
```

运行前面的代码时，将看到以下输出：

输出

```output
main(): 2021/01/05 13:59:58 Hey, I'm a log!
main(): 2021/01/05 13:59:58 Hey, I'm an error log!
exit status 1
```

只需设置一次前缀，日志就会包含日志源自的函数的名称等信息。

可以[在 Go 网站上浏览其他函数](https://golang.org/pkg/log/)。

## 记录到文件

除了将日志打印到控制台之外，你可能还希望将日志发送到文件，以便稍后或实时处理这些日志。

为什么想要将日志发送到文件？ 首先，你可能想要对最终用户隐藏特定信息。 他们可能对这些信息不感兴趣，或者你可能公开了敏感信息。  在文件中添加日志后，可以将所有日志集中在一个位置，并将它们与其他事件关联。 此模式为典型模式：具有可能是临时的分布式应用程序，例如容器。

让我们使用以下代码测试将日志发送到文件：

```go
package main

import (
    "log"
    "os"
)

func main() {
    file, err := os.OpenFile("info.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        log.Fatal(err)
    }

    defer file.Close()

    log.SetOutput(file)
    log.Print("Hey, I'm a log!")
}
```

运行前面的代码时，在控制台中看不到任何内容。 在目录中，你应看到一个名为 info.log 的新文件，其中包含使用 `log.Print()` 函数发送的日志。 请注意，需要首先创建或打开文件，然后将 `log` 包配置为将所有输出发送到文件。 然后，可以像通常做法那样继续使用 `log.Print()` 函数。

## 记录框架

最后，可能有 `log` 包中的函数不足以处理问题的情况。 你可能会发现，使用记录框架而不编写自己的库很有用。 Go 的几个记录框架有 [Logrus](https://github.com/sirupsen/logrus)、[zerolog](https://github.com/rs/zerolog)、[zap](https://github.com/uber-go/zap) 和 [Apex](https://github.com/apex/log)。

让我们来了解一下可以用 zerolog 做什么。

首先，你需要安装包。 如果你已使用此系列文章，则你可能已经在使用 Go 模块，因此你无需执行任何操作。 为以防万一，你可以在工作站上运行以下命令以安装 zerolog 库：

```
go get -u github.com/rs/zerolog/log
```

现在，使用以下代码片段尝试一下：

```go
package main

import (
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func main() {
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix
    log.Print("Hey! I'm a log message!")
}
```

运行前面的代码时，将看到以下输出：

输出

```output
{"level":"debug","time":1609855453,"message":"Hey! I'm a log message!"}
```

请注意，你只需包含正确的导入名称，然后便可以像通常做法那样继续使用 `log.Print()` 函数。 另请注意输出更改为 JSON 格式。 在集中位置运行搜索时，JSON 是一种有用的日志格式。

另一有用功能是你可以快速添加上下文数据，如下所示：

```go
package main

import (
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func main() {
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix

    log.Debug().
        Int("EmployeeID", 1001).
        Msg("Getting employee information")

    log.Debug().
        Str("Name", "John").
        Send()
}
```

运行前面的代码时，将看到以下输出：

输出

```output
{"level":"debug","EmployeeID":1001,"time":1609855731,"message":"Getting employee information"}
{"level":"debug","Name":"John","time":1609855731}
```

注意我们如何将员工 ID 添加为上下文。 它作为另一属性成为 logline 的一部分。 另外，务必要强调的是，你包含的字段是强类型的。

你可以使用 zerolog 实现其他功能，例如使用分级的日志记录、使用格式化的堆栈跟踪，以及使用多个记录器实例来管理不同输出。 有关详细信息，请参阅 [GitHub 站点](https://github.com/rs/zerolog)。

