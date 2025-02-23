# 解析表达式

主观来讲，解析表达式是编写解析器中最有趣的部分，正如我们看看看到的，解析器语句比较直接：从左到右处理每一个 `token`，期待或者拒绝下一个 `token` 直到返回抽象语法树的节点。

另一方面，解析表达式有更多的挑战。操作符优先级是第一个需要考虑的，接下来就是例子：

```monkey
5 * 5 + 10
```

用抽象语法树表示如下

```monkey
((5 * 5) + 10)
```

也就是说 `5 * 5` 在抽象语法树中要更低一点，比加法更早执行。为了生成这样的抽象语法树，解析器需要知道操作符的优先级，也即是说 `*` 的优先级比 `+` 要高。这个先决条件非常重要，比如下面的形式：

```monkey
5 * (5 + 10)
```

在这里括号将 `5 + 10` 表达式组合在一起，使它们的优先级得到提升，现在加法在乘法之前执行，因为括号比 `*` 有更高的优先级。接下来我们会看到更多的例子，优先级扮演者重要的角色。

另一个挑战是在表达式中，同样的 `token` 可能会出现在不同的位置，而 `let` 只能出现 `let` 语句开始的地方，这样很容易帮助我们决定剩下的语句应该是什么。比如下面的表达式：

```monkey
-5-10
```

这里的 `-` 操作符出现在表达式开始的地方，作为前缀操作符，然后作为中缀操作符出现在中间。同样的挑战如下：

```monkey
5 * (add(2, 3) + 10)
```

尽管你可能不会认为括号也是操作符，但是同样的问题也出现在刚刚的 `-` 中。最外卖你的括号表示组合表达式，里面的括号表示调用表达式。 `token` 位置是否有效取决于上下文，借此来确定它们之间的优先级。

## `Monkey` 中的表达式

在 `Monkey` 语言中，除了 `let` 和 `return` 语句外， 其余地都是表达式，这些表达式都有不同的形式。

`Monkey` 语言中的中缀表达式：

```monkey
-5
!true
!false
```

当然还有中缀表达式：

```monkey
5 + 5
5 - 5
5 / 5
5 * 5
```

除了基础的算术操作符，也有一些比较操作符：

```monkey
foo == bar
foo != bar
foo < bar
foo > bar
```

跟之前遇到一样，可以使用括号表达爱是组合起来影响执行顺序：

```monkey
5 * (5 + 5)
((5 + 5) * 5) * 5
```

还有调用表达式：

```moneky
add(2, 3)
add(add(2, 3), add(5, 10)) max(5, add(5, (5 * 5)))
```

标识符同样也是表达式

```monkey
foo * bar / foobar
add(foo, bar)
```

函数在 `Monkey` 语言中是一等公民，字面函数也是表达式。我们可以使用 `let` 语句将一个函数绑定到一个标识符上，字面函数就是语句形式的表达式：

```monkey
let add = fn(x, y) { return x + y; };
```

也可以使用字面函数代替标识符：

```monkey
fn(x, y) { return x +y; }(5, 5)
(fn(x){ return x}(5) + 10) * 10
```

和其他语言一样，`Monkey` 语言中也有 `if` 表达式：

```monkey
let result = if (10 > 5) {true} else {false};
result // => true
```

看到上述所有的不同形式的表达式，我们非常清楚需要很好的方法来正确解析器它们。之前基于 `Token` 的方法解析语句的方法不再适用，这时候就轮到 `Vaughan Patt` 方法大显神威的时候。
