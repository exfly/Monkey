**拓展解析器**

在我们继续之前，我们需要拓展我们的解析器。首先我们需要清理和拓展我们的测试套件，我不会将所有的的改变列出来，但是我们展示一些小的帮助函数，以便让姿势更容易理解。

我们已经有了`testIntegerLiteral`帮助，第二个函数`testIdentifier`可以帮助我们清理好多测试：
```go
// parser/parser_test.go
func testIdentifier(t *testing.T, exp ast.Expression, value string) bool {
	ident, ok := exp.(*ast.Identifier)
	if !ok {
		t.Errorf("exp not *ast.Identifier. got=%T", exp)
		return false
	}
	if ident.Value != value {
		t.Errorf("ident.Value not %s. got=%s", value, ident.Value)
		return false
	}
	if ident.TokenLiteral() != value {
		t.Errorf("ident.TokenLiteral not %s. got=%s", value, ident.TokenLiteral())
		return false
	}
	return true
}
```
最有趣的是使用`testIntegerLiteral`和`testIdentifier`可以构建很多泛化的帮助方法
```go
func testLiteralExpression(t *testing.T, exp ast.Expression, expected interface{}) bool {
	switch v := expected.(type) {
	case int:
		return testIntegerLiteral(t, exp, int64(v))
	case int64:
		return testIntegerLiteral(t, exp, v)
	case string:
		return testIdentifier(t, exp, v)
	}
	t.Errorf("type of exp not handled. got=%T", exp)
	return false
}
func testInfixExpression(t *testing.T, exp ast.Expression, left interface{},
	operator string, right interface{}) bool {
	opExp, ok := exp.(*ast.InfixExpression)
	if !ok {
		t.Errorf("exp is not ast.InfixExpression. got=%T(%s)", exp, exp)
		return false
	}
	if !testLiteralExpression(t, opExp.Left, left) {
		return false
	}
	if opExp.Operator != operator {
		t.Errorf("exp.Operator is not '%s'. got=%q", operator, opExp.Operator)
		return false
	}
	if !testLiteralExpression(t, opExp.Right, right) {
		return false
	}
	return true
}
```
有了这些替代，我们可以如下的方式编写测试代码：
```go
testInfixExpression(t, stmt.Expression, 5, "+", 10) 
testInfixExpression(t, stmt.Expression, "alice", "*", "bob")
```
它将我们的测试解析器生成的抽象语法树更加容易了。

**布尔字面值**

还有一些东西需要我们的编程语言来时间以便在解析器和抽象语法书中使用，在Monkey中我们可以这样使用布尔型
```
true;
false;
let foobar = true;
let barfoo = false;
```
正如标识符和整数字面值，它的抽象语法树表示也非常简单和短小：
```go
type Boolean struct {
	Token token.Token
	Value bool
}
func (b *Boolean) expressionNode()      {}
func (b *Boolean) TokenLiteral() string { return b.Token.Literal }
func (b *Boolean) String() string       { return b.Token.Literal }
```
`Value`字段可以保留`bool`的值，也就意味着我们可以保存`true`或者`fasle`(注意是Go语言中的布尔值)

有了抽象语法树的节点，我们可以增加测试。 测试函数`TestBooleanExpression`和测试函数`TestIdentifierExpression`和`TestIntegerLiteralExpression`非常相像，在这里我就不展示了。 下面的测试足够指出了我们如何即实现解析布尔字面值解析。
```
$ go test ./parser
--- FAIL: TestBooleanExpression (0.00s)
parser_test.go:470: parser has 1 errors
parser_test.go:472: parser error: "no prefix parse function for true found" FAIL
FAIL monkey/parser 0.008s
```
当然，我们需要为`token.TRUE`和`token.FALSE`注册`prefixParseFn`
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.TRUE, p.parseBoolean)
    p.registerPrefix(token.FALSE, p.parseBoolean)
