# 拓展解释器

接下来拓展我们的解释器，首先要拓展清理和拓展测试套件，这里不会将所有的改变列出来，只会展示一些辅助函数，以便让知识更容易理解。

已经有了 `testIntegerLiteral` 辅助函数，第二个 `testIdentifier` 可以帮助清理掉很多测试：

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

最有趣的是使用 `testIntegerLiteral` 和 `testIdentifier` 可以构建出很多泛化的辅助函数：

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

现在我们可以按照如下方式编写测试代码：

```go
testInfixExpression(t, stmt.Expression, 5, "+", 10)
testInfixExpression(t, stmt.Expression, "alice", "*", "bob")
```

## 布尔字面值

在 `Monkey` 中我们也可以使用布尔型变量：

```monkey
true;
false;
let foobar = true;
let barfoo = false;
```

和标识符和整数字面值一样，它在抽象语法树中表示也非常简单：

```go
type Boolean struct {
    Token token.Token
    Value bool
}
func (b *Boolean) expressionNode()      {}
func (b *Boolean) TokenLiteral() string { return b.Token.Literal }
func (b *Boolean) String() string       { return b.Token.Literal }
```

`Value` 字段可以保留 `bool` 值，也就是 `Go` 语言中的 `true` 或者 `false`。有了抽象语法树节点，就可以增加测试，测试函数 `TestBooleanExpression` 和 `TestIdentifierExpression` 以及 `TestIntegerLiteralExpression` 非常一致，这里就不展示具体实现，目前测试还不能通过。

当然需要为 `token.TRUE` 和 `token.FALSE` 注册 `prefixParseFn`。

```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser {
// [...]
    p.registerPrefix(token.TRUE, p.parseBoolean)
    p.registerPrefix(token.FALSE, p.parseBoolean)
// [...]
}
```

`parseBoolean` 方法和之前一样：

```go
//parser/parser.go
func (p *Parser) parseBoolean() ast.Expression {
    return &ast.Boolean{Token: p.curToken, Value: p.curTokenIs(token.TRUE)}
}
```

其中最有趣的部分是内联的 `p.curTokenIs(token.TRUE)` 方法调用，它的调用并不是很直接，但是解释器能够正确的工作，这也是 `Pratt` 方法精彩的部分。

现在测试通过：

```shell
$ go test ./parser
ok monkey/parser 0.006s
```

但是更有趣的地方时我们现在拓展几个测试用例来完成刚刚实现的布尔字面值，第一个选择就是 `TestOperatorPrecedenceParsing`，它使用字符串比较机制：

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

通过 `testBooleanLiteral` 函数，沃恩可以拓展 `testLiteralExpression` 函数来测试布尔型字面值。

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

只需要在 `switch` 语句中增加一个 `case` 分支和新的辅助函数，那么就可以拓展 `TestParsingInfixExpressions` 。

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

同样拓展 `TestParsingPrefixExpressions` 函数，只需要在测试表中增加新的入口：

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

## 分组表达式

接下来是要如何去解析分组表达式，当然在 `Monkey` 中，我们可以通过添加括号对表达式分组从而修改优先级，以便适应它们的执行环境，下面是这个例子：

```monkey
(5+5)*2
```

使用括号将 `5+5` 表达式分组，那么它就拥有更高的优先级，在抽象语法树中位置更低，因而保证执行结果和数学表达式相同。或者你现在想
> 该死的优先级怎么又来了，头都大了！

如果你想跳过本章节，可以直接到达最后，但是我还是不建议你这么做。我们打算不为分组表达式编写专门的单元测试，因为他们不是由抽象语法树节点构成，不需要改变抽象语法树以便正确解析分组表达式。我们需要做的是拓展 `TestOperatorPrecedenceParsing` 测试函数，来确保括号正确地将表达式分组。

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

现在测试是失败的，接下来需要增加下面的代码：

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

现在测试通过了，而且括号正如我们期望的那样，这些都是用 `token` 关联相关函数完成。

## 条件表达式

在 `Monkey` 语言中，我们可以使用 `if` 和 `else` 语句，用法和其他编程语言一样：

```monkey
if (x > y) {
    return x;
}else{
    return y;
}
```

其中 `else` 分支是可选的：

```monkey
if (x > y){
    return x;
}
```

在 `Monkey` 中叫做 `if-else-conditional`条件表达式，也就意味着它能生成值，哪怕 `if` 表达式最后执行，甚至都不需要使用 `return` 语句。

```monkey
let foobar = if (x>y) {x} else {y};
```

为了更好的表示条件语句，我们对它们各个部分进行重新命名：

