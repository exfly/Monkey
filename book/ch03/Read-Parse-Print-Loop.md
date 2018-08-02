到目前为止，我们的REPL应该叫做`RLPL`(Reading-Lex-Print-Loop)。我们还不知道如何去执行代码，如何用`evaluate`去代替`lex`仍然还没有解决问题，但是现在我们能够确定的是解析。现在是时候用`parse`代替`lex`来构建`RPPL`。
```go
// repl/repl.go
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
        p := parser.New(l)
        program := p.ParseProgram() 
        if len(p.Errors()) != 0 {
            printParserErrors(out, p.Errors())
            continue
        }
        io.WriteString(out, program.String())
        io.WriteString(out, "\n") }
    }
    func printParserErrors(out io.Writer, errors []string) { 
        for _, msg := range errors {
            io.WriteString(out, "\t"+msg+"\n") 
        }
    }
}
```
现在我们拓展我们解析的循环部分，对输入到`REPL`中的每一行进行解析。输出的结果是解析出来的`*ast.Program`，然后调用`String()`方法来打印输出，它递归地调用每一语句中的`String`方法。现在我们可以在命令行中使用解析器。
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x = 1 * 2 * 3 * 4 * 5
let x = ((((1 * 2) * 3) * 4) * 5);
>> x * y / 2 + 3 * 8 - 123
((((x * y) / 2) + (3 * 8)) - 123)
>> true == false
(true == false)
>>
```
棒极了，除了调用`String`方法，我们可以调用任何抽象语法树代表字符串类型。我们可以增加`PrettyPrint`方法来输出抽象语法树的节点和内部孩子节点。

但是我们的`RPPL`还有一个巨大的缺陷：当遇到的错误的时候：
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3;
expected next token to be =, got INT instead
>>
```
这不是一个很好的错误消息，我的意思它完成了该完成的工作，但是不够友好。我们的Monkey语言值得更好的方式。下面有很好的`printParserError`函数，增加我们的的用户体验：
```go
// repl/repl.go
func printParserErrors(out io.Writer, errors []string) { 
    io.WriteString(out, "Woops! We ran into some monkey business here!\n") 
    io.WriteString(out, " parser errors:\n")
        for _, msg := range errors { io.WriteString(out, "\t"+msg+"\n")
    } 
}
```
现在好多了，当我们遇到任何解析错误，我们得到详细的错误信息
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3
Woops! We ran into some monkey business here! parser errors:
expected next token to be =, got INT instead
>>
```
