# 内置函数

在本小节中，我们将会为我们的解释器增加内置函数。之所以称之为内置因为它们不是由用户定义，也不是 `Monkey` 的代码，它们内置在解释器中，就是语言的本身。

这些内置函数由我们定义，它们是 `Monkey` 语言和我们解释器实现之间的桥梁，许多语言提供内置函数，这些函数为用户提供这些语言不具备的功能。

举个例子，一个返回当前时间的函数。为了获取当前时间需要访问内核或者其他计算机。向内核发起请求通常由系统调用完成，但是编程语言并没有提供这些功能。编程语言，或者是编译器应该替用户完成这些操作。

再次强调一下，内置函数是由我们定义的。解释器的用户可以调用它们，但是我们定义的。这些函数能够做什么事开放的，唯一严格要求的是它们接受零个或者多个 `object.Object` 参数，返回一个 `object.Object`。

```go
type BuiltinFunction func(args ...Object) Object
```

这是 `Go` 可调用的类型定义，但是因为要让这些 `BuilitinFunction` 能够使用户可用，需要让他们适应对象系统。所以需要做一层封装：

```go
//object/object.go
const (
// [...]
    BUILTIN_OBJ = "BUILTIN"
)
type Builtin struct {
    Fn BuiltinFunction
}
func (b *Builtin) Type() ObjectType { return BUILTIN_OBJ }
func (b *Builtin) Inspect() string { reutnr "builtin funciton" }
```

正如你看到的，这里 `object.Builtin` 并没有什么东西，仅仅是做了一层封装，但是这些对我们来讲就可以开始工作了。

## Len

第一个要添加到内置函数就是 `len` 函数，它返回一个字符串中字符的数量，作为用户在自己定义功能几乎不可能，这也是我们为什么要内置它的原因。我们想要让`len` 这样工作。

```monkey
>> len("Hello World!")
12
>> len("")
9
>> len("Hey Bob, how ya doin?")
21
```

`len` 背后的逻辑非常清楚了，为了更加清晰，可以编写一些测试：

```go
//evaluator/evalutor_test.go
func TestBuiltinFunction(t *testing.T) {
    tests := []struct {
        input    string
        expected interface{}
    }{
        {`len("")`, 0},
        {`len("four")`, 4},
        {`len("hello world")`, 11},
        {`len(1)`, "argument to `len` not supported, got=INTEGER"},
        {`len("one", "two")`, "wrong number of arguments. got=2, want=1"},
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        switch expected := tt.expected.(type) {
        case int:
            testDecimalObject(t, evaluated, int64(expected))
        case string:
            errObj, ok := evaluated.(*object.Error)
            if !ok {
                t.Errorf("object is not Error, got=%T(%+v)",
                    evaluated, evaluated)
            }
            if errObj.Message != expected {
                t.Errorf("wrong err messsage. expected=%q, got=%q",
                    expected, errObj.Message)
            }
        }
    }
}
```

在这一些测试用例，按照这样的顺序依次执行 `len` 函数：空的字符串、正常的字符串、一个带空格的字符串。其实这个是没有关系的，如果空格是在字符串内。最后两个测试用例是更加有趣：如果参数为整型或者错误数量的参数将会返回一个`*object.Error`.

现在测试还不能通过，为了完成它，首先我们要做的是为内置函数提供找到它们的方法，在顶级的 `object.Environment` 中添加它们，并且传递给 `Eval` 函数，但是我们额外环境保存内置函数:

```go
//evalutor/builtins.go
package evaluator
import "monkey/object"
var builtins = map[string]*object.Builtin {
    "len":*object.Builtin{
        Fn: func(args ...object.Object) object.Object{
            return NULL
        },
    },
}
```

同样需要编辑 `evalIdentifier` 函数，用来查找内置函数作为我们当前环境没有找到标识符的时候的返回值。

```go
//evalutor/evaluator.go
func evalIdentifier(
    node *ast.Identifier,
    env *object.Environment,
)object.Object {
    if val, ok := env.Get(node.Value); ok {
        return val
    }
    if builtin, ok := builtins[node.Value]; ok {
        return builin
    }
    return newError("identifier not found: " + node.Value)
}
```

现在 `len` 标识符可以找得到 `len` 内置函数，但是调用仍然不起作用:

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>>len()
ERROR: not a funciton: BUILTIN
```

运行单元测试同样的错误，我们需要告诉我们 `applyFuntion` 关于`*object.Builtin` 和 `object.BuiltinFunction` 相关信息。

```go
func applyFunction(fn object.Object, args []object.Object)object.Object {
    switch fn := fn.(type){
    case *object.Function:
        extendEnv := extendFunctionEnv(fn, args)
        evaluated := Eval(fn.Body, extendEnv)
        return unwrapReturnValue(evaluated)
    case *object.Builtin:
        return fn.Fn(args...)
    default:
        return newError("not a function %s", fn.Type())
    }
}
```

除了调整代码行数，我们所增加的就是增加了 `*object.Builtin` 分支，在这里调用 `object.BuiltinFunction` ，这样做只需要将 `args` 切片作为参数，并且调用函数。值得注意的是我们不必要调用 `unwrapReturnValue` 函数，因为在内置函数中我们不会返回 `*object.ReturnValue`。

现在测试在调用`len`的时候，正确地返回了NULL对象。

现在调用 `len`并不是期望的结果，仅仅返回 `NULL` 对象，但是修复它们很简单，只要写几行 `Go` 函数:

```go
//evalutor/builtin.go
import (
    "monkey/object"
    "unicode/utf8"
)
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
            default:
                return newError("argument to `len` not supported, got=%s",
                    args[0].Type())
            }
        },
    },
}
```

最重要的部分是调用 `Go` 的 `len` 函数，并且返回新分配的 `object.Integer`，除此之外我们还做了错误检查，确保不能调用该函数使用错误的参数数目，或者支持的参数类型。现在测试通过了：

```shell
$ go test ./evaluator
ok monkey/evalutor 0.007s
```

这也意味着我们可以在REPL中使用`len`方法了

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> len("1234")
4
>> len("Hello World!")
12
>> len("wooooohoo!", "len works")
ERROR: wrong number of arguments: got=2, want=1
>> len(12345)
ERROR: argument to `len` no supported. got INTEGER
```
