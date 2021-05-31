# 在 Go 中编写并测试程序

我们来到了最后一个模块，在这里我们将编写一个简单的程序，它几乎使用了我们迄今为止看到的所有概念。 我们的思路是指导你逐步了解如何编写程序、如何构建文件的结构、如何编译文件、如何运行程序，以及如何测试程序。 我们还没有讨论如何在 Go 中编写测试，但是我们将使用本模块来介绍这个重要的主题。

首先，我们将编写程序的核心，该程序将用于网上银行。 用户将通过 API 与程序进行交互。 我们将在 Go 中创建两个项目，用于练习如何从其他程序引用本地程序包。 最后，为了确保我们的核心程序逻辑始终有效，我们将创建一组测试。在浏览器中手动测试该程序之前，可以运行这些测试。

与其他模块一样，你需要通过解决一项挑战来扩展此程序，使你在将来编写其他 Go 程序时能够更加自信。

# 概述网上银行项目

我们来谈谈将要创建的内容。 如前所述，我们将创建两个项目：一个用于程序的核心逻辑，另一个用于通过 Web API 公开逻辑。 假设你现在是一个团队的一员，这个团队正在构建一个网上银行系统。

## 定义功能和要求

我们将要建立的网上银行是一个概念验证，它将确定构建银行程序的可行性。 在第一次迭代中，与核心程序包的交互将通过一个 CLI 程序进行。 我们没有用户界面，也不会将数据持久保存到数据库中。 为了查看客户的对账单，我们将直接公开一个终结点。

网上银行系统将：

- 允许客户创建帐户。
- 允许客户取款。
- 允许客户将资金转到其他帐户。
- 提供包含客户数据和最终余额的对账单。
- 通过终结点公开一个 Web API，用于输出对账单。

我们将一同构建此程序，所以你现在不需要太担心细节。

## 创建初始项目文件

接下来，让我们创建程序所需的初始文件集。 我们将为所有银行核心逻辑和 `main` 程序创建一个 Go 程序包，以使用一些客户和操作（例如存款和转账）来初始化系统。 另外，此 `main` 程序还会启动一个 Web API 服务器，以便为对账单公开一个终结点。

让我们在 `$GOPATH` 目录中创建以下文件结构：

输出

```output
$GOPATH/
  src/
    bankcore/
      go.mod
      bank.go
    bankapi/
      go.mod
      main.go
```

然后，为了确保我们只需要集中精力在合适的文件中编写代码，让我们开始编写一个 `Hello World!` 程序，该程序将确认我们可以从 `bankapi` 主程序调用 `bankcore` 程序包。

将以下代码片段复制并粘贴到 `src/bankcore/bank.go` 中：

Go

```go
package bank

func Hello() string {
    return "Hey! I'm working!"
}
```

我们将使用 Go 模块。 在 `src/bankcore/go.mod` 中添加以下内容，为此程序包提供一个正确的名称，以便以后可以引用它：

Go

```go
module github.com/msft/bank

go 1.14
```

然后，在 `src/bankapi/main.go` 中添加以下代码来调用 `bankcore` 程序包：

Go

```go
package main

import (
    "fmt"

    "github.com/msft/bank"
)

func main() {
    fmt.Println(bank.Hello())
}
```

在 `src/bankapi/go.mod` 中，我们需要在本地引用 `bankcore` 程序包文件，如下所示：

Go

```go
module bankapi

go 1.14

require (
    github.com/msft/bank v0.0.1
)

replace github.com/msft/bank => ../bankcore
```

若要确保一切正常，请在 `$GOPATH/src/bankapi/` 目录中转到终端并运行以下命令：

sh

```sh
go run main.go
```

应会看到以下输出：

输出

```output
Hey! I'm working!
```

此输出确认你的项目文件已完全按预期正确设置。 接下来，我们将开始编写代码，以实现我们的网上银行系统的初始功能集。

## 开始编写测试

在开始编写程序之前，让我们先讨论一下测试并创建我们的第一个测试。 程序包测试为 Go 程序包的自动化测试提供支持。 测试对于确保代码按预期工作非常重要。 通常，程序包中的每个函数都应该有至少一个测试来确认功能。

编写代码时要遵循的一个良好做法是使用测试驱动开发 (TDD) 方法。 使用此方法时，我们将首先编写测试。 我们可以肯定那些测试会失败，因为它们测试的代码还不存在。 然后，我们将编写满足测试条件的代码。
创建测试文件

