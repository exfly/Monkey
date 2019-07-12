# 哈希

接下来要增加的数据类型是哈希表，在 `Monkey` 语言中哈希表在其他语言中也称之为映射表或者字典，它将键映射到值。

为了在 `Monkey` 语言中使用哈希表，规定哈希字面值为一个冒号隔开的键值对列表，它们被包含在一对花括号中，每一个键值对都用逗号与其他键值对分开。下面就是哈希表的样子

```monkey
>> let myHash = {"name":"Jimmy","age":72, "band":"Led Zeppelin"};
>> myHash["name"]
Jimmy
>> myHash["age"]
72
>> myHash["band"]
Led Zeppelin
```

在这个例子中 `myHash` 包含了三个键值对，它们的值都是都是字符串，在哈希表中同样适用中缀表达式，就跟数组中使用的一样。在这个例子中的索引使用字符串，这个在数组中不起作用，并且它也不是哈希表键的唯一数据类型。

```monkey
>> let myHash = {true: "yes, a boolean", 99:"correct, an integer"};
>> myHash[true]
yes, a boolean
>> myHash[99]
correct, an in
```

那同样有效，事实上，除了字符串、整型和布尔型，我们可以使用任何表达式作为索引操作符的索引。

```monkey
>> myHash[5>1]
yes, a boolean
>> myHas[100 - 1]
correct, an integer
```

只要这些表达式的执行结果是要是字符串、整型或者布尔型中的任何一个都可以作为键，在这 `5>1` 执行结果为 `true` ，`100-1` 执行结果为 `99`，两者都是`myHash` 中有效的值。

毫无疑问在 `Monkey` 的哈希表中，我们使用Go语言中的 `map` 作为数据类型，但是我们想要字符串、整型和布尔类型这些不变类型作为键，我们需要在此基础上做些工作。我们在此基础上拓展我们对象系统，但是首先要做的事将哈希字面值转换为`token`

## 词法解析哈希

我们该如何将哈希字面值转换为 `token` 呢？哪些 `token` 需要我们确认出并且作为我们的词法解析器的输出以便给接下来的语法解析器，下面是哈希字面值：

```javascript
{"name":"Jimmy","age":72, "band":"Led Zeppelin"}
```

除了字符串之外，有四个字符集在这里显得特别重要：`{`, `}`,`,`,和`:`。我们已经知道如何解析前面三种，我么的词法解析器将他们解析为 `token.LBRACE`，`token.RBRACE` 和 `token.COMMA`，也就是说我们所做的就是将`:`转换成相应的 `token`。

首先要做的是在`token`包中增加必须的类型

```go
//token/token.go
const (
//[...]
    COLON = ":"
//[...]
)
```

接下来就是为 `Lexer` 的 `NextToken` 方法增加测试使得 `token.COLON` 如期望的值。

```go
//lexer/lexer_test.go
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
{"foo":"bar"}
`
    tests := []struct {
        expectedType    token.TokenType
        expectedLiteral string
    }{
//[...]
        {token.LBRACE, "{"},
        {token.STRING, "foo"},
        {token.COLON, ":"},
        {token.STRING, "bar"},
        {token.RBRACE, "}"},
        {token.EOF, ""},
    }
