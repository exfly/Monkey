**解析LET语句**
在Monkey语言中，变量绑定语句的形式如下：
```
let x = 5;
let y = 10;
let foobar = add(5, 5);
let barfoo = 5 * 5 / 10 + 18 - add(5, 5) + multiply(124);
let anotherName = barfoo;
```
上面这些语句叫做`let`语句，将一个值绑定到一个变量上，`let x = 5`语句将值`5`绑定到变量名`x`上，本小节的我们的工作是正确地解析`let`语句。从现在开始我们将跳过解析表达式过程，它是通过给定的变量获得相应的值得过程。在我们知道如何解析表达式之后再来关注这些。

什么叫做正确的解析`let`语句？它意味着解析能够正确的得到一颗抽象语法树，它能够正确地表示原先`let`语句的全部信息。听上去很有可行，但是我们还没有任何抽象语法树，或者说我们还不知道着东西看上去怎么样？所以我们首先的任务是近距离看看Monkey语言，看看它结构如何。我们可以定义抽象语法树的必要部分，以便它能正确地表示`let`语句。

下面都是Monkey中合法的语句
```
let x = 10;
let y = 15;
let add = fn(a, b) {
    return a + b;
}
```
Monkey中都是一系列语句，在上面的例子中有三个语句，三个变量绑定。`let`语句有如下的形式：
```
let <identifier> = <expression>;
```
一个`let`语句有两个可变的部分组成：一个标识符和一个表达式。在上面例子中，`x`，`y`和`add`都是标识符，`10`,`5`和函数都是表达式。

在我们继续之前，还要进一步说明一下语句和表达式之间的不同是有必要的。表达式能够生成值，然后语句则不会。 `let x =5;`并不生成值，然而`5`能够（它生成5）。`return 5`是语句不会生成值，但是`add(5,5)`却能够生成值。区别就是在于:
表达式生成值，而语句则不会。当然也会做一些改变，这些取决于谁在问你，但是这样已经很好。

语句和表达式的正确的区分：是否生成值，取决于编程语言。在一些编程语言中函数`fn(x,y) {return x+y;}`是表达式，它可以被用在任何表达式可以使用的地方。在其他的编程语言中，字面函数只能作为编程语言函数声明的一部分。一些编程语言拥有`if`表达式，它的条件语句是表达式并且能够生成值。这些都取决于编程语言设计者怎么考虑的。正如你看到的，在Monkey语言中，大部分是表达式，包含字面函数。

回到我们的抽象语法树，看着上述的例子，我们知道我们需要定义两种不同类型的节点：表达式和语句，让我们看看抽象语法树如何开始：

```go
// ast/ast.go
package ast
type Node interface {
    TokenLiteral() string
}
type Statement interface {
    Node
    statementNode()
}
type Expression interface {
    Node
    expressionNode()
}
```
在这里我们定义了三个接口，分别为`Node`,`Statement`和`Expression`。我们抽象语法树中每一个节点必须实现`Node`接口，也就意味着它必须要提供`TokenLiteral`方法，它返回该节点关联的`token`的字面值。`TokenLiteral`将会被用在调试和测试中。抽象语法树将全部由`Node`构成，它们互相连接在一，毕竟这是一棵树。其中的一些节点实现了`Statement`接口，另外一些实现了`Expression`接口。这些接口各自包含了哑方法`statementNode`和`expressionNode`。它们不是严格要求的，但是它们能够帮助我们指导我们的Go编译器能够抛出可能的异常，当我们在应该使用`Expression`地方使用了`Statement`，相反也同理。

接下来就是我们`Node`的第一个实现

```go
//ast/ast.go
type Program struct {
    Statements []Statement
}
func (p *Program) TokenLiteral() string {
    if len(p.Statements) > 0 {
        return p.Statement[0].TokenLiteral()
    }else{
        return ""
    }
}
```
`Program`节点是每一个抽象语法树的根节点，每一个合法的Monkey程序都是一系列语句，这些语句被保存在`Program.Statement`中，它是一系列实现`Statement`接口的节点组成。