首先，我们需要创建用来保存 bankcore 程序包的所有测试的 Go 文件。 创建测试文件时，该文件的名称必须以 _test.go 结尾。 你可以将你想用的任何内容用作文件名的前半部分，但典型做法是使用你要测试的文件的名称。

此外，要编写的每个测试都必须是以 Test 开头的函数。 然后，你通常为你编写的测试编写一个描述性名称，例如 TestDeposit。

转到 $GOPATH/src/bankcore/ 位置，创建一个名为 bank_test.go 的文件，其中包含以下内容：
Go

package bank

import "testing"

func TestAccount(t *testing.T) {

}

打开一个终端，确保你处于 $GOPATH/src/bankcore/ 位置。 然后，使用以下命令在详细模式下运行测试：
sh

go test -v

Go 将查找所有 *_test.go 文件来运行测试，因此你应该会看到以下输出：
输出

=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
PASS
ok      github.com/msft/bank    0.391s

编写将失败的测试

编写任何代码之前，让我们先使用 TDD 为其编写一个将失败的测试。 使用以下代码修改 TestAccount 函数：
Go

package bank

import "testing"

func TestAccount(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    if account.Name == "" {
        t.Error("can't create an Account object")
    }
}

我们引入了一个尚未实现的用于帐户和客户的结构。 并且，我们使用 t.Error() 函数来指示，如果某件事情没有按预期的方式发生，测试将失败。

另请注意，测试具有创建帐户对象（尚不存在）的逻辑。 但是，我们此刻正在设计如何与我们的程序包进行交互。

备注

我们会将用于测试的代码提供给你，因为我们不想逐行解释。 但是，你的心智模型应该是这样的：一点一点地开始，根据需要进行多次迭代。

在我们的案例中，我们将只进行一次迭代：编写测试，确保它失败，然后编写满足测试条件的代码。 在你自己编写代码时，应该先从简单代码开始，逐步增加复杂性。

运行 go test -v 命令时，应该会在输出中看到一个将失败的测试：
输出

# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:6:13: undefined: Account
FAIL    github.com/msft/bank [build failed]

让我们暂时先把它放在这里。 我们将完成此测试，并在为网上银行系统编写逻辑时创建新的测试。

# 编写银行核心程序包

现在，我们已经有了与我们的测试文件一起运行的基础项目，接下来我们将开始编写代码来实现上一个单元的功能和需求。 我们将回到我们讨论过的一些主题，例如错误、结构和方法。

打开 `$GOPATH/src/bankcore/bank.go` 文件，删除 `Hello()` 函数，然后开始编写网上银行系统的核心逻辑。

## 为客户和帐户创建结构

让我们先创建一个 `Customer` 结构，其中将包含要成为银行客户的人员的姓名、地址和电话号码。 此外，我们还需要 `Account` 数据的结构。 由于一个客户可以有多个帐户，因此让我们将客户信息嵌入到帐户对象中。 基本上，我们将创建已在 `TestAccount` 测试中定义的内容。

我们需要的结构可能如下所示：

Go

```go
package bank

// Customer ...
type Customer struct {
    Name    string
    Address string
    Phone   string
}

// Account ...
type Account struct {
    Customer
    Number  int32
    Balance float64
}
```

现在，在终端中运行 `go test -v` 命令时，你应该会看到测试通过：

输出

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
PASS
ok      github.com/msft/bank    0.094s
```

由于我们已实现了 `Customer` 和 `Account` 的结构，因此此测试通过。 现在我们已经有了这些结构，接下来让我们编写一些方法，用于在银行的初始版本中添加所需的功能。 这些功能包括存款、取款和转账。

## 实现存款方法

我们需要从一种允许将资金添加到帐户的方法开始。 但在执行此操作之前，让我们在 `bank_test.go` 文件中创建 `TestDeposit` 函数：

Go

```go
func TestDeposit(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(10)

    if account.Balance != 10 {
        t.Error("balance is not being updated after a deposit")
    }
}
```

运行 `go test -v` 时，应该会在输出中看到一个将失败的测试：

输出

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:32:9: account.Deposit undefined (type Account has no field or method Deposit)
FAIL    github.com/msft/bank [build failed]
```

