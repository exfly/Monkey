# 对象表示

你可能会疑问从来没有说过 `Monkey` 是面向对象的编程语言啊！是的，我从来没有说过并且它也不是面向对象的编程语言。那么为什么需要一个**对象系统**呢，亦或者是**值系统**或者**对象描述**？答案是我们需要定义计算的返回值，需要一个系统来描述我们抽象语法树中的值或者在内存计算中生成的内容。

让我们来看看接下来 `Monkey` 代码中如何计算的：

```monkey
let a = 5;
// [...]
a + a;
```

正如你看到的，我们将一个字面整数绑定到 `a` 变量中。接下来无论发生什么，比如我们遇到一个 `+` 的表达式，需要提取 `a` 绑定的值。为了计算 `a+a`，需要获取 `5`。在抽象语法树中，它使用 `*ast.IntegerLiteral` 对象表示，那么我们该如何去记录这些表达式形式在计算其他的抽象语法树的时候呢？

构建一门解释器语言内部值表示有很多选择，这方面主题的内容在全世界关于解释器和编译器的代码库中导出都有。每个解释器都有自己的实现方式，为了适应解释器语言的要求，它们往往各自不同。

在解释型语言中，可以使用宿主语言来表示一些原生类型（比如整型、布尔型等），但是不能完整地表示，比如在其他语言中值和对象只能使用指针来表示，有些是用原生和指针混合使用。

为什么会有这么多类型呢？其中之一的原因是宿主语言不通，如何在你的解释器中描述一个字符串取决于你是如何实现解释器的，一个用 `Ruby` 编写的解释器不能用同样的方式 `C` 语言描述。除此之外，解释器相互之前是不同的，一些解释语言只需要描绘原始数据类型，比如整型、字符型或者字节，但是在其他解释型语言你需要列表、字典、函数以及其他复合数据类型。这些不同点导致了在类型表示上提出了不同的需求。

除了宿主语言和被解释器的语言，在设计和实现类型描述影响最大的是在计算执行过程中的速度和内存消耗。如果你想要构建一个高效的解释器就不能使用膨胀的类型系统或；如果你想自己编写垃圾收集就需要考虑在系统中如何跟踪每一个对象。换句话说，如果性能不是最关心的东西，那么保持解释器简单明了是非常重要的。

在宿主语言中的解释语言有很多不同的类型描述方法，最好或者是唯一的方法是了解和阅读这些流行的解释器的源代码，这里推荐 [Wren_source_code](https://github.com/munificent/wren)，它包含了两种不同的类型描述方法，可以通过编译条件打开它们。

除了考虑宿主语言在类型的问题，还需要考虑如何公开这些类型描述给解释语言的使用者，也就是公开的 API 接口长什么样？以 `JAVA` 语言为例，提供了基础数据类型（整型、字节、短整型、长整型、单精度浮点型、双精度浮点型、布尔型和字符）和相关引用数据类型，基础类型没有在 `JAVA` 实现中过度描述，它们与原生的一一对应，但是引用数据类型是宿主语言中复合数据类型的引用。

`Ruby` 语言的使用者不要接触基础数据类型，类似原生值类型也是不存在的，因为在 `Ruby` 中所有的都是对象，因此它们都被封装到内部实现中。在 `Ruby` 内部不区分一个字节类型和一个 `pizza` 类的实例对象，它们都是值类型，区别只在于封装了不同的值。

有很多方法将数据类型暴露给使用者，具体方式取决于语言的设计者。再一次我们需要强调的是这是性能方面的考虑，如果你不考虑这个问题，一切都好办了。

## 对象系统基础

目前我们对 `Monkey` 解释器的性能暂时没有考虑，所以我们选择一种简单的方式：我们为每一个遇到的类型描述为一个 `Object` 对象，它是设计的接口，每个值都表示的结构都实现 `Object` 接口。

在性的 `object` 包中，我们先定义 `Object` 接口和 `ObjectType` 类型：

```go
//object/object.go
package object
type ObjectType string

type Object interface {
    Type() ObjectType
    Inspect() string
}
```

和之前 `token` 包和 `Token` 和 `TokenType` 类型相同，不同点在于 `token` 是一个结构，而 `Object` 是一个接口，原因是每个值都需要不同的内部表示。目前 `Monkey` 解释器中有三种数据类型：`null`，布尔型和整型。接下来我们事先整型并且构建我们的值系统：

### Integers

`object.Integer` 类型比较简单：

```go
import (
    "fmt"
)
type Integer struct {
    Value int64
}
func (i *Integer) Inpsect() string { return fmt.Sprintf("%d", i.Value) }
```

在源代码中遇到整型字面值，我们首先将其转换一个 `*ast.IntegerLiteral`，然后在计算这个抽象语法树的节点的时候，我们将其转为一个 `object.Integer`，将它的值存入到结构中然后这这个引用返回出去。

为了让 `object.Integer` 结构实现 `object.Object` 接口，需要增加一个 `Type` 方法返回 `ObjectType`，就像我们`token.TokenType` 一样，我们为每一个 `ObjectType` 定义一个常量：

```go
// object/object.go
import "fmt"
type ObjectType string
const (
    INTEGER_OBJ = "INTEGER"
)
```

现在可以在 `*object.Integer` 结构中添加 `Type()` 方法:

```go
//object/object.go
func (i *Integer) Type() ObjectType { return INTEGER_OBJ }
```

到目前为止，我们完成了 `Integer` 类型，接下来就是布尔型数据。

### Booleans

```go
//object/object.go
const (
//[...]
    BOOLEAN_OBJ = "BOOLEAN"
)

type Boolean struct {
    Value bool
}
func (b *Boolean) Type() ObjectType { return BOOLEAN_OBJ }
func (b *Boolean) Inpsect() string { return fmt.Sprintf("%t", b.Value)}
```

`object.Boolean` 也是非常简单，仅仅是封装了 `bool` 型变量的结构。

### null

1965 奶奶 `Tony Hoare` 在 `ALGOL W` 语言中引入空引用，也叫做[百万美元的错误](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。由于空类型引用导致了无数次系统崩溃，在描述空值的时候， `NULL` 不是一个号的类型。

我也思考过，`Monkey` 语言中是否应该引入空引用，一方面来讲，引入空引用将会将变得不安全；但是我们并不是重新发明一个轮子，而是去学习知识。于是在 `Monkey` 语言中包含了空引用，这也提醒我在设计过程中充分考虑这个因素。

```go
//object/object.go
const (
// [...]
    NULL_OBJ = "NULL"
)
type Null struct {}

func (n *Null) Type() ObjectType { return NULL_OBJ }
func (N *Null) Inspect() string { return "null" }
```

`object.Null` 与 `object.boolean` 和 `object.Integer` 除了没有封装任何东西，其余的都是一样，它表示了缺失值。

有了 `object.Null`，我们的对象系统可以表示布尔型、整型和空类型，对于实现 `Eval` 函数足够了。
