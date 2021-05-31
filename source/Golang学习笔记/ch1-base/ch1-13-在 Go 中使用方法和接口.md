面向对象编程 (OOP) 是一种广受欢迎的编程模式，大部分编程语言都支持（至少部分支持）。 Go 是其中一种语言，但它并不完全支持所有 OOP 原则。

在学习路径的这一阶段，你已掌握足够的基础知识，准备好学习和实现封装和组合等原则。

此模块介绍了接口在 Go 中的工作原理，以及接口在 Go 和其他编程语言中的区别。 Go 中的接口是隐式接口，你将通过此模块了解其工作原理。

此模块还介绍了在 Go 中使用接口的方法和原因。


# 在 Go 中使用方法

Go 中的方法是一种特殊类型的函数，但存在一个简单的区别：你必须在函数名称之前加入一个额外的参数。 此附加参数称为 *接收方*。

如你希望分组函数并将其绑定到自定义类型，则方法非常有用。 Go 中的这一方法类似于在其他编程语言中创建类，因为它允许你实现面向对象编程 (OOP) 模型中的某些功能，例如嵌入、重载和封装。

如要了解方法在 Go 中的重要性，请先学习如何声明一个方法。

## 声明方法

到目前为止，你仅将结构用作可在 Go 中创建的另一种自定义类型。 在此模块中你将了解到，通过添加方法你可以将行为添加到你所创建的结构中。

声明方法的语法如下所示：

```go
func (variable type) MethodName(parameters ...) {
    // method functionality
}
```

但是，在声明方法之前，必须先创建结构。 假设你想要创建一个几何包，并决定创建一个名为 `triangle` 的三角形结构作为此程序包的一个组成部分。 然后，你需要使用一种方法来计算此三角形的周长。 你可以在 Go 中将其表示为：

```go
type triangle struct {
    size int
}

func (t triangle) perimeter() int {
    return t.size * 3
}
```

结构看起来像普通结构，但 `perimeter()` 函数在函数名称之前有一个类型 `triangle` 的额外参数。 也就是说，在使用结构时，你可以按如下方式调用函数：

```go
func main() {
    t := triangle{3}
    fmt.Println("Perimeter:", t.perimeter())
}
```

如果尝试按平常的方式调用 `perimeter()` 函数，则此函数将无法正常工作，因为此函数的签名表明它需要接收方。 正因如此，调用此方法的唯一方式是先声明一个结构，获取此方法的访问权限。 这也意味着，只要此方法属于不同的结构，你甚至可以为其指定相同的名称。 例如，你可以使用 `perimeter()` 函数声明一个 `square` 结构，具体如下所示：

```go
package main

import "fmt"

type triangle struct {
    size int
}

type square struct {
    size int
}

func (t triangle) perimeter() int {
    return t.size * 3
}

func (s square) perimeter() int {
    return s.size * 4
}

func main() {
    t := triangle{3}
    s := square{4}
    fmt.Println("Perimeter (triangle):", t.perimeter())
    fmt.Println("Perimeter (square):", s.perimeter())
}
```

在运行前面的代码时，请注意避免任何错误，你将获得以下输出：

输出

```output
Perimeter (triangle): 9
Perimeter (square): 16
```

通过对 `perimeter()` 函数的两次调用，编译器将根据接收方类型来确定要调用的函数。 这有助于在各程序包之间保持函数的一致性和名称的简短，并避免将包名称作为前缀。 在下一个单元讲解接口时，我们将介绍这样做的重要性。

## 方法中的指针

有时，方法需要更新变量，或者，如果参数太大，则可能需要避免复制它。 在遇到此类情况时，你需要使用指针传递变量的地址。 在之前的模块中，当我们在讨论指针时提到，每次在 Go 中调用函数时，Go 都会复制每个参数值以便使用。

如果你需要更新方法中的接收方变量，也会执行相同的行为。 例如，假设你要创建一个新方法以使三角形的大小增加一倍。 你需要在接收方变量中使用指针，具体如下所示：

```go
func (t *triangle) doubleSize() {
    t.size *= 2
}
```

你可以证明此方法的有效性，具体如下所示：

```go
func main() {
    t := triangle{3}
    t.doubleSize()
    fmt.Println("Size:", t.size)
    fmt.Println("Perimeter:", s.perimeter())
}
```

在运行前面的代码时，你将看到以下输出：

输出

```output
Size: 6
Perimeter: 18
```

