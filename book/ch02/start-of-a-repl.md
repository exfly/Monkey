# REPL

`Monkey` 语言需要 `REPL`，它是 `Read Eval Print Loop` 的简称，这个在解释型语言中都存在，`Python` 中有 `REPL`，`Ruby` 中也有。有时候 `REPL` 也称为控制台，也叫做交互模式，概念都是一样的，`REPL` 读入输入，把它发送到解释器中执行，打印结果和输出，如此循环往复：`Read, Eval, Print, Loop`。

目前我们还不知道 `Eval` `Monkey` 中的代码，我们只知道 `Eval` 是接下来的过程：我们能够将 `Monkey` 源代码转换为 `token`。

下面是一个 `REPL`

```go
// repl/repl.go
package repl
import (
    "bufio"
    "fmt"
    "io"
    "monkey/lexer"
    "monkey/token"
)
const PROMPT = ">> "
func Start(in io.Reader, out io.Writer) {
    scanner := bufio.NewScanner(in)
    for {
        fmt.Printf(PROMPT)
        scanned := scanner.Scan()
        if !scanned {
            return
        }
        line := scanner.Text()
        l := lexer.New(line)
        for tok := l.NextToken(); tok.Type != token.EOF; tok = l.NextToken() {
            fmt.Printf("%+v\n", tok)
        }
    }
}
```

代码直截了当，从输入中读到新的一行，将得到的新的行传递到词法解析器中，最终输出所有得到的 `token` 直到遇到 `EOF`。

在 `main.go` 文件中，添加使用 `REPL` 的欢迎用语。

```go
// main.go
package main
import (
    "fmt"
    "os"
    "os/user"
    "monkey/repl"
)
func main() {
    user, err := user.Current()
    if err != nil {
        panic(err)
    }
    fmt.Printf("Hello %s! This is the Monkey programming language!\n", user.Username)
    fmt.Printf("Feel free to type in commands\n")
    repl.Start(os.Stdin, os.Stdout)
}
```

现在我们使用交互模式生成 `token`

```shell
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let add = fn(x, y) { x + y; };
{Type:let Literal:let}
{Type:IDENT Literal:add}
{Type:= Literal:=}
{Type:fn Literal:fn}
{Type:( Literal:(}
{Type:IDENT Literal:x}
{Type:, Literal:,}
{Type:IDENT Literal:y}
{Type:) Literal:)}
{Type:{ Literal:{}
{Type:IDENT Literal:x}
{Type:+ Literal:+}
{Type:IDENT Literal:y}
{Type:; Literal:;}
{Type:} Literal:}}
{Type:; Literal:;}
>>
```

现在可以解析这些 `token`。
