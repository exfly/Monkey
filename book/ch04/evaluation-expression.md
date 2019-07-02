# 计算表达式

开始编写我们的 `Eval` 函数，现在已经拥有了抽象语法树和崭新的对象系统，这些帮助我们记录在执行 `Monkey` 代码时候得到的结果，是时候考虑计算抽象语法树。

这些 `Eval` 函数签名第一版：

```go
func Eval(node ast.Node) object.Object
```

函数以 `ast.Node` 作为输入，返回一个 `object.Object` 对象，要注意的是在 `ast` 包中的定义的每个节点都实现了 `ast.Node` 接口，因此它们都可以传递个 `Eval` 函数，这也保证了在计算抽象语法树的时候递归调用自己。抽象语法树的节点需要不同形式的计算方法，而 `Eval` 内部决定如何去判别这些形式。举个例子，当传递一个 `*ast.Program` 节点到 `Eval`，那么 `Eval` 应该去计算每个 `*ast.Program.Statements`，通过自身调用每个语句，返回值就是 `Eval` 中最后一个返回值。

我们以实现自计算表达式开始，也就是字面计算，简单来讲就是布尔型和整数型。它们是 `Monkey` 语言的基础，所以非常容易计算。如果我在 `REPL` 中输入 `5`，就输出 `5`；如果我输出 `true`，那么我将得到 `true`。

## Integer Literals

在开始写代码之前，想想它究竟需要完成什么工作？这里我们将一个表达式语句作为输入，它只有一个整型字面值，然后将它计算出来并返回。转换成 `Monkey` 语言就是：提供一个 `*ast.IntegerLiteral`，然后 `Eval` 函数应该返回一个 `*object.Integer` 对象，该对象包含一个 `Value` 字段，而且该值等于 `*ast.IntegerLiteral.Value` 中整型值。

现在为 `evaluator` 包写出单元测试框架

```go
// evaluator/evalutator_test.go

import (
    "monkey/lexer"
    "monkey/object"
    "monkey/parser"
    "testing"
)
func TestEvalIntegerExpression(t *testing.T){
    tests := [] struct {
        input string
        expected int64
    }{
        {"5", 5},
        {"10", 10},
    }
    for _, tt:=range tests{
        evaluated := testEval(tt.input)
        testIntegerObject(t, evaluated, tt.expected)
    }
}
func testEval(input string) object.Object {
    l := lexer.New(input)
    p := parser.New(l)
    program:=p.ParseProgram()
    return Eval(program)
}

func testIntegerObject(t *testing.T, obj object.Object, expected int64) bool{
    result, ok := obj.(*object.Integer)
    if !ok {
        t.Errorf("object is not Integer, got=%T (%+v)", obj, obj)
        return false
    }
    if result.Value != expected {
        t.Errorf("object has wrong value. got=%d, want=%d", result.VAlue, expected)
        return false
    }
}
```

现在单元测试肯定不能通过，因为 `Eval` 函数返回是 `nil` 而不是 `*object.Integer`。在 `Eval` 中没有遍历整个抽象语法树，我们应当从树的顶端开始，接受一个 `*ast.Program`，然后递归遍历每一个节点。

```go
// evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
    switch node := node.(type){

        //Statements
        case *ast.Program:
            return evalStatements(node.Statements)
        case *ast.ExpressionStatement:
            return Eval(node.Expression)

        //Expressions
        case *ast.IntegerLiteral:
            return &object.Integer{Value: node.Value}
    }
    return nil
}

func evalStatements(stmts []ast.Statement) object.Object {
    var result object.Object
    for _, statements := range stmt {
        result = Eval(statements)
    }
    return result
}
```

在 `Monkey` 程序中计算每一个语句，如果语句是一个 `*ast.ExpressionStatement`，我们计算它的表达式，它反映是从一行输入如 `5` 的抽象语法树，它是只包含语句的程序。

```shell
$ go test ./evalutor
ok  monkey/evalutor 0.006s
```

现在测试铜鼓哦，我们可以计算整型字面值，虽然这个和我想象中还不是很像，不过这仅仅简单的开始。接下来拓展我们的计算器，`Eval` 结构不变，我们仅仅是增加和拓展它。

