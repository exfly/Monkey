# 定义 `Token`

首先要做的事在我们的词法分析器中定义 `token`，这些 `token` 在后面会被输出，开始我们可以定义一些 `token`，到后面一点点增加它们。

`Monkey` 语言子集看上去是这样的：

```monkey
let five = 5;
let tne = 10;
let add = fn(x, y) {
    x + y;
};
let result = add(five, ten);
```

首先我们将它们拆解开来，看看它们究竟包含了哪些类型的 `token`？首先是数字，比如 `5` 和 `10`；接下来是变量 `x`， `y`，`add` 和 `result`。而且它们不是数字，仅仅是单词；有些不是变量名，比如 `let` 和 `fn`；最后还有一些特殊的符号 `(`，`)`，`{`，`}`，`=`，`,`，`;`。

对于数字，我们将它们看成单独的类型，在词法解析器中不关心这个数字是 `5` 还是 `10`，只需要知道它是一个数字就行了；对于变量名，也就是标识符也是同样的道理；还有一种看上去像是标识符，但是是语言的一部分，称之为关键字。我们不能讲它们分为同一类，因为当我们的解释器遇到不同的关键字处理是不一样的；最后一种也是视为不同的类别，因为在源码中，当遇到一个 `(` 和 `)` 的时候表现是不一样的。

现在定义 `token` 数据结构，我们需要哪些字段呢？正如我们刚刚看到的，首先需要一个 `token` 类型，以便我们区分整数和右括号；还要有一个字段来保存其字面值，以便我们在将来使用它们，比如整数 `token` 中的 `5` 和 `10` 的信息不能丢。

在新的 `token` 包中，我们定义了 `token` 结构体和 `TokenType` 类型。

```go
//token/token.go
package token
type TokenType string
type Token struct {
    Type TokenType
    Literal string
}
```

`TokenType` 为字符串类型，允许我们使用不同的值作为 `TokenType`，反过来方便我们区分不同类型的 `token`。还有一方面试使用字符串可以很方面我们调试，而不用借助其他辅助工具，我们只需要将字符串打印出来。当然使用字符串可能有性能上的损失，但是还是可以接受。

接下来为我们的 `Monkey` 语言增加有限的 `TokenType` 类型。

```go
//token/token.go
const (
    ILLEGAL   = "ILLEGAL"
    EOF       = "EOF"
    // Identifiers + literals
    IDENT     = "IDENT" //add, foobar, x, y, ...
    INT       = "INT" // 1343456
    // Operator
    ASSIGN    = "="
    PLUS      = "+"
    //Delimiter
    COMMA     = ","
    SEMICOLON = ";"

    GT        = ">"
    LPAREN    = "("
    RPAREN    = ")"
    LBRACE    = "{"
    RBRACE    = "}"
    // Keywords
    FUNCTION  = "FUNCTION"
    LET       = "LET"
)
```

有两个特殊的类型 `ILLEGAL` 和 `EOF`，这个在例子中并没有看到，但是我们的确需要它们，`ILLEGAL` 表明 `token` 表示的字符串我们事先不清楚，而 `EOF` 是 `end of file` 的意思，说明我们的解析器可以停止工作了。

