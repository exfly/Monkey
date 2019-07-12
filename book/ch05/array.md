# 数组

在这一小节将为 `Monkey` 解释器增加数组数据类型，在 `Monkey` 中，数组是可能不同类型元素组成的有序列表。数组中的每一个元素都可以被单独被访。数组可以由字面表达方式：一系列由冒号隔开的元素，并且被包含在方括号中。

初始化一个新的数组，将其绑定到一个标识符中，访问每一个元素的操作如下：

```monkey
>> let myArray = ["Thorsten", "Ball", 28 , fn(x){x * x}];
>> myArray[0]
Thorsten
>> myArray[2]
28
>> myArray[3](2);
4
```

正如你看的那样，`Monkey` 中的数组不关心元素的类型，每一个类型都可以作为一个数组的内容。在这个例子中 `myArray` 包含了两个字符串、一个整型和一个函数。可以通过索引访问每一个元素，就像接下来三行代码，这是通过新的操作符，叫做索引操作符 `array[index]` 完成的。

在本小节中，`len` 内置函数同样支持数组操作，也增加一些内置函数可以操作数组。

```monkey
>> let myArray = ["one", "two", "three"];
>> len(myArray)
3
>> first(myArray)
one
>> rest(myArray)
[two, three]
>> push(myArray, "four")
[one, two, three, four]
```

`Monkey` 语言实现的基础部分是`go` 语言的 `[]object.Object`，这也就意味着我们不需要自己实现数据结构，只需要重用 `go` 的切片。

首先要做的就是让我们的词法分析器识别新的 `token` 。

## 词法分析器支持数据

为了正确解析数组字符串和和索引操作，我们词法解析器需要能够确定更多的`token`。我们需要构造的解析器是 `[,]` 和 `,`，词法解析器先前完成的部分已经完成 `,`，所以我们只需要支持 `[` 和 `]`。

首先要做的是在 `token` 包中定义新的 `token` 类型。

```go
// token/token.go
const (
// [...]
    LBRACKET = "["
    RBRACKET = "]"
// [...]
)
```

第二步需要做的是拓展测试部分，以便适应词法解析器。 这一点非常简单，之前已经重复很多次了。

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
[1,2];
`
    tests := []struct {
        expectedType    token.TokenType
        expectedLiteral string
    }{
//[...]
        {token.LBRACKET, "["},
        {token.INT, "1"},
        {token.COMMA, ","},
        {token.INT, "2"},
        {token.RBRACKET, "]"},
        {token.EOF, ""},
    }
// [...]
}
```

`input` 部分拓展为包含了新的 `token` (在本例中是 `[1,2]`)，新的测试用语句已经被添加，以确保词法解析器中的 `NextToken` 方法能够正确返回`token.LBRACKET` 和 `token.RBRACKET`。

使测试通过也非常简单，只要增加在 `NextToken` 方法中增加四行代码：

```go
//lext/lext.go
func (l *Lexer) NextToken() token.Token {
// [...]
    case '[':
        tok = newToken(token.LBRACKET, l.ch)
    case ']':
        tok = newToken(token.RBRACKET, l.ch)
// [...]
}
```

现在所有的测试已经通过

```shell
$ go test ./lexer
ok monkey/lexer 0.006s
```

所以在我们的语法解析中，我们可以使用 `token.LBRACKET` 和 `tokne.RBRACKET` 来解析数组。

## 解析数组

在我们前面所说的，`Monkey` 数组就是有逗号隔开的列表，并且它们被包含在一个方括号中。

```monkey
[1, 2, 3 + 3, fn(x), add(2, 2)]
```

数组中的每一个元素可以是任何类型的表达式，整型、函数、中缀或者前缀表达式。如果听起来非常复杂，不用担心。在函数调用参数解析那一部分我们已经知道如何解析由逗号隔开的列表表达式。而且我们也知道如何去解析一些被匹配的 `token` 包含的表达式，换句话说也就是我们已经知道如何去处理它们了。

首先要在抽象语法树中定义节点用来表达数组，既然已经知道其中的最基础的内容，代码就很容易解释了：

