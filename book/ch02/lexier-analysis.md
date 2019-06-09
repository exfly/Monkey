# 词法分析

为了让 `Monkey` 源码能够工作，我们需要源码转换为另一种形式，在执行之前，我们会将源代码转换为两次。
![转换](../figures/transform.png)

首先要做的是将源代码转换为 `token`，这个过程叫做词法分析，它是由词法分析器完成的。

`token` 是非常小、分门别类的数据结构，在第二步过程中将这些 `token` 转换为抽象语法树。

下面是简单的例子，它是词法解析器的输入：

```monkey
"let x = 5 + 5;"
```

那么从词法解析器中的输出是

```monkey
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

这里的每一个 `token` 对应着源码一部分，`"let"` 对应 `LET`, `"+"` 对应 `PLUS_SIGN` 等等如此。`IDENTIIER` 对于这 `"x"` 而 `IDENTIIER` 对应具体 `5` 而不是 `"5"`，具体的 `token` 构成有着不同的实现。

还有一点要注意的是，空白符都会被忽略，因为在 `Monkey` 语言中空白符是没有意义的。比如下面的输入时没有问题的：

```monkey
let x = 5;
```

亦或者这么输入

```mokey
let x  =   5;
```

在其他语言，比如 `Python`，空白符是由意义的，这也就意味着词法解析器不能吃掉他们，包括换行符。词法解析器也不能将其当做 `token` 输出。

一个好的词法解析器的每个 `token` 也需要包含行号，列号和文件名，为什么要这么做呢？举个例子来讲，后面解析的过程中如果出现错误，可以输出相应的信息，而不是仅仅输出 `"error: expected semicolon token"`，而是这样输出：

```monkey
"error: expected semicolon token. line 42, coloumn 23, program.monkey"
```

但是我们不打算这样做，并不是实现起来困难，而是会导致我们的 `token` 和词法解析器变得复杂起来，更加难以理解。
