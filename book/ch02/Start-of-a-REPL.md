Monkey语言需要`REPL`，它是`Read Eval Print Loop`的简写，你可能知道在解释型语言中都存在。Python有`REPL`, Ruby也有`REPL`, 每一个JavaScript运行时也有`REPL`，大部分`Lisp`也拥有一个。有时`REPL`也称作控制台，有时也叫做交互模式。概念都是一样的，`REPL`读入输入，把它发送到解释器中执行，打印结果和输出。如此循环往复：Read, Eval, Print, Loop。

目前我们还不知道`Eval`Monkey中的代码，我们只知道`Eval`下面的一个过程：我们能够将Monkey源代码转换为`token`。但是我们知道如何去读和输出一些东西，我还认为循环不是个大问题。

下面是一个`REPL`,它能够将Monkey源码转换为`token`，并且打印出来。我们将会拓展它来增加执行的部分：
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
代码相当直接了：从输入源读取知道遇到新的一行，将得到的新的一行传递到词法解析器中，最终输出所有的`token`知道遇到`EOF`。

在`main.go`文件中，我们添加一些使用`REPL`的欢迎用语。
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
现在我们一个使用交互模式生成`token`
```
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
完美，现在开始解析这些`token`。