```go
// ast/ast.go
type ArrayLiteral struct {
    Token token.Token // the '[' token
    Elements []Expression
}
func (al *ArrayLiteral) expressionNode() {}
func (al *ArrayLiteral) TokenLiteral() string { return al.Token.Literal }
func (al *ArrayLiteral) String() string {
    var out bytes.Buffer
    elements := []string{}
    for _, el := range al.Elements {
        elements := append(elements, el.String())
    }
    out.WriteString("[")
    out.WriteString(string.Join(elements, ", "))
    out.WriteString("]")
    return out.String()
}
```

接下来测试函数用来确保能够正确解析数组，并且返回 `*ast.ArrayLiteral` (增加了测试函数用来检测空的数组来确保不会陷入讨厌的边缘用例)

```go
// parser/parser_test.go
func TestParsingArrayLiteral(t *testing.T) {
    input := `[1, 2*2, 3+3]`
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)
    stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
    array, ok := stmt.Expression.(*ast.ArrayLiteral)
    if !ok {
        t.Fatalf("exp not ast.ArrayLiteral. got=%T", stmt.Expression)
    }
    if len(array.Elements) != 3 {
        t.Fatalf("len(array.Elements) not 3. got=%d", len(array.Elements))
    }
    testIntegerLiteral(t, array.Elements[0], 1)
    testInfixExpression(t, array.Elements[1], 2, "*", 2)
    testInfixExpression(t, array.Elements[2], 3, "+", 3)
}
```

为了确保解析正确工作，测试输入部分包含了两种不同的中缀表达式，尽管整型和布尔型足够了。除此之外测试部分也是非常枯燥，但是确保解析器能够返回 `*ast.ArrayLiteral` 对象并且包含正确数量的元素。

为了让测试通过，我们需要注册新的前缀表达式函数，因为左方括号是一个前缀表达式。

```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
// [...]
    p.registerPrefix(token.LBRACKET, p.parseArrayLiteral)
// [...]
}
func (p *Parser) parserArrayLiteral() ast.Expression{
    array := &ast.ArrayLiteral{Token: p.curToken}
    array.Elements = p.parseExpressionList(token.RBRACKET)
    return array
}
```

先前我们已经添加了 `prefixParserFns`，但是这里值得注意的是我们增加了新的方法 `parseExpressionList` 。该方法修改了先前的 `parserCallArgument` 方法并且变得更加通用，先前是用来解析 `parserCallExpresion` 中的用逗号隔开的参数列表。

```go
//parser/parser.go
func (p *Parser) parseExpressionList(end token.TokenType) []ast.Expression {
    list := make([]ast.Expression, 0)
    if p.peekTokenIs(end) {
        p.nextToken()
        return list
    }
    p.nextToken()
    list = append(list, p.parseExpression(LOWEST))
    for p.peekTokenIs(token.COMMA) {
        p.nextToken()
        p.nextToken()
        list = append(list, p.parseExpression(LOWEST))
    }
    if !p.expectPeek(end) {
        return nil
    }
    return list
}
```

再一次在 `parseCallArugment` 方法中见过这个方法，唯一的变化是这个版本接受一个终结的参数用来告诉方法哪一个是列表的结束。这个新版本的`paseCallExpression`方法现在看上去是这样的：

```go
//parser/parser.go
func (p *Parser) parserCallExpresison(function ast.Expression)ast.Expression{
    exp := &ast.CallExpression{Token:p.curToken, Function:function}
    exp.Argument = p.parserExpresson(token.RPAREN)
    return exp
}
```

唯一改变的地方时我们调用 `paserExpressionList` 使用 `token.RPAREN` （这个用来确定参数的结束)。通过修改几行代码就能重新使用这个方法。现在测试通过：

```shell
$ go test  ./parser
ok monkey/parser 0.007s
```

## 解析索引表达式

为了完整的支持数组，我们不仅仅要求解析出数组，还要解析出索引表达式。或许索引操作符听上去没有什么印象，但是你肯定知道这是什么回事。索引操作符如下所示：

```monkey
myArray[0];
myArray[1];
myArray[2];
```

这些事最简单的形式，但是还有其他很多例子，如下所示：

```monkey
[1, 2, 3, 4][2];
let myArray = [1, 2, 3, 4];
myArray[2];
myArray[2+1];
returnsArray()[1];
```

是的，这些都是正确的，最基本的表达式是`<expression>[<expression>]`。这个看上去足够简单，我们可定定义新的抽象语法树的节点：`ast.IndexExpression` ，结构如下：

