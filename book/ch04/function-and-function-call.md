# 函数和函数调用

这一小节四我们一直努力的方向，这里我们的解释器将会增加对函数和函数调用的支持。当我们完成这这一小节，就能在 `REPL` 中完成如下的操作：

```shell
>> let add = fn(a, b, c, d) { return a + b + c + d};
>> add(1, 2, 3, 4)
10
>>let addThree = fn(x) { return x + 3 };
>>addThree(3);
6
>>let max = fn(x, y) { if (x>y) {x} else { y }};
>>max(5, 10)
10
>> let factorial = fn(n) { if (n==0) {1} else { n * factorial(n-1)}};
>>factorial(5)
120
```

如果上述的代码还不能吸引你， 那么看看下面的函数：传递函数、高阶函数和闭包同样有效。

```shell
>> let callTwoTimes = fn(x, func){ func(func(x))}
>> callTwoTimes(3, addThree);
9
>> let newAdder = fn(x) { fn(n) { x + n}};
>> let addTwo = newAddr(2);
>> addTwo(2);
4
```

是的，接下来就要把上述的功能全部全部实现。

从目前的已经完成的工作来讲，我们需要完成两件其他的事情：一是在对象系统中实现对函数表达的支持；另一个是就是在 `Eval` 函数中增加函数调用支持。首先要承认的是在 `Monkey` 语言中，函数和其它值一样：绑定到特定的变量上；在表达式中使用它们；传递到其他函数中；从其他函数中返回等等操作。因此需要在对象系统中表示出来，这样才能做到传递、赋值和返回值。

```go
//ast/ast.go
type FunctionLiteral struct {
    Token Token.Token // The 'fn' token
    Parameters []*Identifer
    Body *BlockStatement
}
```

貌似在函数对象中不需要 `Token` 字段，但是 `Parameter` 和 `Body` 是需要的，如果没有函数体的话，我们不能执行一个函数；如果没有形参的话，也不能执行函数体。

```go
// object/object.go
const (
//[...]
    FUNCTION_OBJ = "FUNCTION"
)
type Function struct {
    Parameters []*ast.Identifier
    Body *ast.BlockStatement
    Env *Environment
}
func (f *Function) Type() ObjectType { return FUNCTION_OBJ }
func (f *Function) Inspect() string {
    var out bytes.Buffer
    params := []string{}
    for _, p := range f.Parameters {
        params = append(params, p.String())
    }
    out.WriteString("fn")
    out.WriteString("(")
    out.WriteString(strings.Join(params, ", "))
    out.WriteString(") {\n")
    out.WriteString(f.Body.String())
    out.WriteString("\n}")
    return out.String()
}
```

`object.Funciton` 对象除了拥有 `Parameters` 和 `Body` 字段外，还有一个 `Env` 字段。它是指向 `object.Environment` 对象的指针。因为 `Monkey` 语言中包含函数包含当前环境，它支持闭包，也就是它能包含环境定义时候的变量，在后序的访问中也能获取到，因此 `Env` 字段意义重大。

首先是单元测试：

```go
// evaluator/evaluator_test.go
func TestFunctionObject(t *testing.T) {
    input := `fn(x) { x+2; };`
    evaluated := testEval(input)
    fn, ok := evaluated.(*object.Function)
    if !ok {
        t.Fatalf("object is not Function. got=%T(%+v)",
            evaluated, evaluated)
    }
    if len(fn.Parameters) != 1 {
        t.Fatalf("function has wrong parameters. Parameters=%+v",
            fn.Parameters)
    }
    if fn.Parameters[0].String() != "x" {
        t.Fatalf("parameter is not 'x'. got=%q", fn.Parameters[0])
    }
    expectedBody := `(x + 2)`
    if fn.Body.String() != expectedBody {
        t.Fatalf("body is not %q. got=%q", expectedBody, fn.Body)
    }
}
```

测试函数表明执行一个字面函数能否正确放回 `*object.Function` 对象，并且判断函数参数和函数都是正确；接下来测试是否包含正确环境，为了让测试通过在 `Eval` 函数中增加分支。

```go
// evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.FunctionLiteral:
        params := node.Parameters
        body := node.Body
        return &object.Function{Parameters: params, Env: env, Body: body}
//[...]
}
```

现在测试通过了，这里仅仅重用了 `Parameters` 和 `Body` 字段，并且注意到构建函数对象的时候也是使用当前环境。

接下来是函数调用测试：

```go
//evalutor/evalutor_test.go
func TestFunctionApplication(t *testing.T) {
    tests := []struct {
        input    string
        expected int64
    }{
        {"let identity=fn(x){x;}; identity(5);", 5},
        {"let identity=fn(x){return x;}; identity(5);", 5},
        {"let double=fn(x){x*2;}; double(5);", 10},
        {"let add = fn(x, y) { x+y;}; add(5,5);", 10},
        {"let add=fn(x,y){x+y;}; add(5+5, add(5,5));", 20},
        {"fn(x){x;}(5)", 5},
    }
    for _, tt := range tests {
        testDecimalObject(t, testEval(tt.input), tt.expected)
    }
}
```