如果方法仅可访问接收方的信息，则不需要在接收方变量中使用指针。 但是，依据 Go 的约定，如果结构的任何方法具有指针接收方，则此结构的所有方法都必须具有指针接收方，即使某个方法不需要也是如此。

## 声明其他类型的方法

方法的一个关键方面在于，需要为任何类型定义方法，而不只是针对自定义类型（如结构）进行定义。 但是，你不能通过属于其他包的类型来定义结构。 因此，不能在基本类型（如 `string`）上创建方法。

尽管如此，你仍然可以利用一点技巧，基于基本类型创建自定义类型，然后将其用作基本类型。 例如，假设你要创建一个方法，以将字符串从小写字母转换为大写字母。 你可以按如下所示写入方法：

```go
package main

import (
    "fmt"
    "strings"
)

type upperstring string

func (s upperstring) Upper() string {
    return strings.ToUpper(string(s))
}

func main() {
    s := upperstring("Learning Go!")
    fmt.Println(s)
    fmt.Println(s.Upper())
}
```

在运行前面的代码时，你将看到以下输出：

输出

```output
Learning Go!
LEARNING GO!
```

请注意，你在使用新对象 `s` 时，可以在首次打印其值时将其作为字符串。 然后，你在调用 `Upper` 方法时， `s` 打印出类型字符串的所有大写字母。

## 嵌入方法

在之前的模块中，您已了解到可以在一个结构中使用属性，并将同一属性嵌入另一个结构中。 也就是说，可以重用来自一个结构的属性，以避免出现重复并保持代码库的一致性。 类似的观点也适用于方法。 即使接收方不同，也可以调用已嵌入结构的方法。

例如，假设你想要创建一个带有逻辑的新三角形结构，以加入颜色。 此外，你还希望继续使用之前声明的三角形结构。 因此，彩色三角形结构将如下所示：


```go
type coloredTriangle struct {
    triangle
    color string
}
```

然后，你可以初始化 `coloredTriangle` 结构，并从 `triangle` 结构调用 `perimeter()` 方法（甚至访问其字段），具体如下所示：

```go
func main() {
    t := coloredTriangle{triangle{3}, "blue"}
    fmt.Println("Size:", t.size)
    fmt.Println("Perimeter", t.perimeter())
}
```

继续操作并在程序中加入上述更改，以了解嵌入的工作方式。 当使用类似于上一个方法的 `main()` 方法运行程序时，你将看到以下输出：

输出

```output
Size: 3
Perimeter 9
```

如果你熟悉 Java 或等 C++ 等 OOP 语言，则可能会认为 `triangle` 结构看起来像基类， `coloredTriangle` 像子类（如继承），但并不完全如此。 实际上，Go 编译器会通过创建如下的包装器方法来推广 `perimeter()` 方法：


```go
func (t coloredTriangle) perimeter() int {
    return t.triangle.perimeter()
}
```

请注意，接收方是 `coloredTriangle`，它从三角形字段调用 `perimeter()` 方法。 好的一点在于，你不必再创建之前的方法。 你可以选择创建，但 Go 已在内部为你完成了此工作。 我们提供的上述示例仅供学习。

## 重载方法

让我们回到之前讨论过的 `triangle` 示例。 如果要在 `coloredTriangle` 结构中更改 `perimeter()` 方法的实现，会发生什么情况？ 不能存在两个同名的函数。 但是，因为方法需要额外参数（接收方），所以，你可以使用一个同名的方法，只要此方法专门用于要使用的接收方即可。 这就是重载方法的方式。

换而言之，如你想要更改其行为，可以编写我们刚才讨论过的包装器方法。 如果彩色三角形的周长是普通三角形的两倍，则代码将如下所示：


```go
func (t coloredTriangle) perimeter() int {
    return t.size * 3 * 2
}
```

现在，无需更改之前编写的 `main()` 方法中的任何其他内容，具体将如下所示：


```go
func main() {
    t := coloredTriangle{triangle{3}, "blue"}
    fmt.Println("Size:", t.size)
    fmt.Println("Perimeter", t.perimeter())
}
```

运行此方法时，你将得到不同输出：

输出

```output
Size: 3
Perimeter 18
```

但是，如果你仍需要从 `triangle` 结构调用 `perimeter()` 方法，则可通过对其进行显示访问来执行此操作，如下所示：

