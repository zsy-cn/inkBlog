title: 在Go中使用控制流
date: 2019-07-07 00:10:10 
author: Xavier
tags: 
    - golang
type: article
---

# 控制流

## 使用 if 和 else 语句

在任何编程语言中，最基本的控制流都是 `if/else` 语句。 在 Go 中，`if/else` 语句非常简单。 但是，你需要先了解一些差异，然后才能得心应手地编写 Go 程序。

让我们看看 `if` 语句的 Go 语法。

### if 语句的语法

与其他编程语言不同的是，在 Go 中，你不需要在条件中使用括号。 `else` 子句可选。 但是，大括号仍然是必需的。 此外，为了减少行，Go 不支持三元 `if` 语句，因此每次都需要编写完整的 `if` 语句。

下面是 `if` 语句的一个基本示例：

```Go
package main

import "fmt"

func main() {
    x := 27
    if x%2 == 0 {
        fmt.Println(x, "is even")
    }
}
```

在 Visual Studio Code 中，如果你的 Go 语法在条件中包含括号，系统会在你保存程序时自动删除括号。

### 复合 if 语句

Go 支持复合 `if` 语句。 可以使用 `else if` 语句对语句进行嵌套。 下面是一个示例：

```Go
package main

import "fmt"

func givemeanumber() int {
    return -1
}

func main() {
    if num := givemeanumber(); num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has only one digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }
}
```

请注意，在此代码中，`num` 变量存储从 `givemeanumber()` 函数返回的值，并且该变量在所有 `if` 分支中可用。 但是，如果尝试在 `if` 块之外输出 `num` 变量的值，则会出现如下错误：

```Go
package main

import "fmt"

func somenumber() int {
    return -7
}
func main() {
    if num := somenumber(); num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has 1 digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }

    fmt.Println(num)
}
```

运行程序时，错误输出如下所示：

```输出
# command-line-arguments
./main.go:17:14: undefined: num
```

在 Go 中，在 `if` 块内声明变量是惯用的方式，也就是说，它是一种使用在 Go 中常见的约定进行高效编程的方式。

## 使用 switch 语句

像其他编程语言一样，Go 支持 `switch` 语句。 可以使用 `switch` 语句来避免链接多个 `if` 语句。 使用 `switch` 语句，就不需维护和读取包含多个 `if` 语句的代码。 这些语句还可以让复杂的条件更易于构造。 请参阅以下部分的 `switch` 语句。

### 基本 switch 语法

像 `if` 语句一样，`switch` 条件不需要括号。 最简单形式的 `switch` 语句如下所示：

```Go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    sec := time.Now().Unix()
    rand.Seed(sec)
    i := rand.Int31n(10)

    switch i {
    case 0:
        fmt.Print("zero...")
    case 1:
        fmt.Print("one...")
    case 2:
        fmt.Print("two...")
    }

    fmt.Println("ok")
}
```

如果多次运行前面的代码，则每次都会看到不同的输出。 （但是，如果在 Go Playground 中运行代码，则每次都会获得相同的结果。 这是此服务的局限性之一。）

Go 会执行 `switch` 语句的每个用例，直到找到条件的匹配项。 但请注意，前面的代码未涵盖 `num` 变量的值的所有可能情况。 例如，如果 `num` 最终为 `5`，则程序的输出为 `ok`。

也可让默认用例更加具体，像下面这样包含它：

```Go
switch i {
case 0:
    fmt.Print("zero...")
case 1:
    fmt.Print("one...")
case 2:
    fmt.Print("two...")
default:
    fmt.Print("no match...")
}
```

请注意，对于 `default` 用例，不要编写验证表达式， 只需包含 `i` 变量即可，因为你将在 `case` 语句中验证其值。

### 使用多个表达式

有时，多个表达式仅与一个 `case` 语句匹配。 在 Go 中，如果希望 `case `语句包含多个表达式，请使用逗号 (`,`) 来分隔表达式。 此方法可避免代码重复。

以下代码示例演示了如何包含多个表达式。

```Go
package main

import "fmt"

func location(city string) (string, string) {
    var region string
    var continent string
    switch city {
    case "Delhi", "Hyderabad", "Mumbai", "Chennai", "Kochi":
        region, continent = "India", "Asia"
    case "Lafayette", "Louisville", "Boulder":
        region, continent = "Colorado", "USA"
    case "Irvine", "Los Angeles", "San Diego":
        region, continent = "California", "USA"
    default:
        region, continent = "Unknown", "Unknown"
    }
    return region, continent
}
func main() {
    region, continent := location("Irvine")
    fmt.Printf("John works in %s, %s\n", region, continent)
}
```

