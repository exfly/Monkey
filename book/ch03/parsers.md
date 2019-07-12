# 解析器

每一个写过代码的人都听说过解析器，其中大部分是 `parser error` 或者说“我们需要解析它”，“在解析后”，“解析器无法理解输入”。每个开发者都知道解析器的存在，但是他们对着理解正确吗？

但是什么才是真正的解析器呢？它的作用是什么，应该要做什么？下面是[维基百科的定义](https://en.wikipedia.org/wiki/Parsing#Parser):
> 解析器是软件的组成部分，它接受输入数据（通常为文本）然后构建数据结构，往往是一棵解析树、抽象语法树或者其他继承体系结构。它们给出输入结构过程中也会检查其中的语法错误。解析器的先前步骤通常是词法分析器，它的输出是字符串构成的一些列 `token`。

维基百科上的摘录非常容易理解，甚至提到了我们之前提及的词法解析器。

解析器将输入转换成一个数据结构，听上去非常抽象，让我们用简单的`javascript` 例子来说明：

```js
> var input = '{"name": "Thorsten", "age": 28}'
> var output = JSON.parse(input)
> output
{name: 'Thorsen', age: 28}
> output.name
'Thorsen'
> output.age
28
```

这里我们输入一些文本字符串，将其传递给隐藏在 `JSON.parse` 函数后面的解析器，然后得输出值。输出内容用 `Javascript` 对象代表了输入文本，它包含两个字段 `name` 和 `age`，它们的值对应各自的输入，现在我们可以很轻松地使用这个数据结构访问 `name` 和 `age` 字段。

你可能呢会说：
> `JSON` 解析器和编程语言的解析器是不一样的

但是从概念上将本质内容是一样的，一个 `JSON` 解析器用一个数据结构表示输入，而编程语言的解析器也是同样如此；不同点就是 `JSON` 解析器可以从输入看到数据结构，但是换成下面的代码，这个数据结构一下子是看不出来的

```js
if ((5+2*3)==91) { return compterStuff(input1, intput2); }
```

这是为什么呢？因为代码表示的数据结构在概念上更加抽象，而平时工作对于编程语言的解析器输出不是特别熟悉，因为我们很少接触解析后的源代码以及中间的表示形式。`Lisp` 程序员却是个例外，因为在 `Lisp` 代码中，源码和源码表示的数据结构是一样的，这就是所说的 *代码就是数据，数据就是代码*。

在大多数解析器或者编译器中，源代码的中间表示形式的数据结构叫做语法树或者抽象语法树。*抽象*的概念基于这样一个事实：在抽象语法书中，代码的详细细节被忽略。分号、换行、空白、注释、花括号、中括号和括号都是依赖于具体的语言，解析器不会再抽象语法树中表示它们，它们仅仅辅助如何去构建这棵树。

需要承认的是没有一个正确的、通用的抽象语法树适用于每一个解析器。它们实现都是概念上的类似，细节上不同，具体实现依赖于它要解析的编程语言。

假如使用 `javascript` 也包含一个 `MagicLexer` 解析器，它能构建出以一棵用 `javascript` 对象表示的抽象语法树，那么解析的过程之后就会生成如下的内容：

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

上述例子中，解析器输出了抽象语法树，结果相当抽象。这里没有括号，没有冒号也没有花括号。但是它很精确地表示出源代码。这就是解析器所做的工作，它将源代码作为输入（文本和 `token` 都可以），然后输出表示输入的数据结构。在构建数据结构的时候，不可避免地分析输入，检查它们是否满足期待的数据结构，因此解析的过程也称为语法解析。

在本章中，我们将为我们的 `Monkey` 编程语言编写解析器，它的输入时我们前面章节定义好的 `token`，它是由词法分析器完成。然后输出由这些 `token` 定义出来的抽象语法树。