```go
//ast/ast.go
type IndexExpression struct {
    Token token.Token
    Left  Expression
    Index Expression
}

func (ie *IndexExpression) expressionNode()      {}
func (ie *IndexExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *IndexExpression) String() string {
    var out bytes.Buffer
    out.WriteString("(")
    out.WriteString(ie.Left.String())
    out.WriteString("[")
    out.WriteString(ie.Index.String())
    out.WriteString("])")
    return out.String()
}
```

值得注意的是 `left` 和 `index` 字段都是表达式。`left` 使我们可以访问的对象，它可以是任何类型：标识符、字面数组或者函数调用。对于 `index` 也是同样如此，任何表达式都可以。语法上来讲并没有任何改变，在语义上都会生成一个整数。

事实上，`left` 和 `index` 都是表达式更加有助于处理，因为我们可以用`parseExpression` 方法来处理他们。但是第一件事就是我们的测试用例能够知道并且返回一个`*ast.IndexExpression`：

```go
//parser/parser_test.go
func TestParsingIndexExpression(t *testing.T) {
    input := "myArray[1+1]"
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)
    stmt, _ := program.Statements[0].(*ast.ExpressionStatement)
    indexExp, ok := stmt.Expression.(*ast.IndexExpression)
    if !ok {
        t.Fatalf("exp not *ast.IndexExpression. got=%T", stmt.Expression)
    }
    if !testIdentifier(t, indexExp.Left, "myArray") {
        return
    }
    if !testInfixExpression(t, indexExp.Index, 1, "+", 1) {
        return
    }
}
```

现在测试只能确保解析器只能返回正确的抽象语法树当只有单一的表达式语句包单个索引表达式。但是同样重要的是解析器能够正确处理索引表达式的优先级。目前为止，索引操作符拥有最高的操作符，可以拓展我们的`TestOperatorPrecedenceParsing` 测试函数。

```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
    tests := []struct {
        input    string
        expected string
    }{
//[...]
        {"a*[1,2,3,4][b*c]*d", "((a * ([1, 2, 3, 4][(b * c)])) * d)"},
        {"add(a*b[2], b[1], 2 * [1,2][1])", "add((a * (b[2])), (b[1]), (2 * ([1, 2][1])))"},
    }
//[...]
}
```

在 `String()` 方法中增加的 `(` 和 `)` 输出 `*ast.IndexExpression` 有助于我们编写测试用例，因为它能使得索引操作符号清晰的表达。在测试用例中我们期望在中缀表达式中索引操作符的优先级比调用表达式甚至 `*` 操作符更高。

测试是失败的因为解析器不知道任何关于索引表达式的内容，尽管测试中输的的缺失 `prefixParseFn` 方法就是想要的，但是索引表达式并不需要在一个操作符两边有各有一个操作数。为了解析很方便地解析它们，这样做大有裨益，就跟先前函数调用表达式解析一样。具体来讲就是将`[`在`myArray[0]`作为一个中缀表达式，`myArray`作为左边操作数而`0`作为右边操作数。

```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
// [...]
    p.registerInfix(token.LBRACKET, p.parserIndexExpression)
// [...]
}
func (p *Parser) parserIndexExpression(left ast.Expression)ast.Expression{
    exp := &ast.IndexExpression{Token: p.curToken, Left:left}
    p.nextToken()
    p.Index = p.parserExpression(LOWEST)
    if !p.expectPeek(Token.RBRACKET){
        return nil
    }
    return exp
}
```

很棒，但是并没有修复我们没有通过的测试，原因是在Pratt解析中的优先级问题，我们还没有定义索引操作符的优先级。

```go
// parser/parser.go
const (
    _ int = iota
// [...]
    INDEX
)
var precedneces = map[token.TokenType]int{
// [...]
    token.LBRACKET: INDEX
}
```

最重要的一点是将 `INDEX` 放在最后一行，借助 `iota` , 它将`INDEX`最高的优先级。新增加的词条给 `token.LBRACKET` 最高级别的优先级。当然，一切都通过。

```shell
$ go test ./parser
ok monkey/parser 0.007s
```

词法分析完成、语法分析完成，接下来是执行器。

## 执行数组