为了满足前面的测试，让我们创建 `Account` 结构的 `Deposit` 方法。如果收到的金额等于或小于零，该方法会返回一个错误。 否则，直接将收到的金额添加到帐户的余额。

将以下代码用于 `Deposit` 方法：

Go

```go
// Deposit ...
func (a *Account) Deposit(amount float64) error {
    if amount <= 0 {
        return errors.New("the amount to deposit should be greater than zero")
    }

    a.Balance += amount
    return nil
}
```

运行 `go test -v` 时，应该会看到测试通过：

输出

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
PASS
ok      github.com/msft/bank    0.193s
```

你还可以编写一个测试，用于确认当尝试存入的金额为负时会出现错误，如下所示：

Go

```go
func TestDepositInvalid(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    if err := account.Deposit(-10); err == nil {
        t.Error("only positive numbers should be allowed to deposit")
    }
}
```

运行 `go test -v` 命令时，应该会看到测试通过：

输出

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
PASS
ok      github.com/msft/bank    0.197s
```

 备注

从这里开始，我们将为每个方法编写一个测试用例。 但是，你应该为你的程序编写尽可能多的测试，以便涵盖预期的和意外的场景。 例如，在本例中，将对错误处理逻辑进行测试。

## 实现取款方法

在编写取款功能之前，让我们为其编写测试：

Go

```go
func TestWithdraw(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(10)
    account.Withdraw(10)

    if account.Balance != 0 {
        t.Error("balance is not being updated after withdraw")
    }
}
```

运行 `go test -v` 命令时，应该会在输出中看到一个将失败的测试：

输出

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:67:9: account.Withdraw undefined (type Account has no field or method Withdraw)
FAIL    github.com/msft/bank [build failed]
```

让我们实现 `Withdraw` 方法的逻辑，该方法将帐户余额减少的金额就是以参数方式收到的金额。 像之前一样，我们需要验证收到的数字是否大于零，以及帐户中的余额是否足够。

将以下代码用于 `Withdraw` 方法：

Go

```go
// Withdraw ...
func (a *Account) Withdraw(amount float64) error {
    if amount <= 0 {
        return errors.New("the amount to withdraw should be greater than zero")
    }

    if a.Balance < amount {
        return errors.New("the amount to withdraw should be greater than the account's balance")
    }

    a.Balance -= amount
    return nil
}
```

运行 `go test -v` 命令时，应该会看到测试通过：

输出

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
=== RUN   TestWithdraw
--- PASS: TestWithdraw (0.00s)
PASS
ok      github.com/msft/bank    0.250s
```

## 实现对账单方法

我们将编写一个简单的方法来输出对账单，其中包含帐户名称、帐号和余额。 但是，首先让我们创建 `TestStatement` 函数：

Go

```go
func TestStatement(t *testing.T) {
    account := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    account.Deposit(100)
    statement := account.Statement()
    if statement != "1001 - John - 100" {
        t.Error("statement doesn't have the proper format")
    }
}
```

运行 `go test -v` 时，应该会在输出中看到一个将失败的测试：

输出

```output
# github.com/msft/bank [github.com/msft/bank.test]
./bank_test.go:86:22: account.Statement undefined (type Account has no field or method Statement)
FAIL    github.com/msft/bank [build failed]
```

让我们编写 `Statement` 方法，该方法应返回一个简单的字符串。 （你稍后必须覆盖此方法，这是一项挑战。）使用以下代码：

Go

```go
// Statement ...
func (a *Account) Statement() string {
    return fmt.Sprintf("%v - %v - %v", a.Number, a.Name, a.Balance)
}
```

运行 `go test -v` 时，应该会看到测试通过：

输出

```output
=== RUN   TestAccount
--- PASS: TestAccount (0.00s)
=== RUN   TestDeposit
--- PASS: TestDeposit (0.00s)
=== RUN   TestDepositInvalid
--- PASS: TestDepositInvalid (0.00s)
=== RUN   TestWithdraw
--- PASS: TestWithdraw (0.00s)
=== RUN   TestStatement
--- PASS: TestStatement (0.00s)
PASS
ok      github.com/msft/bank    0.328s
```

接下来，请转到下一部分，编写 Web API 来公开 `Statement` 方法。

# 编写银行 API