// [...]
}
```
`parseBoolean`方法正如想象中的一样：
```go
//parser/parser.go
func (p *Parser) parseBoolean() ast.Expression {
    return &ast.Boolean{Token: p.curToken, Value: p.curTokenIs(token.TRUE)}
}
```
其中最有趣的部分是内联的`p.curTokenIs(token.TRUE)`方法的调用，但是它并不是真正有缺。只不过它不是很直接。换句话说，我们的解析器工作的很好，这也是`Pratt`方法的漂亮的地方，它非常容易去拓展。

现在测试通过了
```
$ go test ./parser
ok monkey/parser 0.006s
```
但是更有缺的地方时我们现在拓展几个测试用例来完成刚刚实现的布尔字面值，第一个后选择就是`TestOperatorPrecedenceParsing`，它使用了字符串比较机制
```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
        //[...]
		{"true", "true"},
		{"false", "false"},
		{"3>5==false", "((3 > 5) == false)"},
		{"3<5==true", "((3 < 5) == true)"},
		{"1+(2+3)+4", "((1 + (2 + 3)) + 4)"},
		{"(5+5)*2", "((5 + 5) * 2)"},
		{"2/(5+5)", "(2 / (5 + 5))"},
		{"-(5+5)", "(-(5 + 5))"},
	    //[...]
}
```
通过`testBooleanLiteral`方法，我们可以拓展`testLiteralExpression`函数来测试布尔字面值。
```go
// parser/parser_test.go
func testLiteralExpression(t *testing.T, exp ast.Expression, expected interface{}) bool {
	switch v := expected.(type) {
	//[...]
	case bool:
		return testBooleanLiteral(t, exp, v)
	//[...]
	}
}
func testBooleanLiteral(t *testing.T, exp ast.Expression, value bool) bool {
	bo, ok := exp.(*ast.Boolean)
	if !ok {
		t.Errorf("exp not *ast.Boolean. got=%T", exp)
		return false
	}
	if bo.Value != value {
		t.Errorf("bo.Value not %t, got=%t", value, bo.Value)
		return false
	}
	if bo.TokenLiteral() != fmt.Sprintf("%t", value) {
		t.Errorf("bo.TokenLiteral not %t, got=%s",
			value, bo.TokenLiteral())
		return false
	}
	return true
}
```
没有任何神奇的，只不过在`switch`语句中增加了一个`case`分支和新的帮助函数，在这个地方，我们也可以很容易拓展`TestParsingInfixExpressions`。
```go
// parser/parser_test.go
func TestParsingInfixExpressions(t *testing.T) { 
    infixTests := []struct {
        input string 
        leftValue interface{} 
        operator string 
        rightValue interface{}
}{
// [...]
    {"true == true", true, "==", true}, 
    {"true != false", true, "!=", false}, 
    {"false == false", false, "==", false},
}
// [...]
if !testLiteralExpression(t, exp.Left, tt.leftValue) { 
    return
}
if !testLiteralExpression(t, exp.Right, tt.rightValue) { 
    return
}
//[...]
```
同样`TestParsingPrefixExpressions`也很容易拓展，只需要测试表中增加新的入口：
```go
// parser/parser_test.go
func TestParsingPrefixExpressions(t *testing.T) { 
    prefixTests := []struct {
        input string 
        operator string 
        value interface{}
    }{
    // [...]
    {"!true;", "!", true},
    {"!false;", "!", false}, }
    // [...]
}
```
我们实现了解析布尔型字面值并且拓展了我们的测试，它可以给我们更多的测试覆盖率和更好的工具。

**分组表达式**

接下来我们要做的事如何解析分组的表达式，当然，在Monkey中，我们可以通过添加括号来对表达式进行分组从而修改优先级，以便适应他们执行的环境。接下来看一个例子：
```
(5+5)*2
```
使用括号将`5+5`表达式分组，那么他们有更高的优先级。他们在抽象语法树中的的位置更深，导致后面额执行结果和数学表达式相同。

或许你现在想：*该死的优先级怎么又来了，头都大了*， 如果你想跳过本章节，直接到最后。我建议别这么做！

我们不打算为分组表达式编写单元测试，因为它们不是有抽象语法树节点构成。我们不需要改变我的抽象语法树以便正确解析分组表达式。我们要做的是拓展我们的`TestOperatorPrecedenceParsing`测试函数，来确保括号正确的将表达式分组，而且对抽象语法树的结果又影响。
```go
// parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		//[...]
		{"1+(2+3)+4", "((1 + (2 + 3)) + 4)"},
		{"(5+5)*2", "((5 + 5) * 2)"},
		{"2/(5+5)", "(2 / (5 + 5))"},
		{"-(5+5)", "(-(5 + 5))"},
        {"!(true==true)", "(!(true == true))"},
        //[...]
	}
}
```
当然测试失败的：
```
$ go test ./parser
--- FAIL: TestOperatorPrecedenceParsing (0.00s)
    parser_test.go:531: parser has 3 errors
    parser_test.go:533: parser error: "no prefix parse function for ( found" 
    parser_test.go:533: parser error: "no prefix parse function for ) found" 
    parser_test.go:533: parser error: "no prefix parse function for + found"
