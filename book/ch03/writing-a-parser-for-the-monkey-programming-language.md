# 为 `Monkey` 编程语言编写解析器

在编写解析器有两种策略：自顶向下和自底向上。每个不同的策略都有微小的不同，比如递归下降解析、提前解析或者预测解析都是自顶向下的变种。

这里采用的策略是递归下降解析，更确切的说是一种*自顶向下操作符优先级*解析器，也叫做 `Pratt` 解析器，根据算法的发明人 `Vaughan Pratt` 命名的。

我这里不会涉及到不同策略的细节中，因为这样既占用空间而且无法精确地描述它们。对我而言，指定向下解析和自底向上解析的不同点在于前者从抽象语法树的跟节点开始，而后者恰恰相反。递归下降解析器常常被推荐给新手，因为它和我们想象中构建抽象语法树是一致的。尽管它需要在理解概念之前先编写代码，这也是我为什么开始编写代码而不是详细探究解析策略。

自己动手编写解析器，需要作出一些取舍，我们的解析器不是最快的，也没有正式的证明解析器正确性，而且不提供错误恢复和语法错误。最后需要澄清的是如果没有额外的相关理论学习，弄清解析器非常困难。不管怎样，我们拥有了一个能够完全工作的解析器，而且保留了拓展部分，如果你对这部分内容感兴趣，可以奠定坚实的基础。

首先我们从解析 `let` 和 `return` 开始，当我们能够理解解析语句和基本结构，就会转向表达式解析并且学习如何解析它们（这也是 `Vaughan Pratt` 发挥作用的时候）。最后我们拓展解析器，使它能够解析 `Monkey` 语言更大的子集，最终形成一棵抽象语法树。
