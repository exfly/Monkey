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
