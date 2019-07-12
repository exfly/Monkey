# 返回语句

接下来就是返回语句，这个在任何标准语句都不会出现，在 `Monkey` 语言中不仅仅是函数体使用，在任何地方都会有任何影响，因为它不会改变任何东西，返回语句将停止后面的计算，并返回计算出来的值。

```monkey
5 * 5 * 5;
return 10;
9 * 9 * 9;
```

这个程序将会返回 10， 如果这些语句在一个函数体重，那么调用者将会得到 `10`，最重要的是最后一行代码 `9 * 9 * 9` 表达式将不会被执行。

有其他的方法实现返回语句，在一些宿主语言中我们可以使用 `goto` 或者异常等方式。但是在 `go` 语言中，实现异常捕获非常困难，而且也不想使用 `goto` 这种不简洁的方式，为了支持返回语句，我们会传递一个**返回值**给执行器，当遇到一个 `return` 就会将其封装好，并返回里面的对象。因此我们能跟踪记录它，以便决定是否继续计算。

下面是 `object.ReturnValue` 的实现：

```go
// object/object.go
const (
//[...]
    RETURN_VALUE_OBJ = "RETURN_VALUE"
)
type ReturnValue struct {
    Value Object
}
func (rv *ReturnValue) Type() ObjectType { return RETURN_VALUE_OBJ }
func (rv *RetrunValue) Inspet() string {
    return rv.Value.Inspect()
}
```

下面是返回语句的单元测试：

```go
// evaluator/evaluator_test.go
func TestReturnStatement(t *testing.T){
    tests := []struct {
        input string
        expected int64
    }{
        {"return 10;", 10},
        {"return 10; 9;", 10},
        {"return 2 * 5; 9;", 10},
        {"9; return 2 * 5; 9;", 10},
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        testIntegerObject(t, evaluated, tt.expected)
    }
}
```

为了让测试通过，我们为需要为 `evalStatement` 做一些修改，在 `Eval` 函数中增加一个 `*ast.ReturnStatement` 分支即可：

```go
// evalutor/evaluator.go
func Eval(node *ast.Node) object.Object {
// [...]
    case *ast.ReturnStatement:
        val := Eval(node.ReturnValue)
        return *object.ReturnValue{Value: val}
// [...]
}
func evalStatements(stmts []ast.Statement) object.Object {
    var result object.Object
    for _, statement := range stmts {
        result = Eval(statment)
        if returnValue, ok := result.(*object.ReturnValue);ok {
            return returnValue.Value
        }
    }
    return result
}
```

首先修改的内容是执行器的 `*ast.ReturnValue` 部分，在这里执行待 `return` 语句的表达式，然后将 `Eval` 计算得到的结果封装成 `object.ReturnValue` 对象并跟踪它。

在 `evalStatement` 语句中，执行 `evalProgramStatement` 和 `evalBlockStatement` 方法分别计算一系列语句，检查每一步的执行结果是不是 `object.ReturnValue`，如果是则停止下面的计算，返回那个被封装的值，这一点非常重要，这里并不是返回返回 `object.ReturnValue` 而是封装的值，这也是调用者需要返回的值。

但是问题是我们想持续跟踪 `object.ReturnValues` 而不是一旦遇到的为封装的值，这个在语句块中经常遇到。

```go
if (10 > 1) {
    if (10 > 1){
        return 10;
    }
    return 1;
}
```

这个程序应该返回 `10`，但是在目前的版本中它只返回 `1`，通过一个小测试就可以确认：

```go
// evalutor/evalutor_test.go
func TestReturnStatements(t *testing.T){
    tests :=[]struct {
        input string
        expected int64
    }{
        {`
        if (10 > 1){
            if (10 > 1){
                return 10;
            }
            return 1;
        }
        `, 10,
        },
    }
}
```

这个测试肯定不能通过，我想你已经知道其中的原因，因为在 `Moneky` 中存在嵌套语句，我们不能一遇到 `object.ReturnValue` 就封装的值取出来，所以还需要继续跟踪该值，直达到达最外面的一层语句块再停止执行。当前版本在非嵌套语句中可以很好的执行，但是遇到嵌套的语句块，首先要承认的是不能继续使用 `evalStatement` 函数来执行语句块，这也是为什么要重新命令 `evalProgram` 函数，以便它不能泛化。

```go
// evaluator/evaluator.go
func Eval(node ast.Node) object.Objcet {
// [...]
    case *ast.Program:
        return evalProgram(node)
// [...]
}
func evalProgram(program *ast.Program) object.Object {
    var result object.Object
    for _, statement := range program.Statements {
        result = Eval(statement)
        if returnValue, ok := result.(*object.ReturnValue); ok {
            return returnValue.Value
        }
    }
    return result
}
```

为了执行 `*ast.BlockStatement`，我们引入新的函数 `evalBlockStatement` 方法。

```go
// evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
//[...]
    case *ast.BlockStatement:
        return evalBlockStatement(node)
//[...]
}
func evalBlockStatement(block *ast.BlockStatement) object.Object {
    var result object.Object
    for _, statement := range block.Statements {
        result = Eval(statement)
        if result != nil && result.Type() == object.RETURN_VALUE_OBJ {
            return reuslt
        }
    }
    return result
}
```

在这里对于每个执行结果，我们显示说明不拆封返回值只是检查其 `Type` 方法，如果它是 `object.RETURN_VALUE_OBJ`，我们就简单返回 `*object.ReturnValue` 而不是取出其中的 `.Value` 字段，所以他执行执行的地方就是最外面的语句块，像气泡一样直达 `evalProgram` 方法，然后取出被封装的值。

现在测试通过了：

```shell
go test ./evalutor
ok monkey/evalutor 0.007s
```

我们终于不用再构建一个计算器了，但是 `evalProgram` 和 `evalBlockStatement` 方法对我们来讲还是比较陌生，我们将继续研究它们。
