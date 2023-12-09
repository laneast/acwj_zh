# 第 1 部分：词法扫描的介绍

我们的编译器制作旅程由一个简单的词法扫描器开始。如前部分说的，词法扫描器的工作是标记输入语言的词法元素，或者说是 *token*.

接下来我们由一个仅支持五个词法元素的语言开始：

 + 四个基础数学操作符：`*`, `/`, `+` and `-`
 + 有着1个或者更多数字 `0` .. `9` 的十进制整数

我们扫描的每个 `token` 存在以下结构中（定义在 `defs.h`）：

```c
// Token structure
struct token {
  int token;
  int intvalue;
};
```

其中的 `token` 字段可以取以下值之一（定义在 `defs.h`）：

```c
// Tokens
enum {
  T_PLUS, T_MINUS, T_STAR, T_SLASH, T_INTLIT
};
```

当 token 是 `T_INTLIT`(整型字面量 integer literal) 时，`intvalue 中存放着我们扫描到的整数值。`

## `scan.c` 中的函数

`scan.c` 文件存放着我们的词法扫描器的函数。我们会一次一个字符地读取输入文件。然而，有时候我们会读入过多，此时还需要“放回”一个字符。我们还要记录我们当前已经读到哪一行了，这样我们才能在调试信息中打印出对应的行号。这一切都是由 `next()` 函数来完成：

```c
// 从输入文件中读入下一个字符。
static int next(void) {
  int c;

  if (Putback) {                // 如果存在“放回”的字符，
    c = Putback;                // 则使用该字符。
    Putback = 0;
    return c;
  }

  c = fgetc(Infile);            // 从输入文件中读取
  if ('\n' == c)
    Line++;                     // 增加行号
  return c;
}
```

`Putback` 和 `Line` 变量还有我们的输入文件指针在 `data.h` 定义。

```c
extern_ int     Line;
extern_ int     Putback;
extern_ FILE    *Infile;
```

所有的 C 文件在引入此文件的时候，都会把 `extern_` 替换成 `extern`，但是 `main.c` 是例外，它会移除 `extern_`；这样，这些变量就“属于” `main.c` 了。

最终，我们如何放回一个字符到输入流中呢？这样：

```c
// 放回一个不想要的字符
static void putback(int c) {
  Putback = c;
}
```

## 忽略空白符

我们需要做一个函数来安静地忽略空白符，直到它读入了一个非空白字符，然后将这个字符返回。这样：

```c
// 忽略我们无需处理的输入，比如空白字符，换行字符。
// 返回第一个我们需要处理的字符。
static int skip(void) {
  int c;

  c = next();
  while (' ' == c || '\t' == c || '\n' == c || '\r' == c || '\f' == c) {
    c = next();
  }
  return (c);
}
```

## 扫描 token：`scan()`

现在我们能忽略空白并读取到字符；在多读了一个字符时也能把它放回去。可以开始编写我们第一个词法扫描器了：

```c
// 读取并返回输入内容的第一个 token
// 如果读入的 token 有效，返回 1, 如果没有 token 则返回 0
int scan(struct token *t) {
  int c;

  // 忽略空白
  c = skip();

  // 根据输入字符确定 token 类型
  switch (c) {
  case EOF:
    return (0);
  case '+':
    t->token = T_PLUS;
    break;
  case '-':
    t->token = T_MINUS;
    break;
  case '*':
    t->token = T_STAR;
    break;
  case '/':
    t->token = T_SLASH;
    break;
  default:
    // 更多类型待添加
  }

  // 找到了一个 token
  return (1);
}
```

这就是简单的、针对单字符的 token 的处理了：对每个识别到的字符，转换为 token。你可能要问：为什么不直接把识别的字符放到 `struct token` 中去呢？答案是在以后，我们要识别多字符 token，比如 `==` 还有 `if` 和 `while` 这样的关键字。用枚举列表来表示 token 类型在未来会更容易一些。

## 整型字面量

实际上，我们已经需要面临对这样的情况了，因为我们还需要识别像 `3827`, `87731` 这样的整型字面量。下面是 `switch` 语句中缺失的 `default` 代码：