在完成布尔型之前，先完成 `REPL` 功能。

## 完成 REPL

到现在为止，我们的 `REPL` 中的 `E` 是缺失的，而且我们也仅仅只是完成了 `RPPL(Read-Parse-Print-Loop)`，这一章节中将会实现 `Eval`，这将是构建真正的 `REPL`。

```go
//repl/repl.go
import (
// [...]
    "monkey/evaluator"
)
//[...]
func Start(in io.Reader, out io.Writer){
    scanner := buffio.NewScanner(in)
    for {
        fmt.Printf(PROMPT)
        scanned := scanner.Scan()
        if !scanned {
            return
        }
        line := scanner.Text()
        l := lexer.New(line)
        p := parser.New(l)
        program := p.ParseProgram()
        if len(p.Erros()) != 0 {
            printParseErrors(out, p.Error()
            continue
        }
        evaluated := evalutor.Eval(program)
        if evaluated != nil {
            io.WriteString(out, evaluated.Inspect())
            io.WriteString(out, "\n")
        }
    }
}
```

将 `program` 传递给 `Eval` 方法，如果返回返回值非空，也就是 `*ast.Object` 对象，那么就调用 `Inspect` 方法将其输出，比如 `*object.Integer` 输出的内容就是封装整数值的字符串。使用 `REPL` 可以这样工作：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>>5
5
>>10
10
>>999
999
```

## 布尔型字面值

布尔字面值跟我们刚刚遇到的整型一样，`true` 计算返回 `true`，`false` 计算返回 `false`。在 `Eval` 中实现也是非常简单，下面是单元测试：

```go
// evaluator/evaluator_test.go
func TestEvalBooleanExpression(t *testing.T) {
    tests := []struct {
        input string
        expected bool
    }{
        {"true", true},
        {"false", false},
    }
    for _, tt := range tests {
        evaluated := testEval(tt.intput)
        testBooleanObject(t, evaluated, tt.expected)
    }
}
func testBooleanObject(t *testing.T, obj object.Object, expected bool) bool {
    result, ok := obj.(*object.Boolean)
    if !ok {
        t.Errorf("object has wrong value. got=%T(%+V)", obj, obj)
        return false
    }
    if result.Value != expected {
        t.Errorf("object has wrong value. got=%t, want=%t", result.Value, expected)
        return false
    }
    return true
}
```

以后我们会拓展 `tests` 切片以便支持更多的表达式而不仅仅是布尔型，目前只要保证输入 `true` 和 `false` 能够得到正确的结果即可。

目前单元还不能通过，为了测试通过只需要将 `*ast.IntegerLiteral` 分支拷贝过来并做一些简单的修改

```go
//evaluator/evalutor.go
func Eval(node ast.Node) object.Object {
//[...]
    case *ast.Boolean:
        return &object.Boolean{Value: node.Value}
}
```

让我们看看 `REPL` 是如何工作的：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> true
true
>> false
false
>>
```

现在没有问题，但是我们在每个 `true` 或者 `false` 时候都创建一个 `object.Boolean` 对象，这个有点不对劲。两个 `true` 或者 `false` 内部并没有什么不同，但是我们为什么每次都用新的实例对象呢？这里只有两个不同的值，每次引用不同的值即可，而不是新建一个。

```go
// evaluator/evalutor.go
var (
    TRUE = &object.Boolean{Value: true}
    FALSE = &object.Boolean{Value: false}
)
func Eval(node ast.Node) object.Object {
// [...]
    case *ast.Boolean:
        return nativeBoolToBooleanObject(node.Value)
// [...]
}
func nativeBoolToBooleanObject(input bool) *object.Boolean {
    if input{
        return TRUE
    }
    return FALSE
}
```

现在包中只有两个 `object.Boolean` 对象实例：`TRUE` 和 `FALSE`，我们直接引用它们而不是申请空间去创建它们，这个对我们性能上有小小的提高，同样在 `Null` 类型也是同样处理的。

## Null

