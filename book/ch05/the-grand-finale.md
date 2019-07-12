# 完结

我们 `Monkey` 解析器完全是函数式的，它支持数学表达式，变量绑定，函数和函数的调用，条件表达式、返回语句甚至更高级的概念，比如高阶函数和闭包。还有不同的数据类型：整型、布尔型、字符串、数组和哈希表。

但是我们的解释器还不能通过基础的编程语言功能：输出。我们的 `Monkey` 解释器不能和外部交流。甚至其他语言如 `Bash` 和 `Brainfuck` 都能够做到这点，`Monkey` 也要完成这个，接下来我们要增加最后一个内置函数 `puts`

`puts` 输出给定的参数到 `STDOUT`。它调用穿入参数对象 `Inspect()` 方法并且将他们返回值输出。`Inspect()` 方法是 `Obejct` 接口对象，所以每一个实体都支持这个方法，使用`puts`函数看上去是这样的：

```monkey
>> puts("Hello!")
Hello!
>> puts(1234)
1234
>> puts(fn(x){x(x)})
fn(x) {
    (x * x)
}
```

`puts` 函数是可变参数的函数，它接受无限制数量的参数，并且将每一个参数按行输出

```monkey
>> puts("hello", "world", "how", "are", "you")
hello
world
how
are
you
```

`puts` 仅仅是输出并不生成任何值，我们确保他返回一个 `NULL`:

```monkey
>> let putReturnValue = puts("foobar")
foobar
>>putReturnValue
null
```

这也意味着我们的 `REPL` 将除了输出 `puts` 的内容外，还会输出一个 `null`对象，它看上去像这样：

```monkey
>>puts("Hello!")
Hello!
null
```

现在还有更多的信息和说明来完成最后一个请求，这也是我们这一小节将要构建的，实现`puts`函数如下：

```go
//evaluator/builtins.go
import (
    "fmt"
    "monkey/object"
    "unicode/utf8"
)
var builtins = map[string]*object.Builtin{
// [...]
    "puts": &object.Builtin{
        Fn: func(args ...object.Object) object.Object {
            for _, arg := range args {
                fmt.Println(arg.Inspect())
            }
            return NULL
        }
    }
}
```

现在完成了 `Monkey` 语言的输出功能。

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> puts("Hello World!")
Hello world!
null
```