```monkey
if (<condition>) <consequence> else <alternative>
```

使用方括号包含起来的是 `consequence` 和 `alternative` 语句块，语句块也就是意味着它们是一系列的语句。下面是抽象语法树 `ast.IfExpression` 的定义：

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

我们的 `*ast.IfExpression` 实现了 `ast.Expression` 接口，包含了三个字段来表示 `if-else-conditional`。`Condition` 保存了 `condition`，它可以是任何表达式；`Consequence` 和 `Alternative` 分别指向 `consequence` 和 `alternative`，它们引用的新类型是 `ast.BlockStatement`，代表了条件语句中的花括号包含的语句块。

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

首先增加我们的测试：

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

也增加了 `TestIfElseExpression` 测试函数，然后使用下面的测试输入：

```monkey
if (x < y) { x } else { y }
```

在 `TestIfElseExpression` 中有额外判断 `Alternative` 字段，在 `*ast.IfExpression` 节点判断使用了辅助函数 `testInfixExpression` 和 `testIdentifer`。这样我们专注于表达式本身，来确保解析器剩下的部分能正确解析出来。

现在测试肯定是失败的，我们需要为 `Token.IF` 注册函数 `prefixParseFn` 函数。

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

在解析函数中频繁调用 `expectPeek`， 这个是的确需要的，`expectPeek` 中期望的 `token` 不符合预期，那么它将会生成一个解析错误；如果期望满足，则前进一个 `token`，之后就是 `{`，它标志着语句块的开始。`token` 前进到 `parseBlockStatement` 开始的地方，当前 `p.curToken` 位于 `{` 处。

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

`parseBlockStatement` 调用 `parseStatement` 方法一直遇到 `}`，它标记着语句块的结束。它和顶层的 `parseProgram` 方法非常像，在 `parseProgram` 中，结束的 `token` 是 `token.EOF`。

现在 `TestIfExpression` 测试通过，但是 `TestIfElseExpression` 并没有通过。现在为了支持 `if-else-condition` 中的 `else` 分支，需要检查该分支是否存在，以便能够在 `else` 分支解析它们。

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

该方法允许可选的 `else` 分支，在解析`consequence-block-statement` 后，我们检查是否出现 `token.ELSE`，如果存在则跳过两个 `token`，第一次调用 `nextToken` 的是已经知道 `p.peekToken` 是 `else`，然后调用 `expectPeek` 原因是下一个 `token` 是一个语句块开始 `{`，否则这个程序是非法的。

解析的过程非常容易出错，容易忘记前进 `token` 或者错误地调用 `nextToken` 方法，好在我们有严格的协议保证如何去前进这些 `token`，外加良好的测试用例保证每一步工作都是正确的。

```shell
$ go test ./parser
ok monkey/parser 0.007s
```

## 字面函数

你已经注意到 `parseIfExpression` 方法中增加了很多代码，这个比 `prefixParseFn` 和 `infixParseFn` 方法还要多。因为我们针对不同的 `token` 和表达式类型不得不做很多额外的工作。接下来要做的工作难度和之前差不多，而且涉及到更多的 `token`，接下来我们要解析字面函数。

在 `Moneky` 中，使用字面来定义函数，包含了函数的参数和函数体所有的内容，函数看上去是这样的：

```monkey
fn(x, y) {
    return x+y;
}
```

它开始于关键字 `fn`，紧接着是参数列表，然后是语句块也就是函数体，函数在调用的时候，函数体就会被执行，抽象结构如下：

```monkey
fn <parameters><block statement>
```

我们已经知道如何解析语句块，之前没有遇到参数列表，但是解析它们也不困难，它们仅仅是一系列由逗号隔开的标识符，最外面包含括号。

```monkey
(<parameter one>, <parameter two>, <parameter three>, ...)
```

这个列表也可以是空的

```monkey
fn(){
    return foobar + barfoo;
}
```

它们就是字面函数，那么它们属于抽象语法树中的哪一个节点呢？当然是表达式，将字面函数放在任何表达式位置都是有效的。举个例子，下面将字面函数作为 `let` 语句的表达式部分：

```monkey
let myFunction = fn(x, y) { return x + y; }
```

下面是将字面函数作为 `return` 语句的表达式部分，而且也在其他函数内部：

```monkey
fn(){
    return fn(x, y) { return x > y; }
}
```

也可以将一个字面函数作为其他函数调用的参数：

```monkey
myFunc(x, y, fn(x, y) {return x > y;});
```

虽然看上去很复杂，但是只要解析器将它作为表达式解析成功，剩下的工作自然而然就能完成。