请注意，在 `case` 语句的表达式中包含的值对应于 `switch` 语句验证的变量的数据类型。 如果包含一个整数值作为新的 `case` 语句，程序将不会进行编译。

### 调用函数

`switch` 还可以调用函数。 在该函数中，可以针对可能的返回值编写 `case` 语句。 例如，以下代码调用 `time.Now()` 函数。 它提供的输出取决于当前工作日。

```Go
package main

import (
    "fmt"
    "time"
)

func main() {
    switch time.Now().Weekday().String() {
    case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
        fmt.Println("It's time to learn some Go.")
    default:
        fmt.Println("It's weekend, time to rest!")
    }

    fmt.Println(time.Now().Weekday().String())
}
```

从 `switch` 语句调用函数时，无需更改表达式即可修改其逻辑，因为你始终会验证函数返回的内容。

此外，还可以从 `case` 语句调用函数。 例如，使用此方法可以通过正则表达式来匹配特定模式。 下面是一个示例：

```Go
package main

import "fmt"

import "regexp"

func main() {
    var email = regexp.MustCompile(`^[^@]+@[^@.]+\.[^@.]+`)
    var phone = regexp.MustCompile(`^[(]?[0-9][0-9][0-9][). \-]*[0-9][0-9][0-9][.\-]?[0-9][0-9][0-9][0-9]`)

    contact := "foo@bar.com"

    switch {
    case email.MatchString(contact):
        fmt.Println(contact, "is an email")
    case phone.MatchString(contact):
        fmt.Println(contact, "is a phone number")
    default:
        fmt.Println(contact, "is not recognized")
    }
}
```

请注意，`switch` 块没有任何验证表达式。 我们将在下一部分讨论该概念。

### 省略条件

在 Go 中，可以在 `switch` 语句中省略条件，就像在 `if` 语句中那样。 此模式类似于比较 `true` 值，就像强制 `switch` 语句一直运行一样。

下面是一个示例，说明了如何编写不带条件的 `switch` 语句：

```Go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().Unix())
    r := rand.Float64()
    switch {
    case r > 0.1:
        fmt.Println("Common case, 90% of the time")
    default:
        fmt.Println("10% of the time")
    }
}
```

可以通过此模式更清晰地编写较长的 `if-then-else` 链。

### 使逻辑进入到下一个 case

在某些编程语言中，你会在每个 `case` 语句末尾写一个 `break` 关键字。 但在 Go 中，当逻辑进入某个 `case` 时，它会退出 `switch` 块，除非你显式停止它。 若要使逻辑进入到下一个紧邻的 `case`，请使用 `fallthrough` 关键字。

若要更好地了解此模式，请查看以下代码示例。

```Go
package main

import (
    "fmt"
)

func main() {
    switch num := 15; {
    case num < 50:
        fmt.Printf("%d is less than 50\n", num)
        fallthrough
    case num > 100:
        fmt.Printf("%d is greater than 100\n", num)
        fallthrough
    case num < 200:
        fmt.Printf("%d is less than 200", num)
    }
}
```

运行代码并分析输出：

```输出
15 is less than 50
15 is greater than 100
15 is less than 200
```

你是否看到错误？

请注意，由于 `num` 为 15（小于 50），因此它与第一个 `case` 匹配。 但是，`num` 不大于 100。 由于第一个 `case` 语句包含 `fallthrough` 关键字，因此逻辑会立即转到下一个 `case` 语句，而不会对该 `case` 进行验证。 因此，在使用 `fallthrough` 关键字时必须谨慎。 该代码产生的行为可能不是你想要的。

## 使用 for 循环

另一个常用控制流是循环。 Go 只使用一个循环构造，即 `for` 循环。 但是，你可以通过多种方式表示循环。 此部分介绍 Go 支持的循环模式。

### 基本 for 循环语法

与 `if` 语句和 `switch` 语句一样，`for` 循环表达式不需要括号。 但是，大括号是必需的。

分号 (`;`) 分隔 `for` 循环的三个组件：

- 在第一次迭代之前执行的初始语句（可选）。
- 在每次迭代之前计算的条件表达式。 该条件为 `false` 时，循环会停止。
- 在每次迭代结束时执行的后处理语句（可选）。

如你所见，Go 中的 `for` 循环类似于 C、Java 和 C# 之类的编程语言中的 `for` 循环。

在 Go 中，最简单形式的 `for` 循环如下所示：