这里每个测试都做了同样的事情：定义一个函数，使用参数调用它然后对其生成的值做判断。但是有一点不同的是有些返回值是隐式的，有的使用 `return` 语句，有的使用表达式做为参数，有的多个参数需要执行参数后才传递函数。

同样也需要测试 `*ast.CallExpression` 两种形式：一是函数是变量形式，通过变量来调用函数；另一种是字面函数调用。幸运的是这里两者并没有什么不同，我们已经知道如何去执行变量标识符和字面函数。

```go
// evaluator/evalutor.go
func Eval(node ast.Node, env *object.Environment) objct.Object {
//[...]
    case *ast.CallIfExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
}
```

是的，我们仅仅通过 `Eval` 获取我们需要调用的函数，无论是 `*ast.Identifier` 或者是 `*ast.FunctionLiteral`， `Eval` 都会返回一个 `*object.Function`。但是我们该如何去调用这个 `*object.Function` 呢？第一步是执行参数表达式是，原因非常简单：

```monkey
let add = fn(x, y) { x + y };
add(2 + 2, 5 + 5)
```

在这里将 `4` 和 `10` 作为参数传递个 `add` 函数，而不是 `2+2` 和 `5 +5`。获取参数就是执行一系列表达式，并且记录相应的结果，一旦解析过程发生错误，停止继续执行。

```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.CallExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
        args := evalExpression(node.Arguments, env)
        if len(args) == 1 && isError(args[0]) {
            return args[0]
        }
// [...]
}
func evalExpresion(
    exps []ast.Expression,
    env *object.Environment,
)[]object.Object {
    var result []object.Object
    for _, e := range exps {
        evaluated := Eval(e, env)
        if isError(evaluated) {
            return []object.Object{evaluated}
        }
        result = append(result, evaluated)
    }
    return result
}
```

这里仅仅是迭代 `ast.Expression` 列表并且根据上下文环境执行，如果遇到错误，则停止执行并返回错误。这里从左到右执行参数表达式。所以现在已经有了函数和一些列参数，我们该如何调用函数呢？如何将函数和这些参数联系在一起呢？

显然是需要执行函数体，函数体就是语句块，既然我们已经知道了如何执行语句块了，为什么不知直接调用 `Eval` 函数呢，将函数体传递过去就行了呢？答案就是参数，函数中包含函数参数引用，如果仅仅执行函数体使用的当前的环境，那么就会导致未知引用，从而带来错误，所以执行函数体时候使用当前的环境是没有作用的。

我们需要做的是在执行的时候修改环境变量，以便函数形参能够正确解析到对应的实参。而且也不是简单的将当前的实参插入到当前的环境中，这也不是我们希望的，我们希望表达如下：

```monkey
let i = 5;
let printNum = fn(i){
    puts(i);
};
printNum(10);
puts(i)
```

`puts` 函数功能输出一行内容，上述代码应该输出两行，分别是 `10` 和 `5`，如果直接将实参参入到环境，那么最后一行输出的结果将会是 `10`。所以这边我们需要做的不是保存先前的环境，而是拓展它。拓展环境也就意味着需要创建新的 `object.Environment` 实例，并指向它以便完成拓展，新的空环境包含指向当前的存在的环境。在新的环境中调用 `Get` 方法的时候，如果自身环境不存在该变量对应的名称，则调用它指向的环境，如此递归完成直到最外面的环境，如果还没有找到则返回错误。

```go
// object/environment.go
package object
func NewEncloseEnvironment(outer *Environment) *Environment{
    env := NewEnvironment()
    env.outer = outer
    return env
}
func NewEnvironemt() *Environment{
    s := make(map[string]Object)
    return &Environemnt{store: s, outer:nil}
}
type Environment struct {
    store map[string]Object
    outer *Environment
}
func (e *Environment) Get(name string) (Object, bool){
    obj, ok := e.store[name]
    if !ok && e.outer != nil {
        obj, ok = e.outer.Get(name)
    }
    return obj, ok
}
func (e *Environment) Set(name string, val Object) Object {
    e.store[name] = val
    return val
}
```

`object.Environment` 现在有新的字段 `outer`，它包含其他的 `object.Environment` 对象的引用，`NewEnclosedEnvironment` 函数创建一个包含环境，`Get` 方法也做出相应的修改。新的问题是如何考虑变量的作用域，这里分为内部作用域和外部作用域，在内部作用域中没有发现的变量就会去外部作用域寻找；外部作用域包含内部作用域，而内部作用域拓展外部作用域。

使用新的 `object.Environment`，函数就能正确执行函数体。更新后的 `Eval` 函数如下：