有了这些AST基础模块定义，让我们想想什么样的节点能够表示`let x = 5;`，会有那些字段？定义一个变量的名称，同样我们也需要一个字段来表示等号右边的表达式，它可以指向任何表达式。它不只是指向字面值（在这个例子中是5），因为任何等号右边表达式都是相同的，比如：`let x = 5 * 5;`和`let y = add(2, 2) * 5 / 10;` 都是有效的。为了跟踪记录抽象语法树中的每一个节点，我们需要实现`TokenLiteral()`方法。它有三个字段构成：一个是标识符，一个是可以生成值的表达式，最后是token。

```go
// ast/ast.go
import "monkey/token"
// [...]
type LetStatement struct {
    Token token.Token
    Name *Identifier
    Value Expression
}
func (ls *LetStatement) statementNode() {}
func (ls *LetStatement) TokenLiteral() string { return ls.Token.Literal}

type Identifier struct {
    Token token.Token // the token.IDENT token
    Value string
}
func (i *Identifier) expressionNode() {}
func (i *Identifier) TokenLiteral() string { return i.Token.Literal }
```
`LetStatement`拥有以下字段：`Name`用来保存绑定的标识符；`Value`用来保存表达式生成的值。`statementNode`和`TokenLiteral`方法用来满足`Statement`和`Node`接口。

为了保存标识符绑定值，`let x= 5;`中的`x`。我们需要定义`Identifier`结构，它实现了`Expression`接口，但是在let语句中它不生成值，为什么我们定义一个`Expression`接口，主要是简化问题。标识符在Monkey语言其他部分中会生成值，比如`let x = valueProducingIdentifier;`中，为了让数据类型数量变少，我们将使用`Identifier`来表示被绑定的值，以便后来重用它们，所以标识符实现了expression接口。

`Program`，`LetStatement`和`Identifier`构成了Monkey语言的源代码一部分：
```go
let x = 5;
```
它可以用抽象语法树表示如下：
![](../figures/ast1.png)
现在我们知道它应该长什么样，下一个任务就是构建一棵抽象语法树，首先我们的解析器是这样的：

```go
//parser/parser.go
package parser
import (
    "monkey/ast"
    "monkey/lexer"
    "monkey/token"
)
type Parser struct {
    l *lexer.Lexer
    curToken token.Token
    peekToken token.Token
}
func New(l *lexer.Lexer) *Parser {
    p := &Parser{l: l}
    //Read two tokens, so curToken and peekToken are both set
    p.nextToken()
    p.nextToken()
    return p
}
func (p *Parser) nextToken(){
    p.curToken = p.peekToken
    p.peekToken= p.l.NextToken()
}
func (p *Parser) ParserProgram() *ast.Program{
    return nil
}
```
`Parser`拥有三个字段，分别为`l`，`curToken`和`peekToken`。其中`l`指向词法分析器的一个实例，通过它可以不听的调用`NextToken`方法来获取输入的下一个`token`。`curToken`和`peekToken`就像我们词法解析器拥有的两个指针`position`和`peekPosition`，但是它不是指向输入的字符，而是指向当前的`token`和下一个`token`，它们是同样重要的：我们需要查看当前输入的`token`，通过检查当前的`token`，来确定下一步需要做什么；同样也需要`peekToken`如果`curToken`不能做出相应的决定。比如考虑但单行语句`5;`，当前的`curToken`是`token.INT`，我们需要`peekToken`来决定我们是否在一行代码的末尾还是仅仅是算术表达式的开始。

`New`函数非常明了，`nextToken`是一个小的帮助方法，它用来同时前进`curToken`和`peekToken`指针。现在目前`ParseProgram`是空的。

