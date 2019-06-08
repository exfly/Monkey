# 词法解析器

在开始写代码之前，我们先交代一下本小节的目标。我们将要编写自己的词法解析器，它将源代码作为输入，然后输出整个源代码的 `token`。我们遍历整个输入，然后输出每一个 `token`，这里我们不需要缓冲区保存 `token`，我们只有一个方法叫做 `NextToken()` 方法，它每次输出下一个 `token`。

这意味着我们通过源码初始化词法分析器，然后对源码调用 `NextToken()` 函数，一个 `token` 接着一个 `token`，一个字符接着一个字符。它将整个源码视为字符串，这个做能够简化处理的过程。但是在生产实际环境中，会将文件名和行号添加到 `token` 中，因为这样能更好地跟踪解析的过程和错误分析。最好不使用 `io.Reader` 和文件名来初始化词法分析器，它们会增加我们处理的复杂度，所以我们将从最简单的地方开始，只使用字符串并忽略文件名和行号。

目标已经相当名明确，我们的词法分析器能够做什么已经清楚了。我们创建一个新包，并且添加一个可以持续运行的测试，以获得词法分析器工作状态的反馈。我们可以从最小的地方开始，然后随着词法分析的功能完善再不断添加测试用例。

```go
// lexer/lexer_test.go
package lexer

import (
    "testing"
    "monkey/token"
)

func TestNextToken1(t *testing.T) {
    input := `=+(){},;`

    tests := []struct {
        expectedType    token.TokenType
        expectedLiteral string
    }{
        {token.ASSIGN, "="},
        {token.PLUS, "+"},
        {token.LPAREN, "("},
        {token.RPAREN, ")"},
        {token.LBRACE, "{"},
        {token.RBRACE, "}"},
        {token.COMMA, ","},
        {token.SEMICOLON, ";"},
        {token.EOF, ""},
    }
    l := New(input)
    for i, tt := range tests {
        tok := l.NextToken()
        if tok.Type != tt.expectedType {
            t.Fatalf("tests[%d] - tokentype wrong, expected=%q, got=%q", i, tt.expectedType, tok.Type)
        }
        if tok.Literal != tt.expectedLiteral {
            t.Fatalf("tests[%d] - Literal wrong, expected=%q, got=%q", i, tt.expectedLiteral, tok.Literal)
        }
    }
}
```

当然现在的测试肯定是不能通过的，因为我们还有添加任何代码。接下以一个返回 `*Lexer` 的函数 `New()` 开始词法解析器的旅途。

```go
package lexer

type Lexer struct{
    position     int    //current character position
    readPosition int    //next character position
    ch           byte   //current character
    characters   []byte //byte slice of input string
}

func New(input string) *Lexer {
    l := &Lexer{input:input}
    return l
}
```

`Lexer` 结构中的 `position` 和 `readPosition` 两个字段比较特殊，它们都是用作于访问 `input` 中字符的下标，比如 `l.input[l.readPosition]`。使用两个指针的原因是我们进一步查看下一步字符，以便接下里怎么处理。`readPosition` 永远指向 `input` 中下一个字符，而 `position` 指向 `input` 中和 `ch` 关联的的字符。

第一个辅助函数 `readChar()` 可以让接下来几个字段更加容易理解：

```go
// lexer/lexer.go
func (l *Lexer) readChar(){
    if l.readPosition >= len(l.characters) {
        l.ch = byte(0)
    } else {
        l.ch = l.characters[l.readPosition]
    }
    l.position = l.readPosition
    l.readPosition += 1
}
```

`readChar` 方法的功能是读取下一个字符，以及更新我们在 `input` 字符串的位置。首先它检查我们是否到达 `input` 的末尾，这样无法再继续读取任何支付，如果是的话将 `l.ch` 设置为 `0`，对于我们来讲这代表了 `NULL` 字符。如果我们还没有到达输入的结尾，我们将 `l.ch` 设置为下一个字符，然后将 `l.position` 更新到刚刚使用的 `l.readPosition`，并且将 `l.readPosition` 增加 `1`。通过这种方式 `l.readPosition` 永远指向我们将要读取的下一个位置，而 `l.position` 则永远指向我们读过的最后一个位置。