FAIL
FAIL monkey/parser 0.007s
```
下面是我们最烧脑的部分，为了让测试通过，我们需要增加如下代码:
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.registerPrefix(token.LPAREN, p.parseGroupedExpression)
    // [...]
}
func (p *Parser) parseGroupedExpression() ast.Expression { 
    p.nextToken()
    exp := p.parseExpression(LOWEST)
    if !p.expectPeek(token.RPAREN) { 
        return nil
    }
    return exp 
}
```
是的，我们的测试通过，而且括号正如我们期望的那样，增加了被括号包含的测表达式的优先级。`token`类型关联函数的概念棒极了。

**条件表达式**

在Monkey语言中，我们可以使用`if`和`esle`语句，用法和其他编程语言一样的：
```
if (x > y) {
    return x;
}else{
    return y;
}
```
其中`else`还是可选的
```
if (x > y){
    return x;
}
```
上面的我们都很熟悉了，在Monkey中，`if-else-conditional`条件是表达式。那就意味着他们可以生成值，哪怕`if`表达式是最后执行的。我们甚至不需要在这里使用`return`语句。
```
let foobar = if (x>y) {x} else {y};
```
解释`if-else-conditional`结构是没有必要的，为了清除他们的命名，我们还是这样做了：
```
if (<condition>) <consequence> else <alternative>
```
使用方括号包含起来的`consequence`和`alternative`是语句块，语句块意味着他们是一系列语句（就跟Monkey中`programs`)。

目前我们的所有工作都是成功的，所以至于`if-else-conditional`也不例外。

下面是抽象语法树节点`ast.IfExpression`的定义：
```go
type IfExpression struct {
	Token       token.Token
	Condition   Expression
	Consequence *BlockStatement
	Alternative *BlockStatement
}

func (ie *IfExpression) expressionNode()      {}
func (ie *IfExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *IfExpression) String() string {
	var out bytes.Buffer
	out.WriteString("if")
	out.WriteString(ie.Condition.String())
	out.WriteString(" ")
	out.WriteString(ie.Consequence.String())
	if ie.Alternative != nil {
		out.WriteString("else")
		out.WriteString(ie.Alternative.String())
	}
	return out.String()
}
```
我们的`*ast.IfExpression`实现了`ast.Expression`接口，并且有三个字段代表了`if-else-conditional`。`Condition`保存了`condition`，它可以是任何表达式。`Consequence`和`Alternative`分别指向了`consequence`和`alteranative`。它们引用了新类型`ast.BlockStatement`，正如以前，`consequence/alterantive`是`if-else-condition`照片那个的一系列声明。它也是我们`ast.BlockStatement`代表的内容。
```go
type BlockStatement struct {
	Token      token.Token
	Statements []Statement
}

func (bs *BlockStatement) statementNode()       {}
func (bs *BlockStatement) TokenLiteral() string { return bs.Token.Literal }
func (bs *BlockStatement) String() string {
	var out bytes.Buffer
	for _, s := range bs.Statements {
		out.WriteString(s.String())
	}
	return out.String()
}
```
下一步就是测试，我们现在已经知道如何做了，测试如下：
```go
func TestIfExpression(t *testing.T) {
	input := `if(x<y){x}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Body does not contain %d statements. got=%d\n",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	exp, ok := stmt.Expression.(*ast.IfExpression)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.IfExpression. got=%T",
			stmt.Expression)
	}
	if !testInfixExpression(t, exp.Condition, "x", "<", "y") {
		return
	}
	if len(exp.Consequence.Statements) != 1 {
		t.Errorf("consequence is not 1 statements. got=%d\n",
			len(exp.Consequence.Statements))
	}
	consequence, ok := exp.Consequence.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("Statements[0] is not ast.ExpressionStatement. got=%T",
			exp.Consequence.Statements[0])
	}
	if !testIdentifier(t, consequence.Expression, "x") {
		return
	}
	if exp.Alternative != nil {
		t.Errorf("exp.Alternative.Statement was not nil. got=%+v", exp.Alternative)
	}
}
```
我也增加了`TestIfElseExpression`测试函数，然后使用下面的测试输入
```
if ( x< y ) { x } else { y }
```
在`TestIfElseExpression`中有额外的判断`Alternative`字段部分。所有对于`*ast.IfExpression`节点的结果判断使用了帮助函数`testInfixExpression`和`testIdentifer`。这样我们可以专注于条件表达式本身，来确保我们的解析器剩下的部分正确集成起来。

所有测试都以错误消息返回，我们对此已经非常熟悉了
```
$ go test ./parser
--- FAIL: TestIfExpression (0.00s)
    parser_test.go:659: parser has 3 errors
    parser_test.go:661: parser error: "no prefix parse function for IF found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
