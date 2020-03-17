# JavaScript 与 AST

# 定义

> *It is a hierarchical program representation that presents source code structure according to the grammar of a programming language, each AST node corresponds to an item of a source code.*
>
> **抽象语法树**，是[源代码](https://zh.wikipedia.org/wiki/源代码)[语法](https://zh.wikipedia.org/wiki/语法学)结构的一种抽象表示。它以[树状](https://zh.wikipedia.org/wiki/树_(图论))的形式表现[编程语言](https://zh.wikipedia.org/wiki/编程语言)的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。比如，嵌套括号被隐含在树的结构中，并没有以节点的形式呈现；而类似于 `if-condition-then` 这样的条件跳转语句，可以使用带有三个分支的节点来表示。

# 简单解释

对于一段 JavaScript 代码转为 AST:

```javascript
var a = 2;
```

需要经历几个阶段:

## 词法分析

> First step. Lexical analyzer, also called scanner, it reads a stream of characters (our code) and combines them into tokens using defined rules. Also, it will remove white space characters, comments, etc. In the end, the entire string of code will be split into a list of tokens.

> 这个过程会将由字符组成的字符串分解成（对编程语言来说）有意义的代码块，这些代码块被称为词法单元（token）。例如，考虑程序 var a = 2;。这段程序通常会被分解成为下面这些词法单元：var、a、=、2 、;。空格是否会被当作词法单元，取决于空格在这门语言中是否具有意义。

## 句法分析

> Second step. Syntax analyzer, also called parser, will take a plain list of tokens after Lexical Analysis and turn it into a tree representation, validating language syntax and throwing syntax errors, if such happened.

> 这个过程是将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树。这个树被称为“抽象语法树”（Abstract Syntax Tree，AST）。 var a = 2; 的抽象语法树中可能会有一个叫作 VariableDeclaration 的顶级节点，接下来是一个叫作 Identifier（它的值是 a）的子节点，以及一个叫作 AssignmentExpression 的子节点。AssignmentExpression 节点有一个叫作 NumericLiteral（它的值是 2）的子节点。

## 例子

`var a = 2;` 的 AST.

```json
// acron ESTree
{
    "type": "VariableDeclaration",
    "start": 0,
    "end": 10,
    "declarations": [
        {
            "type": "VariableDeclarator",
            "start": 4,
            "end": 9,
            "id": {
                "type": "Identifier",
                "start": 4,
                "end": 5,
                "name": "a"
            },
            "init": {
                "type": "Literal",
                "start": 8,
                "end": 9,
                "value": 2,
                "raw": "2"
            }
        }
    ],
    "kind": "var"
}
```



# 参考

> https://blog.buildo.io/a-tour-of-abstract-syntax-trees-906c0574a067
>
> https://astexplorer.net/
>
> https://itnext.io/ast-for-javascript-developers-3e79aeb08343
>
> 《你不知道的JavaScript(上卷)》