我们已经知道字面函数有两个主要部分组成：参数类表和函数体语句块，在定义抽象语法树节点要注意到这点。

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

`Parameters` 字段是 `*ast.Identifier` 切片，`Body` 字段是 `*ast.BlockStatement`。下面就是单元测试，这里也同样使用了 `testLiteralExpression` 和 `testInfixExpression` 辅助函数。

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

测试分为三个主要部分：检查 `*ast.FunctionLiteral` 是否存在；检查参数列表是否正确；检查函数体是否报站正确的语句。最后一个检查不是严格执行的，因为在 `IfExpression` 中已经测试了语句块解析。

现在测试不能通过，需要我们为 `token.FUNCTION` 注册新的 `prefixParseFn`。

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

使用 `parseFunctionParameter` 方法看上去是这样的：

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

`parseFunctionParameter` 方法的和兴是构建参数切片，不同从逗号分隔的列表中创建标识符，如果这里为空则退出。这个方法值得我们增加单元测试来检查边界用例，一个空的参数列表，只有一个参数和多个参数等不同情况。

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

现在测试通过了：

```shell
$ go test ./parser
ok monkey/parser 0.007s
```

## 调用表达式

之前我们已经知道如何解析字面函数，下一步就是解析函数调用：调用表达式，下面就是这个结构：

```monkey
<expression>(<comma separated expression>)
```

调用表达式正常使用如下：

```monkey
add(2, 3)
```

如果你认为 `add` 是标识符，那么它既是表达式，参数 `2` 和 `3` 都是表达式，参数就是一系列表达式。

```monkey
add(2+2, 3*3*3)
```

这个也是同样合法的函数调用，第一个参数是中缀表达式 `2+2`，第二个是 `3*3*3`。现在看看这里的函数调用，在这个例子中，函数被绑定到标识符 `add`，这个标识符在执行的时候就返回该函数，也即是我们可以跳过标识符，替换掉 `add` 字面值。

```monkey
fn(x, y){x+y; }(2, 3)
```

也可以使字面函数作为参数：

```monkey
callsFunction(2, 3, fn(x, y) { x + y; });
```

所以再看一下这个结构

```monkey
<expression>(<comma separated expressions>)
```

调用表达式是由一个表达式（执行时候返回一个函数）和一系列表达式组成（函数调动的参数），所以抽象语法树上的节点看上去是这样的。所以抽象语法树看上去是这样的：

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

单元测试对 `*ast.CallExpression` 结构进行判断：

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

目前单元还是不能通过，虽然在调用表达式没有增加新的 `token`，那我们接下来怎么算呢？举个例子来看：

```monkey
add(2, 3);
```

`add` 是由 `prefixParseFn` 解析的标识符，在标识符之后是 `token.LPAREN`，在标识符和参数列表中间，我们需要为 `token.LPAREN` 注册 `infixParseFn` 方法，然后查找关联 `token.LPAREN` 的 `infixParseFn`，调用该函数和解析出来的参数表达式一起使用。

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

`parseCallExpression` 接受一个已经解析出来的 `function` 作为参数，然后使用它构建 `*ast.CallExpression` 节点。为了解析参数列表，调用 `parseCallArguments`，它看上去和 `parseFunctionParameters` 非常类似，除了看上去更加通用，因为它返回 `ast.Expression` 切片而不是 `*ast.Identifier`。

现在测试还是不能通过，因为还没有注册新的 `infixParseFn`，在 `(` 在 `add(1,2)` 中，作为中缀操作符，我们还没有赋予优先级，它还没有右结核性。所以在 `parseExpression` 不会返回期望的结果，但是调用表达式有最高级别的优先级，所以修改我们的优先级非常重要。

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

现在测试通过：

```shell
$ go test ./parser
ok monkey/parser 0.008s
```

## 移除 `TODO`

我们在编写解析 `let` 和 `return` 语句的时候，我们跳过了解析表达式部分：

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

同样的 `TODO` 也出现了 `parseRetrunStatement`，是时候清除它们了。首先需要拓展已经存在的测试来确保正确解析了 `let` 或者 `return` 语句中的表达式部分，这里借助辅助函数，避免过多的分散过度的注意力。下面是 `TestLetStatement` 函数。

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

在 `TestReturnStatement` 也有同样的需求，修正测试非常明了，我们之前做了好多这样的工作。仅仅需要关注在 `parseReturnStatement` 和`parseLetStatement` 中的 `parseExpression` 。而且只需要关注可选的分号，这个中间的 `parseExpressionStatement` 中已经知道了。新版的`parseReturnStatement` 和 `parseLetStatement` 看上去如下：

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