在讨论 `readChar` 的时候，更要说明的是只支持 `ASCII` 码而不是整个 `Unicode` 集合。为什么呢？主要是我想把事情搞得简单点，并且将主要精力放在解释器重点，为了支持 `Unicode` 和 `UTF-8`，我们需要将 `l.ch` 类型从 `byte` 改为 `rune`，因为它们可能包含多个字节，那么 `l.input[l.readPosition]` 也无法再继续工作，接下来也会看到其他的方法和函数也需要作出一些修改，那么 `Monkey` 完整支持 `Unicode` 以及 `Emoji` 作为读者的练习。

`New()` 函数会调用 `readChar` 函数，如此一来在调用 `NextToken()` 之前，我们就已经初始化好 `l.position` 和 `l.readPosition`，`*Lexer` 进入工作状态。

```go:n
func New(input string) *Lexer{
    l := &Lexer{input:input}
    l.readChar()
    return l
}
```

现在我们的测试中 `New(input)` 没有问题，但是 `NextToken()` 方法还没有实现，接下来我们添加第一个版本，

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token{
    //TODO: add more code here
}
```

这是 `NextToken` 方法的基本结构，我们查看当前检查的字符，并根据字符串返回一个 `token`。在返回 `token` 之前需要更新我们的指针在 `input` 中的位置，所以当我们下次调用 `NextToken` 的时候，`l.ch` 已经是最新值。在这 `newToken` 小函数可以帮助我们初始化这些 `token`。

现在我们可以通过刚刚的测试

```shell
$ go test ./lexer
ok monkey/lexer 0.007s
```

接下来我们拓展测试用例

```go
//lexer/lexer_test.go
func TestNextToken1(t *testing.T) {
    input := `=+(){},;`

    tests := []struct {
        expectedType    token.TokenType
        expectedLiteral string
    }{
        {token.ASSIGN, "="},
        {token.PLUS, "+"},
        {token.LPAREN, "("},
        {token.RPAREN, ")"},
        {token.LBRACE, "{"},
        {token.RBRACE, "}"},
        {token.COMMA, ","},
        {token.SEMICOLON, ";"},
        {token.EOF, ""},
    }
    l := New(input)
    for i, tt := range tests {
        tok := l.NextToken()
        if tok.Type != tt.expectedType {
            t.Fatalf("tests[%d] - tokentype wrong, expected=%q, got=%q", i, tt.expectedType, tok.Type)
        }
        if tok.Literal != tt.expectedLiteral {
            t.Fatalf("tests[%d] - Literal wrong, expected=%q, got=%q", i, tt.expectedLiteral, tok.Literal)
        }
    }
}
```

值得注意的是，在这个测试中，`input` 已经发生变化，它看上去像是 `Monkey` 语言的子集。它包含了所有已经转换为 `token` 的符号，但是新增加的内容导致我们的测试失败了，主要包含：标识符、关键字和数字。

我们以标识符和关键字作为我们解析的开始，`lexer` 所需的工作就是识别当前你的字符是否为字母，如果是就需要读取剩下的标识符和关键字，直到遇到一个非标识符或关键字。为了正确解析 `token` 类型，首先要做的事拓展我们的 `switch` 语句。

```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    var tok token.Token

    switch l.ch {
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
    l.readChar()
    return tok
}

func (l *Lexer) readIdentifier() string {
    position := l.position
    for isIdentifier(l.ch) {
        l.readChar()
    }
    return l.input[position:position]
}

