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

现在我们为`ast.Identifier`增加`String()`方法。
```go
// ast/ast.go
func (i *Identifer) String() string { return i.Value }
```
有了这个方法，我们可以仅仅调用`*ast.Program`的`String()`方法，就可以得到整个程序的字符串表示形式。这样可以让我们的`*ast.Program`变得可以测试，让我们以Monkey的源代码作为一个例子：
```
let myVar = anotherVar;
```
如果我们以此构建抽象语法树，我们可以对`String()`的返回值做一些如下的测试：
```go
package ast
import ( 
    "monkey/token"
    "testing"
)
func TestString(t *testing.T) { 
    program := &Program{
    Statements: []Statement{ 
        &LetStatement{
            Token: token.Token{Type: token.LET, Literal: "let"}, Name: &Identifier{
                Token: token.Token{Type: token.IDENT, Literal:"myVar"},
                Value: "myVar", 
            },
            Value: &Identifier{
            Token: token.Token{Type: token.IDENT, Literal:"anotherVar"}, 
            Value: "anotherVar",}, 
            }, 
        }, 
    }
    if program.String() != "let myVar = anotherVar;" { 
        t.Errorf("program.String() wrong. got=%q", program.String())
    } 
}
```
在上述测试中，我们手动构建了抽象语法树。 当为解析器编写测试的时候当然不需要这么。当然需要最解析器生成的抽象语法树需要进行测试。为了测试目的，这个测试，向我们展示通过比较解析的输出可以让我们的测试变得更加可读。这也是变得非常可用如果我们解析表达式。

好消息是所有的工作都已经完成了，是时候完成`Pratt`解析器了。

**实现Pratt解析器**

Pratt解析器的主要思想是将解析函数和`token`类型联系在一起。无论遇到什么类型的`token`，解析函数将会被调用，并且解析出正确的表达式，并且返回抽象语法树的节点。每个`token`类型最多拥有两个相关的解析函数，这取决于该`token`是在前缀还是后缀位置发现的。

首先要做的建立这些关联，我们定义了两种类型的函数：一个前缀解析函数另一个是中缀解析函数。
```go
// parser/parser.go
type (
    prefixParseFn func() ast.Expression
    infixParseFn func(ast.Expression) ast.Expression
)
```
两个函数都返回`ast.Expression`，因为它们是要解析表达式。但是只有`infixParseFn`接受一个参数：另一个表达式。这个参数是中缀操作符的左边表达式，前缀操作符没有左边部分。我知道现在还没有看出有什么意义，但是请接受我这么做，接下来你会看到它们是怎么工作的。从现在请记住，当遇到`token`类型在前缀位置上调用`prefixParseFn`，当遇到`token`类型在中缀位置上，调用`infixParseFn`方法。

为了在遇到不同类型的`token`时候，我们的解析器正确地得到`prefixParseFn`或者`infixParseFn`，我们需要在`Parser`结构中添加两个字典映射：
```go
// parser/parser.go
type Parser struct {
    l *lexer.Lexer 
    errors []string
    curToken token.Token 
    peekToken token.Token
    prefixParseFns map[token.TokenType]prefixParseFn
    infixParseFns map[token.TokenType]infixParseFn 
}
```
有了映射表，我们可以通过检查表（前缀或者中缀)得到关联当前`curToken.Type`得到正确的解析函数。

我们也需要增加添加这两个映射表的词条的帮助函数：
```go
func (p *Parser) registerPrefix(tokenType token.TokenType, fn      prefixParseFn) { 
    p.prefixParseFns[tokenType] = fn
}
func (p *Parser) registerInfix(tokenType token.TokenType, fn infixParseFn) { 
    p.infixParseFns[tokenType] = fn
}
```
现在我们已经准备好去算法的核心部分：

**标识符**

