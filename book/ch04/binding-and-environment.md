# 4.9 绑定和环境

接下来要做的是增加绑定来支持 `let` 语句，但是我们不仅仅需要支持 `let` 语句，还要执行变量的执行，以下面的例子来讲：

```monkey
let x = 5 * 5;
```

我们需要解释器在执行 `x` 的时候得到的结果是 `10`。所以本小节的任务是在执行 `let` 语句的时候将右边计算得到的结果保存到具体名称的变量中，在执行变量的时候首先要查看该变量是否已经绑定到特定值，如果已经绑定，则执行该值，否则返回一个错误。听上去去不错，开始一些测试用例：

```go
func TestStatement(t *testing.T){
    tests := []struct {
        input string
        expected int64
    }{
        {"let a = 4; a ;" , 5},
        {"let a = 5 * 5; ", 25},
        {"let a = 5; let b = a; b;", 5},
        {"let a = 5; let b = a; let c = a + b; c;", 15},
    }
    for _, tt := range tests {
        testIntegerObject(t, testEval(tt.input), tt.expected)
    }
}
```

这些测试主要做了两件事情：一是执行 `let` 语句中那些可以生成值的表达式；二是执行那些绑定值的变量。但是我们也需要一些测试来确保执行那些未绑定的值能够生成正确的错误，因此先拓展已经存在的 `TestErrorHandling` 函数。

```go
//evalutor/evaluator_test.go
func TestErrorHanding(t *testing.T){
    tests := []struct {
        input string
        expectedMessage string
    }{
        {
            "footbar",
            "Identifier not found: foobar",
        },
    }
// [....]
}
```

但是如何让测试通过能？显然要在 `Eval` 函数中增加一个分支来支持 `*ast.LetStatement`，那么要怎么正确处理这个分支呢？

```go
//evalutor/evaluator.go
func Eval(node ast.Node) object.Object {
// [...]
    case *ast.LetStatement:
        val := Eval(node.Value)
        if isError(val){
            return val
        }
    //Huh? Now what?
}
```

该如何记录这个值呢？我们要一个值将它要绑定的变量，如何将其中的一个绑定到另一个中呢？这就考虑所谓的环境大显身手的手，这个概念来自于 `Lisp` 家族语言，尽管这个名称听上去很复杂，但是核心只有一个哈希表而已，用字符串和对象关联起来，下面我们就去实现它。

将 `Environment` 结构添加到 `object` 包中，这个结构只用一个 `map` 结构。

```go
package object
func NewEnvironment() *Environment{
    s := make(map[string]Object)
    return &Environment{store: s}
}
type Environment struct {
    store map[string]Oject
}
func (e *Environemt) Get(name string) (Object, bool) {
    obj, ok := s.store[name]
    return obj, ok
}
func (e *Environemt) Set(name strong, val Oject)object {
    s.store[name] = val
    return val
}
```

为什么我们不直接使用 `map` 结构，而是要做一层封装？到后面函数实现和函数调用就明白，这是接下来工作的基础。那么现在如何在 `Eval` 函数中如何去使用这个 `object.Environment` 对象呢？或者说如何去记录这个环境呢？这里是将 `Eval` 函数中当成一个函数参数传递过去。

```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) objec.Object {
//[...]
}
```

除此之外，其余的不需要做改变，还有其他所有调用 `Eval` 函数以便增加环境使用。不仅仅 `Eval` 函数内部调用 `Eval` 函数，还有其他一些比如 `evalProgram`, `evalIfExpression` 等调用 `Eval` 的函数。另外 `REPL` 也调用 `Eval` 函数，因此也需要增加环境，当然这里只需要一个环境值即可。

```go
//repl/repl.go
func start(in io.Reader, out io.Writer){
    scanner := bufio.NewScanner(in)
    env := object.NewEnvironment()
    for {
// [...]
        evaluated := evaluator.Eval(program, env)
        if evaluated != nil {
            io.WriteString(out, evaluted.Inspect())
            io.WriteString(out, "\n")
        }
    }
}
```

如果不这样做，那么在 `REPL` 中绑定的变量将不会起任何作用，在下一行代码中，相关的绑定的值就不复存在。同样在测试环境中也是同样如此，我们不想在每个测试函数用例中保存状态，每次调用 `testEval` 就会有新的环境，主要目的是避免全局变量导致的 `bug`，在这每个调用 `Eval` 就会得到崭新的环境。

```go
//evalutor/evaluator_test.go
func testEval(input string) object.Object {
    l := lexer.New(input)
    p := parser.New(l)
    program := p.ParseProgram()
    env := object.NewEnviroment()
    return Eval(program, env)
}
```

更新 `Eval` 函数调用后，需要让测试通过，只需要在 `*ast.LetStatement` 分支中，将变量名称和值保存到当前环境中去：

```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.LetStatement:
        val := Eval(node.Value, env)
        if isError(val){
            return val
        }
        env.Set(node.Name.Value, val)
}
```

除此之外还需要在执行变量语句的时候从环境中取出该值：

```go
// evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.Identifier:
        return evalIdentifer(node, env)    
}
func EvalIdentifer(
    node *ast.Idenfier,
    env *object.Environment,
)object.Object {
    val, ok := env.Get(node.Value)
    if !ok {
        return newError("idenfier not found: " + node.Value)
    }
    return val
}
```

`evalIdentifier` 在下面会被拓展，目前可以简单的检查是否绑定到相应的名称中，如查找成功，则返回该值，否则生成一个错误。

```shell
$ go test ./evaluator
ok monkey/evaluator 0.007s
```

现在 `REPL` 也能完整的工作：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let a = 5;
>> let b = a > 3;
>> let c = a * 90;
>> if (b) { 10 } else { 1 };
10
>> let d = if ( c> a) {99}else {100}；
>> d
99
>> d * c * a
245025
```