```go
// evaluator/evaluator.go
func Eval(ndoe ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.CallExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
        args := evalExpression(node.Arguments, env)
        if len(args) == 1 && isError(args[0]) {
            return args[0]
        }
        return applyFunction(function, args)
}
func extendFunctionEnv(
    fn *object.Function,
    args []object.Object,
)*object.Environment {
    env := object.NewEnclosedEnvironment(fn.Env)
    for parmIdx, para m := range fn.Parameters {
        env.Set(param.Value, args[paramIdx])
    }
    return env
}

func upwrapReturnValue(obj object.Object) object.Object {
    if returnValue, ok := obj.(*object.ReturnValue); ok {
        return returnValue.Value
    }
    return obj
}
```

在新的 `applyFunction` 函数中，不仅仅检查是否有 `*object.Function` 而且将 `fn` 参数转换为 `*object.Function` 的引用，以便获取函数的 `.Env` 和 `.Body` 两个字段。在 `extendFunctionEnv` 函数中，我们创建新的 `*object.Environment` 并且指向当前的环境，在新的环境中，它绑定函数调用的形参和实参。

新的环境才是函数体调用的使用的环境，这个返回结果被拆封，以免 `*object.ReturnValue` 被返回，这件事是必须的，否则 `return` 结果将会被上浮，导致整个结果被提前退出。但是仅仅需要函数体在最后一句停止执行，这也是为什么需要将结果拆封，所以 `evalBlockStatement` 不会停止执行函数外面的语句，同样在 `TestReturnStatements` 测试函数中增加了测试用来确保正确执行。

```shell
$ go test ./evaluator
ok monkey/evaluator 0.007s
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let addTwo = fn(x) { x + 2; };
>> addTwo(2)
4
>> let multiply = fn(x, y) { x * y };
>> multiply(50/2, 1 * 2)
50
>> fn(x) { x == 10 }(5)
false
>> fn(x) { x == 10}(10)
true
```

现在思考一个问题：为什么使用函数的环境而不是当前的环境？简单的例子如下：

```go
//evalutor/evalutor_test.go
func TestCloures(t *testing.T){
    intput := `
    let newAdder = fn(x) {
        fn(y) { x + y };
    };
    let addTwo = newAdder(2)
    addTwo(2);
    `
    testIntegerObject(t, testEval(input), 4)
}
```

测试通过：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let newAdder = fn(x) { fn(y) { x + y } };
>> let addTwo = newAdder(2);
>> addTwo(3);
5
>> let addThree = newAdder(3);
>> addThree(10);
13
```

`Moneky` 拥有闭包的特性，这个在解释器中工作得很好，是不是很酷？闭包是函数咋在定义的时候与其所在的环境关联起来，它携带的自身环境在函数调用的时候也能访问它们。比如下面：

```monkey
let newAdder = fn(x){ fn(y) { x + y}};
let addTwo = newAddr(2);
```

`newAdder` 是一个高阶函数，它返回一个函数或者接受一个函数作为参数的函数。在这个例子中 `newAdder` 返回其他的函数，但是它不是其他函数而是一个闭包。`addTwo` 绑定这个闭包，并且它是 `newAdder` 和一个 `2` 单独的实参。那什么导致 `addTwo` 变成一个闭包？答案就是它在调用时候绑定的内容。当 `addTwo` 被调用的时候，它不仅仅能够访问他的的实参 `y` 参数，而且能够访问它在创建的时候 `newAdder(2)` 的参数 `2`。尽管它绑定的参数已经离开他它的作用域，而且不在当前的环境中：

```monkey
>>let newAdder = fn(x){ fn(y) { x + y}};
>>let addTwo = newAdder(2);
>> x
ERROR: identifier not found: x
```

`x` 在顶级环境中也没有绑定到一个值，但是 `addTwo` 仍然能访问它：

```monkey
>> addTwo(3)
5
```

换句话说 `addTwo` 闭包仍然可以访问它在定义的时候的环境，也就是当`newAdder` 函数体最后一句执行的时候。最后一行是字面函数，当一个字面函数被创建的时候，构建一个`object.Function`，并且保存它一个指向当前环境指针的字段`.Env`。

当我们后来执行 `addTwo` 的函数体的时候，我们并不是在当前环境中执行，而是在函数的环境中执行。我们拓展函数环境并且将其传递给`Eval`函数而不是当前环境。为什么要这么做呢？因为我们可以仍然可以访问它们：为了可以使用闭包；为了我们可以做出炫酷的东西出来。

同样值得我们注意的是：不仅仅可以从其他函数中返回参数，也可以在函数中接受其他函数作为参数。函数是Monkey语言中的一等公民，可以像其他类型一样进行传递。

```monkey
>> let add = fn(a, b) { a + b }
>> let sub = fn(a, b) { a - b }
>> let applyFunc = fn(a, b, func){ func (a, b) }
>> applyFunc(2, 2, add);
4
>> applyFunc(10, 2, sub);
8
```

在这里我们将 `add` 和 `sub` 函数作为参数传递 `applyFunc`。它调用函数没有任何问题，`func` 参数作为一个函数对象，它然后调用其他两个参数，所有东西在解释器里都是能够工作。

现在我们的 `Monkey` 解释器知识函数、函数调用、高阶函数和闭包。
