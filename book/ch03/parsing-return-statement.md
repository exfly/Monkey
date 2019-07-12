# 解析 `Retrun` 语句

本小节我们将要解析 `return` 语句，在解析之前我们要在 `ast` 包中定义好必要的类型，以便能够在抽象语法树中表示它们。

接下就是 `Monkey` 语言中 `return` 语句的示例：

```monkey
return 5;
return 10;
return add(1, 5);
```

有了 `let` 语句的经验，可以很容易地看出 `return` 语句的结构：

```monkey
return <expression>;
```

`return` 语句由单个 `return` 关键字和一个表达式组成，这样定义 `asr.RetrunStatement` 非常简单。

```go
// ast/ast.go
type ReturnStatement struct {
    Token token.Token // the 'return' token
    ReturnValue Expression
}
func (rs *ReturnStatement) statementNode() {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```

上述的节点定义和之前 `let` 语句非常相似：用一个字段来初始化 `token`， `ReturnValue` 字段用来表示将要返回的表达式。现在我们会跳过解析表达式的过程，只处理分号。`statementNode` 和 `TokenLiteral` 方法用来实现 `Node` 和 `Statement` 接口。

接下来的测试内容和之前 `let` 语句也差不多：

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

现在测试是失败的，没关系因为在 `ParseProgram` 方法中还没有考虑 `token.RETURN` 考虑进去。

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

接下来是 `parseReturnStatement` 方法，该方法也是非常简单：

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

它唯一做的事就是构建一个 `ast.ReturnStatement` 实例，当前 `token` 就会作为 `Token` 字段，然后调用 `nextToken` 方法来解析表达式，这个现在暂时省略了，直到遇到一个分号。现在测试已经通过了：

```shell
$ go test ./parser
ok monkey/parser 0.009s
```

现在已经解析了 `Monkey` 语言中全部语句，接下来就是解析表达式。