--- FAIL: TestIfElseExpression (0.00s)
    parser_test.go:659: parser has 6 errors
    parser_test.go:661: parser error: "no prefix parse function for IF found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found" 
    parser_test.go:661: parser error: "no prefix parse function for ELSE found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
首先我们开始第一个失败测试`TestifExpression`， 显然我们需要为`Token.IF`注册第`prefixParseFn`。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.IF, p.parseIfExpression)
// [...]
}
func (p *Parser) parseIfExpression() ast.Expression { 
    expression := &ast.IfExpression{Token: p.curToken}
    if !p.expectPeek(token.LPAREN) { 
        return nil
    }
    p.nextToken()
    expression.Condition = p.parseExpression(LOWEST)
    if !p.expectPeek(token.RPAREN) { 
        return nil
    }
    if !p.expectPeek(token.LBRACE) { 
        return nil
    }
    expression.Consequence = p.parseBlockStatement()
    return expression
}
```
在这个解析函数中，我们频繁调用`expectPeek`，这不仅仅是必须的，而且是有意义的。`expectPeek`如果期望的`token`不符合预期，那么它将会增加给解析增加一个错误，如果满足期望，它将会前进一个`token`，这的确是我们需要的，在这之后就是`{`，它标志着语句块的开始。

这个方法遵循我们我们的解析函数协议：`token`前进至`parseBlockStatement`开始的地方，当前`p.curToken`位于`{`处。下面是`parseBlockStatement`解析函数：
```go
// parser/parser.go
func (p *Parser) parseBlockStatement() *ast.BlockStatement {
	block := &ast.BlockStatement{Token: p.curToken}
	block.Statements = []ast.Statement{}
	p.nextToken()
	for !p.curTokenIs(token.RBRACE) {
		stmt := p.parseStatement()
		if stmt != nil {
			block.Statements = append(block.Statements, stmt)
		}
		p.nextToken()
	}
	return block
}
```
`parseBlockStatement`调用`parseStatement`方法一直到遇到`}`，它标记着语句块的结束。它上去和我们的顶层`parseProgram`方法非常像，在哪里我们不停地调用`parseStatement`方法，知道遇到结束`token`，在`ParseProgram`中，该结束`token`是`token.EOF`。这循环的重复并没有没有错误的影响，所以我们保留它们并且注意它们的测试用例：
```
$ go test ./parser
--- FAIL: TestIfElseExpression (0.00s)
    parser_test.go:659: parser has 3 errors
    parser_test.go:661: parser error: "no prefix parse function for ELSE found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
`TestIfExpression`测试通过，但是`TestIfElseExpression`并没有通过。现在为了支持`if-else-condition`中的`else`分支，我们需要检查该分支是否存在，以便我们能够在`else`分支解析它们。
```go
// parser/parser.go
func (p *Parser) parseIfExpression() ast.Expression {
//[...]
	expression.Consequence = p.parseBlockStatement()
	if p.peekTokenIs(token.ELSE) {
		p.nextToken()
		if !p.expectPeek(token.LBRACE) {
			return nil
		}
		expression.Alternative = p.parseBlockStatement()
	}
	return expression
}
```
这就是全部了，这个方法允许可选的`else`分支但是没有增加错误，在我们解析`consequence-block-statement`后，我们检查是否出现`token.ELSE`，如果存在我们跳过两个`token`，第一次调用`nextToken`的原因我们已经知道`p.peekToken`是`else`，然后我们调用`expectPeek`的原因是下一个`token`是一个语句块的开始部分`{`，否则这个程序是无效的。

是的，解析的过程非常容易出现错误，非常容易忘了前进`token`或者错误地调用`nextToken`方法。通过严格的协议表明每一个解析函数怎么去前进`token`很好的帮助我们。幸运的是我们用很好的测试用例来保证我们知道每一步工作是怎样的：
```
$ go test ./parser
ok monkey/parser 0.007s
```

**函数字面**

你已经注意到`parseIfExpression`方法中我们增加了很多代码，这个比任何一个`prefixParseFn`和`infixParseFn`方法还要多。因为我们针对不同的`token`和表达式类型有不得不做很多工作。接下来我们要做的的工作难度和之前的差不多，而且也涉及到更多的`token`类型。我们将要解析函数字面。

在Monkey中，通过函数字面来定义函数，包含了函数的参数和函数所做的内容。函数看上去是这样的：
```
fn(x, y) {
    return x+y;
}
```
它开始于关键字`fn`，紧接着是参数列表，然后是语句块，它就是函数体，函数在调用的时候，函数体将会被执行。抽象结构如下
```
fn <parameters><block statement>
```
我们已经知道语句快了，也只到如何解析它们。参数列表之前没有遇到过，但是解析它们并不困难。它们仅仅是一系列由逗号隔开的标识符，并且有括号包含起来：
```
(<parameter one>, <parameter two>, <parameter three>, ...)
```
这个列表可以是空的
```
fn(){
    return foobar + barfoo;
}
```
它们就是函数字面，那么他们属于那种抽象语法树节点类型呢？当然是表达式。将函数字面放到任何表达式位置都是有效的。举个例子，下面是将函数字面作为`let`语句的表达式部分：
```
let myFunction = fn(x, y) { return x + y; }
```
下面是将函数字面作为`return`语句的表达式部分，而且它还在其他函数内部：
```
fn(){
    return fn(x, y) { return x > y; }
}
```
也可以将一个函数字面作为其他函数调用的参数
```
myFunc(x, y, fn(x, y) {return x > y;});
```
听上去有点复杂，但是一点也不。其中最棒的是我么你的解析器一旦作为表达式解析成功，剩下的工作它将正确解析成函数。

我们已经知道函数字面值有两个主要的部分：参数列表和函数体的语句块，在定义抽象语法树节点的时候要注意这一点。

```go
type FunctionLiteral struct {
	Token      token.Token
	Parameters []*Identifier
	Body       *BlockStatement
}

func (fl *FunctionLiteral) expressionNode()      {}
func (fl *FunctionLiteral) TokenLiteral() string { return fl.Token.Literal }
func (fl *FunctionLiteral) String() string {
	var out bytes.Buffer
	params := make([]string, 0)
	for _, p := range fl.Parameters {
		params = append(params, p.String())
	}
	out.WriteString(fl.TokenLiteral())
	out.WriteString("(")
	out.WriteString(strings.Join(params, ", "))
	out.WriteString(") ")
	out.WriteString(fl.Body.String())
	return out.String()

}
```
`Parameters`字段是`*ast.Identfiers`的切片，`Body`字段是`*ast.BlockStatement`，这个我们之前看过。

下面是测试，在这里我们也使用了`testLiteralExpression`和`testInfixExpression`帮助函数：

```go
func TestFunctionLiteralParsing(t *testing.T) {
	input := `fn(x,y){x+y;}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Body does not coantin %d statement. got=%d",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	function, ok := stmt.Expression.(*ast.FunctionLiteral)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.FunctionLiteral. got=%T",
			stmt.Expression)
	}
	if len(function.Parameters) != 2 {
		t.Fatalf("stmt.Expression is not ast.FunctionLiteral. got=%T",
			stmt.Expression)
	}
	testLiteralExpression(t, function.Parameters[0], "x")
	testLiteralExpression(t, function.Parameters[1], "y")
	if len(function.Body.Statements) != 1 {
		t.Fatalf("function.Body.Statements has not 1 Statements. got=%d\n",
			len(function.Body.Statements))
	}
	bodyStmt, ok := function.Body.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("function body stmt is not ast.ExpressionStatement. got=%T",
			function.Body.Statements[0])
	}
	testInfixExpression(t, bodyStmt.Expression, "x", "+", "y")
}
```
测试主要分为三个主要部分：检查`*ast.FunctionLiteral`是否存在；检查参数列表是否正确；确保函数体包含正确的语句。最后一部分不是严格必须的，我们我们已经在`IfExpression`中已经测试用语句块，但是我们在这里重复之前工作也是无关紧要。

仅仅定义了`ast.FunctionLiteral`并没有改变我们的解析器，测试失败了
```