我们现在开始可能在Monkey编程语言中最简单表达式：标识符。使用标识符的表达式语句的样子如下：
```
foobar;
```
当然`foobar`非常多元的了，在其他上下文环境中标识符也是表达式，并不仅仅是表达式语句。
```
add(foobar, barfoo);
foobar + barfoo;
if (boobar) {
    // [....]
}
```
在这里，我们将标识符作为函数额参数，作为中缀表达式的在操作树，同样也可以作为条件语句的单独表达式。它们可以被用在上述的所有的环境中，因为标识符也是表达式，就像`1+2`。就跟任何表达式一样，标识符能够生成值，它输出它们绑定的值。

让我们开始以测试开始
```go
func TestIdentifierExpression(t *testing.T) {
	input := "foobar;"
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program has not enough statements. got=%d",
			len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	ident, ok := stmt.Expression.(*ast.Identifier)
	if !ok {
		t.Fatalf("exp not *ast.Identifier. got=%T", stmt.Expression)
	}
	if ident.Value != "foobar" {
		t.Errorf("ident.Value not %s. got=%s", "foobar", ident.Value)
	}
	if ident.TokenLiteral() != "foobar" {
		t.Errorf("ident.TokenLiteral not %s. got=%s", "foobar",
			ident.TokenLiteral())
	}
}
```
它看上去有很多行代码，但是仅仅是简单的工作。我们解析输入`foobar;`，检查解析错误，然后判断`*ast.Program`节点中`Statment`数量，然后检查是不是只有一个语句，并且类型是`*ast.ExpressionStatement`。然后检查`*ast.ExpressionStatement.Expression`是否为`*ast.Identifier`。最后检查我们的标识符是否拥有正确的值`foobar`。

当然，我们的解释器测试是失败的
```
$ go test ./parser
--- FAIL: TestIdentifierExpression (0.00s)
parser_test.go:110: program has not enough statements. got=0 
FAIL
FAIL monkey/parser 0.007s
```
解析器目前还不知道任何关于表达式相关知识，我们需要编写`parseExpression`方法。

首先要做的是的拓展我们解析器的`parseStatement()`方法，以便于它能解析表达式语句。由于在Monkey中只有两个真正额语句`let`语句和`return`语句，如果我们没有遇到上述的语句，我们就尝试解析表达式语句:
```go
// parser/parser.go
func (p *Parser) parseStatement() ast.Statement { 
    switch p.curToken.Type {
    case token.LET:
        return p.parseLetStatement() 
    case token.RETURN:
        return p.parseReturnStatement() 
    default:
        return p.parseExpressionStatement() 
    }
}
```
解析表达式语句看上去是这样的：
```go

// parser/parser.go
func (p *Parser) parseExpressionStatement() *ast.ExpressionStatement { 
    tmt := &ast.ExpressionStatement{Token: p.curToken}
    stmt.Expression = p.parseExpression(LOWEST)
    if p.peekTokenIs(token.SEMICOLON) { 
        p.nextToken()
    }
    return stmt 
}
```
我们已经知道这么做了：构建我们的抽象语法树，然后调用其他解析函数来填充字段。在这个例子中，我们有些不同了：我们调用的`parseExpression()`方法并不存在，仅仅包含了一个常量`LOWEST`，而且它也不存在。然后我们检查备选的分号。是的，它是备选的。如果`peekToken`是`token.SEMICOLON`，我们前进`curToken`。如果不存在，同样也是无所谓的，我们不会往解析器中添加错误。那是因为我们希望表达式语句的分号是备选的。（因为在REPL中输入`5+5`更加容易一点）

如果我们现在运行测试，我们可以看到一个编译错误，因为`LOWEST`是未定义的。没关系，现在我们添加它，定义Monkey编程语言中的优先级。
```go
// parser/parser.go
const (
    _ int = iota
    LOWEST
    EQUALS // == 
    LESSGREATER // > or < 
    SUM // + 
    PRODUCT // *
    PREFIX // -X or !X
    CALL // myFunction(X) )
```
在这里我们使用`iota`给下面的每一个常量自增的值，空白标识符`_`占据着零值。剩下的常量将会被赋值为`1`到`7`，数字不重要，但是他们的循序和关系非常重要。我们需要这些常量要作什么接下来会给出答案：`*`操作符比`==`符优先级要高？