```Go
func main() {
    sum := 0
    for i := 1; i <= 100; i++ {
        sum += i
    }
    fmt.Println("sum of 1..100 is", sum)
}
```

让我们看看如何在 Go 中以其他方式编写循环。

### 空的预处理语句和后处理语句

在某些编程语言中，可以使用 `while` 关键字编写循环模式，在这些模式中，只有条件表达式是必需的。 Go 没有 `while` 关键字。 但是，你可以改用 `for` 循环。 此预配使预处理语句和后处理语句变得可选。

使用以下代码片段确认是否可以在不使用预处理语句和后处理语句的情况下使用 `for` 循环。

```Go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    var num int64
    rand.Seed(time.Now().Unix())
    for num != 5 {
        num = rand.Int63n(15)
        fmt.Println(num)
    }
}
```

只要 `num` 变量保存的值与 `5` 不同，程序就会输出一个随机数。
无限循环和 break 语句

可以在 Go 中编写的另一种循环模式是无限循环。 在这种情况下，你不编写条件表达式，也不编写预处理语句或后处理语句， 而是采取退出循环的方式进行编写。 否则，逻辑永远都不会退出。 若要使逻辑退出循环，请使用 `break` 关键字。

若要正确编写无限循环，请在 `for` 关键字后面使用大括号，如下所示：

```Go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    var num int32
    sec := time.Now().Unix()
    rand.Seed(sec)

    for {
        fmt.Print("Writting inside the loop...")
        if num = rand.Int31n(10); num == 5 {
            fmt.Println("finish!")
            break
        }
        fmt.Println(num)
    }
}
```

每次运行此代码时，都会得到不同的输出。

### Continue 语句

在 Go 中，可以使用 `continue` 关键字跳过循环的当前迭代。 例如，可以使用此关键字在循环继续之前运行验证。 也可以在编写无限循环并需要等待资源变得可用时使用它。

此示例使用 continue 关键字：

```Go
package main

import "fmt"

func main() {
    sum := 0
    for num := 1; num <= 100; num++ {
        if num%5 == 0 {
            continue
        }
        sum += num
    }
    fmt.Println("The sum of 1 to 100, but excluding numbers divisible by 5, is", sum)
}
```

此示例有一个 `for` 循环，该循环从 1 迭代到 100，并在每次迭代中将当前数字添加到总和。 在此循环的当前迭代中，将跳过每个可被 5 整除的数字，不会将其添加到总和。

## 使用 defer、panic 和 recover 函数

现在，请考虑 Go 中一些不太常用的控制流：`defer`、`panic` 和 `recover`。 如你所见，Go 在某些方面采用惯用的方式，而下面这三种控制流则是 Go 独有的。

其中的每个函数都有几个用例。 你将在此处了解最重要的用例。 让我们开始学习第一个函数。

### defer 函数

在 Go 中，`defer` 语句会推迟函数（包括任何参数）的运行，直到包含 `defer` 语句的函数完成。 通常情况下，当你想要避免忘记任务（例如关闭文件或运行清理进程）时，可以推迟某个函数的运行。

可以根据需要推迟任意多个函数。 `defer` 语句按逆序运行，先运行最后一个，最后运行第一个。

通过运行以下示例代码来查看此模式的工作原理：

```Go
package main

import "fmt"

func main() {
    for i := 1; i <= 4; i++ {
        defer fmt.Println("deferred", -i)
        fmt.Println("regular", i)
    }
}
```

下面是代码输出：

```输出
regular 1
regular 2
regular 3
regular 4
deferred -4
deferred -3
deferred -2
deferred -1
```

在此示例中，请注意，每次推迟 `fmt.Println("deferred", -i)` 时，都会存储 `i` 的值，并会将其运行任务添加到队列中。 在 `main()` 函数输出完 `regular` 值后，所有推迟的调用都会运行。 这就是你看到输出采用逆序（后进先出）的原因。

`defer` 函数的一个典型用例是在使用完文件后将其关闭。 下面是一个示例：

```Go
package main

import (
    "io"
    "os"
)

func main() {
    f, err := os.Create("notes.txt")
    if err != nil {
        return
    }
    defer f.Close()

    if _, err = io.WriteString(f, "Learning Go!"); err != nil {
        return
    }

    f.Sync()
}
```

创建或打开某个文件后，可以推迟 `f.Close()` 函数的执行，以免在你向文件中写入内容后忘记关闭该文件。

### panic 函数