在我们编写测试和填充`ParseProgram`方法之前，我先向你展示一下递归下降分析背后的基础思想和结构。它可以帮助我们很好的理解后面的分析器。接下来主要部分是解析器的伪代码。仔细阅读它并且尝试去理解`parseProgram`函数里发生了什么。
```js
function parseProgram() { 
    program = newProgramASTNode()
    advanceTokens()
    for (currentToken() != EOF_TOKEN) {
        statement = null
        if (currentToken() == LET_TOKEN) { 
            statement = parseLetStatement()
        } else if (currentToken() == RETURN_TOKEN) { 
            statement = parseReturnStatement()
        } else if (currentToken() == IF_TOKEN) { 
            statement = parseIfStatement()
        }
        if (statement != null) { 
            program.Statements.push(statement)
        }
        advanceTokens() 
    }
    return program
}
function parseLetStatement() { 
    advanceTokens()
    identifier = parseIdentifier()
    advanceTokens()
    if currentToken() != EQUAL_TOKEN { 
        parseError("no equal sign!") 
        return null
    }
    advanceTokens()
    value = parseExpression()
    variableStatement = newVariableStatementASTNode() 
    variableStatement.identifier = identifier 
    variableStatement.value = value
    return variableStatement
function parseIdentifier() { 
    identifier = newIdentifierASTNode() 
    identifier.token = currentToken() 
    return identifier
}
function parseExpression() {
    if (currentToken() == INTEGER_TOKEN) {
        if (nextToken() == PLUS_TOKEN) { 
            return parseOperatorExpression()
        } else if (nextToken() == SEMICOLON_TOKEN) { 
            return parseIntegerLiteral()
        }
    } else if (currentToken() == LEFT_PAREN) {
        return parseGroupedExpression() 
    }
        // [...]
}
function parseOperatorExpression() {
    operatorExpression = newOperatorExpression()
    operatorExpression.left = parseIntegerLiteral() 
    operatorExpression.operator = currentToken() 
    operatorExpression.right = parseExpression()
    return operatorExpression() 
}
// [...]
```
虽然上面的伪代码中有很多省略，但是递归下降解析的基础思想是有的。入口是`parseProgram`函数，它构造了抽象语法树的根节点，然后通过调用其他函数构建孩子节点，语句。这些抽象语法树是基于当前`token`，其他函数也是相互调用，递归执行。

最递归调用的部分在`parseExpression`中，但是我们已经知道为了解析表达式`5 + 5`，我们需要首先解析`5 +`然后再一次调用`parserExpression()`来解析剩下来的部分，因为在`+`后面可能是其他操作符表达式，比如`5+5*10`。我们将会接下来仔细查看解析该表达式的细节，因为他是解析中最复杂的部分同样也是最漂亮的部分，也就是`Pratt`解析。

但是到现在，我们已经知道解析器将要做什么。它不停地读取`tokens`然后检查当前`token`来决定接下来需要做什么：是调用其他的解析函数呢还是抛出一个异常。每一个函数都有各自的任务，可能创建抽象语法树的节点。所以在`parseProgram()`中的主循环可以不停前进`token`来决定接下来做什么。

如果你看了上述的伪代码，你可能会想：这看上去很容易理解嘛！我有一个很好的消息告诉你，我们`ParseProgram`方法和解析器看上去很类似，让我们开始工作吧！

再一次，在我们编写`parseProgram`方法，我们可以编写测试。下面的测试可以确保`let`语句解析正确。

```go
// parser/parser_test.go
package parser

import (
	"fmt"
	"monkey/ast"
	"monkey/lexer"
	"testing"
)

func TestLetStatements(t *testing.T) {
	tests := []struct {
		input              string
		expectedIdentifier string
		expectedValue      interface{}
	}{
		{"let x =5;", "x", 5},
		{"let z =1.3;", "z", 1.3},
		{"let y = true;", "y", true},
		{"let foobar=y;", "foobar", "y"},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain 1 statements. got=%d",
				len(program.Statements))
		}
		stmt := program.Statements[0]
		if !testLetStatement(t, stmt, tt.expectedIdentifier) {
			return
		}
		val := stmt.(*ast.LetStatement).Value
		if !testLiteralExpression(t, val, tt.expectedValue) {
			return
		}
	}
}

func testLetStatement(t *testing.T, s ast.Statement, name string) bool {
	if s.TokenLiteral() != "let" {
		t.Errorf("s.TokenLiteral not 'let'. got %q", s.TokenLiteral())
		return false
	}
	letStmt, ok := s.(*ast.LetStatement)
	if !ok {
		t.Errorf("s not *ast.LetStatement. got=%T", s)
		return false
	}
	if letStmt.Name.Value != name {
		t.Errorf("s.Name not '%s'. got=%s", name, letStmt.Name.Value)
		return false
	}
	if letStmt.Name.TokenLiteral() != name {
		t.Errorf("s.Name not '%s'. got=%s", name, letStmt.Name.Value)
		return false
	}
	return true
}
```
测试用例遵循我们先前词法解析器的测试规则，也是和其他每一个单元测试同样：我们提供Monkey代码作为输入，然后设置我们想要的抽象语法树的期待值，抽象语法树是由有解析器生成的。我们将会尽可能的检查抽象语法树的节点，确保我们没有丢失任何东西。

