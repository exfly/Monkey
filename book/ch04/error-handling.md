# 错误处理

还记得我们之前返回的 `NULL` 对象，这会我们将这里这个类型。首先先定义一下什么是真正的错误处理机制，这并不是用户自定义异常，而且内部的错误处理，主要包含错误操作符、不支持的操作运算或者出现的用户或者内部异常。

至于这个中异常处理方法的实现方式有多种，但是大部分是处理返回语句，原因很简单，因为错误和返回语句很相似，都是停止执行语句。

首先需要处理的一个错误对象：

```go
// object/object.go
const (
// [...]
    ERROR_OBJ = "ERROR"
)
type Error struct {
    Message string
}
func (e *Error) Type() OjectType { return ERROR_OBJ }
func (e *Error) Inspect() string { return "ERROR: " + e.Message}
```

可以看出 `object.Error` 仅仅封装一个 `string` 类型的字段，用来错误消息。在生产使用的解释器，还需要将栈调用信息、错误所在的具体行和列添加到这个消息中。这个并不难实现，只需要在词法解析器中将行列好添加进去，至于为什么没有这么做，因为我想把事情做得简单一点，只需要一个错误消息即可，它能反馈信息来反应停止执行的代码原因。

现在是增加单元测试：

```go
// evalutor/evalutor_test.go
func TestErrorHandling(t *testing.T) {
    tests := []struct {
        input           string
        expectedMessage string
    }{
        {"5+true;", "type mismatch: INTEGER + BOOLEAN"},
        {"5+true; 5;", "type mismatch: INTEGER + BOOLEAN"},
        {"-true", "unknown operator: -BOOLEAN"},
        {"true+false", "unknown operator: BOOLEAN + BOOLEAN"},
        {"5;true+false;5", "unknown operator: BOOLEAN + BOOLEAN"},
        {"if (10>1) { true+false;}", "unknown operator: BOOLEAN + BOOLEAN"},
        {`if (10 > 1) {
      if (10>1) {
            return true+false;
            }
            return 1;
}`, "unknown operator: BOOLEAN + BOOLEAN"},
        {"foobar", "identifier not found: foobar"},
        {`"Hello" - "World"`, "unknown operator: STRING - STRING"},
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        errObj, ok := evaluated.(*object.Error)
        if !ok {
            t.Errorf("no error object returned. got=%T(%+v)",
                evaluated, evaluated)
            continue
        }
        if errObj.Message != tt.expectedMessage {
            t.Errorf("wrong error message. expected=%q, got=%q",
                tt.expectedMessage, errObj.Message)
        }
    }
}
```

对于测试用例实际上判断两件事情：一是对于不支持的操作生成一些错误异常；二是阻止这些错误异常继续执行。生成错误异常并且传递给 `Eval` 函数非常简单，我们只需要一个辅助函数来帮助我们认为需要的地方创建一个 `*object.Errors` 对象即可。

```go
//evaluator/evalutor.go
func newError(format string, a ...interface{}) *object.Error{
    return &object.Error{Message: fmt.Sprintf(format, a...)}
}
```

这个 `newError` 函数用在我们不知道返回什么的时候返回，而不是仅仅返回 `NULL`。

```go
// evalutor/evalutor.go
func evalPrefixExpresion(operator string, right object.Object) object.Object {
    switch operator {
// [...]
    default:
        return newError("unknown operator:%s%s", operator, right.Type())
    }
}
func evalInfixExpression(
    operator string,
    left, right object.Object,
)object.Object {
    switch {
// [...]
    case left.Type() != right.Type():
        return newError("type mismatch:%s %s %s",
        left.Type(), operator, right.Type())
    default:
        return newError("unknown operator: %s %s %s",
        left.Type(), operator, right.Type())
    }
}
func evalMinusPrefixOperatorExpression(right object.Object) object.Object {
    return right.Type() != object.INTEGER_OBJ {
        return newError("unknon operator: -%s", right.Type())
    }
// [...]
}
func evalIntegerInfixExpression(
    operator string
    left, right object.Object,
)object.Object {
//[...]
    switch operator {
//[...]
    default:
        return newError("unknown operator: %s %s %s",
        left.Type(), operator, right.Type())
    }
}
```

现在测试还不能全部通过，因为需要在 `evalProgram` 和 `evalBlockStatement` 函数中的入口增加一些错误处理。

```go
func evalProgram(program *ast.Program) object.Object {
    var result object.Object
    for _, statement := range program.Statements {
        result = Eval(statement)
        switch result := result.(type) {
        case *object.ReturnValue:
            return result.Value
        case *object.Error:
            return result
        }
    }
}
func evalBlockStatment(block *ast.BlockStatement) object.Object {
    var result object.Object
    for _, statement := range block.Statements {
        result = Eval(statement)
        if result != nil {
            rt := result.Type()
            if rt == object.RETURN_VALUE_OBJ || rt == object.ERROR_OBJ {
                return result
            }
        }
    }
}
```

现在测试通过了，但是还需要在调用 `Eval` 函数调用是否有错误异常，为了防止错误传递，我们需要在源头上上浮到程序执行的地方：

```go
// evalutro/evalutor.go
func isError(obj object.Object) bool {
    if obj != nil {
        return obj.Type() == object.ERROR_OBJ
    }
    return false
}
func Eval(node ast.Node) object.Object {
    switch node := node.(type) {
//[...]
    case *ast.ReturnStatement:
        val := Eval(node.ReturnValue)
        if isError(val){
            return val
        }
        return &object.ReturnValue{Value: val}
//[...]
    case *ast.PrefixExpression:
        right := Eval(node.Right)
        if isError(right){
            return right
        }
        return evalPrefixExpression(node.Operator, right)
    case *ast.InfixPression:
        left := Eval(node.Left)
        if isError(left){
            return left
        }
        right := Eval(node.right)
        if isError(right){
            return right
        }
        return evalInfixExpression(node.operator, left, right)
    }
// [...]
}
func evalIfExpression(ie *ast.IfExpression) object.Object{
    condition := Eval(ie.Condition)
    if isError(condition){
        return condition
    }
// [...]
}
```

现在错误异常处理完毕。
