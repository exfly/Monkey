## 前言

> 解释器是令人着迷的

虽然每个人对上面的观点都有自己的看法，但是我始终认为这是正确的，接下来我将一步步告诉你其中的原因。

表面上来看，解释器非常简单：输入内容，然后得到结果。它就是一个程序，读入其他程序代码，然后生成对应的东西。但是如果你越考虑这个问题，你就觉得它更加迷人。看上去随机组成的字符，包括字符、数字或者其他特殊意义的符号输入到解释器后就变得有意义了。计算机本质上是只能理解 `0` 和 `1` 的机器，但是却能理解我们输入给他的字符并转换成相关的指令，这些都是解释器在读取过程中完成的*翻译*。

我曾经不停地问我自己：解释器到底是如何工作的？最近我才知道只有我自己写一个解释器才能真正明白其中的答案，所以开始着手这件事情。

关于解释器有很多书、文章、博客或者教程，它们大多数划分到下面两种风格中：一种是涉及的主题非常宏大，充斥着理论知识，面向是专业的学术研究者；另一种却是非常简短，仅仅提供简单的基础介绍，将其中很多功能借助外部工具当做黑盒子使用。

这次我想与众不同，我确实想知道解释器细节包括词法解释器和语法解释器如何工作，尤其类 `C` 语言这样有花括号和分号的编程语言。开始的时候我不知道如何下手，需要从那些学术书籍中找到答案，当然从充斥着冗长、理论化的解释和数学符号内容中很难找到我想要答案。

我想要的是一个介于 900 页关于编译器的书和用 `ruby` 50行代码编写的 `Lisp` 解释器的博文之间的内容。我想这本书是为喜欢一探究竟的人亦或者是喜欢知道一切如何工作的人准备的，当然也包括我。

在这本书中，我将从零开始编写一门语言的解释器，而不借助任何第三方工具或者库。这门语言不会在生产实际中使用，也不会过多关注性能上的工作。虽然这个解释器支持的编程语言会或多或少有些功能上缺失，但是我们会从中学到很多。

由于解释器种类繁多而且并没有很大的共性，所以用通用的语句很难去描述解释器。笼统的来讲就是它阅读源代码并且读取它，生成一系列可继续操作的结果。与此同时编译器就恰恰相反，它读取源代码并且生成背后机器理解的代码。

有些解释器非常短小简单，甚至没有包含解析的步骤，仅仅是立马解析输入内容，比如 `Brainfuck` 语言。

在其他精心涉及的解释器中，包含了大量优化。使用先进的解析和计算技术。其中还有一些不计算输入，而且将其编译成字节码的中间码，然后计算这些中间码。还有更先进的是 `JIT` 解释器，它将输入内容编译成本地机器码，最后执行。

还有一种解释器是解析源代码，然后构建一棵抽象语法树(`AST, Abstract Syntax Tree`)，解释器开始计算这棵树，这叫做 `Tree-Walking` 解释器，因为解释的流程好像在这棵树上行走。

本书我们将会构建一个类 `Tree-Walking` 的解释器。除此之外，我们还会构建词法解析器、语法解析器、树表达式和最后的计算器。我们会看到什么是 `token`，什么是抽象语法树、如何去构建这棵树，如何去计算这个树以及如果拓展我们的语言和实现一些内置函数。
