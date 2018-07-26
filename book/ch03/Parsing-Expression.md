**解析表达式**

主管的来讲，解析表达式是编写解析器中最有趣的部分。正如我们刚刚看到的，解析语句相对比较直接。我们从左到右处理每个`token`，期待或者拒绝下一个token直到我们返回抽象语法树的节点。

从另一方面来讲，解析表表达式有更多的挑战。操作符优先级是第一个需要考虑的，家接下来就是一个例子，我们说我们想要解析如下的算术表达式：
```
5 * 5 + 10
```
我们想要让我们的抽象语法树用以下的的形式来表达：
```
((5 * 5) + 10)
```
也就是说`5*5`在抽象语法树中要更低一些，比加法更早地执行。为了生成这样的抽象语法树，解析器需要知道操作符的优先级，也就是`*`的优先级比`+`要高，这也是常见的操作符优先级例子，但是在某些情况下，这个就非常重要了。考虑到如下形式：
```
5 * (5 + 10)
```
在这里，括号将`5+10`表达式组在一起，使它们的优先级得到提升。现在加法在乘法之前执行。因为括号比`*`有更高的优先级。接下来我们会看到更多的例子，优先级扮演着重要的角色。

另外一个挑战是在表达式中，同样的`token`可以出现在不同的位置。与之相反的是，`let`只能出现在`let`语句的开始的地方，这样很容易帮我们决定剩下的语句应该是什么。比如看下面的表达式：
```
-5-10
```
在这里`-`操作符出现在表达式开始的地方，作为前缀操作符。然后组委中缀操作符出现在中间。同样的挑战如下：
```
5 * (add(2, 3) + 10)
```
尽管你可能不会认为括号也作为操作符，但是它同样出现了刚刚`-`中的问题。最外面的括号表示组合表达式，里面的括号表示了调用表达式。`token`位置是否有效取决于上下文，来确定它们之间的优先级。

**Monkey中的表达式**

在Monkey编程语言中，除了`let`和`return`语句外，其余的都是表达式，这些表达式有不同的形式。

Monkey语言中的前缀表达式
```
-5
!true
!false
```
当然也有中缀表达式
```
5 + 5
5 - 5
5 / 5
5 * 5
```
除了基础的算术操作符，也有一些比较操作符：
```
foo == bar
foo != bar
foo < bar
foo > bar
```
当然正如我们先前看到的，我们可以使用括号将表达式组合起来影响执行的顺序：
```
5 * (5 + 5)
(( 5 + 5 ) * 5) * 5
```
还有调用表达式
```
add(2, 3)
add(add(2, 3), add(5, 10)) max(5, add(5, (5 * 5)))
```
标识符同样也是表达式
```
foo * bar / foobar
add(foo, bar)
```
函数在Monkey中也是一等公民，字面函数也是表达式。我们可以使用`let`语句讲一个函数绑定到一个名字上。字面函数就是语句形式的表达式。
```
let add = fn(x, y) { return x+y };
```
我们也可以用字面函数代替标识符
```
fn(x, y) { return x + y }(5, 5) 
(fn(x) { return x }(5) + 10 ) * 10
```
和其他语言一样，我们也有`if`表达式
```
let result = if (10 > 5) {true} else {false};
result // => true
```
看到上述所有的不同形式的表达式，我们非常清楚需要一个很好的方法来正确的解析它们。我们老的方法是基于当前的Token来决定接下来要做什么并不能再次帮助我们，现在就是`Vaughan Patt`方法大显神威的时候。

**指定向下操作符优先级**
Vaughan Patt在他的`指顶向下操作符优先级`论文中提出了新的方法来解析它们，他的原文是这样的：
> ……这非常容易理解、实现和使用，在实际使用中非常高效，而且满足大多部分用户来的需要。

这篇论文在1973发表，但是在Pratt提出这个想法后很多年都没有支持者。知道最近几年，其他开发人员发现了Pratt的论文，编写实现他们导致`Pratt`的方法变得流行起来。Douglas Crockford的`JavaSrcipt: The Good Part`文章中，显示了如何将`Pratt`的想法用JavaScript实现。同样也有一片特别推荐的文章，是由Bob Nystrom写的，他是著名的`Game Programming Patterns`书的作者。它认为Pratt的方法非常容易理解和实现，并且简单的用Java实现了几个例子。

这个解析方法由三种描述，被发明出来作为基于上下文无关文法和BNF的解析器的替代方案。

同样也有主要的不同点：不同于由解析函数（在parseLetStatement中）由语法规则（在BNF或者EBNF中定义）定的，Pratt方法通过将单个`token`类型和函数联系起来。该想法中最重要的部分是每个`token`类型有两个解析函数，取决于该`token`的位置：中缀或者前缀。

我猜现在还没有更大的意义，我们还没有看到如何将一个解析函数和语法结合起来，所以将`token`类型而不是语法规则并没有产生很大的作用，老实地说，我在写这小节的时候，我面临则“鸡生蛋和蛋生鸡”的问题。有没有更好的方法用抽象的术语来解释这个算法和展示实现过程？会不会导致你不停地翻页？或者可能导致你跳过实现的部分？

