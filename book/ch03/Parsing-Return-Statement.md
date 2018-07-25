解析`Retrun`语句

我先前说过，我们会一步步填充看上去很空旷的`ParseProgram`方法。现在就是时候，我们将要解析`return`语句。在解析之前要做的就是在`ast`包中定义额必要的结构以便我们在抽象语法树中能够表示它们。

接下来是在Monkey语言中的`return`语句：
```
return 5;
return 10;
return add(15);
```
有了`let`语句的经验，我们可以很容易地看出`return`语句的结构：
```
return <expression>;
```
`return`语句由单个`return`关键字和一个表达式组成，这样定义`ast.ReturnStatement`也就是非常简单的了：
```go
// ast/ast.go
type ReturnStatement struct {
    Token token.Token // the 'return' token
    ReturnValue Expression
}
func (rs *ReturnStatement) statementNode() {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```
上述的节点定义中，所有出现的内容你都看过：它有一个字段用来初始化`token`和`ReturnValue`字段来表示将要返回的表达式。现在我们会跳过解析表达式过程，只处理分号，后面我们会回来处理该部分的。`statementNode`和`TokenLiteral`方法使之能够满足`Node`和`Statement`接口。而且这些方法是定义在`*ast.LetStatement`上的。

接下来的测试部分和之前的`let`语句也是差不多相同。
```go
// parser/parser_test.go
func TestReturnStatements(t *testing.T) { 
    input := `
return 5; 
return 10; 
return 993322; `
    l := lexer.New(input) p := New(l)
    program := p.ParseProgram() checkParserErrors(t, p)
    if len(program.Statements) != 3 {
        t.Fatalf("program.Statements does not contain 3 statements. got=%d", len(program.Statements))
    }
    for _, stmt := range program.Statements {
    returnStmt, ok := stmt.(*ast.ReturnStatement) 
    if !ok {
        t.Errorf("stmt not *ast.returnStatement. got=%T", stmt)
        continue
    }
    if returnStmt.TokenLiteral() != "return" {
        t.Errorf("returnStmt.TokenLiteral not 'return', got %q", returnStmt.TokenLiteral())
    } 
}
```
当然测试会进行拓展只要表达式解析完成即可。但是没关系，测试不是一成不变的。但是现在测试时失败的：
```
$ go test ./parser
--- FAIL: TestReturnStatements (0.00s)
parser_test.go:77: program.Statements does not contain 3 statements. got=0 FAIL
FAIL monkey/parser 0.007s
```
所以现在通过修改`ParseProgram`方法将`token.RETURN`考虑进去就能够让测试通过：
```go

// parser/parser.go
func (p *Parser) parseStatement() ast.Statement { 
    switch p.curToken.Type {
    case token.LET:
        return p.parseLetStatement() 
    case token.RETURN:
        return p.parseReturnStatement() 
    default:
        return nil
    } 
}
```
接下来就是`parseReturnStatement`方法，该方法也是非常简单，没有很多的复杂的地方：
```go
// parser/parser.go
func (p *Parser) parseReturnStatement() *ast.ReturnStatement { 
    stmt := &ast.ReturnStatement{Token: p.curToken}
    p.nextToken()
    // TODO: We're skipping the expressions until we 
    // encounter a semicolon
    for !p.curTokenIs(token.SEMICOLON) {
        p.nextToken() 
    }
    return stmt 
}
```
它唯一做的事情就是构建一个`ast.ReturnStatement`对象，当前的`token`就是作为`Token`字段，然后通过调用`nextToken()`方法来解析表达式，但是现在并没有做，它跳过了每一个表达式直至遇到一个分号，就这样我们的测试通过了：
```
$ go test ./parser
ok monkey/parser 0.009s
```
可以再一次庆祝了，我们现在可以解析Monkey语言中的所有语句了。是的，在`Monkey`中只有两种语句：`Let`语句和`Retrun`语句。接下来我么我们的编程语言中只包含了表达式，接下来我们将会解析它们。