$ go test ./parser
--- FAIL: TestFunctionLiteralParsing (0.00s)
    parser_test.go:755: parser has 6 errors
    parser_test.go:757: parser error: "no prefix parse function for FUNCTION found" 
    parser_test.go:757: parser error: "expected next token to be ), got , instead" 
    parser_test.go:757: parser error: "no prefix parse function for , found" 
    parser_test.go:757: parser error: "no prefix parse function for ) found" 
    parser_test.go:757: parser error: "no prefix parse function for { found" 
    parser_test.go:757: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
显然我们需要为`token.FUNCTION`注册新的`prefixParseFn`。
```go
// parser/parsr.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.FUNCTION, p.parseFunctionLiteral)
// [...]
}
func (p *Parser) parseFunctionLiteral() ast.Expression { 
    lit := &ast.FunctionLiteral{Token: p.curToken}
    if !p.expectPeek(token.LPAREN) { 
        return nil
    }
    lit.Parameters = p.parseFunctionParameters()
    if !p.expectPeek(token.LBRACE) { 
        return nil
    }
    lit.Body = p.parseBlockStatement()
    return lit 
}
```
使用的`parseFunctionParameter`方法看上去是这样的
```go
//parser/parser.go
func (p *Parser) parseFunctionParameters() []*ast.Identifier {
	identifiers := make([]*ast.Identifier, 0)
	if p.peekTokenIs(token.RPAREN) {
		p.nextToken()
		return identifiers
	}
	p.nextToken()
	ident := &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	identifiers = append(identifiers, ident)

	for p.peekTokenIs(token.COMMA) {
		p.nextToken()
		p.nextToken()
		ident := &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
		identifiers = append(identifiers, ident)
	}
	if !p.expectPeek(token.RPAREN) {
		return nil
	}
	return identifiers
}
```
`parseFunctionParameter`核心的部分是构建参数切片通过不停地从逗号分隔的列表中创建标识符。这也表面如果列表为空，将会退出。