```c
  default:

    // 如果是数字，就把整个整型字面量扫描进来
    if (isdigit(c)) {
      t->intvalue = scanint(c);
      t->token = T_INTLIT;
      break;
    }

    printf("Unrecognised character %c on line %d\n", c, Line);
    exit(1);
```

一旦我们遇到一个十进制数字符，我们就调用 `scanint()` 辅助函数，并将第一个字符传给它。它会返回扫描到的整型数值。为了做到这一点，它必须每次读入一个字符，检查是不是合法的数字，并构建出最终的数字。以下是代码：

```c
// 从输入文件中扫描并返回一个整型字面量。
static int scanint(int c) {
  int k, val = 0;

  // 将每个字符转换为 int 值
  while ((k = chrpos("0123456789", c)) >= 0) {
    val = val * 10 + k;
    c = next();
  }

  // 遇到一个非整型字符，把它放回去
  putback(c);
  return val;
}
```

我们从 0 值的 `val` 开始。每获得一个在 `0` 到 `9` 之间的数字，我们就用 `chrpos()` 把它转换成一个 `int` 值。我们把之前的 `var` 变大至 10 倍，并把新的数字加进去。

例如，假设我们有这样的字符：`3`, `2`, `8`, 我们这样做：

 + `val= 0 * 10 + 3`, 即 3
 + `val= 3 * 10 + 2`, 即 32
 + `val= 32 * 10 + 8`, 即 328

在最后，你发现我们调用了 `putback(c)` 了吗？我们遇到了一个非十进制数字，我们不能简单地丢弃它，不过好在我们能把它放回转入流以便之后使用。

你可能要问：为什么不简单地把 `c` 减去 '0' 的 ASCII 值呢？答案是，之后我们可以用 `chrpos("0123456789abcdef")` 来对十六进制数字进行转换。


下面是 `chrpos()` 的代码：

```c
// 返回字符 c 在字符串 s 中的位置，找不到 c 的话就返回 -1
static int chrpos(char *s, int c) {
  char *p;

  p = strchr(s, c);
  return (p ? p - s : -1);
}
```

这就是目前 `scan.c` 中词法扫描器的代码了。

## 让扫描器开始工作

在 `main.c` 中的代码将扫描器投入使用。`main()` 函数打开一个文件，并扫描其中的 token：

```c
void main(int argc, char *argv[]) {
  ...
  init();
  ...
  Infile = fopen(argv[1], "r");
  ...
  scanfile();
  exit(0);
}
```

`scanfile()` 在存在新的 token 时循环，并打印出 token 的详细信息：

```c
// 可打印的 token 列表
char *tokstr[] = { "+", "-", "*", "/", "intlit" };

// 循环扫描输入文件中的所有 token。
// 打印找到的每个 token 的详细信息。
static void scanfile() {
  struct token T;

  while (scan(&T)) {
    printf("Token %s", tokstr[T.token]);
    if (T.token == T_INTLIT)
      printf(", value %d", T.intvalue);
    printf("\n");
  }
}
```

## 一些示例输入文件

我提供了一些示例输入文件，你可以看看扫描器能从中找到什么 token，还有扫描器拒绝了哪些输入文件。

```
$ make
cc -o scanner -g main.c scan.c

$ cat input01
2 + 3 * 5 - 8 / 3

$ ./scanner input01
Token intlit, value 2
Token +
Token intlit, value 3
Token *
Token intlit, value 5
Token -
Token intlit, value 8
Token /
Token intlit, value 3

$ cat input04
23 +
18 -
45.6 * 2
/ 18

$ ./scanner input04
Token intlit, value 23
Token +
Token intlit, value 18
Token -
Token intlit, value 45
Unrecognised character . on line 3
```

## 结论及下一步

我们完成了一个简单的能识别四个数学运算符和整型字面量的词法扫描器。我们注意到我们需要忽略空白字符，并在读入过多时放回一些字符。

单字符扫描起来很简单，但是多字符 token 就有点难了。但是最终，`scan()` 函数从输入文件中将下一个 token 以 `struct token` 值的形式返回：

```c
struct token {
  int token;
  int intvalue;
};
```

在我们编译器旅程的下一部分中，我们会构建一个递归下降(recursive descent)分析器来解析输入文件的语法，计算并输出每个文件的最终值。[Next step](../02_Parser/Readme.md)
