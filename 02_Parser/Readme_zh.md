# 第 2 部分：解析的介绍

在我们的编译器制作旅程中的本部分，我会介绍解析器的基本概念。如我在第 1 部分中提到的，解析器的工作是识别输入内容的语法和结构元素，并确保它们符合语言的 *语法(grammar)*。

我们已经有一些可以被扫描到的语言元素了，它们就是我们的 token：

 + 四个基础的数学运算符：`*`, `/`, `+` and `-`
 + 由一个或更多的数字 `0` .. `9` 组成的十进制整数

现在让我们来定义一下我们的解析器所能识别的语言的语法吧。

## BNF: 巴科斯-诺尔范式(Backus-Naur Form)

在处理计算机语言的时候，你总会在某个遇到 [BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form). 在此我仅介绍足够我们识别语法所使用的 BNF 语法。

我们想要能表达整数的数学运算表达式。下面是该语法的 BNF 描述：

```
expression: number
          | expression '*' expression
          | expression '/' expression
          | expression '+' expression
          | expression '-' expression
          ;

number:  T_INTLIT
         ;
```

竖线分隔了语法中的选项，由此上述内容表达了：

  + 表达式可能仅仅是数字(number)，或
  + 表达式由 '*' token 分开的两个表达式组成，或
  + 表达式由 '/' token 分开的两个表达式组成，或
  + 表达式由 '+' token 分开的两个表达式组成，或
  + 表达式由 '-' token 分开的两个表达式组成
  + 数字(number) 总是一个 T_INTLIT token

很明显，BNF 定义的语法是 *递归(recursive)* 的：一个表达式通过引用别的表达式来定义。但是会有方法来 *终止(bottom-out)* 递归：当一个表达式是数字(number)，它总是一个 T_INTLIT token，从而不再递归。

在 BNF 中，我们说“表达式(expression)”和“数字(number)”是 *非终结(non-terminal)* 符号，因为它们是由语法规则生成的。然而，T_INTLIT 是一个 *终结(terminal)* 符号，因为它并非由任何规则定义，而是由语言中已识别的 token 来定义的。同样的，四个数学运算符的 token 也是终结符号。

## 递归下降解析

因为我们的语言是递归的，所以试着用递归的方式来解析它是合理的。我们需要做的是读入一个 token，再 *向前观察(look ahead)* 下一个 token。基于下一个 token，我们就能决定使用哪一条路径来解析输入内容。这可能要求我们递归调用一个已经调用了的函数。

在我们的情况下，任何表达式的第一个 token 都会是一个数字，然后可能会跟着数学运算符。在那之后可能是一个单独的数字，或者是另一个新的表达式的开始。我们如何递归的解析它呢？

我们可以编写如下的伪代码：

```
function expression() {
  扫描并确定第一个 token 是数字。如果不是数字，则错误
  获得下一个 token
  如果到达输入的结尾，则返回。此为最基本的情况（译注，仅有一个数字的情况）

  否则，调用 expression()（译注，此情况为刚刚获得的“下一个 token”是数学运算符，再后面应该跟着一个完整的表达式：一个数字，或者一个数字再加一个运算符后面跟着另一个表达式，一直递归到最后仅有一个数字）
}
```

让我们以 `2 + 3 - 5 T_EOF` 作为输入来运行这个函数。这里的 `T_EOF` 是表示输入结束的 token。我会给每次调用的 `expression()` 标上编号。

```
expression0:
  扫描得到 2, 它是一个数字
  获得下一个 token, +, 它不是 T_EOF
  调用 expression()

    expression1:
      扫描得到 3, 它是一个数字
      获得下一个 token, -, 它不是 T_EOF
      调用 expression()

        expression2:
          扫描得到 5, 它是一个数字
          获得下一个 token, T_EOF, 所以从 expression2 返回

      从 expression1 返回
  从 expression0 返回
```

是的，这个函数能够递归地解析 `2 + 3 - 5 T_EOF`。

