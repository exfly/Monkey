# 拓展词法解析器

为了后序编写解析器的需要，我们需要拓展我们的词法解析器以便识别出 `Monkey` 语言中更多的 `token`。所以本小节我们将支持 `==`，`!`，`!=`，`/`，`*`，`<`，`>` 以及关键字 `true`， `false`，`if`，`else` 和 `return`。

需要新增的 `token` 可以分为下面三种

- 单个字符的 `token`，比如 `-`；
- 两个字符的 `token`，比如 `==`；
- 关键词，比如 `return`。

我们已经知道如何处理单个字符和关键词 `token`，所以我们首先要处理这些，然后拓展的解析器来支持两个字符的 `token`。

增加支持 `-`， `/`，`*`，`<` 和 `>` 非常普通， 首先要做的是修改之前 `lexer/lexer_test.go` 中的测试用例，以便包含这些字符。

```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
`
// [...]
}
```

看上去像是 `Monkey` 语言的代码片段，但是其中有些行是没有意义的，比如 `!-/*5`。词法解析器不关心这些代码是否有意义，这些下一个阶段的工作，词法解析仅仅需要转换为 `token` 就可以了，所以我们的测试用例尽量包含所有的 `token`，而且包含一些错误、边界用例、换行处理和多个数字处理等等。

运行测试得到一些未定义的错误，因为测试包含了一些未定义的 `tokenTypes`。为了修正它们，我们需要在 `token/token.go` 中增加更多的常量：

```go
// token/token.go
const (
// [...]
// Operators
    ASSIGN = "="
    PLUS  = "+"
    MINUS = "-"
    BANG = "!"
    ASTERISK = "*"
    SLASH = "/"

    LT = "<"
    GT = ">"
//[...]
)
```

目前测试还是失败的，因为我们并没有返回期望的 `TokenTypes`。所以我们拓展我们的 `switch` 语句中 `NextToken()` 方法：

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
// [...]
    switch l.ch {
    case '=':
        tok = newToken(token.ASSIGN, l.ch)
    case '+':
        tok = newToken(token.PLUS, l.ch)
    case '-':
        tok = newToken(token.MINUS, l.ch)
    case '!':
        tok = newToken(token.BANG, l.ch)
    case '/':
        tok = newToken(token.SLASH, l.ch)
    case '*':
        tok = newToken(token.ASTERISK, l.ch)
    case '<':
        tok = newToken(token.LT, l.ch)
    case '>':
        tok = newToken(token.GT, l.ch)
    case ';':
        tok =  newToken(token.SEMICOLON, l.ch)
    case ',':
        tok = newToken(token.COMMA, l.ch)
// [...]
}
```

新增的 `switch` 语句中的 `case` 条件包含了 `token/token.go` 中定义的常量，现在运行测试全部通过。

```shell
$ go test ./lexer
ok monkey/lexer 0.007s
```

现在单个字符的 `token` 已经成功解析，下一步我们增加关键词 `true`，`false`，`if`，`else` 和 `return` 的解析。

再一次我们拓展测试以便包含这些新的关键词，在这里我们的希望在 `TestNextToken` 中 `input` 是这样的：

```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
    input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
if (5 < 10) { return true;
} else {
return false;
}`
// [...]
}
```

这个测试甚至不能编译，因为测试中期待的类型并没有定义，为了修正它，不仅仅需要增加新的类型，还需要在 `LookupIdent` 查找表增加。

```go
const (
//[...]
    FUNCTION  = "FUNCTION"
    LET       = "LET"
    TRUE      = "TRUE"
    FALSE     = "FALSE"
    IF        = "IF"
    ELSE      = "ELSE"
    RETURN    = "RETURN"
)

// reversed keywords
var keywords = map[string]TokenType{
    "fn":     FUNCTION,
    "let":    LET,
    "true":   TRUE,
    "false":  FALSE,
    "if":     IF,
    "else":   ELSE,
    "return": RETURN,
}
```

现在测试通过

```shell
$ go test ./lexer
ok monkey/lexer 0.007s
```

现在我们开始解析两个字符的 `token`，在 `Monkey` 中增加 `==` 和 `!=` 两种 `token`。

你可能会想：为什么我们不通过仅仅增加 `switch` 语句中的 `case` 条件来达到目的呢？我们现在重用已有的 `case` 分支 `=` 和 `!`，并拓展它们。首先我们现在测试中增加测试用例，看它们是否返回 `=` 和 `==`。

```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
if (5 < 10) { return true;
} else {
return false;
}
10 == 10; 10 != 9; `
// [...]
}
```

在进入 `NextToken` 方法之前，我们需要增加一个辅助方法 `peekChar()`。

```go
// lexer/lexer.go
func (l *Lexer) peekChar() byte {
    if l.readPosition >= len(l.input) {
        return byte(0)
    } else {
        return l.input[l.readPosition]
    }
}
```

除了不增加 `l.position` 和 `l.readPosition`之外，`peekChar` 和 `readChar` 方法非常像，它仅仅是像前查看一个字符而不是移动它。大部分词法解析器都有类似和解析器都会有类似 `peek` 方法，它仅返回下一个字符，难点在于在源码中需要向前看多少字符才有意义。

现在增加 `TokenType` 常量。

```go
// token/token.go
const (
//[...]
    EQ = "=="
    NOT_EQ = "!="
)
```

即使有了定义的 `token.EQ` 和 `token.NOT_EQ`，我们测试仍然没有通过。当词法解析器遇到 `==` 的时候，它创建两个 `token.ASSIGN` 而不是一个 `token.EQ`。解决方法就是在 `=` 和  `!` 分支中使用刚刚 `peekChar()` 方法，如果下一个字符为 `=`，则返回 `token.EQ` 和 `token.NOT_EQ`。

```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    // [...]
    switch l.ch {
    case '=':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            tok = token.Token{Type: token.EQ, Literal: string(ch) + string(l.ch)}
        } else {
            tok = newToken(token.ASSIGN, l.ch)
        }
// [...]
    case '!':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            tok = token.Token{Type: token.NOT_EQ, Literal: string(ch) + string(l.ch)}
        } else {
            tok = newToken(token.BANG, l.ch)
        }
// [...]
}
```

注意我们在调用 `l.readChar()` 之前，我们将 `l.ch` 保存到局部变量中。这种方式不会丢掉当前的字符而且可以很安全的前进字符，词法解析器也需要更新 `l.position` 和 `l.readPosition`。如果我们需要支持更多两个字符的 `token`，我们需要抽象出方法 `makeTwoCharToken` 来进行分析。

现在测试通过过了

```shell
$ go test ./lexer
ok monkey/lexer 0.006s
```

现在我们的词法解析器能够生成拓展后的 `token` 集，我们要开始准备编写解析器，但是之前，还有一件事要做。
