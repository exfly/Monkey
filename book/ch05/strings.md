# 字符串

在 `Monkey` 语言中，字符串就是一系列字符。它们是一等类型，可以绑定标识符、作为函数参数调用或者函数的返回值，看上去和其他编程语言一样：用双引号包含起来的字符。

除了数据类型本身，本小节中，我们也将增加字符串连接操作，通过中缀表达式 `+` 完成。

最终我们的效果如下：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let firstName = "Thorsten"
>> let lastName = "Ball"
>> let fullName = fn(first, last) { first + " " + last};
>> fullName(firstName, lastName)
Thorsten Ball
```

## 词法解析器支持字符串

首先要做的是在词法解析器中支持字符串字面值。字符串的基本数据结构如下所示：

```monkey
"<sequence of characters>"
```

就是一系列由双引号包含起来的一些列字符。想要词法解析器做的就是在我们处理每一个字面的字符串，所以 `"Hello World"` 就会变成单一的token.而不是三个`"`，`Hello World` 和 `"` `token`，使用单个 `token` 可以在我们的语法解析器中处理起来变得容易。所以我们需要在词法解析器部分做大量的工作。

当然使用多个 `token` 也是可以的，对有些情况和解析器非常有用，我们就可以在标识符周围使用`"`，但是这里已经拥有了 `token` `INT` 作为整型，它将自身的字符串字面值作为 `Literal` 字段。

现在开始对 `token` 和词法分析器做一步处理。首先要做的的是将 `STRING`  `token` 类型增加到 `token` 包中:

```go
//token/token.go
const (
// [...]
    STRING = "STRING"
// [...]
)
```

与此同时为词法解析器增加一些测试用例，确保能够正确支持字符串类型。因此拓展`TestNextToken` 测试函数的 `input` 内容：

```go
func TestNextToken2(t *testing.T) {
    input := `let five=5;