答案是我决定不采用这两种方式。我们接下来要做的不是实现解析表达式而是近距离地看这个算法。最后我们将会拓展和完成该算法使之能够解析Monkey中的所有的表达式。

在开始编写代码之前，我们先厘清一些概念。

**概念**

`前缀操作符`是出现在操作符前面的操作符，比如：
```
--5
```
在这里操作符`--`，操作数是整数`5`，操作符出现在操作数前面。

`后缀操作符`是出现操作数后面的操作符，比如：
```
foobar++
```
在这里操作符`++`，操作数是标识符`foobar`，操作符出现在操作数后面。在Monkey语言中，我们不会构建任何后缀操作数。不是因为技术上的限制，而是保持本书内容的范围。

而`中缀操作符`我们先前已经看过了，它出现在两个操作符之间，比如
```
5 * 8
```
在这里`*`操作符出现在两个整数`5`和`8`之间，中缀操作符出现在二元表达式中。

其他的概念我们已经接触到了，比如`操作符优先级`。也叫做`操作顺序`，它表明了不同操作符之间的优先级，一个重要的例子就是我们先前看到的：
```
5 + 5 * 10
```
上述表达式的结果是55不是100，那是因为`*`操作符有更高的优先级，它比`+`操作符更重要，它应该比其他的操作符先执行。我有时候想，操作符的优先级比作操作符的粘性：用来描述操作数应该和哪一个操作符粘在一起。

这些事基础的概念:`前缀`, `后缀`, `中缀`和`优先级`。让这些概念在我们脑海中定义好是非常重要的。我们将会在其他地方使用它们。

现在我们开始输入和编写一些代码！

**准备抽象语法树**

为了解析表达式，我们首先要做的是准备抽象语法树。 正如我们先前看到的，在Monkey中程序是由一系列的语句构成的。一些事`let`语句，其余的是`return`语句，我们需要添加第三种类型语句：表达式语句。

这听上述非常混淆，我之前告诉你`let`和`return`语句是Monkey语言中唯二的语句，但是表示语句不是一种与众不同的语句；它是只有表达式构成的语句，它仅仅是一个封装器，我们的确需要它因为在Monkey中这是合法的。我们可以输入如下的代码：
```
let x = 5;
x + 10;
```
第一行是一个`let`语句，第二行是表达式语句。其他语言中没有表达式语句，但是在大部分脚本语言中，只有一行表达式的代码是可能的。所以我们需要在抽象语法树中增加节点类型
```go 
// ast/ast.go
type ExpressionStatement struct {
    Token token.Token // the first token of the expression 
    Expression Expression
}
func (es *ExpressionStatement) statementNode() {}
func (es *ExpressionStatement) TokenLiteral() string { return es.Token.Literal }    
```
这个`ast.ExpressionStatement`类型有两个字段：一个是`Token`字段，这个每一个节点都用，另外是`Expression`字段，它用来保存表达式。`ast.ExpressionStatement`实现了`ast.Statement`接口，也就意味着我们可以将它添加到`ast.Program`的`Statement`切片中。这也是我们为什么添加`ast.ExressopnStatement`的原因。

有了`ast.ExresspionStatement`定义，我们可以重用先前的工作。但是为了使我们的工作更轻松点，我们可以在我们抽象语法树节点添加`String()`方法。它允许我们输出抽象语法树节点，这样用来调试和比较其他节点。这样在测试中非常有用。

我们将要让`String()`方法称为`Node`接口的一部分：
```go
// ast/ast.go
type Node interface { 
    TokenLiteral() string 
    String() string
}
```
现在`ast`包中每一个节点都需要实现该方法。由于这些改变，我们的代码将不会被编译，因为编译器认为我们的抽象语法树中的节点没有完全实现`Node`接口。首先我们先为`*ast.Program`添加`String()`方法：
```go
import (
     // [...]
    "bytes"
)
func (p *Program) String() string {
    var out bytes.Buffer
    for _, s := range p.Statements {
        out.WriteString(s.String())
    }
    return out.String()
}
```
该方法并没有做很多的工作，它仅仅是创建了一个缓冲对象，然后往避免添加每一个语句调用`String()`的返回值。然后将缓冲区对象作为字符串返回。它将全部工作交给了`*ast.Program`中的`Statement`来完成。

真正的工作是由下面三个语句`ast.LetStatement`，`ast.ReturnStatement`和`ast.ExpressionStatement`的`String()`方法完成的。
```go
// ast/ast.go
func (ls *LetStatement) String() string { 
    var out bytes.Buffer
    out.WriteString(ls.TokenLiteral() + " ") 
    out.WriteString(ls.Name.String()) 
    out.WriteString(" = ")
    if ls.Value != nil { 
        out.WriteString(ls.Value.String())
    }
    out.WriteString(";")
    return out.String() 
}
func (rs *ReturnStatement) String() string { 
    var out bytes.Buffer
    out.WriteString(rs.TokenLiteral() + " ")
    if rs.ReturnValue != nil { 
        out.WriteString(rs.ReturnValue.String())
    }
    out.WriteString(";")
    return out.String() 
}
func (es *ExpressionStatement) String() string { 
    if es.Expression != nil {
        return es.Expression.String() 
    }
    return "" 
}
```
当我们构件好表达式，其中的空值检查我们接下来会去掉的。