当然，我们还没对输入做任何的事情，但是那并不是解析器的工作。解析器的工作仅仅是 *识别* 输入，并警告语法错误。别的功能会对输入进行 *语义分析(semantic analysis)*，也就是理解并执行输入的含义。

> 之后，你会看到这并不正确。很多情况下，将语法分析和语义分析纠缠在一起是合理的。

## 抽象语法树

做语义分析，我们要么解释已识别的输入，或者将它转换成另一种格式，比如汇编。在旅程的这一部分，我们会为输入创建一个解释器。要达到这一目标，我们首先要将输入转换成一个[抽象语法树(abstract syntax tree)](https://en.wikipedia.org/wiki/Abstract_syntax_tree)，也被称为 AST.

我强烈推荐你去读一读下面这篇关于 AST 的简短解释：

 + [Leveling Up One’s Parsing Game With ASTs](https://medium.com/basecs/leveling-up-ones-parsing-game-with-asts-d7a6fc2400ff) 作者 Vaidehi Joshi

它写得很好，非常有助于解释 AST 的目的和结构。别担心，你回来的时候我还是会在此处的。

我们要构造的 AST 中的每个节点的结构定义在 `defs.h` 中：

```c
// AST 节点的类型
enum {
  A_ADD, A_SUBTRACT, A_MULTIPLY, A_DIVIDE, A_INTLIT
};

// 抽象语法树的结构
struct ASTnode {
  int op;                               // 这一个树要执行的操作(Operation)
  struct ASTnode *left;                 // 左右子树
  struct ASTnode *right;
  int intvalue;                         // A_INTLIT 类型的整数值
};
```

某些 AST 节点，比如 `op` 值为 `A_ADD` 和 `A_SUBTRACT` 拥有两个子 AST，分别由 `left` 和 `right` 指向。稍后，我们会将这两个子树的值进行加法或减法运算。

另一种情况是，`op` 值为 A_INTLIT 的 AST 节点表示了一个整数值。它没有子树，仅有一个 `intvalue` 字段的值。

## 构建 AST 节点和树

`tree.c` 中的代码包含了构建 AST 的函数。最为通用的函数 `mkastnode()` 接受 AST 结构的四个字段值，分配节点，填充字段值，并返回该节点的指针。

```c
// 创建并返回一个通用的 AST 节点
struct ASTnode *mkastnode(int op, struct ASTnode *left,
                          struct ASTnode *right, int intvalue) {
  struct ASTnode *n;

  // 为新的 AST 节点分配内存
  n = (struct ASTnode *) malloc(sizeof(struct ASTnode));
  if (n == NULL) {
    fprintf(stderr, "Unable to malloc in mkastnode()\n");
    exit(1);
  }
  // 复制字段值并返回
  n->op = op;
  n->left = left;
  n->right = right;
  n->intvalue = intvalue;
  return (n);
}
```

基于此，我们可以编写用于创建 AST 叶节点的具体函数（也就是没有子节点的节点），并创建只有一个节点的 AST 节点。

```c
// 创建一个 AST 叶节点
struct ASTnode *mkastleaf(int op, int intvalue) {
  return (mkastnode(op, NULL, NULL, intvalue));
}

// 创建一个一元 AST 节点：仅有一个子节点
struct ASTnode *mkastunary(int op, struct ASTnode *left, int intvalue) {
  return (mkastnode(op, left, NULL, intvalue));
```

## AST 的目的

我们将用一个 AST 来存储每个识别出的表达式，之后我们就可以递归地遍历它来计算表达式的最终值。我们希望能处理数学运算符的优先级。下面上一个例子：

考虑表达式 `2 * 3 + 4 * 5`。现在，乘法的优先级高于加法。由此我们希望将乘法 *绑定(bind)* 在一起，在加法之前优先处理。

如果我们生成了这样的 AST 树：

```
          +
         / \
        /   \
       /     \
      *       *
     / \     / \
    2   3   4   5
```

然后在我们遍历树时，我们先处理 `2*3`，然后 `4*5`。一旦有了这些结果，我们就能把它们传递给树的根节点来进行加法运算。

## 一个简单的表达式解析器

现在，我们可以复用扫描器扫描到的 token 值，作为 AST 节点操作的数值，但是我喜欢把 token 和 AST 节点的概念分开。所以，首先我会做一个函数将 token 值映射为 AST 节点操作值。这部分，还有解析器的其它部分的在 `expr.c` 文件中：

```c
// 将 token 转换为 AST 操作
int arithop(int tok) {
  switch (tok) {
    case T_PLUS:
      return (A_ADD);
    case T_MINUS:
      return (A_SUBTRACT);
    case T_STAR:
      return (A_MULTIPLY);
    case T_SLASH:
      return (A_DIVIDE);
    default:
      fprintf(stderr, "unknown token in arithop() on line %d\n", Line);
      exit(1);
  }
}
```

switch 语句中的 default 语句用在我们无法将 token 转换成 AST 节点。它会是我们解析器中的语法检测部分。

我们需要一个确定下一个 token 是整数，并创建一个保存这个字面值的 AST 节点的函数。以下就是这个函数：

```c
// 解析一个主要元素，并返回表示它的 AST 节点
static struct ASTnode *primary(void) {
  struct ASTnode *n;

  // 对于 INTLIT token，创建一个叶节点，并扫描下一个 token.
  // 否则，对于其它任何标记，都产生语法错误。
  switch (Token.token) {
    case T_INTLIT:
      n = mkastleaf(A_INTLIT, Token.intvalue);
      scan(&Token);
      return (n);
    default:
      fprintf(stderr, "syntax error on line %d\n", Line);
      exit(1);
  }
}
```

此代码假设存在一个叫 `Token` 的全局变量，且其中已经存放着从输入内容中扫描到的最新 token。它定义在 `data.h`：

```c
extern_ struct token    Token;
```

在 `main()` 中：

```c
  scan(&Token);                 // 从输入中获得第一个 token
  n = binexpr();                // 解析文件中的表达式
```

现在我们可以编写解析器的代码了：

```c
// 返回一个根节点为二元操作的 AST 树
struct ASTnode *binexpr(void) {
  struct ASTnode *n, *left, *right;
  int nodetype;

  // 获得左树中的整数字面量
  // 并同时获得下一个 token
  left = primary();

  // 如果没有 token 了，则直接返回左侧节点
  if (Token.token == T_EOF)
    return (left);

  // 将 token转换为一个节点类型
  nodetype = arithop(Token.token);

  // 获得下一个 token
  scan(&Token);

  // 递归获得右侧的树
  right = binexpr();

  // 现在用两侧的子树根据一个树
  n = mkastnode(nodetype, left, right, 0);
  return (n);
}
```

注意，这个简单的解析器代码中没有任何处理不同操作符优先级的内容。在这样的情况下，代码将所有的操作符视作同一优先级。如果你按此代码解析解析了表达式  `2 * 3 + 4 * 5`，你会看到如下的 AST：

```
     *
    / \
   2   +
      / \
     3   *
        / \
       4   5
```

这显然是不正确的。它会先做 `4*5`，得到 20，然后再处理 `3+20` 得到 23，而非做 `2*3` 得到 6 的操作。

所以为什么我要这么做？我是想给你展示一下，实现一个简单的解析器很容易，但是让它同时处理语义分析则是困难的。

## 解析树

现在我们有了（不正确的）AST 树，让我们写一些代码来解析它。我们仍然会用递归的代码来遍历树。以下是伪代码：

```
interpretTree:
  首先，解析左子树，并获得它的值
  然后，解析右子树，并获得它的值
  对两个子树执行节点中的根所对应的操作，并返回它的值
```

回到正确的 AST 树：

```
          +
         / \
        /   \
       /     \
      *       *
     / \     / \
    2   3   4   5
```

调用的结构应该是这样的：

```
interpretTree0(+ 节点):
  调用 interpretTree1(左侧的 * 节点):
     调用 interpretTree2(数字 2 的节点):
       无数学操作，直接返回 2
     调用 interpretTree3(数字 3 的节点):
       无数学操作，直接返回 3
     执行 2 * 3, 返回 6

  调用 interpretTree1(右侧的 * 节点):
     调用 interpretTree2(数字 4 的节点):
       无数学操作，直接返回 4
     调用 interpretTree3(数字 5 的节点):
       无数学操作，直接返回 5
     执行 4 * 5, 返回 20

  执行 6 + 20, 返回 26
```

## 解析树的代码

代码存放于  `interp.c`中，并遵循上面的伪代码：

```c
// 对于一个 AST，解析它里面的操作，并返回一个最终值
int interpretAST(struct ASTnode *n) {
  int leftval, rightval;

  // 获得左、右子树的值
  if (n->left)
    leftval = interpretAST(n->left);
  if (n->right)
    rightval = interpretAST(n->right);

  switch (n->op) {
    case A_ADD:
      return (leftval + rightval);
    case A_SUBTRACT:
      return (leftval - rightval);
    case A_MULTIPLY:
      return (leftval * rightval);
    case A_DIVIDE:
      return (leftval / rightval);
    case A_INTLIT:
      return (n->intvalue);
    default:
      fprintf(stderr, "Unknown AST operator %d\n", n->op);
      exit(1);
  }
}
```

再说一次，switch 语句中的 default 语句块在遇到无法解析 AST 结点时执行。它会被用作我们的解析器的语法检测。

## 构建解析器

此外还有一些别的代码，比如在 `main()` 中调用解析器的部分：

```c
  scan(&Token);                 // 从输入中获得第一个 token
  n = binexpr();                // 解析文件中的表达式
  printf("%d\n", interpretAST(n));      // 计算最终结果
  exit(0);
```

现在你可以用下面的命令构建解析器：

```
$ make
cc -o parser -g expr.c interp.c main.c scan.c tree.c
```

我提供了一些文件以供你测试解析器，不过当然你也可以创造你自己的测试文件。记住，计算的结果是不正确的，但是解析器应该能够检测出一此错误，比如连续数字、连续操作符或者输入的结尾缺少数字。我还在解释器中加入了一些调试代码，这样你就能看到 AST 树节点是以什么顺序计算的：

```
$ cat input01
2 + 3 * 5 - 8 / 3

$ ./parser input01
int 2
int 3
int 5
int 8
int 3
8 / 3
5 - 2
3 * 3
2 + 9
11

$ cat input02
13 -6+  4*
5
       +
08 / 3

$ ./parser input02
int 13
int 6
int 4
int 5
int 8
int 3
8 / 3
5 + 2
4 * 7
6 + 28
13 - 34
-21

$ cat input03
12 34 + -56 * / - - 8 + * 2

$ ./parser input03
unknown token in arithop() on line 1

$ cat input04
23 +
18 -
45.6 * 2
/ 18

$ ./parser input04
Unrecognised character . on line 3

$ cat input05
23 * 456abcdefg

$ ./parser input05
Unrecognised character a on line 1
```

## 结论及下一步

解析器能识别语言的语法并检查输入至编译器的是否符合语法。如果不符合，解析器应打印出错误信息。因为我们的表达式语法是递归的，所以我们选择编写一个递归解析器来识别我们的表达式。

现在，如上面所展示的那样，解析器可以正常工作了，但是它未能正确的获得输入的语义。换句话说，它并未能正确的计算表达式的值。

在我们的编译器旅程的下一步中，我们会修改解析器，让它也能对表达式进行正确的语义分析以计算出正确的数学结果。[Next step](../03_Precedence/Readme.md)
