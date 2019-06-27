# RPPL

到目前为止，我们的 `REPL` 应该叫做 `RLPL(Reading-Lex-Print-Loop)`，因为我们还不知道如何执行代码，也就是还没有完成 `evaluate` 去代替 `lex` 这个步骤。目前用 `parse` 代替 `lex` 来构建 `RPPL`。

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

现在拓展我们的解析的循环部分，对输入的每一行进行解析。输出的结果是解析出来的 `*ast.Program` 使用 `String()` 方法来打印输出，它递归调用每一句中 `String` 方法，就可以在命令行中使用解析器。

```shell
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x = 1 * 2 * 3 * 4 * 5
let x = ((((1 * 2) * 3) * 4) * 5);
>> x * y / 2 + 3 * 8 - 123
((((x * y) / 2) + (3 * 8)) - 123)
>> true == false
(true == false)
```

除了调用 `String` 方法，可以调用任何抽象语法树代表字符串类型，也可以增加 `PrettyPrint` 方法输出抽象语法树的节点和内部孩子节点。

但是我们 `RPPL` 还有一个巨大缺陷，当遇到错误的时候：

```shell
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3;
expected next token to be =, got INT instead
```

这不是好的错误消息，虽然能够完成工作，但是不够友好。为了让 `Monkey` 语言值得更好的体验，提供 `printParserError` 方法，增加我们的用户体验：

```go
// repl/repl.go
func printParserErrors(out io.Writer, errors []string) { 
    io.WriteString(out, "Woops! We ran into some monkey business here!\n")
    io.WriteString(out, " parser errors:\n")
        for _, msg := range errors { io.WriteString(out, "\t"+msg+"\n")
    }
}
```

现在我们能够得到的错误信息如下：

```shell
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3
Woops! We ran into some monkey business here! parser errors:
expected next token to be =, got INT instead
>>
```

