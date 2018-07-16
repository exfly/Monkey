# 定义 Token
首先要做的事在我们的词法解析器中定义我们的token，这些token将会被输出。开始以一些token的定义，然后在一点点增加它们。

Monkey语言的子集看上去是这样的：
```
let five = 5;
let ten = 10;
let add = fn(x, y) {
    x + y;
};
let result = add(five, ten);
```
首先将他们拆解开来，它们包含了那些类型的token?首先由数字，比如`5`和`10`，这是非常显然的。然后我们拥有变量名`x`,`y`,`add`和`result`，而且它们不是数字，仅仅是单词。但是也有些也不是变量名，比如`let`和`fn`，当然也有一些特殊的字符比如`(`，`)`，`{`，`}`，`=`，`,`，`;`，。

这些数字都是整数，我们将他们看作一种单独的类型。在我们的词法解析器或者语法解析器中不关心这个数字是5还是10，我们仅仅想知道这是一个数字。同样的道理比如变量名称。我们称它们为标识符，也将其视为同一类。现在其他的，看上去好像是标识符，但是并不是因为他们是语言的一部分，称之为关键字。我们不能将他们分组为同一类，因为当我们的解析器遇到它们的时候表现地都是不一样的。同样也是的，当我们遇到最后一个类别的时候，我们也将他们视为各自类别，因为在源码中，当遇到一个`(`和`)`的时候表现是完全不一样的。

让我们来定义我们的`token`数据结构，我们需要哪些字段？正如我们刚刚看到的，首先我们需要一个`type`树形，以便我们能够区分整数和右括号。然后还需要一个字段来保存其字面值，以便我们能够在将来能够重用它们，比如整数token它是5或者10不能丢失。

在新的`token`包中，我们定义了`token`结构体和`TokenType`类型：
```go
//token/token.go
package token
type TokenType string
type Token struct {
    Type TokenType
    Literal string
}
```
我们定义了`TokenType`为一个一个字符串，这个允许我们可以使用不同的值作为`TokenType`，它反过来可以允许我们区分不同的类型的token。使用字符串的好处是可以很容易的调试，而不需要借助其他辅助工具，我们只需要将字符串打印出来。当然，使用字符串可能导致使用整型和字节同样的性能，但是这本书使用字符串还算是完美。

正如我么你看到的，在Monkey语言中，可以使用的token类型不多。这也就意味着我们使用有限的`TokenType`类型，我们增加如下代码：
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
如上所示，有两种特特别的类型`ILLEGAL`和`EOF`，在例子中我们并没有看到，但是我们的确需要它们，`ILLEGAL`表明了token或者的字符我们事先不清楚，`EOF`是`end of file`，它表明了我们的解析器可以停止。

接下来我们开始我们的词法解析器。