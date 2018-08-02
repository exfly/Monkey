为了在后面编写我们的的解析器的时候的需要，我们需要拓展我们的的词法分析器以便它能够识别出Monkey语言中更多的`token`。所以在本小节中我们将增加支持的`==`, `!`,`!=`,`-`,`/`,`*`,`<`,`>`以及关键词`true`,`false`,`if`,`else`和`return`。

需要新增加的`token`可以分为下面三种类别：单个字符的`token`(如`-`)，两个字符的`token`（如`==`）和关键词（如`return`）。我们已经知道如何处理单个字符和关键词`token`，所以我们首先要只是这些，然后在拓展我们词法分析器以便支持两个字符的`token`。

增加支持`-`,`/`,`*`, `<`和`>`非常普通，首先要做的是修改我们`lext/lex_test.go`中的测试用例，以便包含这些字符。正如我们之前做的，本章中的的的的所有测试代码修改是通过修改`tests`表，接下来将不会再次重复声明。
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
注意尽管输入看上像是Monkey语言的代码片段，但是一些行是没有意义上的。比如`!-/*5`,词法分析器并不会告诉我们这些代码是否有意义，这是下一个阶段所做的工作。词法分析器仅仅是将他们转换为`token`，所以我们的测试用例尽量包含所有的`token`，而且尽量包含一些错误，边界用例，换行的处理，或者多个数字的处理等等。

运行测试将会得到一些未定义的错误，因为测试包含了一些未定义的`TokenTypes`。为了修正它们，我们需要在`token/token.go`中增加下面的常量
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
有了新的常量，测试仍然是失败的，因为我们并没有返回期望的`TokenTypes`.
```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
    lexer_test.go:84: tests[36] - tokentype wrong. expected="!", got="ILLEGAL" FAIL
FAIL monkey/lexer 0.007s
```
为了让测试通过，需要我们拓展我们的`switch`语句中的`NextToken()`方法：
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
新增加的switch语句中的`case`条件记录了在`token/token.go`中常量的结构，简单的改变让测试通过了：
```
$ go test ./lexer
ok monkey/lexer 0.007s
```
现在新的单个字符的`token`已经成功添加了，下一步我们增加关键词的`true`,`false`,`if`，`else`和`return`词法分析。

再一次我们拓展测试以便包含这些新的关键词，在这里我们希望在`TestNextToken`中的`input`是这样的：
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
这个测试甚至不能编译，为测试中期待的类型并没有定义，为了修正它，意味着我们仅仅需要增加新的。增加`LookupIdent`来查找表：
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
仅仅修稿了编译错误就能够让测试通过了：
```
$ go test ./lexer
ok monkey/lexer 0.007s
```
现在词法分析器认出了新的关键词。

在我们进入下一章节开始解析器的时候，我们仍然需要拓展我们的词法解析器，以便它能够认出由两个字符组成的`token`，我们想增加的`token`是`==`和`!=`。

你可能会想：为什么我们不通过仅仅增加`switch`语句中的`case`条件来达到目的呢？因为这个当前的字符`l.ch`作为表达式与之相违背，我们不将当个字符与`==`进行比较。

我们能够做的是重用现在的case分支`=`和`!`，并拓展它们。首先要做的是在测试中增加用例，看它们能否返回`=`和`==`。拓展之后的测试代码如下：
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
在我们进入`NextToken`方法之前，我们需要新的帮助方法`peekChar()`
```go
// lexer/lexer.go
func (l *Lexer) peekChar() byte {
    if l.readPosition >= len(l.input) {
        return 0 
    } else {
        return l.input[l.readPosition] 
    }
}
```
`peekChar`和`readChar()`方法非常像，除了它不增加`l.position`和`l.readPosition`。我们仅仅是向前看一个字符而不是移动它，我们已经知道调用`readChar()`将会返回什么。大部分的词法解析器和解析器都有类似`peek`函数来向前看，它仅仅返回下一个字符。难度在于不同的源代码中要向前看多少才有意义。

当`peekChar()`增加后，新的测试输入没法编译，当然因为使用了尚未定义的测试，再次修改也非常容易
```go
// token/token.go
const (
//[...]
    EQ = "=="
    NOT_EQ = "!="
)
```
有了定义的`token.EQ`和`token.NOT_EQ`，我们的测试返回了失败的消息：
```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
    lexer_test.go:118: tests[66] - tokentype wrong. expected="==", got="=" FAIL
FAIL monkey/lexer 0.007s
```
当词法解析器遇到`==`的时候，它创建了两个`token.ASSIGN`而不是一个`token.EQ`。解决方案是我们使用新的`peekChar()`方法，在`=`和`!`分支中使用`peek`的方法。如果下一个字符`=`我们将穿件`token.EQ`或者`token.NOT_EQ`。
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
注意到在调用`l.readChar()`之前，我们将`l.ch`保存到局部变量。这种方法不会丢掉挡墙的字符而且很安全的前进词法分析器，词法分析器也更新的`l.position`和`l.readPosition`。如果我们要支持更多的两个字符的`token`，我们需要抽象出方法`makeTwoCharToken`来进行分析，因为两个分支非常像。既然在在这里只有`==`和`!=`两字符`token`，让我们就丢在这里，看看能够正确工作：
```
$ go test ./lexer
ok monkey/lexer 0.006s
```
是的，测试通过了。现在词法分析器能够生成拓展后的`token`集，我们要开始准备编写解析器了。但是在做之前，我们还有一件事要做。
