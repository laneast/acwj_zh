# 第 0 部分：旅程的介绍

我决定开始一场编译器制作的旅程。在过去，我写过[汇编器（assemblers）](https://github.com/DoctorWkt/pdp7-unix/blob/master/tools/as7)，和一个无类型语言的[简单编译器（simple compiler）](https://github.com/DoctorWkt/h-compiler)。但从未写过一个能够编译自身的编译器，所以我决定开始这次旅程。

在此过程中，我会将我的工作记录下来，以便其他人可以跟进。这也有助于我理清自己的思路和想法。希望这对你我都有所帮助。

## 旅程的目标

下面是我此次旅程的目标：

 + 编写一个能编译自身的编译器。我觉得只有能够编译自身，那才能称为一个 *真正* 的编译器。
 + 要在至少一个真正的硬件平台上运行。我见过一些针对假想机器的编译器，我希望我的编译器能在真正的硬件上运行。如果可能的话，我希望它还能支持多个后端，来支持不同的硬件平台。
 + 实践优于研究。在编译器领域已经有大量的研究，我希望进行一场从零开始的旅程，所以我会倾向于实践而非偏重理论。即便如此，我有时也会引入（并实现）一些基于理论的东西。
 + 遵循 KISS 原则：保持简单。我会坚持使用 Ken Thompson 的原则：“如有疑惑，蛮力破之！”
 + 积跬步以至千里。我会将整个旅程分解为大量简单步骤而非大步跃进。这会让每次新增内容小巧而易于理解。

## 目标语言

选择目标语言是件困难的事情。如果我选择了诸如 Python, Go 等高级语言，就意味着我要实现一大堆类与库，因为它们是这个语言内置的部分。

我也可以实现一个如 Lisp 之类的语言，但是这又[很容易实现](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-039.pdf)。

所以我回到旧的备选，实现 C 语言的一个能编译自身的子集。

C 语言是汇编语言的升级（这一说法适用于部分 C 语言的子集，而非 [C18](https://en.wikipedia.org/wiki/C18_(C_standard_revision))），这也会使 C 语言转换为汇编语言的任务更简单。嗯，还因为我喜欢 C 语言。

## 编译器的基础工作

编译器的工作是将一种语言（通常是高级语言）的输入转换成不同的输出语言（通常相比输入来说低级一些）。主要步骤有：

![](Figs/parsing_steps.png)

 + 进行[词法分析（lexical analysis）](https://en.wikipedia.org/wiki/Lexical_analysis)来识别词法元素。在一些语言中，`=` 与 `==` 是不同的，所以不能只读取单一的 `=`。我们把这些词法元素叫做 *token*。
 + [解析（Parse）](https://en.wikipedia.org/wiki/Parsing)输入，比如识别输入的语法和结构，确保符合语言的 *语法(grammar)*。比如你的语言可能有如下的分支结构：

```
      if (x < 23) {
        print("x is smaller than 23\n");
      }
```

> 但是在别的语言中你可能会写成：

```
      if (x < 23):
        print("x is smaller than 23\n")
```

> 这也是编译器检查语法错误的地方，比如在上面的 *print* 语句末尾是否丢失了分号。

 + 对输入进行[语义分析（semantic analysis）](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))，搞懂输入的含义。这和识别语法与结构不同。举个例子，在英语中，句子由 `<主语> <动词> <形容词> <宾语>` 组成。下面两个句子有着相同的结构，但是完全不同的含义：

```
          David ate lovely bananas.
          Jennifer hates green tomatoes.
```

 + 将输入[转换（Translate）](https://en.wikipedia.org/wiki/Code_generation_(compiler))成另一种语言。在此我们把输入逐步地转换成低级语言。

## 资源

互联网上有很多的编译器资源，下面是我会参考的那些。

### 学习资源

如果你想从一些关于编译器的书本、论文、工具开始的话，我强烈推荐以下列表：

  + [Curated list of awesome resources on Compilers, Interpreters and Runtimes](https://github.com/aalhour/awesome-compilers) by Ahmad Alhour

### 现有的编译器

While I'm going to build my own compiler, I plan on looking at other compilers
for ideas and probably also borrow some of their code. Here are the ones
I'm looking at:
尽管我是在自己做编译器，但我仍计划从别的编译器中获取灵感，可能还会借鉴一些他们的代码。下面是我参考的编译器。

  + [SubC](http://www.t3x.org/subc/) 作者是 Nils M Holm
  + [Swieros C Compiler](https://github.com/rswier/swieros/blob/master/root/bin/c.c) 作者 Robert Swierczek
  + [fbcc](https://github.com/DoctorWkt/fbcc) 作者 Fabrice Bellard
  + [tcc](https://bellard.org/tcc/) 作者同样有 Fabrice Bellard, 还有一些其他人
  + [catc](https://github.com/yui0/catc) 作者 Yuichiro Nakada
  + [amacc](https://github.com/jserv/amacc) 作者 Jim Huang
  + [Small C](https://en.wikipedia.org/wiki/Small-C) 作者 Ron Cain, James E. Hendrix 并被其他人进行衍生。

具体地说，我会使用 SubC 编译器的大量点子和部分代码。

## 搭建开发环境

如果你想要跟着本次旅程一起来，下面是你需要的东西。我会使用 Linux 开发环境，所以下载并设置好你喜欢的 Linux 系统，我使用的是 Lubuntu 18.04。

我的目标硬件平台有两个：Intel x86-64 和 32位的 ARM。我会使用一台运行着 Lubuntu 18.04 的电脑作为 Intel 目标硬件平台，并使用运行着 Raspbian 的树莓派作为 ARM 目标硬件平台。

在 Intel 平台上，我们需要已存在的 C 编译器，所以我们要先安装它（下面是 Ubuntu/Debian 的安装命令）：

```
  $ sudo apt-get install build-essential
```

如果在 vanilla Linux 系统上还需要别的工具的话，请告诉我。

最后，克隆此 Github 库。

## 下一步

在本编译器编写旅程的下一步中，我们会编写代码，来扫描输入文件，找到我们语言的词法元素，也就是 *token*. [Next step](../01_Scanner/Readme.md)