// [...]
}
```

我们可以仅仅通过增加`:`来进行测试，但是使用的哈希表来进提供更多的上下文来进行测试。

```go
//lexer/lexer.go
func (l *Lexer) NextToken() token.Token{
//[...]
    case ':':
        tok = newTokne(token.COLON, l.ch)
//[...]
}
```

只需要两行代码，词法解析器就可以将 `token.COLON` 解析出来

```shell
$ got test ./lexer
ok monkey/lexer 0.006s
```

现在 `token.LBRACE`, `token.RBRACE` 和 `token.COMMA` 以及新的`tokne.COLON` 用来解析哈希表。

## 解析哈希表

在我们开始编写解析器甚至开写测试，让我们先看看哈希表额基础语法结构

```monkey
{<expression>:<expression>, <expression>:<expression>}
```

这是由逗号隔开的键值对，每一个键值对包含两个表达式，一个生成哈希表的键，另一个生成值，键和值用冒号隔开，所有的键值对有一对花括号包含起来。转换到抽象语法树的节点，我们该如何记录每一个键值对呢？我们该如何去做？是的，用 `map`字典来表示，但是字典中的键和值如何表示呢？

先前说过，只有字符串、整型和布尔型才能作为哈希表的键，但是我们不能再解析器中强制这样。我们需要在执行阶段验证哈希表的键的类型，并且生成相应的错误。

由于很多不同的表达式都能生成字符串，整型或者布尔型，不仅仅是其中的字面类型，强制数据类型在解析阶段会阻止我们写出下面的代码：

```monkey
let key = "name";
let ahs = {key: "monkey"};
```

在这里 `key` 的执行生成 `"name"`，这是合法的哈希键值，尽管它是一个标识符。为了这么做，需要允许在哈希字面中任何表达式作为键，任何表达式作为值。至少在解析阶段，我们的`ast.HashLiteral`的定义如下：

```go
//ast/ast.go
type HashLiteral struct {
    Token token.Token
    Pair map[Expression]Expression
}
func (hl *HashLiteral) expressionNode() {}
func (hl *HashLiteral) TokenLiteral() string {
    return h1.Token.Literal
}
func (hl *HashLiteral) String() string {
    var out bytes.Buffer
    pairs := []string{}
    for key, value := range hl.Pairs{
        pairs = append(pair, key.String()+":"+value.String()_
    }
    out.WriteString("{")
    out.WriteString(strings.Join(pairs, ", "))
    out.WriteString("}")
    return out.String()
}
```

现在我们已经清楚了哈希表的结构，并且已经定义了 `ast.HashLiteral` 结构，现在我们可以在解析器中编写测试。

```go
//parser/parser_test.go
func TestParsingHashLiteral(t *testing.T) {
    input := `{"one":1, "two":2, "three":3}`
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)
    stmt := program.Statements[0].(*ast.ExpressionStatement)
    hash, ok := stmt.Expression.(*ast.HashLiteral)
    if !ok {
        t.Fatalf("exp is not ast.HashLiteral. got=%T", stmt.Expression)
    }
    if len(hash.Pairs) != 3 {
        t.Errorf("hash.Pairs has wrong legnth. got=%d", len(hash.Pairs))
    }
    expected := map[string]int64{
        "one":   1,
        "two":   2,
        "three": 3,
    }
    for key, value := range hash.Pairs {
        literal, ok := key.(*ast.StringLiteral)
        if !ok {
            t.Errorf("key is not ast.StringLiteral. got=%T", key)
        }
        expectedValue := expected[literal.String()]
        testIntegerLiteral(t, value, expectedValue)
    }
}
```

当然，我们可以确保解析空的哈希表示正确，因为边界条件在代码中已经考虑到了。

```go
//parser/parser_test.go
func TestParsingEmptyHashLiteral(t *testing.T) {
    input := "{}"
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)
    stmt := program.Statements[0].(*ast.ExpressionStatement)
    hash, ok := stmt.Expression.(*ast.HashLiteral)
    if !ok {
        t.Fatalf("exp isn not ast.HashLiteral. got=%T",
            stmt.Expression)
    }
    if len(hash.Pairs) != 0 {
        t.Errorf("hash.Pairs has wrong length. got=%d", len(hash.Pairs))
    }
}
```

也增加一些新的测试，这些和 `TestHashLiteralStringKeys` 方法比较类似，但是使用使用整型和布尔型作为哈希的键。用来确保它们都能各自转换为 `*ast.IntegerLiteral` 和 `ast.Boolean`。在这里第五个测试函数确保哈希表中的值可以是任何表达式，甚至是操作符表达式。

```go
func TestParsingHashLiteralWithExpression(t *testing.T) {
    input := `{"one":0+1, "two":10 - 8, "three": 15/5}`
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)
    stmt := program.Statements[0].(*ast.ExpressionStatement)
    hash, ok := stmt.Expression.(*ast.HashLiteral)
    if !ok {
        t.Fatalf("exp is not ast.HashLiteral. got=%T",
            stmt.Expression)
    }
    if len(hash.Pairs) != 3 {
        t.Errorf("hash.Pairs has wrong length. got=%d",
            len(hash.Pairs))
    }
    tests := map[string]func(ast.Expression){
        "one": func(e ast.Expression) {
            testInfixExpression(t, e, 0, "+", 1)
        },
        "two": func(e ast.Expression) {
            testInfixExpression(t, e, 10, "-", 8)
        },
        "three": func(e ast.Expression) {
            testInfixExpression(t, e, 15, "/", 5)
        },
    }
    for key, value := range hash.Pairs {
        literal, ok := key.(*ast.StringLiteral)
        if !ok {
            t.Errorf("key is not ast.StringLiteral. got=%T", key)
            continue
        }
        testFunc, ok := tests[literal.String()]
        if !ok {
            t.Errorf("No test function for key %q found", literal.String())
            continue
        }
        testFunc(value)
    }
}
```

现在测试还不能通过，好消息只需要一个函数能够让所有测试通过，确切来说就是`prefixParseFn` 函数。由于 `token.LBRACE` 在哈希表中属于前缀表达式，就跟解析数组中的 `token.LBRACKET` 一样， 我们可以定义一个 `parseHashLiteral` 方法来作为 `prefixParseFn`。

```go
// parser/parser.go
func New(l *lexer.Lexer) *Parser {
//[...]
    p.registerPrefix(token.LBRACE, p.ParseHashLiteral)
//[...]
}
func (p *Parser) parseHashLiteral() ast.Expression {
    hash := &ast.HashLiteral{Token: p.curToken}
    hash.Pairs = make(map[ast.Expression]ast.Expression)
    for !p.peekTokenIs(token.RBRACE) {
        p.nextToken()
        key := p.parseExpression(LOWEST)
        if !p.expectPeek(token.COLON) {
            return nil
        }
        p.nextToken()
        value := p.parseExpression(LOWEST)
        hash.Pairs[key] = value
        if !p.peekTokenIs(token.RBRACE) && !p.expectPeek(token.COMMA) {
            return nil
        }
    }
    if !p.expectPeek(token.RBRACE) {
        return nil
    }
    return hash
}
```

它看上去很恐怖，但是在 `parseHashLiteral` 中没有任何新的东西，它仅仅是不停循环整个 `key-value` 表达式对，并且检查 `token.RBRACE` 调用`parseExpression` 两次。这就完成了 `hash.Pairs` 中最重要的部分。

```shell
$ go test ./parser
ok monkey/parser 0.006s
```

所有的解析测试通过了，通过增加测试的数量，我们可以有理由确信我们的解析器知道如何正确处理哈希表。这也是意味着将要解释器中的哈希表最有趣的部分：表示哈希表中的对象系统和执行哈希表。

## 哈希对象

除了拓展词法解析器和语法解析器，增加数据类型也就意味着我们它们要在对象系统的表达出来。我们已经成功的为整数、字符串和数组完成了这些工作，但是完成这些数据类型只要定义一个结构，其拥有一个正确的`.Value`字段。但是哈希表需要做一些额外的工作，接下来我解释为什么。

首先假设我们定义新的`object.Hash`类型如下：

```go
type Hash strcut {
    Pairs map[Object]Object
}
```

显然我们只需要借助 `go` 语言中的 `map` 数据结构完成哈希表示，但是这样做的化，我们该如何实现哈希表中的键值对呢？更重要的的是我们该如何获取相应的值？

考虑如下的Monkey代码片段

```monkey
let hash = {"name": "Moneky"};
hash["name"]
```

让我们使用上述定义的 `object.Hash` 来执行上述两行代码，在执行第一行代码的时候，我们将每个键值对放到 `map[Object]Obejct` 字典中去，导致`.Pairs` 拥有如下的映射关系：一个 `*obejct.String` 的 `.Value` 字段`"name"` 映射到 `*object.String` 的 `.Value` 字段的 `"Monkey"` 中。

到目前为止没有任何问题，问题出现了，当我们执行第二行代码的时候，我们使用索引表达式来访问 `Moneky` 字符串。

在第二行代码中 `"name"` 字符串的索引表达式执行结果为重新分配的`*obejct.String` 对象。尽管这个新的 `*obejct.String` 的 `.Value` 字段中包含了 `"name"`，它和键值对中的 `*object.String` 相同，但是我们不能使用新的来恢复 `Monkey` 字符串。

原因是它们是指向不同内存位置的指针，但是事实它们内存指向的位置内容并没有关系。比较这些指针将会告诉我们他们是不同的，也就一位置我们新创建的`*object.String` 作为 `key` 并不能得到 `Monkey`。这也是 `Go` 语言中指正比较不同。

接下来的例子就是展示我们使用`object.Hash`所面临的的难题。

```go
name1 := &object.String{Value:"name"}
monkey := &object.String{Value:"Monkey"}