这个方法值得我们在编写一系列测试来检查边界用例，一个空的参数列表，只有一个参数和多个参数不同情况。
```go
func TestFunctionParameterParsing(t *testing.T) {
	tests := []struct {
		input            string
		expectedParameer []string
	}{
		{"fn(){}", []string{}},
		{"fn(x){}", []string{"x"}},
		{"fn(x,y){}", []string{"x", "y"}},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		stmt := program.Statements[0].(*ast.ExpressionStatement)
		function := stmt.Expression.(*ast.FunctionLiteral)
		if len(function.Parameters) != len(tt.expectedParameer) {
			t.Errorf("length parameters wrong. want %d, got=%d\n",
				len(tt.expectedParameer), len(function.Parameters))
		}
		for i, ident := range tt.expectedParameer {
			testLiteralExpression(t, function.Parameters[i], ident)
		}
	}
}
```
现在测试通过了
```
$ go test ./parser
ok monkey/parser 0.007s
```
现在我们已经正确解析了函数字面。


**调用表达式**

既然我们知道如何解析函数字面，下一步就应该解析函数调用：调用表达式。下面是结构：
```
<expression>(<comma separated expression>)
```
下面是调用表达式正常使用
```
add(2, 3)
```
如果你认为`add`是标识符，标识符就是表达式，参数`2`和`3`都是表达式，参数就是一系列表达式。
```
add(2+2, 3*3*3)
```
这个同样也是有效的，第一个参数是一个中缀表达式`2+2`，第二个是`3*3*3`。现在让我们看看这里函数调用，这个例子中，函数被绑定到标识符`add`，这个标识符在执行的时候返回该函数，这也就意味着我们可以直接跳过标识符，替换掉`add`函数字面。
```
fn(x, y){x+y; }(2, 3)
```
这也是有效的，我么也可以使用函数字面值作为参数
```
callsFunction(2, 3, fn(x, y) { x + y; });
```
所以再一次看看该结构
```
<expression>(<comma separated expressions>)
```
调用表达式有个一个表达式（执行的时候返回一个函数）和一系列表达式，它们是函数调用的是否的参数。所以抽象语法树的节点看上去是这样的
```go
// ast/ast.go
type CallExpression struct {
	Token     token.Token
	Function  Expression
	Arguments []Expression
}

func (ce *CallExpression) expressionNode()      {}
func (ce *CallExpression) TokenLiteral() string { return ce.Token.Literal }
func (ce *CallExpression) String() string {
	var out bytes.Buffer
	args := make([]string, 0)
	for _, a := range ce.Arguments {
		args = append(args, a.String())
	}
	out.WriteString(ce.Function.String())
	out.WriteString("(")
	out.WriteString(strings.Join(args, ", "))
	out.WriteString(")")
	return out.String()
}

type StringLiteral struct {
	Token token.Token
	Value string
}
```
测试也是同样如此，它对`*ast.CallExpression`的结构进行判断
```go
// parser/parser_test.go
func TestCallExpressionParsing(t *testing.T) {
	input := `add(1, 2*3, 4+5)`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Statements doest not contain %d statements. got=%d\n",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("stmt is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	exp, ok := stmt.Expression.(*ast.CallExpression)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.CallExpression. got=%T",
			stmt.Expression)
	}
	if !testIdentifier(t, exp.Function, "add") {
		return
	}
	if len(exp.Arguments) != 3 {
		t.Fatalf("wrong length of arguments. got=%d", len(exp.Arguments))
	}
	testLiteralExpression(t, exp.Arguments[0], 1)
	testInfixExpression(t, exp.Arguments[1], 2, "*", 3)
	testInfixExpression(t, exp.Arguments[2], 4, "+", 5)
}
```
通过调用函数字面值和参数解析很好的帮我们将参数解析测试部分分开。通过测试确保我们每个临界测试正确工作。我也增加了`TestCallExpressionParameterParsing`测试方法在本章的代码中。

