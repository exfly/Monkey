# 用 Go 语言实现解释器

翻译人员

- [gaufung](https://github.com/gaufung)

- [Jehu Lu](https://github.com/lwhile)

目录

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


<h2 id="ch01-introduction">1 前言</h2>