就跟布尔型 `true` 和 `false` 只有一个一样，对于 `null` 类型也应该只有一个，没有其他空类型的变种，一个对象要么为空要么不为空。所以我们首先创建一个 `NULL` 对象，以便在整个计算过程中都能应用它。

```go
// evaluator/evaluator.go
var (
    NULL = &object.NUll{}
    TRUE = &object.Boolean{Value:true}
    FALSE = &object.Boolean{Value:false}
)
```

现在有了整数字面值和三个 `NULL`， `TRUE` 和 `FALSE`，就可以开始准备计算表达式了。

## 前缀表达式

在 `Monkey` 中最简单的操作符表达式是前缀表达式，即单操作数表达式，也就是操作数紧跟在操作符之后。在先前的解析过程总表示过，很多语言喜欢采用前缀表达式，因为解析它们很简单。`Monkey` 语言支持两种前缀操作符：`!` 和 `-`。

计算操作符表达式并不难，我们一点点实现构建我们设计出来的行为，但是要注意的是，我们想要达到的目标远远超出我们的预期。

首先开始实现支持 `!` 操作符，下面是测试用例：

```go
// evaluator/evalutor_test.go

func TestBangOperator(t *testing.T){
    tests := []struct{
        input string
        expected bool
    }{
        {"!true", false},
        {"!false", true},
        {"!5", false},
        {"!!true", true},
        {"!!false", false},
        {"!!5", true}
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        testBooleanObjec(t, "evaluted", tt.expected)
    }
}
```

`!true` 和 `!false` 表达式和它们期望值一样合理，但是 `!5` 其他语言设计觉得返回一个错误，但是我想说的是 `5` 行为上表现就是 `truthy`。

这个测试肯定不能通过，因为 `Eval` 返回一个 `nil` 而不是 `TRUE` 或者 `FALSE`，计算前缀表达式第一个就是计算它的操作数，然后用它的操作符来计算结果。

```go
// evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
//[...]
    case *ast.PrefixExpression:
        right :=Eval(node.Right)
        return evalPrefix(node.Operator, right)
}
```

在第一步调用后，右边的值可能是 `*object.Integer` 后者 `*object.Boolean` 甚至是一个 `NULL` 。然后按照右边的操作数传递到 `evalPrefixExpression` 函数中，用来检查这个操作符是否支持。

```go
// evalutor/evaluator.go
func evalPrefixExpression(operator string, right object.Object)object.Object{
    switch operator{
        case "!":
            return evalBangOperatorExpression(right)
        default:
            return NULL
    }
}
```

如果操作符不支持就返回 `NULL`，这是最好的选择吗？这个还不确定，不过目前来看是最好的方式。`evalBangOperatorExpression` 函数即是 `!` 操作符的具体实现。

```go
func evalBangOperatorExpression(right object.Object) object.Object {
    switch right {
    case TRUE:
        return FALSE
    case FALSE:
        return TRUE
    case NULL:
        return TRUE
    default:
        return FALSE
    }
}
```

现在所有的测试全部通过

```shell
$ go test ./evalutor
ok  monkey/evaluator 0.007s
```

接下来继续 `-` 前缀操作符，我们可以拓展 `TestEvalIntegerExpression` 测试函数以增加测试用例：

```go
//evalutor/evalutor_test.go
func TestEvalIntegerExpression(t *testing.T){
    tests := []struct {
        input string
        expected int64
    }{
        {"5", 5},
        {"10", 10},
        {"-5", -5},
        {"-10", -10},
    }
// [...]
}
```

为了测试 `-` 前缀表达式，我们选择拓展测试而不是重新编写测试函数，主要原因有两个：一是整型是唯一支持 `-` 操作符的前缀操作数；二是测试方法应该包含所有整型的运算方法以便达到清晰的目的。我们已经提前拓展了 `evalPrefixExpression` 方法以便让测试通过，只需要在 `switch` 语句下添加新的分支。

```go
// evaluator/evalutor.go
func evalPrefixExpression(operator string, right object.Object) object.Object {
    switch operator{
    case "!":
        return evalBangOperatorExpression(right)
    case "-":
        return evalMinusPrefixOperatorExpression(right)
    default:
        return NULL
    }
}
```