我将不选择`Mock`或者`Stub`方法而是提供源代码作为输入而不是tokens，因为那样做的话将会让我们的测试可读和可理解。当然我们的词法解析器中的bug将会使我们的测试变得面目全非并且生成很多噪声。但是我尽量将这个风险降到最低，尤其是当比较实用源代码作为输入的时候的优势的的时候。

在这里测试用例两个需要特别注意，第一个是我们忽略了`*ast.LetStatement`中的`Value`字段，为什么我们不去检查我们数字解析得是否正确呢？答案是我们是要做的，但是我们需要首先确保在解析`let`语句正确执行，所以忽略了`Value`字段。

第二个是帮助函数`testLetStatement`，它看上去使用单独函数像是过度设计化，但是我们在将来很需要这个函数，它能够将我们的测试用例变得可读而不是一行行的类型转换。

除此之外，在本章中我们将不会再次看全部的解析器测试，因为他们太长了，但是本书提供的代码包含全部代码。

在开始之前，测试时失败的：
```
$ go test ./parser
--- FAIL: TestLetStatement (0.00s)
parser_test.go:20: ParseProgram() return nil
FAIL
FAIL monkey/parser 0.007s
```
是时候开始填充我们的`ParseProgram()`方法了：
```go
// parser/parser.go
func (p *Parser) ParseProgram() *ast.Program{
    program := &ast.Program{}
    program.Statements = []ast.Statement{}
    for p.curToken.Type != token.EOF {
        stmt := p.parseStatement()
        if stmt != nil {
            program.Statements = append(program.Statements, stmt)
        }
        p.nextToken()
    }
    return program
}
```
是不是看上去和`parseProgram()`伪代码函数非常相像？是的，我之前告诉你着很像。

`parseProgram()`首先要做的事构建抽象语法树的根节点：`*ast.Program`，然后他迭代每一个`token`直至遇到`token.EOF`。它重复调用`nextToken`方法，该方法能够同时前进`p.curToken`和`p.peekToken`两个指针，每一次迭代调用`parseStatement`方法，它负责解析语句。如果它返回值不是`nil`而是`ast.Statement`，那么返回值将会添加到根节点的`Statements`切片中，如果没有任何`token`可供解析，那么`*ast.Program`将会被返回。

那么`pasreStatement`方法看上去是这样的：
```go
// parser/parser.go
func (p *Parser) parseStatement() *ast.Statement{
    switch p.curToken.Type{
    case token.LET:
        return p.parseLetStatement()
    default:
        return nil
    }
}
```
不用担心，`switch`语句将会得到更多的分支，但是现在，它仅仅地调用`parseLetStatement`方法如果只遇到`token.LET`。`parseLetStatement`方法将会让我们的测试通过：
```go
//parser/parser.go
func (p *Parser) parseLetStatement() *ast.LetStatement {
    stmt := &ast.LetStatement{Token: p.curToken}
    if !p.expectPeek(Token.IDENT){
        return nil
    }
    stmt.Name = &ast.Identifer{Token:p.curToken, Value: p.curToken.Literal}
    if !p.expectPeek(toekn.ASSIGN) {
        return nil
    }
    // TODO: We're skipping the expression util we
    // encoutner a semicolon
    for !p.curTokenIs(token.SEMICOLON){
        p.nextToken()
    }
    return stmt
}
func (p *Parser) curTokenIs(t token.TokenType) bool { 
    return p.curToken.Type == t
}
func (p *Parser) peekTokenIs(t token.TokenType) bool { 
    return p.peekToken.Type == t
}
func (p *Parser) expectPeek(t token.TokenType) bool { 
    if p.peekTokenIs(t) {
        p.nextToken()
        return true 
    } else {
        return false
    } 
}
```
它奏效了，我们的测试通过了
```
$ go test ./parser
ok monkey/parser 0.007s
```
我们可以解析`let`语句了，是不是很神奇，但是等等！

