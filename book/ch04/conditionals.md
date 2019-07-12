# 条件语句

在计算器中实现条件语句非常简单，难点就在于条件判断：

```monkey
if (x > 10) {
    puts("everything okay!")
}else{
    puts("x is too low!")
    shutdownSystem()
}
```

在计算 `if-else` 表达式中最重要的是计算正确的分支，如果条件为 `true` 就不需要计算 `else` 分支，只要关注 `if` 分支即可，如果不满足则计算 `else` 分支。

在 `Monkey` 语言例子中，条件语句就会执行，当条件类 `true` 的时候：

```monkey
let x = 10;
if(x){
    puts("everything okay");
}else{
    puts("x is too high");
    shutdownSystem();
}
```

在上面的例子中 `everything okay` 就会输出，因为 `x` 会绑定值 `10`, `10` 计算不为空为真值。和先前一样，我们先增加测试用例：

```go
func TestIfElseExpression(t *testing.T){
    tests := []struct {
        input string
        expected interface{}
    }{
        {"if (true) {10}", 10},
        {"if (false) {10}", nil},
        {"if (1) {10}", 10},
        {"if (1<2) {10}", 10},
        {"if (1<2) { 10} else {20}", 10},
        {"if (1>2) {10} else {20}", 20},
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

当条件没有没有被执行，它就返回 `NULL`：

```go
if (false){ 10 }
```

因为 `else` 分支缺失，因此这语句就会生成 `NULL`。现在测试肯定不能通过，因为我们并没有返回 `*object.Integer` 或者 `NULL`，接下来为 `switch` 增加新的分支，就可以通过全部测试：

```go
//evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
// [...]
    case *ast.BlockStatement:
        return evalStatments(node.Statements)
    case *ast.IfExpression:
        return evalIfexpression(node)
// [...]
}
func evalIfExpression(ie *ast.IfExpression) object.Object{
    condition := Eval(ie.Condition)
    if isTruthy(condition){
        return Eval(is.Consequeces)
    }else if ie.Alternaive != nil {
        return Eval(is.Alternative)
    }else{
        return NULL
    }
}
func isTruthy(obj object.Object) bool {
    switch obj {
    case NULL:
        return false
    case TRUE:
        return true
    case FALSE:
        return false
    default:
        return true
    }
}
```

正如之前所说，难点就在于决定计算哪个分支，所有决定分支都封装在逻辑 `evalIfExpresion` 函数，`isTruthy` 作用是相等判断式。上述两个函数也增加了 `*ast.BlockStatement` 条件分支，因为 `*ast.IfExpression` 中 `.Consequences` 和 `.Alternative` 也是语句块。

我们增加了两个新的具体函数来表达 `Monkey` 语言语法，重用已经存在的函数，仅仅增加一条支持条件语句，然后测试通过。现在解释器支持 `if-else` 表达式，现在离开简单计算器领域，向编程语言进军：

```shell
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> if (5*5+10>34) { 99 } else { 100 }
99
>> if ((100/2) + 250 * 2==1000){9999}
9999
```
