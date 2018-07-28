# 词法解析器
在开始编写代码之前，让我们说明一下本小节的目标。我们将要编写自己的词法解析器，它将源代码作为输入，并且输出整个源代码的的token。我们将重新遍历整个输入，然后依次输出每个token。这不需要缓冲区来保存token，因为这里只有一个方法叫做`NextToken()`的方法，它将依次输入下一个token。

这意味着我们将通过源码初始化词法分析器，然后让其不断得对源码调用 `NextToken()` 函数，一个 token 接着一个 token，一个字符接着另外一个字符。我们将会视源码为字符串，这会让整个过程变得简单。而且在生产环境中，将文件名和行号加到 token 中是有意义的，这样能更好得追踪解析过程和分析错误。所以最好使用 `io.Reader` 和文件名初始化词法分析器。但是因为这会增加我们处理的复杂度，所以我们不会在这里处理这些，我们将从简单的地方开始，只使用字符串并忽略文件名和行号。

目标明确后，我们需要为词法分析器做什么已经很清晰了。我们创建一个新的包，并且添加一个可以持续运行的测试，以获得词法分析器工作状态的反馈。我们可以从最小的地方开始，然后随着词法分析器功能的完善再不断添加测试用例。



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

当然，测试会失败 -- 因为我们还没有添加任何代码:

```shell
$ go test ./lexer
# monkey/lexer
lexer/lexer_test.go: 27 undefined: New
FAIL monkey/lexer [build failed]
```

接下来让我们以一个返回 `*Lexer` 的函数 `New()` 开始：

```go
package lexer 

type Lexer struct{
    position     int    //current character position
	readPosition int    //next character position
	ch           rune   //current character
	characters   []rune //rune slice of input string
}

func New(input string) *Lexer {
    l := &Lexer{input:input}
    return l
}
```

Lexer 的大多数字段都能自解释。有可能产生一些困惑的是 `position` 和 `readPosition`。它们都被用于访问 `input` 中字符的下标。例如：`l.input[l.readPosition]`。这两个指针指向我们的输入字符串的原因，是我们需要进一步查看输入并查看当前字符，以查看接下来会发生什么。readPosition 永远指向 input 中的下一个字符，而position 则指向 input 中的与 ch 字节相关联的字符。

第一个辅助函数被称为 `readChar()` 可以让这几个字段更容易被理解：

```go
// lexer/lexer.go
func (l *Lexer) readChar() {
	if l.readPosition >= len(l.characters) {
		l.ch = rune(0)
	} else {
		l.ch = l.characters[l.readPosition]
	}
	l.position = l.readPosition
	l.readPosition += 1
}
```

readChar 函数的目的在于返回下一个字符给我们，以及更新我们在 input 字符串的位置。它第一件要做的事是检查我们是否读到了 input 的末尾，无法再读取任何字符。如果是的话会将 `l.ch` 置为 0，对于我们来说，这个 ASCII 码代表了 “NUL” 字符，意味着我们还没有读取任何内容。但是如果我们还没有达到输入的结尾，那么它会通过访问将 `l.ch` 设置为下一个字符，然后将 `l.position` 更新到刚刚使用的 `l.readPosition` ，并且`l.readPosition` 增加1。通过这种方式，`l.readPosition` 永远指向我们将要读取的下一个位置，而 `l.position` 则永远指向我们读过的最后一个位置。

在谈论 readChar 的时候，要说明的一点是，其只支持 ASCII 码而不是整个 Unicode 集合。为什么？因为这能够让我们保持事情的简单性，并且把精力放在解释器的要点上。为了支持 Unicode 和 UTF-8，我们需要将 `l.ch` 的类型从 byte 改为 rune, 因为它们可能包含多个字节。`l.input[l.readPosition]` 也无法再继续工作。接下来我们也将会看到，一些其他的方法和函数也需要做出一点变化。让 Monkey 完整得支持 Unicode（以及 emojis!）就做为读者的一个练习吧。

接下来我们对 `New()` 函数调用 readChar，如此一来，在调用 `NextToken()`之前，我们已经初始化好了 `l.position` 和 `l.readPosition`，  `*Lexer` 进入了工作状态。

```go
func New(input string) *Lexer {
    l := &Lexer{input:input}
    l.readChar()
    return l
}
```

现在我们的测试告诉我们调用 `New(input)` 不再有任何问题，但是 `NextToken()` 方法还是没有实现。接下来让我们添加第一个版本

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    // TODO: add more code here
}
```

这是 `NextToken()` 方法的基本结构。我们查看当前检查的字符，并根据字符串返回一个 token。在返回 token 之前需要更新我们的指针在 input 中的位置，所以当我们下次再次调用 `NextToken` 的时候，`l.ch` 已经是最新的值。一个叫做 `newToken` 的小函数帮助我们初始化那些 token。

运行测试，我们可以看到他们得到了通过：

```shell
$ go test ./lexer
ok monkey/lexer 0.007s
```

棒极了！接下来我们扩展测试用例	

```go
// lexer/lexer_test.go
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

值得注意的是，在这个测试中，input 已经发生了变化。它看起来像是 Monkey 语言的一个子集。它包含了所有我们已经成功转化为 token 的符号，但是一个新的事情导致我们的测试失败了：标识符、 关键字、数字。 