执行数组并不难，将数组映射到 `Go` 语言的切片类型非常简单，也就不需要自己实现新的数据结构。仅仅需要定义新的 `object.Array` 类型，它将是数组类型生成的结果。定义 `object.Array` 非常简单，因为在Monkey语言中数组非常简单：它们仅仅就是一系列对象。

```go
// object/object.go
const (
// [...]
    ARRAY_OBJ = "ARRAY"
)
type Array struct {
    Elements []object
}
func (ao *Array) Type() ObjectType { return ARRAY_OBJ }
func (ao *Array) Inspect() string {
    var out bytes.Buffer
    elements := []string{}
    for _, e := range ao.Elements {
        elements = append(elements, e.Inspect())
    }
    out.WriteString("[")
    out.WriteString(string.Join(elements, ", "))
    out.WriteString("]")
    return out.String()
}
```

下面就是数组计算的测试：

```go
//evalutor/evalutor_test.go
func TestArrayLiterals(t *testing.T) {
    input := `[1, 2*2, 3+3]`
    evaluated := testEval(input)
    result, ok := evaluated.(*object.Array)
    if !ok {
        t.Fatalf("object is not Array, got=%T(%v)",
            evaluated, evaluated)
    }
    if len(result.Elements) != 3 {
        t.Fatalf("array has wrong num of elements. got=%d",
            len(result.Elements))
    }
    testDecimalObject(t, result.Elements[0], 1)
    testDecimalObject(t, result.Elements[1], 4)
    testDecimalObject(t, result.Elements[2], 6)
}
```

我们可以重用已经存在的代码让测试通过，就像我们在解析器中所做的。我们可以重用先前函数调用部分代码，也就是增加分支用来执行 `*ast.ArrayLiteral` 来生成数组对象:

```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.ArrayLiteral:
        elements := evalExpression(node.Elements, env)
        if len(elements) == 1 && isError(elements[0]) {
            return elements[0]
        }
        return &object.Array{Elements: elements}
}
```

测试通过，并且我们可以在REPL中使用字面数组并且生成数组：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> [1, 2, 3, 4]
[1, 2, 3, 4]
>> let double = fn(x) { x  * 2};
>> [1, double(2), 3 * 3, 4 - 3]
[1, 2, 9, 1]
```

是不是很神奇，但是我们现在还不能通过索引表达式访问单个元素。

## 执行索引表达式

好消息是解析索引表达式比执行索引表达式难多了，我们已经完成解析部分。唯一问题是可能出现移位错误在访问数组中的元素。因此我们在测试集中增加一些测试：

```go
// evalutor/evalutor_test.go
func TestArrayIndexExpression(t *testing.T) {
    tests := []struct {
        input    string
        expected interface{}
    }{
        {
            "[1,2,3][0]",
            1,
        },
        {
            "[1,2,3][1]",
            2,
        },
        {
            "[1,2,3][2]",
            3,
        },
        {
            "let i =0; [1][i]",
            1,
        },
        {
            "let myArray=[1,2,3];myArray[2];",
            3,
        },
        {
            "let myArray=[1,2,3];myArray[0]+myArray[1]+myArray[2]",
            6,
        },
        {
            "let myArray=[1,2,3];let i = myArray[0]; myArray[i]",
            2,
        },
        {
            "[1,2,3][3]",
            nil,
        },
        {
            "[1,2,3][-1]",
            nil,
        },
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        integer, ok := tt.expected.(int)
        if ok {
            testDecimalObject(t, evaluated, int64(integer))
        } else {
            testNullObject(t, evaluated)
        }
    }
}
```

要承认的是这些测试有点多，好多测试我们在其他地方已经测试完毕。但是这些测试非常容易编写，也非常可读。特别注意一下这些测试所期望的行为，它包含了我们先前没有考虑过得内容，当索引的的大小超出数组的边界，我们将会返回 `NULL`。其他一些语言会生成一个错误或者返回一个`null`，我们这里选择`null`。

正如我们期望的，测试无法通过。那么我们该如何修复和执行索引表达式呢？正如先前看到，索引操作符的左边操作数可以是任何表达式，而索引表达式也也可以是任何表达式。这就意味着我们在执行中缀表达式之前要先执行两边表达的表达式，否则我们直接访问标识符或者函数的标识符将无法工作。

在这我们增加一个`*ast.IndexExpression`分支，来调用`Eval`函数。

```go
// evalutor/evalutor/go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.IndexExpression:
        left := Eval(node.Left, env)
        if isEror(left){
            return left
        }
        index := Eval(node.Index, env)
        is isError(index){
            return index
        }
        return evalIndexExpression(left, index)