让我们从`parseLetStatement`开始，它使用当前指向的`token`(`token.LET`)构建了`*ast.LetStatement`节点，然后调用`expectPeek`方法确保下一个`token`是否符合预期。首先它期待`token.IDENT`，然后使用它来构建一个`*ast.Identifer`节点，然后期待一个等号，最后它跳过了等号后面的表达式解析直到要一个冒号。只要我们知道如何解析表达式，跳过的部分将会被替换。

`curTokenIs`和`peekTokenIs`方法不要太多的解释，它们非常有用，以至于我们在填充我们的解析器的时候一次次看到它们，我们可以用`!p.curToken(token.EOF)`代替for循环中`p.curToken.Type != token.EOF`条件。

除了看看这些小方法， 让我们讨论一下`expectPeek`方法，`expectPeek`方法也是一个判断函数，它们原先的的目的是确保解析过程中`Token`的顺序是否正确。 我们`expectPeek`方法用来检查`peekToken`类型是否满足要求，只有满足要求才会调用`nextToken`方法。 正如你看到的，这个在解析器中频繁出现。

但是如果我们遇到的`token`在`expectPeek`方法中不是期待的类型？目前，我们就是返回`nil`，在`ParseProgram`方法中将会忽略该空值。它会导致该语句会被忽略，因为错误的输入。你可以想象一下，它将会使得调试变得非常困难，因为没有人想接触那些没有么错误错误的解析器。

幸运的是，我们将会做一些微小的改动来将解决该问题：
```go
// parser/parser.go
type Parser struct { 
// [...]
errors []string 
// [...]
}
func New(l *lexer.Lexer) *Parser { 
    p := &Parser{
        l: l,
        errors: []string{}, 
    }
// [...]
}
func (p *Parser) Errors() []string { 
    return p.errors
}
func (p *Parser) peekError(t token.TokenType) {
    msg := fmt.Sprintf("expected next token to be %s, got %s instead", t, p.peekToken.Type)
    p.errors = append(p.errors, msg) 
}
```
现在解析器吼了`errors`字段，它们就是字符串切片，该字段在`New`方法的时候被初始化，当`peekToken`不匹配的时候，帮助函数`peekError`用来增加错误到`errors`中，通过使用`Errors`方法可以检查我们的解析器是否遇到任何错误。

拓展我们的测试来确保这个和我们期望的是一样的：
```go
// parser/parser_test.go
func TestLetStatements(t *testing.T) { 
// [...]
    program := p.ParseProgram() checkParserErrors(t, p)
// [...]
}
func checkParserErrors(t *testing.T, p *Parser) { 
    errors := p.Errors()
    if len(errors) == 0 {
        return
    }
    t.Errorf("parser has %d errors", len(errors)) 
    for _, msg := range errors {
        t.Errorf("parser error: %q", msg) 
    }
    t.FailNow() 
}
```
新的`checkParserErrors`帮助函数仅仅是我们的解析器中的错误，如果有任何错误，它将错误输出并且停止当前测试。

但是我们的当前解析器中没有生成任何错误，通过改变`epectPeek`我们可以自动增加一个错误如果我们调用得到错误的`token`。
```go
// parser/parser.go
func (p *Parser) expectPeek(t token.TokenType) bool { 
    if p.peekTokenIs(t) {
        p.nextToken()
        return true 
    } else {
        p.peekError(t)
        return false
    }
}
```
现在我们可以改变我们的测试用例从这样：
```
input := `
let x = 5;
let y = 10;
let foobar = 838383;
`
```
变成每个一个输入都是无效的，因为都是了相关的`token`
```
input := ` let x 5;
let = 10; 
let 838383; `
```
我们运行我们的测试可以看到新的解析错误
```
$ go test ./parser
--- FAIL: TestLetStatements (0.00s)
parser_test.go:20: parser_test.go:22: got INT instead" parser_test.go:22:
got = instead" parser_test.go:22: got INT instead"
parser has 3 errors
parser error: "expected next token to be =,\
parser error: "expected next token to be IDENT,\ parser error: "expected next token to be IDENT,\
FAIL
FAIL monkey/parser 0.007s
```
正如你看到的，我们的解析器展示的一些微小的功能：它输出它遇到的每一个错误的语句。它没有在遇到第一个的时候就退出了，而是将他们保存下来以便我们重新运行程序去获取全部语法错误。它相当有用，尽管我们没有错误的行号和列号。