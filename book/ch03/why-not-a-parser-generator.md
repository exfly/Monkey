# 为什么不用解析器生成器

或许你已经听说过解析器生成器，比如 `yacc`，`bison` 或者 `ANLTR` 等等。解析器生成器是一种通过阅读语言的描述形式，将解析器输出的工具。输出的代码可以被编译或者解释，然后生成语法树。

解析器生成器有很多种，主要不同点在于输入的形式和输出的语言。它们大部分使用上下文无关语法（CFG，Context Free Grammer)作为输入，`CFG` 是一列规则，它描述了如何构建正确的语句，最长江的 `CGF` 形式是 `BNF, Backus-Nau Form` 或者 `EBNF，Extend Backus-Nau Form`。

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

这是 `EcmaScript` 语言使用的[ `BNF` 描述](http://tomcopeland.blogs.com/EcmaScript.html)，解析器生成器可以将这些类似描述转换为 `C` 代码。

你可能认为我们应该使用解析器生成器而不是自己手动写，那么就可跳过本章节，问题就已经基本上解决了。解析的过程是计算机科学中最容易理解的部分，很多聪明的人在这个上面花费了大量的时间，都很多已有的成果，为什么不站在巨人的肩膀上呢？

但是我认为学习编写自己的解析过程不是浪费时间，而是包含巨大的价值。只有自己写过解析器，至少尝试过就能看到这些解析器提供的便利性已经包含的缺陷。对我而言，只有我自己写过第一个解析器，它们的概念才会想鼠标单机一样让人容易理解，才会知道代码是如何转换的。

如果有人建议你使用解析器生成器，我想他们肯定自己之前写过解析器，或者他们知道需要自己完成什么目标工作，因为在工作生产环境中，解析器的正确性和健壮性是优先考虑的。

这里我们将会学习如何编写解析器，我们想知道解析器是如何工作的。还是同样的观点，只有自己手动去完成一个解析器，才知道它是如何工作。