// [...]
}
```

接下来是 `evalIndexExpression` 函数

```go
// evalutor/evalutor.go
func evalIndexExpression(left, index object.Object) object.Object{
    switch {
    case left.Type() == object.Array_OBJ && index.Type() == object.INTEGER_OBJ:
        return evalArrayIndexExpression(left, index)
    default:
        return newError("index operator not support: %s", left.Type())
    }
}
```

使用 `if` 条件语句足够了，但是在下一章节我们将会增加其他 `case` 分支。除了错误处理，最重要是 `evalArrayIndexExpression` 函数。

```go
func evalArrayIndexExpression(array, index object.Object) object.Object {
    arrayObject := array.(*object.Array)
    idx := index.(*object.Integer).Value
    max := int64(len(arrayObject.Element) - 1)
    if id<0 || idx > max {
        return NULL
    }
    return arrayObject.Element[idx]
}
```

在这我们从数组中按照给定的索引获取相应的元素， 除此之外一些类型判断和类型转换也非常直接，它检查给定的索引值是否在给定范围内，如果超出范围返回 `NULL`，正如我们在测试中给出结果一样， 现在测试通过了。

```shell
$ go test ./evalutor
ok monkey/evalutor 0.007s
```

`REPL` 也如期工作：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let a = [1, 2 * 2, 10 - 5, 8 / 2]
>> a[0]
1
>> a[1]
4
>> a[5 - 3]
5
>> a[99]
null
```

## 为数组增加内置函数

现在可以通过数组字面值构建数组，也可以通过中缀表达式访问数组中的元素。以上两个功能可以使得数组非常有用，但是为了让数组更加有用，我们需要增加一些内置函数来使他们变得更加方便，这一小节就做这样的事。

在这一小节中将不会增加任何测试代码，因为这些特定的测试将会占据空间并且不会增加任何新的东西。为内置函数测试的框架已经包含在 `TestBuiltFunctions`，仅仅需要在已有的框架中增加测试方法，你就会发现它们完美适应这些内容。

我们的目标是增加新的内置函数，但是首先要做的事不是增加它们，而是修改已有的代码。我们需要 `len` 函数支持数组操作，目前它只支持字符串类型。

```go
//evaluator/builtins.go
var builtins = map[string]*object.Builtin{
    "len": {
        Fn: func(args ...object.Object) object.Object {
            if len(args) != 1 {
                return newError("wrong number of arguments. got=%d, want=1",
                    len(args))
            }
            switch arg := args[0].(type) {
            case *object.String:
                return &object.Integer{Value: int64(len(arg.Value))}
            case *object.Array:
                return &object.Integer{Value: int64(len(arg.Elements))}
            default:
                return newError("argument to `len` not supported, got=%s",
                    args[0].Type())
            }
        },
    },
//[...]
}
```

唯一的变化就是增加的`*object.Array`分支，接下来增加其他新的函数。

首先要增加的内置函数是 `first`，它返回数组的第一个元素。同样你也可以调用`myArray[0]` 达到同样的效果，但是 `frist` 相当简洁，接下来就是其实现：

```go
//evalutor/builtins.go
var builtins = map[string]*object.Builtin{
//[...]
    "first":&object.Builtin{
        Fn: func(args ...object.Object) object.Object{
            if len(args) != 1{
                return newError("wrong number of arguments. got=%d, want=1", len(args))
            }
            if args[0].Type() != object.ARRAY_OBJ {
                return newERror("argument to 'frist' must be ARRAY, got %s", args[0].Type())
            }
            arr := args[0].(*object.Array)
            if len(arr.Elements) > 0 {
                return arr.Elements[0]
            }
            return NULL
        },
    },
}
```

棒极了，那么接下来是什么呢？是的，我们接下来将要增加的的是`last`。`last`函数的目的是返回数组的最后一个元素，使用索引操作符是 `myArray[len(myArray) - 1]`，正如其表达的一样，实现 `last` 函数并不困难，跟`first`类似。