`evalMinusPrefixOperatorExpresion` 函数实现如下：

```go
// evalutor/evalutor.go
func evalMinusPrefixOperatorExpression(right object.Object)object.Object {
    if right.Type() != object.INTEGER_OBJ {
        return NULL
    }
    val := right.(*object.Integer).Value
    return &object.Integer{Value : -value}
}
```

首先检查操作数是否为正数，如果不是返回 `NULL`；如果是，我们提取 `*object.Integer` 中的值，然后重新分配一个对象封装它的相反值。

现在测试通过了：

```shell
$ go test ./evaluator
ok  monkey/evalutor 0.0007s
```

在继续中缀表达式之前，我们可以在 `REPL` 中计算出前缀表达式的值：

```shell
$ go run main.go
Hello mrnugget! This is the Monkey programming language！
Feel free to tyoe in comamnds
>> -5
5
>> !true
false
>> !-5
false
>> !! -5
true
>> !!!! -5
true
>> -true
null
```

## 中缀表达式

下面是 `Monkey` 语言支持的八种中缀表达式

```monkey
5 + 5;
5 - 5;
5 * 5;
5 / 5;

5 > 5;
5 < 5;
5 == 5;
5 != 5;
```

这八个操作符可以分为两组，一是操作符生成布尔型值结果；另一种是生成另外的值。我们开始实现第二组操作符 `+, -, *, /`。开始只支持整数操作数，只要它能工作，我们就可以开始实现生成布尔型操作符。

测试框架已经就绪，只需要拓展 `TestEvalIntegerExpression` 函数即可适应新的操作符。

```go
//evalutor/evalutor_test.go
func TestEvalIntegerExpression(t *testing.T){
    tests :=[] struct {
        input string
        expected int64
    }{
       {"5", 5},
        {"10", 10},
        {"5", 5},
        {"10", 10},
        {"-5", -5},
        {"-10", -10},
        {"5+5+5+5-10", 10},
        {"2*2*2*2*2", 32},
        {"-50+100+ -50", 0},
        {"5*2+10", 20},
        {"5+2*10", 25},
        {"20 + 2 * -10", 0},
        {"50/2 * 2 +10", 60},
        {"2*(5+10)", 30},
        {"3*3*3+10", 37},
        {"3*(3*3)+10", 37},
        {"(5+10*2+15/3)*2+-10", 50},
    }
//[...]
}
```

为了通过这些测试，首先要做的事拓展 `Eval` 函数中的 `switch` 语句：

```go
//evaluator/evalutor.go
func Eval(node ast.Node) objet.Object{
// [...]
    case *ast.InfixExpression:
        left := Eval(node.Left)
        right := Eval(node.Right)
        return evalInfixExpression(node.operator, left, right)
// [...]
```

就跟 `*ast.PrefixExpression` 一样，我们首先计算操作数。现在我们有两个操作数，左右两边各一个抽象语法树节点。我们已经知道这可能是表达式、函数调用、整型字面值或者操作符表达式等等。这个我们不需要关心，这个有 `Eval` 函数负责。在计算玩操作数后，我们将两个返回值和操作符传递到 `evalInfixExpression` 函数中：

```go
func evalInfixExpression(
    operator string,
    left, right object.Object,
)object.Object {
    switch {
    case left.Type()==object.INTEGER_OBJ && right.Type()==object.INTEGER_OBJ:
        return evalIntegerInfixExpression(operator,left,right)
    default:
        return NULL
    }
}
```

一旦两边的操作数不是整数，就返回 `NULL`，当然后面会拓展这个函数，但是为了测试通过足够了。重点在于 `evalIntegerInfixExpression` 函数，该函数中，我们疯转了 `*object.Integers` 的操作加、减、乘和除。