现在，我们已经构建了网上银行的核心逻辑，接下来让我们构建一个 Web API，以通过浏览器（甚至是命令行）对该逻辑进行测试。 目前，我们不使用数据库来持久保存数据，因此必须创建一个全局变量，以便将所有帐户存储在内存中。

此外，我们将跳过测试部分，以免本指南的操作持续太长时间。 理想情况下，你应该遵循我们在构建核心程序包时遵循的相同方法，在编写代码之前编写测试。

## 在内存中设置帐户

我们将为帐户使用在程序启动时创建的内存映射，而不是使用数据库来持久保存数据。 另外，我们将使用映射通过帐号来访问帐户信息。

转到 `$GOPATH/src/bankapi/main.go` 文件，添加以下代码来创建全局 `accounts` 变量并使用一个帐户来初始化该变量。 （此代码类似于我们之前创建测试时添加的代码。）

Go

```go
package main

import (
    "github.com/msft/bank"
)

var accounts = map[float64]*bank.Account{}

func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }
}
```

使用 `go run main.go` 来运行程序以确保没有任何错误。 此程序目前不做任何其他事情，因此我们将添加逻辑来创建一个 Web API。

## 公开对账单方法

正如你在以前的模块中看到的那样，采用 Go 创建 Web API 非常简单。 我们将继续使用 `net/http` 程序包。 我们还将使用 `HandleFunc` 和 `ListenAndServe` 函数来公开终结点并启动服务器。 `HandleFunc` 函数需要一个你要公开的 URL 路径的名称，以及包含该终结点的逻辑的函数的名称。

首先，我们将公开用来输出某个帐户的对账单的功能。 将以下函数复制并粘贴到 `main.go` 中：

Go

```go
func statement(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            fmt.Fprintf(w, account.Statement())
        }
    }
}
```

`statement` 函数的第一个重点是接收用于将响应写回到浏览器的对象 (`w http.ResponseWriter`)。 它还会接收用于访问 HTTP 请求中的信息的请求对象 (`req *http.Request`)。

然后请注意，我们使用 `req.URL.Query().Get()` 函数从查询字符串读取参数。 这是我们将通过 HTTP 调用发送的帐号。 我们将使用该值来访问帐户映射并获取其信息。

由于我们要从用户那里获取数据，因此应包括一些验证以避免出现故障。 当我们知道自己具有有效帐号后，就可以调用 `Statement()` 方法，并将它返回的字符串输出到浏览器 (`fmt.Fprintf(w, account.Statement())`)。

现在，修改 `main()` 函数，使其类似于：

Go

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

如果在运行程序 (`go run main.go`) 时未看到任何错误或输出，则表明它在正常运行。 打开一个 Web 浏览器并输入 URL `http://localhost:8000/statement?number=1001`，或者运行以下命令：

sh

```sh
curl http://localhost:8000/statement\?number=1001
```

应会看到以下输出：

输出

```output
1001 - John - 0
```

## 公开存款方法

接下来，我们将继续使用相同的方法来公开存款方法。 在本例中，我们要向内存中的帐户增加资金。 每次调用 `Deposit()` 方法时，余额都应当增加。

在主程序中，添加一个 `deposit()` 函数，如下所示。 此函数从查询字符串中获取帐号，验证 `accounts` 映射中是否存在该帐户，验证要存入的金额是否为有效的数字，然后调用 `Deposit()` 方法。

Go

```go
func deposit(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")
    amountqs := req.URL.Query().Get("amount")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else if amount, err := strconv.ParseFloat(amountqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid amount number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            err := account.Deposit(amount)
            if err != nil {
                fmt.Fprintf(w, "%v", err)
            } else {
                fmt.Fprintf(w, account.Statement())
            }
        }
    }
}
```

请注意，此函数遵循类似的方法来获取和验证它从用户那里收到的数据。 我们还直接在 `if` 语句中声明和使用变量。 最后，将一些资金添加到帐户后，我们将输出对账单来查看新的帐户余额。

现在，你应公开一个调用 `deposit` 函数的 `/deposit` 终结点。 将 `main()` 函数修改为如下所示的形式：

Go

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    http.HandleFunc("/deposit", deposit)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

如果在运行程序 (`go run main.go`) 时未看到任何错误或输出，则表明它在正常运行。 打开一个 Web 浏览器并输入 URL `http://localhost:8000/deposit?number=1001&amount=100`，或者运行以下命令：

