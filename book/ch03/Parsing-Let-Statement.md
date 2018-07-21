**解析LET语句**
在Monkey语言中，变量绑定语句的形式如下：
```
let x = 5;
let y = 10;
let foobar = add(5, 5);
let barfoo = 5 * 5 / 10 + 18 - add(5, 5) + multiply(124);
let anotherName = barfoo;
```
上面这些语句叫做`let`语句，将一个值绑定到一个变量上，`let x = 5`语句将值`5`绑定到变量名`x`上，本小节的我们的工作是正确地解析`let`语句。从现在开始我们将跳过解析表达式过程，它是通过给定的变量获得相应的值得过程。在我们知道如何解析表达式之后再来关注这些。

什么叫做正确的解析`let`语句？它意味着解析能够正确的得到一颗抽象语法树，它能够正确地表示原先`let`语句的全部信息。听上去很有可行，但是我们还没有任何抽象语法树，或者说我们还不知道着东西看上去怎么样？所以我们首先的任务是近距离看看Monkey语言，看看它结构如何。我们可以定义抽象语法树的必要部分，以便它能正确地表示`let`语句。

下面都是Monkey中合法的语句
```
let x = 10;
let y = 15;
let add = fn(a, b) {
    return a + b;
}
```
Monkey中都是一系列语句，在上面的例子中有三个语句，三个变量绑定。`let`语句有如下的形式：
```
let <identifier> = <expression>;
```
一个`let`语句有两个可变的部分组成：一个标识符和一个表达式。在上面例子中，`x`，`y`和`add`都是标识符，`10`,`5`和函数都是表达式。

在我们继续之前，还要进一步说明一下语句和表达式之间的不同是有必要的。表达式能够生成值，然后语句则不会。 `let x =5;`并不生成值，然而`5`能够（它生成5）。`return 5`是语句不会生成值，但是`add(5,5)`却能够生成值。区别就是在于:
表达式生成值，而语句则不会。当然也会做一些改变，这些取决于谁在问你，但是这样已经很好。

语句和表达式的正确的区分：是否生成值，取决于编程语言。在一些编程语言中函数`fn(x,y) {return x+y;}`是表达式，它可以被用在任何表达式可以使用的地方。在其他的编程语言中，字面函数只能作为编程语言函数声明的一部分。一些编程语言拥有`if`表达式，它的条件语句是表达式并且能够生成值。这些都取决于编程语言设计者怎么考虑的。正如你看到的，在Monkey语言中，大部分是表达式，包含字面函数。

回到我们的抽象语法树，看着上述的例子，我们知道我们需要定义两种不同类型的节点：表达式和语句，让我们看看抽象语法树如何开始：

```go
// ast/ast.go
package ast
type Node interface {
    TokenLiteral() string
}
type Statement interface {
    Node
    statementNode()
}
type Expression interface {
    Node
    expressionNode()
}
```
在这里我们定义了三个接口，分别为`Node`,`Statement`和`Expression`。我们抽象语法树中每一个节点必须实现`Node`接口，也就意味着它必须要提供`TokenLiteral`方法，它返回该节点关联的`token`的字面值。`TokenLiteral`将会被用在调试和测试中。抽象语法树将全部由`Node`构成，它们互相连接在一，毕竟这是一棵树。其中的一些节点实现了`Statement`接口，另外一些实现了`Expression`接口。这些接口各自包含了哑方法`statementNode`和`expressionNode`。它们不是严格要求的，但是它们能够帮助我们指导我们的Go编译器能够抛出可能的异常，当我们在应该使用`Expression`地方使用了`Statement`，相反也同理。

接下来就是我们`Node`的第一个实现

```go
//ast/ast.go
type Program struct {
    Statements []Statement
}
func (p *Program) TokenLiteral() string {
    if len(p.Statements) > 0 {
        return p.Statement[0].TokenLiteral()
    }else{
        return ""
    }
}
```
`Program`节点是每一个抽象语法树的根节点，每一个合法的Monkey程序都是一系列语句，这些语句被保存在`Program.Statement`中，它是一系列实现`Statement`接口的节点组成。



