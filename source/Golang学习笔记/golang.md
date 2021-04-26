title: Golang 基础语法
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - Golang
type: article
---

# Golang 基础语法

## Go 命令行操作

## CGO

工具 cgo 提供了对 FFI（外部函数接口）的支持，能够使用 Go 代码安全地调用 C 语言库，你可以访问 cgo 文档主页：[http://golang.org/cmd/cgo](http://golang.org/cmd/cgo)。cgo 会替代 Go 编译器来产生可以组合在同一个包中的 Go 和 C 代码。在实际开发中一般使用 cgo 创建单独的 C 代码包。

如果你想要在你的 Go 程序中使用 cgo，则必须在单独的一行使用 `import "C"` 来导入，一般来说你可能还需要 `import "unsafe"`。

然后，你可以在 `import "C"` 之前使用注释（单行或多行注释均可）的形式导入 C 语言库（甚至有效的 C 语言代码），它们之间没有空行，例如：

// #include <stdio.h>
// #include <stdlib.h>
import "C"
名称 "C" 并不属于标准库的一部分，这只是 cgo 集成的一个特殊名称用于引用 C 的命名空间。在这个命名空间里所包含的 C 类型都可以被使用，例如 `C.uint`、`C.long` 等等，还有 libc 中的函数 `C.random()` 等也可以被调用。

当你想要使用某个类型作为 C 中函数的参数时，必须将其转换为 C 中的类型，反之亦然，例如：

```go
var i int
C.uint(i) // 从 Go 中的 int 转换为 C 中的无符号 int
int(C.random()) // 从 C 中 random() 函数返回的 long 转换为 Go 中的 int
```

下面的 2 个 Go 函数 `Random()` 和 `Seed()` 分别调用了 C 中的 `C.random()` 和 `C.srandom()`。

示例 3.2 [c1.go](examples/chapter_3/CandGo/c1.go)

```go
package rand

// #include <stdlib.h>
import "C"

func Random() int {
  return int(C.random())
}

func Seed(i int) {
  C.srandom(C.uint(i))
}
```

C 当中并没有明确的字符串类型，如果你想要将一个 string 类型的变量从 Go 转换到 C 时，可以使用 `C.CString(s)`；同样，可以使用 `C.GoString(cs)` 从 C 转换到 Go 中的 string 类型。

Go 的内存管理机制无法管理通过 C 代码分配的内存。

开发人员需要通过手动调用 `C.free` 来释放变量的内存：

```go
defer C.free(unsafe.Pointer(Cvariable))
```

这一行最好紧跟在使用 C 代码创建某个变量之后，这样就不会忘记释放内存了。下面的代码展示了如何使用 cgo 创建变量、使用并释放其内存：

示例 3.3 [c2.go](examples/chapter_3/CandGo/c2.go)

```go
package print

// #include <stdio.h>
// #include <stdlib.h>
import "C"
import "unsafe"

func Print(s string) {
	cs := C.CString(s)
	defer C.free(unsafe.Pointer(cs))
	C.fputs(cs, (*C.FILE)(C.stdout))
}
```

**构建 cgo 包**

你可以在使用将会在第 9.5 节讲到的 Makefile 文件（因为我们使用了一个独立的包），除了使用变量 GOFILES 之外，还需要使用变量 CGOFILES 来列出需要使用 cgo 编译的文件列表。例如，示例 3.2 中的代码就可以使用包含以下内容的 Makefile 文件来编译，你可以使用 gomake 或 make：

include $(GOROOT)/src/Make.inc
TARG=rand
CGOFILES=\
c1.go\
include $(GOROOT)/src/Make.pkg

## 文件名、关键字与标识符


Go 的源文件以 `.go` 为后缀名存储在计算机中，这些文件名均由小写字母组成，如 `scanner.go` 。如果文件名由多个部分组成，则使用下划线 `_` 对它们进行分隔，如 `scanner_test.go` 。文件名不包含空格或其他特殊字符。

一个源文件可以包含任意多行的代码，Go 本身没有对源文件的大小进行限制。

你会发现在 Go 代码中的几乎所有东西都有一个名称或标识符。另外，Go 语言也是区分大小写的，这与 C 家族中的其它语言相同。有效的标识符必须以字母（可以使用任何 UTF-8 编码的字符或 `_`）开头，然后紧跟着 0 个或多个字符或 Unicode 数字，如：X56、group1、_x23、i、өԑ12。

以下是无效的标识符：

- 1ab（以数字开头）
- case（Go 语言的关键字）
- a+b（运算符是不允许的）

`_` 本身就是一个特殊的标识符，被称为空白标识符。它可以像其他标识符那样用于变量的声明或赋值（任何类型都可以赋值给它），但任何赋给这个标识符的值都将被抛弃，因此这些值不能在后续的代码中使用，也不可以使用这个标识符作为变量对其它变量进行赋值或运算。

在编码过程中，你可能会遇到没有名称的变量、类型或方法。虽然这不是必须的，但有时候这样做可以极大地增强代码的灵活性，这些变量被统称为匿名变量。

下面列举了 Go 代码中会使用到的 25 个关键字或保留字：

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>break</td>
    <td>default</td>
    <td>func</td>
    <td>interface</td>
    <td>select</td>
  </tr>
  <tr>
    <td>case</td>
    <td>defer</td>
    <td>go</td>
    <td>map</td>
    <td>struct</td>
  </tr>
  <tr>
    <td>chan</td>
    <td>else</td>
    <td>goto</td>
    <td>package</td>
    <td>switch</td>
  </tr>
  <tr>
    <td>const</td>
    <td>fallthrough</td>
    <td>if</td>
    <td>range</td>
    <td>type</td>
  </tr>
  <tr>
    <td>continue</td>
    <td>for</td>
    <td>import</td>
    <td>return</td>
    <td>var</td>
  </tr>
</table>

之所以刻意地将 Go 代码中的关键字保持的这么少，是为了简化在编译过程第一步中的代码解析。和其它语言一样，关键字不能够作标识符使用。

除了以上介绍的这些关键字，Go 语言还有 36 个预定义标识符，其中包含了基本类型的名称和一些基本的内置函数（第 6.5 节），它们的作用都将在接下来的章节中进行进一步地讲解。

<table class="table table-bordered table-striped table-condensed">
  <tr>
    <td>append</td>
    <td>bool</td>
    <td>byte</td>
    <td>cap</td>
    <td>close</td>
    <td>complex</td>
    <td>complex64</td>
    <td>complex128</td>
    <td>uint16</td>
  </tr>
  <tr>
    <td>copy</td>
    <td>false</td>
    <td>float32</td>
    <td>float64</td>
    <td>imag</td>
    <td>int</td>
    <td>int8</td>
    <td>int16</td>
    <td>uint32</td>
  </tr>
  <tr>
    <td>int32</td>
    <td>int64</td>
    <td>iota</td>
    <td>len</td>
    <td>make</td>
    <td>new</td>
    <td>nil</td>
    <td>panic</td>
    <td>uint64</td>
  </tr>
  <tr>
    <td>print</td>
    <td>println</td>
    <td>real</td>
    <td>recover</td>
    <td>string</td>
    <td>true</td>
    <td>uint</td>
    <td>uint8</td>
    <td>uintptr</td>
  </tr>
</table>

程序一般由关键字、常量、变量、运算符、类型和函数组成。

程序中可能会使用到这些分隔符：括号 `()`，中括号 `[]` 和大括号 `{}`。

程序中可能会使用到这些标点符号：`.`、`,`、`;`、`:` 和 `…`。

程序的代码通过语句来实现结构化。每个语句不需要像 C 家族中的其它语言一样以分号 `;` 结尾，因为这些工作都将由 Go 编译器自动完成。

如果你打算将多个语句写在同一行，它们则必须使用 `;` 人为区分，但在实际开发中我们并不鼓励这种做法。

## 变量

## 常量

## 类型


## 包


包是结构化代码的一种方式：每个程序都由包（通常简称为 pkg）的概念组成，可以使用自身的包或者从其它包中导入内容。

如同其它一些编程语言中的类库或命名空间的概念，每个 Go 文件都属于且仅属于一个包。一个包可以由许多以 `.go` 为扩展名的源文件组成，因此文件名和包名一般来说都是不相同的。

你必须在源文件中非注释的第一行指明这个文件属于哪个包，如：`package main`。`package main`表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 `main` 的包。

一个应用程序可以包含不同的包，而且即使你只使用 main 包也不必把所有的代码都写在一个巨大的文件里：你可以用一些较小的文件，并且在每个文件非注释的第一行都使用 `package main` 来指明这些文件都属于 main 包。如果你打算编译包名不是为 main 的源文件，如 `pack1`，编译后产生的对象文件将会是 `pack1.a` 而不是可执行程序。另外要注意的是，所有的包名都应该使用小写字母。


**标准库**

在 Go 的安装文件里包含了一些可以直接使用的包，即标准库。在 Windows 下，标准库的位置在 Go 根目录下的子目录 `pkg\windows_386` 中；在 Linux 下，标准库在 Go 根目录下的子目录 `pkg\linux_amd64` 中（如果是安装的是 32 位，则在 `linux_386` 目录中）。一般情况下，标准包会存放在 `$GOROOT/pkg/$GOOS_$GOARCH/` 目录下。

Go 的标准库包含了大量的包（如：fmt 和 os），但是你也可以创建自己的包（第 9 章）。

如果想要构建一个程序，则包和包内的文件都必须以正确的顺序进行编译。包的依赖关系决定了其构建顺序。

属于同一个包的源文件必须全部被一起编译，一个包即是编译时的一个单元，因此根据惯例，每个目录都只包含一个包。

**如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。**

Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 `.o` 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。

如果 `A.go` 依赖 `B.go`，而 `B.go` 又依赖 `C.go`：

- 编译 `C.go`, `B.go`, 然后是 `A.go`.
- 为了编译 `A.go`, 编译器读取的是 `B.o` 而不是 `C.o`.

这种机制对于编译大型的项目时可以显著地提升编译速度。


**每一段代码只会被编译一次**

一个 Go 程序是通过 `import` 关键字将一组包链接在一起。

`import "fmt"` 告诉 Go 编译器这个程序需要使用 `fmt` 包（的函数，或其他元素），`fmt` 包实现了格式化 IO（输入/输出）的函数。包名被封闭在半角双引号 `""` 中。如果你打算从已编译的包中导入并加载公开声明的方法，不需要插入已编译包的源代码。

如果需要多个包，它们可以被分别导入：

```go
import "fmt"
import "os"
```

或：

```go
import "fmt"; import "os"
```

但是还有更短且更优雅的方法（被称为因式分解关键字，该方法同样适用于 const、var 和 type 的声明或定义）：

```go
import (
   "fmt"
   "os"
)
```

它甚至还可以更短的形式，但使用 gofmt 后将会被强制换行：

```go
import ("fmt"; "os")
```

当你导入多个包时，最好按照字母顺序排列包名，这样做更加清晰易读。

如果包名不是以 `.` 或 `/` 开头，如 `"fmt"` 或者 `"container/list"`，则 Go 会在全局文件进行查找；如果包名以 `./` 开头，则 Go 会在相对目录中查找；如果包名以 `/` 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找。

*译者注：以相对路径在GOPATH下导入包会产生报错信息*

*报错信息：local import "./XXX" in non-local package*

*引用：[Go programs cannot use relative import paths within a work space.](https://golang.org/cmd/go/#hdr-Relative_import_paths )*

*注解：在GOPATH外可以以相对路径的形式执行go build（go install 不可以）*

导入包即等同于包含了这个包的所有的代码对象。

除了符号 `_`，包中所有代码对象的标识符必须是唯一的，以避免名称冲突。但是相同的标识符可以在不同的包中使用，因为可以使用包名来区分它们。

包通过下面这个被编译器强制执行的规则来决定是否将自身的代码对象暴露给外部文件：

**可见性规则**

当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。

（大写字母可以使用任何 Unicode 编码的字符，比如希腊文，不仅仅是 ASCII 码中的大写字母）。

因此，在导入一个外部包后，能够且只能够访问该包中导出的对象。

假设在包 pack1 中我们有一个变量或函数叫做 Thing（以 T 开头，所以它能够被导出），那么在当前包中导入 pack1 包，Thing 就可以像面向对象语言那样使用点标记来调用：`pack1.Thing`（pack1 在这里是不可以省略的）。

因此包也可以作为命名空间使用，帮助避免命名冲突（名称冲突）：两个包中的同名变量的区别在于他们的包名，例如 `pack1.Thing` 和 `pack2.Thing`。

你可以通过使用包的别名来解决包名之间的名称冲突，或者说根据你的个人喜好对包名进行重新设置，如：`import fm "fmt"`。下面的代码展示了如何使用包的别名：

示例 4.2 [alias.go](examples/chapter_4/alias.go)

```go
package main

import fm "fmt" // alias3

func main() {
   fm.Println("hello, world")
}
```

**注意事项**

如果你导入了一个包却没有使用它，则会在构建程序时引发错误，如 `imported and not used: os`，这正是遵循了 Go 的格言：“没有不必要的代码！“。

**包的分级声明和初始化**

你可以在使用 `import` 导入包之后定义或声明 0 个或多个常量（const）、变量（var）和类型（type），这些对象的作用域都是全局的（在本包范围内），所以可以被本包中所有的函数调用（如 [gotemplate.go](examples/chapter_4/gotemplate.go) 源文件中的 c 和 v），然后声明一个或多个函数（func）。

## 函数


这是定义一个函数最简单的格式：

```go
func functionName()
```

你可以在括号 `()` 中写入 0 个或多个函数的参数（使用逗号 `,` 分隔），每个参数的名称后面必须紧跟着该参数的类型。

main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行该函数）。如果你的 main 包的源代码没有包含 main 函数，则会引发构建错误 `undefined: main.main`。main 函数既没有参数，也没有返回类型（与 C 家族中的其它语言恰好相反）。如果你不小心为 main 函数添加了参数或者返回类型，将会引发构建错误：

func main must have no arguments and no return values results.
在程序开始执行并完成初始化后，第一个调用（程序的入口点）的函数是 `main.main()`（如：C 语言），该函数一旦返回就表示程序已成功执行并立即退出。

函数里的代码（函数体）使用大括号 `{}` 括起来。

左大括号 `{` 必须与方法的声明放在同一行，这是编译器的强制规定，否则你在使用 gofmt 时就会出现错误提示：

`build-error: syntax error: unexpected semicolon or newline before {`
（这是因为编译器会产生 `func main() ;` 这样的结果，很明显这错误的）

**Go 语言虽然看起来不使用分号作为语句的结束，但实际上这一过程是由编译器自动完成，因此才会引发像上面这样的错误**

右大括号 `}` 需要被放在紧接着函数体的下一行。如果你的函数非常简短，你也可以将它们放在同一行：

```go
func Sum(a, b int) int { return a + b }
```

对于大括号 `{}` 的使用规则在任何时候都是相同的（如：if 语句等）。

因此符合规范的函数一般写成如下的形式：

```go
func functionName(parameter_list) (return_value_list) {
   …
}
```

其中：

- parameter_list 的形式为 (param1 type1, param2 type2, …)
- return_value_list 的形式为 (ret1 type1, ret2 type2, …)

只有当某个函数需要被外部包调用的时候才使用大写字母开头，并遵循 Pascal 命名法；否则就遵循骆驼命名法，即第一个单词的首字母小写，其余单词的首字母大写。

下面这一行调用了 `fmt` 包中的 `Println` 函数，可以将字符串输出到控制台，并在最后自动增加换行字符 `\n`：

```go
fmt.Println（"hello, world"）
```

使用 `fmt.Print("hello, world\n")` 可以得到相同的结果。

`Print` 和 `Println` 这两个函数也支持使用变量，如：`fmt.Println(arr)`。如果没有特别指定，它们会以默认的打印格式将变量 `arr` 输出到控制台。

单纯地打印一个字符串或变量甚至可以使用预定义的方法来实现，如：`print`、`println：print("ABC")`、`println("ABC")`、`println(i)`（带一个变量 i）。

这些函数只可以用于调试阶段，在部署程序的时候务必将它们替换成 `fmt` 中的相关函数。

当被调用函数的代码执行到结束符 `}` 或返回语句时就会返回，然后程序继续执行调用该函数之后的代码。

程序正常退出的代码为 0 即 `Program exited with code 0`；如果程序因为异常而被终止，则会返回非零值，如：1。这个数值可以用来测试是否成功执行一个程序。

## 4.2.3 注释

示例 4.2 [hello_world2.go](examples/chapter_4/hello_world2.go)

```go
package main

import "fmt" // Package implementing formatted I/O.

func main() {
   fmt.Printf("Καλημέρα κόσμε; or こんにちは 世界\n")
}
```

上面这个例子通过打印 `Καλημέρα κόσμε; or こんにちは 世界` 展示了如何在 Go 中使用国际化字符，以及如何使用注释。

注释不会被编译，但可以通过 godoc 来使用（第 3.6 节）。

单行注释是最常见的注释形式，你可以在任何地方使用以 `//` 开头的单行注释。多行注释也叫块注释，均已以 `/*` 开头，并以 `*/` 结尾，且不可以嵌套使用，多行注释一般用于包的文档描述或注释成块的代码片段。

每一个包应该有相关注释，在 package 语句之前的块注释将被默认认为是这个包的文档说明，其中应该提供一些相关信息并对整体功能做简要的介绍。一个包可以分散在多个文件中，但是只需要在其中一个进行注释说明即可。当开发人员需要了解包的一些情况时，自然会用 godoc 来显示包的文档说明，在首行的简要注释之后可以用成段的注释来进行更详细的说明，而不必拥挤在一起。另外，在多段注释之间应以空行分隔加以区分。

示例：

```go
// Package superman implements methods for saving the world.
//
// Experience has shown that a small number of procedures can prove
// helpful when attempting to save the world.
package superman
```

几乎所有全局作用域的类型、常量、变量、函数和被导出的对象都应该有一个合理的注释。如果这种注释（称为文档注释）出现在函数前面，例如函数 Abcd，则要以 `"Abcd..."` 作为开头。

示例：

```go
// enterOrbit causes Superman to fly into low Earth orbit, a position
// that presents several possibilities for planet salvation.
func enterOrbit() error {
   ...
}
```

godoc 工具（第 3.6 节）会收集这些注释并产生一个技术文档。

## 4.2.4 类型

变量（或常量）包含数据，这些数据可以有不同的数据类型，简称类型。使用 var 声明的变量的值会自动初始化为该类型的零值。类型定义了某个变量的值的集合与可对其进行操作的集合。

类型可以是基本类型，如：int、float、bool、string；结构化的（复合的），如：struct、array、slice、map、channel；只描述类型的行为的，如：interface。

结构化的类型没有真正的值，它使用 nil 作为默认值（在 Objective-C 中是 nil，在 Java 中是 null，在 C 和 C++ 中是NULL或 0）。值得注意的是，Go 语言中不存在类型继承。

函数也可以是一个确定的类型，就是以函数作为返回类型。这种类型的声明要写在函数名和可选的参数列表之后，例如：

```go
func FunctionName (a typea, b typeb) typeFunc
```

你可以在函数体中的某处返回使用类型为 typeFunc 的变量 var：

```go
return var
```

一个函数可以拥有多返回值，返回类型之间需要使用逗号分割，并使用小括号 `()` 将它们括起来，如：

```go
func FunctionName (a typea, b typeb) (t1 type1, t2 type2)
```

示例： 函数 Atoi (第 4.7 节)：`func Atoi(s string) (i int, err error)`

返回的形式：

```go
return var1, var2
```

这种多返回值一般用于判断某个函数是否执行成功（true/false）或与其它返回值一同返回错误消息（详见之后的并行赋值）。

使用 type 关键字可以定义你自己的类型，你可能想要定义一个结构体(第 10 章)，但是也可以定义一个已经存在的类型的别名，如：

```go
type IZ int
```

**这里并不是真正意义上的别名，因为使用这种方法定义之后的类型可以拥有更多的特性，且在类型转换时必须显式转换。**

然后我们可以使用下面的方式声明变量：

```go
var a IZ = 5
```

这里我们可以看到 int 是变量 a 的底层类型，这也使得它们之间存在相互转换的可能（第 4.2.6 节）。

如果你有多个类型需要定义，可以使用因式分解关键字的方式，例如：

```go
type (
   IZ int
   FZ float64
   STR string
)
```

每个值都必须在经过编译后属于某个类型（编译器必须能够推断出所有值的类型），因为 Go 语言是一种静态类型语言。

## 4.2.5 Go 程序的一般结构

下面的程序可以被顺利编译但什么都做不了，不过这很好地展示了一个 Go 程序的首选结构。这种结构并没有被强制要求，编译器也不关心 main 函数在前还是变量的声明在前，但使用统一的结构能够在从上至下阅读 Go 代码时有更好的体验。

所有的结构将在这一章或接下来的章节中进一步地解释说明，但总体思路如下：

- 在完成包的 import 之后，开始对常量、变量和类型的定义或声明。
- 如果存在 init 函数的话，则对该函数进行定义（这是一个特殊的函数，每个含有该函数的包都会首先执行这个函数）。
- 如果当前包是 main 包，则定义 main 函数。
- 然后定义其余的函数，首先是类型的方法，接着是按照 main 函数中先后调用的顺序来定义相关函数，如果有很多函数，则可以按照字母顺序来进行排序。

示例 4.4 [gotemplate.go](examples/chapter_4/gotemplate.go)

```go
package main

import (
   "fmt"
)

const c = "C"

var v int = 5

type T struct{}

func init() { // initialization of package
}

func main() {
   var a int
   Func1()
   // ...
   fmt.Println(a)
}

func (t T) Method1() {
   //...
}

func Func1() { // exported function Func1
   //...
}
```

Go 程序的执行（程序启动）顺序如下：

1. 按顺序导入所有被 main 包引用的其它包，然后在每个包中执行如下流程：
2. 如果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次。
3. 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 init 函数的话，则调用该函数。
4. 在完成这一切之后，main 也执行同样的过程，最后调用 main 函数开始执行程序。

## 4.2.6 类型转换

在必要以及可行的情况下，一个类型的值可以被转换成另一种类型的值。由于 Go 语言不存在隐式类型转换，因此所有的转换都必须显式说明，就像调用一个函数一样（类型在这里的作用可以看作是一种函数）：

```go
valueOfTypeB = typeB(valueOfTypeA)
```

**类型 B 的值 = 类型 B(类型 A 的值)**

示例：

```go
a := 5.0
b := int(a)
```

但这只能在定义正确的情况下转换成功，例如从一个取值范围较小的类型转换到一个取值范围较大的类型（例如将 int16 转换为 int32）。当从一个取值范围较大的转换到取值范围较小的类型时（例如将 int32 转换为 int16 或将 float32 转换为 int），会发生精度丢失（截断）的情况。当编译器捕捉到非法的类型转换时会引发编译时错误，否则将引发运行时错误。

具有相同底层类型的变量之间可以相互转换：

```go
var a IZ = 5
c := int(a)
d := IZ(c)
```

## 4.2.7 Go 命名规范

干净、可读的代码和简洁性是 Go 追求的主要目标。通过 gofmt 来强制实现统一的代码风格。Go 语言中对象的命名也应该是简洁且有意义的。像 Java 和 Python 中那样使用混合着大小写和下划线的冗长的名称会严重降低代码的可读性。名称不需要指出自己所属的包，因为在调用的时候会使用包名作为限定符。返回某个对象的函数或方法的名称一般都是使用名词，没有 `Get...` 之类的字符，如果是用于修改某个对象，则使用 `SetName`。有必须要的话可以使用大小写混合的方式，如 MixedCaps 或 mixedCaps，而不是使用下划线来分割多个名称。

## 切片

## 数组

## 指针

## 结构

## 方法

## 接口

## 协程（Goroutine）

## 信道

## 缓冲区

## 选择（Select）

## 互斥锁（Mutex）

## 延迟（Defer）机制

## 错误

## 严重（Panic）异常

## 恢复（Recover）

# Go 模组

## Go 依赖管理工具

## 语义版本控制（Semantic Versioning）

## 版本、脚本、存储库等其它特性

# SQL 基础

# 基本开发技能

## Git

## HTTP/HTTPS

## 数据结构与算法

## Scrum、看板、项目管理

## OAuth

## JWT

## SOLID 是面向对象设计5大重要原则

* [单一职责原则（SRP）](https://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label0)
* [开放封闭原则（OCP）](https://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label1)
* [里氏替换原则（LSP）](https://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label2)
* [接口隔离原则（ISP）](https://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label3)
* [依赖倒置原则（DIP）](https://www.cnblogs.com/OceanEyes/p/overview-of-solid-principles.html#_label4)

## KISS原则和YAGNI原则：

```


# Kiss原则：尽量保持代码简单

# 代码少并不一定满足KISS原则，我们还是考虑逻辑的复杂度，代码的可读性等

# 本身负责的问题，用复杂的方法解决，并不违背kiss原则：
	例如：KMP 算法本身具有逻辑复杂、实现难度大、可读性差的特点，但是对于当我们需要处理长文本字符串匹配问题，遇到系统性能瓶颈的时候我们应该考虑使用kmp算法

# 同样的代码，在某个业务场景下满足 KISS 原则，换一个应用场景可能就不满足了
	我们平时开发使用普通的工具类的时候就可以，但是如果考虑系统系统性能的时候，这时候我们就需要自己实现工具类来提高性能

# 如何写出满足KISS原则的代码？
	1.不要使用同时可能不懂的技术来实现代码
	2.不要重复造轮子，要善于使用已经有的工具类
	3.不要过度的优化，不要过度使用一些奇技淫巧（位运算，复杂的if-else）来优化代码，来牺牲代码的可读性


#建议：
	在开发中，不要过度的设计，其实用简单的方法解决复杂的问题，更能体现一个人的能力



#YAGNI原则：不要去设计当前用不到的功能，不要去编写当前用不到的代码
	例如：
		1.系统暂时使用redis来存储配置信息，以后可能会用到zookeeper，我们就没有必要提前写zookeeper代码，但是我们还是要预留好扩展点，等到需要的时候，再去实现这部分代码
		2.我们在项目中不要提前引入不需要依赖的开发包
#KISS和YAGNI区别
	KISS是“如何做”的问题（尽量保持简单），而YAGNI是“要不要做”的问题（当前不需要就不去做）

# 针对什么时候重复造轮子，什么时候应该使用现成的工具类，开源框架？
	# 什么时候重复造轮子
		1.想学习轮子的原理
		2.现有的轮子不能满足性能的需求，也没有好的替代品（因为轮子一般会考虑大而全的问题，这对性能有损耗）
		3.小而简单的问题
	# 什么时候应该使用现成的工具类，开源框架
		对于工作中，应该使用工具类和开源框架，没有必要去自己造轮子，增加了开发成本，增加了开发周期
```

# 命令行界面

## cobra

## urfave/cli

# 网页架构+路由

## Echo

## Beego

## Gin

# 对象关系映射（ORMs）

## Gorm

## Xorm

# 高速缓存（Caching）

## GCache

## 分布式缓存（Distributed Cache）

- Go-Redis

# 数据库

## PostgreSQL

## MongoDB

## Redis

## ElasticSearch

# 日志（Log Management System）

- Sentry.io

## 日志框架（Log Framwork）

- Zap
- Logrus

# 实时通讯

- Melody
- Centrifugo

# API 客户端（API Clients）

- GraphQL
- REST

# 测试

## 单元测试（Unit Testing）

### 模拟（Mocking）

### 框架

### 断言

## 行为测试（Behavior Testing）

## 集成测试（Integration Testing）

## 端对端测试（E2E Testing）

# 任务调度

- gron

# RPC

## gRPC-Go

## gRPC-gateway

## Protocol Buffers

# 微服务框架

- Go-Kit
- Micro
- rpcx
- istio

# 单元测试
如果测试文件与源文件没在一个文件夹，在子文件夹加`-coverpkg=./...`参数
`gocov` 是另一个工具(https://github.com/axw/gocov)
```
go test -v ./... -coverprofile=cover.out
go test -v ./... -coverprofile=cover.out -coverpkg=./...
go tool cover -html=cover.out
gocov test | gocov report > b.out
gocov test -v ./... -coverpkg=./...  | gocov report > b.out
```

# golint
错误检查以及 min-confidence 不同等级的错误提示
未指出 min-confidence 在不同等级有不同区别的提示，默认都是严格显示的。当 mini-confidence 大于 1 时，所有错误都会被接受。

1. **exported type T should have comment or be unexported**

   ```
      要暴露出去的结构体或者函数都应该写注释。
   1
   ```

2. **comment on exported type U should be of the form "U …"**

   ```
      定义的type要被使用，对函数或者type写注释，注释要以函数名或type名字开头。
      在以 "_test.go" 结尾的文件，不应该要求写注释
   12
   ```

   eg:

   ```go
       	 // H is a type.
       	 type H int
   12
   ```

3. **a blank import should be only in a main or test package, or have a comment justifying it**

   ```
      import 时要使用 '_' 时，要写注释。在 main package 或者测试文件中不需要写注释。
   1
   ```

4. **exported const Whatsit should have comment (or a comment on this block) or be unexported**

   ```
      const 模块要添加注释。
   1
   ```

5. **context.Context should be the first parameter of a function**

   ```
      context 参数要放在函数参数的第一个， 当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
   1
   ```

6. **should not use basic type string as key in context.WithValue**

   ```
      context.WithValue 的 key 参数不能是基本类型。
   1
   ```

7. **if block ends with a return statement, so drop this else and outdent its block**

   ```
      如果 else 中包含 return 语句， 则去掉整个 else 把 else 中的语句移到外面。
   1
   ```

8. **error should be the last type when returning multiple items**

   ```
      返回参数中有 error 时， 把 error 作为返回参数的最后一个参数。 当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
   1
   ```

9. **should replace errors.New(fmt.Sprintf(…)) with fmt.Errorf(…)**

   ```
      如果想打印指定的error内容， 应该使用 errors.New(fmt.Sprintf(...)) 。
   1
   ```

10. **error var should have name of the form errFoo**

    ```
      给 error.New() 声明一个变量应该使用 err 或者 Err 开头。  当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
    1
    ```

11. **error strings should not be capitalized or end with punctuation or a newline**

    ```
      error 信息不应该以大写字母开头 或者 以标点符号或者换行结尾。当 min-confidence 大于 0.8 时会忽略， 小于等于 0.8 会显示。
    1
    ```

12. **should not use dot imports**

    ```
      不应该使用 '.' 导入
    1
    ```

13. **should replace x += 1 with x++**

    ```
      x += 1 应该用 x++; y -= 1 应该用 y--. 当 min-confidence 大于 0.8 时会忽略， 小于等于0.8会显示。
    1
    ```

14. **don’t use an underscore in package name**

    ```
      包名中不应该使用 '_'. error 信息不应该以大写字母开头 或者 以标点符号或者换行结尾。当 min-confidence 大于 0.9 时会忽略， 小于等于 0.9 会显示。
    1
    ```

15. **don’t use underscores in Go names**

    ```
      变量名、函数名、参数名（入参， 返回参数）不应该使用下划线， 应该使用驼峰命名法。 当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
    1
    ```

16. **struct field Url should be URL**

    ```
      简写应该都大写。 'URL' 'ID' 'qID' 'isEOF'等。当 min-confidence 大于 0.8 时会忽略， 小于等于0.8会显示。
    1
    ```

17. **don’t use ALL_CAPS in Go names; use CamelCase**

    ```
      不要用全大写加下划线。 当 min-confidence 大于 0.8 时会忽略， 小于等于 0.8 会显示。
    1
    ```

18. **don’t use leading k in Go names**

    ```
      const 修饰的常量不能以 小写字母 'k' 接大写字母开头。 当 min-confidence 大于 0.8 时会忽略， 小于等于0.8会显示。
    1
    ```

19. **don’t use MixedCaps in package name**

    ```
      go 的包名不要用大小写混合，应该全小写。
    1
    ```

20. **should have a package comment, unless it’s in another file for this package**

    ```
      应该对包写注释。 当 min-confidence 大于 0.2 时会忽略， 小于等于 0.2 会显示。
    1
    ```

21. **package comment should be of the form "Package testdata …"**

    ```
      包的注释应该以 "Package packagename" 开头。
    1
    ```

22. **package comment should not have leading space**

    ```
      包注释最前面不应该有空格
    1
    ```

23. **package comment is detached; there should be no blank lines between it and the package statement**

    ```
      包注释和 'package ' 之间不应该有空行。 当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
    1
    ```

24. **should omit 2nd value from range; this loop is equivalent to `for x := range ...`**

    ```
      用range写for循环时，如果第二个参数不需要， 可以直接写成 `for x := range ...`
      `for x, _ = range ...` ===> `for x := range ...`
    12
    ```

25. **should omit values from range; this loop is equivalent to `for range ...`**

    ```
      如果 range 循环两个参数都不需要，可以写成 `for range ...`。
      `for _, _ = range ...` || `for _ = range ...` ===> `for range ...`
    ```

26. **receiver name should be a reflection of its identity; don’t use generic names such as “this” or "self"**

```sh
      结构体的方法， 引用结构体的名称应该要关联结构体。
      `func (this foo) f1()` ===> `func (f foo) f1()`
```

1.  **receiver name should not be an underscore, omit the name if it is unused**

    ```
      结构体的方法， 引用结构体的名称不应该使用 '_'。如果没有使用，删除它。
    ```

2.  **receiver name a should be consistent with previous receiver name b for bar**

    ```
      同一结构体的方法，引用结构体的名称应该一致。
    ```

3.  **exported method U.Other should have comment or be unexported**

    ```
      对外暴露的结构体对应的方法应该有注释。
    ```

4.  **type name will be used as donut.DonutMaker by other packages, and that stutters; consider calling this Maker**

    ```
      对外暴露的结构体或者变量，命名时不应该以包名作为前缀。例如你的包名叫 'dount', 要定义一个结构体，结构体的名称应该用 'Maker', 不应该使用 'DountMaker', 因为别的包导入的时候会用 'dount.Maker', 使用 'dount.DountMaker' 会有信息冗余。当 min-confidence 大于 0.8 时会忽略， 小于等于0.8会显示。
    ```

5.  **var rpcTimeoutMsec is of type \*time.Duration; don’t use unit-specific suffix “Msec”**

    ```
     如果变量的结果是 *time.Duration 类型，不要用 'Msec' 或者 'Secs' 结尾。当 min-confidence 大于 0.9 时会忽略， 小于等于0.9会显示。
    ```

6.  **exported func Exported returns unexported type foo.hidden, which can be annoying to use**

    ```
      非该结构体的要暴露的函数返回参数为该结构体。这样用起来会很麻烦。当 min-confidence 大于 0.8 时会忽略， 小于等于0.8会显示。
    ```