let ten =10;
let add = fn(x, y){
  x+y;
};
let result = add(five, ten);
!-/*5;
5<10>5;

if(5<10){
    return true;
}else{
    return false;
}
10 == 10;
10 != 9;
"foobar"
"foo bar"
`
    tests := []struct {
        expectedType    token.TokenType
        expectedLiteral string
    }{
// [...]
        {token.STRING, "foobar"},
        {token.STRING, "foo bar"},
        {token.EOF, ""},
// [...]
    }
// [...]
}
```

现在 `input` 多了两行包含字符串字面值，我们想把他们变成 `token`。`foobar` 确保词法解析器能够工作；`foo bar` 也是同样如此。

当然目前测试时失败的，我们还没在词法解析器中做任何东西：

想要让测试通过比你想象中的还要简单，我们所需要做的就是在词法解析器中为 `"` 增加分支和一些简单的帮助方法

```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
// [...]
    switch l.ch {
// [...]
    case '"':
        tok.Type = token.STRING
        tok.Literal = l.readString()
// [...]
    }
// [...]
}
func (l *Lexer) readString() string {
    position := l.position + 1
    for {
        l.readChar()
        if l.ch == '"' {
            break
        }
    }
    return l.input[position:l.position]
}
```

一个 `case` 分支和一个辅助函数，它不停的调用 `readChar` 函数直至遇到另外一个双引号。

如果你认为这样比较简单，可以增加转义字符的支持比如 `"hello \"world\""` ，`"hello \n world"` 和 `"hello\t\t\tworld"` 等等，现在测试通过。

```shell
$go test ./lexer
ok monkey/lexer 0.006s
```

接下来要做的事如何在语法解析器中处理它们。

## 解析字符串

为了让语法解析器能够将 `token.STRING` 转变为抽象语法树的节点，需要定义节点。幸运的是定义这个节点非常简单，它看上去和 `ast.IntegerLiteral` 非常相似，除了 `Value` 字段不是 `int64` 类型而是 `string` 类型。

```go
//ast/ast.go
type StringLiteral struct {
    Token token.Token
    Value string
}
func (sl *StringLiteral) expressionNode() {}
func (sl *StringLiteral) TokenLiteral() string { return sl.Token.Literal}
func (sl *StringLiteral) String() string { return sl.Token.Literal }
```

当然字面字符串是表达式而不是语句，它们执行得到字符串。有了上面的定义，我们编写一些测试来确保我们的解析器知道能够处理`token.STRING` `token` 并且输出 `*ast.StringLiteral`。

```go
//parse/parser_test.go
func TestStringLiteralExpression(t *testing.T){
    input := `"hello world";`
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)

    stmt := program.Statements[0].(*ast.ExpressionStatement)
    literal, ok := stmt.Expression.(*ast.StringLiteral)
    if !ok {
        t.Fatalf("exp not *ast.StringLiteral. got=%T", stmt.Expression)
    }
    if literal.Value != "hello world" {
        t.Errorf("literal.Value not %q, got=%q", "hello world", literal.Value)
    }
}
```

在前面我们已经看到很多次了并且我们也知道如何处理它们，我们所需要做的就是为在`prefixParsefn`注册`token.STRING`处理的函数。 这个解析函数返回一个`*ast.StringLiteral`。

```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
//[...]
    p.registerPrefix(token.STRING, p.parseStringLiteral)
//[...]
}
func (p *Parser) parseStringLiteral() ast.Expression{
    return &ast.StringLiteral{Token: p.curToken, Value: p.curToken.Literal}
}
```

现在测试通过

```shell
$go test ./parser
ok monkey/parser 0.007s
```

现在解析器将字面字符串变成 `token.STRING` 然后在解析器中将其变成`*ast.StringLiteral` 节点。现在我们已经准备在对象系统和执行器中做出一些变化。

## 执行字符串

在对象系统中表达字符串和整型一样简单，但是最大的原因是我们重用了 `Go` 语言的 `string` 类型。想想一下在客户端语言中增加一种宿主语言没有的数据结构，这将是需要大量的工作，比如 `C` 语言。但是现在只需要一个包含字符串类型新的对象。

```go
//object/object.go
const (
//[...]
    STRING_OBJ = "STRING"
)
type String struct {
    Value string
}
func (s *String) Type() ObjectType { return STRING_OBJ }
func (s *String) Inspect() string { return s.Value }
```

在我们的执行器中需要的做的就是将 `*ast.StringLiteral` 变成`object.String` 对象，下面是测试：

```go
//evalutor/evalutor_test.go
func TestStringLiteral(t *testing.T) {
    input := `"Hello World!"`
    evaluated := testEval(input)
    str, ok := evaluated.(*object.String)
    if !ok {
        t.Fatalf("object is not String. got=%T(%+v)",
            evaluated, evaluated)
    }
    if str.Value != "Hello World!" {
        t.Errorf("String has wrong value. got=%q", str.Value)
    }
}
```

`Eval`函数调用返回的不是`*object.String`而是一个`nil`。让测试通过只需要增加的代码比解析器中还要少，只需要两行。

```go
//evaluator/evaluator.go
func Eval(node ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.StringLiteral:
        return &object.String{Value:node.Value}
}
```

这样做可以让我们的测试通过，并且可以在REPL中使用字符串

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> "Hello World!"
Hello World
>> let hello = "Hello there, fellow Monkey users and fans"
>> hello
Hello there, fellow Monkey users and fans!
>> let giveMeHello = fn() {"Hello"}
>> giveMeHello()
Hello!
```

## 字符串拼接

拥有字符串类型相当不错，但是除了创建它们好像并不能做些其他的什么。我们现在就做出一些改变，解释器提供字符串拼接的功能，通过支持 `+` 操作符和字符串操作符即可。

同样拓展我们的`TestErrorHanding`函数来确保我们只增加了 `+` 操作符。

```go
//evalutor/evalutor_test.go
func TestErrorHanding(t *testing.T){
    tests :=[]struct {
        input string
        expectedMssage string
    }{
//[...]
        {
            `"Hello" - "World"`,
            "unknow operator: STRING - STRING"

        },
//[...]
    }
//[...]
}
```

测试还不能通过，我们需要改变的地方是 `evalInfixExpression`。在这里我们需要在已经存在的 `switch` 语句中增加分支，以便能够执行操作符两边都是字符串类型。

```go
//evalutor/evalutor.go
func evalStringInfixExpression(
    operator string,
    left, right object.Object,
)object.Object {
    if operator != "+" {
        return newError("unknow operator: %s %s %s",left.Type(), operator, right.Type())
    }
    leftVal := left.(*object.String).Value
    rightVal := right.(*object.String).Value
    return &object.String{Value: leftVal + rightVal}
}
```

首先检查是否为正确的操作符，如果它支持 `+` 操作符，将字符串对象封装起来，并且拼接左右两个操作数变成新的字符串。

如果我们想要支持更多的字符串操作，我们可以在这边添加。比如我们想要增减支持字符串比较的操作符 `==` 和 `!=`。指针比较并不起作用，至少不是我们想要的的结果，字符串的比较需要比较它们的值而不是指针。

这样我们的测试通过了

```shell
$ go test ./evaluator
ok monkey/evalutor 0.007s
```

现在我们能够使用字面字符串了，我们可以传递它们，绑定它们，将它们从函数中返回，也可以拼接它们。

```monkey
>> let makeGreeter = fn(greeting) { fn (name) { greeting + "" + name + "!" }}
>> let hello = makeGreeter("Hello")
>> hello("Thorsten")
Hello Thorsten!
>> let heythere = makeGreete("Hey There")
>> heythere("Thorsten")
Hey there Thorsten!
```

现在我们的解释器可以完成字符串的工作。