```go
func main() {
    t := coloredTriangle{triangle{3}, "blue"}
    fmt.Println("Size:", t.size)
    fmt.Println("Perimeter (colored)", t.perimeter())
    fmt.Println("Perimeter (normal)", t.triangle.perimeter())
}
```

运行此代码时，应会看到以下输出：

输出

```output
Size: 3
Perimeter (colored) 18
Perimeter (normal) 9
```

你可能已经注意到，在 Go 中，你可以 *替代* 方法，并在需要时仍访问 *原始* 方法。

## 方法中的封装

“封装”表示对象的发送方（客户端）无法访问某个方法。 通常，在其他编程语言中，你会将 `private` 或 `public` 关键字放在方法名称之前。 在 Go 中，只需使用大写标识符，即可公开方法，使用非大写的标识符将方法设为私有方法。

Go 中的封装仅在程序包之间有效。 换句话说，你只能隐藏来自其他程序包的实现详细信息，而不能隐藏程序包本身。

如要进行尝试，请创建新程序包 `geometry` 并按如下方式将三角形结构移入其中：


```go
package geometry

type Triangle struct {
    size int
}

func (t *Triangle) doubleSize() {
    t.size *= 2
}

func (t *Triangle) SetSize(size int) {
    t.size = size
}

func (t *Triangle) Perimeter() int {
    t.doubleSize()
    return t.size * 3
}
```

你可以使用上述程序包，具体如下所示：

```go
func main() {
    t := geometry.Triangle{}
    t.SetSize(3)
    fmt.Println("Perimeter", t.Perimeter())
}
```

此时你应获得以下输出：

输出

```output
Perimeter 18
```

如要尝试从 `main()` 函数中调用 `size` 字段或 `doubleSize()` 方法，程序将死机，如下所示：

```go
func main() {
    t := geometry.Triangle{}
    t.SetSize(3)
    fmt.Println("Size", t.size)
    fmt.Println("Perimeter", t.Perimeter())
}
```

在运行前面的代码时，你将看到以下错误：

输出

```output
./main.go:12:23: t.size undefined (cannot refer to unexported field or method size)
```

# 在 Go 中使用接口

Go 中的接口是一种用于表示其他类型的行为的数据类型。 接口类似于对象应满足的蓝图或协定。 在你使用接口时，你的基本代码将变得更加灵活、适应性更强，因为你编写的代码未绑定到特定的实现。 因此，你可以快速扩展程序的功能。 在本模块，你将了解相关原因。

与其他编程语言中的接口不同，Go 中的接口是满足隐式实现的。 Go 并不提供用于实现接口的关键字，因此，如果你之前使用的是其他编程语言中的接口，但不熟悉 Go，那么此概念可能会造成混淆。

在此模块中，我们将使用多个示例来探讨 Go 中的接口，并演示如何充分利用这些接口。

## 声明接口

Go 中的接口是一种抽象类型，只包括具体类型必须拥有或实现的方法。 正因如此，我们说接口类似于蓝图。

假设你希望在几何包中创建一个接口来指示形状必须实现的方法。 你可以按如下所示定义接口：

```go
type Shape interface {
    Perimeter() float64
    Area() float64
}
```

`Shape` 接口表示你想要考虑 `Shape` 的任何类型都需要同时具有 `Perimeter()` 和 `Area()` 方法。 例如，在创建 `Square` 结构时，它必须实现两种方法，而不是仅实现一种。 另外，请注意接口不包含这些方法的实现细节（例如，用于计算某个形状的周长和面积）。 接口仅表示一种协定。 三角形、圆圈和正方形等形状有不同的计算面积和周长方式。

## 实现接口

正如上文所讨论的内容，你没有用于实现接口的关键字。 当 Go 中的接口具有接口所需的所有方法时，则满足按类型的隐式实现。

让我们创建一个 `Square` 结构，此结构具有 `Shape` 接口中的两个方法，具体如下方的示例代码所示：

```go
type Square struct {
    size float64
}

func (s Square) Area() float64 {
    return s.size * s.size
}

func (s Square) Perimeter() float64 {
    return s.size * 4
}
```

请注意 `Square` 结构的方法签名与 `Shape` 接口的签名的匹配方式。 但是，另一个接口可能具有不同的名称，但方法相同。 Go 如何或何时知道某个具体类型正在实现哪个接口？ 在运行时，Go 会知道你何时使用了接口。

如要演示如何使用接口，你可以编写以下内容：