接下来使用标识符和关键字作为我们的开始。我们的 lexer 所需要做的就是识别当前的字符是否是一个字母，如果是，其就需要读取剩下的标识符和关键字，直到其遇到一个非标识符或者关键字，这样我们就可以使用正确 token 类型。第一步我们需要扩展我们的 switch 语句。



``` go
// lexer/lexer/go

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



我们为 switch 语句添加了一个默认的分支，因此，只要 `l.ch `不是识别出的字符之一，我们就可以检查标识符。我们同样可以添加类型为 `token.ILLEGAL` 的token 的生成。如果我们到此为止，我们真的不知道如何处理当前的字符并且将其声明为 `token.ILLEGAL`



辅助函数`isLetter`检查所给的参数是否是一个字母。这听起来很容易，但是要注意的是， `isLetter` 的改动会对我们的解析器解析出来语言的语言产生出乎意料的影响。正如你看到的那样，在我们例子中检查了 `ch == '_'` ，这意味着我们将 `_` 视为字母，并且允许其出现在标识符和关键字中。这表明我们可以使用类似 `foo_bar`做为变量名。其他编程语言甚至允许 ! 和 ？作为标志符。如果你也想要把它们作为标识符，这里就是你实现的地方。



`readIdentifier()` 的功能就像函数名所展现的那样，其读取一个标识符，并更新我们分析器的读取位置，直到遇到一个非字母的字符。



在 switch 语句的默认分支中，我们使用 `readIdentifier` 为我们当前的 token 设置字段 `Literal` 。但是 `Type`  字段呢？目前我们已经读取了像 `let`，`fn` 或者 `foobar`，除了语言关键字之外，我们还需要能够告诉用户定义的标识符。我们需要一个函数，它能够为我们拥有的 token 返回正确的 `TokenType`。那么除了 token 包，还有那个地方更适合添加这样一个函数呢？



```go
// token/token.go

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



`LookupIdent` 检查通过关键字表检查所给的标识符是否是关键字。如果是，其返回关键字的常量 `TokenType`。如果其不是，则返回一个为所有用户自定义标识符的 `token.IDENT`。



有了这个函数，我们就可以完成标识符和关键字的词法分析了：



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



我们使用 `return tok` 在这里提前结束是必要的，因为当调用 `readIdentifier()` 的时候，我们会重复调用 `readChar()` 并更新我们的 `readPosition` 和 `position` 字段到当前标识符的最后一个字符。所以我们不需要在 switch 语句后再次调用 `NextToken()`



接下来运行我们的测试，我们可以检查 `let` 目前是否是标识符。不过测试还是无法通过：

```shell
$ go test ./lexer

--- FAIL: TestNextToken (0.000s)
  lexer_test.go:70: tests[1] - tokentype wrong. expected="IDENT", got="ILLEGAL"
FAIL
FAIL monkey/lexer 0.008s
```



问题出在我们下一个我们希望得到的 token：一个为 "five" 的 `IDENT` token 。实际上我们得到的是一个 `ILLEGAL` token。为什么？原因在于 “let" 与 “five" 之间隔了空格。但是在 Monkey 里空格不重要，所以我们需要整个跳过它。



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



像这种辅助函数在很多分析器都能找到，有会被称为 `eatWhitespace`，有时则被称为 `consumeWhitespace`，或者其他完全不同的名字。哪些字符需要被跳过取决于被分析的语言。例如，有些语言在实现的时候会为换行符创建  token，如果它们不在 token 流中的正确位置，则将会抛出错误。为了让接下来的步骤简单些，我们暂时跳过对换行符的分析。



使用 `skipWhitespace()` 处理 `let five = 5;`时，数字 5 将会被跳过。这是正确的，因为到目前为止，该函数还不知道将数字转化为 token。接下来是添加该功能的时候了：

就像我们之前为标识符所做的一样，我们需要在 switch 语句的默认分支中添加更多的功能。



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



正如你所看到的，添加的代码与读取标识符和关键字的部分密切相关。`readNumber` 函数与 `readIdentifier` 函数一样，只不过使用 `isDigit` 代替了 `isLetter` 实现相应的功能。我们可以通过将字符识别函数作为参数传递来概括它，但是为了简单起见并且容易理解，我们并不选择这么做。



`isDigit` 函数和 `isLetter` 函数一样简单，它只返回传入的字节是否为 0 ~ 9 的拉丁字母。

添加该函数之后，我们的测试可以通过了：

```shell
$ go test ./lexer
ok	monkey/lexer 0.000s
```



我不知道你是否注意到，我们在 `readNumber` 函数中简化了很多东西。我们只读取了整数类型，浮点类型呢？或者是用 16 进制表示的数字？又或者是 8 进制？我们忽略了它们，并且只能说 Monkey 并不支持它们。当然，原因还在于本书的教育目标和有限的范围。



是时候打开香槟做庆祝了，我们成功得将测试用例中用到的 Monkey 语言小子集成功转换为 token。



有了这个胜利，我们很容易扩展词法分析器，通过它可以标记更多的 Monkey 源代码。