```go
//evalutro/builtin.go
var builtins = map[string]*object.Builtin{
"last": {
        Fn: func(args ...object.Object) object.Object {
            if len(args) != 1 {
                return newError("wrong number of arguments. got=%d, want=1",
                    len(args))
            }
            if args[0].Type() != object.ARRAY_OBJ {
                return newError("argument to `last` must be ARRAY, got %s",
                    args[0].Type())
            }
            arr := args[0].(*object.Array)
            length := len(arr.Elements)
            if length > 0 {
                return arr.Elements[length-1]
            }
            return NULL
        },
    },
}
```

接下来我们将要增加类似 `Scheme` 语言中的 `cdr` 函数，在其他语言中同样也叫做 `tail` 函数，在这里我们将叫做 `rest`。该函数返回新的数组包含其传入的参数的数组，除了第一个元素。使用起来如下所示：

```monkey
>> let a = [1, 2, 3, 4];
>> rest(a)
[2, 3, 4]
>> rest(rest(a))
[3, 4]
>> rest(rest(rest(a)))
[4]
>> rest(rest(rest(rest(a))))
[]
>> rest(rest(rest(rest(rest(a)))))
null
```

实现非常简单，但是要记住我们返回的是新分配的数组，我们并没有修改传入的数组。

```go
//evalutor/builitins.go
var builtins = map[string]*object.Builtin{
// [...]
    "rest": {
        Fn: func(args ...object.Object) object.Object {
            if len(args) != 1 {
                return newError("wrong number of arguments. got=%d, want=1",
                    len(args))
            }
            if args[0].Type() != object.ARRAY_OBJ {
                return newError("argument to `rest` must be ARRAY, got=%s",
                    args[0].Type())
            }
            arr := args[0].(*object.Array)
            length := len(arr.Elements)
            if length > 0 {
                newElements := make([]object.Object, length-1, length-1)
                copy(newElements, arr.Elements[1:length])
                return &object.Array{Elements: newElements}
            }
            return NULL

        },
    },
}
```

接下来为数组增加的内置函数是 `push`, 它在一个数组的后面增加一个新的元素，这里同样我们并不修改传入的数组，而是重新分配同样元素的数组，并且增加新增加的元素。在 `Monkey` 中，数组是不可修改的，下面是其实现操作：

```monkey
>> let a = [1, 2, 3, 4]
>> let b = push(a, 5)
>> a
[1, 2, 3, 4]
>> b
[1, 2, 3, 4, 5]
```

接下来就是实现：

```go
//evalutor/builtins.go
var builtins = map[string]*object.Builtin{
//[...]
"push": {
        Fn: func(args ...object.Object) object.Object {
            if len(args) != 2 {
                return newError("wrong number of arguments. got=%d, want=1",
                    len(args))
            }
            if args[0].Type() != object.ARRAY_OBJ {
                return newError("argument to `push` must be ARRAY, got=%s",
                    args[0].Type())
            }
            arr := args[0].(*object.Array)
            length := len(arr.Elements)
            newElements := make([]object.Object, length+1, length+1)
            copy(newElements, arr.Elements)
            newElements[length] = args[1]
            return &object.Array{Elements: newElements}
        },
    },
}
```

## 测试驱动数组

现在我们有数组，索引表达式和一些和数组一同工作的内置函数。是适合做一些工作，看看我我么你现在能做什么。

通过 `first`，`rest` 和 `push` 函数我们可以构建 `map` 函数

```monkey
let map =fn(arr, f){
    let iter = fn(arr, accumlated){
        if (len(arr) == 0) {
            return accumleated
        }else{
            iter(rest(arr), push(accumate, f(first(arr))));
        }
    }
    iter(arr, []);
}
```

通过 `map` 我们可以这样操作

```monkey
>> let a= [1, 2, 3, 4];
>> let double = fn(x){ x + 2 };
>> map(a, double);
[2, 4, 6, 8]
```

基于内置函数我们同样可以定义`reduce`函数：

```monkey
let d=reduce = fn(arr, initial, f){
    let iter = fn(arra, result){
        if (len(arr) == 0){
            result
        }else{
            iter(rest(arr), f(result, first(arr)));
        }
    };
    iter(arr, initial)
}
```

基于`reduce`， 同样我们也可以定义`sum`函数

```monkey
let sum = fn(arr){
    reduce(arr, 0, fn(initial, el){ initial + el });
}
```

同样可以这样工作：

```monkey
>> sum([1, 2, 3, 4, 5])
15
```
