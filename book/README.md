**用GO语言编写解析器**

# 翻译计划
由于本人在互联网公司工作，工作时间较长，翻译工作只能在有限的下班时间展开。现在邀请一起翻译，<del>目前所有章节翻译工作尚未有其他人认领<-del>，需要认领章节的，邮箱联系本人，邮件内容包含认领的小节和个人GitHub地址。

**翻译表**  

翻译人员 | 章节
---|---
[gaufung](https://github.com/gaufung) | 1.1 - 1.3
[Jehu Lu](https://github.com/lwhile)  | 2.1 - 2.5
[momaek](https://github.com/momaek)  | 3.1 - 3.9
  

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
- [4 计算](#ch04-evaluation)
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

但是我认为更重要的是本书提供的Go语言代码与那些更底层的代码非常类似，比如C，C++和Rust。或者这跟Go语言的本身相关，更注重简洁性,<del>its stripped-down charm and lack of programming language constructs that are absednt in other languages and hard to translate</del>。 或许这也是我为本书选择Go语言的原因。同样原因，这本书中将不会有任何元编程的相关技巧，虽然那样做何以走一些捷径，但是两周之后将没有人能够看懂。没有强大的面向对象设计和模式</del>that need pen, paper and the sentence "actually, it's pretty easy" to explain</del>。

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
<h2 id="ch02-defining-our-tokens">2.2 定义Token</h2>
<h2 id="ch02-the-lexer">2.3 词法分析器</h2>
<h2 id="ch02-extending-our-token-set-and-lexer">2.4 拓展Token集和词法分析器</h2>
<h2 id="ch02-start-of-a-repl">2.5 REPL编写</h2>

<h1 id="ch03-parsing">3 语法解析</h1>
<h2 id="ch03-parsers">3.1 语法解析器</h2>
<h2 id="ch03-why-not-a-parser-generator">3.2 为何不采用语法生成器</h2>
<h2 id="h04-writing-a-parser-for-the-monkey-programming-language">3.3 为Monkey编程语言编写语法解析器</h2>
<h2 id="ch03-parsing-let-statement">3.4 解析Let语言</h2>
<h2 id="ch03-parsing-retrun-statement">3.5 解析Return语句</h2>
<h2 id="ch03-parsing-expression)">3.6 解析表达式</h2>
<h2 id="ch03-how-pratt-parsing-works">3.7 Pratt解析法如何工作</h2>
<h2 id="ch03-extending-the-parser">3.8 拓展解析器</h2>
<h2 id="ch03-read-parse-print-loop">3.9 REPL</h2>


<h1 id="ch04-Evaluation">4 计算</h1>

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

<h2 id="ch04-strategies-of-evaluation">4.2 计算策略</h2>
计算这一部分也是实现解释器过程中变化最多的部分，无论哪一种语言的实现。当计算源代码的时候，有很多不同的策略可以供选择。在本书简单介绍解释器架构的时候，我已经提示到这一点，现在我们手上已经有了抽象语法树（AST），现在问题来了，接下来我们需要做些什么，如何去计算这一棵树？我们接下来看看不同的观点。

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



<h2 id="ch05-hashes">5.5 哈希表</h2>
<h2 id="ch05-the-grand-finale">5.6 完结</h2>

<h1 id="ch06-resource">6 资源</h1>
<h1 id="ch07-feedback">7 反馈</h1>

