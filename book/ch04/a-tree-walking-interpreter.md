# 树遍历解释器

我们将要构建一棵树遍历解释器，对之前解析步骤完成的抽象语法树边遍历边解释，跳过预处理和编译的步骤。

我们的解释器和进店的 `Lisp` 解释器一样，其设计受到《计算机程序结构和描述》（`The Structure and Interpretation of Computer Program`) 中描述的解释器影响很大，由于是关于环境的使用。但是这并不意味着我们去复制这个特定的解释器，如果你足够了解的话，你会从中看到很多解释器的影子。这也是我们这样设计的原因，它很容易理解也很容易的拓展。

整个过程包含两部分工作：一是遍历整个数执行；另一个是用我们的宿主语言 `Go` 来表达 `Monkey` 中的值。执行这个过程听上去很强大，但是它仅仅是一个 `eval` 函数的调用，这函数的唯一作用是计算抽象语法树的值，下面的伪代码就是用来表示解释器在上下文是怎么执行和遍历树。

```monkey
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

正如你所见的，`eval` 函数是递归的， 如果 `astNode` 是 `infixExpression`，那么调用 `eval` 自身两次分别计算左右两边的操作数，它将会引起另外其他中缀表达式计算或者整型计算或者布尔型计算。在构建和测试抽象语法树的过程中我们也看到这种递归的形式，在这里也是同样的概念，唯一的区别是在这我们计算而不是构建这棵抽象语法树。

从伪代码的片段中，我们可以大致想想对这函数扩展是非常简单的，这也是我们非常擅长的工作。在这我们会一点点构建自己的 `eval` 函数，随着解释器的拓展，我们一点点增加新的分支和功能。

在这片段中最有趣的是返回语句，它们到底返回什么？下面的两行是将函数调用的返回值绑定到相应的变量中

```monkey
leftEvaluated = eval(astNode.Left)
rightEvaluated = eval(astNode.Right)
```

那么它返回的是什么？返回值的类型又是很么？也就说我们的解释器中拥有哪些内存对象系统？