到目前为止，测试还是失败的，我们得到了错误消息
```
$ go test ./parser
--- FAIL: TestCallExpressionParsing (0.00s)
    parser_test.go:853: parser has 4 errors
    parser_test.go:855: parser error: "expected next token to be ), got , instead" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for ) found"
FAIL
FAIL monkey/parser 0.007s
```
这个错误消息并没有很多信息，为什么没有错误消息提示我们为调用表达式注册`prefixParseFn`?因为我们在调用表达式中并没有增加新的`token`，那我们接下来怎么算呢？举个例子看一看
```
add(2, 3);
```
`add`是由`prefixParseFn`解析的标识符，在标识符之后是`token.LPAREN`，在标识符和参数列表中间，在中缀位置。是的我们有需要为`token.LPAREN`注册`infixParseFn`，这种方式我们解析为函数的比到时是，然后检查关联`token.LPAREN`的`infixParseFn`，调用该函数和进行解析的参数表达式一起使用。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.registerInfix(token.LPAREN, p.parseCallExpression)
    // [...]
}
// parse function call
func (p *Parser) parseCallExpression(function ast.Expression) ast.Expression {
	exp := &ast.CallExpression{Token: p.curToken, Function: function}
	exp.Arguments = p.parseExpressionList(token.RPAREN)
	return exp
}