sh

```sh
curl http://localhost:8000/deposit\?number=1001&amount=100
```

应会看到以下输出：

输出

```output
1001 - John - 100
```

如果多次进行相同的调用，则帐户余额将继续增加。 尝试确认内存中的 `accounts` 映射在运行时是否更新。 如果你停止程序，则你存入的任何存款都会丢失，但在此初始版本中这是意料之中的情况。

## 公开取款方法

最后，让我们公开从帐户中取款的方法。 同样，我们将先在主程序中创建 `withdraw` 函数。 此函数将验证帐号信息，取款并输出从核心程序包收到的任何错误。 在主程序中添加以下函数：

Go

```go
func withdraw(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")
    amountqs := req.URL.Query().Get("amount")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else if amount, err := strconv.ParseFloat(amountqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid amount number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            err := account.Withdraw(amount)
            if err != nil {
                fmt.Fprintf(w, "%v", err)
            } else {
                fmt.Fprintf(w, account.Statement())
            }
        }
    }
}
```

现在，在 `main()` 函数中添加 `/withdraw` 终结点以公开你在 `withdraw()` 函数中实现的逻辑。 将 `main()` 函数修改为如下所示的形式：

Go

```go
func main() {
    accounts[1001] = &bank.Account{
        Customer: bank.Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number: 1001,
    }

    http.HandleFunc("/statement", statement)
    http.HandleFunc("/deposit", deposit)
    http.HandleFunc("/withdraw", withdraw)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}
```

如果在运行程序 (`go run main.go`) 时未看到任何错误或输出，则表明它在正常运行。 打开一个 Web 浏览器并输入 URL `http://localhost:8000/withdraw?number=1001&amount=100`，或者运行以下命令：

sh

```sh
curl http://localhost:8000/withdraw\?number=1001&amount=100
```

应会看到以下输出：

输出

```output
the amount to withdraw should be greater than the account's balance
```

请注意，我们收到的错误来自核心程序包。 当程序启动时，帐户余额为零。 因此，你无法提取任何金额的存款。 多次调用 `/deposit` 终结点来添加资金，并再次调用 `/withdraw` 终结点来确认它正常运行：

sh

```sh
curl http://localhost:8000/deposit\?number=1001&amount=100
curl http://localhost:8000/deposit\?number=1001&amount=100
curl http://localhost:8000/deposit\?number=1001&amount=100
curl http://localhost:8000/withdraw\?number=1001&amount=100
```

应会看到以下输出：

输出

```output
1001 - John - 200
```

就这么简单！ 你已创建了一个 Web API，用于公开你从头构建的一个程序包中的功能。 请转到下一部分继续进行练习。 这次，你将编写自己的解决方案来完成一项挑战。

# 挑战 - 完成银行项目功能

你的程序已具备一些基本功能。 但是，它还缺少一项功能：向其他帐户转账的功能。 这一挑战包括添加该功能，以及我们认为可为现有 API 增加价值的另一项功能。

## 实现转账方法

若要创建转账方法，应牢记以下几点：

- 你需要实现向其他帐户转账的功能。 在本例中，你必须使用至少两个帐户来初始化程序，而不是像之前一样只使用一个帐户。
- 由于你要在核心程序包中添加新方法，因此请首先创建测试用例，以确保你编写正确的逻辑来进行转账。 请密切注意在函数与指针之间进行通信的方式。
- 你的转账方法应当接收你要转账的金额以及你将在其中增加资金的帐户对象。 请确保重用存款和取款方法以避免重复（特别是对于错误处理）。
- 请记住，如果你没有足够的资金，则无法向其他帐户转账。

## 修改对账单终结点以返回 JSON 对象

目前，`/statement` 终结点会返回一个字符串。如果你想将其公开为 API，则该字符串没有用处。 修改终结点以采用 JSON 格式返回帐户对象：

输出

```output
"{\"Name\":\"John\",\"Address\":\"Los Angeles, California\",\"Phone\":\"(213) 555 0147\",\"Number\":1001,\"Balance\":0}"
```

我们希望你进行此更改，因为使用你的核心程序包的用户有可能希望实施不同的对账单方法来更改输出。 因此，你需要进行适当的更改，以使核心程序包具有可扩展性。 换句话说，你需要执行以下操作：