pairs := map[object.Object]object.Object{}
pairs[name1] = monkey
fmt.Printf("pairs[name1]=%+v\n", pairs[name1])
//=> pairs[name1]=&{Value:Monkey}
name2 := &obejct.String{Value:"name"}
fmt.Printf("pairs[name2]=%+v\n", pairs[name2])
//=> paris[name2]=<nil>
fmt.Printf("(name1==name2)=%\t\n", name1==name2
//=> (name1 == name2)=false
```

其中的一个解决方案就是迭代`.Pairs`中键，检查`.object.String`的`.Value`字段，判断他们是否相等。但是这种方法将会使得查找特定 `key` 的时间复杂度从`O(1)`变成 `O(n)`，这个完全违背了我们当初选择哈希的意图。另外一种选择方案是定义 `Pairs` 作为 `map[string]Object` ， 然后是用`.Value` 字段 `.object.String` 作为其中作为键，虽然有效，但是对于整数和布尔型数据不起作用。

我们现在要做的为哈希表创建一个键，使得他们很容易的区分开来。我们需要为`object.String` 生成一个哈希键，并且对于相同的 `object.String` 比较是相同的。 对于 `object.Integer` 和 `object.Boolean` 也是同样如此。但是对于 `*object.String` 和 `*object.Integer` 或者`*object.Boolean` 必须不可能相等，这些哈希的键值必须不能相等。

接下来是一组测试函数，它们表达了我们想要在我们的对象系统的期望的行为：

```go
// object/object_test.go
package object
import "tesing"
func TestStringHashKey(t *testing.T) {
    hello1 := &String{Value: "Hello World"}
    hello2 := &String{Value: "Hello World"}
    diff1 := &String{Value: "My name is johnny"}
    diff2 := &String{Value: "My name is johnny"}
    if hello1.HashKey() != hello2.HashKey() {
        t.Errorf("string with same content have different keys")
    }
    if diff1.HashKey() != diff2.HashKey() {
        t.Errorf("string with same content have differnet keys")
    }
    if hello1.HashKey() == diff1.HashKey() {
        t.Errorf("string with different have same hash key")
    }
}
```

从 `HashKey()` 方法就是的确我们想要得到的，不仅仅是 `*object.String`而且是 `*object.Boolean` 和 `*object.Integer`，这也是同样的测试函数都存在的原因。

接下来我们将要为以下三个类型都实现 `HashKey()` 方法。

```go
//obejct/object.go
import (
// [...]
    "hash/fnv"
)
type HashKey() struct {
    Type ObjectType
    Value uint64
}
func (b *Boolean) HashKey() HashKey {
    var value uint64
    if b.Value {
        value = 1
    }else{
        value = 0
    }
    return HashKey{Type: b.Type(), Value: uint64(i.Value)}
}
func (i *Integer) HashKey() HashKey {
    return HashKey{Type: i.Type(), Value: uint64(i.Value)}
}
func (s *String) HashKey() HashKey {
    h := fnv.New64a()
    h.Write([]byte(s.Value))
    return HashKey{Type: s.Type(), Value: h.Sum64()}
}
```

每一个 `HashKey()` 方法返回一个 `HashKey`，就跟你看到定义的一样，`HashKey` 并没有什么神奇的地方，`Type` 字段包含了一个 `Object.Type`，它能有效的区分不同类型的键值。 `Value` 字段包含了真正哈希值，就是一个整数。由于他们就是整数，所以我们对于不同的 `HashKey` 使用 `==` 操作符来比较不同。这也意味着我们使用 `HashKey` 就跟 `Go` 语言中 `map` 一样。

现在先前的问题可以使用 `HashKey` 来解决。

```go
name1 := &object.String{Value: "name"} monkey := &object.String{Value: "Monkey"}
pairs := map[object.HashKey]object.Object{} pairs[name1.HashKey()] = monkey
fmt.Printf("pairs[name1.HashKey()]=%+v\n", pairs[name1.HashKey()])
// => pairs[name1.HashKey()]=&{Value:Monkey}
name2 := &object.String{Value: "name"} fmt.Printf("pairs[name2.HashKey()]=%+v\n", pairs[name2.HashKey()])
// => pairs[name2.HashKey()]=&{Value:Monkey}
fmt.Printf("(name1 == name2)=%t\n", name1 == name2) // => (name1 == name2)=false
fmt.Printf("(name1.HashKey() == name2.HashKey())=%t\n", name1.HashKey() == name2.HashKey())
// => (name1.HashKey() == name2.HashKey())=true
```

就这事我们想要的，`HashKey` 中定义的 `HashKey()` 方法可以很好地解决使用原生的 `Hash` 定义，这也让测试通过。

```shell
$ go test ./object
ok monkey/object 0.008s
```

现在我们可以顶一个 `object.Hash`，并且使用新的 `HashKey` 类型：

```go
//object/object.go
const (
// [...]
    HASH_OBJ = "HASH"
)
type HashPair struct {
    Key Object
    Value Object
}
type Hash struct {
    Pairs map[HashKey]HashPair
}
func (h *Hash) Type() ObjectType { return HASH_OBJ }
```

增加了 `Hash` 和 `HashPair`，`HashPair` 是 `Hash.Pairs`的值类型。你可能会奇怪为什么我们不直接定义 `map[HashKey]Object` 来代替 `Pairs`

原因就在于 `Inspect()` 方法，当我们在 `REPL`中输出 `Monkey` 中的哈希表时候，我们想要打印出每一个值关联的的键，仅仅打印出 `HashKey` 是没有用的。所以我们记录每一个 `HashKey` 关联的对象，通过使用 `HashPairs` 是有用的。在这里我们保存了原先的键对象和值对象。通过这种方法，我们调用 `Inspect()` 方法，可以生成 `*object.Hash` 对象，下面就是 `Inspect()`方法实现：

```go
//object/object.go
func (h *Hash) Inspect() string {
    var out bytes.Buffer
    pairs := make([]string, 0)
    for _, pair := range h.Pairs {
        pairs = append(pairs, fmt.Sprintf("%s: %s",
            pair.Key.Inspect(), pair.Value.Inspect()))
    }
    out.WriteString("{")
    out.WriteString(strings.Join(pairs, ", "))
    out.WriteString("}")
    return out.String()
}
```

`Inspect()` 方法不是唯一我们记录对象的 `HashKey` 原因，它也是想要实现`range` 函数，为每个哈希对象迭代它们的每一个键值。或者我们想要增加`firstPair` 函数，它返回第一个键值对，就跟数组一样。总之记录键值对是非常有用的，尽管现在我们好像在 `Inspect` 中使用了它们。

现在好像所有都完成了，但是我们还有一件小事需要去完成：

```go
//object/object.go
type Hashable interface {
    HashKey() HashKey
}
```

在这里我们使用 `Hashable` 接口，来确保给定的对象使用可以使用哈希键，目前来讲只有 `*object.String`，`*object.Boolean` 和 `*object.Integer`对象实现了这个接口。还有一件事可以优化 `HashKey()` 方法，可以借助他们的缓存值，这个听上去是个不错的性能优化的优化方法。

## 执行哈希表

首先增加一些测试用例：

```go
func TestHashLiterals(t *testing.T) {
    input := `let two="two";
    {
        "one":10-9,
        two:1+1,
        "thr" + "ee" : 6/2,
        4 : 4,
        true:5,
        false:6
    }`
    evaluated := testEval(input)
    result, ok := evaluated.(*object.Hash)
    if !ok {
        t.Fatalf("Eval did't return Hash. got=%T(%+v)",
            evaluated, evaluated)
    }
    expected := map[object.HashKey]int64{
        (&object.String{Value: "one"}).HashKey():   1,
        (&object.String{Value: "two"}).HashKey():   2,
        (&object.String{Value: "three"}).HashKey(): 3,
        (&object.Integer{Value: 4}).HashKey():      4,
        TRUE.HashKey():                             5,
        FALSE.HashKey():                            6,
    }
    if len(result.Pairs) != len(expected) {
        t.Fatalf("Hash has wrong num of pairs. got=%d", len(result.Pairs))
    }
    for expectedKey, expectedValue := range expected {
        pair, ok := result.Pairs[expectedKey]
        if !ok {
            t.Errorf("no pair for given key in Pairs")
        }
        testDecimalObject(t, pair.Value, expectedValue)
    }

}
```

这个测试函数展示了当我们遇到 `*ast.HashLiteral` 的时候，如何执行`Eval`：一个 `*object.Hash` 对象包含许多 `HashPair`，将他们映射到对应的 `HashKey` 中。它同样展示其他的要求：字符串、标识符、中缀表达式、布尔型和整型都是合法的键。任何表达式都是合法的，只要它们能够生成的对象实现了`Hashable` 接口，那么就可以作为哈希键。

它们的值可以有任何表达式推导出来。在这里我们通过 `10-9` 来生成 `1` ， `6 / 2` 变成 `3` 来进行测试。

目前测试时失败的，为了让测试通过需要拓展我们的`Eval`函数，增加`*ast.HashLiterals`分支即可。

```go
//evalutor/evalutor.go
func Eval(node ast.Node, env *object.Environment) object.Obejct {
// [...]
    case *ast.HashLiteral:
        return evalHashLiteral(node, env)
// [...]
}
```

这个`evalHashLiteral`函数可能看上去非常吓人，但是事实上非常简单：

```go
//evaluator/evaluator.go
func evalHashLiteral(node *ast.HashLiteral, env *object.Environment) object.Object {
    pairs := make(map[object.HashKey]object.HashPair)
    for keyNode, valueNode := range node.Pairs {
        key := Eval(keyNode, env)
        if isError(key) {
            return key
        }
        hashKey, ok := key.(object.Hashable)
        if !ok {
            return newError("unusable as hash key: %s", key.Type())
        }
        value := Eval(valueNode, env)
        if isError(value) {
            return value
        }
        hashed := hashKey.HashKey()
        pairs[hashed] = object.HashPair{Key: key, Value: value}

    }
    return &object.Hash{Pairs: pairs}
}
```

当迭代 `node.Pairs` 时候，`keyNode` 首先被执行，除了检查它调用 `Eval`函数的时候是否出现错误，我们同样做了一次类型判断：它是否实现了`object.Hashable` 接口，否则它不能作为哈希键。这也是我们为什么增加`Hashbale` 定义。

当我们再次调用 `Eval` 函数，去执行 `valueNode`。如果没有生成错误，那么将新生成的键值对存放到 `pairs` 字典中。我们通过调用 `HashKey` 对象调用`HashKey()` 方法生成一个键。然后初始化一个新的 `HashPair`，并且指向`key` 和 `value`，然后将他们添加到 `pairs` 中。

现在我们的测试通过:

```shell
$ go test ./evaluator
ok monkey/evaluator 0.007s
```

这也意味着我们可以在我们的REPL中使用哈希表：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> {"name": "Monkey", "age":0, "type":"Language", "status":"awesome"}
{age:0, type: language, status:awesome, name:Monkey}
```

现在还不能获取索引的元素

```monkey
>> let bob = {"name": "Bob", "age", 99}
>> bob["name"]
ERROR: index operator not supported: HASH
```

接下来我们将会修复这个问题

## 执行哈希表的索引表达式

还记得在 `evalIndexExpression` 中增加的 `switch` 语句吗？还记得我曾经告诉你我们将会增加其他 `case` 分支，现在我们就来做这件事。

开始我们增加一些测试函数来确保我们可以通过索引表达式访问哈希表中的值：

```go
//evaluator/evaluator.go
func TestHashIndexExpression(t *testing.T) {
    tests := []struct {
        input    string
        expected interface{}
    }{
        {
            `{"foo":5}["foo"]`,
            5,
        },
        {
            `{"foo":5}["bar"]`,
            nil,
        },
        {
            `let key = "foo"; {"foo":5}[key]`,
            5,
        },
        {
            `{}["foo"]`,
            nil,
        },
        {
            `{5:5}[5]`,
            5,
        },
        {
            `{true:5}[true]`,
            5,
        },
        {
            `{false:5}[false]`,
            5,
        },
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

就跟 `TestArrayIndexExpression` 中一样，我们确保我们使用索引表达式能够得到正确的值，只不过这次是哈希表。不同的是在测试用例中我们使用字符串、整型和布尔型哈希键来获取哈希表中的值。所以从本质上来讲就是这些测试就是用来判断这个 `HashKey` 方法在不同数据类型情况下是否正确调用。

同样这边确保如果使用一个没有实现 `object.Hashable` 对象作为哈希表中的键将会生成一个错误，我们可以在 `TestErrorHandling` 测试函数中增加相应的测试用例。

```go
//evaluator/evaluator.go
func TestErrorHandling(t *testing.T){
    tests := []struct{
        input string
        expectedMessage string
    }{
//[...]
        {
            `{"name", "monkey"}[fn(x){x}]`,
            "unusable as hash key: FUNCTION",
        },
    }
// [...]
}
```

目前测试是失败的，这也意味着我们要在 `evalIndexExpression` 函数中的`switch` 语句中增加新的 `case` 分支：

```go
//evalutor/evaluator.go
func evalIndexExpression(left, index object.Object) object.Object {
    switch {
    case left.Type() == object.ARRAY_OBJ && index.Type() == object.INTEGER_OBJ:
        return evalArrayIndexExpression(left, index)
    case left.Type() == object.HASH_OBJ:
        return evalHashIndexExpression(left, index)
    default:
        return newError("index operator not support:%s", left.Type())
    }
}
```

现在新的分支调用新的函数：`evalHashInexExpression`，已经知道`evalHashInexExpression` 该如何去工作，因为成功在测试`object.Hashable` 使用过。

```go
func evalHashIndexExpression(hash, index object.Object) object.Object {
    hashObject := hash.(*object.Hash)
    key, ok := index.(object.Hashable)
    if !ok {
        return newError("unusable as hash key: %s", index.Type())
    }
    pair, ok := hashObject.Pairs[key.HashKey()]
    if !ok {
        return NULL
    }
    return pair.Value
}
```

现在测试通过

```shell
$ go test ./evaluator
ok monkey/evalutor 0.007s
```

现在我们成功地从哈希表中获取相应的值，不信就可以往下看：

```shell
$ go run main.go
Hello mrnugget! This is the monkey programming language!
Feel free to type in commands
>> let people = [{"name": "Alice", "age": 24}, {"name":"Anna", "age":28}];
>> people[0]["name"]
Alice
>> people[1]["age"]
28
>> people[1]["age] + people[0]["age"]
52
>> let getName = fn(person){ person["name"];};
>> getName(people[0]);
Alice
>> getName(people[1]);
Anna
```