```go
func main() {
    var s Shape = Square{3}
    fmt.Printf("%T\n", s)
    fmt.Println("Area: ", s.Area())
    fmt.Println("Perimeter:", s.Perimeter())
}
```

运行前面的代码时，你将看到以下输出：

输出

```output
main.Square
Area:  9
Perimeter: 12
```

此时，无论你是否使用接口，都没有任何区别。 接下来，让我们创建另一种类型，如 `Circle`，然后探讨接口有用的原因。 以下是 `Circle` 结构的代码：

```go
type Circle struct {
    radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.radius
}
```

现在，让我们重构 `main()` 函数，并创建一个函数来打印其收到的对象类型，以及其面积和周长，具体如下所示：

```go
func printInformation(s Shape) {
    fmt.Printf("%T\n", s)
    fmt.Println("Area: ", s.Area())
    fmt.Println("Perimeter:", s.Perimeter())
    fmt.Println()
}
```

请注意 `printInformation` 函数具有参数 `Shape`。 这意味着，你可以将 `Square` 或 `Circle` 对象发送到此函数，尽管输出会有所不同，但仍可使用。 `main()` 函数此时将如下所示：

```go
func main() {
    var s Shape = Square{3}
    printInformation(s)

    c := Circle{6}
    printInformation(c)
}
```

请注意，对于 `c` 对象，我们不能将其指定为 `Shape` 对象。 但是，`printInformation` 函数需要一个对象来实现 `Shape` 接口中定义的方法。

在运行程序时，你将会看到以下输出：

输出

```output
main.Square
Area:  9
Perimeter: 12

main.Circle
Area:  113.09733552923255
Perimeter: 37.69911184307752
```

请注意，此时你不会得到错误，输出会根据其收到的对象类型而变化。 你还可以看到输出中的对象类型不涉及 `Shape` 接口的任何内容。

使用接口的优点在于，对于 `Shape`的每个新类型或实现，`printInformation` 函数都不需要更改。 正如之前所述，当你使用接口时，代码会变得更灵活、更容易扩展。

## 实现字符串接口

扩展现有功能的一个简单示例是使用 `Stringer`，它是具有 `String()` 方法的接口，具体如下所示：

```go
type Stringer interface {
    String() string
}
```

`fmt.Printf` 函数使用此接口来输出值，这意味着你可以编写自定义 `String()` 方法来打印自定义字符串，具体如下所示：

```go
package main

import "fmt"

type Person struct {
    Name, Country string
}

func (p Person) String() string {
    return fmt.Sprintf("%v is from %v", p.Name, p.Country)
}
func main() {
    rs := Person{"John Doe", "USA"}
    ab := Person{"Mark Collins", "United Kingdom"}
    fmt.Printf("%s\n%s\n", rs, ab)
}
```

运行前面的代码时，你将看到以下输出：

输出

```output
John Doe is from USA
Mark Collins is from United Kingdom
```

如你所见，你已使用自定义类型（结构）来写入 `String()` 方法的自定义版本。 这是在 Go 中实现接口的一种常用方法，正如我们之前的讲解，你还会在许多程序中找到此方法的示例。

## 扩展现有实现

假设你具有以下代码，并且希望通过编写负责处理某些数据的 `Writer` 方法的自定义实现来扩展其功能。

通过使用以下代码，你可以创建一个程序，此程序使用 GitHub API 从 Microsoft 获取三个存储库：

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "os"
)