```go
// evalutor/evalutor.go
func evalIntegerInfixExpression(
    operator string
    left, right object.Object,
)object.Object{
    leftVal := left.(*object.Integer).Value
    rightVal := right.(*object.Integer).Value
    switch operator{
    case "+":
        return &object.Integer{Value:leftVal+rightVal}
    case "-":
        return &object.Integer{Value:leftVal-rightVal}
    case "*":
        return &object.Integer{Value:leftVal*rightVal}
    case "/":
        return &object.Integer{Value:leftVal/rightVal}
    default:
        return NULL
    }
}
```

现在测试通过了：

```shell
$ go test ./evalutor
ok monkey/evalutor 0.007s
```

接下来拓展我们的 `TestEvalBooleanExpression` 函数，为上述操作符增加新的测试用例，因为它们都能生成布尔型数值。

```go
//evaluator/evaluator_test.go
func TestEvalBooleanExpression(t *testing.T){
    tests := []struct {
        input string
        expected bool
    }{
        {"true", true},
        {"false", false},
        {"1<2", true},
        {"1>2", false},
        {"1<1", false},
        {"1>1", false},
        {"1==1", true},
        {"1!=1", false},
        {"1==2", false},
        {"1!=2", true},
    }
}
```

除此之外，还需要在 `evalIntegerInfixExpression` 函数增加一些代码，它们能够保证上述的单元测试能够通过。

```go
// evaluator/evaluator.go
func evalIntegerInfixExpression(
    operator string
    left, right object.Object,
) object.Object {
    leftVal := right.(*object.Integer).Value
    rightVal := right.(*obejct.Integer).Value
    switch operator {
// [...]
    case "<":
        return nativeBoolToBooleanObject(leftVal < rightVal)
    case ">":
        return nativeBoolToBooleanObject(leftVal > rightVal)
    case "==":
        return nativeBoolToBooleanObject(leftVal == rightVal)
    case "!=":
        return nativeBoolToBooleanObject(leftVal != rightVal)
    default:
        return NULL
    }
}
```

`nativeBoolToBoolean` 函数在布尔型字面值的是由已经使用，现在在比较为封装的值的时候又重用它们。至少对于整数而言，我们现在已经完全支持八种中缀操作符，剩下的工作就是支持布尔型操作数。

`Monkey` 语言支持布尔型操作数的相等判断 `==` 和 `!=`，它不支持布尔型数值的加减乘除，这样减少的开发的工作量。首先要做的是增加测试内容，跟以前一样，拓展已有的测试方法：

```go
func TestEvalBooleanExpression(t *testing.T){
    tests := []struct{
        input string
        expected bool
    }{
// [...]
        {"true == true", true},
        {"false == false", true},
        {"true == false", false},
        {"true != false", true},
        {"(1<2)==true", true},
        {"(1<2) == false", false},
        {"(1>2) == true", false},
        {"(1>2)==false", true},
    }
// [...]
}
```

毫无疑问，目前的单元测试无法通过，我们需要增加一些额外的判断来让测试通过。

```go
//evalutor/evalutor.go
func evalInfixExpression(
    opertor string 
    left, right object.Obejct,
)object.Object {
    switch {
// [...]
    case operator == "==":
        return nativeBoolToBooleanObject(left == right)
    case oeprator == "!=":
        return nativeBoolToBooleanObject(left != right)
    default:
        return NULL
    }
}
```

我们只是在 `evalInfixExpression` 函数中增加四行代码，测试通过了。通过指针比较两个布尔型值是否相等。这样做的原因是我们所有的布尔型指针都指向了 `TRUE` 和 `FALSE`，对于 `NULL` 也是同样的道理。这个对于整型和其他数据类型并不奏效，因为对于 `*object.Integer` 我们每次都分配内存生成 `object.Integer`，所以每次的指针都是不一样的，否则像 `5==5` 就是 `false`，这个与事实不符。

现在我们已经完成得差不多了，让我们看看现在解释器能够做什么！

```shell
$go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> 5 * 5 + 10
35
>> 3 + 4 * 5 == 3 * 1 + 4 * 5
true
>> 5 * 10 > 40 + 5
true
>> (5 > 5 == true) != false
false
>> 500 / 2 != 250
false
```

到目前为止，我们已经完成了一个函数计算器，接下来让我们继续完成功能，使它变得更像一个编程语言。