在`parseExpressionStatement`中，我们将最低可能优先级传递给`parseExpression`，因为目前我们还没有解析任何东西，我们不能比较任何优先级。我保证过不了多久，我们会将其变得有意义。让我们开始编写`parseExpression`:
```go
// parser/parser.go
func (p *Parser) parseExpression(precedence int) ast.Expression {       prefix := p.prefixParseFns[p.curToken.Type]
    if prefix == nil {
        return nil
    }
    leftExp := prefix()
    return leftExp 
}
```
这是初步版本，它所做的全部工作就是检查`p.curToken`在前缀位置上是否关联了一个解析函数。如果有，调用解析函数，如果没有然后`nil`。现在只能做这些因为我们还没有将`token`关联到任何函数，接下来我们要做的是：
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.prefixParseFns = make(map[token.TokenType]prefixParseFn) p.registerPrefix(token.IDENT, p.parseIdentifier)
// [...]
}
func (p *Parser) parseIdentifier() ast.Expression {
    return &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
}
```
我们修改了`New()`函数，来初始化`prefixParseFns`字典并且注册解析函数：如果我们遇到`token.IDENT`，解析函数调用`parseIdentifier`方法。

`parseIdentifier`方法并没有做很多工作，它仅仅返回一个`*ast.Identifier`包含了当前`token`作为`Token`字段和`token`的明面值作为`token`的值。它没有前进`token`，也没有调用`nextToken`。这是非常重要的，我们所有的函数`prefixParseFn`和`infixParseFn`都遵循如下协议规则：开始于`curToken`，它是和你当前`token`类型关联的，然后返回你的表达式类型的最后一个`token`。千万不要前进太远。

信不信由你，我们的测试通过了：
```
$ go test ./parser
ok monkey/parser 0.007s
```
我们成功地解析了标识符表达式。好的，在开始庆祝之前，让我们长吸一口气，我们接下来将会编写更过的解析函数。

**整数字面值**

解析整数字面值和标识符一样简单，它看上去是这样的：
```
5;
```
是的，整数字面值也是表达式。它生成的值就是整数本身，再一次，请思考一下整数字面值可以出现哪些地方可以明白为什么它是表达式：
```
let x = 5;
add(5, 10);
5 + 5 + 5;
```
我们可以使用任何表达式来替换整型字面值都是合法的，比如标识符、调用表达式，分组表达式、函数字面值诸如此类。所有的表达式都可以互换的，整型字面值也是其中的一个。

接下来的测试用例和之前的标识符测试用例比较类似：
```go
func TestIntegerLiteralExpression(t *testing.T) {
	input := `5;`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program has not enough statements. got=%d",
			len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statemtnets[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	integer, ok := stmt.Expression.(*ast.IntegerLiteral)
	if !ok {
		t.Fatalf("exp is not *ast.IntegerLiteral. got=%T", stmt.Expression)
	}
	if integer.Value != 5 {
		t.Errorf("integer.Value not %d. got=%d", 5, integer.Value)
	}
	if integer.TokenLiteral() != "5" {
		t.Errorf("integer.TokenLiteral not %s. got=%s", "5", integer.TokenLiteral())
	}
}
```
正如之前标识符测试用例中的一样，我们使用简单的输入，传递个解析器。然后检查解析器没有遇到任何错误再次输出`*ast.Program.Statement`的数量。 紧接着我们做一次判断，是一个参数是否为`*ast.ExpressionStatement`，最后我们其他类型为`*ast.IntegerLiteral`。

当然测试时不能通过的，因为`*ast.IntegerLiteral`还不存在，但是定义它并不难：
```go
// ast/ast.go
type IntegerLiteral struct { 
    Token token.Token
    Value int64
}
func (il *IntegerLiteral) expressionNode() {}
func (il *IntegerLiteral) TokenLiteral() string { return il.Token.Literal } 
func (il *IntegerLiteral) String() string { return il.Token.Literal }
```
`*ast.IntergerLiteral`实现了`ast.Expression`接口，正如`*ast.Identifier`一样，但是有一个显著的不同是它的结构本身：`Value`字段类型是`int64`而不是`string`。 这个字段用来保存整型字面值代表的真正值。当我们在构建`*ast.IntegerLiteral`的时候，我们需要将`*ast.Integeral.Token.Literal()`的字符串转换为`int64`。

调用解析函数最好的位置在关联`Token.INT`，该函数名叫`parseIntegerLiteral`：
```go
// parser/parser.go
import ( 
    // [...]
    "strconv"
)
func (p *Parser) parseIntegerLiteral() ast.Expression { 
    lit := &ast.IntegerLiteral{Token: p.curToken}
    value, err := strconv.ParseInt(p.curToken.Literal, 0, 64) 
    if err != nil {
        msg := fmt.Sprintf("could not parse %q as integer", p.curToken.Literal) 
        p.errors = append(p.errors, msg)
        return nil
    }
    lit.Value = value
    return lit 
}
```
和`parseIdentifier`方法比起来相当简单了。唯一不同的是它调用`strconv.ParseInt`，该函数将一个字符串转换一个`int64`。这个值将会保存在我们将会返回的`*ast.IntegerLiteral`节点的`Value`字段中。如果转换失败，我们在解析器的错误列表中增加一个错误。

但是我们的测试还是不能通过：
```
$ go test ./parser
--- FAIL: TestIntegerLiteralExpression (0.00s)
parser_test.go:162: exp not *ast.IntegerLiteral. got=<nil> FAIL
FAIL monkey/parser 0.008s
```

在抽象语法树中，我们得到的是一个`nil`而不是`*ast.IntegerLiteral`。原因是`parseExpresion`方法不能为当前`token.INT`得到一个`parsefixParseFn`方法。 我们注册`parseIntegerLiteral`方法就能让测试通过。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.prefixParseFns = make(map[token.TokenType]prefixParseFn) p.registerPrefix(token.IDENT, p.parseIdentifier) p.registerPrefix(token.INT, p.parseIntegerLiteral)
    // [...]
}
```
当`parseIntegerLiteral`注册完毕，我们的解析器就知道如何处理`token.INT`了， 它会调用`parseIntegerLiteral`方法，然后返回`*ast.IntegerLiteral`。这样做的话，我们的测试通过了。
```
$ go test ./parser
ok monkey/parser 0.007s
```
现在已经完成标识符和整数字面值解析，让我们开始继续解析前缀操作数。


**前缀操作数**

在Monkey语言中有两种前缀操作符:`!`和`-`，它们的用法和其他编程语言一样：
```
-5;
!foobar;
5 + -10;
```
使用的方式如下：
```
<prefix operator><expression>
```
任何在前缀操作符后面的表达式都是操作数，下面这些都是有效的：
```
！isGreaterThanZero(2);
5 + -add(5, 5);
```
这也就意味着抽象语法树中前缀操作符表达式可以指向任何表达式作为它的操作数。

但是首先是测试，接下来是前缀操作符的测试用例：
```go
func TestParsingPrefixExpression(t *testing.T) {
	prefixTests := []struct {
		input        string
		operator     string
		integerValue int64
	}{
		{"!5;", "!", 5},
		{"-15;", "-", 15},
	}
	for _, tt := range prefixTests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		if len(program.Statements) != 1 {
			t.Fatalf("program.Statement doest not contain %d statements. got=%d\n",
				1, len(program.Statements))
		}
		stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
		if !ok {
			t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
				program.Statements[0])
		}
		exp, ok := stmt.Expression.(*ast.PrefixExpression)
		if !ok {
			t.Fatalf("stmt is not ast.PrefixExpression. got=%T", stmt.Expression)
		}
		if exp.Operator != tt.operator {
			t.Fatalf("exp operator is not '%s'. got=%s",
				tt.operator, exp.Operator)
		}
		if !testLiteralLiteral(t, exp.Right, tt.integerValue) {
			return
		}
	}
}
```
在测试函数中包含了很多代码，主要是两个原因：首先通过`t.Errorf`方法检查错误占据了大量的内容；还有一点是使用表驱动的测试方法。这个方法可以帮助我们省去好多测试代码。虽然只有两行代码，但是重复的编写测试代码意味着我们需要写很多重复的代码。因为测试背后的逻辑代码是相同的，所以我们共同使用测试。两个测试用例(!5和-15作为输入)不同点仅仅在于其期望的操作符和整数不同。

在测试函数中，我们迭代切片中的输入，然后对生成的抽象语法树进行判断。正如你看到的，在最后我们使用新的帮助函数`testIntegerLiteral`来测试`*ast.PrefixExpression`的`Right`字段的值是否为正确的整数。 下面就是帮助函数，这样我们可以专注于`*ast.PrefixExpression`的测试用例，不过它的字段接下我们会用到。
```go 
func testIntegerLiteral(t *testing.T, il ast.Expression, value int64) bool {
	integ, ok := il.(*ast.IntegerLiteral)
	if !ok {
		t.Errorf("il not *ast.IntegerLiteral. got=%T", il)
		return false
	}
	if integ.Value != value {
		t.Errorf("integ.Value not %d. got=%d", value, integ.Value)
		return false
	}
	if integ.TokenLiteral() != fmt.Sprintf("%d", value) {
		t.Errorf("integ.TokenLiteral not %d. got=%s", value,
			integ.TokenLiteral())
		return false
	}
	return true
}
```
上述函数并没有新奇的地方，我们之前在`TestIntegerLiteralExpression`中已经看到了，但是通过小的帮助函数可以让我们的新测试变得更叫可读。

正如我们期望的，测试还是没有通过：
```
$ go test ./parser
# monkey/parser
parser/parser_test.go:210: undefined: ast.PrefixExpression FAIL monkey/parser [build failed]
```
我们需要定义`ast.PrefixExpression`节点：
```go
//ast/ast.go
type PrefixExpression struct {
	Token    token.Token // the prefix token, e.g. !
	Operator string
	Right    Expression
}

func (pe *PrefixExpression) expressionNode()      {}
func (pe *PrefixExpression) TokenLiteral() string { return pe.Token.Literal }
func (pe *PrefixExpression) String() string {
	var out bytes.Buffer
	out.WriteString("(")
	out.WriteString(pe.Operator)
	out.WriteString(pe.Right.String())
	out.WriteString(")")
	return out.String()
}
```
同样也没有什么稀奇的，`*ast.PrefixExpression`节点有两个值得注意的字段：`Operator`和`Right`。`Operator`是一个字符串，它将会是`-`或者`!`。`Right`字段将会包含操作符右边的表达式。

虽然`*ast.PrefixExpression`已经定义，但是测试同样是失败，伴随这奇怪的错误信息
```
$ go test ./parser
--- FAIL: TestParsingPrefixExpressions (0.00s)
parser_test.go:198: program.Statements does not contain 1 statements. got=2 FAIL
FAIL monkey/parser 0.007s
```
为什么`program.Statements`包含一个语句而不是期待的两个？原因是`parseExpression`并没有认出我们的前缀操作符，仅仅返回一个`nil`，所以`program.Statements`不包含一个语句仅仅是一个`nil`。

我们可以做的更好，通过拓展我们的解析器。 `parseExpression`方法可以给出更好的错误信息：
```go
// parser/parser.go
func (p *Parser) noPrefixParseFnError(t token.TokenType) {
	msg := fmt.Sprintf("no prefix parse function for %s found", t)
	p.errors = append(p.errors, msg)
}
func (p *Parser) parseExpression(precedence int) ast.Expression {
	prefix := p.prefixParseFns[p.curToken.Type]
	if prefix == nil {
		p.noPrefixParseFnError(p.curToken.Type)
		return nil
	}
	leftExp := prefix()
	return leftExp
}
```
小小的帮助函数`noPrefixParseFnError`仅仅是在解析器的`error`字段增加了格式化的错误信息。但是一旦我们的测试失败，我们能够得到更好的错误消息。
```
$ go test ./parser
--- FAIL: TestParsingPrefixExpressions (0.00s)
parser_test.go:227: parser has 1 errors
parser_test.go:229: parser error: "no prefix parse function for ! found" FAIL
FAIL monkey/parser 0.010s
```
现在非常清楚我们需要做什么：编写前缀表达式的解析函数并且注册到我们的解析器中。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.registerPrefix(token.BANG, p.parsePrefixExpression)
    p.registerPrefix(token.MINUS, p.parsePrefixExpression)
    // [...]
}
func (p *Parser) parsePrefixExpression() ast.Expression { 
    expression := &ast.PrefixExpression{
        Token: p.curToken,
        Operator: p.curToken.Literal, 
    }
    p.nextToken()
    expression.Right = p.parseExpression(PREFIX)
    return expression 
}
```
对于`token.BANG`和`token.MINUS`，我们注册同样的方法`prefixParseFn`:它仅仅创建了`parsePrefixExpression`，该方法构建了抽象语法树的节点，在这个例子中是`*ast.PrefixExpression`，就跟我们之前看到的解析函数。但是有一点不同的是：它的确前进了`token`的位置通过调用`p.nextToken()`方法。

当`parsePrefixExpression`被调用，`p.curToken`是`token.BANG`或者`token.MINUS`其中的一个，否则它都不会被调用。为了正确解析前缀表达式比如`-5`，不止一个`token`被消费掉了。所以在使用`p.curToken`之后创建一个`*ast.PrefixExpression`节点。这个方法前进`token`然后再次调用`parseExpression`方法。这时候前缀操作符将作为参数，虽然现在还没有使用，不久我们就会看到使用这个的好处。

现在，当`parseExpression`被`parsePrefixExpression`调用的时候，输入的`token`已经前进一个并且当前的`token`是位于前缀操作符的后面。在`-5`的例子中，当`parseExpression`被调用的时候，`p.curToken.Type`是`token.INT`。然后`parseExpression`检查注册的前缀解析函数，然后发现了`parseIntegerLiteral`，它构建了`*ast.IntegerLiteral`节点并且返回。 然后`parsePrefixExpression`使用刚刚返回值填充`*ast.PrefixExpression`的`Right`字段。

现在我们的测试通过了：
```
$ go test ./parser
ok monkey/parser 0.007s
```
还记得我们之前关于我们解析函数定义的协议在这里得到实践：`parsePrefixExpression`开始于`p.curToken`(前缀操作符),然后返回的时候`p.curToken`是前缀操作数的位置，它是表达式最后一个`token`。这些`token`前进了足够的位置。这些代码足够整洁，这也是递归方法的强大之处。

当然，`parseExpression`方法中的`precedence`参数仍然非常困惑，因为目前它们还没有被使用。但是我们发现了一些比这还重要的内容：这个值得改变取决于调用者是否知道当前上下文环境。`parseExpressionStatements`对于优先级层次一无所知所以仅仅使用`LOWEST`。但是`parsePrefixExpression`传递个给`PREFIX`优先级给`parseExpression`函数，因为它要解析前缀表达式。

现在我们知道`precedence`在`parseExpression`中如何使用，因为我们接下来要解析中缀表达式。

**中缀表达式**