// parse hash literal
func (p *Parser) parseHashLiteral() ast.Expression {
	hash := &ast.HashLiteral{Token: p.curToken}
	hash.Pairs = make(map[ast.Expression]ast.Expression)
	for !p.peekTokenIs(token.RBRACE) {
		p.nextToken()
		key := p.parseExpression(LOWEST)
		if !p.expectPeek(token.COLON) {
			return nil
		}
		p.nextToken()
		value := p.parseExpression(LOWEST)
		hash.Pairs[key] = value
		if !p.peekTokenIs(token.RBRACE) && !p.expectPeek(token.COMMA) {
			return nil
		}
	}
	if !p.expectPeek(token.RBRACE) {
		return nil
	}
	return hash
}
```
`parseCallExpression`接受一个已经解析的`function`作为参数，使用它构建一个`*ast.CallExpression`节点。为了解析参数列表，我们调用`parseCallArguments`, 它看上去和`parseFunctionParameters`非常类似，除了它看上去更加通用，因为它返回`ast.Expression`切片而不是`*ast.Identifier`.

下面的测试错误我们之前看过，我们还没有注册新的`infixParseFn`
```
$ go test ./parser
--- FAIL: TestCallExpressionParsing (0.00s)
    parser_test.go:853: parser has 4 errors
    parser_test.go:855: parser error: "expected next token to be ), got , instead" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for ) found"
FAIL
FAIL monkey/parser 0.007s
```
原因是在于`(`在`add(1, 2)`中，作为中缀操作符，但是我们我们还没有赋予优先级。它还没有右结合性。所以`parseExpression`不会返回我们期望的结果。但是调用表达式拥有最高的级别的优先级，所以现在修正我们的优先级表非常重要：
```go
// parser/parser.go
var precedences = map[token.TokenType]int{ 
    // [...]
token.LPAREN: CALL, 
}
```
为了确保我们额调用表达式拥有最高的优先级，我们拓展我们的`TestOperatorPrecedenceParsing`测试函数
```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		//[...]
		{"a + add(b*c)+d", "((a + add((b * c))) + d)"},
		{"a*[1,2,3,4][b*c]*d", "((a * ([1, 2, 3, 4][(b * c)])) * d)"},
    }
}
//[...]
}
```
现在再一次运行测试，我们可以看到它们都通过了
```
$ go test ./parser
ok monkey/parser 0.008s
```
是的，所有的测试都通过了，好消息是我们已经做完了基本解析器的工作。在本书的最后我们还会再回来的。现在抽象语法树全部定义好，解析器也能正确工作。是时候将我们的话题转移到执行部分。

在做之前，染我么移除先前TODO的内容和拓展我们的REPL以便集成到解析器中。

**移除TODO**
在我们编写解析`let`和`return`语句的时候，我们跳过了解析表达式部分：
```go
func (p *Parser) parseLetStatement() *ast.LetStatement {
	stmt := &ast.LetStatement{Token: p.curToken}
	if !p.expectPeek(token.IDENT) {
		return nil
	}
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}
    //TODO: We're skipping the expression until we
    // encounter a semicolon
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}
```
同样的`TODO`也出现在`parseRetrunStatement`, 是时候清除它们了。首先，我们需要拓展我们已经存在的测试来确保我们正确解析了`let`或者`return`语句中的表达式部分。我们使用帮助函数（这样不会转移我们注意力）和不同的表达式类型，这样我们知道`parseExpression`被正确集成了。

下面是`TestLetStatement`函数看上去的样子：
```go
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
```
在`TestReturnStatement`也有同样的需求，修正测试非常平常了，我们之前做了好多这样的工作。我们仅仅需要关注在`parseReturnStatement`和`parseLetStatement`中的`parseExpression`。而且我们主需要关注可选的分号，这个我们之间的`parseExpressionStatement`中已经知道了。新版的`parseReturnStatement`和`parseLetStatement`看上去如下
```go
func (p *Parser) parseLetStatement() *ast.LetStatement {
	stmt := &ast.LetStatement{Token: p.curToken}
	if !p.expectPeek(token.IDENT) {
		return nil
	}
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}
	p.nextToken()
	stmt.Value = p.parseExpression(LOWEST)
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}

// parse RETURN Statement
func (p *Parser) parseReturnStatement() *ast.ReturnStatement {
	stmt := &ast.ReturnStatement{Token: p.curToken}
	p.nextToken()
	stmt.ReturnValue = p.parseExpression(LOWEST)
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}
```
现在所有的TODO都移除了。