func main() {
    resp, err := http.Get("https://api.github.com/users/microsoft/repos?page=15&per_page=5")
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    io.Copy(os.Stdout, resp.Body)
}
```

运行前面的代码时，你会收到类似于以下输出的内容（已缩短以便改善可读性）：

输出

```output
[{"id":276496384,"node_id":"MDEwOlJlcG9zaXRvcnkyNzY0OTYzODQ=","name":"-Users-deepakdahiya-Desktop-juhibubash-test21zzzzzzzzzzz","full_name":"microsoft/-Users-deepakdahiya-Desktop-juhibubash-test21zzzzzzzzzzz","private":false,"owner":{"login":"microsoft","id":6154722,"node_id":"MDEyOk9yZ2FuaXphdGlvbjYxNTQ3MjI=","avatar_url":"https://avatars2.githubusercontent.com/u/6154722?v=4","gravatar_id":"","url":"https://api.github.com/users/microsoft","html_url":"https://github.com/micro
....
```

请注意，`io.Copy(os.Stdout, resp.Body)` 调用是指将通过对 GitHub API 的调用获取的内容打印到终端。 假设你想要写入自己的实现以缩短你在终端中看到的内容。 在查看 [`io.Copy` 函数的源](https://golang.org/pkg/io/#Copy) 时，你将看到以下内容：

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

如果你深入查看第一个参数 `dst Writer` 的详细信息，你会注意到 `Writer` 是 [接口](https://golang.org/pkg/io/#Writer)：

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

你可以继续浏览 `io` 程序包的源代码，直到找到 [`Copy` 调用 `Write` 方法](https://golang.org/src/io/io.go?s=12980%3A13040#L411?azure-portal=true)的位置。 我们暂时不做任何处理。

由于 `Writer` 是接口，并且是 `Copy` 函数需要的对象，你可以编写 `Write` 方法的自定义实现。 因此，你可以自定义打印到终端的内容。

实现接口所需的第一项操作是创建自定义类型。 在这种情况下，你可以创建一个空结构，因为你只需按如下所示编写自定义 `Write` 方法即可：

```go
type customWriter struct{}
```

现在，你已准备就绪，可开始编写自定义 `Write` 函数。 此时，你还需要编写一个结构，以便将 JSON 格式的 API 响应解析为 Golang 对象。 你可以使用“JSON 转 Go”站点从 JSON 有效负载创建结构。 因此，`Write` 方法可能如下所示：

```go
type GitHubResponse []struct {
    FullName string `json:"full_name"`
}

func (w customWriter) Write(p []byte) (n int, err error) {
    var resp GitHubResponse
    json.Unmarshal(p, &resp)
    for _, r := range resp {
        fmt.Println(r.FullName)
    }
    return len(p), nil
}
```

最后，你必须修改 `main()` 函数以使用你的自定义对象，具体如下所示：

```go
func main() {
    resp, err := http.Get("https://api.github.com/users/microsoft/repos?page=15&per_page=5")
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    writer := customWriter{}
    io.Copy(writer, resp.Body)
}
```

在运行程序时，你将会看到以下输出：

输出

```output
microsoft/aed-blockchain-learn-content
microsoft/aed-content-nasa-su20
microsoft/aed-external-learn-template
microsoft/aed-go-learn-content
microsoft/aed-learn-template
```

由于你写入的自定义 `Write` 方法，输出效果现在更好。 以下是程序的最终版本：

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

type GitHubResponse []struct {
    FullName string `json:"full_name"`
}

type customWriter struct{}

func (w customWriter) Write(p []byte) (n int, err error) {
    var resp GitHubResponse
    json.Unmarshal(p, &resp)
    for _, r := range resp {
        fmt.Println(r.FullName)
    }
    return len(p), nil
}

func main() {
    resp, err := http.Get("https://api.github.com/users/microsoft/repos?page=15&per_page=5")
    if err != nil {
        fmt.Println("Error:", err)
        os.Exit(1)
    }

    writer := customWriter{}
    io.Copy(writer, resp.Body)
}
```

## 编写自定义服务器 API

最后，我们一起来探讨接口的另一种用例，如果你要创建服务器 API，你可能会发现此用例非常实用。 编写 Web 服务器的常用方式是使用 `net/http` 程序包中的 `http.Handler` 接口，具体如下所示（无需写入此代码）：

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

请注意 `ListenAndServe` 函数需要服务器地址（如 `http://localhost:8000`）以及将响应从调用调度至服务器地址的 `Handler` 的实例。

接下来，创建并浏览以下程序：

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

type dollars float32

func (d dollars) String() string {
    return fmt.Sprintf("$%.2f", d)
}

type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func main() {
    db := database{"Go T-Shirt": 25, "Go Jacket": 55}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}
```

在浏览前面的代码之前，让我们按如下所示运行程序：

Bash

```bash
go run main.go
```

如果没有得到任何输出，说明情况不错。 此时，在新浏览器窗口中打开 `http://localhost:8000`，或在终端中运行以下命令：

Bash

```bash
curl http://localhost:8000
```

现在你应会看到以下输出：

输出

```output
Go T-Shirt: $25.00
Go Jacket: $55.00
```

让我们一起慢慢回顾之前的代码，了解其用途并观察 Go 接口的功能。 首先，创建 `float32` 类型的自定义类型，然后编写 `String()` 方法的自定义实现，以便稍后使用。

```go
type dollars float32

func (d dollars) String() string {
    return fmt.Sprintf("$%.2f", d)
}
```

然后，写入 `http.Handler` 可使用的 `ServeHTTP` 方法的实现。 请注意，我们重新创建了自定义类型，但这次它是映射，而不是结构。 接下来，我们通过使用 `database` 类型作为接收方来写入 `ServeHTTP` 方法。 此方法的实现使用来自接收方的数据，然后对其进行循环访问，再输出每一项。


```go
type database map[string]dollars

func (db database) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}
```

最后，在 `main()` 函数中，我们将 `database` 类型实例化，并使用一些值对其进行初始化。 我们使用 `http.ListenAndServe` 函数启动了 HTTP 服务器，在其中定义了服务器地址，包括要使用的端口和实现 `ServerHTTP` 方法自定义版本的 `db` 对象。 因此，在你运行程序时，Go 将使用此方法的实现，这也正是你在服务器 API 中使用和实现接口的方式。

```go
func main() {
    db := database{"Go T-Shirt": 25, "Go Jacket": 55}
    log.Fatal(http.ListenAndServe("localhost:8000", db))
}
```

使用 `http.Handle` 函数时，可以在服务器 API 中找到接口的其他用例。 有关详细信息，请参阅 Go 网站上[编写 Web 应用程序](https://golang.org/doc/articles/wiki)帖子。

下面是一项挑战，旨在帮助你练习已学习的方法和接口等内容。 你还将运用之前模块中学到的经验，例如，创建和使用自己的程序包。

## 创建用于管理在线商店的程序包

编写一个程序，此程序使用自定义程序包来管理在线商店的帐户。 你的挑战包括以下四个要素：

    创建一个名为 Account 的自定义类型，此类型包含帐户所有者的名字和姓氏。 此类型还必须加入 ChangeName 的功能。

    创建另一个名为 Employee 的自定义类型，此类型包含用于将贷方数额存储为类型 float64 并嵌入 Account 对象的变量。 类型还必须包含 AddCredits、RemoveCredits 和 CheckCredits 的功能。 你需要展示你可以通过 Employee 对象更改帐户名称。

    将字符串方法写入 Account 对象，以便按包含名字和姓氏的格式打印 Employee 名称。

    最后，编写使用已创建程序包的程序，并测试此挑战中列出的所有功能。 也就是说，主程序应更改名称、打印名称、添加贷方、删除贷方以及检查余额。

在此处，你可以找到上一项挑战的解决方案。

## 商店程序包

下方是适用于商店程序包的代码：

```go
package store

import (
    "errors"
    "fmt"
)

type Account struct {
    FirstName string
    LastName  string
}

type Employee struct {
    Account
    Credits float64
}

func (a *Account) ChangeName(newname string) {
    a.FirstName = newname
}

func (e Employee) String() string {
    return fmt.Sprintf("Name: %s %s\nCredits: %.2f\n", e.FirstName, e.LastName, e.Credits)
}

func CreateEmployee(firstName, lastName string, credits float64) (*Employee, error) {
    return &Employee{Account{firstName, lastName}, credits}, nil
}

func (e *Employee) AddCredits(amount float64) (float64, error) {
    if amount > 0.0 {
        e.Credits += amount
        return e.Credits, nil
    }
    return 0.0, errors.New("Invalid credit amount.")
}

func (e *Employee) RemoveCredits(amount float64) (float64, error) {
    if amount > 0.0 {
        if amount <= e.Credits {
            e.Credits -= amount
            return e.Credits, nil
        }
        return 0.0, errors.New("You can't remove more credits than the account has.")
    }
    return 0.0, errors.New("You can't remove negative numbers.")
}

func (e *Employee) CheckCredits() float64 {
    return e.Credits
}
```

下方是主程序用于测试所有功能的代码：

```go
package main

import (
    "fmt"
    "store"
)

func main() {
    bruce, _ := store.CreateEmployee("Bruce", "Lee", 500)
    fmt.Println(bruce.CheckCredits())
    credits, err := bruce.AddCredits(250)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("New Credits Balance = ", credits)
    }

    _, err = bruce.RemoveCredits(2500)
    if err != nil {
        fmt.Println("Can't withdraw or overdrawn!", err)
    }

    bruce.ChangeName("Mark")

    fmt.Println(bruce)
}
```

