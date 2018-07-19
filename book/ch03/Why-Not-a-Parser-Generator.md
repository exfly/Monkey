为什么不适用解析器生成器

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