1. 创建一个包含 `Statement() string` 函数的接口。

2. 在核心程序包中创建一个新的 `Statement()` 函数，该函数接收你以参数形式创建的接口。 此函数应该调用你的结构已有的 `Statement()` 方法。

   当你执行此操作时，系统会允许你创建自定义 `Account` 结构和自定义 `Statement()` 方法。 如果不记得如何执行此操作，你可以返回到有关结构（嵌入）和接口的模块。

祝你编码愉快！

## 银行核心测试：bank_test.go

Go

```go
//...

func TestTransfer(t *testing.T) {
    accountA := Account{
        Customer: Customer{
            Name:    "John",
            Address: "Los Angeles, California",
            Phone:   "(213) 555 0147",
        },
        Number:  1001,
        Balance: 0,
    }

    accountB := Account{
        Customer: Customer{
            Name:    "Mark",
            Address: "Irvine, California",
            Phone:   "(949) 555 0198",
        },
        Number:  1001,
        Balance: 0,
    }

    accountA.Deposit(100)
    err := accountA.Transfer(50, &accountB)

    if accountA.Balance != 50 && accountB.Balance != 50 {
        t.Error("transfer from account A to account B is not working", err)
    }
}
```

## 银行核心：bank.go

Go

```go
package bank

import (
    "errors"
    "fmt"
)

//...

// Transfer function
func (a *Account) Transfer(amount float64, dest *Account) error {
    if amount <= 0 {
        return errors.New("the amount to transfer should be greater than zero")
    }

    if a.Balance < amount {
        return errors.New("the amount to transfer should be greater than the account's balance")
    }

    a.Withdraw(amount)
    dest.Deposit(amount)
    return nil
}

// Bank ...
type Bank interface {
    Statement() string
}

// Statement ...
func Statement(b Bank) string {
    return b.Statement()
}
```

## 银行 API：main.go

Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"

    "github.com/msft/bank"
)

var accounts = map[float64]*CustomAccount{}

func main() {
    accounts[1001] = &CustomAccount{
        Account: &bank.Account{
            Customer: bank.Customer{
                Name:    "John",
                Address: "Los Angeles, California",
                Phone:   "(213) 555 0147",
            },
            Number: 1001,
        },
    }

    accounts[1002] = &CustomAccount{
        Account: &bank.Account{
            Customer: bank.Customer{
                Name:    "Mark",
                Address: "Irvine, California",
                Phone:   "(949) 555 0198",
            },
            Number: 1002,
        },
    }

    http.HandleFunc("/statement", statement)
    http.HandleFunc("/deposit", deposit)
    http.HandleFunc("/withdraw", withdraw)
    http.HandleFunc("/transfer", transfer)
    log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

//...

func transfer(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")
    amountqs := req.URL.Query().Get("amount")
    destqs := req.URL.Query().Get("dest")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    if number, err := strconv.ParseFloat(numberqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else if amount, err := strconv.ParseFloat(amountqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid amount number!")
    } else if dest, err := strconv.ParseFloat(destqs, 64); err != nil {
        fmt.Fprintf(w, "Invalid account destination number!")
    } else {
        if accountA, ok := accounts[number]; !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else if accountB, ok := accounts[dest]; !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", dest)
        } else {
            err := accountA.Transfer(amount, accountB.Account)
            if err != nil {
                fmt.Fprintf(w, "%v", err)
            } else {
                fmt.Fprintf(w, accountA.Statement())
            }
        }
    }
}

func statement(w http.ResponseWriter, req *http.Request) {
    numberqs := req.URL.Query().Get("number")

    if numberqs == "" {
        fmt.Fprintf(w, "Account number is missing!")
        return
    }

    number, err := strconv.ParseFloat(numberqs, 64)
    if err != nil {
        fmt.Fprintf(w, "Invalid account number!")
    } else {
        account, ok := accounts[number]
        if !ok {
            fmt.Fprintf(w, "Account with number %v can't be found!", number)
        } else {
            json.NewEncoder(w).Encode(bank.Statement(account))
        }
    }
}

// CustomAccount ...
type CustomAccount struct {
    *bank.Account
}

// Statement ...
func (c *CustomAccount) Statement() string {
    json, err := json.Marshal(c)
    if err != nil {
        return err.Error()
    }

    return string(json)
}
```