运行时错误会使 Go 程序进入紧急状态。 可以强制程序进入紧急状态，但运行时错误（例如数组访问超出范围、取消对空指针的引用）也可能会导致进入紧急状态。

内置 `panic()` 函数会停止正常的控制流。 所有推迟的函数调用都会正常运行。 进程会在堆栈中继续，直到所有函数都返回。 然后，程序会崩溃并记录日志消息。 此消息包含错误和堆栈跟踪，有助于诊断问题的根本原因。

调用 `panic()` 函数时，可以添加任何值作为参数。 通常，你会发送一条错误消息，说明为什么会进入紧急状态。

例如，将 `panic` 和 `defer` 函数组合起来，以了解控制流中断的方式。 但是，请继续运行任何清理进程。 请使用以下代码片段：

```Go
package main

import "fmt"

func main() {
    g(0)
    fmt.Println("Program finished successfully!")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic("Panic in g() (major)")
    }
    defer fmt.Println("Defer in g()", i)
    fmt.Println("Printing in g()", i)
    g(i + 1)
}
```

运行代码时，输出如下所示：

```输出
Printing in g() 0
Printing in g() 1
Printing in g() 2
Printing in g() 3
Panicking!
Defer in g() 3
Defer in g() 2
Defer in g() 1
Defer in g() 0
panic: Panic in g() (major)

goroutine 1 [running]:
main.g(0x4)
        /Users/johndoe/go/src/helloworld/main.go:13 +0x22e
main.g(0x3)
        /Users/johndoe/go/src/helloworld/main.go:17 +0x17a
main.g(0x2)
        /Users/johndoe/go/src/helloworld/main.go:17 +0x17a
main.g(0x1)
        /Users/johndoe/go/src/helloworld/main.go:17 +0x17a
main.g(0x0)
        /Users/johndoe/go/src/helloworld/main.go:17 +0x17a
main.main()
        /Users/johndoe/go/src/helloworld/main.go:6 +0x2a
exit status 2
```

下面是运行代码时会发生的情况：

1. 一切正常运行。 程序输出 g() 函数接收的值。
2. 当 i 大于 3 时，程序会进入紧急状态。 会显示“Panicking!”消息。 此时，控制流中断，所有推迟的函数都开始输出“Defer in g()”消息。
3. 程序崩溃，并显示完整的堆栈跟踪。 不会显示“Program finished successfully!”消息。

在发生未预料到的严重错误时，系统通常会运行对 `panic()` 的调用。 若要避免程序崩溃，可以使用 `recover()` 函数。 下一部分将介绍该函数。

### recover 函数

有时，你可能想要避免程序崩溃，改为在内部报告错误。 或者，你可能想要先清理混乱情况，然后再让程序崩溃。 例如，你可能想要关闭与某个资源的连接，以免出现更多问题。

Go 提供内置函数 `recover()`，允许你在出现紧急状况之后重新获得控制权。 只能在已推迟的函数中使用此函数。 如果调用 `recover()` 函数，则在正常运行的情况下，它会返回 `nil`，没有任何其他作用。

请尝试修改前面的代码，添加对 `recover()` 函数的调用，如下所示：

```Go
package main

import "fmt"

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in main", r)
        }
    }()
    g(0)
    fmt.Println("Program finished successfully!")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic("Panic in g() (major)")
    }
    defer fmt.Println("Defer in g()", i)
    fmt.Println("Printing in g()", i)
    g(i + 1)
}
```

运行程序时，输出应该如下所示：

```输出
Printing in g() 0
Printing in g() 1
Printing in g() 2
Printing in g() 3
Panicking!
Defer in g() 3
Defer in g() 2
Defer in g() 1
Defer in g() 0
Recovered in main Panic in g() (major)
```

你是否看到了相对于上一版本的差异？ 主要差异在于，你不再看到堆栈跟踪错误。

在 `main()` 函数中，你会将一个可以调用 `recover()` 函数的匿名函数推迟。 当程序处于紧急状态时，对 `recover()` 的调用无法返回 `nil`。 你可以在此处执行一些操作来清理混乱，但在这种情况下，你可以直接输出一些内容。

`panic` 和 `recover` 的组合是 Go 处理异常的惯用方式。 其他编程语言使用 `try/catch` 块。 Go 首选此处所述的方法。

有关详细信息，请参阅在 `Go 中添加内置 try 函数的建议`。

## 使用控制流编写程序

### 编写 FizzBuzz 程序

首先，编写一个用于输出数字（1 到 100）的程序，其中有以下变化：

