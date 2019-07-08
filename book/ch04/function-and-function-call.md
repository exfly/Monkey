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