func isLetter(ch byte) bool {
    return 'a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || ch == '_'
}
```

我们为 `switch` 语句增加了一个默认分支，因此只要 `l.ch` 是识别出的字符之一，我们可以检查是否为标识符。同样我们可以添加 `token.ILLEGAL` 类型。到目前为止，当我们不知道如何处理当前字符，一律声明为 `token.ILLEGAL`。

辅助函数 `isLetter` 检查字符是否为一个字母。听上去很简单，但是要注意的是这个函数对我们解析器的功能有显著的影响。正如这上面的代码中，我们检查了 `ch == '_'`，意味着我们将 `_` 设置为视为字母，并且允许出现在标识符和关键字中。这也就是说明了 `foo_bar` 也可以作为变量名，其他语言甚至允许 `!` 和 `?` 作为标识符，如果你想增加支持其他字符为标识符，对这个函数做出修改。

`readIdentifier()` 的功能正如名字所描述的，其读取一个标识符并更新词法分析器的读取位置，直到遇到一个非字母的字符。

在 `switch` 语句的默认分支中，使用 `readIndentifer` 读到的是当前 `token` 中的 `Literal` 字段，那么 `Type` 呢？ 目前我们已经读取了像 `let`, `fn` 或者 `foobar`，除了语言的关键字，我们还需要知道用户定义的标识符类型。因此我们需要知道一个函数来判断返回 `token` 的 `TokenType` 字段。

```go
var keywords = map[string]TokenType{
    "fn":     FUNCTION,
    "let":    LET
}

// LookupIdentifier used to determinate whether identifier is keyword nor not
func LookupIdentifier(identifier string) TokenType {
    if tok, ok := keywords[identifier]; ok {
        return tok
    }
    return IDENT
}
```

`LookupIdentifier` 函数检查标识符是否为给定的关键字，如果是，则返回关键字常量 `TokenType`；如果不是，则返回所有默认的标识符 `token.IDENT`。

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    switch l.ch {
    // [...]
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.Type = token.LookupIdent(tok.Literal)
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
}
```

这里使用 `return tok` 提前结束，因为当调用 `readIdentifier()` 的时候，我们会重复调用 `readChar()` 函数并更新我们的 `readPosition` 和 `position` 字段的值直到最后一个，所以不需要在 `switch` 语句后面调用 `NextToken()` 函数。

还有一点，在 `Monkey` 语言中不需要关注空格，所以需要跳过它们。

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    l.skipWhitespace()
    switch l.ch {
        // [...]
    }
}

func (l *Lexer) skipWhitespace() {
    for l.ch == ' ' || l.ch == '\t' || l.ch == '\n' || l.ch == '\r' {
        l.readChar()
    }
}
```

在很多词法分析器中都能找到这个辅助函数，有的称为 `eatWhitespace`，有的叫做 `consumeWhitespace` 或者其他名字。哪些字符需要跳过取决于被分析的语言。例如在有些语言实现中为为每个换行符创建 `token`，如果它们不在 `token` 流中正确的位置，就会抛出错误。为了接下来步骤简单些，我们暂时跳过换行符的分析。

和之前一样，我们还需要为 `switch` 语句默认的分支中增加更多的功能。

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token() {
    var tok token.Token
    l.skipWhitespace()
    switch l.ch {
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.TYpe = token.LookupIdent(tok.Literal)
            return ok
        } else if isDigit(l.ch) {
            tok.Type = token.INT
            tok.Literal = l.readNumber()
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
}

func (l *Lexer) readNumber() string {
    position := l.position
    for isDigit(l.ch) {
        l.readChar()
    }
    return l.input[position:l.position]
}

func isDigit(ch byte) bool {
    return '0' <= ch && ch <= '9'
}
```

这个增加了 `readNumber` 函数，它和 `readIdentifer` 函数一样，只不过使用 `isDigit` 代替 `isLetter` 实现相应的功能。为了表示明确，我们为每一个功能单独设置函数。`isDigit` 函数和 `isLetter` 函数一样，它只返回传入的字节是否为 `0~9` 阿拉伯数字。

添加该函数后，我们可以通过测试了：

```shell
$ go test ./lexer
ok monkey/lexer 0.000s
```

不知你是否注意到，我们在 `readNumber` 函数中简化了很多，比如只读了整数类型，那么浮点型呢？亦或者 `16/8` 进制数字呢？在这里 `Monkey` 语言都忽略了它们，如果读者有兴趣可以拓展它们。

上面是词法分析器的基础，我们可以很容易拓展它们来标记更多的 `Monkey` 源代码。