- 如果数字可被 3 整除，则输出 Fizz。
- 如果数字可被 5 整除，则输出 Buzz。
- 如果数字可同时被 3 和 5 整除，则输出 FizzBuzz。
- 如果前面的情况都不符合，则输出该数字。

尝试使用 switch 语句。

```Go
func FizzBuzz(num int){
    switch {
    case 0 == num%(3*5):
        fmt.Printf("FizzBuzz\n")
        break
    case 0 == num%3:
        fmt.Printf("Fizz\n")
        break
    case 0 == num%5:
        fmt.Printf("Buzz\n")
        break
    default :
        fmt.Printf("%d\n", num)
    }
}
```

### 推测平方根

编写一个程序来推测数字的平方根。 使用以下公式：

sroot = sroot − (sroot − x) / (2 * sroot)

此公式适用于牛顿的方法。

运行此公式的次数越多，就越接近数字的平方根。 将 sroot 变量初始化为 1，无论你要查找其平方根的数字如何。 重复此计算最多 10 次，然后输出每次推测的结果。

你需要的计算次数可能少于 10 次。 如果获得的结果与前面的一次运行的结果相同，则可以停止循环。 数字越大，需要运行的计算次数越多。 为简单起见，你将最多重复计算 10 次。

例如，如果输入数字为 25，则输出应如下所示：

```输出
A guess for square root is  13
A guess for square root is  7.461538461538462
A guess for square root is  5.406026962727994
A guess for square root is  5.015247601944898
A guess for square root is  5.000023178253949
A guess for square root is  5.000000000053723
A guess for square root is  5
Square root is: 5
```

```Go
func Sroot(num int){
    for i:=1 ; 1<=10; i++ {

    }
}
```






要求用户输入一个数字，如果该数字为负数，则进入紧急状态

编写一个要求用户输入一个数字的程序。 在开始时使用以下代码片段：
Go

package main

import (
    "fmt"
)

func main() {
    val := 0
    fmt.Print("Enter number: ")
    fmt.Scanf("%d", &val)
    fmt.Println("You entered:", val)
}

此程序要求用户输入一个数字，然后将其输出。 修改示例代码，使之符合以下要求：

    持续要求用户输入一个整数。 此循环的退出条件应该是用户输入了一个负数。
    当用户输入负数时，让程序崩溃。 然后输出堆栈跟踪错误。
    如果数字为 0，则输出“0 is neither negative nor positive”。 继续要求用户输入数字。
    如果数字为正数，则输出“You entered: X”（其中的 X 为输入的数字）。 继续要求用户输入数字。

现在，请忽略用户输入的内容可能不是整数这种可能性。

编写 FizzBuzz 程序

该挑战的解决方案可能如下所示：
Go

package main

import (
    "fmt"
    "strconv"
)



func main() {
    for num := 1; num <= 100; num++ {
        fmt.Println(fizzbuzz(num))
    }
}

对于 FizzBuzz 用例，请将 3 乘以 5，因为结果可被 3 和 5 整除。 还可以包含一个 AND 条件来检查数字是否可被 3 和 5 整除。
推测平方根

该挑战的解决方案可能如下所示：
Go

package main

import "fmt"

func sqrt(num float64) float64 {
    currguess := 1.0
    prevguess := 0.0

    for count := 1; count <= 10; count++ {
        prevguess = currguess
        currguess = prevguess - (prevguess*prevguess-num)/(2*prevguess)
        if currguess == prevguess {
            break
        }
        fmt.Println("A guess for square root is ", currguess)
    }
    return currguess
}

func main() {
    var num float64 = 25
    fmt.Println("Square root is:", sqrt(num))
}

此解决方案在循环内包含一个 if 语句。 如果以前的数字和当前的数字相同，则此语句会停止该逻辑。 对于较大的数字，你需要的计算次数可能会超出 10 次。 因此，你可以更改代码，使用一个无限循环来运行计算，直到以前的推测结果和当前的推测结果相同。 为了避免小数精度问题，需要使用舍入数字。
要求用户输入一个数字，如果该数字为负数，则进入紧急状态

该挑战的解决方案可能如下所示：
Go

package main

import (
    "fmt"
)

func main() {
    val := 0

    for {
        fmt.Print("Enter number: ")
        fmt.Scanf("%d", &val)

        switch {
        case val < 0:
            panic("You entered a negative number!")
        case val == 0:
            fmt.Println("0 is neither negative nor positive")
        default:
            fmt.Println("You entered:", val)
        }
    }
}

请记住，目的是练习无限循环和 switch 语句。