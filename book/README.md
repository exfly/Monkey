**用GO语言编写解析器**

**翻译人员**  

- [gaufung](https://github.com/gaufung)

- [Jehu Lu](https://github.com/lwhile)

**目录**

- [1 前言](#ch01-introduction)
    - [1.1 Monkey 编程语言和解释器](#ch01-the-monkey-programming-language-and-interpreter)
    - [1.2 为什么使用 Go 语言](#ch01-why-go)
    - [1.3 如何使用这本书](#ch01-how-to-use-this-book)
- [2 词法分析器](#ch02-lexing)
    - [2.1 词法分析](#ch02-lexical-analysis)
    - [2.2 定义Token](#ch02-defining-our-tokens)
    - [2.3 词法分析器](#ch02-the-lexer)
    - [2.4 拓展Token集和词法分析器](#ch02-extending-our-token-set-and-lexer)
    - [2.5 REPL编写](#ch02-start-of-a-repl)
- [3 语法解析](#ch03-parsing)
    - [3.1 语法解析器](#ch03-parsers)
    - [3.2 为何不采用语法生成器](#ch03-why-not-a-parser-generator)
    - [3.3 为Monkey编程语言编写语法解析器](#ch04-writing-a-parser-for-the-monkey-programming-language)
    - [3.4 解析Let语言](#ch03-parsing-let-statement)
    - [3.5 解析Return语句](#ch03-parsing-retrun-statement)
    - [3.6 解析表达式](#ch03-parsing-expression)
    - [3.7 Pratt解析法如何工作](#ch03-how-pratt-parsing-works)
    - [3.8 拓展解析器](#ch03-extending-the-parser)
    - [3.9 REPL](#ch03-read-parse-print-loop)
- [4 执行](#ch04-evaluation)
    - [4.1 符号赋值](#ch04-giving-meaning-to-symbols)
    - [4.2 计算策略](#ch04-strategies-of-evaluation)
    - [4.3 树遍历计算](#ch04-a-tree-walking-interpreter)
    - [4.4 表达对象](#ch04-representing-objects)
    - [4.5 表达式计算](#ch04-evaluaiton-expression)
    - [4.6 条件语句](#ch04-conditionals)
    - [4.7 返回语句](#ch04-return-statement)
    - [4.8 错误处理](#ch04-error-handling)
    - [4.9 绑定和环境](#ch04-binding-and-environment)
    - [4.10 函数和函数调用](#ch04-function-and-function-call)
    - [4.11 垃圾回收](#ch04-trash-out)
- [5 拓展解释器](#ch05-extending-the-interpreter)
    - [5.1 数据类型和函数](#ch05-data-type-and-functions)
    - [5.2 字符串](#ch05-strings)
    - [5.3 内置函数](#ch05-built-in-functions)
    - [5.4 数组](#ch05-array)
    - [5.5 哈希表](#ch05-hashes)
    - [5.6 完结](#ch05-the-grand-finale)
- [6 资源](#ch06-resources)
- [7 反馈](#ch07-feedback)


<h1 id="ch01-introduction">1 前言</h1>
首先第一句话要说的应该是“解释器是具有魔法的”， 一个不愿意透露姓名的早期阅读者说道：”这听上去好像有点傻“。但是我并没有这样认为，我始终坚持解释器非常有魔力。让我一点点告诉你为什么。

表面上来看，解释器看上去误以为很简单：文本写入，得到一些东西出来。他们就是一个程序把其他的程序代码作为输入，并且生成一些东西。是不是很简单， 对吗？但是你越考虑这个问题，你就越觉得这个更加迷人。看上去随机的字符，包括字母、数字或者特殊的符号被输送到解释器后就变得有意义，这些都是解释器赋予的意义。它从无意义中发现意义，电脑只是一个建立在只能理解0和1上的机器，但是却能够理解我们输送的字符并且做出相应的操作，这些都是解释器在读取的过程中进行的翻译。

我曾经不停的问自己：解释器到底是如何工作的？当问题第一次在我脑海中形成的时候，我已近知道只有我自己写一个解释器我才能明白问题的答案。所以我就开始着手进行这件事。

有好多书籍、文章、博客或者教程是关于解释器，但是它们绝大多数涉及两个风格中的其中一个。一是涉及的主题非常宏大，难以置信的理论知识，面向那些已经非常理解这些主题的读者；另外就是非常简短，仅仅提供了简单的介绍，将外部工作当做一个黑盒子并且以玩具版的解释器为关心的重点。

其中一个基础来源就是本书后面的资源，因为解释器仅仅说明了语法简的解释型编程语言。 我并不想走捷径，我确实想知道解释器如何工作的并且理解词法分析器和句法解析器是如何工作的。尤其是类C一样带花括号和分号的编程语言，当我还不知道如何开始解析它们，那些学术上的书籍包含着我要寻找的答案。当然对我来说从哪些冗长的，理论化解释和数学符号中，我很难得到我想要的答案。

我想要的东西是介于一个900页的关于编译器的书和用50行ruby代码写一个Lisp解释器的博客之间的内容。

为了你也包括我，写了这本书。我希望这本书是为了那些喜欢一探究竟的人，亦或者是那些喜欢通过了解一些如何工作的而学习的人。

在这本书中我们将从零开始为我们自己的编程语言写一本解释器。我们将不会使用任何第三方的工作或者库。这些将不会再生产实际中使用，也不会对性能测试上做一些工作。当然，这个解释器支持的编程语言会缺失一些功能，但是我们能够从中学到很多。

由于解释器中的种类繁多并且没有很相像，所以用通用的语句描述解释器非常困难。我们能够说的就是它们共有的基础属性就是读取源代码并且计算它，没有产生一些后续执行的可视化、即可结果。编译器确实恰恰相关的，它读取源代码并且生成背后机器理解的其他代码。

有些解释器非常短小简单，甚至没有涉及解析的的步骤。它们仅仅是立马解析输入，看看类似Brainfuck中的一个就明白我说的意思。

在其他精心设计的解释器中，包含了大量高度优化和使用了先进的解析和计算技术。其中一些不去计算其中的输入，将其中编译成叫做字节码的中间表达代码，然后计算这些字节码。更先进的就是叫做JIT解释器，它将输入编译成本地机器码，然后执行。

但是，在上述表示的两种类别中，有一种解释器能够解析源代码，然后构建一颗抽象语法树(AST)然后计算这棵树。这种类型的解释器有时叫做"tree-walking"解释器，因为它就像在抽象语法树上行走然后解释它。

在这本书中，我们将构建一个"tree-walking"解释器。

我们将会构建我们的词法解析器，语法解析器，树表达式和最后的计算器。我们将会看到什么是token, 什么是抽象语法树，如何去构建这一颗树，如何计算这个树和如何去拓展我们的编程语言和一些内置函数。

<h2 id="ch01-the-monkey-programming-language-and-interpreter">1.1 Monkey 编程语言和解释器</h2>
每一个解释器都是为了解释一种特定的编程语言，这也是你如何实现一门编程语言。如果没有编译器或者解释器，任何一种一种编程语言仅仅就是一些特定的符号而已。

我们将要解析和计算的语言叫 Monkey。 这是专门为本书设计的编程语言，也是本书中的解释器实现的编程语言。

它的所有语言特性列表如下：
- C语言类似语法
- 变量绑定
- 整型和布尔型
- 算术表达式
- 内置函数
- 第一类和高阶函数
- 闭包
- 字符串类型
- 数组类型
- 散列类型

接下来我们将要具体的查看如何去实现上述的每一个功能，再次之前我们先看看 Monkey 是什么样子的。

在Monkey中我们如何将值绑定到名称上

```
let age = 1;
let name = "Monkey";
let result = 10 * (20 / 2);
```

除了整型、布尔型和字符串类型，我们构建的Monkey解释器同样也支持数组和哈希，接下来将展示整型数组是怎样的：
```
let myArray=[1, 2, 3, 4, 5];
```
接下来就是哈希，每个值对应于相应的键：
```
let thorsten = {"name": "Thorsten", "age": 28}
```
可以通过索引表达式来访问数组和哈希表中的元素
```
myArray[0]       // => 1
thorsten["name"] // => "Thorsten"
```

`let` 语句也可以用来将函数绑定变量上，接下来就是一个将连个数字相加的小函数
```
let add = fn(a, b) { return a + b; };
```
但是 Monkey 不但支持 `return` 语句， 而且支持隐式返回值，也就是说我们不使用 `return` 直接返回。
```
let add = fn(a, b) { a + b; };
```
调用函数也非常简单
```
add (1, 2);
```
展示一个更加复杂的函数，比如 `fibonacci` 函数可以返回第N个斐波那契数字。
```
let fibonacci = fn(x) {
    if (x == 0) {
        0
    } else {
        if (x == 1) {
            1
        } else {
            fibonacci(x - 1) + fibonacci(x - 2);
        }
    }
};
```
可以看到可以递归地调用 `fibonacci` 函数！

Monkey 也支持一种特殊类型的函数，叫做高阶函数。这些函数能够将其他函数作为参数，接下来就是例子
```
let twice = fn (f, x) {
    return f(f(x));
};
let addTwo = fn(x) {
    return x +  2;
};
twice(addTwo, 2);
```
在这里 `twice` 函数接受两个参数：一个叫做 `addTwo` 的函数，另一个是整数2。 它调用 `addTwo` 两次，第一次用2作为参数，第二次用第一次调用的返回值作为参数，最终生成返回6。
所以，我们可以将函数作为函数调用的参数，函数在Monkey中也就是一个值，就像整数和字符串。这种功能叫做函数是一等公民。

我们在本书中将要构建的解析器将会实现上述所有功能，在REPL中，它首先读取源代码中的token，然后解析它。然后构建一颗内部表达的抽象语法树，然后计算这颗树。它将拥有如下几个主要部分：
- 词法解析器
- 语法解析器
- 抽象语法树
- 内部对象体系
- 计算器

我们将从低到上按照这个顺序进行构建。也许可以这样中，从源代码到。最终的结果输出，但是这种方法的缺点就是在第一章结束后它不能生成一个 `Hello World`。但是它的优势就是可以很简单的理解这个部分如何组合在一起，数据是如何在整个程序中流动的。

但是我们为什么叫它 `Monkey` 呢？ 因为猴子是一个漂亮、优雅、迷人和好玩的生物，这一点跟就跟解释器一样的，也是这本书的名称的原因。


<h2 id="ch01-why-go">1.2 为何选择Go语言</h2>
如果你到目前为止还没有注意到标题的中的词Go，先第一层意思首恭喜你，非常有纪念意义；第二层意思就是我们将要使用Go语言编写解析器。那为什么我们使用Go语言呢？

我喜欢用Go写代码，我喜欢用这门语言和它提供的标准库以及工具，另外考虑的就是Go拥有一些特性对这本中的内容非常适合。

Go非常容易阅读和理解，你不需要理解这本书中编写的Go语言代码，甚至如果你不是有经验的Go语言的程序员，我打赌你能够完全理解这本书中哪怕你从来没有写过一行关于Go语言的代码。

另一个原因就是Go语言提供非常棒的工具，本书中我们编写的解释器关注的重点背后的想法的和概念。通过Go语言的格式化命令 `gofmt` 和内置的测试框架，我们可以专注与我们的解释器不用去关心第三方库、工具和依赖。在这本书中我们将不会使用任何其他的工具，仅仅使用Go语言提供的。

但是我认为更重要的是本书提供的Go语言代码与那些更底层的代码非常类似，比如C，C++和Rust。或者这跟Go语言的本身相关，更注重简洁性。或许这也是我为本书选择Go语言的原因。同样原因，这本书中将不会有任何元编程的相关技巧，虽然那样做何以走一些捷径，但是两周之后将没有人能够看懂。没有强大的面向对象设计和模式。

所有的原因都是书中的代码能够让你更好的理解（概念上的和技术层面上的）和重复使用它们。如果你在读完这本书后，打算用其他语言写自己的解释器，那将会是非常容易上手的。通过这本书，我想给你提供一个理解和构造一个解释器的起点。


<h2 id="ch01-how-to-use-this-book">1.3 如何使用这本书</h2>
这本书既不是一本参考手册，也不是关于描述实现解释器相关概念的论文的集合。这本书用来从头到尾，按照我推荐的顺序阅读，同时输入和修改提供的代码。

每一个章节是建立在先前章节上的，主要包括代码和内容。在每一章节中我们一点一点地构建我们的解释器。为了使它更容易理解，本书提供了一个叫`code`的文件夹，如果你购买的本书没有改文件夹，你可以从下面的地址下载到

[https://interpreterbook.com/waiig_cod_1.1.zip](https://interpreterbook.com/waiig_cod_1.1.zip)

`code` 文件夹被分成几个子文件夹，每一章节分为一个文件夹，其中包含了相应章节的内容。

有时我仅仅偶尔想起一些代码，但是并没有书中显示这些代码（因为他不仅仅占用了太多的空间，因为它们是一些测试文件的中的测试用例，或者仅仅是一些细节）。你能够在相应章节中找到这些代码。
接下来你需要哪些功能呢？不多，一个文本编辑器和Go编程语言，任何Go语言版本大于1.0即可工作。但是为了将来版本，我进行一些免责申明：我在编写的时候使用的是Go1.7。

我同样推荐使用[direnv](https://direnv.net)，它能根据你的 `.envrc` 文件改变你的shell环境。本书的`code`文件夹中的每一个子文件中都有一个 `.envrc`文件，它用来将`GOPATH` 添加到其子文件下中，它将允许我们不同章节下的代码都能够工作。
让我们开始行动吧！

<h1 id="ch02-lexing">2 词法分析器</h1>
<h2 id="ch02-lexical-analysis">2.1 词法分析</h2>
为了让我们的源码工作，我们需要转变一种更加容易访问的形式。就跟我们纯文本工作一样简单。它会变得更加沉重但是更快的，如果我们想要将一种编程语言转换为另外一种编程语言。

所以我们所需要做的工作就是将源代码用另外一种新式表达出来，以便更加容易工作。在执行之前，我们将会源代码的形式转变为两次。
![](../figures/transform.png)
首先要做的转换为将源代码转换为token，这个过程叫做词法分析，它是由词法分析器完成的。

token本身是非常小，分门别类的的数据结构，它们将会被送到解析器中，这将是第二步的转换，将token转换为抽象语法树。

下面是简单的例子，它是词法解析器的一个输入：
```
"let x = 5 + 5;"
```
那么从词法解析器中的输出将会是
```
[
    LET,
    IDENTIIER("x"),
    EQUAL_SIGN,
    INTEGER(5),
    PLUS_SIGN,
    INTEGER(5),
    SEMICOLON
]
```
这里每一个token都对应于源码中的一部分。 `"let"`对应着`LET`, `"+"`对应着`PLUS_SIGN`，等等如此。其余像`IDENTIFIER`和`INTEGER`对应着具体的值`5`而不是`"5"`，在`INTEGER`和`"x"`对应`INDENTIFIER`。但是具体的token构成有着不同的实现。比如，一些词法解析器将`"5"`转换为一个整型，但是其他的解析并不这样做。

还有一点值得注意的是，空白符将会被忽略。在我们的例子中是可以的，因为空白符在Monkey语言中没有意义的。比如如果我们输入如下是没有问题的:
```
let x = 5;
```
或者这么输入:
```
let x  =  5;
```
像其他语言，比如Python，空白符是有意义的，这也就意味着词法解析器不能讲它们"吃掉"，也包括换行符。词法解析器也将它们作为token输出。

一个好的的词法解析器的每个token也可能包含的行号，列号以及文件名。为什么要这么做呢？举个例子来讲，在后面的过程中如果我们解析的过程中，如果出现错误，可以输出相应的信息，而不是仅仅的输出`"error: expected semicolon token"`，可以输出：
```
"error: expected semicolon token. line 42, coloumn 23, program.monkey"
```
但是我们并不打算这么做，并不是因为这个太复杂，而是它会将我们的token和词法解析器变得复杂起来，变得更加难以理解。
<h2 id="ch02-defining-our-tokens">2.2 定义Token</h2>
首先要做的事在我们的词法解析器中定义我们的token，这些token将会被输出。开始以一些token的定义，然后在一点点增加它们。

Monkey语言的子集看上去是这样的：
```
let five = 5;
let ten = 10;
let add = fn(x, y) {
    x + y;
};
let result = add(five, ten);
```
首先将他们拆解开来，它们包含了那些类型的token?首先由数字，比如`5`和`10`，这是非常显然的。然后我们拥有变量名`x`,`y`,`add`和`result`，而且它们不是数字，仅仅是单词。但是也有些也不是变量名，比如`let`和`fn`，当然也有一些特殊的字符比如`(`，`)`，`{`，`}`，`=`，`,`，`;`，。

这些数字都是整数，我们将他们看作一种单独的类型。在我们的词法解析器或者语法解析器中不关心这个数字是5还是10，我们仅仅想知道这是一个数字。同样的道理比如变量名称。我们称它们为标识符，也将其视为同一类。现在其他的，看上去好像是标识符，但是并不是因为他们是语言的一部分，称之为关键字。我们不能将他们分组为同一类，因为当我们的解析器遇到它们的时候表现地都是不一样的。同样也是的，当我们遇到最后一个类别的时候，我们也将他们视为各自类别，因为在源码中，当遇到一个`(`和`)`的时候表现是完全不一样的。

让我们来定义我们的`token`数据结构，我们需要哪些字段？正如我们刚刚看到的，首先我们需要一个`type`树形，以便我们能够区分整数和右括号。然后还需要一个字段来保存其字面值，以便我们能够在将来能够重用它们，比如整数token它是5或者10不能丢失。

在新的`token`包中，我们定义了`token`结构体和`TokenType`类型：
```go
//token/token.go
package token
type TokenType string
type Token struct {
    Type TokenType
    Literal string
}
```
我们定义了`TokenType`为一个一个字符串，这个允许我们可以使用不同的值作为`TokenType`，它反过来可以允许我们区分不同的类型的token。使用字符串的好处是可以很容易的调试，而不需要借助其他辅助工具，我们只需要将字符串打印出来。当然，使用字符串可能导致使用整型和字节同样的性能，但是这本书使用字符串还算是完美。

正如我么你看到的，在Monkey语言中，可以使用的token类型不多。这也就意味着我们使用有限的`TokenType`类型，我们增加如下代码：
```go
//token/token.go
const (
	ILLEGAL   = "ILLEGAL"
    EOF       = "EOF"
    // Identifiers + literals
	IDENT     = "IDENT" //add, foobar, x, y, ...
    INT       = "INT" // 1343456
    // Operator
	ASSIGN    = "="
    PLUS      = "+"
    //Delimiter
	COMMA     = ","
	SEMICOLON = ";"
	
	GT        = ">"
	LPAREN    = "("
	RPAREN    = ")"
	LBRACE    = "{"
    RBRACE    = "}"
    // Keywords
	FUNCTION  = "FUNCTION"
	LET       = "LET"
)
```
如上所示，有两种特特别的类型`ILLEGAL`和`EOF`，在例子中我们并没有看到，但是我们的确需要它们，`ILLEGAL`表明了token或者的字符我们事先不清楚，`EOF`是`end of file`，它表明了我们的解析器可以停止。

接下来我们开始我们的词法解析器。
<h2 id="ch02-the-lexer">2.3 词法分析器</h2>
在开始编写代码之前，让我们说明一下本小节的目标。我们将要编写自己的词法解析器，它将源代码作为输入，并且输出整个源代码的的token。我们将重新遍历整个输入，然后依次输出每个token。这不需要缓冲区来保存token，因为这里只有一个方法叫做`NextToken()`的方法，它将依次输入下一个token。

这意味着我们将通过源码初始化词法分析器，然后让其不断得对源码调用 `NextToken()` 函数，一个 token 接着一个 token，一个字符接着另外一个字符。我们将会视源码为字符串，这会让整个过程变得简单。而且在生产环境中，将文件名和行号加到 token 中是有意义的，这样能更好得追踪解析过程和分析错误。所以最好使用 `io.Reader` 和文件名初始化词法分析器。但是因为这会增加我们处理的复杂度，所以我们不会在这里处理这些，我们将从简单的地方开始，只使用字符串并忽略文件名和行号。

目标明确后，我们需要为词法分析器做什么已经很清晰了。我们创建一个新的包，并且添加一个可以持续运行的测试，以获得词法分析器工作状态的反馈。我们可以从最小的地方开始，然后随着词法分析器功能的完善再不断添加测试用例。



```go
// lexer/lexer_test.go

package lexer 

import (
    "testing"
    
    "monkey/token"
)

func TestNextToken1(t *testing.T) {
	input := `=+(){},;`

	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
		{token.ASSIGN, "="},
		{token.PLUS, "+"},
		{token.LPAREN, "("},
		{token.RPAREN, ")"},
		{token.LBRACE, "{"},
		{token.RBRACE, "}"},
		{token.COMMA, ","},
		{token.SEMICOLON, ";"},
		{token.EOF, ""},
	}
	l := New(input)
	for i, tt := range tests {
		tok := l.NextToken()
		if tok.Type != tt.expectedType {
			t.Fatalf("tests[%d] - tokentype wrong, expected=%q, got=%q", i, tt.expectedType, tok.Type)
		}
		if tok.Literal != tt.expectedLiteral {
			t.Fatalf("tests[%d] - Literal wrong, expected=%q, got=%q", i, tt.expectedLiteral, tok.Literal)
		}
	}
}

```

当然，测试会失败 -- 因为我们还没有添加任何代码:

```shell
$ go test ./lexer
# monkey/lexer
lexer/lexer_test.go: 27 undefined: New
FAIL monkey/lexer [build failed]
```

接下来让我们以一个返回 `*Lexer` 的函数 `New()` 开始：

```go
package lexer 

type Lexer struct{
    position     int    //current character position
	readPosition int    //next character position
	ch           rune   //current character
	characters   []rune //rune slice of input string
}

func New(input string) *Lexer {
    l := &Lexer{input:input}
    return l
}
```

Lexer 的大多数字段都能自解释。有可能产生一些困惑的是 `position` 和 `readPosition`。它们都被用于访问 `input` 中字符的下标。例如：`l.input[l.readPosition]`。这两个指针指向我们的输入字符串的原因，是我们需要进一步查看输入并查看当前字符，以查看接下来会发生什么。readPosition 永远指向 input 中的下一个字符，而position 则指向 input 中的与 ch 字节相关联的字符。

第一个辅助函数被称为 `readChar()` 可以让这几个字段更容易被理解：

```go
// lexer/lexer.go
func (l *Lexer) readChar() {
	if l.readPosition >= len(l.characters) {
		l.ch = rune(0)
	} else {
		l.ch = l.characters[l.readPosition]
	}
	l.position = l.readPosition
	l.readPosition += 1
}
```

readChar 函数的目的在于返回下一个字符给我们，以及更新我们在 input 字符串的位置。它第一件要做的事是检查我们是否读到了 input 的末尾，无法再读取任何字符。如果是的话会将 `l.ch` 置为 0，对于我们来说，这个 ASCII 码代表了 “NUL” 字符，意味着我们还没有读取任何内容。但是如果我们还没有达到输入的结尾，那么它会通过访问将 `l.ch` 设置为下一个字符，然后将 `l.position` 更新到刚刚使用的 `l.readPosition` ，并且`l.readPosition` 增加1。通过这种方式，`l.readPosition` 永远指向我们将要读取的下一个位置，而 `l.position` 则永远指向我们读过的最后一个位置。

在谈论 readChar 的时候，要说明的一点是，其只支持 ASCII 码而不是整个 Unicode 集合。为什么？因为这能够让我们保持事情的简单性，并且把精力放在解释器的要点上。为了支持 Unicode 和 UTF-8，我们需要将 `l.ch` 的类型从 byte 改为 rune, 因为它们可能包含多个字节。`l.input[l.readPosition]` 也无法再继续工作。接下来我们也将会看到，一些其他的方法和函数也需要做出一点变化。让 Monkey 完整得支持 Unicode（以及 emojis!）就做为读者的一个练习吧。

接下来我们对 `New()` 函数调用 readChar，如此一来，在调用 `NextToken()`之前，我们已经初始化好了 `l.position` 和 `l.readPosition`，  `*Lexer` 进入了工作状态。

```go
func New(input string) *Lexer {
    l := &Lexer{input:input}
    l.readChar()
    return l
}
```

现在我们的测试告诉我们调用 `New(input)` 不再有任何问题，但是 `NextToken()` 方法还是没有实现。接下来让我们添加第一个版本

```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    // TODO: add more code here
}
```

这是 `NextToken()` 方法的基本结构。我们查看当前检查的字符，并根据字符串返回一个 token。在返回 token 之前需要更新我们的指针在 input 中的位置，所以当我们下次再次调用 `NextToken` 的时候，`l.ch` 已经是最新的值。一个叫做 `newToken` 的小函数帮助我们初始化那些 token。

运行测试，我们可以看到他们得到了通过：

```shell
$ go test ./lexer
ok monkey/lexer 0.007s
```

棒极了！接下来我们扩展测试用例	

```go
// lexer/lexer_test.go
func TestNextToken1(t *testing.T) {
	input := `=+(){},;`

	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
		{token.ASSIGN, "="},
		{token.PLUS, "+"},
		{token.LPAREN, "("},
		{token.RPAREN, ")"},
		{token.LBRACE, "{"},
		{token.RBRACE, "}"},
		{token.COMMA, ","},
		{token.SEMICOLON, ";"},
		{token.EOF, ""},
	}
	l := New(input)
	for i, tt := range tests {
		tok := l.NextToken()
		if tok.Type != tt.expectedType {
			t.Fatalf("tests[%d] - tokentype wrong, expected=%q, got=%q", i, tt.expectedType, tok.Type)
		}
		if tok.Literal != tt.expectedLiteral {
			t.Fatalf("tests[%d] - Literal wrong, expected=%q, got=%q", i, tt.expectedLiteral, tok.Literal)
		}
	}
}
```

值得注意的是，在这个测试中，input 已经发生了变化。它看起来像是 Monkey 语言的一个子集。它包含了所有我们已经成功转化为 token 的符号，但是一个新的事情导致我们的测试失败了：标识符、 关键字、数字。 



接下来使用标识符和关键字作为我们的开始。我们的 lexer 所需要做的就是识别当前的字符是否是一个字母，如果是，其就需要读取剩下的标识符和关键字，直到其遇到一个非标识符或者关键字，这样我们就可以使用正确 token 类型。第一步我们需要扩展我们的 switch 语句。



``` go
// lexer/lexer/go

func (l *Lexer) NextToken() token.Token {
	var tok token.Token

	switch l.ch {
	default:
		if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
			return tok
		} else {
            tok = newToken(token.ILLEGAL, l.ch)
		}
	}
	l.readChar()
	return tok
}

func (l *Lexer) readIdentifier() string {
	position := l.position
	for isIdentifier(l.ch) {
		l.readChar()
	}
    return l.input[position:position]
}

func isLetter(ch byte) bool {
    return 'a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || ch == '_'
}
```



我们为 switch 语句添加了一个默认的分支，因此，只要 `l.ch `不是识别出的字符之一，我们就可以检查标识符。我们同样可以添加类型为 `token.ILLEGAL` 的token 的生成。如果我们到此为止，我们真的不知道如何处理当前的字符并且将其声明为 `token.ILLEGAL`



辅助函数`isLetter`检查所给的参数是否是一个字母。这听起来很容易，但是要注意的是， `isLetter` 的改动会对我们的解析器解析出来语言的语言产生出乎意料的影响。正如你看到的那样，在我们例子中检查了 `ch == '_'` ，这意味着我们将 `_` 视为字母，并且允许其出现在标识符和关键字中。这表明我们可以使用类似 `foo_bar`做为变量名。其他编程语言甚至允许 ! 和 ？作为标志符。如果你也想要把它们作为标识符，这里就是你实现的地方。



`readIdentifier()` 的功能就像函数名所展现的那样，其读取一个标识符，并更新我们分析器的读取位置，直到遇到一个非字母的字符。



在 switch 语句的默认分支中，我们使用 `readIdentifier` 为我们当前的 token 设置字段 `Literal` 。但是 `Type`  字段呢？目前我们已经读取了像 `let`，`fn` 或者 `foobar`，除了语言关键字之外，我们还需要能够告诉用户定义的标识符。我们需要一个函数，它能够为我们拥有的 token 返回正确的 `TokenType`。那么除了 token 包，还有那个地方更适合添加这样一个函数呢？



```go
// token/token.go

var keywords = map[string]TokenType{
	"fn":     FUNCTION,
	"let":    LET
}

// LookupIdentifier used to determinate whether identifier is keyword nor not
func LookupIdentifier(identifier string) TokenType {
	if tok, ok := keywords[identifier]; ok {
		return tok
	}
	return IDENT
}

```



`LookupIdent` 检查通过关键字表检查所给的标识符是否是关键字。如果是，其返回关键字的常量 `TokenType`。如果其不是，则返回一个为所有用户自定义标识符的 `token.IDENT`。



有了这个函数，我们就可以完成标识符和关键字的词法分析了：



```go
// lexer/lexer.go

func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    
    switch l.ch {
    // [...]
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.Type = token.LookupIdent(tok.Literal)
            return tok
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
}
```



我们使用 `return tok` 在这里提前结束是必要的，因为当调用 `readIdentifier()` 的时候，我们会重复调用 `readChar()` 并更新我们的 `readPosition` 和 `position` 字段到当前标识符的最后一个字符。所以我们不需要在 switch 语句后再次调用 `NextToken()`



接下来运行我们的测试，我们可以检查 `let` 目前是否是标识符。不过测试还是无法通过：

```shell
$ go test ./lexer

--- FAIL: TestNextToken (0.000s)
  lexer_test.go:70: tests[1] - tokentype wrong. expected="IDENT", got="ILLEGAL"
FAIL
FAIL monkey/lexer 0.008s
```



问题出在我们下一个我们希望得到的 token：一个为 "five" 的 `IDENT` token 。实际上我们得到的是一个 `ILLEGAL` token。为什么？原因在于 “let" 与 “five" 之间隔了空格。但是在 Monkey 里空格不重要，所以我们需要整个跳过它。



```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
    var tok token.Token
    l.skipWhitespace()
    switch l.ch {
        // [...]
    }
}

func (l *Lexer) skipWhitespace() {
    for l.ch == ' ' || l.ch == '\t' || l.ch == '\n' || l.ch == '\r' {
        l.readChar()
    }
}
```



像这种辅助函数在很多分析器都能找到，有会被称为 `eatWhitespace`，有时则被称为 `consumeWhitespace`，或者其他完全不同的名字。哪些字符需要被跳过取决于被分析的语言。例如，有些语言在实现的时候会为换行符创建  token，如果它们不在 token 流中的正确位置，则将会抛出错误。为了让接下来的步骤简单些，我们暂时跳过对换行符的分析。



使用 `skipWhitespace()` 处理 `let five = 5;`时，数字 5 将会被跳过。这是正确的，因为到目前为止，该函数还不知道将数字转化为 token。接下来是添加该功能的时候了：

就像我们之前为标识符所做的一样，我们需要在 switch 语句的默认分支中添加更多的功能。



```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token() {
    var tok token.Token
    l.skipWhitespace()
    switch l.ch {
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.TYpe = token.LookupIdent(tok.Literal)
            return ok
        } else if isDigit(l.ch) {
            tok.Type = token.INT
            tok.Literal = l.readNumber()
        } else {
            tok = newToken(token.ILLEGAL, l.ch)
        }
    }
}

func (l *Lexer) readNumber() string {
    position := l.position
    for isDigit(l.ch) {
        l.readChar()
    }
    return l.input[position:l.position]
}

func isDigit(ch byte) bool {
    return '0' <= ch && ch <= '9'
}
```



正如你所看到的，添加的代码与读取标识符和关键字的部分密切相关。`readNumber` 函数与 `readIdentifier` 函数一样，只不过使用 `isDigit` 代替了 `isLetter` 实现相应的功能。我们可以通过将字符识别函数作为参数传递来概括它，但是为了简单起见并且容易理解，我们并不选择这么做。



`isDigit` 函数和 `isLetter` 函数一样简单，它只返回传入的字节是否为 0 ~ 9 的拉丁字母。

添加该函数之后，我们的测试可以通过了：

```shell
$ go test ./lexer
ok	monkey/lexer 0.000s
```



我不知道你是否注意到，我们在 `readNumber` 函数中简化了很多东西。我们只读取了整数类型，浮点类型呢？或者是用 16 进制表示的数字？又或者是 8 进制？我们忽略了它们，并且只能说 Monkey 并不支持它们。当然，原因还在于本书的教育目标和有限的范围。



是时候打开香槟做庆祝了，我们成功得将测试用例中用到的 Monkey 语言小子集成功转换为 token。



有了这个胜利，我们很容易扩展词法分析器，通过它可以标记更多的 Monkey 源代码。
<h2 id="ch02-extending-our-token-set-and-lexer">2.4 拓展Token集和词法分析器</h2>
为了在后面编写我们的的解析器的时候的需要，我们需要拓展我们的的词法分析器以便它能够识别出Monkey语言中更多的`token`。所以在本小节中我们将增加支持的`==`, `!`,`!=`,`-`,`/`,`*`,`<`,`>`以及关键词`true`,`false`,`if`,`else`和`return`。

需要新增加的`token`可以分为下面三种类别：单个字符的`token`(如`-`)，两个字符的`token`（如`==`）和关键词（如`return`）。我们已经知道如何处理单个字符和关键词`token`，所以我们首先要只是这些，然后在拓展我们词法分析器以便支持两个字符的`token`。

增加支持`-`,`/`,`*`, `<`和`>`非常普通，首先要做的是修改我们`lext/lex_test.go`中的测试用例，以便包含这些字符。正如我们之前做的，本章中的的的的所有测试代码修改是通过修改`tests`表，接下来将不会再次重复声明。
```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
`
// [...]
}
```
注意尽管输入看上像是Monkey语言的代码片段，但是一些行是没有意义上的。比如`!-/*5`,词法分析器并不会告诉我们这些代码是否有意义，这是下一个阶段所做的工作。词法分析器仅仅是将他们转换为`token`，所以我们的测试用例尽量包含所有的`token`，而且尽量包含一些错误，边界用例，换行的处理，或者多个数字的处理等等。

运行测试将会得到一些未定义的错误，因为测试包含了一些未定义的`TokenTypes`。为了修正它们，我们需要在`token/token.go`中增加下面的常量
```go
// token/token.go
const ( 
// [...]
// Operators
    ASSIGN = "="
    PLUS  = "+"
    MINUS = "-"
    BANG = "!"
    ASTERISK = "*"
    SLASH = "/"
    
    LT = "<"
    GT = ">"
//[...]
)
```
有了新的常量，测试仍然是失败的，因为我们并没有返回期望的`TokenTypes`.
```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
    lexer_test.go:84: tests[36] - tokentype wrong. expected="!", got="ILLEGAL" FAIL
FAIL monkey/lexer 0.007s
```
为了让测试通过，需要我们拓展我们的`switch`语句中的`NextToken()`方法：
```go
// lexer/lexer.go
func (l *Lexer) NextToken() token.Token { 
// [...]
    switch l.ch { 
    case '=':
        tok = newToken(token.ASSIGN, l.ch)
    case '+': 
        tok = newToken(token.PLUS, l.ch)
    case '-': 
        tok = newToken(token.MINUS, l.ch)
    case '!': 
        tok = newToken(token.BANG, l.ch)
    case '/': 
        tok = newToken(token.SLASH, l.ch)
    case '*': 
        tok = newToken(token.ASTERISK, l.ch)
    case '<': 
        tok = newToken(token.LT, l.ch)
    case '>': 
        tok = newToken(token.GT, l.ch)
    case ';': 
        tok =  newToken(token.SEMICOLON, l.ch)
    case ',': 
        tok = newToken(token.COMMA, l.ch)
// [...]
}
```
新增加的switch语句中的`case`条件记录了在`token/token.go`中常量的结构，简单的改变让测试通过了：
```
$ go test ./lexer
ok monkey/lexer 0.007s
```
现在新的单个字符的`token`已经成功添加了，下一步我们增加关键词的`true`,`false`,`if`，`else`和`return`词法分析。

再一次我们拓展测试以便包含这些新的关键词，在这里我们希望在`TestNextToken`中的`input`是这样的：
```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
    input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
if (5 < 10) { return true;
} else {
return false;
}`
// [...]
}
```
这个测试甚至不能编译，为测试中期待的类型并没有定义，为了修正它，意味着我们仅仅需要增加新的。增加`LookupIdent`来查找表：
```go
const (
//[...]
	FUNCTION  = "FUNCTION"
	LET       = "LET"
	TRUE      = "TRUE"
	FALSE     = "FALSE"
	IF        = "IF"
	ELSE      = "ELSE"
	RETURN    = "RETURN"
)

// reversed keywords
var keywords = map[string]TokenType{
	"fn":     FUNCTION,
	"let":    LET,
	"true":   TRUE,
	"false":  FALSE,
	"if":     IF,
	"else":   ELSE,
	"return": RETURN,
}
```
仅仅修稿了编译错误就能够让测试通过了：
```
$ go test ./lexer
ok monkey/lexer 0.007s
```
现在词法分析器认出了新的关键词。

在我们进入下一章节开始解析器的时候，我们仍然需要拓展我们的词法解析器，以便它能够认出由两个字符组成的`token`，我们想增加的`token`是`==`和`!=`。

你可能会想：为什么我们不通过仅仅增加`switch`语句中的`case`条件来达到目的呢？因为这个当前的字符`l.ch`作为表达式与之相违背，我们不将当个字符与`==`进行比较。

我们能够做的是重用现在的case分支`=`和`!`，并拓展它们。首先要做的是在测试中增加用例，看它们能否返回`=`和`==`。拓展之后的测试代码如下：
```go
// lexer/lexer_test.go
func TestNextToken(t *testing.T) { 
input := `let five = 5;
let ten = 10;
let add = fn(x, y) { x + y;
};
let result = add(five, ten); !-/*5;
5 < 10 > 5;
if (5 < 10) { return true;
} else {
return false;
}
10 == 10; 10 != 9; `
// [...] 
}
```
在我们进入`NextToken`方法之前，我们需要新的帮助方法`peekChar()`
```go
// lexer/lexer.go
func (l *Lexer) peekChar() byte {
    if l.readPosition >= len(l.input) {
        return 0 
    } else {
        return l.input[l.readPosition] 
    }
}
```
`peekChar`和`readChar()`方法非常像，除了它不增加`l.position`和`l.readPosition`。我们仅仅是向前看一个字符而不是移动它，我们已经知道调用`readChar()`将会返回什么。大部分的词法解析器和解析器都有类似`peek`函数来向前看，它仅仅返回下一个字符。难度在于不同的源代码中要向前看多少才有意义。

当`peekChar()`增加后，新的测试输入没法编译，当然因为使用了尚未定义的测试，再次修改也非常容易
```go
// token/token.go
const (
//[...]
    EQ = "=="
    NOT_EQ = "!="
)
```
有了定义的`token.EQ`和`token.NOT_EQ`，我们的测试返回了失败的消息：
```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
    lexer_test.go:118: tests[66] - tokentype wrong. expected="==", got="=" FAIL
FAIL monkey/lexer 0.007s
```
当词法解析器遇到`==`的时候，它创建了两个`token.ASSIGN`而不是一个`token.EQ`。解决方案是我们使用新的`peekChar()`方法，在`=`和`!`分支中使用`peek`的方法。如果下一个字符`=`我们将穿件`token.EQ`或者`token.NOT_EQ`。
```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token { 
    // [...]
    switch l.ch { 
    case '=':
        if l.peekChar() == '=' { 
            ch := l.ch
            l.readChar()
            tok = token.Token{Type: token.EQ, Literal: string(ch) + string(l.ch)} 
        } else {
            tok = newToken(token.ASSIGN, l.ch) 
        }
// [...]
    case '!':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            tok = token.Token{Type: token.NOT_EQ, Literal: string(ch) + string(l.ch)}
        } else {
            tok = newToken(token.BANG, l.ch)
        }
// [...]
}
```
注意到在调用`l.readChar()`之前，我们将`l.ch`保存到局部变量。这种方法不会丢掉挡墙的字符而且很安全的前进词法分析器，词法分析器也更新的`l.position`和`l.readPosition`。如果我们要支持更多的两个字符的`token`，我们需要抽象出方法`makeTwoCharToken`来进行分析，因为两个分支非常像。既然在在这里只有`==`和`!=`两字符`token`，让我们就丢在这里，看看能够正确工作：
```
$ go test ./lexer
ok monkey/lexer 0.006s
```
是的，测试通过了。现在词法分析器能够生成拓展后的`token`集，我们要开始准备编写解析器了。但是在做之前，我们还有一件事要做。
<h2 id="ch02-start-of-a-repl">2.5 REPL编写</h2>
Monkey语言需要`REPL`，它是`Read Eval Print Loop`的简写，你可能知道在解释型语言中都存在。Python有`REPL`, Ruby也有`REPL`, 每一个JavaScript运行时也有`REPL`，大部分`Lisp`也拥有一个。有时`REPL`也称作控制台，有时也叫做交互模式。概念都是一样的，`REPL`读入输入，把它发送到解释器中执行，打印结果和输出。如此循环往复：Read, Eval, Print, Loop。

目前我们还不知道`Eval`Monkey中的代码，我们只知道`Eval`下面的一个过程：我们能够将Monkey源代码转换为`token`。但是我们知道如何去读和输出一些东西，我还认为循环不是个大问题。

下面是一个`REPL`,它能够将Monkey源码转换为`token`，并且打印出来。我们将会拓展它来增加执行的部分：
```go
// repl/repl.go
package repl
import ( 
    "bufio"
    "fmt"
    "io" 
    "monkey/lexer" 
    "monkey/token"
)
const PROMPT = ">> "
func Start(in io.Reader, out io.Writer) { 
    scanner := bufio.NewScanner(in)
    for {
        fmt.Printf(PROMPT) 
        scanned := scanner.Scan() 
        if !scanned {
            return
        }
        line := scanner.Text() 
        l := lexer.New(line)
        for tok := l.NextToken(); tok.Type != token.EOF; tok = l.NextToken() { 
            fmt.Printf("%+v\n", tok)
        } 
    }
}
```
代码相当直接了：从输入源读取知道遇到新的一行，将得到的新的一行传递到词法解析器中，最终输出所有的`token`知道遇到`EOF`。

在`main.go`文件中，我们添加一些使用`REPL`的欢迎用语。
```go
// main.go
package main
import ( 
    "fmt"
    "os" 
    "os/user" 
    "monkey/repl"
)
func main() {
    user, err := user.Current() 
    if err != nil {
        panic(err) 
    }
    fmt.Printf("Hello %s! This is the Monkey programming language!\n", user.Username)
    fmt.Printf("Feel free to type in commands\n")
    repl.Start(os.Stdin, os.Stdout)
}
```
现在我们一个使用交互模式生成`token`
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let add = fn(x, y) { x + y; };
{Type:let Literal:let}
{Type:IDENT Literal:add}
{Type:= Literal:=}
{Type:fn Literal:fn}
{Type:( Literal:(}
{Type:IDENT Literal:x}
{Type:, Literal:,}
{Type:IDENT Literal:y}
{Type:) Literal:)}
{Type:{ Literal:{}
{Type:IDENT Literal:x}
{Type:+ Literal:+}
{Type:IDENT Literal:y}
{Type:; Literal:;}
{Type:} Literal:}}
{Type:; Literal:;}
>>
```
完美，现在开始解析这些`token`。
<h1 id="ch03-parsing">3 语法解析</h1>
<h2 id="ch03-parsers">3.1 语法解析器</h2>
每一个写过代码的人都听说过解析器，其中大部分都是`parser error`，或许也听说的比如这样“我们需要解析它”，“在它解析后“，”这个解析器搞砸了输入“。”解析器“这个词和”编译器“，
"解释器”和“编程语言”一样普通。每个人都知道解析器存在，但是它们理解是正确的吗？原因是还有谁应该对`parser error`负责。

但是什么才是真正的解析器？它的作用是什么，它应该要做些什么？下面是维基百科的[定义](https://en.wikipedia.org/wiki/Parsing#Parser):
> 解析器是软件的组成部分，它输入数据（通常为文本）并且构建数据结构，往往是一种解析树，抽象语法树或者其他继承体系结构。它们给出了输入的结构，在过程中也检查的其中语法错误。解析器的的先前步骤通常是词法分析器，它从输如的字符串序列生成token。

从维基百科中计算机科学主题的摘录的文章非常容易理解，我们甚至认出了我们词法解析器。

一个解析器将一个输入转变为一个数据结构，它代表了输入。它听上去相当抽象，所以让我用几个例子来举例说明，下面一个`JavaScript`代码：
```js
> var input = '{"name": "Thorsten", "age":28}'
> var output = JSON.parse(input)
> output
{name: 'Thorsen', age: 28}
> output.name
'Thorsen'
> output.age
28
```
我们的输入时一些文本、字符串。我们将他们传递给隐藏在`JSON.parse`函数中的解析器然后得到输出值，输出内容代表了输入：一个JavaScript对象，包含了两个字段`name`和`age`,他们的值对应着输入。现在我们可以很轻松的使用这个数据结构访问`name`和`age`字段。

但是你可能会说：”一个JSON解析器和一个编程语言的解析器是不一样的”。我想会说的是，他们是不相同的，至少不同的概念层面上的。一个JSON解析器输入文本然后构建一个数据结构来表示输入，这也是编程语言的解析器的所做的工作。不同点在于JSON解析器你可以根据输入看到数据结构，然后当你看看在这个
```js
if ((5+2*3)==91) { return computerStuff(input1, input2); }
```
这个数据结构当然不能一下子看出来，这也是为什么，至少对于我来讲，它的不同点在于概念层面上更深。我猜测这概念层面上的不同主要在于缺乏对于编程语言解析器的熟悉程度和它生成的数据结构。相对于编写编程语言，我有更多的很多写JSON的经验，通过解析器解析它，查看它的输出和结构。作为编程语言的使用者，我们很少看到和接触过解析后的源代码，以及中间的表示形式。Lisp程序员是这个规则的例外，在Lisp代码中，表示源码的数据结构和Lisp用户是同样的。解析后的源代码和和程序的数据是一样的。"代码就是数据，数据就是代码”这句话你有时会从Lisp程序员嘴中说出。

所以为了让我们对编程语言解析器的概念层面的理解提升到我们对一些序列化语言（比如JSON，YAML, TOML, INI等等），我们需要理解他们生成的数据结构。

在大多数解析器和编译器中，源代码中的中间表示形式的数据结构叫做语法树或者抽象语法树。“抽象”的概念基于这样一个事实，在抽象语法树中代码的详细细节我们忽略掉了。分号、换行、空白、注释、花括号、中括号和括号这些都是依赖于语言，解析器并不会在抽象语法树中表示它们，它们仅仅指导如何去构建这颗树。

一个事实需要主要的是，没有一个唯一正确的、通用的抽象语法树适用于每一个解析器。它们实现都是比较类似、概念上是一样的，但是细节上有所不同。具体的实现取决于它要解析的编程语言。

一个小例子可以说清楚，加入说我们有如下的源代码：
```js
if ( 3 * 5 > 10) {
    return "hello";
}else{
    return "goodbye";
}
```
假设我们使用`JavaScript`，拥有一个`MagicLexer`解析器，它能构建出一个用JavaSrcipt对象构成的抽象语法树，那么解析过程中将会生成如下的内容:
```js
> var input = `if (3 * 5 > 10) { return "hello";} else { reutrn "goodby" ;}`
> var tokens = MagicLexer.parse(input)
> MagicLexer.parser(tokens);
{
    type: "if-statement",
    condition :{
        type: "operator-expression",
        operator : ">",
        left: {
            type: "operator-exression",
            operator: "*",
            left: {
                type: "integer-literal", value : 3
            },
            right :{
                type: "integer-literal", value: 5
            }
        },
        right:{
            type:"integer-literal", value: 10
        }
    },
    consequence: {
        type: "return-statement",
        returnValue: {type: "string-literal", value: "hello"}
    },
    alternative:{
        type: "return-statement",
        returnValue: {type: "string-literal", value: "goodbye"}
    }
}
```
正如你看到的，解析器输出的抽象语法树，相当抽象。在这里没有括号，没有冒号和花括号。但是它很精确的表示出了源代码，你认为呢？我打赌你当你看源代码的时候就能看到抽象语法树了。

所以这就是解析器所做的工作。它将源代码作为输入（文本和token都可以）然后输出数据结构，它代表了源代码。当构建数据结构的时候，它们不可避免地分析了输入，检查它们是否满足期待数据结构，因此解析的过程也被称为语法分析。

在本章中，我们将为我们的Monkey编程语言编写解析器。它的输入将会我们前面章节定义好的`token`,它们是有词法分析器完成，我们将会定义我们自己的抽象语法树，它将适应我们的解析器，然后构建一个抽象语法树的实例通过递归解析`token`。
<h2 id="ch03-why-not-a-parser-generator">3.2 为何不采用语法生成器</h2>

或许你已经听说过解析器生成器，就像很多工具比如`yacc`, `bison`或者`ANTLR`。解析器生成器通过读取语言的一种描述形式，将解析器输出的工具。输出的代码可以被编译或者解释，然后读取源代码就能生成语法树。

有很多种解析器生成器，主要不同在于输入的形式不同和输出的语言。他们大部分使用上下文无关文法（*context-free grammer*，CFG）作为输入，CFG是一系列规则，它们描述了如何构建正确的语句，最常见的CFG的形式是`Backus-Nau Form(BNF)`或者`Extend Backus-Nau Form(EBNF)`。
```bnf
PrimaryExpression ::= "this"
                    | ObjectLiteral
                    | ("(" Exression ")")
                    | Identifier
                    | ArrayLiteral
                    | Literal
Literal ::=( <DECIMAL_LITERAL>
        | <HEX_INTEGER_LITERAL>
        | <STRING_LITERAL>
        | <BOOLEAN_LITERAL>
        | <NULL_LITERAL>
        | <REGULAR_EXPRESSION_LITERAL> )
Identifier ::= <IDENTIFIER_NAME> 
ArrayLiteral ::= "[" ( ( Elision )? "]"
            | ElementList Elision "]"
            | ( ElementList )? "]" ) 
ElementList ::= ( Elision )? AssignmentExpression 
                ( Elision AssignmentExpression )*
Elision ::= ( "," )+
ObjectLiteral ::= "{" ( PropertyNameAndValueList )? "}"
PropertyNameAndValueList ::= PropertyNameAndValue ( "," PropertyNameAndValue
                        | "," )* 
PropertyNameAndValue ::= PropertyName ":" AssignmentExpression
PropertyName ::= Identifier
                | <STRING_LITERAL>
                | <DECIMAL_LITERAL>
```
这是`EcmaScirpt`语法使用BNF的[全部描述](http://tomcopeland.blogs.com/EcmaScript.html)。 一个解析器生成器将这些类似的东西转变为C代码。

获取你也听说过你应该使用解析器生成器而不是自己手动写。可以跳过本章节，为题已经解决了。这样做的原因是解析器已经自动帮你做好了。解析过程是计算机科学最容易理解的的分支，很多聪明的人在上面投资了大量的时间，它们的成果有CFG，BNF，EBNF，解析器生成器和一些先进的解析技巧，那你为什么不充分利用这些成果呢？

但是我认为学习编写自己的解析不是浪费时间，我事实上认为价值巨大。只有自己写过自己的解析器，至少尝试过，你才会看到这些解析器生成器提供的便利，它们拥有的缺陷以及它们解决的问题。 对我来讲，当我饿写完自己第一个解析器， 解析器生成器的概念就像鼠标单击一样。只有你真正理解它是怎么工作的的，才知道它们如何转换代码的。

但多数人建议使用解析器生成器的人，当然们开始一个解析器或者编译器的时候这样做的原因是他们之前编写过解析器。它们已经见识过问题和可行的解决的解决方案，它们决定使用现成的工具，它们是正确的，当你希望在生产环境中使用解析器因为正确性和健壮性是首要考虑的目标。当然你不应该尝试些一个解析器，尤其当你从未编写过。

但是我们在这我们将要学习去编写，我们想要知道解析器如何工作的。我的观点是当我们做一些事的最好的方法是动手去做，自己去编写一个解析器是相当有趣的。
<h2 id="ch04-writing-a-parser-for-the-monkey-programming-language">3.3 为Monkey编程语言编写语法解析器</h2>
当解析编程语言的时候有两种策略：至顶向下解析和至底向上解析。每一个不同的策略都有微小的不同。比如：递归下降解析，提前解析或者预测解析都是至顶向下解析的变种。

在这里我们将要编写的是递归下降解析，更确切的说是一种“至顶向下操作符优先级”解析器。有时候也叫做`Pratt`解析器，根据发明人`Vaughan Pratt`命名。

我将不会涉及到不同解析策略的细节中去，因为这样既占用空间而且我也不能足够精确的描述它们。让我来讲，至顶向下解析和至底向上解析的不同点在于，前者从抽象语法树的根节点开始，而后者是恰恰相反。递归下降解析器至顶向下解析， 它经常被推荐给新手，因为它和我们想象中构建抽象语法树构建是一样的。我个人发现递归方法开始于根节点非常棒，尽管它需要在理解概念之前编写代码。这也是我为什么开始编写代码，而不是探索解析策略。

现在我们自己编写解析器，我们不得不做出一些取舍。我们的解析器将不会永远都很快。我们也没有正式的证明解析器的正确性。而且错误恢复和语法错误提示将不会提供。最后一点要澄清的是弄清楚解析器是非常困难的，如果没有额外的学习相关理论知识。但是我们将会拥有一个完全能够工作的解析器，并且开放了拓展和提高的部分，如果你对此非常感兴趣，你将会拥有了一个很好的理解。

我们将会从解析语言开始：`let`和`return`语言。当我们能够解析语句和基本的基本的结构，我们将会转向表达式并且学习如何解析它们（这是也是`Vaughan Pratt`将会发挥作用）。最后我们将拓展解析器，使它能够解析Monkey语言的更大的子集。最后我们将构建一颗抽象语法树。
<h2 id="ch03-parsing-let-statement">3.4 解析Let语言</h2>
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

有了这些AST基础模块定义，让我们想想什么样的节点能够表示`let x = 5;`，会有那些字段？定义一个变量的名称，同样我们也需要一个字段来表示等号右边的表达式，它可以指向任何表达式。它不只是指向字面值（在这个例子中是5），因为任何等号右边表达式都是相同的，比如：`let x = 5 * 5;`和`let y = add(2, 2) * 5 / 10;` 都是有效的。为了跟踪记录抽象语法树中的每一个节点，我们需要实现`TokenLiteral()`方法。它有三个字段构成：一个是标识符，一个是可以生成值的表达式，最后是token。

```go
// ast/ast.go
import "monkey/token"
// [...]
type LetStatement struct {
    Token token.Token
    Name *Identifier
    Value Expression
}
func (ls *LetStatement) statementNode() {}
func (ls *LetStatement) TokenLiteral() string { return ls.Token.Literal}

type Identifier struct {
    Token token.Token // the token.IDENT token
    Value string
}
func (i *Identifier) expressionNode() {}
func (i *Identifier) TokenLiteral() string { return i.Token.Literal }
```
`LetStatement`拥有以下字段：`Name`用来保存绑定的标识符；`Value`用来保存表达式生成的值。`statementNode`和`TokenLiteral`方法用来满足`Statement`和`Node`接口。

为了保存标识符绑定值，`let x= 5;`中的`x`。我们需要定义`Identifier`结构，它实现了`Expression`接口，但是在let语句中它不生成值，为什么我们定义一个`Expression`接口，主要是简化问题。标识符在Monkey语言其他部分中会生成值，比如`let x = valueProducingIdentifier;`中，为了让数据类型数量变少，我们将使用`Identifier`来表示被绑定的值，以便后来重用它们，所以标识符实现了expression接口。

`Program`，`LetStatement`和`Identifier`构成了Monkey语言的源代码一部分：
```go
let x = 5;
```
它可以用抽象语法树表示如下：
![](../figures/ast1.png)
现在我们知道它应该长什么样，下一个任务就是构建一棵抽象语法树，首先我们的解析器是这样的：

```go
//parser/parser.go
package parser
import (
    "monkey/ast"
    "monkey/lexer"
    "monkey/token"
)
type Parser struct {
    l *lexer.Lexer
    curToken token.Token
    peekToken token.Token
}
func New(l *lexer.Lexer) *Parser {
    p := &Parser{l: l}
    //Read two tokens, so curToken and peekToken are both set
    p.nextToken()
    p.nextToken()
    return p
}
func (p *Parser) nextToken(){
    p.curToken = p.peekToken
    p.peekToken= p.l.NextToken()
}
func (p *Parser) ParserProgram() *ast.Program{
    return nil
}
```
`Parser`拥有三个字段，分别为`l`，`curToken`和`peekToken`。其中`l`指向词法分析器的一个实例，通过它可以不听的调用`NextToken`方法来获取输入的下一个`token`。`curToken`和`peekToken`就像我们词法解析器拥有的两个指针`position`和`peekPosition`，但是它不是指向输入的字符，而是指向当前的`token`和下一个`token`，它们是同样重要的：我们需要查看当前输入的`token`，通过检查当前的`token`，来确定下一步需要做什么；同样也需要`peekToken`如果`curToken`不能做出相应的决定。比如考虑但单行语句`5;`，当前的`curToken`是`token.INT`，我们需要`peekToken`来决定我们是否在一行代码的末尾还是仅仅是算术表达式的开始。

`New`函数非常明了，`nextToken`是一个小的帮助方法，它用来同时前进`curToken`和`peekToken`指针。现在目前`ParseProgram`是空的。

在我们编写测试和填充`ParseProgram`方法之前，我先向你展示一下递归下降分析背后的基础思想和结构。它可以帮助我们很好的理解后面的分析器。接下来主要部分是解析器的伪代码。仔细阅读它并且尝试去理解`parseProgram`函数里发生了什么。
```js
function parseProgram() { 
    program = newProgramASTNode()
    advanceTokens()
    for (currentToken() != EOF_TOKEN) {
        statement = null
        if (currentToken() == LET_TOKEN) { 
            statement = parseLetStatement()
        } else if (currentToken() == RETURN_TOKEN) { 
            statement = parseReturnStatement()
        } else if (currentToken() == IF_TOKEN) { 
            statement = parseIfStatement()
        }
        if (statement != null) { 
            program.Statements.push(statement)
        }
        advanceTokens() 
    }
    return program
}
function parseLetStatement() { 
    advanceTokens()
    identifier = parseIdentifier()
    advanceTokens()
    if currentToken() != EQUAL_TOKEN { 
        parseError("no equal sign!") 
        return null
    }
    advanceTokens()
    value = parseExpression()
    variableStatement = newVariableStatementASTNode() 
    variableStatement.identifier = identifier 
    variableStatement.value = value
    return variableStatement
function parseIdentifier() { 
    identifier = newIdentifierASTNode() 
    identifier.token = currentToken() 
    return identifier
}
function parseExpression() {
    if (currentToken() == INTEGER_TOKEN) {
        if (nextToken() == PLUS_TOKEN) { 
            return parseOperatorExpression()
        } else if (nextToken() == SEMICOLON_TOKEN) { 
            return parseIntegerLiteral()
        }
    } else if (currentToken() == LEFT_PAREN) {
        return parseGroupedExpression() 
    }
        // [...]
}
function parseOperatorExpression() {
    operatorExpression = newOperatorExpression()
    operatorExpression.left = parseIntegerLiteral() 
    operatorExpression.operator = currentToken() 
    operatorExpression.right = parseExpression()
    return operatorExpression() 
}
// [...]
```
虽然上面的伪代码中有很多省略，但是递归下降解析的基础思想是有的。入口是`parseProgram`函数，它构造了抽象语法树的根节点，然后通过调用其他函数构建孩子节点，语句。这些抽象语法树是基于当前`token`，其他函数也是相互调用，递归执行。

最递归调用的部分在`parseExpression`中，但是我们已经知道为了解析表达式`5 + 5`，我们需要首先解析`5 +`然后再一次调用`parserExpression()`来解析剩下来的部分，因为在`+`后面可能是其他操作符表达式，比如`5+5*10`。我们将会接下来仔细查看解析该表达式的细节，因为他是解析中最复杂的部分同样也是最漂亮的部分，也就是`Pratt`解析。

但是到现在，我们已经知道解析器将要做什么。它不停地读取`tokens`然后检查当前`token`来决定接下来需要做什么：是调用其他的解析函数呢还是抛出一个异常。每一个函数都有各自的任务，可能创建抽象语法树的节点。所以在`parseProgram()`中的主循环可以不停前进`token`来决定接下来做什么。

如果你看了上述的伪代码，你可能会想：这看上去很容易理解嘛！我有一个很好的消息告诉你，我们`ParseProgram`方法和解析器看上去很类似，让我们开始工作吧！

再一次，在我们编写`parseProgram`方法，我们可以编写测试。下面的测试可以确保`let`语句解析正确。

```go
// parser/parser_test.go
package parser

import (
	"fmt"
	"monkey/ast"
	"monkey/lexer"
	"testing"
)

func TestLetStatements(t *testing.T) {
	tests := []struct {
		input              string
		expectedIdentifier string
		expectedValue      interface{}
	}{
		{"let x =5;", "x", 5},
		{"let z =1.3;", "z", 1.3},
		{"let y = true;", "y", true},
		{"let foobar=y;", "foobar", "y"},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain 1 statements. got=%d",
				len(program.Statements))
		}
		stmt := program.Statements[0]
		if !testLetStatement(t, stmt, tt.expectedIdentifier) {
			return
		}
		val := stmt.(*ast.LetStatement).Value
		if !testLiteralExpression(t, val, tt.expectedValue) {
			return
		}
	}
}

func testLetStatement(t *testing.T, s ast.Statement, name string) bool {
	if s.TokenLiteral() != "let" {
		t.Errorf("s.TokenLiteral not 'let'. got %q", s.TokenLiteral())
		return false
	}
	letStmt, ok := s.(*ast.LetStatement)
	if !ok {
		t.Errorf("s not *ast.LetStatement. got=%T", s)
		return false
	}
	if letStmt.Name.Value != name {
		t.Errorf("s.Name not '%s'. got=%s", name, letStmt.Name.Value)
		return false
	}
	if letStmt.Name.TokenLiteral() != name {
		t.Errorf("s.Name not '%s'. got=%s", name, letStmt.Name.Value)
		return false
	}
	return true
}
```
测试用例遵循我们先前词法解析器的测试规则，也是和其他每一个单元测试同样：我们提供Monkey代码作为输入，然后设置我们想要的抽象语法树的期待值，抽象语法树是由有解析器生成的。我们将会尽可能的检查抽象语法树的节点，确保我们没有丢失任何东西。

我将不选择`Mock`或者`Stub`方法而是提供源代码作为输入而不是tokens，因为那样做的话将会让我们的测试可读和可理解。当然我们的词法解析器中的bug将会使我们的测试变得面目全非并且生成很多噪声。但是我尽量将这个风险降到最低，尤其是当比较实用源代码作为输入的时候的优势的的时候。

在这里测试用例两个需要特别注意，第一个是我们忽略了`*ast.LetStatement`中的`Value`字段，为什么我们不去检查我们数字解析得是否正确呢？答案是我们是要做的，但是我们需要首先确保在解析`let`语句正确执行，所以忽略了`Value`字段。

第二个是帮助函数`testLetStatement`，它看上去使用单独函数像是过度设计化，但是我们在将来很需要这个函数，它能够将我们的测试用例变得可读而不是一行行的类型转换。

除此之外，在本章中我们将不会再次看全部的解析器测试，因为他们太长了，但是本书提供的代码包含全部代码。

在开始之前，测试时失败的：
```
$ go test ./parser
--- FAIL: TestLetStatement (0.00s)
parser_test.go:20: ParseProgram() return nil
FAIL
FAIL monkey/parser 0.007s
```
是时候开始填充我们的`ParseProgram()`方法了：
```go
// parser/parser.go
func (p *Parser) ParseProgram() *ast.Program{
    program := &ast.Program{}
    program.Statements = []ast.Statement{}
    for p.curToken.Type != token.EOF {
        stmt := p.parseStatement()
        if stmt != nil {
            program.Statements = append(program.Statements, stmt)
        }
        p.nextToken()
    }
    return program
}
```
是不是看上去和`parseProgram()`伪代码函数非常相像？是的，我之前告诉你着很像。

`parseProgram()`首先要做的事构建抽象语法树的根节点：`*ast.Program`，然后他迭代每一个`token`直至遇到`token.EOF`。它重复调用`nextToken`方法，该方法能够同时前进`p.curToken`和`p.peekToken`两个指针，每一次迭代调用`parseStatement`方法，它负责解析语句。如果它返回值不是`nil`而是`ast.Statement`，那么返回值将会添加到根节点的`Statements`切片中，如果没有任何`token`可供解析，那么`*ast.Program`将会被返回。

那么`pasreStatement`方法看上去是这样的：
```go
// parser/parser.go
func (p *Parser) parseStatement() *ast.Statement{
    switch p.curToken.Type{
    case token.LET:
        return p.parseLetStatement()
    default:
        return nil
    }
}
```
不用担心，`switch`语句将会得到更多的分支，但是现在，它仅仅地调用`parseLetStatement`方法如果只遇到`token.LET`。`parseLetStatement`方法将会让我们的测试通过：
```go
//parser/parser.go
func (p *Parser) parseLetStatement() *ast.LetStatement {
    stmt := &ast.LetStatement{Token: p.curToken}
    if !p.expectPeek(Token.IDENT){
        return nil
    }
    stmt.Name = &ast.Identifer{Token:p.curToken, Value: p.curToken.Literal}
    if !p.expectPeek(toekn.ASSIGN) {
        return nil
    }
    // TODO: We're skipping the expression util we
    // encoutner a semicolon
    for !p.curTokenIs(token.SEMICOLON){
        p.nextToken()
    }
    return stmt
}
func (p *Parser) curTokenIs(t token.TokenType) bool { 
    return p.curToken.Type == t
}
func (p *Parser) peekTokenIs(t token.TokenType) bool { 
    return p.peekToken.Type == t
}
func (p *Parser) expectPeek(t token.TokenType) bool { 
    if p.peekTokenIs(t) {
        p.nextToken()
        return true 
    } else {
        return false
    } 
}
```
它奏效了，我们的测试通过了
```
$ go test ./parser
ok monkey/parser 0.007s
```
我们可以解析`let`语句了，是不是很神奇，但是等等！

让我们从`parseLetStatement`开始，它使用当前指向的`token`(`token.LET`)构建了`*ast.LetStatement`节点，然后调用`expectPeek`方法确保下一个`token`是否符合预期。首先它期待`token.IDENT`，然后使用它来构建一个`*ast.Identifer`节点，然后期待一个等号，最后它跳过了等号后面的表达式解析直到要一个冒号。只要我们知道如何解析表达式，跳过的部分将会被替换。

`curTokenIs`和`peekTokenIs`方法不要太多的解释，它们非常有用，以至于我们在填充我们的解析器的时候一次次看到它们，我们可以用`!p.curToken(token.EOF)`代替for循环中`p.curToken.Type != token.EOF`条件。

除了看看这些小方法， 让我们讨论一下`expectPeek`方法，`expectPeek`方法也是一个判断函数，它们原先的的目的是确保解析过程中`Token`的顺序是否正确。 我们`expectPeek`方法用来检查`peekToken`类型是否满足要求，只有满足要求才会调用`nextToken`方法。 正如你看到的，这个在解析器中频繁出现。

但是如果我们遇到的`token`在`expectPeek`方法中不是期待的类型？目前，我们就是返回`nil`，在`ParseProgram`方法中将会忽略该空值。它会导致该语句会被忽略，因为错误的输入。你可以想象一下，它将会使得调试变得非常困难，因为没有人想接触那些没有么错误错误的解析器。

幸运的是，我们将会做一些微小的改动来将解决该问题：
```go
// parser/parser.go
type Parser struct { 
// [...]
errors []string 
// [...]
}
func New(l *lexer.Lexer) *Parser { 
    p := &Parser{
        l: l,
        errors: []string{}, 
    }
// [...]
}
func (p *Parser) Errors() []string { 
    return p.errors
}
func (p *Parser) peekError(t token.TokenType) {
    msg := fmt.Sprintf("expected next token to be %s, got %s instead", t, p.peekToken.Type)
    p.errors = append(p.errors, msg) 
}
```
现在解析器吼了`errors`字段，它们就是字符串切片，该字段在`New`方法的时候被初始化，当`peekToken`不匹配的时候，帮助函数`peekError`用来增加错误到`errors`中，通过使用`Errors`方法可以检查我们的解析器是否遇到任何错误。

拓展我们的测试来确保这个和我们期望的是一样的：
```go
// parser/parser_test.go
func TestLetStatements(t *testing.T) { 
// [...]
    program := p.ParseProgram() checkParserErrors(t, p)
// [...]
}
func checkParserErrors(t *testing.T, p *Parser) { 
    errors := p.Errors()
    if len(errors) == 0 {
        return
    }
    t.Errorf("parser has %d errors", len(errors)) 
    for _, msg := range errors {
        t.Errorf("parser error: %q", msg) 
    }
    t.FailNow() 
}
```
新的`checkParserErrors`帮助函数仅仅是我们的解析器中的错误，如果有任何错误，它将错误输出并且停止当前测试。

但是我们的当前解析器中没有生成任何错误，通过改变`epectPeek`我们可以自动增加一个错误如果我们调用得到错误的`token`。
```go
// parser/parser.go
func (p *Parser) expectPeek(t token.TokenType) bool { 
    if p.peekTokenIs(t) {
        p.nextToken()
        return true 
    } else {
        p.peekError(t)
        return false
    }
}
```
现在我们可以改变我们的测试用例从这样：
```
input := `
let x = 5;
let y = 10;
let foobar = 838383;
`
```
变成每个一个输入都是无效的，因为都是了相关的`token`
```
input := ` let x 5;
let = 10; 
let 838383; `
```
我们运行我们的测试可以看到新的解析错误
```
$ go test ./parser
--- FAIL: TestLetStatements (0.00s)
parser_test.go:20: parser_test.go:22: got INT instead" parser_test.go:22:
got = instead" parser_test.go:22: got INT instead"
parser has 3 errors
parser error: "expected next token to be =,\
parser error: "expected next token to be IDENT,\ parser error: "expected next token to be IDENT,\
FAIL
FAIL monkey/parser 0.007s
```
正如你看到的，我们的解析器展示的一些微小的功能：它输出它遇到的每一个错误的语句。它没有在遇到第一个的时候就退出了，而是将他们保存下来以便我们重新运行程序去获取全部语法错误。它相当有用，尽管我们没有错误的行号和列号。
<h2 id="ch03-parsing-retrun-statement">3.5 解析Return语句</h2>
我先前说过，我们会一步步填充看上去很空旷的`ParseProgram`方法。现在就是时候，我们将要解析`return`语句。在解析之前要做的就是在`ast`包中定义额必要的结构以便我们在抽象语法树中能够表示它们。

接下来是在Monkey语言中的`return`语句：
```
return 5;
return 10;
return add(15);
```
有了`let`语句的经验，我们可以很容易地看出`return`语句的结构：
```
return <expression>;
```
`return`语句由单个`return`关键字和一个表达式组成，这样定义`ast.ReturnStatement`也就是非常简单的了：
```go
// ast/ast.go
type ReturnStatement struct {
    Token token.Token // the 'return' token
    ReturnValue Expression
}
func (rs *ReturnStatement) statementNode() {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```
上述的节点定义中，所有出现的内容你都看过：它有一个字段用来初始化`token`和`ReturnValue`字段来表示将要返回的表达式。现在我们会跳过解析表达式过程，只处理分号，后面我们会回来处理该部分的。`statementNode`和`TokenLiteral`方法使之能够满足`Node`和`Statement`接口。而且这些方法是定义在`*ast.LetStatement`上的。

接下来的测试部分和之前的`let`语句也是差不多相同。
```go
// parser/parser_test.go
func TestReturnStatements(t *testing.T) { 
    input := `
return 5; 
return 10; 
return 993322; `
    l := lexer.New(input) p := New(l)
    program := p.ParseProgram() checkParserErrors(t, p)
    if len(program.Statements) != 3 {
        t.Fatalf("program.Statements does not contain 3 statements. got=%d", len(program.Statements))
    }
    for _, stmt := range program.Statements {
    returnStmt, ok := stmt.(*ast.ReturnStatement) 
    if !ok {
        t.Errorf("stmt not *ast.returnStatement. got=%T", stmt)
        continue
    }
    if returnStmt.TokenLiteral() != "return" {
        t.Errorf("returnStmt.TokenLiteral not 'return', got %q", returnStmt.TokenLiteral())
    } 
}
```
当然测试会进行拓展只要表达式解析完成即可。但是没关系，测试不是一成不变的。但是现在测试时失败的：
```
$ go test ./parser
--- FAIL: TestReturnStatements (0.00s)
parser_test.go:77: program.Statements does not contain 3 statements. got=0 FAIL
FAIL monkey/parser 0.007s
```
所以现在通过修改`ParseProgram`方法将`token.RETURN`考虑进去就能够让测试通过：
```go

// parser/parser.go
func (p *Parser) parseStatement() ast.Statement { 
    switch p.curToken.Type {
    case token.LET:
        return p.parseLetStatement() 
    case token.RETURN:
        return p.parseReturnStatement() 
    default:
        return nil
    } 
}
```
接下来就是`parseReturnStatement`方法，该方法也是非常简单，没有很多的复杂的地方：
```go
// parser/parser.go
func (p *Parser) parseReturnStatement() *ast.ReturnStatement { 
    stmt := &ast.ReturnStatement{Token: p.curToken}
    p.nextToken()
    // TODO: We're skipping the expressions until we 
    // encounter a semicolon
    for !p.curTokenIs(token.SEMICOLON) {
        p.nextToken() 
    }
    return stmt 
}
```
它唯一做的事情就是构建一个`ast.ReturnStatement`对象，当前的`token`就是作为`Token`字段，然后通过调用`nextToken()`方法来解析表达式，但是现在并没有做，它跳过了每一个表达式直至遇到一个分号，就这样我们的测试通过了：
```
$ go test ./parser
ok monkey/parser 0.009s
```
可以再一次庆祝了，我们现在可以解析Monkey语言中的所有语句了。是的，在`Monkey`中只有两种语句：`Let`语句和`Retrun`语句。接下来我么我们的编程语言中只包含了表达式，接下来我们将会解析它们。
<h2 id="ch03-parsing-expression">3.6 解析表达式</h2>
主观的来讲，解析表达式是编写解析器中最有趣的部分。正如我们刚刚看到的，解析语句相对比较直接。我们从左到右处理每个`token`，期待或者拒绝下一个token直到我们返回抽象语法树的节点。

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

接下来我们将要解析下面八种中缀操作符
```
5 + 5; 
5 - 5; 
5 * 5; 
5 / 5; 
5 > 5; 
5 < 5;
5 == 5; 
5 != 5;
```
不要被这些`5`所打扰，正如前缀表达式一样，这里我们可以在左边和右边使用任何表达式。
```
<expression><infix operator><expression>
```
因为左右两边两个操作数，因此有时这种表达式又称为二元表达式，与此相反的叫做一元表达式。尽管我们可以在操作符的两边使用任何表达式，但是我们打算编写一些测试仅仅包含整型字面值。只要我们能够让测试通过，我们将会拓展它们至更多的操作数类型。下面是测试：
```go
func TestParsingInfixExpression(t *testing.T) {
	infixTests := []struct {
		input      string
		leftValue  int64
		operator   string
		rightValue int64
	}{
		{"0.4+1.3", 0.4, "+", 1.3},
		{"5+5;", 5, "+", 5},
		{"5-5;", 5, "-", 5},
		{"5*5;", 5, "*", 5},
		{"5/5;", 5, "/", 5},
		{"5>5;", 5, ">", 5},
		{"5<5;", 5, "<", 5},
		{"5==5;", 5, "==", 5},
		{"5!=5;", 5, "!=", 5},
	}
	for _, tt := range infixTests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain %d statements. got=%d\n",
				1, len(program.Statements))
		}
		stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
		if !ok {
			t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
				program.Statements[0])
		}
		if !testInfixExpression(t, stmt.Expression, tt.leftValue, tt.operator, tt.rightValue) {
			return
		}
	}
}
```
这些测试直接从`TestParsingPrefixExpressions`拷贝过来的，除了我们现在结果的抽象语法树的节点的左边和右边都做了相等判断。在这里同样也是用表驱动测试方法，以便我们将来能够对它们进行拓展。

当然测试是失败的，当然我们没有发现`*ast.InfixExpression`的定义。为了真正让测试失败，我们定义`ast.InfixExpression`:
```go
//ast/ast.go
type InfixExpression struct {
	Token    token.Token
	Left     Expression
	Operator string
	Right    Expression
}

func (ie *InfixExpression) expressionNode()      {}
func (ie *InfixExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *InfixExpression) String() string {
	var out bytes.Buffer
	out.WriteString("(")
	out.WriteString(ie.Left.String())
	out.WriteString(" " + ie.Operator + " ")
	out.WriteString(ie.Right.String())
	out.WriteString(")")
	return out.String()
}
```
就跟之前`ast.PrefixExpression`，我们定义`ast.InfixExpression`，通过定义`expressionNode()`，`TokenLiteral()`和`String()`实现了`ast.Expression`和`ast.Node`接口。唯一的不同是`ast.PrefixExpression`有了新的字段`Left`，它可以保存任何表达式。

通过这种方式，我们可以运行测试。现在测试返回新的错误信息：
```
$ go test ./parser
--- FAIL: TestParsingInfixExpressions (0.00s)
parser_test.go:246: parser has 1 errors
parser_test.go:248: parser error: "no prefix parse function for + found" FAIL
FAIL monkey/parser 0.007s
```
但是错误信息很清晰：它说"没有找到为加号处理的解析函数”，但是问题在于我们并不想让我们的解析器找到前缀解析函数，我们想找到中缀解析函数。

到这个点上，我们将会从好奇转向惊叹，因为我们现在要完成的`parseExpression`方法。为了完成它，我们首先需要一个优先级表和一些帮助方法。
```go
// parser/parser.go
var precedences = map[token.TokenType]int{
	token.EQ:       EQUALS,
	token.NOT_EQ:   EQUALS,
	token.LT:       LESSGREATER,
	token.GT:       LESSGREATER,
	token.PLUS:     SUM,
	token.MINUS:    SUM,
	token.SLASH:    PRODUCT,
	token.ASTERISK: PRODUCT,
}
func (p *Parser) peekPrecedence() int {
	if p, ok := precedences[p.peekToken.Type]; ok {
		return p
	}
	return LOWEST
}
func (p *Parser) curPrecedence() int {
	if p, ok := precedences[p.curToken.Type]; ok {
		return p
	}
	return LOWEST
}
```
`precedences`是我们的优先级表：它关联Token类型到它们的优先级。优先级的值就是我们先前定义的常量，一系列自增的整数。这个表告诉我们+（`token.PLUS`）和-（`token.MINUS`）拥有相同的优先级，但是它们比`*`（`token.ASTERISK`)和`/`（`token.SLASH`）优先级低。

`peekPrecedence`方法返回`p.peekToken`关联的`token`类型的优先级，如果它没有发现`p.peekToken`的优先级，则返回默认优先级`LOWEST`。最低的优先级每一个操作符都可以拥有。`curPrecedence`方法做同样的工作，只不过针对的是`p.curToken`。

接下来要做的事为一个中缀解析函数注册给所有的中缀操作符：
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.infixParseFns = make(map[token.TokenType]infixParseFn) p.registerInfix(token.PLUS, p.parseInfixExpression) p.registerInfix(token.MINUS, p.parseInfixExpression) p.registerInfix(token.SLASH, p.parseInfixExpression) p.registerInfix(token.ASTERISK, p.parseInfixExpression) p.registerInfix(token.EQ, p.parseInfixExpression) p.registerInfix(token.NOT_EQ, p.parseInfixExpression) p.registerInfix(token.LT, p.parseInfixExpression) p.registerInfix(token.GT, p.parseInfixExpression)
// [...]
}
```
我们先前已经拥有了`registerInfix`方法，到现在我们最终使用它们。每一个中缀操作符关联到同一个解析函数中，叫做`parseInfixExpression`，它看上去是这样的：
```go
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression {
	expression := &ast.InfixExpression{
		Token:    p.curToken,
		Operator: p.curToken.Literal,
		Left:     left,
	}
	precedence := p.curPrecedence()
	p.nextToken()
	expression.Right = p.parseExpression(precedence)
	return expression
}
```
最显著的不同在于，不同于`parsePrefixExpression`方法，在新的方法中我们接受了一个参数，`ast.Expression`类型的`left`。它用这个参数构建`ast.InfixExpression`节点中的`Left`字段，然后将当前`token`(也就是当前中缀表达式的操作符)的优先级保存至变量`precedence`中。然后调用`nextToken`方法前进token。最后调用`parseExpression`方法来获取`Right`字段，其中将`precedence`作为参数传递给方法。

是时候揭开`Pratt`解析法的神秘面纱了，这是`parseExpression`函数的最后一个版本：
```go
func (p *Parser) parseExpression(precedence int) ast.Expression {
	prefix := p.prefixParseFns[p.curToken.Type]
	if prefix == nil {
		p.noPrefixParseFnError(p.curToken.Type)
		return nil
	}
	leftExp := prefix()
	for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() {
		infix := p.infixParseFns[p.peekToken.Type]
		if infix == nil {
			return leftExp
		}
		p.nextToken()
		leftExp = infix(leftExp)
	}
	return leftExp
}
```
Duang, 我们的测试通过了：
```
$ go test ./parser
ok monkey/parser 0.006s
```
现在我们能够正式的解析中缀表达式了！等一下，究竟发生了？它究竟是怎么工作的？

显然`parseExpression`现在能够做一些工作了，我们已经知道如何找到当前`token`关联的解析函数，然后调用它。这些我们在前缀操作符、标识符和整型字面值中也做了相关工作。

新的内容在`parseExpression`函数中的中间循环部分，在中间循环体中，不停为下一个`token`寻找`infixParseFns`。如果找到这样的函数，调用它。将表达式作为参数传递给他，并将结果作为新值。不停的重复以上过程知道遇到一个更高的优先级的`token`。

工作非常棒，让我们看看不同一些测试用例，有更多的操作符和不同的优先级，看看抽象语法树是否正确描述。
```go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		{"-a * b", "((-a) * b)"},
		{"!-a", "(!(-a))"},
		{"a+b+c", "((a + b) + c)"},
		{"a+b-c", "((a + b) - c)"},
		{"a*b*c", "((a * b) * c)"},
		{"a*b/c", "((a * b) / c)"},
		{"a+b/c", "(a + (b / c))"},
		{"a+b*c+d/e-f", "(((a + (b * c)) + (d / e)) - f)"},
		{"3+4;-5*5", "(3 + 4)((-5) * 5)"},
		{"5>4==3<4", "((5 > 4) == (3 < 4))"},
		{"5<4!=3>4", "((5 < 4) != (3 > 4))"},
		{"3+4*5==3*1+4*5", "((3 + (4 * 5)) == ((3 * 1) + (4 * 5)))"},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		actual := program.String()
		if actual != tt.expected {
			t.Errorf("expected=%q, got=%q", tt.expected, actual)
		}
	}
}
```
所有测试都通过了，这个相当神奇，难道不是吗？不同的`*ast.InfixExpression`正确地嵌套，这个幸亏我们在抽象语法树的节点中`String()`方法中正确使用括号。

如果你正在抓耳牢骚想知道这个如何工作的，别担心。我们接下来近距离看看我们的`parseExpression`方法。
<h2 id="ch03-how-pratt-parsing-works">3.7 Pratt解析法如何工作</h2>
`parseExpression`背后的算法和解析函数与优先级共同组合完整地描述了`Vaughan Pratt`在`Top Down Operator Precedence`论文中的主要思想，但是和我们的实现过程还是有一点点不同。

`Pratt`没有使用`Parser`结构而且也没有定义`*Parser`的相关方法，也没有使用字典，当然也没有使用`Go`语言。这边论文比go语言早发表36年。还有一些名称叫法上的不同：`prefixParseFns`在`Pratt`方法中叫做`nuds`(`null denotations`的简称)，`infixParseFns`叫做`leds`(`left denotations`的简称)。

Pratt方法虽然是由伪代码完成，但是我们的`parseExpression`方法看上去和Pratt论文中给出的十分相似。它几乎使用了同样的算法而没有任何改变。

我们打算跳过理论来解答为什么它能工作，仅仅展示它是怎么工作的和各个不同的组件（`parseExpression`,解析函数和优先级）如何组织在一起。假设我们将要解析如下的表达式语句：
```
1 + 2 + 3;
```
最大的挑战不是在于结果的抽象语法树如何表示每一个操作符和操作数，而是将不同的抽象语法树的节点嵌套在一起。我们想要的抽象语法树（序列化为字符串）看上去是这样的：
```
((1+2)+3)
```
这个抽象语法树需要两个`*ast.InfixExpression`节点，位置更高的`*ast.InfixExpression`节点做的右边是整数字面值`3`，左边应该是其他的`*ast.InfixExpression`。该节点做左有两边分别各自为整数字面值`1`和`2`。
![](../figures/ast4.png)

当解析`1 + 2 + 3`的时候，这个就是我我们解析器的输出。但是如何做到的呢？我们将在接下来的几个段落中回答这个问题。我们将会近距离看看我们解析器如何工作的，尤其当`parseExpressionStatement`第一次被调用的时候。当阅读接下面段落的时候，可以翻看之前的代码。

让我们开始吧，当我们解析`1+2+3;`的时候将会发生什么？

`parseExpressionStatement`调用`parseExpression(LOWEST)`，`p.curToken`和`p.peekToken`当前指向`1`和第一个`+`。
![](./figures/expression1.png)

首先`parseExpression`检查是否有`parseParseFn`关联当前的`p.curToken.Type`，因为它是`token.INT`，所以调用`parseIntegerLiteral`方法。 该方法返回一个`*ast.IntegerLiteral.Expression`并赋值给`leftExp`。

接下来是在`parseExpression`中的循环部分，这个条件结果为true。
```go
for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() { 
    // [...]
}
```
`p.peekToken`不是`token.SEMICOLON`而且`peekPrecedence`(它返回`+`的优先级)比当前函数传递过来的`LOWEST`要高。下面是我们已经定义好的优先级：
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
    CALL // myFunction(X)
)
```
所以条件判断为true，并且`paseExpression`执行循环体部分，它看上去是这样的：
```go
infix := p.infixParseFns[p.peekToken.Type] 
if infix == nil {
    return leftExp 
}
p.nextToken()
leftExp = infix(leftExp)
```
它获取`p.peekToken.Type`关联的`infixParseFn`，它就是我们在`*Parser`中定义的`parseInfixExpression`方法，在调用该函数和将返回值赋给`leftExp`之前，将Token位置前进。
![](./figures/expression2.png)

在当前`token`的状态下，它调用`parseInfixExpression`方法，将已经解析好的`*ast.IntegerLiteral`传递给它。 接下来的`parseInfixExpression`是最有趣的部分，下面是方法：
```go
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression {
	expression := &ast.InfixExpression{
		Token:    p.curToken,
		Operator: p.curToken.Literal,
		Left:     left,
	}
	precedence := p.curPrecedence()
	p.nextToken()
	expression.Right = p.parseExpression(precedence)
	return expression
}
```
值得注意的是在`left`是我们已经解析的`*ast.IntegerLiteral`，它是字面值1。

`parseInfixExpression`保存了`p.curToken`的优先级（第一个`+`的优先级），然后前进`token`并且调用`parseExpression`方法，将之前保存的优先级作为参数传递给它。现在`parseExpression`第二次被调用，现在各个`token`看上去是这样的：
![](./figures/expression3.png)

首先`parseExpression`再一次查询`p.curToken`的对应的`prefixParseFn`方法，再一次还是`parseIntegerLiteral`方法，但是现在循环体的执行并没有返回true：表达式`1+2+3`的第一个`+`的优先级并不比第二个`+`的优先级低，它们是相等的。所以循环体将不会被执行。`ast.IntegerLiteral`将代表`2`被返回。

现在回到`parseInfixExpression`中，`paseExpression`的返回值将赋值给新创建的`*ast.InfixExpression`的`Right`字段。就跟我们下面看到的：
![](./figures/ast2.png)

`*ast.InfixExpression`通过调用`parseInfixExpression`方法返回，现在回到我们最外面调用的`parseExpression`方法，在这里的优先级仍然是`LOWEST`。我们回到我们开始执行的循环的部分，并且再次执行循环。
```go
for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() { 
    // [...]
}
```
现在执行结果为`true`，因为`precedence`是`LOWEST`，而且`peekPrecedence`返回表达式第二个加法的优先级。`parseExpression`再一次执行循环函数体。不同的是现在的`leftExp`不再是`*ast.IntegerLiteral`中的`1`，而是`*ast.InfixExpression`返回的`praseInfixExpression`，代表着1+2。

在`parseExpression`循环中获取`p.peekToek.Type`关联的`parseInfixExpression`，前进`token`并且调用`parseInfixExpression`方法，在这里使用`leftExp`组我诶参数。 `parseInfixExpression`再次调用`parseExpression`方法，这个时候返回最后的`*ast.IntegerLiteral`（代表着表达式中的3)。

最后，在循环体的最后，`leftExp`看上去是这样的
![](./figures/ast3.png)
它的确就是我们想要的，操作符和操作数正确的嵌套在一起。我们的`token`看上去是这样的
![](./figures/expression4.png)

循环的条件执行结果为`false`
```go
for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() { 
    // [...]
}
```
现在`p.peekTokenIs(token.SEMICOLON)`执行结果为`true`，它将停止执行循环体内部。（调用`p.peekTokenIs(token.SEMICOLON)`不是严格必须的，我们的`peekPrecedence`方法遇到默认值返回`LOWEST`。但是我认为分号作为表达式结束符更加清晰，更加容易明白）

循环部分已经完成`leftExp`被返回，我们回到`parseExpressionStatement`中，最终我们拥有了正确的`*ast.InfixExpression`，然后将他们当做`Expression`存放到`*ast.ExpressionStatement`中。

现在我们知道我们的解析器如何去正确的解析`1+2+3`，这相当神奇。我认为`precedence`和`peekPrecedence`的使用也是相当有趣。


但是优先级真正关系的是什么？在我们的例子中，每一个操作符(+)拥有相同的优先级。那么如果操作符拥有不同的优先级？我们能不能将`LOWEST`作为默认值，将所有的操作符作为`HIGHEST`。

不行，这样会导致错误的抽象语法树，含有操作符的表达式的目标是有更高的优先级的操作数的深度比低优先级的操作更深，而这些都是有个优先级完成的。

当`parseExpression`被调用的时候，`precedences`代表当前`parseExpression`的"右绑定"能力，什么叫做"右绑定"能力呢？也就是当前表达式的对右边的对`token`，操作数和操作符的绑定能力。

假设我们的当前的右绑定能力最高，那么我们目前解析的的部分（复制给`letfExp`)将不会传递给下一个操作数对应的`infixParseFn`方法。将不会终止与一个左孩子节点，因为循环条件将不会执行为True。

与之相反存在的能力也是有的，叫做左绑定能力。但是什么值能够展示左绑定能力呢？因为在`parseExpresion`方法中，我们`precedence`参数代表着当前右绑定能力，那么下一个操作数的左绑定能力来自哪里？简单来讲就是来自我们调用`peekPrecedence`。方法的返回值代表了下一个操作符的左绑定能力。

它们都来自于循环中的`precedence < p.peekPrecedence()`条件表达式，这个条件检查下一个操作符或者`token`的左绑定是否比当前的右绑定值高。如果是的，我们解析过程将陷入下一个操作符，从左到右，然后结束于下一个操作符对应的`infixParseFn`方法。

做一个例子来讲：比如我们解析`-1+2`,我们想要抽象语法树的表达是这样的`(-1)+2`而不是`-(1+2)`。第一个方法结束语`token.MINUS`绑定的`prefixParseFn`:`parsePrefixExpression`。让我们在看一看`parsePrefixExpression`方法的全部代码：
```go
func (p *Parser) parsePrefixExpression() ast.Expression {
	expression := &ast.PrefixExpression{
		Token:    p.curToken,
		Operator: p.curToken.Literal,
	}
	p.nextToken()
	expression.Right = p.parseExpression(PREFIX)
	return expression
}
```
它将`PREFIX`传递给`parseExpression`作为优先级，它也是当前`parseExpression`占据的右绑定能力。在我们先前定义中，`PREFIX`的优先级非常高。这也导致`parseExpression(PREFIX)`永远不会将`-1`的`1`传递个其他的`infixParseFn`。在这个例子中`precedence < p.peekPrecedence()`永远不会为`True`，也就意味着没有其他的`infixParseFn`将我们的`1`作为我们的左孩子节点。而是将`1`返回给我们前缀表达式的右孩子。

回到我们最外面的`parseExpression`方法，在第一个`leftExp := prefix()`之后，`precedence`仍然还是`LOWEST`，因为它仍然是我们最外面的调用。我们的右绑定能力仍然是`LOWEST`，而现在的`p.peekToken`是指向`1+2`中的`+`号。

我么现在是在循环的条件部分，并且执行来决定是否我们要执行我们的循环体。现在表明`+`操作数的优先级比当前的右绑定能力高，我们已经解析好了`-1`的前缀表达式，并且将它出的传递给`+`关联的`infixParseFn`. `+`的左绑定能力吸住了我们目前已经解析好的内容作为它在抽象语法树中的左孩子。

`+`关联的`infixParseFn`是`parseInfixExpression`，它现在使用`+`的优先级作为他的右绑定能力，而没有使用`LOWEST`，因为这样做的话将导致另外一个`+`的拥有更高的左绑定值，并且将它吸入。如果这样做的话，表达式`a+b+c`将会返回`(a+(b+c))`，而不是我们期望的`((a+b)+c)`。

将前缀操作符设置为高优先级是有用的，和中缀表达式很好的配合工作。在经典的优先级例子:`1+2*3`, `*`的左绑定能力比`+`的右绑定值高。所以解析的过程中将会`2`作为参数传递给`*`关联的`infixParseFn`方法。

值得注意的是我们的解析器中，每一个`token`都拥有相同的左右绑定值，我们仅仅是用同一个值赋给他们两个，具体的值这个取决于上下文环境。

如果我们的操作符必须右结合而不是左结合（在例子中`+`的结果是`(a+(b+c)`而不是`((a+b)+c)`），那么我们的必须使用较小的右绑定值在解析操作符表达式的右孩子的时候。 如果你考虑到其他语言`++`或者`--`的操作符，它们可以用在前缀和后缀的位置，你就可以看到区分左右绑定值的是非常有用的。

因为我们的没有分别为设置左绑定和右绑定值，仅仅使用了相同的值，我们不能仅仅通过改变定义来达到这一点。但是举个例子来讲，为了让`+`进行右结合，我们可以在调用`parseExpression`方法的时候减少其优先级。
```go
// parser/parser.go
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression { 
    expression := &ast.InfixExpression{
        Token: p.curToken, 
        Operator: p.curToken.Literal, 
        Left: left,
    }
    precedence := p.curPrecedence()
    p.nextToken()
    expression.Right = p.parseExpression(precedence)
    //                                    ^^^decrement here for right-associativtiy
    return expression
}
```
为了示范的目的，让我们短暂地改变这个方法，然看将会发生什么？
```go
// parser/parser.go
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression {
    expression := &ast.InfixExpression {
        Token: p.curToken,
        Operator: p.curToken.Literal,
        Left: left,
    }
    precedence := p.curPrecedence()
    p.nextToken()
    if expression.Operator == "+" {
        expression.Right = p.parseExpression(precedence - 1)
    }else{
        expresion.Right = p.parseExpression(precedence)
    }
    return expression
}
```
有着这些改变，我们的测试表明现在的`+`是右结合的：
```
$ go test -run TestOperatorPrecedenceParsing ./parser 
--- FAIL: TestOperatorPrecedenceParsing (0.00s)
parser_test.go:359: expected="((a + b) + c)", got="(a + (b + c))" 
parser_test.go:359: expected="((a + b) - c)", got="(a + (b - c))" 
parser_test.go:359: expected="(((a + (b * c)) + (d / e)) - f)",\
got="(a + ((b * c) + ((d / e) - f)))" 
FAIL
```
这个让我们更加深入的了解到了`parseExpression`方法，如果你现在还不确定是否掌握了它是如何工作的，别担心，我也是这样的。将我们每一个解析过程跟踪打印出来，它将会对我们非常有帮助。在随书的本章的代码中我将下面的代码放在了`./parser/parse_tracing.go`中，这个我们从来没有见过，该文件中包含了两个定义的函数`trace`和`untrace`很好地帮助我们去理解解析器所做的工作。使用如下：
```go 
// parser/parser.go
func (p *Parser) parseExpressionStatement() *ast.ExpressionStatement { 
    defer untrace(trace("parseExpressionStatement"))
// [...]
}
func (p *Parser) parseExpression(precedence int) ast.Expression { 
    defer untrace(trace("parseExpression"))
// [...]
}
func (p *Parser) parseIntegerLiteral() ast.Expression { 
    defer untrace(trace("parseIntegerLiteral"))
// [...]
}
func (p *Parser) parsePrefixExpression() ast.Expression { 
    defer untrace(trace("parsePrefixExpression"))
// [...]
}
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression { 
    defer untrace(trace("parseInfixExpression"))
// [...]
}
```
通过这个记录的声明，现在使用解析器的时候可以看到他是如何工作的，下面是我们在解析`-1*2+3`的时候的输出
```
$ go test -v -run TestOperatorPrecedenceParsing ./parser 
=== RUN TestOperatorPrecedenceParsing
BEGIN parseExpressionStatement
    BEGIN parseExpression
        BEGIN parsePrefixExpression
            BEGIN parseExpression
                BEGIN parseIntegerLiteral
                END parseIntegerLiteral 
            END parseExpression
        END parsePrefixExpression 
        BEGIN parseInfixExpression
            BEGIN parseExpression
                BEGIN parseIntegerLiteral
                END parseIntegerLiteral 
            END parseExpression
        END parseInfixExpression 
        BEGIN parseInfixExpression
            BEGIN parseExpression
                BEGIN parseIntegerLiteral
                END parseIntegerLiteral 
            END parseExpression
        END parseInfixExpression 
    END parseExpression
END parseExpressionStatement
--- PASS: TestOperatorPrecedenceParsing (0.00s) PASS
ok monkey/parser 0.008s
```
<h2 id="ch03-extending-the-parser">3.8 拓展解析器</h2>
在我们继续之前，我们需要拓展我们的解析器。首先我们需要清理和拓展我们的测试套件，我不会将所有的的改变列出来，但是我们展示一些小的帮助函数，以便让姿势更容易理解。

我们已经有了`testIntegerLiteral`帮助，第二个函数`testIdentifier`可以帮助我们清理好多测试：
```go
// parser/parser_test.go
func testIdentifier(t *testing.T, exp ast.Expression, value string) bool {
	ident, ok := exp.(*ast.Identifier)
	if !ok {
		t.Errorf("exp not *ast.Identifier. got=%T", exp)
		return false
	}
	if ident.Value != value {
		t.Errorf("ident.Value not %s. got=%s", value, ident.Value)
		return false
	}
	if ident.TokenLiteral() != value {
		t.Errorf("ident.TokenLiteral not %s. got=%s", value, ident.TokenLiteral())
		return false
	}
	return true
}
```
最有趣的是使用`testIntegerLiteral`和`testIdentifier`可以构建很多泛化的帮助方法
```go
func testLiteralExpression(t *testing.T, exp ast.Expression, expected interface{}) bool {
	switch v := expected.(type) {
	case int:
		return testIntegerLiteral(t, exp, int64(v))
	case int64:
		return testIntegerLiteral(t, exp, v)
	case string:
		return testIdentifier(t, exp, v)
	}
	t.Errorf("type of exp not handled. got=%T", exp)
	return false
}
func testInfixExpression(t *testing.T, exp ast.Expression, left interface{},
	operator string, right interface{}) bool {
	opExp, ok := exp.(*ast.InfixExpression)
	if !ok {
		t.Errorf("exp is not ast.InfixExpression. got=%T(%s)", exp, exp)
		return false
	}
	if !testLiteralExpression(t, opExp.Left, left) {
		return false
	}
	if opExp.Operator != operator {
		t.Errorf("exp.Operator is not '%s'. got=%q", operator, opExp.Operator)
		return false
	}
	if !testLiteralExpression(t, opExp.Right, right) {
		return false
	}
	return true
}
```
有了这些替代，我们可以如下的方式编写测试代码：
```go
testInfixExpression(t, stmt.Expression, 5, "+", 10) 
testInfixExpression(t, stmt.Expression, "alice", "*", "bob")
```
它将我们的测试解析器生成的抽象语法树更加容易了。

**布尔字面值**

还有一些东西需要我们的编程语言来时间以便在解析器和抽象语法书中使用，在Monkey中我们可以这样使用布尔型
```
true;
false;
let foobar = true;
let barfoo = false;
```
正如标识符和整数字面值，它的抽象语法树表示也非常简单和短小：
```go
type Boolean struct {
	Token token.Token
	Value bool
}
func (b *Boolean) expressionNode()      {}
func (b *Boolean) TokenLiteral() string { return b.Token.Literal }
func (b *Boolean) String() string       { return b.Token.Literal }
```
`Value`字段可以保留`bool`的值，也就意味着我们可以保存`true`或者`fasle`(注意是Go语言中的布尔值)

有了抽象语法树的节点，我们可以增加测试。 测试函数`TestBooleanExpression`和测试函数`TestIdentifierExpression`和`TestIntegerLiteralExpression`非常相像，在这里我就不展示了。 下面的测试足够指出了我们如何即实现解析布尔字面值解析。
```
$ go test ./parser
--- FAIL: TestBooleanExpression (0.00s)
parser_test.go:470: parser has 1 errors
parser_test.go:472: parser error: "no prefix parse function for true found" FAIL
FAIL monkey/parser 0.008s
```
当然，我们需要为`token.TRUE`和`token.FALSE`注册`prefixParseFn`
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.TRUE, p.parseBoolean)
    p.registerPrefix(token.FALSE, p.parseBoolean)
// [...]
}
```
`parseBoolean`方法正如想象中的一样：
```go
//parser/parser.go
func (p *Parser) parseBoolean() ast.Expression {
    return &ast.Boolean{Token: p.curToken, Value: p.curTokenIs(token.TRUE)}
}
```
其中最有趣的部分是内联的`p.curTokenIs(token.TRUE)`方法的调用，但是它并不是真正有缺。只不过它不是很直接。换句话说，我们的解析器工作的很好，这也是`Pratt`方法的漂亮的地方，它非常容易去拓展。

现在测试通过了
```
$ go test ./parser
ok monkey/parser 0.006s
```
但是更有缺的地方时我们现在拓展几个测试用例来完成刚刚实现的布尔字面值，第一个后选择就是`TestOperatorPrecedenceParsing`，它使用了字符串比较机制
```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
        //[...]
		{"true", "true"},
		{"false", "false"},
		{"3>5==false", "((3 > 5) == false)"},
		{"3<5==true", "((3 < 5) == true)"},
		{"1+(2+3)+4", "((1 + (2 + 3)) + 4)"},
		{"(5+5)*2", "((5 + 5) * 2)"},
		{"2/(5+5)", "(2 / (5 + 5))"},
		{"-(5+5)", "(-(5 + 5))"},
	    //[...]
}
```
通过`testBooleanLiteral`方法，我们可以拓展`testLiteralExpression`函数来测试布尔字面值。
```go
// parser/parser_test.go
func testLiteralExpression(t *testing.T, exp ast.Expression, expected interface{}) bool {
	switch v := expected.(type) {
	//[...]
	case bool:
		return testBooleanLiteral(t, exp, v)
	//[...]
	}
}
func testBooleanLiteral(t *testing.T, exp ast.Expression, value bool) bool {
	bo, ok := exp.(*ast.Boolean)
	if !ok {
		t.Errorf("exp not *ast.Boolean. got=%T", exp)
		return false
	}
	if bo.Value != value {
		t.Errorf("bo.Value not %t, got=%t", value, bo.Value)
		return false
	}
	if bo.TokenLiteral() != fmt.Sprintf("%t", value) {
		t.Errorf("bo.TokenLiteral not %t, got=%s",
			value, bo.TokenLiteral())
		return false
	}
	return true
}
```
没有任何神奇的，只不过在`switch`语句中增加了一个`case`分支和新的帮助函数，在这个地方，我们也可以很容易拓展`TestParsingInfixExpressions`。
```go
// parser/parser_test.go
func TestParsingInfixExpressions(t *testing.T) { 
    infixTests := []struct {
        input string 
        leftValue interface{} 
        operator string 
        rightValue interface{}
}{
// [...]
    {"true == true", true, "==", true}, 
    {"true != false", true, "!=", false}, 
    {"false == false", false, "==", false},
}
// [...]
if !testLiteralExpression(t, exp.Left, tt.leftValue) { 
    return
}
if !testLiteralExpression(t, exp.Right, tt.rightValue) { 
    return
}
//[...]
```
同样`TestParsingPrefixExpressions`也很容易拓展，只需要测试表中增加新的入口：
```go
// parser/parser_test.go
func TestParsingPrefixExpressions(t *testing.T) { 
    prefixTests := []struct {
        input string 
        operator string 
        value interface{}
    }{
    // [...]
    {"!true;", "!", true},
    {"!false;", "!", false}, }
    // [...]
}
```
我们实现了解析布尔型字面值并且拓展了我们的测试，它可以给我们更多的测试覆盖率和更好的工具。

**分组表达式**

接下来我们要做的事如何解析分组的表达式，当然，在Monkey中，我们可以通过添加括号来对表达式进行分组从而修改优先级，以便适应他们执行的环境。接下来看一个例子：
```
(5+5)*2
```
使用括号将`5+5`表达式分组，那么他们有更高的优先级。他们在抽象语法树中的的位置更深，导致后面额执行结果和数学表达式相同。

或许你现在想：*该死的优先级怎么又来了，头都大了*， 如果你想跳过本章节，直接到最后。我建议别这么做！

我们不打算为分组表达式编写单元测试，因为它们不是有抽象语法树节点构成。我们不需要改变我的抽象语法树以便正确解析分组表达式。我们要做的是拓展我们的`TestOperatorPrecedenceParsing`测试函数，来确保括号正确的将表达式分组，而且对抽象语法树的结果又影响。
```go
// parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		//[...]
		{"1+(2+3)+4", "((1 + (2 + 3)) + 4)"},
		{"(5+5)*2", "((5 + 5) * 2)"},
		{"2/(5+5)", "(2 / (5 + 5))"},
		{"-(5+5)", "(-(5 + 5))"},
        {"!(true==true)", "(!(true == true))"},
        //[...]
	}
}
```
当然测试失败的：
```
$ go test ./parser
--- FAIL: TestOperatorPrecedenceParsing (0.00s)
    parser_test.go:531: parser has 3 errors
    parser_test.go:533: parser error: "no prefix parse function for ( found" 
    parser_test.go:533: parser error: "no prefix parse function for ) found" 
    parser_test.go:533: parser error: "no prefix parse function for + found"
FAIL
FAIL monkey/parser 0.007s
```
下面是我们最烧脑的部分，为了让测试通过，我们需要增加如下代码:
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.registerPrefix(token.LPAREN, p.parseGroupedExpression)
    // [...]
}
func (p *Parser) parseGroupedExpression() ast.Expression { 
    p.nextToken()
    exp := p.parseExpression(LOWEST)
    if !p.expectPeek(token.RPAREN) { 
        return nil
    }
    return exp 
}
```
是的，我们的测试通过，而且括号正如我们期望的那样，增加了被括号包含的测表达式的优先级。`token`类型关联函数的概念棒极了。

**条件表达式**

在Monkey语言中，我们可以使用`if`和`esle`语句，用法和其他编程语言一样的：
```
if (x > y) {
    return x;
}else{
    return y;
}
```
其中`else`还是可选的
```
if (x > y){
    return x;
}
```
上面的我们都很熟悉了，在Monkey中，`if-else-conditional`条件是表达式。那就意味着他们可以生成值，哪怕`if`表达式是最后执行的。我们甚至不需要在这里使用`return`语句。
```
let foobar = if (x>y) {x} else {y};
```
解释`if-else-conditional`结构是没有必要的，为了清除他们的命名，我们还是这样做了：
```
if (<condition>) <consequence> else <alternative>
```
使用方括号包含起来的`consequence`和`alternative`是语句块，语句块意味着他们是一系列语句（就跟Monkey中`programs`)。

目前我们的所有工作都是成功的，所以至于`if-else-conditional`也不例外。

下面是抽象语法树节点`ast.IfExpression`的定义：
```go
type IfExpression struct {
	Token       token.Token
	Condition   Expression
	Consequence *BlockStatement
	Alternative *BlockStatement
}

func (ie *IfExpression) expressionNode()      {}
func (ie *IfExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *IfExpression) String() string {
	var out bytes.Buffer
	out.WriteString("if")
	out.WriteString(ie.Condition.String())
	out.WriteString(" ")
	out.WriteString(ie.Consequence.String())
	if ie.Alternative != nil {
		out.WriteString("else")
		out.WriteString(ie.Alternative.String())
	}
	return out.String()
}
```
我们的`*ast.IfExpression`实现了`ast.Expression`接口，并且有三个字段代表了`if-else-conditional`。`Condition`保存了`condition`，它可以是任何表达式。`Consequence`和`Alternative`分别指向了`consequence`和`alteranative`。它们引用了新类型`ast.BlockStatement`，正如以前，`consequence/alterantive`是`if-else-condition`照片那个的一系列声明。它也是我们`ast.BlockStatement`代表的内容。
```go
type BlockStatement struct {
	Token      token.Token
	Statements []Statement
}

func (bs *BlockStatement) statementNode()       {}
func (bs *BlockStatement) TokenLiteral() string { return bs.Token.Literal }
func (bs *BlockStatement) String() string {
	var out bytes.Buffer
	for _, s := range bs.Statements {
		out.WriteString(s.String())
	}
	return out.String()
}
```
下一步就是测试，我们现在已经知道如何做了，测试如下：
```go
func TestIfExpression(t *testing.T) {
	input := `if(x<y){x}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Body does not contain %d statements. got=%d\n",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	exp, ok := stmt.Expression.(*ast.IfExpression)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.IfExpression. got=%T",
			stmt.Expression)
	}
	if !testInfixExpression(t, exp.Condition, "x", "<", "y") {
		return
	}
	if len(exp.Consequence.Statements) != 1 {
		t.Errorf("consequence is not 1 statements. got=%d\n",
			len(exp.Consequence.Statements))
	}
	consequence, ok := exp.Consequence.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("Statements[0] is not ast.ExpressionStatement. got=%T",
			exp.Consequence.Statements[0])
	}
	if !testIdentifier(t, consequence.Expression, "x") {
		return
	}
	if exp.Alternative != nil {
		t.Errorf("exp.Alternative.Statement was not nil. got=%+v", exp.Alternative)
	}
}
```
我也增加了`TestIfElseExpression`测试函数，然后使用下面的测试输入
```
if ( x< y ) { x } else { y }
```
在`TestIfElseExpression`中有额外的判断`Alternative`字段部分。所有对于`*ast.IfExpression`节点的结果判断使用了帮助函数`testInfixExpression`和`testIdentifer`。这样我们可以专注于条件表达式本身，来确保我们的解析器剩下的部分正确集成起来。

所有测试都以错误消息返回，我们对此已经非常熟悉了
```
$ go test ./parser
--- FAIL: TestIfExpression (0.00s)
    parser_test.go:659: parser has 3 errors
    parser_test.go:661: parser error: "no prefix parse function for IF found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
--- FAIL: TestIfElseExpression (0.00s)
    parser_test.go:659: parser has 6 errors
    parser_test.go:661: parser error: "no prefix parse function for IF found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found" 
    parser_test.go:661: parser error: "no prefix parse function for ELSE found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
首先我们开始第一个失败测试`TestifExpression`， 显然我们需要为`Token.IF`注册第`prefixParseFn`。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.IF, p.parseIfExpression)
// [...]
}
func (p *Parser) parseIfExpression() ast.Expression { 
    expression := &ast.IfExpression{Token: p.curToken}
    if !p.expectPeek(token.LPAREN) { 
        return nil
    }
    p.nextToken()
    expression.Condition = p.parseExpression(LOWEST)
    if !p.expectPeek(token.RPAREN) { 
        return nil
    }
    if !p.expectPeek(token.LBRACE) { 
        return nil
    }
    expression.Consequence = p.parseBlockStatement()
    return expression
}
```
在这个解析函数中，我们频繁调用`expectPeek`，这不仅仅是必须的，而且是有意义的。`expectPeek`如果期望的`token`不符合预期，那么它将会增加给解析增加一个错误，如果满足期望，它将会前进一个`token`，这的确是我们需要的，在这之后就是`{`，它标志着语句块的开始。

这个方法遵循我们我们的解析函数协议：`token`前进至`parseBlockStatement`开始的地方，当前`p.curToken`位于`{`处。下面是`parseBlockStatement`解析函数：
```go
// parser/parser.go
func (p *Parser) parseBlockStatement() *ast.BlockStatement {
	block := &ast.BlockStatement{Token: p.curToken}
	block.Statements = []ast.Statement{}
	p.nextToken()
	for !p.curTokenIs(token.RBRACE) {
		stmt := p.parseStatement()
		if stmt != nil {
			block.Statements = append(block.Statements, stmt)
		}
		p.nextToken()
	}
	return block
}
```
`parseBlockStatement`调用`parseStatement`方法一直到遇到`}`，它标记着语句块的结束。它上去和我们的顶层`parseProgram`方法非常像，在哪里我们不停地调用`parseStatement`方法，知道遇到结束`token`，在`ParseProgram`中，该结束`token`是`token.EOF`。这循环的重复并没有没有错误的影响，所以我们保留它们并且注意它们的测试用例：
```
$ go test ./parser
--- FAIL: TestIfElseExpression (0.00s)
    parser_test.go:659: parser has 3 errors
    parser_test.go:661: parser error: "no prefix parse function for ELSE found" 
    parser_test.go:661: parser error: "no prefix parse function for { found" 
    parser_test.go:661: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
`TestIfExpression`测试通过，但是`TestIfElseExpression`并没有通过。现在为了支持`if-else-condition`中的`else`分支，我们需要检查该分支是否存在，以便我们能够在`else`分支解析它们。
```go
// parser/parser.go
func (p *Parser) parseIfExpression() ast.Expression {
//[...]
	expression.Consequence = p.parseBlockStatement()
	if p.peekTokenIs(token.ELSE) {
		p.nextToken()
		if !p.expectPeek(token.LBRACE) {
			return nil
		}
		expression.Alternative = p.parseBlockStatement()
	}
	return expression
}
```
这就是全部了，这个方法允许可选的`else`分支但是没有增加错误，在我们解析`consequence-block-statement`后，我们检查是否出现`token.ELSE`，如果存在我们跳过两个`token`，第一次调用`nextToken`的原因我们已经知道`p.peekToken`是`else`，然后我们调用`expectPeek`的原因是下一个`token`是一个语句块的开始部分`{`，否则这个程序是无效的。

是的，解析的过程非常容易出现错误，非常容易忘了前进`token`或者错误地调用`nextToken`方法。通过严格的协议表明每一个解析函数怎么去前进`token`很好的帮助我们。幸运的是我们用很好的测试用例来保证我们知道每一步工作是怎样的：
```
$ go test ./parser
ok monkey/parser 0.007s
```

**函数字面**

你已经注意到`parseIfExpression`方法中我们增加了很多代码，这个比任何一个`prefixParseFn`和`infixParseFn`方法还要多。因为我们针对不同的`token`和表达式类型有不得不做很多工作。接下来我们要做的的工作难度和之前的差不多，而且也涉及到更多的`token`类型。我们将要解析函数字面。

在Monkey中，通过函数字面来定义函数，包含了函数的参数和函数所做的内容。函数看上去是这样的：
```
fn(x, y) {
    return x+y;
}
```
它开始于关键字`fn`，紧接着是参数列表，然后是语句块，它就是函数体，函数在调用的时候，函数体将会被执行。抽象结构如下
```
fn <parameters><block statement>
```
我们已经知道语句快了，也只到如何解析它们。参数列表之前没有遇到过，但是解析它们并不困难。它们仅仅是一系列由逗号隔开的标识符，并且有括号包含起来：
```
(<parameter one>, <parameter two>, <parameter three>, ...)
```
这个列表可以是空的
```
fn(){
    return foobar + barfoo;
}
```
它们就是函数字面，那么他们属于那种抽象语法树节点类型呢？当然是表达式。将函数字面放到任何表达式位置都是有效的。举个例子，下面是将函数字面作为`let`语句的表达式部分：
```
let myFunction = fn(x, y) { return x + y; }
```
下面是将函数字面作为`return`语句的表达式部分，而且它还在其他函数内部：
```
fn(){
    return fn(x, y) { return x > y; }
}
```
也可以将一个函数字面作为其他函数调用的参数
```
myFunc(x, y, fn(x, y) {return x > y;});
```
听上去有点复杂，但是一点也不。其中最棒的是我么你的解析器一旦作为表达式解析成功，剩下的工作它将正确解析成函数。

我们已经知道函数字面值有两个主要的部分：参数列表和函数体的语句块，在定义抽象语法树节点的时候要注意这一点。

```go
type FunctionLiteral struct {
	Token      token.Token
	Parameters []*Identifier
	Body       *BlockStatement
}

func (fl *FunctionLiteral) expressionNode()      {}
func (fl *FunctionLiteral) TokenLiteral() string { return fl.Token.Literal }
func (fl *FunctionLiteral) String() string {
	var out bytes.Buffer
	params := make([]string, 0)
	for _, p := range fl.Parameters {
		params = append(params, p.String())
	}
	out.WriteString(fl.TokenLiteral())
	out.WriteString("(")
	out.WriteString(strings.Join(params, ", "))
	out.WriteString(") ")
	out.WriteString(fl.Body.String())
	return out.String()

}
```
`Parameters`字段是`*ast.Identfiers`的切片，`Body`字段是`*ast.BlockStatement`，这个我们之前看过。

下面是测试，在这里我们也使用了`testLiteralExpression`和`testInfixExpression`帮助函数：

```go
func TestFunctionLiteralParsing(t *testing.T) {
	input := `fn(x,y){x+y;}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Body does not coantin %d statement. got=%d",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	function, ok := stmt.Expression.(*ast.FunctionLiteral)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.FunctionLiteral. got=%T",
			stmt.Expression)
	}
	if len(function.Parameters) != 2 {
		t.Fatalf("stmt.Expression is not ast.FunctionLiteral. got=%T",
			stmt.Expression)
	}
	testLiteralExpression(t, function.Parameters[0], "x")
	testLiteralExpression(t, function.Parameters[1], "y")
	if len(function.Body.Statements) != 1 {
		t.Fatalf("function.Body.Statements has not 1 Statements. got=%d\n",
			len(function.Body.Statements))
	}
	bodyStmt, ok := function.Body.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("function body stmt is not ast.ExpressionStatement. got=%T",
			function.Body.Statements[0])
	}
	testInfixExpression(t, bodyStmt.Expression, "x", "+", "y")
}
```
测试主要分为三个主要部分：检查`*ast.FunctionLiteral`是否存在；检查参数列表是否正确；确保函数体包含正确的语句。最后一部分不是严格必须的，我们我们已经在`IfExpression`中已经测试用语句块，但是我们在这里重复之前工作也是无关紧要。

仅仅定义了`ast.FunctionLiteral`并没有改变我们的解析器，测试失败了
```

$ go test ./parser
--- FAIL: TestFunctionLiteralParsing (0.00s)
    parser_test.go:755: parser has 6 errors
    parser_test.go:757: parser error: "no prefix parse function for FUNCTION found" 
    parser_test.go:757: parser error: "expected next token to be ), got , instead" 
    parser_test.go:757: parser error: "no prefix parse function for , found" 
    parser_test.go:757: parser error: "no prefix parse function for ) found" 
    parser_test.go:757: parser error: "no prefix parse function for { found" 
    parser_test.go:757: parser error: "no prefix parse function for } found"
FAIL
FAIL monkey/parser 0.007s
```
显然我们需要为`token.FUNCTION`注册新的`prefixParseFn`。
```go
// parser/parsr.go
func New(l *lexer.Lexer) *Parser { 
// [...]
    p.registerPrefix(token.FUNCTION, p.parseFunctionLiteral)
// [...]
}
func (p *Parser) parseFunctionLiteral() ast.Expression { 
    lit := &ast.FunctionLiteral{Token: p.curToken}
    if !p.expectPeek(token.LPAREN) { 
        return nil
    }
    lit.Parameters = p.parseFunctionParameters()
    if !p.expectPeek(token.LBRACE) { 
        return nil
    }
    lit.Body = p.parseBlockStatement()
    return lit 
}
```
使用的`parseFunctionParameter`方法看上去是这样的
```go
//parser/parser.go
func (p *Parser) parseFunctionParameters() []*ast.Identifier {
	identifiers := make([]*ast.Identifier, 0)
	if p.peekTokenIs(token.RPAREN) {
		p.nextToken()
		return identifiers
	}
	p.nextToken()
	ident := &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	identifiers = append(identifiers, ident)

	for p.peekTokenIs(token.COMMA) {
		p.nextToken()
		p.nextToken()
		ident := &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
		identifiers = append(identifiers, ident)
	}
	if !p.expectPeek(token.RPAREN) {
		return nil
	}
	return identifiers
}
```
`parseFunctionParameter`核心的部分是构建参数切片通过不停地从逗号分隔的列表中创建标识符。这也表面如果列表为空，将会退出。

这个方法值得我们在编写一系列测试来检查边界用例，一个空的参数列表，只有一个参数和多个参数不同情况。
```go
func TestFunctionParameterParsing(t *testing.T) {
	tests := []struct {
		input            string
		expectedParameer []string
	}{
		{"fn(){}", []string{}},
		{"fn(x){}", []string{"x"}},
		{"fn(x,y){}", []string{"x", "y"}},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		stmt := program.Statements[0].(*ast.ExpressionStatement)
		function := stmt.Expression.(*ast.FunctionLiteral)
		if len(function.Parameters) != len(tt.expectedParameer) {
			t.Errorf("length parameters wrong. want %d, got=%d\n",
				len(tt.expectedParameer), len(function.Parameters))
		}
		for i, ident := range tt.expectedParameer {
			testLiteralExpression(t, function.Parameters[i], ident)
		}
	}
}
```
现在测试通过了
```
$ go test ./parser
ok monkey/parser 0.007s
```
现在我们已经正确解析了函数字面。


**调用表达式**

既然我们知道如何解析函数字面，下一步就应该解析函数调用：调用表达式。下面是结构：
```
<expression>(<comma separated expression>)
```
下面是调用表达式正常使用
```
add(2, 3)
```
如果你认为`add`是标识符，标识符就是表达式，参数`2`和`3`都是表达式，参数就是一系列表达式。
```
add(2+2, 3*3*3)
```
这个同样也是有效的，第一个参数是一个中缀表达式`2+2`，第二个是`3*3*3`。现在让我们看看这里函数调用，这个例子中，函数被绑定到标识符`add`，这个标识符在执行的时候返回该函数，这也就意味着我们可以直接跳过标识符，替换掉`add`函数字面。
```
fn(x, y){x+y; }(2, 3)
```
这也是有效的，我么也可以使用函数字面值作为参数
```
callsFunction(2, 3, fn(x, y) { x + y; });
```
所以再一次看看该结构
```
<expression>(<comma separated expressions>)
```
调用表达式有个一个表达式（执行的时候返回一个函数）和一系列表达式，它们是函数调用的是否的参数。所以抽象语法树的节点看上去是这样的
```go
// ast/ast.go
type CallExpression struct {
	Token     token.Token
	Function  Expression
	Arguments []Expression
}

func (ce *CallExpression) expressionNode()      {}
func (ce *CallExpression) TokenLiteral() string { return ce.Token.Literal }
func (ce *CallExpression) String() string {
	var out bytes.Buffer
	args := make([]string, 0)
	for _, a := range ce.Arguments {
		args = append(args, a.String())
	}
	out.WriteString(ce.Function.String())
	out.WriteString("(")
	out.WriteString(strings.Join(args, ", "))
	out.WriteString(")")
	return out.String()
}

type StringLiteral struct {
	Token token.Token
	Value string
}
```
测试也是同样如此，它对`*ast.CallExpression`的结构进行判断
```go
// parser/parser_test.go
func TestCallExpressionParsing(t *testing.T) {
	input := `add(1, 2*3, 4+5)`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	if len(program.Statements) != 1 {
		t.Fatalf("program.Statements doest not contain %d statements. got=%d\n",
			1, len(program.Statements))
	}
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	if !ok {
		t.Fatalf("stmt is not ast.ExpressionStatement. got=%T",
			program.Statements[0])
	}
	exp, ok := stmt.Expression.(*ast.CallExpression)
	if !ok {
		t.Fatalf("stmt.Expression is not ast.CallExpression. got=%T",
			stmt.Expression)
	}
	if !testIdentifier(t, exp.Function, "add") {
		return
	}
	if len(exp.Arguments) != 3 {
		t.Fatalf("wrong length of arguments. got=%d", len(exp.Arguments))
	}
	testLiteralExpression(t, exp.Arguments[0], 1)
	testInfixExpression(t, exp.Arguments[1], 2, "*", 3)
	testInfixExpression(t, exp.Arguments[2], 4, "+", 5)
}
```
通过调用函数字面值和参数解析很好的帮我们将参数解析测试部分分开。通过测试确保我们每个临界测试正确工作。我也增加了`TestCallExpressionParameterParsing`测试方法在本章的代码中。

到目前为止，测试还是失败的，我们得到了错误消息
```
$ go test ./parser
--- FAIL: TestCallExpressionParsing (0.00s)
    parser_test.go:853: parser has 4 errors
    parser_test.go:855: parser error: "expected next token to be ), got , instead" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for ) found"
FAIL
FAIL monkey/parser 0.007s
```
这个错误消息并没有很多信息，为什么没有错误消息提示我们为调用表达式注册`prefixParseFn`?因为我们在调用表达式中并没有增加新的`token`，那我们接下来怎么算呢？举个例子看一看
```
add(2, 3);
```
`add`是由`prefixParseFn`解析的标识符，在标识符之后是`token.LPAREN`，在标识符和参数列表中间，在中缀位置。是的我们有需要为`token.LPAREN`注册`infixParseFn`，这种方式我们解析为函数的比到时是，然后检查关联`token.LPAREN`的`infixParseFn`，调用该函数和进行解析的参数表达式一起使用。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser { 
    // [...]
    p.registerInfix(token.LPAREN, p.parseCallExpression)
    // [...]
}
// parse function call
func (p *Parser) parseCallExpression(function ast.Expression) ast.Expression {
	exp := &ast.CallExpression{Token: p.curToken, Function: function}
	exp.Arguments = p.parseExpressionList(token.RPAREN)
	return exp
}

// parse hash literal
func (p *Parser) parseHashLiteral() ast.Expression {
	hash := &ast.HashLiteral{Token: p.curToken}
	hash.Pairs = make(map[ast.Expression]ast.Expression)
	for !p.peekTokenIs(token.RBRACE) {
		p.nextToken()
		key := p.parseExpression(LOWEST)
		if !p.expectPeek(token.COLON) {
			return nil
		}
		p.nextToken()
		value := p.parseExpression(LOWEST)
		hash.Pairs[key] = value
		if !p.peekTokenIs(token.RBRACE) && !p.expectPeek(token.COMMA) {
			return nil
		}
	}
	if !p.expectPeek(token.RBRACE) {
		return nil
	}
	return hash
}
```
`parseCallExpression`接受一个已经解析的`function`作为参数，使用它构建一个`*ast.CallExpression`节点。为了解析参数列表，我们调用`parseCallArguments`, 它看上去和`parseFunctionParameters`非常类似，除了它看上去更加通用，因为它返回`ast.Expression`切片而不是`*ast.Identifier`.

下面的测试错误我们之前看过，我们还没有注册新的`infixParseFn`
```
$ go test ./parser
--- FAIL: TestCallExpressionParsing (0.00s)
    parser_test.go:853: parser has 4 errors
    parser_test.go:855: parser error: "expected next token to be ), got , instead" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for , found" 
    parser_test.go:855: parser error: "no prefix parse function for ) found"
FAIL
FAIL monkey/parser 0.007s
```
原因是在于`(`在`add(1, 2)`中，作为中缀操作符，但是我们我们还没有赋予优先级。它还没有右结合性。所以`parseExpression`不会返回我们期望的结果。但是调用表达式拥有最高的级别的优先级，所以现在修正我们的优先级表非常重要：
```go
// parser/parser.go
var precedences = map[token.TokenType]int{ 
    // [...]
token.LPAREN: CALL, 
}
```
为了确保我们额调用表达式拥有最高的优先级，我们拓展我们的`TestOperatorPrecedenceParsing`测试函数
```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
		//[...]
		{"a + add(b*c)+d", "((a + add((b * c))) + d)"},
		{"a*[1,2,3,4][b*c]*d", "((a * ([1, 2, 3, 4][(b * c)])) * d)"},
    }
}
//[...]
}
```
现在再一次运行测试，我们可以看到它们都通过了
```
$ go test ./parser
ok monkey/parser 0.008s
```
是的，所有的测试都通过了，好消息是我们已经做完了基本解析器的工作。在本书的最后我们还会再回来的。现在抽象语法树全部定义好，解析器也能正确工作。是时候将我们的话题转移到执行部分。

在做之前，染我么移除先前TODO的内容和拓展我们的REPL以便集成到解析器中。

**移除TODO**
在我们编写解析`let`和`return`语句的时候，我们跳过了解析表达式部分：
```go
func (p *Parser) parseLetStatement() *ast.LetStatement {
	stmt := &ast.LetStatement{Token: p.curToken}
	if !p.expectPeek(token.IDENT) {
		return nil
	}
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}
    //TODO: We're skipping the expression until we
    // encounter a semicolon
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}
```
同样的`TODO`也出现在`parseRetrunStatement`, 是时候清除它们了。首先，我们需要拓展我们已经存在的测试来确保我们正确解析了`let`或者`return`语句中的表达式部分。我们使用帮助函数（这样不会转移我们注意力）和不同的表达式类型，这样我们知道`parseExpression`被正确集成了。

下面是`TestLetStatement`函数看上去的样子：
```go
func TestLetStatements(t *testing.T) {
	tests := []struct {
		input              string
		expectedIdentifier string
		expectedValue      interface{}
	}{
		{"let x =5;", "x", 5},
		{"let z =1.3;", "z", 1.3},
		{"let y = true;", "y", true},
		{"let foobar=y;", "foobar", "y"},
	}
	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)
		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain 1 statements. got=%d",
				len(program.Statements))
		}
		stmt := program.Statements[0]
		if !testLetStatement(t, stmt, tt.expectedIdentifier) {
			return
		}
		val := stmt.(*ast.LetStatement).Value
		if !testLiteralExpression(t, val, tt.expectedValue) {
			return
		}
	}
}
```
在`TestReturnStatement`也有同样的需求，修正测试非常平常了，我们之前做了好多这样的工作。我们仅仅需要关注在`parseReturnStatement`和`parseLetStatement`中的`parseExpression`。而且我们主需要关注可选的分号，这个我们之间的`parseExpressionStatement`中已经知道了。新版的`parseReturnStatement`和`parseLetStatement`看上去如下
```go
func (p *Parser) parseLetStatement() *ast.LetStatement {
	stmt := &ast.LetStatement{Token: p.curToken}
	if !p.expectPeek(token.IDENT) {
		return nil
	}
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}
	p.nextToken()
	stmt.Value = p.parseExpression(LOWEST)
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}

// parse RETURN Statement
func (p *Parser) parseReturnStatement() *ast.ReturnStatement {
	stmt := &ast.ReturnStatement{Token: p.curToken}
	p.nextToken()
	stmt.ReturnValue = p.parseExpression(LOWEST)
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}
```
现在所有的TODO都移除了。
<h2 id="ch03-read-parse-print-loop">3.9 REPL</h2>
到目前为止，我们的REPL应该叫做`RLPL`(Reading-Lex-Print-Loop)。我们还不知道如何去执行代码，如何用`evaluate`去代替`lex`仍然还没有解决问题，但是现在我们能够确定的是解析。现在是时候用`parse`代替`lex`来构建`RPPL`。
```go
// repl/repl.go
func Start(in io.Reader, out io.Writer) { 
    scanner := bufio.NewScanner(in)
    for {
        fmt.Printf(PROMPT) 
        scanned := scanner.Scan() 
        if !scanned {
            return
        }
        line := scanner.Text() 
        l := lexer.New(line) 
        p := parser.New(l)
        program := p.ParseProgram() 
        if len(p.Errors()) != 0 {
            printParserErrors(out, p.Errors())
            continue
        }
        io.WriteString(out, program.String())
        io.WriteString(out, "\n") }
    }
    func printParserErrors(out io.Writer, errors []string) { 
        for _, msg := range errors {
            io.WriteString(out, "\t"+msg+"\n") 
        }
    }
}
```
现在我们拓展我们解析的循环部分，对输入到`REPL`中的每一行进行解析。输出的结果是解析出来的`*ast.Program`，然后调用`String()`方法来打印输出，它递归地调用每一语句中的`String`方法。现在我们可以在命令行中使用解析器。
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x = 1 * 2 * 3 * 4 * 5
let x = ((((1 * 2) * 3) * 4) * 5);
>> x * y / 2 + 3 * 8 - 123
((((x * y) / 2) + (3 * 8)) - 123)
>> true == false
(true == false)
>>
```
棒极了，除了调用`String`方法，我们可以调用任何抽象语法树代表字符串类型。我们可以增加`PrettyPrint`方法来输出抽象语法树的节点和内部孩子节点。

但是我们的`RPPL`还有一个巨大的缺陷：当遇到的错误的时候：
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3;
expected next token to be =, got INT instead
>>
```
这不是一个很好的错误消息，我的意思它完成了该完成的工作，但是不够友好。我们的Monkey语言值得更好的方式。下面有很好的`printParserError`函数，增加我们的的用户体验：
```go
// repl/repl.go
func printParserErrors(out io.Writer, errors []string) { 
    io.WriteString(out, "Woops! We ran into some monkey business here!\n") 
    io.WriteString(out, " parser errors:\n")
        for _, msg := range errors { io.WriteString(out, "\t"+msg+"\n")
    } 
}
```
现在好多了，当我们遇到任何解析错误，我们得到详细的错误信息
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language! Feel free to type in commands
>> let x 12 * 3
Woops! We ran into some monkey business here! parser errors:
expected next token to be =, got INT instead
>>
```

<h1 id="ch04-Evaluation">4 执行</h1>

<h2 id="ch04-giving-meaning-to-symbols">4.1 符号赋值</h2>

终于到达这一章：计算。在 `REPL`中的 `E` 就是解释器在处理源代码的的过程中的最后一步。在这一步，源代码将会变得有意义。没有计算，诸如 `1+2`这样的表达式仅仅是一连串的字符，token或者代表表达式的树状结构，而并没有意味着任何事情，当然，我们都知道 `1+2` 变成 `3`, `3.5 > 1` 是 `true`, `5<1` 是 `false`, `put("Hello World!")` 将会输出友好的消息。

计算的过程就是定义解释器如何将编程语言编程解释后的结果。
```
let num = 5;
if (num) {
    return a;
} else {
    return b;
}
```
是否返回 `a` 或者 `b` 将去取决于解释器在计算整数 `5` 是否为真。在一些语言中，它是真值，但是在其他语言中，我们需要用一个表达式来生成布尔值，如 `5 != 0`。

考虑如下
```
let one = fn() {
    printLine("one");
    return 1;
};

let two = fn() {
    printLine("two");
    return 2;
};
add(one(), two())
```
输出的结果是第一个 `one` 还是 `two` 函数亦或者是其他的？ 这个取决于特定的编程语言，最终是取决于解释器的实现，它是如何在一个函数调用的表达式中计算参数的顺序。

这样微小的选择方式在本章中还有很多，我们将决定在 Monkey 编程语言中如何进行工作，并且我们的解释器如何计算源代码。

或许你非常怀疑我告诉你写一个解析器非常好玩，但是请相信我，这事最重要的部分，在这部分Monkey编程语言将会变得有生命力起来，尤其是源代码变得运动起来，仿佛开始呼吸一样。

<h2 id="ch04-strategies-of-evaluation">4.2 执行策略</h2>
执行这一部分也是实现解释器过程中变化最多的部分，无论哪一种语言的实现。当计算源代码的时候，有很多不同的策略可以供选择。在本书简单介绍解释器架构的时候，我已经提示到这一点，现在我们手上已经有了抽象语法树（AST），现在问题来了，接下来我们需要做些什么，如何去计算这一棵树？我们接下来看看不同的观点。

在开始之前，值得注意的是解释器和编译器的界限是非常模糊的。常常来讲，解释器通常不离开可执行的环境（与此相关，编译器则脱离可执行环境）它并没有哪些现实中高优化的编程语言那么快。

就跟之前说的，显而易见经典的处理抽象语法树的方法就是解释它，遍历整个语法树，访问树种的每一个节点，并且计算出每一个节点的意义如：输出字符串，加两个数字，执行一个函数内部，等等如此。解释器这样工作的方式也叫“树的遍历”，这也是解释器的原型。有时在计算的过程中也做一些进一步优化处理，来重写抽象语法树（比如移除未使用的变量）或者将其改变成另一种中间形式的表达，对于递归或者循环等计算非常适用。

其他解释器同样遍历整个抽象语法树，与解释抽象语法树本身不同，它们首先将其转换成字节码。字节码同样也是抽象语法树的中间表达形式。这个特定的格式代码各式各样，主要取决于前段和宿主的编程语言。这种中间代码与汇编语言非常相似，可以打赌每一个字节码的定义中包含的 `push` 和 `pop` 等相关的栈操作数。但是字节码并不是原生的机器码，或者汇编语言。它不能被操作系统执行，CPU 也不是解释运行。它仅仅能够被虚拟机解释，这也是一个解释器，就跟 VMWare 和 VirtualBox 一样模拟真正的机器和CPU。虚拟机模式一个能理解这个特定格式的字节码的机器，这种处理方式能够提供一个很好的性能上的优势。

这种策略的变化一点也不影响抽象语法树，除了构建一个语法树，然后解析器直接生成字节码，那么是否现在我们仍然在讨论解释器或者编译器？难道生成字节码然后解释它（或许应该叫做执行）不是编译的一种形式？我想说的是：解释器和编译器之间的界线非常模糊，甚至令人感到糊涂。这样去想：一些编程语言的实现过程是这样的解析源代码，构建抽象语法树，然后将抽象语法树转变成字节码。但是不是在虚拟机上执行相关字节码的相关的操作，而是虚拟机将字节码编译成原生的机器码，仅仅是在执行之前，这种方式叫做 `JIT`解释器或者编译器。

其他编译器或者解释器跳过编程成字节码，它们递归地遍历整个抽象语法树，但是在执行一个特定的分支的所有节点之前将其编译成原生机器码，然后执行，同样也叫做 `JIT`。

一个微小的变种就是一种混合模式，解释器递归的遍历抽象语法树，只有当某一个特定的分支在计算很多次后才将其分支编译成机器码。

是不是很神奇？有很多种方法来完成计算这部分任务，其中有很多交叉和变种。

选择哪一种策略方式很大一部分取决于性能和可移植型的需要，以及你想要这个语言如何被解释。遍历整个语法树和递归的去计算可能是所有方法中最慢的一种，但是很容易去构建、拓展、思考以及可适配性。

将代码转换的字节码的的编译器在处理字节码上是非常快的，但是创建这种编译器也变得非常复杂和困难。将JIT转换成混合模式也需要你同时支持不同的计算机体系结构，一旦如果你想要解释器同时通过在ARM和x86架构的CPU上。

以上所有的实现方法在现实的编程语言中都能够被找到，大部分时候，实现的方式随着编程语言的发展而改变，Ruby就是很好的例子。在1.8以及之前的版本，Ruby的解释器是树遍历的，边遍历抽象语法树边执行，但是从1.9版本开始变成虚拟机架构，现在Ruby解释器解析源代码，构建抽象语法树然后将抽象语法树编译成字节码，并在虚拟机中执行，这一点咋性能上有了很大的提高。

WebKit Javascript的引擎 JavaScriptCore 和它的叫Squirrelfish同样采用抽象语法树遍历和执行的方式，在2008年之后转向了虚拟机和字节码解释。现在该引擎拥有4各部不同阶段的JIT编译，为了取得很好性能上的表现，在不同的阶段对程序进行不同形式的解释。

另一个例子就是Lua， 主要的实现方式是将其编译成字节码，然后在一个寄存器为基础的虚拟机上执行，在12年后，LuaJIT作为另外一种实现实现方式出现了。LuaJIT的实现者 Mike Pall的目标是创建一个尽可能快的Lua编译器。事实同样如此，通过JIT的方式将繁琐的字节码格式转换成不同基础架构的机器码，从各个性能测试来看，LuaJIT比原生的Lua解释器要快，而且不仅仅是一点点快，有时至少快50倍。

所以所有编译器都是从很小的改进空间开始，这也是我们开始要做的原因。有很多方法来构建更快的解释器，但是将不会很容易理解，在这里我们将理解和开始着手构建。

<h2 id="ch04-a-tree-walking-interpreter">4.3 树遍历计算</h2>
我们将要构建一个遍历树解释器，以前面解析步骤完成构建好的抽象语法树，边遍历边进行解释，跳过将预处理和编译的步骤。

我们的解释器将会和经典的Lisp解释器一样，我们采用的的设计受《计算机程序结构和描述》（The Structure and Interpretation of Computer Program）中描述的解释器的影响很大，尤其是关于环境的使用。但是这并不意味着我们想要去复制一个特定的解释器，如果你足够了解的话，我们是使用一个你在其他很多解释器中看得到的蓝图。这也是为什么这种特定的设计很流行的原因。它很容易的着手启动起来，也很容易理解和后期的拓展。

我们只需要执行两部分工作：一是遍历整个树计算；另一个是用我们的宿主语言Go来表达Monkey中的值。计算听上去好像很强大、很宏大。但是它仅仅是一个 `eval` 函数的调用，它这函数的唯一作用就是计算AST的值。接下来就是伪代码的版本来表来在解释器的上下文中什么是计算和树遍历。
```
function eval(astNode){
    if (astNode is integerliteral){
        return astNode.integerValue
    }else if (astNode is booleanLiteral){
        return astNode.booleanValue
    }else if (astNode is infixExpression) {
        leftEvaluated = eval(astNode.Left)
        rightEvaluated = eval(astNode.Right)
        if astNode.Operator == "+" {
            return leftEvaluated + rightEvalueated
        }esle if astNode.Operator == "-" {
            return leftEvaluated - rightEvalueted
        }
    }
}
```

正如你所见到的，`eval` 函数是递归的，如果 `astNode` 是 `infixExpression`，那么调用 `eval` 本身两次来分别计算左边操作数和右边操作数，它将会导致计算另外的中缀表达式计算或者整型计算或者布尔型计算亦或者一个变量。在构建和测试抽象语法树的过程中我们见到过这种递归的形式，在这里我们也是应用同样的概念，唯一区别是我们是计算而不是构建这颗抽象语法树。

从伪代码的片段我们可以大致想象对这函数进行扩展是非常简单的，这也是我们非常擅长的工作。我们将会一点一点地构建我们自己的 `eval` 函数，随着拓展我们的解释器，我们一点点增加新的分支和功能。

在这代码片段里，最好玩的是返回语句。它们到底是返回什么？下面两行是将函数调用的返回值绑定至相应的变量中。
```
leftEvaluated = eval(astNode.Left)
rightEvaluated = eval(astNode.Right)
```
那么它们返回的是什么？返回值的类型又是什么？这个问题的答案就是：在我们的解释器中拥有哪些内部的对象系统？

<h2 id="ch04-representing-objects">4.4 表达对象</h2>
等一下，什么？你从来没有说过Monkey是面向对象的编程语言啊！是的，我从来没有说过并且它也不是面向对象的编程语言。那为什么我们需要一个“对象系统”呢，亦或者是“值系统”或者“对象描述”？答案是我们需要定义我们计算返回的值。我们需要一个系统来描述我们抽象语法树种的值或者我们在内存中计算的生成出来的内容。

让我们来看看在接下来的Monkey代码中如何计算值
```
let a = 5;
// [...]
a + a;
```
正如你看到的，我们将一个字面整数绑到 `a` 的变量中。接下来无论发生什么，比如我们遇到了一个 `+` 的表达式，我们需要获取 `a` 绑定的值。 为了计算 `a+a`，我们需要去获取5。在抽象语法树中，它使用 `*ast.IntegerLiteral`对象来表达。但是我们如何去记录这些表达形式在计算剩余的抽象语法树的时候呢？

当在构建一门解释器语言的内部值表达的时候有很多种选择， 这方面的主题的智慧在全世界的关于解释器和编译的代码库中到处都有。每一个解释器都有其自己的实现方式，为了适应解释语言的要求，它们往往与先前的有点点不同。

在解释型语言中我们可以使用宿主语言来描述一些原生的类型（比如整型、布尔型等等），但是不是封装所有类型。在其他语言中值和对象都只能是用指针来表示，然而在其他的却是原生类型和指针混合使用。

为什么会有这么多种类呢？其中之一是宿主语言不同，如何在你的解释语言中描述一个字符串取决于你的实现解释器的语言的如何描述。一个用 Ruby 编写的解释器是不能用同样的方式用C语言来描述。

除此之外，被解释语言不同也是同样的原因。一些解释语言只需要仅仅描述原始的数据类型，比如整型、字符类型或者字节。但是其他的你需要有列表、字典、函数和复合数据类型。这些不同点导致了在值描述提出了不同的需求。

除了宿主语言和被解释的语言， 在设计和实现值描述影响最大的是在计算执行的过程中的速度和内存消耗。如果你想构建一个高效的解释器就不能使用膨胀的值系统；如果你想编写自己的垃圾收集器，你需要考虑如何在系统中跟踪每一个值。但是换一句话说，如果你不关心性能，保持解释器简单明了是非常重要，除非有更高的要求提出。

但是问题在于此， 在宿主语言中的解释语言有许许多多不同的值描述的方法。最好的方法，或许是唯一的方法去了解不同描述方法是去阅读这些流行的解释器的源代码。我发自内心推荐[Wren_source_code](https://github.com/munificent/wren)，它包含了两种不同的值描述方法，可以通过编译条件打开和关闭它们。

除了考虑宿主语言在值描述的问题，还需要考虑如何公开这些值描述给解释语言的使用者，也就是说我们公开的API接口长得什么样？

以JAVA语言举例，提供了基础数据类型（整型，字节，短整型，长整型，单精度浮点型，双精度浮点型，布尔型和字符）和引用数据类型给使用者。基础数据类型没有很大的描述在Java实现中，它们与原生的部分一一映射，但是引用数据类型是宿主语言中复合数据类型的引用。

Ruby语言的使用者不需要接触基础数据类型，类似原生值类型是不存在的，因为在Ruby中所有的都是对象，因此他们被封装到内部实现中。在Ruby内部不区分一个字节类型和一个 `pizza`类的实例对象，他们都是值类型，只不过封装了不同的值。

有许多方法将编程语言的数据类型暴露给使用者，这个去取决于如何设计者语言，以及我们在依次强调的性能要求。如果你不考虑性能问题，所有事情都好办了；但是你考虑了，你需要采取一些更聪明的选择来达到这个目标。

## 对象系统的基础

因为我们对Monkey解释器的性能暂时没有考虑，所以我们选择一种简单的方式：我们将要为我们每一个遇到的类型进行描述为一个 `Object`，它是我么设计的接口，每一个值都被封装到一个结构中，该结构将实现 `Object` 接口。

在新的 `object` 包中，我们定义 `Object` 接口和 `ObjectType` 类型：
```go
//object/object.go
package object
type ObjectType string

type Object interface {
    Type() ObjectType
    Inspect() string
}
```
它看上去很简单，和我们先前在 `token`包以及 `Token`和 `TokenType`类型非常像，不同点就是 `token`是一个结构，而 `Object`是一个接口。原因是每一个值都需要不同的内部表达而这样做很容易定义不同的结构类型而不是将布尔型和整型都放到同样的结构体字段中。

目前在Monkey解释器中我们只有三种数据类型: null，布尔型和整型。让我们开始实现整型并构建我们的值系统。

**Integers**

`object.Integer`类型就跟你想象中的一样小巧：
```go
import (
    "fmt"
)
type Integer struct {
    Value int64
}
func (i *Integer) Inpsect() string { return fmt.Sprintf("%d", i.Value) }
```
无论什么时候我们在源代码中遇到整型字面值，我们首先将其转换成一个 `ast.IntegerLiteral`，然后在计算那个抽象语法树的节点的时候，我们将其转换为一个 `object.Integer`，将它的值存入我们结构中然后这个结构的引用传出去。

为了让 `object.Integer`结构去实现 `object.Object`接口，仍然需要一个 `Type()`方法用来返回它的 `ObjectType`。就像我们在 `token.TokenType`，我们为每一个 `ObjectType`定义一个常量：
```go
// object/object.go
import "fmt"
type ObjectType string 
const (
    INTEGER_OBJ = "INTEGER"
)
```
正如我说的，就像我们在 `token` 包中所做的一样，这样我们就可以将 `Type()` 方法添加到`*object.Integer`结构中：
```go
//object/object.go
func (i *Integer) Type() ObjectType { return INTEGER_OBJ }
```
到此为止， 我们完成了 `Integer`类型，接下来进入下一个数据类型：布尔型。

**Booleans**
如果你在本小节中期待一个大的事情，我很抱歉你会失望的，`object.Boolean` 也是一个很简单的东西。
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
仅仅是封装了一个`bool`型变量的结构。

我们即将结束对象系统的基础内容，在来时我们 `Eval` 函数之前我们需要做一个其他的事情。

**Null**
1965年Tony Hoare 在 ALGOL W 语言中引入了空引用，也叫做他的[百万美元的错误](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。由于他的引用，空引用导致了无初次的系统奔溃，当一个描述中没有值得时候。Null(在其他语言中也称为nil)没有好的名声。

我自己也思考过，Monkey语言中是否应该有空引用，一方面来讲，不要引入，因为编程语言将会变得安全如果它不允许有空或者空引用，但另一方面来讲，我们并不是重新发明一个轮子，而是去学习一些东西。我发现我自己在处理null的时候导致我再次思考什么时候有机会试用它。就像当我车里有个爆炸性的东西会让我开车更慢更小心一点。这一点让我非常满意的选择能够设计一门编程语言。于是我实现了 `Null` 数据类型同时让我仔细小心当我在后面使用它的时候。
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
`object.Null`与`object.boolean` 和 `object.Integer` 一样，除了它没有封装任何值。它代表了值得缺失。

有了 `object.Null`, 我们的对象系统现在就可以代表布尔型、整型和空类型，对于我们开始`Eval` 函数足够了。

<h2 id="ch04-evaluaiton-expression">4.5 表达式计算</h2>

好了， 开始编写我们的 `Eval` 函数。我们现有拥有了抽象语法树和赞新的对象系统，这些让我们开始记录我们在执行Monkey代码的时候遇到的值，是时候考试计算抽象语法树。

这是我们 `Eval` 函数签名的第一版：
```go
func Eval(node ast.Node) object.Object
```
函数以一个 `ast.Node` 作为输入，返回一个 `object.Object`对象。要注意我们在 `ast`包中定义的每一个节点都实现了 `ast.Node` 接口，因此它们都可以传递给 `Eval` 函数。所以它允许我们在计算部分抽象语法树的时候递归地调用它自己。抽象语法树的节点需要不同形式的计算方式，而 `Eval`则决定如何去判别这些形式。举一个例子，我们传递一个 `*ast.Promgram`节点到`Eval`，那么 `Eval`应该做些做些什么去计算每一个 `*ast.Program.Statements`通过调用自身的时候每一个语句，返回值是我们在算计最后一个时候的返回值。

我们以实现自计算的表达式开始，也就是我们称之为字面计算值，简单来说就是布尔型和整型。它们是Monkey语言的基础也是非常容易去计算的，因为它们就是计算自身。如果我在`REPL`中输入5，那么5也是要输出的，如果我输入`true`，那么我将得到`true`。

听上去很简单？的确如此，让我们 “输入5， 得到5”变成现实。

**Integer Literals**
在开始写代码之前，想想它究竟意味着什么？ 我们将一个表达式语句作为一个输入，它只包含一个整型字面值，然后将其计算出来并返回。

转换为我们系统的语言就是，提供一个`*ast.IntegerLiteral`，我们的 `Eval` 函数应该返回一个`*object.Integer`对象，该对象包含一个 `Value` 字段，而且该值等于 `*ast.IntegerLiteral.Value`中的整型值。

我们很容易为新的`evaluator`包写出我们测试框架：
```go
// evaluator/evalutator_test.go

import (
    "monkey/lexer"
    "monkey/object"
    "monkey/parser"
    "testing"
)
func TestEvalIntegerExpression(t *testing.T){
    tests := [] struct {
        input string
        expected int64
    }{
        {"5", 5},
        {"10", 10},
    }
    for _, tt:=range tests{
        evaluated := testEval(tt.input)
        testIntegerObject(t, evaluated, tt.expected)
    }
}
func testEval(input string) object.Object {
    l := lexer.New(input)
    p := parser.New(l)
    program:=p.ParseProgram()
    return Eval(program)
}

func testIntegerObject(t *testing.T, obj object.Object, expected int64) bool{
    result, ok := obj.(*object.Integer)
    if !ok {
        t.Errorf("object is not Integer, got=%T (%+v)", obj, obj)
        return false
    }
    if result.Value != expected {
        t.Errorf("object has wrong value. got=%d, want=%d", result.VAlue, expected)
        return false
    }
}
```
一点也不奇怪，这就是我们刚刚说的内容，除了它目前还不能工作。测试依然是失败的因为 `Eval`函数返回的是 `nil` 而不是 `*object.Integer`
```
$ go test ./evaluator
--- FAIL: TestEvalIntegerExpression (0.00s)
    evalutor_test.go:36: object is not Integer. got=<nil>(<nil>)
    evalutor_test.go:36: object is not Integer. got=<nil>(<nil>)
FAIL
FAIL    monkey/evalutor     0.006s
```
失败的原因是我们从未遇到`*ast.IntegerLiteral`在`Eval`， 我们并没有遍历整个抽象语法树，我们应当从树的顶端开始，接受一个`*ast.Program`，然后递归的遍历每一个节点，但是事实上我们并没有这么做，而是仅仅等待一个 `*ast.IntegerLiteral`。 接下来的修改就是真正地遍历这颗树然后计算`*ast.Program`中的每一个语句。
```go
// evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
    switch node := node.(type){

        //Statements
        case *ast.Program:
            return evalStatements(node.Statements)
        case *ast.ExpressionStatement:
            return Eval(node.Expression)
        
        //Expressions
        case *ast.IntegerLiteral:
            return &object.Integer{Value: node.Value}
    }
    return nil
}

func evalStatements(stmts []ast.Statement) object.Object {
    var result object.Object
    for _, statements := range stmt {
        result = Eval(statements)
    }
    return result
}
```
在我们的`Monkey`程序中计算每一个语句，如果语句是一个`*ast.ExpressionStatement`，我们计算它的表达式，它反映的就是我们从一行输入如`5`的抽象语法树，它是一个只包含一个语句的程序，一个包含一个整型字面表达式表达式的语句（不是返回语句或者声明语句）。
```
$ go test ./evalutor
ok  monkey/evalutor 0.006s
```
好的，现在测试通过了，我们可以计算整型字面值了！*大家好，如果我么输入一个数字，通过几千行代码，我们就可以将其输出出来，测试也是同样如此*，但是这些与我们想象中的还是不是很像，不过就才是简单的开始，我们将看到如果进行计算工作并且如果去拓展我们的计算器。`Eval`的结构不会改变，我们仅仅是增加或者拓展它。

接下来要做的是布尔型字面值得计算，但是在开始做之前，我们应该先庆祝我们的一个计算的成功并且犒劳自己，让我们先把`REPL`中的`E`功能完成。

**完成REPL**
到现在为止，在我们的 `REPL`中的中的 `E`是缺失的，而且现在我们也仅仅只有 `REPL`（Read-Pare-Print-Loop）。现在我们有了`Eval`就可以构建一个真正的 `REPL`

在 `repl`包中使用计算器就跟你想象中的一样简单：
```go
//repl/repl.go
import (
// [...]
    "monkey/evaluator"
)
//[...]
func Start(in io.Reader, out io.Writer){
    scanner := buffio.NewScanner(in)
    for {
        fmt.Printf(PROMPT)
        scanned := scanner.Scan()
        if !scanned {
            return
        }
        line := scanner.Text()
        l := lexer.New(line)
        p := parser.New(l)
        program := p.ParseProgram()
        if len(p.Erros()) != 0 {
            printParseErrors(out, p.Error()
            continue
        }
        evaluated := evalutor.Eval(program)
        if evaluated != nil {
            io.WriteString(out, evaluated.Inspect())
            io.WriteString(out, "\n")
        }
    }
}
```
不是输出`program`(抽象语法树返回的值)，我么将`program`传递给 `Eval`函数，如果 `Eval` 函数返回一个非空的值，也就是 `*object.Object`对象，我们使用`Inspect()`方法将其输出。在`*object.Integer`中输出的结果就是其封装的整数值的字符串。

我们使用 `REPL`可以这样工作：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>>5
5
>>10
10
>>999
999
```
是不是感觉很不错？词法解析、语法解析、计算都在这里。


**布尔字面值**
布尔字面值就跟我刚刚遇到的整型一样，用来计算它们自己：`true`计算返回`true`,`false`计算返回`false`。在`Eval`中实现这个就跟实现整型字面值一样简单，接下来就是枯燥的测试。
```go
// evaluator/evaluator_test.go
func TestEvalBooleanExpression(t *testing.T) {
    tests := []struct {
        input string
        expected bool
    }{
        {"true", true},
        {"false", false},
    }
    for _, tt := range tests {
        evaluated := testEval(tt.intput)
        testBooleanObject(t, evaluated, tt.expected)
    }
}
func testBooleanObject(t *testing.T, obj object.Object, expected bool) bool {
    result, ok := obj.(*object.Boolean)
    if !ok {
        t.Errorf("object has wrong value. got=%T(%+V)", obj, obj)
        return false
    }
    if result.Value != expected {
        t.Errorf("object has wrong value. got=%t, want=%t", result.Value, expected)
        return false
    }
    return true
}
```
以后我们会拓展`tests`切片以便能够支持更多的表达式而不是布尔型。现在我们只需要确保当我们输入`true`和`false`能够得到正确的输出。这个测试目前当然是失败的：
```
$ go test ./evaluator
--- FAIL: TestEvalBooleanExpression (0.00s)
    evaluator_test.go:42: Eval didn't return BooleanObject. got=<nil>(nil)
    evaluator_test.go:42: Eval didn't return BooleanObject. got=<nil>(nil)
FAIL
FAIL mongkey/evalutor 0.006s
```
为了使测试通过也非常简单，只需要将`*ast.IntegerLiteral`分支拷贝过来并做一些简单的修改即可：
```go
//evaluator/evalutor.go
func Eval(node ast.Node) object.Object {
//[...]
    case *ast.Boolean:
        return &object.Boolean{Value: node.Value}
}
```
让我们看看在`REPL`如果进行工作的
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> true
true
>> false
false
>>
```
完美!但是等等，我们是不是在这里每一个`true`或者`false`的时候都创建一个`object.Boolean`对象是不是有点不对劲？在两个`true`或者`fasle`内部之间是并没有什么不同的，但是我们为什么每次都使用新的实例对象呢？在这里只有连个不同的值，因此在这里我们只需要引用即可，而不是创建一个新的。
```go
// evaluator/evalutor.go
var (
    TRUE = &object.Boolean{Value: true}
    FALSE = &object.Boolean{Value: false}
)
func Eval(node ast.Node) object.Object {
// [...]
    case *ast.Boolean:
        return nativeBoolToBooleanObject(node.Value)
// [...]
}
func nativeBoolToBooleanObject(input bool) *object.Boolean {
    if input{
        return TRUE
    }
    return FALSE
}
```
现在在我们包中只有两个`object.Boolean`对象实例：`TRUE`和`FALSE`，我们引用它们而不是申请空间去创建它们。这个对我们性能小小提升很具有意义，而这些并不需要很多的工作。我们在`null`类型中同样这么处理的。


**Null**

就跟布尔型`true`和`false`各只有一个一样，对于 `null` 类型也应该只有一个，没有其他空类型的变种，没有任何其他乱七八糟的空类型，只有一个空类型。一个对象要么为空，要么不为空。所以我们首先创建一个`NULL`对象，以便我们在整个计算过程中都能引用它。
```go
// evaluator/evaluator.go
var (
    NULL = &object.NUll{}
    TRUE = &object.Boolean{Value:true}
    FALSE = &object.Boolean{Value:false}
)
```
因此我们只有一份`NULL`引用。

有了整型字面值和三个`NULL`, `TRUE` 和 `FALSE`，我们可以开始准备计算我么你操作表达式。

**前缀表达式**

在Monkey中中最简单的操作数表达式就是就前缀表达式，即单操作数表达式，也就是操作数跟在操作符之后。在我们先前解析过程中提到过，很多语言构造喜欢采用前缀表达式，因为这样解析它很简单。但是在本小节中的前缀表达式就是由一个操作数和一个操作符组成的操作符表达式。Monkey语言支持两种前缀操作数:`!`和`-`。

计算操作符表达式（尤其是前缀操作和操作数）并不难，我们一点点实现构建我们设计出来的行为。但是我们要特别注意的是，我们我们想要达成的结果远超我们预期。要记住，在计算的过程中处理输入语言的意义，我们定义了Monkey语言的文法，一个微小的改变在计算操作符表达式的时候会导致一系列无法预知的问题，测试能帮助我们判断是我们想要的结果。

首先我们开始实现支持`!`操作符，测试展示了这个操作符它的操作符变成一个布尔值，然后取反。
```go
// evaluator/evalutor_test.go

func TestBangOperator(t *testing.T){
    tests := []struct{
        input string
        expected bool
    }{
        {"!true", false},
        {"!false", true},
        {"!5", false},
        {"!!true", true},
        {"!!false", false},
        {"!!5", true}
    }
    for _, tt := range tests {
        evaluated := testEval(tt.input)
        testBooleanObjec(t, "evaluted", tt.expected)
    }
}
```

正如我说的，这就是我们让着语言如何工作方式，`!true`和`!false`表达式和它们的期望值看上去很合理，但是`!5`其他语言设计者觉得应该是返回一个错误，但是我们想要说的是`5`行为上表现就是`truthy`.

这个测试肯定不能通过，因为 `Eval`返回一个`nil`而不是`TRUE`或者`FALSE`. 计算前缀表达式第一个就是计算他的操作数，然后用它的操作符来计算结果。
```go
// evalutor/evalutor.go
func Eval(node ast.Node) object.Object {
//[...]
    case *ast.PrefixExpression:
        right :=Eval(node.Right)
        return evalPrefix(node.Operator, right)
}
```
在第一步调用后，右边的值可一个是 `*object.Integer` 或者是 `*object.Boolean`甚至可能是一个 `NULL`。 我们按着右边的操作数然后传递到`evalPrefixExpression`函数中，它用来检查这个操作符是否支持。
```go
// evalutor/evaluator.go
func evalPrefixExpression(operator string, right object.Object)object.Object{
    switch operator{
        case "!":
            return evalBangOperatorExpression(right)
        default:
            return NULL
    }
}
```
如果操作符不支持就返回NULL，这是最好的选择吗？或许是，或许也不是。但是到目前为止，这显然是最好的选择，因为我们还没有实现其他任何错误方式。

`evalBangOperatorExpression` 函数及时 `!` 操作数具体的操作。
```go
func evalBangOperatorExpression(right object.Object) object.Object {
    switch right {
    case TRUE:
        return FALSE
    case FALSE:
        return TRUE
    case NULL:
        return TRUE
    default:
        return FALSE
    }
}
```
当然，我们的测试全部通过
```
$ go test ./evalutor
ok  monkey/evaluator 0.007s
```

让我们继续 `-` 前缀操作符，我们可以拓展我们的 `TestEvalIntegerExpression` 测试函数以纳入一下用例：
```go
//evalutor/evalutor_test.go
func TestEvalIntegerExpression(t *testing.T){
    tests := []struct {
        input string
        expected int64
    }{
        {"5", 5},
        {"10", 10},
        {"-5", -5},
        {"-10", -10},
    }
// [...]
}
```

为了测试`-`前缀表达式， 我选择拓展测试而不是重新编写一个测试方法主要是两个原因：一是整型是唯一支持`-`操作符的前缀操作数；第二是测试方法应该包含所有整型的运算方法以便达到清晰的目的。

我们已经提前拓展了 `evalPrefixExpression` 方法以便能让测试通过，只需要在switch语句下添加新的分支：
```go
// evaluator/evalutor.go
func evalPrefixExpression(operator string, right object.Object) object.Object {
    switch operator{
    case "!":
        return evalBangOperatorExpression(right)
    case "-":
        return evalMinusPrefixOperatorExpression(right)
    default:
        return NULL
    }
}
```
`evalMinusPrefixOperatorExpresion` 函数看上去像这样：
```go
// evalutor/evalutor.go
func evalMinusPrefixOperatorExpression(right object.Object)object.Object {
    if right.Type() != object.INTEGER_OBJ {
        return NULL
    }
    val := right.(*object.Integer).Value
    return &object.Integer{Value : -value}
}
```
首先先检查操作数是否为整数，如果不是，返回NULL；如果是，我们提取 `*object.Integer`中的值，然后重新分配一个对象来封装它的相反值。

是不是只需要做简单的事，但是他的确能够工作:
```
$ go test ./evaluator
ok  monkey/evalutor 0.0007s
```
棒极了，在继续中缀表达式之前，我们可以在REPL中给出我们前缀表达式的值：
```
$ go run main.go
Hello mrnugget! This is the Monkey programming language！
Feel free to tyoe in comamnds
>> -5
5
>> !true
false
>> !-5
false
>> !! -5
true
>> !!!! -5
true
>> -true
null
```

**中缀表达式**
作为一个新人，下面是八个Monkey语言支持的的中缀表达式
```go
5 + 5;
5 - 5;
5 * 5;
5 / 5;

5 > 5;
5 < 5;
5 == 5;
5 != 5;
```
这八个操作符可以被分为两组，一组的操作符生成布尔型值作为结果，另外一组生成。我们开始实现第二组操作符 `+, -, *, /`。刚开始我们只支持整数操作数，只要他能工作，我们就支持操作数两边是布尔型值。

测试框架已经准备就绪，我们仅仅只要拓展 `TestEvalIntegerExpression`测试方法来适应新的操作符。
```go
//evalutor/evalutor_test.go
func TestEvalIntegerExpression(t *testing.T){
    tests :=[] struct {
        input string
        expected int64
    }{
       {"5", 5},
		{"10", 10},
		{"5", 5},
		{"10", 10},
		{"-5", -5},
		{"-10", -10},
		{"5+5+5+5-10", 10},
		{"2*2*2*2*2", 32},
		{"-50+100+ -50", 0},
		{"5*2+10", 20},
		{"5+2*10", 25},
		{"20 + 2 * -10", 0},
		{"50/2 * 2 +10", 60},
		{"2*(5+10)", 30},
		{"3*3*3+10", 37},
		{"3*(3*3)+10", 37},
		{"(5+10*2+15/3)*2+-10", 50},
    }
//[...]
}
```
这里有些测试用例可以删除，因为他们与其他的很有些重复并且增加了一些新的。实话来讲，我是非常高兴当这些测试通过的时候，我明白了我们所做的工作完成了，忍不住问自己，是不是这么简单？实际情况是的确这么简单。

为了让这些测试用例通过，首先要做的是拓展在`Eval`函数中的`switch`语句：
```go
//evaluator/evalutor.go
func Eval(node ast.Node) objet.Object{
// [...]
    case *ast.InfixExpression:
        left := Eval(node.Left)
        right := Eval(node.Right)
        return evalInfixExpression(node.operator, left, right)
// [...]
}
```
就跟`*ast.PrefixExpression`一样，我们首先计算操作数。 现在我们有两个操作数，左边和右边各一个抽象语法树节点，我么已经知道，这可能是其他的表达式、一个函数调用、一个整型字面值、一个操作符表达式等等。我们不去关心这个，让`Eval`函数去关心这个。

在计算完操作数之后，我们将两个返回值和操作符传递到 `evalIntegerInfixExpressions`函数中去，函数是这样子的：
```go
func evalInfixExpression(
    operator string,
    left, right object.Object,
)object.Object {
    switch {
    case left.Type()==object.INTEGER_OBJ && right.Type()==object.INTEGER_OBJ:
        return evalIntegerInfixExpression(operator,left,right)
    default:
        return NULL
    }
}
```
正如我刚刚保证的，一旦两边的操作数都不是整数的时候，我们就返回`NULL`, 当然我们后面将要拓展我们的函数，但是为了测试通过，这就足够了。重点在于`evalIntegerInfixExpression`函数中，在该函数中，我们封装的`*objet.Integers`的操作有加、减、乘和除。
```go
// evalutor/evalutor.go
func evalIntegerInfixExpression(
    operator string
    left, right object.Object,
)object.Object{
    leftVal := left.(*object.Integer).Value
    rightVal := right.(*object.Integer).Value
    switch operator{
    case "+":
        return &object.Integer{Value:leftVal+rightVal}
    case "-":
        return &object.Integer{Value:leftVal-rightVal}
    case "*":
        return &object.Integer{Value:leftVal*rightVal}
    case "/":
        return &object.Integer{Value:leftVal/rightVal}
    default:
        return NULL
    }
}
```
现在信不信由你，测试通过了：
```
$ go test ./evalutor
ok monkey/evalutor 0.007s
```

好的，我们继续前进，我们过会儿会再回到这边，以便支持哪些能够生成布尔值的操作符`==`,`!=`,`<`和`>`。

我们可以拓展我们的`TestEvalBooleanExpression`方法，为上述的操作符增加测试用例，*因为它们都能生成布尔型值。

```go
//evaluator/evaluator_test.go
func TestEvalBooleanExpression(t *testing.T){
    tests := []struct {
        input string
        expected bool
    }{
        {"true", true},
		{"false", false},
        {"1<2", true},
		{"1>2", false},
		{"1<1", false},
		{"1>1", false},
		{"1==1", true},
		{"1!=1", false},
		{"1==2", false},
		{"1!=2", true},
    }
}
```

除此之外，我们还需要增加一些代码在 `evalIntegerInfixExpression`函数中，它们能够保证测试能够通过：
```go
// evaluator/evaluator.go
func evalIntegerInfixExpression(
    operator string
    left, right object.Object,
) object.Object {
    leftVal := right.(*object.Integer).Value
    rightVal := right.(*obejct.Integer).Value
    switch operator {
// [...]
    case "<":
        return nativeBoolToBooleanObject(leftVal < rightVal)
    case ">":
        return nativeBoolToBooleanObject(leftVal > rightVal)
    case "==":
        return nativeBoolToBooleanObject(leftVal == rightVal)
    case "!=":
        return nativeBoolToBooleanObject(leftVal != rightVal)
    default:
        return NULL
    }
}
```

`nativeBoolToBoolean`方法是我们用在布尔字面值的时候，现在我们在比较未封装的值比较的过程中又重新使用了它们。

至少对于整数来说，我们现在已经完全支持八种中缀操作符，剩下的工作就是支持布尔型操作数。

Monkey语言支持的布尔操作数是相等判别符`==`和`!=`。它不支持布尔型数值的加减乘除。检查`true`是否比`false`大，`<`和`>`能够做或运算都是不支持的，这些都减少了我们的工作量。

首先我么你需要做的事情，你知道的，就是增加测试内容，就跟以前一样，我们拓展已有的测试方法，我们使用`TestEvalBooleanExpression`函数并增加`==`和`!=`操作符的测试用例。

```go
func TestEvalBooleanExpression(t *testing.T){
    tests := []struct{
        input string
        expected bool
    }{
// [...]
        {"true == true", true},
		{"false == false", true},
		{"true == false", false},
		{"true != false", true},
		{"(1<2)==true", true},
		{"(1<2) == false", false},
		{"(1>2) == true", false},
        {"(1>2)==false", true},
    }
// [...]
}
```
严格来讲，只需要五个测试用例就足够了，但是我们又增加了四个测试用例来检查生成布尔型值得比较。

到目前为止，没有任何惊奇的地方，仅仅又是一系列失败的测试用例
```
$ go test ./evalutor
--- FAIL: TestEvalBooleanExpression (0.00s)
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
evalutor_test.go:121: object is not Boolean. got=*object.Null {&{}}
FAIL
FAIL monkey/evalutor 0.007s
```
接下来就是增加一些简单的东西让测试通过：
```go
//evalutor/evalutor.go
func evalInfixExpression(
    opertor string 
    left, right object.Obejct,
)object.Object {
    switch {
// [...]
    case operator == "==":
        return nativeBoolToBooleanObject(left == right)
    case oeprator == "!=":
        return nativeBoolToBooleanObject(left != right)
    default:
        return NULL
    }
}
```

是的，我们只是在已经存在的`evalInfixExpression`函数中增加四行代码，测试就通过了，我们通过指针的比较来检查两个布尔型值之间的相等。这样做的原因是我们指向布尔型的指针只有两个`TRUE`和`FALSE`，如果有其他值也是`TRUE`，也就是内存地址一样，它就是`true`。对于`NULL`也是同样的道理。

但是对于整型或者其他数据类型并不奏效，因为对于`*object.Integer`我们每次都分配内存来生成`object.Integer`实例因而我们就能有新的指针。我们无法比较指向不同实例的指针。我们无法比较指向不同实例的指针，否则像`5==5`就会是false，而这个并不是我们想要的。在这种情况下，我们要明确的指出是比较值而不是封装值得对象。

这也是为什么我们在`switch`语句中首先检查整型部分的分支，它们将会首先进行匹配，只要我们留心其他啊操作数的类型是在指针比较之前，我们的成果就能很好的工作。

想象一下，如果十年之后，Monkey语言变得出名了，许多人开始来研究讨论。忽视了这个半吊子地设计这门语言，那么我们就变得非常出名了。 有人就是去StackOverflow上去问，为什么在Monkey语言中，整型比较会比布尔型比较慢得多。你和我，或者其他人就会回答到，因为在Monkey语言中，不允许使用指针比较整型数据，在比较之前，需要拆封它们的值，然后进行比较。相对而言，布尔型操作比较就比较快。 我们将会加上答案的最后加上这么一句”因为源代码是我写的“。

有点跑题了，回到正题。我们做到了，相当高兴，差不多都可以开始庆祝了。是时候开香槟庆祝了吗？对的，看看我们的解释器现在能做什么！

```
$go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> 5 * 5 + 10
35
>> 3 + 4 * 5 == 3 * 1 + 4 * 5
true
>> 5 * 10 > 40 + 5
true
>> (5 > 5 == true) != false
false
>> 500 / 2 != 250
false
```
到目前为止，我们已经完成了一个函数计算器， 接下来让我继续增加，使它变得更像一个编程语言。
<h2 id="ch04-conditionals">4.6 条件语句</h2>
你将会惊奇在我么的计算器中实现条件语句如此简单，实现他们唯一的难点就是知道在何时实现他们，整个条件语句最关键点就是在于条件判断，而且计算过程与条件判断息息相关，考虑到这种情况：
```go
if (x>10){
    puts("everything okay!")
}else{
    put("x is too low!")
    shutdownSystem()
}
```
在计算`if-else`表达式中最重要的事是计算正确的分支，如果条件满足为true，我们不必计算`else`分支，只计算`if`分支即可，如果不满足，只需要计算`else`分支即可。

换句话说，也就是我们计算`else`分支是当计算条件`x>10`为...? 好像并不是非常准确，难道我们要计算`everyhing okay!`分支吗，当这个条件表达式为`true`或者它能推导出类似`true`的值吗，也就是非假或者非空？

这是最难的部分，因为这是设计选择，语言的选择部分必须要正确，正确处理代码序列。

在Monkey语言例子中，条件语句将会被执行，当条件为类`true`的时候：
```go
let x = 10;
if(x){
    puts("everything okay");
}else{
    puts("x is too high");
    shutdownSystem();
}
```
在上述的例子中，`everything okay`将会被答应出来。为什么呢？ 因为`x`将会被绑定到10， 计算10并且10不是空，也不是假值。 这就是条件语句在Monkey语言中如何工作的。

跟先前讨论一样，我们先增加一些测试用例：
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
这个测试函数的作用我们还没有讨论，当条件没有没有被执行，它的返回值就是NULL:
```go
if (false) { 10 }
```
因为`else`分支丢失，因此条件语句将会生成`NULL`。

为了做一些类型判断和转换，我们在`field`字段中修改了部分内容。但是测试用例非常简单易读，很清楚地反应了我们想要做些什么。当然测试肯定是失败的，因为我们并没有返回`*object.Integer`或者`NULL`：
```shell
$ go test ./evalutor
--- FAIL: TestIfElseExpression(0.00s)
....
FAIL
FAIL monkey/evalutor 0.007s
```
先前我告诉你，你将会非常吃惊，支持条件语句实现过程如此简单，现在信不信我了？现在只需要增加一些代码，以便测试通过：
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
正如我先前所说的，难点就在于决定计算那个分支，所有的决定分支部分在封装到逻辑步骤非常清楚的`evalIfExpresion`函数中，`isTruthy`是相等判断式。上述两个函数也增加了`*ast.BlockStatement`条件分支，因为`*ast.IfExpression`中的`.Consequences`和`.Alternative`都是语句块。

我们增加了两个新的具体函数来表达Monkey语言的相关的语法。重用了已经存在的函数，仅仅增加一些以便支持条件语句，然后我们的测试通过。现在我们的解释器支持`if-else`表达式。我们现在离开简单的计算器领域，开始像编程语言进军：
```
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> if (5*5+10>34) { 99 } else { 100 }
99
>> if ((100/2) + 250 * 2==1000){9999}
9999
```
<h2 id="ch04-return-statement">4.7 返回语句</h2>
接下来的返回语句，这个在任何标准的计算器都不会出现的，但是`Monkey`有。 它不仅仅在函数体而且作为`Monkey`语言的顶层的语句。 它在任何地方使用都不会有任何影响，因为它不改变任何东西。返回语句将停止后面的所有计算，并且带着它计算的值离开。

这儿有个一个最上层的返回语句：
```go
5 * 5 * 5;
return 10;
9 * 9 * 9;
```
执行这个程序将会返回10. 如果这些语句在一个函数体内，那么调用者将会得到10。最重要的是最后一行代码：`9 * 9 * 9`表达式将永远不会被执行。

有一些不同的方式去实现返回语句，在一些宿主语言中我们可以使用`goto`或者异常等方式。但是在`go`语言中，实现异常捕获非常困难，而且我们也不想采用`goto`这种不简洁的方式。为了支持返回语句，我们会传递一个`返回值`给执行器， 当我们遇到一个 `return`, 我们将会其封装起来，并返回里面的对象，因此我们能够跟踪记录它，以便是否决定是否继续计算。

接下来就是刚刚说的对象`object.ReturnValue`的实现：
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
仅仅是封装了其他的`object`对象，真正有趣的是当它在使用的时候。

接下来的测试示范了我们在Monkey编程上下文中使用返回语句:
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
为了让测试通过，我们需要在`evalStatement`做一些修改，我们已经在`Eval`函数中增加了一个`*ast.ReturnStatement`分支：
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
首先需要修改的部分是执行器的`*ast.ReturnValue`，在这我们执行带有return语句的表达式。然后将`Eval`计算得到的结果封装成`object.ReturnValue`对象并记录跟踪它。

在`evalStatement`语句中，我们执行`evalProgramStatement`和`evalBlockStatement`方法来分别计算一系列的语句。我们检查每一步执行的结果是不是`object.ReturnValue`，如果是则停止计算，然后返回那个被封装的值。有一点非常重要，我们不返回`object.ReturnValue`，而是返回它封装的值，这也是用户所需要被返回的值。

但是问题是我们有时候想持续跟跟踪`object.ReturnValues`而不是一遇到就把获取未封装的值。这个在语句块中经常遇到。
```go
if (10 > 1) {
    if (10 > 1){
        return 10;
    }
    return 1;
}
```
这个程序应该返回10，但是在目前我们版本中，它只是返回1，一个小的测试可以确认：
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
正如我们所期待的，这个测试是失败的
```
$ go test ./evalutor
--- FAIL: TestReturnStatements(0.00s)
    evaluator_test.go:159: object has wrong value. got=1, want=10
FAIL
FAIL monkey/evalutor 0.007s
```
我敢打赌你已经发现我们当前的版本出现的问题，但是还是让我说出来：如果我们有嵌套的语句块（这是在Money语言中完全合法的）我们不能一遇到`object.ReturnValue`就将封装的值取出来，因为我们还要继续跟踪该值，直到我们到达最外面一层语句块，停止执行。

在当前版本中非嵌套的语句块可以很好地执行，但是遇到嵌套的语句块，首先要做的事承认我们不能再继续使用`evalStatement`函数来执行语句块。这也是我们为什么要重新命名`evalProgram`函数让其不是那么泛化。

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
为了执行`*ast.BlockStatement`，我们引入了新的函数`evalBlockStatement`:
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
在这里对每一个执行结果，我们显式说明不拆封返回值只是检查其`Type()`方法，如果它是`object.RETURN_VALUE_OBJ`， 我们就简单的返回`*object.ReturnValue`而不是取出其中的`.Value`字段，所以它停止执行可能的外部语句块，像气泡一样一直到达`evalProgram`方法，然后取出被封装的值（在后面设置到函数调用的时候，这部分将会做出一些改变）

测试通过：
```
go test ./evalutor
ok monkey/evalutor 0.007s
```
返回语句完成，我们终于不再是构建一个计算器。由于`evalProgram`和`evalBlockStatement`对我们来讲还是很陌生，我们将继续研究它们。

<h2 id="ch04-error-handling">4.8 错误处理</h2>

还记得我们先前返回`NULL`对象， 我说过你现在不用担心这个，过会我们会回来处理的。现在正是处理真正的错误时候，以免后面太迟了。总体上来讲，我们只需要回退一小部分先前的代码。老实地说，我先前并没有一开始就实现错误处理是因为我认为实现表达式比处理处理有趣多了。但是现在我们必须把它加上，否则将来调试起来我们的解释器将会非常笨重。

首先，让我们先定义一下什么是真正的错误处理机制，这个并不是什么用户自定义异常，而是内部的错误处理。是那些错误操作符，不支持的操作运算亦或者是在执行过程中出现的用户或者内部的异常。

至于这些错误处理方式的实现由多种方法，但是大多数都是处理返回语句，原因也很简单，因为错误和返回语句是相似的，都是停止执行一系列语句。

首先我们需要处理一个错误对象
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
你可以看到 `object.Error` 真的非常简单， 它仅仅封装了一个`string`对象来处理错误消息。在一个生产的解释器中，我们需要将栈调用信息给这个错误对象，通过增加一些错误的所在的具体行和列信息添加到消息中。这些并不难实现，只需要在词法解析的过程中将行和列号添加到其中去。至于为什么我们的解释器并没有这么做，是因为我们想把事情做得简单一点，我们只需要一个错误消息即可， 它能给我的很多具体的反馈信息并且能够停止执行所有代码。

我们会在几处合适的地方增加错误机制，以便提高我们解释器的容错能力。现在测试函数能够展示我们的错误处理机制如何工作的：
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

但我们运算测试的时候，又遇到了我们的老朋友`NULL`。
```
$ go test ./evaluator
--- FAIL TestErrorHanding (0.00s)
---
FAIL
FAIL monkey/evaluator 0.007s
```
但是在这出现了`*object.Integer`对象， 这是因为这些测试用例只实际上判断了两件事情：一是对于不支持的操作生成一些错误异常；二是阻止这些错误异常进一步执行。当`*object.Integer`被返回的时候，这些测试失败了，执行过程也并没有正确的停止。

生成错误异常并且传递到`Eval`函数很简单，我们只需要一个辅助函数来帮助我们在我们认为需要的地方创建一个`*object.Errors`对象即可。
```go
//evaluator/evalutor.go
func newError(format string, a ...interface{}) *object.Error{
    return &object.Error{Message: fmt.Sprintf(format, a...)}
}
```
这个`newError`函数用在我们不知道不知道返回什么值得时候，而不是简单地返回`NULL`:
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
通过上述的的改变，先前的一些列的失败测试用例只剩下两个
```
$ go test ./evaluator
--- FAIL TestErrorHanding (0.00s)
---
FAIL
FAIL monkey/evaluator 0.007s
```
上述的输出表明，通过生成错误的确可以停止执行。我们已经知道如何继续跟踪下去，是的在`evalProgram`和`evalBlockStatement`函数中，我们在这些函数的入口增加一些错误处理过程。
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
这样做的的话，执行器在正确的位置停下来并且测试通过：
```
$ go test ./evalutor
ok monkey/evaluator 0.010s
```
最后还需要做一件事，我们需要检查当我们在调用`Eval`函数的的时候是否有错误异常。为了防止错误被传递，我们需要将其从源头上浮到程序执行的地方：
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
好了，错误异常处理完毕。
<h2 id="ch04-binding-and-environment">4.9 绑定和环境</h2>
接下来我们要做的事是增加绑定用来以便支持`Let`语句。但是不仅仅我们需要支持`Let`语句，还要支持变量的执行。用一个代码片段来说明我们要去执行的内容：
```go
let x = 5 * 5;
```
仅仅支持上述的这种语句是不够的，我们同样也需要确保解释器在执行`x`的时候得到的是10。

所以本小节的任务是执行`let`语句和变量。我们在执行`let`语句的时候通过计算右边的表达式时候获取一个值，并且将该值保存到一个具体名称的变量中。在执行变量的时候的，我们首先查看该变量是否我们已经绑定了特定的值。如果已经绑定，则执行该值，否则我们返回一个错误。

听上去好像很不错的方案？好的，首先开始一些测试：
```go
func TestStatement(t *testing.T){
    tests := []struct {
        input string
        expected int64
    }{
        {"let a = 4; a ;" , 5},
        {"let a = 5 * 5; ", 25},
        {"let a = 5; let b = a; b;", 5},
        {"let a = 5; let b = a; let c = a + b; c;", 15},
    }
    for _, tt := range tests {
        testIntegerObject(t, testEval(tt.input), tt.expected)
    }
}
```
这些测试主要做了两件事：一是执行`let`语句中那些可以产生值的表达式；二是执行那些绑定值得变量。但是我们也需要一些测试来确保当我们执行那些未绑定值得变量时候生成一些错误。因此我们简单地拓展已经存在的`TestErrorHandling`函数：
```go
//evalutor/evaluator_test.go
func TestErrorHanding(t *testing.T){
    tests := []struct {
        input string
        expectedMessage string
    }{
        {
            "footbar",
            "Identifier not found: foobar",
        },
    }
// [....]
}
```
但是我们如何让我们的测试通过呢？显然第一件事就是我们需要在`Eval`再增加一个用例分支`*ast.LetStatment`，在这个分支中，我们计算的分支表达的是否正确？
```go
//evalutor/evaluator.go
func Eval(node ast.Node) object.Object {
// [...]
    case *ast.LetStatement:
        val := Eval(node.Value)
        if isError(val){
            return val
        }
    //Huh? Now what?
}
```
正如注释中所说的，接下来怎么办呢？我们改如何记录值呢？我们有一个值和它要绑定的变量名。我们该如何将其中的一个绑定到另一个中呢？

在这里我们所说的环境将要发挥作用，环境就是我们用来记录一个值和其相应变量的名称。因为在其他解释器中也采用环境概念描述，这是个传统，尤其是`Lisp`家族语言。尽管这个名称听上去很复杂，但是在环境最核心的就是一个哈希表，用来将字符串和对象关联起来，这也是我们接下来我们要去实现的。

我们将`Environemt`结构添加到`object`包中，是的，它仅仅将`map`封装起来。
```go
package object
func NewEnvironment() *Environment{
    s := make(map[string]Object)
    return &Environment{store: s}
}
type Environment struct {
    store map[string]Oject
}
func (e *Environemt) Get(name string) (Object, bool) {
    obj, ok := s.store[name]
    return obj, ok
}
func (e *Environemt) Set(name strong, val Oject)object {
    s.store[name] = val
    return val
}
```
让我猜猜你现在想些什么，为什么不直接时候`map`?为什么要做一层封装？到后面我们开始实现函数和函数调用的时候你就明白了，这是我们接下来工作的基础。

正如其用法`object.Environment`，这能够自解释，但是我们我们如何在`Eval`函数中使用呢？怎么或者如何记录这个环境呢？我们在`Eval`函数中作为一个参数传递过去。
```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) objec.Object {
//[...]
}
```
除此之外，不需要做其他改变，因为我们需要修改其他任何调用`Eval`函数以便增加环境的使用。不仅仅是`Eval`函数内部调用`Eval`函数，而且是其他一些函数比如`evalProgram`,`evalIfExpression`等调用`Eval`的函数。这些都需要手动的编辑的工作， 因此在这里不展示这些改变的工作。

与此同时`REPL`也调用了`Eval`函数，因此也需要增加环境。当然在`REPL`中只需要一个环境值即可：
```go
//repl/repl.go
func start(in io.Reader, out io.Writer){
    scanner := bufio.NewScanner(in)
    env := object.NewEnvironment()
    for {
// [...]
        evaluated := evaluator.Eval(program, env)
        if evaluated != nil {
            io.WriteString(out, evaluted.Inspect())
            io.WriteString(out, "\n")
        }        
    }
}
```
在这里我们使用的环境`env`在调用`Eval`函数之前，如果不这样做，那么`REPL`中绑定一个变量将不会有有任何作用。只要在下一行代码中执行，相关联的代码将不会存在新的环境中。

同样在我们的测试环境中也是如此，我们不想在每一个测试函数测试用例中保存状态。每个调用的`testEval`都会拥有一个新的环境，这边避免一些全局状态影响导致的一些Bug。在这里每个调用`Eval`就会得到一个崭新的环境：
```go
//evalutor/evaluator_test.go
func testEval(input string) object.Object {
    l := lexer.New(input)
    p := parser.New(l)
    program := p.ParseProgram()
    env := object.NewEnviroment()
    return Eval(program, env)
}
```
更新`Eval`函数调用后，我们开始让其通过，这样做并不困难。在`*ast.LetStatement`中的分支中，我们用了的变量的名称和值，我们将其保存到当前环境中去：
```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.LetStatement:
        val := Eval(node.Value, env)
        if isError(val){
            return val
        }
        env.Set(node.Name.Value, val)
}
```
现在我们已经在环境变量中增加值在执行`let`语句的时候，但是我们还需要获取其绑定的值当我们在执行变量语句的时候，这些很容易做：
```go
// evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.Identifier:
        return evalIdentifer(node, env)    
}
func EvalIdentifer(
    node *ast.Idenfier,
    env *object.Environment,
)object.Object {
    val, ok := env.Get(node.Value)
    if !ok {
        return newError("idenfier not found: " + node.Value)
    }
    return val
}
```
`evalIdentifier`在下一节将会被拓展，但是目前可以简单检查值是否绑定到相应的名称中。如果已经绑定成功，返回该值，否则生成一个错误。

看上去如此：
```
$ go test ./evaluator
ok monkey/evaluator 0.007s
```
是的，这就是我们刚开始所说的，现在我们已经在编程领域的稳稳站住了。
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let a = 5;
>> let b = a > 3;
>> let c = a * 90;
>> if (b) { 10 } else { 1 };
10
>> let d = if ( c> a) {99}else {100}；
>> d
99
>> d * c * a
245025
```

<h2 id="ch04-function-and-function-call">4.10 函数和函数调用</h2>
这一节是我们一直工作努力的方向， 我们将在我们的解释器中增加对函数和函数调用的支持。当我们完成这一小节，我们就能在`REPL`中做如下的操作：

```
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
如果上述的代码还不能吸引你，那么请看看下面，传递函数，高阶函数和闭包同样奏效：
```
>> let callTwoTimes = fn(x, func){ func(func(x))}
>> callTwoTimes(3, addThree);
9
>> let newAdder = fn(x) { fn(n) { x + n}};
>> let addTwo = newAddr(2);
>> addTwo(2);
4
```

是的，接下来我们将要把上述的所有功能全部实现。

从目前来讲我们已经完成的工作来看，我们还需要完成其他两件事情：一是在对象系统中对应我们的函数表达；另外一件是在`Eval`函数中增加函数调用支持。

别担心，这些都很简单。上一节我们所做的工作即将发挥作用了，我们可以在此基础上重用和拓展先前的成果，你将会看出许多事情仅仅是适应特定的环境的。

我们将仅仅遵循每次只完成一步工作的原则，第一步就是关注如何实现内部函数表达。

我们必须要承认的是在Monkey语言中，函数和其他值一样：绑定到特定的变量名称上，在表达式中使用它们，将它们传递到其它函数中，从其它函数中返回等等如此。因此我们需要在对象系统中表达出来，这样做才能传递、赋值和返回。

那我们如何在内部表达呢？作为一个`Oject`?是的，在`ast.FunctionLiteral`中开始我们的工作:
```go
//ast/ast.go
type FunctionLiteral struct {
    Token Token.Token // The 'fn' token
    Parameters []*Identifer
    Body *BlockStatement
}
```
貌似在函数对象中我们不需要`Token`字段，但是`Parameter`和`Body`是需要的。 如果没有函数体的话，我们不能执行一个函数；如果没有形参的话，我们也不能执行函数体。但是除此之外我们还需要第三个字段在函数对象：
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
`object.Function`对象除了拥有`Parameters`和`Body`字段外，还有`Env`字段。它是一个指向`object.Environment`对象的指针。因为Monkey语言中函数包含其当前的环境变量。它支持闭包，也就是在它能包含函数定义时候的环境变量，在将来访问的时候也能获取得到。因此使用`Env`字段意义重大。

通过上述的定义，我们可以通过编写测试来判断我们的解释器是否知道如何去构建函数:
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
测试函数将表明我们的执行一个字面函数表达式能够正确返回`*object.Function`对象，并且函数参数和函数体都是正确的。接来下我们会测试是否或者正确环境。为了让测试通过，我们需要在`Eval`函数中增加一些`case`分支。
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
测试通过了，是不是很简单？我们仅仅是重用了`Parameter`和`Body`字段，并且注意到购将函数对象的时候，我们也使用当前环境。

使用相对低层次的测试用例，我们可以确保我们已经构建好内部函数的表达。我们可以将注意点转移到函数应用，也就是说拓展我们的解释器以便可以调用函数，对此而言，测试用例非常容易读懂：
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
在这里每一个测试都做了同样的事情：定义一个函数、使用参数调用它然后对其的生成值做一次判断。但是有一点不同的是，有的返回值都是隐式的，有的使用`return`语句，有的使用表达式作为参数，有的多个参数需要被执行参数之后才传递函数。

我们同样也需要测试`*ast.CallExpression`的两种形式：一种是函数是一个变量，通过变量来调用函数。另一种是字面函数调用。幸运的是两者并没有任何不同的影响。我们已经知道了如何执行标识符和函数字面值：
```go
// evaluator/evalutor.go
func Eval(node ast.Node, env *object.Environment) objct.Object {
//[...]
    case *ast.CallIfExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
}
```
是的，我们仅仅通过调用`Eval`获取我们要调用的函数，无论它是`*ast.Identifier`或者是`*ast.FunctionLiteral`。`Eval`都会返回一个`*object.Function`。

但是我们如何调用这个`*object.Function`呢？第一步是执行参数是表达的，原因非常简单：
```
let add = fn(x, y) { x + y };
add(2 + 2, 5 + 5)
```
在这里我们将`4`和`10`作为参数传递给`add`函数，而不是表达`2+2`和`5+5`。

执行参数就是执行一系列表达式，并且记录生成值。但是我们必须停止执行只要发生错误，代码如下：
```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.CallExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
        args := evalExpression(node.Arguments, env)
        if len(args) == 1 && isError(args[0]) {
            return args[0]
        }
// [...]
}
func evalExpresion(
    exps []ast.Expression,
    env *object.Environment,
)[]object.Object {
    var result []object.Object
    for _, e := range exps {
        evaluated := Eval(e, env)
        if isError(evaluated) {
            return []object.Object{evaluated}
        }
        result = append(result, evaluated)
    }
    return result
}
```
这里没有一点魔法，我们仅仅是迭代`ast.Expression`列表并且根据上下文环境进行执行。如果遇到错误，我们停止执行并且返回该错误。在这里我们从左到右执行参数计算。希望我们没有编写代码测试判断参数计算的顺序，即便做了我们也是非常安全的。

所以，现在我们既有了函数和一系列执行出来的参数，我们该如何调动函数呢？如何将函数和这些参数联系在一起呢？

明显的答案是我们需要执行函数体，函数体就是语句块。我们已经知道如何去执行语句块了，所以为什么不直接调用`Eval`函数，并且将函数体传递过去就行了呢？答案是：参数。函数体包含了函数参数的引用，如果仅仅执行函数体使用当前的环境将会导致引用一些未知的名称，这些将会导致错误。所以执行函数体使用当前环境将不会起作用的。

我们需要的做的就是在执行的时候修改环境，以便函数形参能够解析到正确的实参。而且我们也不能简单的将当前的实参插入到当前的环境中，这也不是我们希望的。我们希望如下表达：
```
let i = 5;
let printNum = fn(i){
    puts(i);
};
printNum(10);
puts(i)
```
`puts`函数将输出一行内容，在上述代码应该输出两行，分别包含`10`he `5`。如果我们在执行函数`printNum`的函数体的时候，最后一行将会输出`10`。

所以仅仅将函数的实参加入到当前环境中以便函数体调用并没有作用。 我们需要做的不是保存先前的环境，而是拓展环境。

拓展环境意味着我们创建一个新的`object.Environment`实例，并且指向它以便完成拓展。通过这样创建包含了一个崭新的空的环境并且包含当前已经存在的环境。

当新的环境调用`Get`方法的时候，如果自身环境不存在该变量对应的名称，则调用它包含了环境。如果包含的环境也没有改名称，则调用包含环境的包含环境直到没有其他包含环境。我们可以放心的说："错误，未知标识符：foobar"。
```go
// object/environment.go
package object
func NewEncloseEnvironment(outer *Environment) *Environment{
    env := NewEnvironment()
    env.outer = outer
    return env
}
func NewEnvironemt() *Environment{
    s := make(map[string]Object)
    return &Environemnt{store: s, outer:nil}
}
type Environment struct {
    store map[string]Object
    outer *Environment
}
func (e *Environment) Get(name string) (Object, bool){
    obj, ok := e.store[name]
    if !ok && e.outer != nil {
        obj, ok = e.outer.Get(name)
    }
    return obj, ok
}
func (e *Environment) Set(name string, val Object) Object {
    e.store[name] = val
    return val
}
```
`object.Environment`现在有新的字段`outer`，它包含了其他`object.Environment`对象的引用，也就是包含环境。`NewEnclosedEnvironment`函数很容易地创建一个包含环境。`Get`方法也做了相应的修改，它现在对给定的名称也会查询包含环境。

新的问题是我们该如何考虑变量的作用域。在这里有内部作用域和外部作用域。对于没有在内部作用域中发现的，将会去外部作用域中寻找。外部作用域包含内部作用域，而内部作用域拓展外部作用域。

使用我们更新后的`object.Environment`，函数就能正确的执行函数体。要记住，先前的问题是在函数的实参可能重写绑定的环境，但是现在通过创建新的环境，并且包含在当前的环境中避免这种问题出现。

更新后的`Eval`函数如下所示，它能完全正确处理函数调用：
```go
// evaluator/evaluator.go
func Eval(ndoe ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.CallExpression:
        function := Eval(node.Function, env)
        if isError(function){
            return function
        }
        args := evalExpression(node.Arguments, env)
        if len(args) == 1 && isError(args[0]) {
            return args[0]
        }
        return applyFunction(function, args)
}
func extendFunctionEnv(
    fn *object.Function,
    args []object.Object,
)*object.Environment {
    env := object.NewEnclosedEnvironment(fn.Env)
    for parmIdx, para m := range fn.Parameters {
        env.Set(param.Value, args[paramIdx])
    }
    return env
}

func upwrapReturnValue(obj object.Object) object.Object {
    if returnValue, ok := obj.(*object.ReturnValue); ok {
        return returnValue.Value
    }
    return obj
}
```
在新的`applyFunction`函数中，我们不仅仅检查我们是否有`*object.Function`而且还讲`fn`的参数转换为`*object.Function`的引用，以便能够获取函数的`.Env`和`.Body`两个字段（这些在`object.Object`中并不拥有）。

在`extendFunctionEnv`函数中，我们创建了新的`*object.Environment`并且它被函数的的环境变量包含着。在新的被包含的环境中，它绑定了参数调用的形参和实参。

新的环境才是函数体调用执行使用的环境，这个返回结果被拆封， 以免`*object.ReturnValue`被返回。这样做事必须的，否则`return`结果将会被上浮，导致整个执行被退出，但是我们仅仅需要在函数体在最后一句停止执行。这就是我们为什么需要将返回结果拆封，所以`evalBlockStatement`不会停止执行函数外面的语句。我同样也在`TestReturnStatements`测试函数中增加了些测试用例来确保它们能够工作。

记下来一部分就是我们最后剩下来的一部分工作：

```
$ go test ./evaluator
ok monkey/evaluator 0.007s
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let addTwo = fn(x) { x + 2; };
>> addTwo(2)
4
>> let multiply = fn(x, y) { x * y };
>> multiply(50/2, 1 * 2)
50
>> fn(x) { x == 10 }(5)
false
>> fn(x) { x == 10}(10)
true
```
是的，它成功了，现在我们终于可以定义和调用函数。在我们庆祝之前，还值得我们进一步查看函数和它所在的环境。我们还不清楚它能做些什么。

我想你一直在思考一个问题：我们为什么不适用函数的环境而不是当前的环境，简单来说如下所示：
```go
//evalutor/evalutor_test.go
func TestCloures(t *testing.T){
    intput := `
    let newAdder = fn(x) {
        fn(y) { x + y };
    };
    let addTwo = newAdder(2)
    addTwo(2);
    `
    testIntegerObject(t, testEval(input), 4)
}
```
测试时通过的：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let newAdder = fn(x) { fn(y) { x + y } };
>> let addTwo = newAdder(2);
>> addTwo(3);
5
>> let addThree = newAdder(3);
>> addThree(10);
13
```
Monkey拥有闭包的特性并且在我们的解释器中工作得很好，是不是很酷？但是闭包和原先的问题还是非常明了。闭包是函数在定义的时候与其所在环境关联起来。它携带自身的环境在函数调用的时候也能访问它们。

在上述例子中最重要的两行代码如下：
```
let newAdder = fn(x){ fn(y) { x + y}};
let addTwo = newAddr(2);
```
`newAdder`是一个高阶函数，它是一种返回一个函数或者接受一个函数作为参数的函数。在这个例子中`newAdder`返回其他的函数，但是它不是其他函数而是一个闭包。`addTwo`绑定这个闭包，并且它是`newAdder`和一个`2`单独的实参。

那什么导致`addTwo`变成一个闭包？答案就是它在调用时候绑定的内容。当`addTwo`被调用的时候，它不仅仅能够访问他的的实参`y`参数，而且能够访问它在创建的时候`newAdder(2)`的参数`2`。尽管它绑定的参数已经离开他它的作用域，而且不在当前的环境中：
```
>>let newAdder = fn(x){ fn(y) { x + y}};
>>let addTwo = newAdder(2);
>> x
ERROR: identifier not found: x
```
`x`在顶级的环境中，并没有绑定一个值，但是`addTwo`仍然访问它：
```
>>addTwo(3)
5
```
换句话说，`addTwo`闭包仍然可以访问它在定义的时候的环境，也就是当`newAdder`函数体最后一句执行的时候。最后一行是字面函数，当一个字面函数被创建的时候，我们购将一个`object.Function`，并且保存它一个指向当前环境指针的字段`.Env`。

当我们后来执行`addTwo`的函数体的时候，我们并不是在当前环境中执行，而是在函数的环境中执行。我们拓展函数环境并且将其传递给`Eval`函数而不是当前环境。为什么要这么做呢？因为我们可以仍然可以访问它们；为了可以使用闭包；为了我们可以做出炫酷的东西出来。

既然我们在讨论神奇的东西，同样值得我们注意的是：我们不仅仅可以从其他函数中返回参数，也可以在函数中接受其他函数作为参数。是的，函数是Monkey语言中的一等公民，我们可以像其他类型一样进行传递。
```
>> let add = fn(a, b) { a + b }
>> let sub = fn(a, b) { a - b }
>> let applyFunc = fn(a, b, func){ func (a, b) }
>> applyFunc(2, 2, add);
4
>> applyFunc(10, 2, sub);
8
```
在这里我们将`add`和`sub`函数作为参数传递`applyFunc`。它调用函数没有任何问题，`func`参数作为一个函数对象，它然后调用其他两个参数，所有东西在我们的解释器里仅仅有条的工作。

我知道你现在想的内容，在这已经写好了一个模板给你：
> 亲爱的朋友（姓名）：还记得我曾经说过我会成为一个厉害的人并且做一些伟大的事情，还记得我吗？如今我的Monkey解释器成功了并且它支持函数，高阶函数、闭包和整型数学运算。总而言之：我从来没这么快乐过！

我们做到了，我们创建了一个完整的Monkey解释器，它支持函数，函数调用，高阶函数和闭包。
<h2 id="ch04-trash-out">4.11 垃圾回收</h2>
在本书开始阶段，我保证我们会亲自构建一个函数式解释器而不会走任何捷径，从零开始而不使用任何第三方工具。我们的确做到了，但是还是内容需要澄清一下。

考虑运行一下Monkey代码片段会发生什么事情：
```
let counter = fn(x){
    if (x > 100){
        return true;
    } else {
        let foobar = 9999;
        counter(x+1)；
    }
};
counter(0)
```
显然，它会返回一个`true`在执行函数体101遍后，但是在最后一遍递归返回时候发生了许多事情：

第一件事执行`if-else`表达式条件：`x>100`，如果它的生成值不是真值，那么`else`分支将会被执行。在此之中的整型字面值`9999`将会绑定到标识符`foobar`，而它从未被引用过。而是`x+1`被执行了，结果导致调用`Eval`函数被调用，并且调用`counter`，如此往复直至`x>100`判断为`TRUE`。

问题的关键是每一次调用`counter`，将会有许多对象被分配，或者将`Eval`函数的内容放入我们的对象系统。 每一次执行`counter`的函数体会带来`object.Integer`被分配和实例化，例如这些未被使用的`9999`整型和`x+1`的结果。甚至字面值`100`和`1`在每次执行`counter`的函数体时候也会产生新的`object.Integer`对象。

如果我们修改我们的`Eval`函数并且记录跟踪每一个`&object.Integer{}`实例，我们将会看到运行这个短小的代码实例差不多400多次的`object.Integers`分配。

那么问题在哪呢？

我们的对象存储在内存中，使用的对象越多，需要的内存越多。尽管例子中的对象的数量相对于其他程序非常少，但是内存不是无限的。

每次调用`counter`函数，我们解释器需要的内存就会增加，直至将内存全部用尽然后操作系统杀死这个程序。但是我们在运行上面的代码片段并对内存监控，发现它并不是持续上涨并且也不是下降。既不是上升也不是下降，这是为什么？

问题的答案就是我不得不讨论的：我们重用了Go语言额垃圾回收机制并且作为我们Monkey语言的垃圾回收机制，这些我们不需要自己完成。

Go语言垃圾回收机制（GC）就是我们为什么不会用完内存的原因，它帮我们管理内存。尽管我们调用`counter`函数很多次，我们增加了很多未被使用的整型字面值和分配的对象，我们不会用光内存。因为GC记录每一个`object.Integer`是否仍然可达。当它注意到一个对象不可达后，它释放该内存使之可以再次使用。

上述的例子生成了很多整型对象，但是在调用`counter`后，它们都不再可达。字面值`1`和`100`同无意义的`9999`绑定的`foobar`，在`counter`返回后这些对象将不会可达。在上述例子中`1`和`100`很显然它不再可达，因为它们并没有绑定任何标志符。但是`9999`绑定的`foobar`不可达的原因是当函数返回时，它超出了作用域。为函数体执行而创建的环境也被销毁（这些都是GC完成的）。

这些不可达的对象是不可达的并且占据着内存，这也是GC收集他们并且释放它们使用的内存。

这些对我们来讲非常有用，帮助我们省下了很多工作。如果我们编写我们解释器使用C语言，在那里我们将不会拥有GC，我们需要自己为解释器的用户实现内存管理。

那么实现这个GC需要怎么做？简单来讲：记录每个对象分配和它的引用，留下足够的内存足够为将来对象的分配并且归还那些不在需要的对象的内存。最后最重要的一点是，如果没有这个机制，将会泄露内存并且最终用光内存。

有很多方法能够实现上述要求，包含了很多算法和实现。比如，有一种基础的“标记清除”算法。为了实现它需要考虑是GC是否为分代GC，是否为STW类型GC还是并行GC，如何组织内存如何处理内存碎片。尽管考虑上述所有需求，高效的GC算法事项仍然需要很多工作。

你或许会问自己：**在这我们是使用Go语言的GC，那我们能不能自己编写GC而不用GO语言的？**

不幸的是，不能这样做。我们需要禁用Go语言的GC并找到方法来接管它所有的工作。说起来容易做起来难。这是巨大的工作量，需要小心地考虑内存分配和释放问题。

这也是我决定在本书不增加 *让我们编写我们的GC而不是Go的GC*的小节而是重新使用Go语言的GC。垃圾回收是一个巨大的主题并且超出了本书讨论的返回。但是我希望本小节给你GC大致映像以及它解决了什么问题，或许你知道了如果你想将这个解释器转移到其他宿主语言上需要考虑什么。不管怎样，我们的解释器完成了，剩下的就是拓展，增加一些数据类型和方法使之能够更加好用。


<h1 id="ch05-extending-the-interpreter">5 拓展解释器</h1>
<h2 id="ch05-data-type-and-functions">5.1 数据类型和函数</h2>
尽管我们的解释器非常棒，拥有一些很炫酷的特性比如一等函数和闭包，但是Monkey语言的用户能够使用的数据类型只有整型和布尔型。如果习惯其他编程语言，这个将会非常不方便。本章中我们将会改变这种状况，我们会增加一些新的数据类型。

这样做的好处是能够帮助我们重新回顾我们的解释器，我么将会增减新的token，修改我们的词法分析器，拓展我们的语法解析器，最终帮我们我们的解释器和对象系统支持这些数据类型。

好消息是这些数据类型已经在Go语言中存在了，那就意味着我们只需要让让它们在Monkey语言中可以使用即可。我们需要从零开始完成，这不是非常困难。由于我们本书的名称不是“使用Go语实现常见的数据结构”

除此之外，我们会通过增加一些新的函数以便让我们的解释器更加强大。当然作为解释器的用户可以定义我们想要完成的函数，但是这些会限制我们能够做的工作。新的函数叫做内置函数，它们非常强大，由于它们能够方位Monkey语言的内部工作机制。

首先我们需要增加的数据类型是字符串类型，几乎每个编程语言都支持该类型，因此Monkey语言也应该拥有字符串。

<h2 id="ch05-strings">5.2 字符串</h2>
在Monkey语言中，字符串就是一系列字符。它们是一等类型，可以绑定标识符、作为函数参数调用或者函数的返回值，它们看上去和其他编程语言一样：用双引号包含起来的字符。

除了数据类型本身，本小节中，我们也将增加字符串连接操作，通过中缀表达式`+`完成。

最终我们的效果如下：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let firstName = "Thorsten"
>> let lastName = "Ball"
>> let fullName = fn(first, last) { first + " " + last};
>> fullName(firstName, lastName)
Thorsten Ball
```
**词法解析器支持字符串**
首先要做的是在词法解析器中支持字符串字面值。字符串的基本数据结构如下所示：

```
"<sequence of characters>"
```
是不是很简单，就是一系列由双引号包含起来的一些列字符。

我们想要词法解析器做的就是在我们处理每一个字面的字符串，所以`"Hello World"`就会变成单一的token.而不是三个`"`,`Hello World`和`"`token，使用单个Token可以在我们的语法解析器中处理起来变得容易。所以我们需要在词法解析器部分做大量的工作。

当然，使用多个token也是可以的，对有些情况和解析器非常有用，我们就可以在标识符周围使用`"`，但是我们这里，我们已经拥有了token `INT` 作为整型，它将自身的字符串字面值作为`Literal`字段。

现在开始我们对我们的`token`和词法分析器做一步处理，从最开始我们还没有动过它们，我确信它们它们工作得很好。

首先要做的的是将`STRING` token类型增加到 `token` 包中:

```go
//token/token.go
const (
// [...]
    STRING = "STRING"
// [...]
)
```
与此同时，我们为词法解析器增加一些测试用例，以便是不是正确的支持字符串类型。因此我们拓展我们`TestNextToken`测试函数的`input`内容：
```go
func TestNextToken2(t *testing.T) {
	input := `let five=5;
let ten =10;
let add = fn(x, y){
  x+y;
};
let result = add(five, ten);
!-/*5;
5<10>5;

if(5<10){
	return true;
}else{
	return false;
}
10 == 10;
10 != 9;
"foobar"
"foo bar"
`
	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
// [...]
		{token.STRING, "foobar"},
		{token.STRING, "foo bar"},
        {token.EOF, ""},
// [...]
	}
// [...]
}
```
现在`input`多了两行包含字符串字面值，我们想把他们变成token。`foobar`确保词法解析器能够工作；`foo bar`也同样能够工作，尽管中间包含了一个空白字符。

当然目前测试时失败的，因为我们还没在词法解析器中做任何东西：
```
$ go test ./lexer
--- FAIL: TestNextToken (0.00s)
lext_test.go:122: tests[172] - tokenType wrong. expected="STRING", got="ILLEGAL"
FAIL
FAIL monkey/lexer 0.006s
```

想要让测试通过比你想象中的还要简单，我们所需要做的就是在词法解析器中为`"`增加分支和一些简单的帮助方法
```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token {
// [...]
    switch l.ch {
// [...]
    case '"':
        tok.Type = token.STRING
        tok.Literal = l.readString()
// [...]
    }
// [...]
}
func (l *Lexer) readString() string {
    position := l.position + 1
    for {
        l.readChar()
        if l.ch == '"' {
            break
        }
    }
    return l.input[position:l.position]
}
```
这些所做的并没有什么神秘的：一个`case`分支和一个帮助函数，它不停的调用`readChar`函数直至遇到另外一个双引号。

如果你认为这样比较简单，可以去增加转义字符的支持比如`"hello \"world\""`，`"hello \n world"`和`"hello\t\t\tworld"`等等。
与此同时，我们的测试通过了
```
$go test ./lexer
ok monkey/lexer 0.006s
```
好的，我们的词法解析器可以处理字面字符串了，接下来要做的事如何在语法解析器中处理它们。

**解析字符串**

为了让我们的语法解析器能够将`token.STRING`转变为抽象语法树的节点，我们需要定义节点。幸运的是，定义这个节点非常简单，它看上去和`ast.IntegerLiteral`非常相似，除了`Value`字段不是`int64`类型而是`string`类型。
```go
//ast/ast.go
type StringLiteral struct {
    Token token.Token
    Value string
}
func (sl *StringLiteral) expressionNode() {}
func (sl *StringLiteral) TokenLiteral() string { return sl.Token.Literal}
func (sl *StringLiteral) String() string { return sl.Token.Literal }
```
当然，字面字符串是表达式而不是语句，它们执行得到字符串。

有了上面的定义，我们编写一些测试来确保我们的解析器知道能够处理`token.STRING` token 并且输出`*ast.StringLiteral`

```go
//parse/parser_test.go
func TestStringLiteralExpression(t *testing.T){
    input := `"hello world";`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)

	stmt := program.Statements[0].(*ast.ExpressionStatement)
	literal, ok := stmt.Expression.(*ast.StringLiteral)
	if !ok {
		t.Fatalf("exp not *ast.StringLiteral. got=%T", stmt.Expression)
	}
	if literal.Value != "hello world" {
		t.Errorf("literal.Value not %q, got=%q", "hello world", literal.Value)
	}
}
```
运行测试仍然是解析错误
```
$ go test ./parser
-- FAIL: TestStringLiteralExpression(0.00s)
    parser_test.go:888: parser has 1 errors
    parser_test.go:890: parser error: "no prefix parse function for STRING found"
FAIL
FAIL monkey/parser 0.007s
```
在前面我们已经看到很多次了并且我们也知道如何处理它们，我们所需要做的就是为在`prefixParsefn`注册`token.STRING`处理的函数。 这个解析函数返回一个`*ast.StringLiteral`。
```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
//[...]
    p.registerPrefix(token.STRING, p.parseStringLiteral)
//[...]
}
func (p *Parser) parseStringLiteral() ast.Expression{
    return &ast.StringLiteral{Token: p.curToken, Value: p.curToken.Literal}
}
```
三行新代码就能够让我们的测试通过了：
```
$go test ./parser
ok monkey/parser 0.007s
```
现在我们的解析器将字面字符串变成`token.STRING`然后在解析器中将其变成`*ast.StringLiteral`节点。现在我们已经准备在对象系统和执行器中做出一些变化。

**执行字符串**

在我们对象系统中表达字符串和整型一样简单，但是最大的原因是我们重用了Go语言的string类型。想想一下在客户端语言中增加一种宿主语言没有的数据结构，这将是需要大量的工作，比如C语言。但是现在我们只需要一个包含字符串类型新的对象。
```go
//object/object.go
const (
//[...]
    STRING_OBJ = "STRING"
)
type String struct {
    Value string
}
func (s *String) Type() ObjectType { return STRING_OBJ }
func (s *String) Inspect() string { return s.Value }
```
在我们的执行器中需要的做的就是将`*ast.StringLiteral`变成`object.String`对象， 接下来额测试表明我们的工作非常简单：
```go
//evalutor/evalutor_test.go
func TestStringLiteral(t *testing.T) {
	input := `"Hello World!"`
	evaluated := testEval(input)
	str, ok := evaluated.(*object.String)
	if !ok {
		t.Fatalf("object is not String. got=%T(%+v)",
			evaluated, evaluated)
	}
	if str.Value != "Hello World!" {
		t.Errorf("String has wrong value. got=%q", str.Value)
	}
}
```
`Eval`函数调用返回的不是`*object.String`而是一个`nil`。
```
$ go test ./evalutor
--- FAIL: TestStringLiteral (0.00s)
evalutor_test.go:317: object is not string. got=<nil>(nil)
FAIL
FAIL monkey/evalutor 0.007s
```
让测试通过只需要增加的代码比解析器中还要少，就两行：
```go
//evaluator/evaluator.go
func Eval(node ast.Node, env *object.Environment) object.Object {
//[...]
    case *ast.StringLiteral:
        return &object.String{Value:node.Value}
}
```
这样做可以让我们的测试通过，并且可以在REPL中使用字符串
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> "Hello World!"
Hello World
>> let hello = "Hello there, fellow Monkey users and fans"
>> hello
Hello there, fellow Monkey users and fans!
>> let giveMeHello = fn() {"Hello"}
>> giveMeHello()
Hello!
```
现在在解析器中我们完全支持字符串了，获取我应该这样
```
>> "This is amazing"
This is amazing
```

**字符串拼接**

拥有字符串类型相当不错，但是除了创建它们好像并不能做些其他的什么。我们现在就做出一些改变，我们将我们的解释器提供字符串拼接的功能，通过支持`+`操作符和字符串操作符即可。

我么也同样拓展我们的`TestErrorHanding`函数来确保我们只增加了`+`操作符。
```go
//evalutor/evalutor_test.go
func TestErrorHanding(t *testing.T){
    tests :=[]struct {
        input string
        expectedMssage string
    }{
//[...]
        {
            `"Hello" - "World"`,
            "unknow operator: STRING - STRING"

        },
//[...]        
    }
//[...]
}
```
这个测试是失败的，但是我们的拼接测试还是失败的。
```
$ go test ./evaluator
--- FAIL: TestStringConcatentation (0.00s)
evalutor_test.go:336: object is not String. got= *object.Error(&{Message:unknow operator: STRING + STRING})
FAIL
FAIL monkey/evalutor 0.007s
```
我们需要改变的地方是`evalInfixExpression`。在这里我们需要在已经存在的switch语句中增加分支，以便能够执行操作符两边都是字符串类型的操作。

```go
//evalutor/evalutor.go
func evalStringInfixExpression(
    operator string,
    left, right object.Object,
)object.Object {
    if operator != "+" {
        return newError("unknow operator: %s %s %s",left.Type(), operator, right.Type())
    }
    leftVal := left.(*object.String).Value
    rightVal := right.(*object.String).Value
    return &object.String{Value: leftVal + rightVal}
}
```
首先我们要做的事检查是否为正确的操作符，如果它支持`+`操作符，我们将字符串对象封装起来，并且拼接左右两个操作数变成新的字符串。

如果我们想要支持更多的字符串操作，我们可以在这边添加。比如我们想要增减支持字符串比较的操作符`==`和`!=`。指针的比较并不起作用，至少不是我们想要的的结果，字符串的比较我们需要比较它们的值而不是指针。

这样我们的测试通过了
```
$ go test ./evaluator
ok monkey/evalutor 0.007s
```
现在我们能够使用字面字符串了，我们可以传递它们，绑定它们，将它们从函数中返回，也可以拼接它们。
```go
>> let makeGreeter = fn(greeting) { fn (name) { greeting + "" + name + "!" }}
>> let hello = makeGreeter("Hello")
>> hello("Thorsten")
Hello Thorsten!
>> let heythere = makeGreete("Hey There")
>> heythere("Thorsten")
Hey there Thorsten!
```
现在我们可以说字符串在我们的解释器中工作非常棒，但我们仍然可以增加新的东西来帮助它们工作。
<h2 id="ch05-built-in-functions">5.3 内置函数</h2>
在本小节中，我们将会为我们的解释器增加内置函数，之所以称之为内置因为它们不是有用户定义的，它们不是Monkey的代码，他们内置在解释器中，就是语言的本身。

这些内置函数由我们定义，它们是`Monkey`语言和我们解释器实现中的桥梁，许多语言提供内置函数，这些函数给用户提供这些语言不具备的功能。

举个例子，一个返回当前时间的函数。为了获取当前时间需要访问内核或者其他计算机。向内核发起请求通常由系统调用完成，但是编程语言并没有提供这些功能。编程语言，或者是编译器应该替用户完成这些操作。

再次强调一下，内置函数是由我们定义的。解释器的用户可以调用它们，但是我们定义的。这些函数能够做什么事开放的，唯一严格要求的是它们接受零个或者多个`object.Object`参数，返回一个`object.Object`。
```go
type BuiltinFunction func(args ...Object) Object
```
这是Go可调用的类型定义，但是因为要让这些`BuilitinFunction`能够使用户可用，需要让他们适应对象系统。所以我们需要做一层封装：
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
正如你看到的，这里`object.Builtin`并没有什么东西，仅仅是做了一层封装，但是这些对我们来讲就可以开始工作了。

**len**

第一个要添加到内置函数就是`len`函数， 它返回一个字符串中字符的数量，作为用户在自己定义功能几乎不可能，这也是我们为什么要内置它的原因。我们想要让`len`这样工作。
```
>> len("Hello World!")
12
>> len("")
9
>> len("Hey Bob, how ya doin?")
21
```
我认为这个能够让`len`背后的逻辑非常清楚了，为了更加清晰，我们可以很容易的编写一些测试：
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
在这我们有一些测试用例，按照这样的顺序依次执行`len`函数：空的字符串、正常的字符串、一个带空格的字符串。其实这个是没有关系的，如果空格是在字符串内。最后两个测试用例是更加有趣：如果参数为整型或者错误数量的参数将会返回一个`*object.Error`.

如果我们运行测试，我们可以看到函数返回错误，但是这些错误并不是我们在测试用例想要的错误：
```
$ go test ./evalutor
--- FAIL: TestBuiltinFunction(0.00s)
evalutor_test.go:389: object is not Integer. got=*obejct.Error(&{Message:identider not found: len})
evalutor_test.go:389: object is not Integer. got=*obejct.Error(&{Message:identider not found: len})
evalutor_test.go:389: object is not Integer. got=*obejct.Error(&{Message:identider not found: len})
evalutor_test.go:389: object is not Integer. got=*obejct.Error(&{Message:identider not found: len})
evalutor_test.go:389: object is not Integer. got=*obejct.Error(&{Message:identider not found: len})
FAIL
FAIL monkey/evalutor 0.007s
```
`len`没有找到，原因是我们还没有定义它。

为了完成它，首先我们要做的是为内置函数提供找到它们的方法，一个选择是在顶级的`object.Environment`中添加它们，并且传递给`Eval`函数，但是我们额外环境保存内置函数:
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
同样我们需要编辑我们`evalIdentifier`函数，用来查找内置函数作为我们当前环境没有找到标识符的时候的返回值。
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
现在`len`标识符可以找得到`len`内置函数，但是调用仍然不起作用:
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>>len()
ERROR: not a funciton: BUILTIN
```
运行单元测试同样的错误，我们需要告诉我们`applyFuntion`关于`*object.Builtin`和`object.BuiltinFunction`相关信息。
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
除了调整代码行数，我们所增加的就是增加了`*object.Builtin`分支，在这里我们调用`object.BuiltinFunction`，这样做只需要将`args`切片作为参数，并且调用函数。

值得注意的是我们不必要调用`unwrapReturnValue`函数，因为在内置函数中我们不会返回`*object.ReturnValue`。

现在测试在调用`len`的时候，正确地返回了NULL对象。
```
$ go test ./evaluator
--- FAIL: TestBuiltinFuncitons (0.00s)
evalutor_test.go:389: object is not Integer. got=*object.NULL(&{})
evalutor_test.go:389: object is not Integer. got=*object.NULL(&{})
evalutor_test.go:389: object is not Integer. got=*object.NULL(&{})
evalutor_test.go:366: object is not Error. got=*object.NULL(&{})
evalutor_test.go:366: object is not Error. got=*object.NULL(&{})
FAIL
FAIL monkey/evalutor 0.007s
```
这表明虽然调用了`len`，但是它们仅仅返回NULL对象，但是修复它们很简单，只要写几行Go函数:
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
最重要的部分是调用Go的`len`函数，并且返回新分配的`object.Integer`，除此之外我们还做了错误检查，确保我们不能调用该函数使用错误的参数数目，或者支持的参数类型。我们的测试通过了：
```
$ go test ./evaluator
ok monkey/evalutor 0.007s
```
这也意味着我们可以在REPL中使用`len`方法了
```
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
完美，我们第一个内置函数可以工作了。
<h2 id="ch05-array">5.4 数组</h2>
在这一小节中我们将为我们的Monkey解释器增加数据类型是数据，在Monkey中，数据是可能不同类型元素组成的有序列表。数组中的每一个元素都可以被单独被访。数组可以由字面表达方式：一系列由冒号隔开的元素，并且被包含在方括号中。

初始化一个新的数组，将其绑定到一个标识符中，访问每一个元素的操作如下：
```
>> let myArray = ["Thorsten", "Ball", 28 , fn(x){x * x}];
>> myArray[0]
Thorsten
>> myArray[2]
28
>> myArray[3](2);
4
```
正如你看的那样，Monkey中的数据不在乎元素的类型，每一个类型都可以作为一个数组的内容。在这个例子中`myArray`包含了两个字符串、一个整型和一个函数。

可以通过索引访问每一个元素，就像接下来三行代码，这是通过新的操作符，叫做索引操作符:`array[index]`。

在本小节中，我们将`len`内置函数同样支持数组操作，也增加一些内置函数可以操作数组。
```
>> let myArray = ["one", "two", "three"];
>> len(myArray)
3
>> first(myArray)
one
>> rest(myArray)
[two, three]
>> push(myArray, "four")
[one, two, three, four]
```
为我们Monkey语言实现的基础部分是go语言的`[]object.Object`，这也就意味着我们不需要自己实现数据结构，只需要重用go的切片。

听起来不错？是的，我们首先要做的就是让我们的词法分析器识别新的token。

**词法分析器支持数据**

为了正确解析数组字符串和和索引操作，我们词法解析器需要能够确定更多的token。我们需要构造的解析器是`[,]`和`,`，词法解析器先前完成的部分已经完成`,`，所以我们只需要支持`[`和`]`。

首先要做的是在`token`包中定义新的`token`类型。
```go
// token/token.go
const (
// [...]
    LBRACKET = "["
    RBRACKET = "]"
// [...]
)
```
第二步需要做的是拓展我们的测试部分，以便适应词法解析器。 这一点非常简单，之前已经重复很多次了。
```go
func TestNextToken2(t *testing.T) {
	input := `let five=5;
let ten =10;
let add = fn(x, y){
  x+y;
};
let result = add(five, ten);
!-/*5;
5<10>5;
if(5<10){
	return true;
}else{
	return false;
}
10 == 10;
10 != 9;
"foobar"
"foo bar"
[1,2];
`
	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
//[...]
		{token.LBRACKET, "["},
		{token.INT, "1"},
		{token.COMMA, ","},
		{token.INT, "2"},
		{token.RBRACKET, "]"},
		{token.EOF, ""},
    }
// [...]
}
```
`input`部分拓展为包含了新的`token`(在本例中是[1,2])，新的测试用语句已经被添加，以确保词法解析器中的`NextToken`方法能够正确返回`token.LBRACKET`和`token.RBRACKET`。

使测试通过也非常简单，只要增加在`NextToken`方法中增加四行代码：
```go
//lext/lext.go
func (l *Lexer) NextToken() token.Token {
// [...]
    case '[':
        tok = newToken(token.LBRACKET, l.ch)
    case ']':
        tok = newToken(token.RBRACKET, l.ch)
// [...]
}
```
是的，所有的测试已经通过
```
$ go test ./lexer
ok monkey/lexer 0.006s
```
所以在我们的语法解析中，我们可以使用`token.LBRACKET`和`tokne.RBRACKET`来解析数组。

**解析数组**
在我们前面所说的，Monkey中的数组就是有逗号隔开的列表，并且它们被包含在一个方括号中。
```
[1, 2, 3 + 3, fn(x), add(2, 2)]
```
数组中的每一个元素可以是任何类型的表达式，整型、函数、中缀或者前缀表达式。

如果听起来非常复杂，不用担心。在函数调用参数解析那一部分我们已经知道如何解析由逗号隔开的列表表达式。而且我们也知道如何去解析一些被匹配的token包含的表达式，换句话说也就是我们已经知道如何去处理它们了。

首先要做的事在抽象语法树中定义节点用来表达数组，既然我们已经知道其中的最基础的内容，代码就很容易解释了：
```go
// ast/ast.go
type ArrayLiteral struct {
    Token token.Token // the '[' token
    Elements []Expression
}
func (al *ArrayLiteral) expressionNode() {}
func (al *ArrayLiteral) TokenLiteral() string { return al.Token.Literal }
func (al *ArrayLiteral) String() string {
    var out bytes.Buffer
    elements := []string{}
    for _, el := range al.Elements {
        elements := append(elements, el.String())
    }
    out.WriteString("[")
    out.WriteString(string.Join(elements, ", "))
    out.WriteString("]")
    return out.String()
}
```
接下来的测试函数用来确保能够正确解析数组，并且返回`*ast.ArrayLiteral`(我也增加了测试函数用来检测空的数组来确保我们不会陷入讨厌的边缘用例)
```go
// parser/parser_test.go
func TestParsingArrayLiteral(t *testing.T) {
	input := `[1, 2*2, 3+3]`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
	array, ok := stmt.Expression.(*ast.ArrayLiteral)
	if !ok {
		t.Fatalf("exp not ast.ArrayLiteral. got=%T", stmt.Expression)
	}
	if len(array.Elements) != 3 {
		t.Fatalf("len(array.Elements) not 3. got=%d", len(array.Elements))
	}
	testIntegerLiteral(t, array.Elements[0], 1)
	testInfixExpression(t, array.Elements[1], 2, "*", 2)
	testInfixExpression(t, array.Elements[2], 3, "+", 3)
}
```
为了确保解析正确工作，测试输入部分包含了两种不同的中缀表达式，尽管整型和布尔型足够了。除此之外，测试部分也是非常枯燥，但是确保解析器能够返回`*ast.ArrayLiteral`对象并且包含正确数量的元素。

为了让测试通过，我们需要注册新的前缀表达式，因为左方括号是一个前缀表达式。
```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
// [...]
    p.registerPrefix(token.LBRACKET, p.parseArrayLiteral)
// [...]
}
func (p *Parser) parserArrayLiteral() ast.Expression{
    array := &ast.ArrayLiteral{Token: p.curToken}
    array.Elements = p.parseExpressionList(token.RBRACKET)
    return array
}
```
先前我们已经添加了`prefixParserFns`，所以这里并没有什么值得兴奋的。但是这里值得注意的是我们增加了新的方法`parseExpressionList`。该方法修改了先前的`parserCallArgument`方法并且变得更加通用，先前是用来解析`parserCallExpresion`中的用逗号隔开的参数列表。

```go 
//parser/parser.go
func (p *Parser) parseExpressionList(end token.TokenType) []ast.Expression {
	list := make([]ast.Expression, 0)
	if p.peekTokenIs(end) {
		p.nextToken()
		return list
	}
	p.nextToken()
	list = append(list, p.parseExpression(LOWEST))
	for p.peekTokenIs(token.COMMA) {
		p.nextToken()
		p.nextToken()
		list = append(list, p.parseExpression(LOWEST))
	}
	if !p.expectPeek(end) {
		return nil
	}
	return list
}
```
再一次，我们在`parseCallArugment`方法中见过这个方法，唯一的变化是这个版本接受一个终结的参数用来告诉方法哪一个是列表的结束。这个新版本的`paseCallExpression`方法现在看上去是这样的：
```go
//parser/parser.go
func (p *Parser) parserCallExpresison(function ast.Expression)ast.Expression{
    exp := &ast.CallExpression{Token:p.curToken, Function:function}
    exp.Argument = p.parserExpresson(token.RPAREN)
    return exp
}
```
唯一改变的地方时我们调用`paserExpressionList`使用`token.RPAREN`（这个用来确定参数的结束)。我们通过修改几行代码，能够重新使用这个方法。棒极了，最重要的是我们的测试通过了：
```
$ go test  ./parser
ok monkey/parser 0.007s
```

**解析索引表达式**

为了完整的支持数组，我们不仅仅要求解析出数组，还要解析出索引表达式。或许索引操作符听上去没有什么映象，但是我打赌你肯定知道这是什么回事。索引操作符如下所示：
```
myArray[0];
myArray[1];
myArray[2];
```
这些事最简单的形式，但是还有其他很多例子，如下所示：
```
[1, 2, 3, 4][2];
let myArray = [1, 2, 3, 4];
myArray[2];
myArray[2+1];
returnsArray()[1];
```
是的，这些都是正确的，最基本的表达式是`<expression>[<expression>]`。这个看上去足够简单，我们可定定义新的抽象语法树的节点：`ast.IndexExpression`，结构如下：
```go
//ast/ast.go
type IndexExpression struct {
	Token token.Token
	Left  Expression
	Index Expression
}

func (ie *IndexExpression) expressionNode()      {}
func (ie *IndexExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *IndexExpression) String() string {
	var out bytes.Buffer
	out.WriteString("(")
	out.WriteString(ie.Left.String())
	out.WriteString("[")
	out.WriteString(ie.Index.String())
	out.WriteString("])")
	return out.String()
}
```
值得注意的是`left`和`index`字段都是表达式。 `left`使我们可以访问的对象，它可以是任何类型：标识符、字面数组或者函数调用。对于`index`也是同样如此，任何表达式都可以。语法上来讲并没有任何改变，在语义上都会生成一个整数。

事实上，`left`和`index`都是表达式更加有助于处理，因为我们可以用`parseExpression`方法来处理他们。但是第一件事就是我们的测试用例能够知道并且返回一个`*ast.IndexExpression`:
```go
//parser/parser_test.go
func TestParsingIndexExpression(t *testing.T) {
	input := "myArray[1+1]"
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	stmt, _ := program.Statements[0].(*ast.ExpressionStatement)
	indexExp, ok := stmt.Expression.(*ast.IndexExpression)
	if !ok {
		t.Fatalf("exp not *ast.IndexExpression. got=%T", stmt.Expression)
	}
	if !testIdentifier(t, indexExp.Left, "myArray") {
		return
	}
	if !testInfixExpression(t, indexExp.Index, 1, "+", 1) {
		return
	}
}
```
现在测试只能确保解析器只能返回正确的抽象语法树当只有单一的表达式语句包单个索引表达式。但是同样重要的是解析器能够正确处理索引表达式的优先级。目前为止，索引操作符拥有最高的操作符，可以拓展我们的`TestOperatorPrecedenceParsing`测试函数。
```go
//parser/parser_test.go
func TestOperatorPrecedenceParsing(t *testing.T) {
	tests := []struct {
		input    string
		expected string
	}{
//[...]
		{"a*[1,2,3,4][b*c]*d", "((a * ([1, 2, 3, 4][(b * c)])) * d)"},
		{"add(a*b[2], b[1], 2 * [1,2][1])", "add((a * (b[2])), (b[1]), (2 * ([1, 2][1])))"},
	}
//[...]
}
```
在`String()`方法中增加的`(`和`)`输出`*ast.IndexExpression`有助于我们编写测试用例，因为它能使得索引操作符号清晰的表达。在测试用例中我们期望在中缀表达式中索引操作符的优先级比调用表达式甚至`*`操作符更高。

测试是失败的因为我们的解析器不知道任何关于索引表达式的内容：
```
$ go test ./parser
--- FAIL: TestOperatorPrecedenceParsing (0.00s)
---
FAIL
FAIL monkey/parser 0.007s
```
尽管测试中输的的缺失`prefixParseFn`方法就是我们想要的，但是索引表达式并不需要在一个操作符两边有各有一个操作数。为了解析很方便地解析它们，我们这样做大有裨益，就跟先前函数调用表达式解析一样。具体来讲就是将`[`在`myArray[0]`作为一个中缀表达式，`myArray`作为左边操作数而`0`作为右边操作数。这样做的话，我们解析的实现就非常棒了:
```go
//parser/parser.go
func New(l *lexer.Lexer) *Parser{
// [...]
    p.registerInfix(token.LBRACKET, p.parserIndexExpression)
// [...]
}
func (p *Parser) parserIndexExpression(left ast.Expression)ast.Expression{
    exp := &ast.IndexExpression{Token: p.curToken, Left:left}
    p.nextToken()
    p.Index = p.parserExpression(LOWEST)
    if !p.expectPeek(Token.RBRACKET){
        return nil
    }
    return exp
}
```
很棒，但是并没有修复我们没有通过的测试
```
$ go test ./parser
--- FAIL: TestOperatorPrecedenceParsing (0.00s)
---
FAIL
FAIL monkey/parser 0.008s
```
原因是在Pratt解析中的优先级问题，我们还没有定义索引操作符的优先级。
```go
// parser/parser.go
const (
    _ int = iota
// [...]
    INDEX
)
var precedneces = map[token.TokenType]int{
// [...]
    token.LBRACKET: INDEX
}
```
最重要的一点是将`INDEX`放在最后一行，借助`iota`, 它将`INDEX`最高的优先级。新增加的词条给`token.LBRACKET`最高级别的优先级。当然，一切都通过。
```
$ go test ./parser
ok monkey/parser 0.007s
```
词法分析完成、语法分析完成，接下来是执行器。

**执行数组**

执行数组并不难，将数组映射到Go语言的切片类型非常简单，我们也就不需要自己实现新的数据结构。我们仅仅需要定义新的`object.Array`类型，它将是数组类型生成的结果。定义`object.Array`非常简单，因为在Monkey语言中数组非常简单：它们仅仅就是一系列对象。
```go
// object/object.go
const (
// [...]
    ARRAY_OBJ = "ARRAY"
)
type Array struct {
    Elements []object
}
func (ao *Array) Type() ObjectType { return ARRAY_OBJ }
func (ao *Array) Inspect() string {
    var out bytes.Buffer
    elements := []string{}
    for _, e := range ao.Elements {
        elements = append(elements, e.Inspect())
    }
    out.WriteString("[")
    out.WriteString(string.Join(elements, ", "))
    out.WriteString("]")
    return out.String()
}
```
我认为你会同意我说的最复杂的部分就是`Inspect`方法的定义，而且那是相当容易理解的。

下面就是数组的执行测试：
```go
//evalutor/evalutor_test.go
func TestArrayLiterals(t *testing.T) {
	input := `[1, 2*2, 3+3]`
	evaluated := testEval(input)
	result, ok := evaluated.(*object.Array)
	if !ok {
		t.Fatalf("object is not Array, got=%T(%v)",
			evaluated, evaluated)
	}
	if len(result.Elements) != 3 {
		t.Fatalf("array has wrong num of elements. got=%d",
			len(result.Elements))
	}
	testDecimalObject(t, result.Elements[0], 1)
	testDecimalObject(t, result.Elements[1], 4)
	testDecimalObject(t, result.Elements[2], 6)
}
```
我们可以重用已经存在的代码让测试通过，就像我们在解析器中所做的。我们可以重用先前函数调用部分代码，也就是增加分支用来执行`*ast.ArrayLiteral`来生成数组对象:
```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.ArrayLiteral:
        elements := evalExpression(node.Elements, env)
        if len(elements) == 1 && isError(elements[0]) {
            return elements[0]
        }
        return &object.Array{Elements: elements}
}
```
测试通过，并且我们可以在REPL中使用字面数组并且生成数组：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> [1, 2, 3, 4]
[1, 2, 3, 4]
>> let double = fn(x) { x  * 2};
>> [1, double(2), 3 * 3, 4 - 3]
[1, 2, 9, 1]
```
是不是很神奇，但是我们现在还不能通过索引表达式访问单个元素。

**执行索引表达式**
好消息是解析索引表达式比执行索引表达式难多了，我们已经完成解析部分。唯一问题是可能出现移位错误在访问数组中的元素。因此我们在测试集中增加一些测试：
```go
// evalutor/evalutor_test.go
func TestArrayIndexExpression(t *testing.T) {
	tests := []struct {
		input    string
		expected interface{}
	}{
		{
			"[1,2,3][0]",
			1,
		},
		{
			"[1,2,3][1]",
			2,
		},
		{
			"[1,2,3][2]",
			3,
		},
		{
			"let i =0; [1][i]",
			1,
		},
		{
			"let myArray=[1,2,3];myArray[2];",
			3,
		},
		{
			"let myArray=[1,2,3];myArray[0]+myArray[1]+myArray[2]",
			6,
		},
		{
			"let myArray=[1,2,3];let i = myArray[0]; myArray[i]",
			2,
		},
		{
			"[1,2,3][3]",
			nil,
		},
		{
			"[1,2,3][-1]",
			nil,
		},
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
我们承认这些测试有点多，好多测试我们在其他地方已经测试完毕。但是这些测试非常容易编写，也非常可读。

特别注意一下这些测试所期望的行为， 它包含了我们先前没有考虑过得内容，当索引的的大小超出数组的边界，我们将会返回NULL。其他一些语言会生成一个错误或者返回一个null，我们这里选择null。

正如我们期望的，测试失败了，失败信息如下：
```
$ go test ./evalutor
--- FAIL: TestArrayIndexExpression(0.00s)
panic: runtime error: invalid memory address or nil pointer deference
FAIL monkey/evalutor 0.011s
```
那么那么我们该如何修复和执行索引表达式呢？正如我们先前看到，索引操作符的左边操作数可以是任何表达式，而索引表达式也也可以是任何表达式。这就意味着我们在执行中缀表达式之前要先执行两边表达的表达式。否则我们直接访问标识符或者函数的标识符将无法工作。

在这我们增加一个`*ast.IndexExpression`分支，来调用`Eval`函数。
```go
// evalutor/evalutor/go
func Eval(node ast.Node, env *object.Environment) object.Object {
// [...]
    case *ast.IndexExpression:
        left := Eval(node.Left, env)
        if isEror(left){
            return left
        }
        index := Eval(node.Index, env)
        is isError(index){
            return index
        }
        return evalIndexExpression(left, index)
// [...]
}
```
接下来是`evalIndexExpression`函数
```go
// evalutor/evalutor.go
func evalIndexExpression(left, index object.Object) object.Object{
    switch {
    case left.Type() == object.Array_OBJ && index.Type() == object.INTEGER_OBJ:
        return evalArrayIndexExpression(left, index)
    default:
        return newError("index operator not support: %s", left.Type())
    }
}
```
使用`if`条件语句足够了，但是在下一章节我们将会增加其他`case`分支。除了错误处理，最重要是`evalArrayIndexExpression`函数中。
```go
func evalArrayIndexExpression(array, index object.Object) object.Object {
    arrayObject := array.(*object.Array)
    idx := index.(*object.Integer).Value
    max := int64(len(arrayObject.Element) - 1)
    if id<0 || idx > max {
        return NULL
    }
    return arrayObject.Element[idx]
}
```
在这我们从数组中按照给定的索引获取相应的元素， 除此之外一些类型判断和类型转换也非常直接，它检查给定的索引值是否在给定范围内，如果超出范围返回NULL，正如我们在测试中给出医德一样， 现在测试通过了。
```
$ go test ./evalutor
ok monkey/evalutor 0.007s
```
现在放轻松，深呼吸一下看看接下来会发生什么：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let a = [1, 2 * 2, 10 - 5, 8 / 2]
>> a[0]
1
>> a[1]
4
>> a[5 - 3]
5
>> a[99]
null
```
从数组中获取元素成功了，非常棒。我想字再一次说实现这些功能非常神奇。


**为数组增加内置函数**

现在我们可以通过数组字面值构建数组，也可以通过中缀表达式访问数组中的元素。以上两个功能可以使得数组非常有用，但是为了让数组更加有用，我们需要增加一些内置函数来使他们变得更加方便，这一小节我们就做这事。

在这一小节中我们将不会增加任何测试代码， 应为这些特定的测试将会占据空间并且不会增加任何新的东西。我们的为内置函数测试的框架已经包含在`TestBuiltFunctions`，仅仅需要在已有的框架中增加，你会发现他们很好地适应。

我们的目标是增加新的内置函数，但是我么首先要做的事不知增加他们，而是修改已有的代码。我们需要`len`函数支持数组操作，目前它只支持字符串类型。
```go
//evaluator/builtins.go
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
			case *object.Array:
				return &object.Integer{Value: int64(len(arg.Elements))}
			default:
				return newError("argument to `len` not supported, got=%s",
					args[0].Type())
			}
		},
    },
//[...]
}
```
唯一的变化就是增加的`*object.Array`分支，接下来我们需要增加其他新的函数。

首先要增加的内置函数是`first`,它返回数组的第一个元素。同样你也可以调用`myArray[0]`达到同样的效果，但是`frist`相当简洁，接下来就是其实现：
```go
//evalutor/builtins.go
var builtins = map[string]*object.Builtin{
//[...]
    "first":&object.Builtin{
        Fn: func(args ...object.Object) object.Object{
            if len(args) != 1{
                return newError("wrong number of arguments. got=%d, want=1", len(args))
            }
            if args[0].Type() != object.ARRAY_OBJ {
                return newERror("argument to 'frist' must be ARRAY, got %s", args[0].Type())
            }
            arr := args[0].(*object.Array)
            if len(arr.Elements) > 0 {
                return arr.Elements[0]
            }
            return NULL
        },
    },
}
```
棒极了，那么接下来是什么呢？是的，我们接下来将要增加的的是`last`。

`last`函数的目的是返回数组的最后一个元素，使用索引操作符是`myArray[len(myArray) - 1]`，正如其表达的一样，实现`last`函数并不困难，就跟`first`类似。
```go
//evalutro/builtin.go
var builtins = map[string]*object.Builtin{
"last": {
		Fn: func(args ...object.Object) object.Object {
			if len(args) != 1 {
				return newError("wrong number of arguments. got=%d, want=1",
					len(args))
			}
			if args[0].Type() != object.ARRAY_OBJ {
				return newError("argument to `last` must be ARRAY, got %s",
					args[0].Type())
			}
			arr := args[0].(*object.Array)
			length := len(arr.Elements)
			if length > 0 {
				return arr.Elements[length-1]
			}
			return NULL
		},
    },
}
```
接下来我们将要增加类似`Scheme`语言中的`cdr`函数，在其他语言中同样也叫做`tail`函数，在这里我们将叫做`rest`。该函数返回新的数组包含其传入的参数的数组，除了第一个元素。使用起来如下所示：
```
>> let a = [1, 2, 3, 4];
>> rest(a)
[2, 3, 4]
>> rest(rest(a))
[3, 4]
>> rest(rest(rest(a)))
[4]
>> rest(rest(rest(rest(a))))
[]
>> rest(rest(rest(rest(rest(a)))))
null
```
实现非常简单，但是要记住我们返回的是新分配的数组，我们并没有修改传入的数组。
```go
//evalutor/builitins.go
var builtins = map[string]*object.Builtin{
// [...]
    "rest": {
		Fn: func(args ...object.Object) object.Object {
			if len(args) != 1 {
				return newError("wrong number of arguments. got=%d, want=1",
					len(args))
			}
			if args[0].Type() != object.ARRAY_OBJ {
				return newError("argument to `rest` must be ARRAY, got=%s",
					args[0].Type())
			}
			arr := args[0].(*object.Array)
			length := len(arr.Elements)
			if length > 0 {
				newElements := make([]object.Object, length-1, length-1)
				copy(newElements, arr.Elements[1:length])
				return &object.Array{Elements: newElements}
			}
			return NULL

		},
	},
}
```
接下来我们为数组增加的内置函数是`push`, 它在一个数组的后面增加一个新的元素，这里同样我们并不修改传入的数组，而是重新分配同样元素的数组，并且增加新增加的元素。在Monkey中，数组是不可修改的，下面是其实现操作：
```
>> let a = [1, 2, 3, 4]
>> let b = push(a, 5)
>> a
[1, 2, 3, 4]
>> b
[1, 2, 3, 4, 5]
```
接下来就是实现：
```go
//evalutor/builtins.go
var builtins = map[string]*object.Builtin{
//[...]
"push": {
		Fn: func(args ...object.Object) object.Object {
			if len(args) != 2 {
				return newError("wrong number of arguments. got=%d, want=1",
					len(args))
			}
			if args[0].Type() != object.ARRAY_OBJ {
				return newError("argument to `push` must be ARRAY, got=%s",
					args[0].Type())
			}
			arr := args[0].(*object.Array)
			length := len(arr.Elements)
			newElements := make([]object.Object, length+1, length+1)
			copy(newElements, arr.Elements)
			newElements[length] = args[1]
			return &object.Array{Elements: newElements}
		},
    },
}
```

**测试驱动数组**

现在我们有数组，索引表达式和一些和数组一同工作的内置函数。是适合做一些工作，看看我我么你现在能做什么。

通过`first`,`rest`和`push`函数我们可以构建`mao`函数
```
let map =fn(arr, f){
    let iter = fn(arr, accumlated){
        if (len(arr) == 0) {
            return accumleated
        }else{
            iter(rest(arr), push(accumate, f(first(arr))));
        }
    }
    iter(arr, []);
}
```
通过`map`我们可以这样操作
```
>> let a= [1, 2, 3, 4];
>> let double = fn(x){ x + 2 };
>> map(a, double);
[2, 4, 6, 8]
```
是不是很神奇，基于内置函数我们同样可以定义`reduce`函数
```
let d=reduce = fn(arr, initial, f){
    let iter = fn(arra, result){
        if (len(arr) == 0){
            result
        }else{
            iter(rest(arr), f(result, first(arr)));
        }
    };
    iter(arr, initial)
}
```
基于`reduce`， 同样我们也可以定义`sum`函数
```
let sum = fn(arr){
    reduce(arr, 0, fn(initial, el){ initial + el });
}
```
同样也是工作了：
```
>> sum([1, 2, 3, 4, 5])
15
```
现在看看我们的解释器，可以有`map`函数，`reduce`同样如此，我们完成了巨大的工作。

这些不是全部，我们还有许多可以做，我希望你能够探索更多可能的数据类型和内置函数。在这之前，你可以在你朋友家人之前骄傲一下。
<h2 id="ch05-hashes">5.5 哈希表</h2>
接下来我们要增加的数据类型是哈希表，在Monkey语言中的哈希表在其他语言中也称之为映射表或者字典，它将键映射到值。

为了在Monkey语言中使用哈希表，规定哈希字面值为：一个冒号隔开的键值对列表，它们被包含在一对花括号中，每一个键值对都用逗号与其他键值对分开，下面就是哈希表的样子
```
>> let myHash = {"name":"Jimmy","age":72, "band":"Led Zeppelin"};
>> myHash["name"]
Jimmy
>> myHash["age"]
72
>> myHash["band"]
Led Zeppelin
```
在这个例子中`myHash`包含了三个键值对，它们的值都是都是字符串，正如你看到的那样，我们在哈希表中同样适用中缀表达式，就跟数组中使用的一样。在这个例子中的索引使用字符串，这个在数组中不起作用，并且它也不是哈希表键的唯一数据类型。

```
>> let myHash = {true: "yes, a boolean", 99:"correct, an integer"};
>> myHash[true]
yes, a boolean
>> myHash[99]
correct, an in
```
那同样有效，事实上，除了字符串、整型和布尔型，我们可以使用任何表达式作为索引操作符的索引。
```
>> myHash[5>1]
yes, a boolean
>> myHas[100 - 1]
correct, an integer
```
只要这些表达式的执行结果是要是字符串、整型或者布尔型中的任何一个都可以作为键，在这`5>1`执行结果为`true`，`100-1`执行结果为`99`，两者都是`myHash`中有效的值。

不奇怪的是我们在Monkey的哈希表中，我们使用Go语言中的`map`作为数据类型，但是我们想要字符串、整型和布尔类型这些不变类型作为键，我们需要在此基础上做些工作。我们在此基础上拓展我们对象系统，但是首先要做的事将哈希字面值转换为`token`

**词法解析哈希**

我们该如何将哈希字面值转换为`token`呢？哪些`token`需要我们确认出并且作为我们的词法解析器的输出以便给接下来的语法解析器，下面是哈希字面值：
```javascript
{"name":"Jimmy","age":72, "band":"Led Zeppelin"}
```
除了字符串之外，有四个字符集在这里显得特别重要：`{`, `}`,`,`,和`:`。我们已经知道如何解析前面三种，我么的词法解析器将他们解析为`token.LBRACE`，`token.RBRACE`和`token.COMMA`，也就是说我们所做的就是将`:`转换成相应的`token`。

首先要做的是在`token`包中增加必须的类型
```go
//token/token.go
const (
//[...]
    COLON = ":"
//[...]
)
```
接下来就是为`Lexer`的`NextToken`方法增加测试使得`token.COLON`如期望所示。
```go
//lexer/lexer_test.go
func TestNextToken2(t *testing.T) {
	input := `let five=5;
let ten =10;
let add = fn(x, y){
  x+y;
};
let result = add(five, ten);
!-/*5;
5<10>5;

if(5<10){
	return true;
}else{
	return false;
}
10 == 10;
10 != 9;
"foobar"
"foo bar"
[1,2];
{"foo":"bar"}
`
	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
//[...]
		{token.LBRACE, "{"},
		{token.STRING, "foo"},
		{token.COLON, ":"},
		{token.STRING, "bar"},
		{token.RBRACE, "}"},
		{token.EOF, ""},
	}
// [...]
}
```
我们可以仅仅通过增加`:`来进行测试，但是使用的哈希表来进提供更多的上下文来进行测试。
```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token{
//[...]
    case ':':
        tok = newTokne(token.COLON, l.ch)
//[...]
}
```
只需要两行代码，词法解析器就可以将`token.COLON`解析出来
```
$ got test ./lexer
ok monkey/lexer 0.006s
```
我们现在反悔了`token.LBRACE`, `token.RBRACE`和`token.COMMA`以及新的`tokne.COLON`，这是我们用来解析哈希表。

**解析哈希表**

在我们开始编写解析器甚至开写测试，让我们先看看哈希表额基础语法结构
```
{<expression>:<expression>, <expression>:<expression>}
```
这是由逗号隔开的键值对，每一个键值对包含两个表达式，一个生成哈希表的键，另一个生成值。键和值用冒号隔开，所有的键值对有一对花括号包含起来。

转换到抽象语法树的节点，我们该如何记录每一个键值对呢？我们该如何去做？是的，用`map`字典来表示，但是字典中的键和值如何表示呢？

我们先前说过，只有字符串、整型和布尔型才能作为哈希表的键，但是我们不能再解析器中强制这样。我们需要在执行阶段验证哈希表的键的类型，并且生成相应的错误。

由于很多不同的表达式都能生成字符串，整型或者布尔型，不仅仅是其中的字面类型，强制数据类型在解析阶段会阻止我们写出下面的代码：
```
let key = "name";
let ahs = {key: "monkey"};
```
在这里`key`的执行生成`"name"`，这是合法的哈希键值，尽管它是一个标识符。为了允许这么做，我们需要允许在哈希字面中任何表达式作为键，任何表达式作为值。至少在解析阶段，我们的`ast.HashLiteral`的定义如下：
```go
//ast/ast.go
type HashLiteral struct {
    Token token.Token
    Pair map[Expression]Expression
}
func (hl *HashLiteral) expressionNode() {}
func (hl *HashLiteral) TokenLiteral() string {
    return h1.Token.Literal
}
func (hl *HashLiteral) String() string {
    var out bytes.Buffer
    pairs := []string{}
    for key, value := range hl.Pairs{
        pairs = append(pair, key.String()+":"+value.String()_
    }
    out.WriteString("{")
    out.WriteString(strings.Join(pairs, ", "))
    out.WriteString("}")
    return out.String()
}
```
现在我们已经清楚了哈希表的结构，并且已经定义了`ast.HashLiteral`结构，现在我们可以在解析器中编写测试。
```go
//parser/parser_test.go
func TestParsingHashLiteral(t *testing.T) {
	input := `{"one":1, "two":2, "three":3}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	stmt := program.Statements[0].(*ast.ExpressionStatement)
	hash, ok := stmt.Expression.(*ast.HashLiteral)
	if !ok {
		t.Fatalf("exp is not ast.HashLiteral. got=%T", stmt.Expression)
	}
	if len(hash.Pairs) != 3 {
		t.Errorf("hash.Pairs has wrong legnth. got=%d", len(hash.Pairs))
	}
	expected := map[string]int64{
		"one":   1,
		"two":   2,
		"three": 3,
	}
	for key, value := range hash.Pairs {
		literal, ok := key.(*ast.StringLiteral)
		if !ok {
			t.Errorf("key is not ast.StringLiteral. got=%T", key)
		}
		expectedValue := expected[literal.String()]
		testIntegerLiteral(t, value, expectedValue)
	}
}
```
当然，我们可以确保解析空的哈希表示正确，因为边界条件在代码中已经考虑到了。
```go
//parser/parser_test.go
func TestParsingEmptyHashLiteral(t *testing.T) {
	input := "{}"
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	stmt := program.Statements[0].(*ast.ExpressionStatement)
	hash, ok := stmt.Expression.(*ast.HashLiteral)
	if !ok {
		t.Fatalf("exp isn not ast.HashLiteral. got=%T",
			stmt.Expression)
	}
	if len(hash.Pairs) != 0 {
		t.Errorf("hash.Pairs has wrong length. got=%d", len(hash.Pairs))
	}
}
```
我同样也增加一些新的测试，这些和`TestHashLiteralStringKeys`方法比较类似，但是使用使用整型和布尔型作为哈希的键。用来确保它们都能各自转换为`*ast.IntegerLiteral`和`ast.Boolean`。在这里第五个测试函数确保哈希表中的值可以是任何表达式，甚至是操作符表达式。
```go
func TestParsingHashLiteralWithExpression(t *testing.T) {
	input := `{"one":0+1, "two":10 - 8, "three": 15/5}`
	l := lexer.New(input)
	p := New(l)
	program := p.ParseProgram()
	checkParserErrors(t, p)
	stmt := program.Statements[0].(*ast.ExpressionStatement)
	hash, ok := stmt.Expression.(*ast.HashLiteral)
	if !ok {
		t.Fatalf("exp is not ast.HashLiteral. got=%T",
			stmt.Expression)
	}
	if len(hash.Pairs) != 3 {
		t.Errorf("hash.Pairs has wrong length. got=%d",
			len(hash.Pairs))
	}
	tests := map[string]func(ast.Expression){
		"one": func(e ast.Expression) {
			testInfixExpression(t, e, 0, "+", 1)
		},
		"two": func(e ast.Expression) {
			testInfixExpression(t, e, 10, "-", 8)
		},
		"three": func(e ast.Expression) {
			testInfixExpression(t, e, 15, "/", 5)
		},
	}
	for key, value := range hash.Pairs {
		literal, ok := key.(*ast.StringLiteral)
		if !ok {
			t.Errorf("key is not ast.StringLiteral. got=%T", key)
			continue
		}
		testFunc, ok := tests[literal.String()]
		if !ok {
			t.Errorf("No test function for key %q found", literal.String())
			continue
		}
		testFunc(value)
	}
}
```
所以这些测试函数工作如何呢？说实话，我们将会得到测试不通过和解析的错误。
```
$ go test ./parser
--- FAIL: TestParsingHashLiteralWithExpression (0.00s)
--- FAIL: TestParsingHashLiteralsStringKeys (0.00s)
--- FAIL: TestParsingHashLiteralsBooleanKeys (0.00s)
--- FAIL: TestParsingHashLiteralsIntegerKeys (0.00s)
--- FAIL: TestParsingHashLiteralsWithExpressions (0.00s)
FAIL
FAIL monkey/parser 0.008s
```
说起来可能不信，好消息只需要一个函数能够让所有测试通过，确切来说就是`prefixParseFn`函数。由于`token.LBRACE`在哈希表中属于前缀表达式，就跟解析数组中的`token.LBRACKET`一样， 我们可以定义一个`parseHashLiteral`方法来作为`prefixParseFn`。
```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser {
//[...]
    p.registerPrefix(token.LBRACE, p.ParseHashLiteral)
//[...]
}
func (p *Parser) parseHashLiteral() ast.Expression {
	hash := &ast.HashLiteral{Token: p.curToken}
	hash.Pairs = make(map[ast.Expression]ast.Expression)
	for !p.peekTokenIs(token.RBRACE) {
		p.nextToken()
		key := p.parseExpression(LOWEST)
		if !p.expectPeek(token.COLON) {
			return nil
		}
		p.nextToken()
		value := p.parseExpression(LOWEST)
		hash.Pairs[key] = value
		if !p.peekTokenIs(token.RBRACE) && !p.expectPeek(token.COMMA) {
			return nil
		}
	}
	if !p.expectPeek(token.RBRACE) {
		return nil
	}
	return hash
}
```
它看上去很恐怖，但是在`parseHashLiteral`中没有任何新的东西，它仅仅是不停循环整个`key-value`表达式对，并且检查`token.RBRACE`调用`parseExpression`两次。这就完成了`hash.Pairs`中最重要的部分，工作很棒！
```
$ go test ./parser
ok monkey/parser 0.006s
```
所有的解析测试通过了，通过增加测试的数量，我们可以有理由确信我们的解析器知道如何正确处理哈希表。这也是意味着我们将要为我们的解释器中的哈希表最有趣的部分：表示哈希表中的对象系统和执行哈希表。

**哈希对象**

除了拓展词法解析器和语法解析器，增加数据类型也就意味着我们它们要在对象系统的表达出来。我们已经成功的为整数、字符串和数组完成了这些工作，但是完成这些数据类型只要定义一个结构，其拥有一个正确的`.Value`字段。但是哈希表需要做一些额外的工作，接下来我解释为什么。

首先假设我们定义新的`object.Hash`类型如下：
```go
type Hash strcut {
    Pairs map[Object]Object
}
```
显然我们只需要借助`go`语言中的`map`数据结构完成哈希表示，但是这样做的化，我们该如何实现哈希表中的键值对呢？更重要的的是我们该如何获取相应的值？

考虑如下的Monkey代码片段
```
let hash = {"name": "Moneky"};
hash["name"]
```
让我们使用上述定义的`object.Hash`来执行上述两行代码，在执行第一行代码的时候，我们将每个键值对放到`map[Object]Obejct`字典中去，导致`.Pairs`拥有如下的映射关系：一个`*obejct.String`的`.Value`字段`"name"`映射到`*object.String`的`.Value`字段的`"Monkey"`中。

到目前为止没有任何问题，问题出现了，当我们执行第二行代码的时候，我们使用索引表达式来访问`Moneky`字符串。

在第二行代码中`"name"`字符串的索引表达式执行结果为重新分配的`*obejct.String`对象。尽管这个新的`*obejct.String`的`.Value`字段中包含了`"name"`，它和键值对中的`*object.String`相同，但是我们不能使用新的来恢复`Monkey`字符串。

原因是它们是指向不同内存位置的指针，但是事实它们内存指向的位置内容并没有关系。比较这些指针将会告诉我们他们是不同的，也就一位置我们新创建的`*object.String`作为`key`并不能得到`Monkey`。这也是Go语言中指正比较不同。

接下来的例子就是展示我们使用`object.Hash`所面临的的难题。
```go
name1 := &object.String{Value:"name"}
monkey := &object.String{Value:"Monkey"}

pairs := map[object.Object]object.Object{}
pairs[name1] = monkey
fmt.Printf("pairs[name1]=%+v\n", pairs[name1])
//=> pairs[name1]=&{Value:Monkey}
name2 := &obejct.String{Value:"name"}
fmt.Printf("pairs[name2]=%+v\n", pairs[name2])
//=> paris[name2]=<nil>
fmt.Printf("(name1==name2)=%\t\n", name1==name2
//=> (name1 == name2)=false
```
其中的一个解决方案就是迭代`.Pairs`中键，检查`.object.String`的`.Value`字段，判断他们是否相等。但是这种方法将会使得查找特定`key`的时间复杂度冲`O(1)`变成`O(n)`，这个完全违背了我们当初选择哈希的意图。

另外一种选择方案是定义`Pairs`作为`map[string]Object`， 然后是用`.Value`字段`.object.String`作为其中作为键，虽然有效，但是对于整数和布尔型数据不起作用。

我们现在要做的为哈希表创建一个键，使得他们很容易的区分开来。我们需要为`object.String`生成一个哈希键，并且对于相同的`object.String`比较是相同的。 对于`object.Integer`和`object.Boolean`也是同样如此。但是对于`*object.String`和`*object.Integer`或者`*object.Boolean`必须不可能相等，这些哈希的键值必须不能相等。

接下来是一组测试函数，它们表达了我们想要在我们的对象系统的期望的行为：
```go
// object/object_test.go
package object
import "tesing"
func TestStringHashKey(t *testing.T) {
	hello1 := &String{Value: "Hello World"}
	hello2 := &String{Value: "Hello World"}
	diff1 := &String{Value: "My name is johnny"}
	diff2 := &String{Value: "My name is johnny"}
	if hello1.HashKey() != hello2.HashKey() {
		t.Errorf("string with same content have different keys")
	}
	if diff1.HashKey() != diff2.HashKey() {
		t.Errorf("string with same content have differnet keys")
	}
	if hello1.HashKey() == diff1.HashKey() {
		t.Errorf("string with different have same hash key")
	}
}
```
从`HashKey()`方法就是的确我们想要得到的，不仅仅是`*object.String`而且是`*object.Boolean`和`*object.Integer`，这也是同样的测试函数都存在的原因。

接下来我们将要为以下三个类型都实现`HashKey()`方法。
```go
//obejct/object.go
import (
// [...]
    "hash/fnv"
)
type HashKey() struct {
    Type ObjectType
    Value uint64
}
func (b *Boolean) HashKey() HashKey {
    var value uint64
    if b.Value {
        value = 1
    }else{
        value = 0
    }
    return HashKey{Type: b.Type(), Value: uint64(i.Value)}
}
func (i *Integer) HashKey() HashKey {
    return HashKey{Type: i.Type(), Value: uint64(i.Value)}
}
func (s *String) HashKey() HashKey {
    h := fnv.New64a()
    h.Write([]byte(s.Value))
    return HashKey{Type: s.Type(), Value: h.Sum64()}
}
```
每一个`HashKey()`方法返回一个`HashKey`，就跟你看到定义的一样，`HashKey`并没有什么神奇的地方，`Type`字段包含了一个`Object.Type`，它能有效的区分不同类型的键值。 `Value`字段包含了真正哈希值，就是一个整数。由于他们就是整数，所以我们对于不同的`HashKey`使用`==`操作符来比较不同。这也意味着我们使用`HashKey`就跟Go语言中`map`一样。

现在先前的问题可以使用`HashKey`来解决。
```go
name1 := &object.String{Value: "name"} monkey := &object.String{Value: "Monkey"}
pairs := map[object.HashKey]object.Object{} pairs[name1.HashKey()] = monkey
fmt.Printf("pairs[name1.HashKey()]=%+v\n", pairs[name1.HashKey()]) 
// => pairs[name1.HashKey()]=&{Value:Monkey}
name2 := &object.String{Value: "name"} fmt.Printf("pairs[name2.HashKey()]=%+v\n", pairs[name2.HashKey()])
// => pairs[name2.HashKey()]=&{Value:Monkey}
fmt.Printf("(name1 == name2)=%t\n", name1 == name2) // => (name1 == name2)=false
fmt.Printf("(name1.HashKey() == name2.HashKey())=%t\n", name1.HashKey() == name2.HashKey())
// => (name1.HashKey() == name2.HashKey())=true
```
就这事我们想要的，`HashKey`中定义的`HashKey()`方法可以很好地解决使用原生的`Hash`定义，这也让测试通过。
```
$ go test ./object
ok monkey/object 0.008s
```
现在我们可以顶一个`object.Hash`，并且使用新的`HashKey`类型：
```go
//object/object.go
const (
// [...]
    HASH_OBJ = "HASH"
)
type HashPair struct {
    Key Object
    Value Object
}
type Hash struct {
    Pairs map[HashKey]HashPair
}
func (h *Hash) Type() ObjectType { return HASH_OBJ }
```
增加了`Hash`和`HashPair`，`HashPair`是`Hash.Pairs`的值类型。你可能会奇怪为什么我们不直接定义`map[HashKey]Object`来代替`Pairs`

原因就在于`Inspect()`方法，当我们在REPL中输出Monkey中的哈希表时候，我们想要打印出每一个值关联的的键，仅仅打印出`HashKey`是没有用的。所以我们记录每一个`HashKey`关联的对象，通过使用`HashPairs`是有用的。在这里我们保存了原先的键对象和值对象。通过这种方法，我们调用`Inspect()`方法，可以生成`*object.Hash`对象，下面就是`Inspect()`方法实现：
```go
//object/object.go
func (h *Hash) Inspect() string {
	var out bytes.Buffer
	pairs := make([]string, 0)
	for _, pair := range h.Pairs {
		pairs = append(pairs, fmt.Sprintf("%s: %s",
			pair.Key.Inspect(), pair.Value.Inspect()))
	}
	out.WriteString("{")
	out.WriteString(strings.Join(pairs, ", "))
	out.WriteString("}")
	return out.String()
}
```
`Inspect()`方法不是唯一我们记录对象的`HashKey`原因，它也是非常重要我们想要实现`range`函数，为每个哈希对象迭代它们的每一个键值。或者我们想要增加`firstPair`函数，它返回第一个键值对，就跟数组一样。总之，记录键值对是非常有用的，尽管现在我们好像在`Inspect`中使用了它们。

现在好像所有都完成了，但是我们还有一件小事需要去完成：
```go
//object/object.go
type Hashable interface {
    HashKey() HashKey
}
```
在这里我们使用`Hashable`接口，来确保给定的对象使用可以使用哈希键，目前来讲只有`*object.String`，`*object.Boolean`和`*object.Integer`对象实现了这个接口。

现在还有一件事需要做，我们可以优化`HashKey()`方法，可以借助他们的缓存值，这个听上去是个不错的性能优化的优化方法。

**执行哈希表**

我们将要开始执行哈希表而且我很诚实地告诉你，最难的部分已经过去了。让我们享受这个过程，编写一些测试：
```go
func TestHashLiterals(t *testing.T) {
	input := `let two="two";
	{
		"one":10-9,
		two:1+1,
		"thr" + "ee" : 6/2,
		4 : 4,
		true:5,
		false:6
	}`
	evaluated := testEval(input)
	result, ok := evaluated.(*object.Hash)
	if !ok {
		t.Fatalf("Eval did't return Hash. got=%T(%+v)",
			evaluated, evaluated)
	}
	expected := map[object.HashKey]int64{
		(&object.String{Value: "one"}).HashKey():   1,
		(&object.String{Value: "two"}).HashKey():   2,
		(&object.String{Value: "three"}).HashKey(): 3,
		(&object.Integer{Value: 4}).HashKey():      4,
		TRUE.HashKey():                             5,
		FALSE.HashKey():                            6,
	}
	if len(result.Pairs) != len(expected) {
		t.Fatalf("Hash has wrong num of pairs. got=%d", len(result.Pairs))
	}
	for expectedKey, expectedValue := range expected {
		pair, ok := result.Pairs[expectedKey]
		if !ok {
			t.Errorf("no pair for given key in Pairs")
		}
		testDecimalObject(t, pair.Value, expectedValue)
	}

}
```
这个测试函数展示了当我们遇到`*ast.HashLiteral`的时候，如何执行`Eval`:一个`*object.Hash`对象包含许多`HashPair`，将他们映射到对应的`HashKey`中。

它同样展示其他的要求：字符串、标识符、中缀表达式、布尔型和整型都是合法的键。任何表达式都是合法的，只要它们能够生成的对象实现了`Hashable`接口，那么就可以作为哈希键。

它们的值可以有任何表达式推导出来。在这里我们通过`10-9`来生成`1`，`6 / 2`变成`3`来进行测试。

正如你期望的，测试是失败的：
```
$go test ./evalutor
--- FAIL: TestHashLiterals (0.00s)
evalutor_test.go: 522: Eval didn't return Hash. got=<nil>(<nil>)
FAIL
FAIL monkey/evalutor 0.008s
```
尽管我们知道如何让测试通过，我们需要拓展我们的`Eval`函数通过增加`*ast.HashLiterals`分支。
```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Obejct {
// [...]
    case *ast.HashLiteral:
        return evalHashLiteral(node, env)
// [...]
}
```
这个`evalHashLiteral`函数可能看上去非常吓人，但是相信我，非常简单：
```go
//evaluator/evaluator.go
func evalHashLiteral(node *ast.HashLiteral, env *object.Environment) object.Object {
	pairs := make(map[object.HashKey]object.HashPair)
	for keyNode, valueNode := range node.Pairs {
		key := Eval(keyNode, env)
		if isError(key) {
			return key
		}
		hashKey, ok := key.(object.Hashable)
		if !ok {
			return newError("unusable as hash key: %s", key.Type())
		}
		value := Eval(valueNode, env)
		if isError(value) {
			return value
		}
		hashed := hashKey.HashKey()
		pairs[hashed] = object.HashPair{Key: key, Value: value}

	}
	return &object.Hash{Pairs: pairs}
}
```
当迭代`node.Pairs`时候，`keyNode`首先被执行，除了检查它调用`Eval`函数的时候是否出现错误，我们同样做了一次类型判断：它是否实现了`object.Hashable`接口，否则它不能作为哈希键。这也是我们为什么增加`Hashbale`定义。

当我们再次调用`Eval`函数，去执行`valueNode`。如果没有生成错误，那么我们将新生成的键值对存放到我们的`pairs`字典中。我们通过调用`HashKey`对象调用`HashKey()`方法生成一个键。然后初始化一个新的`HashPair`，并且指向`key`和`value`然后将他们添加到`pairs`中。

现在我们的测试通过:
```
$ go test ./evaluator
ok monkey/evaluator 0.007s
```
这也意味着我们可以在我们的REPL中使用哈希表：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> {"name": "Monkey", "age":0, "type":"Language", "status":"awesome"}
{age:0, type: language, status:awesome, name:Monkey}
```
是不是很神奇？但是我们现在还不能获取哈希表中的元素，这个好像让他们看上去没啥用处。
```
>> let bob = {"name": "Bob", "age", 99}
>> bob["name"]
ERROR: index operator not supported: HASH
```
接下来我们将会修复这个问题

**执行哈希表的索引表达式**

还记得我们在`evalIndexExpression`中增加的`switch`语句吗？还记得我曾经告诉你我们将会增加其他`case`分支，现在我们就来做这件事。

开始我们增加一些测试函数来确保我们可以通过索引表达式访问哈希表中的值：
```go
//evaluator/evaluator.go
func TestHashIndexExpression(t *testing.T) {
	tests := []struct {
		input    string
		expected interface{}
	}{
		{
			`{"foo":5}["foo"]`,
			5,
		},
		{
			`{"foo":5}["bar"]`,
			nil,
		},
		{
			`let key = "foo"; {"foo":5}[key]`,
			5,
		},
		{
			`{}["foo"]`,
			nil,
		},
		{
			`{5:5}[5]`,
			5,
		},
		{
			`{true:5}[true]`,
			5,
		},
		{
			`{false:5}[false]`,
			5,
		},
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
就跟`TestArrayIndexExpression`中一样，我们确保我们使用索引表达式能够得到正确的值，只不过这次是哈希表。不同的是在测试用例中我们使用字符串、整型和布尔型哈希键来获取哈希表中的值。所以从本质上来讲就是这些测试就是用来判断这个`HashKey`方法在不同数据类型情况下是否正确调用。

同样这边确保如果使用一个没有实现`object.Hashable`对象作为哈希表中的键将会生成一个错误，我们可以在`TestErrorHandling`测试函数中增加相应的测试用例。
```go
//evaluator/evaluator.go
func TestErrorHandling(t *testing.T){
    tests := []struct{
        input string
        expectedMessage string
    }{
//[...]
        {
            `{"name", "monkey"}[fn(x){x}]`,
            "unusable as hash key: FUNCTION",
        },
    }
// [...]
}
```
运行这些测试将会导致测试失败：
```
$ go test ./evalutor
--- FAIL: TestErrorHandling (0.00s)
evaluator_test.go: no error obejct returned. got=*obejct.Null(&n{})
--- FAIL: TestHashIndexExpression(0.00s)
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
evaltor_test.go:611: object is not Integer. got=*object.Null(&{})
FAIL
FAIL monkey/evalutor 0.007s
```
这也意味着我们要在`evalIndexExpression`函数中的`switch`语句中增加新的`case`分支：
```go
//evalutor/evaluator.go
func evalIndexExpression(left, index object.Object) object.Object {
	switch {
	case left.Type() == object.ARRAY_OBJ && index.Type() == object.INTEGER_OBJ:
		return evalArrayIndexExpression(left, index)
	case left.Type() == object.HASH_OBJ:
		return evalHashIndexExpression(left, index)
	default:
		return newError("index operator not support:%s", left.Type())
	}
}
```
现在新的分支调用新的函数：`evalHashInexExpression`。现在我们已经知道`evalHashInexExpression`该如何去工作，因为我们成功测试`object.Hashable`如何使用。
```go
func evalHashIndexExpression(hash, index object.Object) object.Object {
	hashObject := hash.(*object.Hash)
	key, ok := index.(object.Hashable)
	if !ok {
		return newError("unusable as hash key: %s", index.Type())
	}
	pair, ok := hashObject.Pairs[key.HashKey()]
	if !ok {
		return NULL
	}
	return pair.Value
}
```
增加`evalHashIndexExpression`可以使得我们的测试通过：
```
$ go test ./evaluator
ok monkey/evalutor 0.007s
```
现在我们成功地从哈希表中获取相应的值，不信就可以往下看：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let people = [{"name": "Alice", "age": 24}, {"name":"Anna", "age":28}];
>> people[0]["name"]
Alice
>> people[1]["age"]
28
>> people[1]["age] + people[0]["age"]
52
>> let getName = fn(person){ person["name"];};
>> getName(people[0]);
Alice
>> getName(people[1]);
Anna
```
<h2 id="ch05-the-grand-finale">5.6 完结</h2>
我们Monkey解析器完全是函数式的，它支持数学表达式，变量绑定，函数和函数的调用，条件表达式、返回语句甚至更高级的概念，比如高阶函数和闭包。还有不同的数据类型：整型、布尔型、字符串、数组和哈希表。我们可以为此感到骄傲。

但是我们的解释器还不能通过基础的编程语言测试：输出。是的，我们的Monkey解释器不能喝外部交流。甚至其他语言如`Bash`和`Brainfuck`都能够做到这点，显然我们也要完成这个。接下来我们要增加最后一个内置函数`puts`

`puts`输出给定的参数到`STDOUT`。它调用穿入参数对象`Inspect()`方法并且将他们返回值输出。`Inspect()`方法是`Obejct`接口对象，所以每一个实体都支持这个方法，使用`puts`函数看上去是这样的：
```
>> puts("Hello!")
Hello!
>> puts(1234)
1234
>> puts(fn(x){x(x)})
fn(x) {
    (x * x)
}
```
`puts`函数是可变参数的函数，它接受无限制数量的参数，并且将每一个参数按行输出
```
>> puts("hello", "world", "how", "are", "you")
hello
world
how
are
you
```
`puts`仅仅是输出并不生成任何值，我们确保他返回一个`NULL`:
```
>> let putReturnValue = puts("foobar")
foobar
>>putReturnValue
null
```
这也意味着我们的REPL将除了输出`puts`的内容外，还会输出一个`null`对象，它看上去像这样：
```
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
通过上述，我们完成了。
在第三章中，Monkey编程语言诞生了，现在它开始呼吸。通过本章，它似乎开始说话了。现在Monkey就是一个真实的编程语言：
```
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> puts("Hello World!")
Hello world!
null
```
<h1 id="ch06-resources">6 资源</h1>

**书籍**

- Abelson, Harold and Sussman, Gerald Jay with Sussman, Julie. 1996. Structure and Interpretation of Computer Programs, Second Edition. MIT Press.
- Appel, Andrew W.. 2004. Modern Compiler Implementation in C. Cambridge University Press.
- Cooper, Keith D. and Torczon Linda. 2011. Engineering a Compiler, Second Edition. Morgan Kaufmann.
- Grune, Dick and Jacobs, Ceriel. 1990. Parsing Techniques. A Practical Guide.. Ellis Horwood Limited.
- Grune, Dick and van Reeuwijk, Kees and Bal Henri E. and Jacobs, Ceriel J.H. Jacobs and Langendoen, Koen. 2012. Modern Compiler Design, Second Edition. Springer
- Nisan, Noam and Schocken, Shimon. 2008. The Elements Of Computing Systems. MIT Press.

**论文**

- Ayock, John. 2003. A Brief History of Just-In-Time. In ACM Computing Surveys, Vol. 35, No. 2, June 2003
- Ertl, M. Anton and Gregg, David. 2003. The Structure and Performance of Efficient Interpreters. In Journal Of Instruction-Level Parallelism 5 (2003)
- Ghuloum, Abdulaziz. 2006. An Incremental Approach To Compiler Construction. In Proceedings of the 2006 Scheme and Functional Programming Workshop.
- Ierusalimschy, Robert and de Figueiredo, Luiz Henrique and Celes Waldemar. The Implementation of Lua 5.0. [https://www.lua.org/doc/jucs05.pdf](https://www.lua.org/doc/jucs05.pdf)
- Pratt, Vaughan R. 1973. Top Down Operator Precedence. Massachusetts Institute of Technology.
- Romer, Theodore H. and Lee, Dennis and Voelker, Geoffrey M. and Wolman, Alec and Wong, Wayne A. and Baer, Jean-Loup and Bershad, Brian N. and Levy, Henry M.. 1996. The Structure and Performance of Interpreters. In ASPLOS VII Proceedings of the seventh international conference on Architectural support for programming languages and operating systems.
- Dybvig, R. Kent. 2006. The Development of Chez Scheme. In ACM ICFP '06

**网络资源**
- Jack W. Crenshaw - Let's Build a Compiler! -
[http://compilers.iecc.com/crenshaw/tutorfinal.pdf](http://compilers.iecc.com/crenshaw/tutorfinal.pdf)
- Bob Nystrom - Pratt Parsers: Expression Parsing Made Easy -[http://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/](http://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/)
- Shriram Krishnamurthi and Joe Gibbs Politz - Programming Languages: Application and Interpretation - (http://papl.cs.brown.edu/2015/)[http://papl.cs.brown.edu/2015/]
- A Python Interpreter Written In Python - [http://aosabook.org/en/500L/a-python-interpreter- written-in-python.html](http://aosabook.org/en/500L/a-python-interpreter- written-in-python.html)
- Dr. Dobbs - Bob: A Tiny Object-Oriented Language - [http://www.drdobbs.com/open- source/bob-a-tiny-object-oriented-language/184409401](http://www.drdobbs.com/open- source/bob-a-tiny-object-oriented-language/184409401)
- Nick Desaulniers - Interpreter, Compiler, JIT - [https://nickdesaulniers.github.io/blog/2015/05/25/interpreter-compiler-jit/](https://nickdesaulniers.github.io/blog/2015/05/25/interpreter-compiler-jit/)
- Peter Norvig - (How to Write a (Lisp) Interpreter (in Python)) - [http://norvig.com/lispy.html](http://norvig.com/lispy.html ) 
- Fredrik Lundh - - Simple Town-Down Parsing In Python - [http://effbot.org/zone/simple-top-down-parsing.html](http://effbot.org/zone/simple-top-down-parsing.html)
- Mihai Bazon - How to implement a programming language in JavaScript - [http://lisperator.net/pltut/](http://lisperator.net/pltut/)
- Mary Rose Cook - Little Lisp interpreter - [https://www.recurse.com/blog/21-little-lisp-interpreter](https://www.recurse.com/blog/21-little-lisp-interpreter)
- Peter Michaux - Scheme From Scratch - [http://peter.michaux.ca/articles/scheme-from-scratch-introduction](http://peter.michaux.ca/articles/scheme-from-scratch-introduction)
- Make a Lisp - [https://github.com/kanaka/mal]( https://github.com/kanaka/mal)
- Matt Might - Compiling Scheme to C with closure conversion - [http://matt.might.net/articles/compiling-scheme-to-c/](http://matt.might.net/articles/compiling-scheme-to-c/)
- Rob Pike - Implementing a bignum calculator - [https://www.youtube.com/watch?v=PXoG0WX0r_E](https://www.youtube.com/watch?v=PXoG0WX0r_E)
- Rob Pike - Lexical Scanning in Go - [https://www.youtube.com/watch?v=HxaD_trXwRE](https://www.youtube.com/watch?v=HxaD_trXwRE)

**源码**

- The Wren Programming Language - [https://github.com/munificent/wren](https://github.com/munificent/wren)
- Otto - A JavaScript Interpreter In Go - [https://github.com/robertkrimen/otto](https://github.com/robertkrimen/otto)
- The Go Programming Language - [https://github.com/golang/go](https://github.com/golang/go)
- The Lua Programming Language (1.1, 3.1, 5.3.2) - [https://www.lua.org/versions.html](https://www.lua.org/versions.html)
- The Ruby Programming Language - [https://github.com/ruby/ruby](https://github.com/ruby/ruby)
- c4 - C in four functions - [https://github.com/rswier/c4](https://github.com/rswier/c4)
- tcc - Tiny C Compiler - [https://github.com/LuaDist/tcc](https://github.com/LuaDist/tcc)
- 8cc - A Small C Compiler - [https://github.com/rui314/8cc](https://github.com/rui314/8cc)
- Fedjmike/mini-c - [https://github.com/Fedjmike/mini-c](https://github.com/Fedjmike/mini-c)
- thejameskyle/the-super-tiny-compiler - [https://github.com/thejameskyle/the-super-tiny-compiler](https://github.com/thejameskyle/the-super-tiny-compiler)
- lisp.c - [https://gist.github.com/sanxiyn/523967](https://gist.github.com/sanxiyn/523967)


<h1 id="ch07-feedback">7 反馈</h1>

如果你发现了输入错误、代码中的一些错误、提供一些建议或者仅仅问一些问题，尽管给我发邮件：

me@thorstenball.com

(译者注：)
如果发现翻译错误或者更改，也同样给我([gaufung](https://github.com/gaufung))发邮件：

gaufung@outlook.com

或者登陆译者个人网站: [https://gaufung.com](https://gaufung.com)