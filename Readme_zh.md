# 编译器开发之旅

在此 Github 库中，我将记录我开发可编译自身的 C 语言子集编译器的旅程。我会写下详细过程，这样在你希望跟进时可以了解到我做了什么，还能了解一些相关的编译器理论。

理论不会太多，因为我希望这是一场实践性的旅程。

以下是到目前为止我完成的工作：

 + [第  0 部分](00_Introduction/Readme_zh.md):  旅程的介绍
 + [第  1 部分](01_Scanner/Readme.md):          词法扫描的介绍
 + [第  2 部分](02_Parser/Readme.md):           解析的介绍
 + [Part  3](03_Precedence/Readme.md):    Operator Precedence
 + [Part  4](04_Assembly/Readme.md):      An Actual Compiler
 + [Part  5](05_Statements/Readme.md):    Statements
 + [Part  6](06_Variables/Readme.md):     Variables
 + [Part  7](07_Comparisons/Readme.md):   Comparison Operators
 + [Part  8](08_If_Statements/Readme.md): If Statements
 + [Part  9](09_While_Loops/Readme.md):   While Loops
 + [Part 10](10_For_Loops/Readme.md):     For Loops
 + [Part 11](11_Functions_pt1/Readme.md): Functions, part 1
 + [Part 12](12_Types_pt1/Readme.md):     Types, part 1
 + [Part 13](13_Functions_pt2/Readme.md): Functions, part 2
 + [Part 14](14_ARM_Platform/Readme.md):  Generating ARM Assembly Code
 + [Part 15](15_Pointers_pt1/Readme.md):  Pointers, part 1
 + [Part 16](16_Global_Vars/Readme.md):   Declaring Global Variables Properly
 + [Part 17](17_Scaling_Offsets/Readme.md): Better Type Checking and Pointer Offsets
 + [Part 18](18_Lvalues_Revisited/Readme.md): Lvalues and Rvalues Revisited
 + [Part 19](19_Arrays_pt1/Readme.md):    Arrays, part 1
 + [Part 20](20_Char_Str_Literals/Readme.md): Character and String Literals
 + [Part 21](21_More_Operators/Readme.md): More Operators
 + [Part 22](22_Design_Locals/Readme.md): Design Ideas for Local Variables and Function Calls
 + [Part 23](23_Local_Variables/Readme.md): Local Variables
 + [Part 24](24_Function_Params/Readme.md): Function Parameters
 + [Part 25](25_Function_Arguments/Readme.md): Function Calls and Arguments
 + [Part 26](26_Prototypes/Readme.md):    Function Prototypes
 + [Part 27](27_Testing_Errors/Readme.md): Regression Testing and a Nice Surprise
 + [Part 28](28_Runtime_Flags/Readme.md): Adding More Run-time Flags
 + [Part 29](29_Refactoring/Readme.md):   A Bit of Refactoring
 + [Part 30](30_Design_Composites/Readme.md): Designing Structs, Unions and Enums
 + [Part 31](31_Struct_Declarations/Readme.md): Implementing Structs, Part 1
 + [Part 32](32_Struct_Access_pt1/Readme.md): Accessing Members in a Struct
 + [Part 33](33_Unions/Readme.md):        Implementing Unions and Member Access
 + [Part 34](34_Enums_and_Typedefs/Readme.md): Enums and Typedefs
 + [Part 35](35_Preprocessor/Readme.md):  The C Pre-Processor
 + [Part 36](36_Break_Continue/Readme.md): `break` and `continue`
 + [Part 37](37_Switch/Readme.md):        Switch Statements
 + [Part 38](38_Dangling_Else/Readme.md): Dangling Else and More
 + [Part 39](39_Var_Initialisation_pt1/Readme.md): Variable Initialisation, part 1
 + [Part 40](40_Var_Initialisation_pt2/Readme.md): Global Variable Initialisation
 + [Part 41](41_Local_Var_Init/Readme.md): Local Variable Initialisation
 + [Part 42](42_Casting/Readme.md):       Type Casting and NULL
 + [Part 43](43_More_Operators/Readme.md): Bugfixes and More Operators
 + [Part 44](44_Fold_Optimisation/Readme.md): Constant Folding
 + [Part 45](45_Globals_Again/Readme.md): Global Variable Declarations, revisited
 + [Part 46](46_Void_Functions/Readme.md): Void Function Parameters and Scanning Changes
 + [Part 47](47_Sizeof/Readme.md):        A Subset of `sizeof`
 + [Part 48](48_Static/Readme.md):        A Subset of `static`
 + [Part 49](49_Ternary/Readme.md):       The Ternary Operator
 + [Part 50](50_Mop_up_pt1/Readme.md):    Mopping Up, part 1
 + [Part 51](51_Arrays_pt2/Readme.md):    Arrays, part 2
 + [Part 52](52_Pointers_pt2/Readme.md):  Pointers, part 2
 + [Part 53](53_Mop_up_pt2/Readme.md):    Mopping Up, part 2
 + [Part 54](54_Reg_Spills/Readme.md):    Spilling Registers
 + [Part 55](55_Lazy_Evaluation/Readme.md): Lazy Evaluation
 + [Part 56](56_Local_Arrays/Readme.md):  Local Arrays
 + [Part 57](57_Mop_up_pt3/Readme.md):    Mopping Up, part 3
 + [Part 58](58_Ptr_Increments/Readme.md): Fixing Pointer Increments/Decrements
 + [Part 59](59_WDIW_pt1/Readme.md):      Why Doesn't It Work, part 1
 + [Part 60](60_TripleTest/Readme.md):    Passing the Triple Test
 + [Part 61](61_What_Next/Readme.md):     What's Next?
 + [Part 62](62_Cleanup/Readme.md):       Code Cleanup
 + [Part 63](63_QBE/Readme.md):           A New Backend using QBE

后续不定期更新，随时过来看看是否有更新。

## 版权

译者注：本章节是为了方便阅读而做的中文翻译，但是实际版权以附在下方的[英文版权](#Copyrights)为准；本翻译采用和英文原作相同的授权方式。

我从 Nils M Holm 写的 [SubC](http://www.t3x.org/subc/) 编译器中借鉴了一些代码和许多想法，他的代码处于公有领域(public domain)，我觉得我的代码与之有足够的不同，所以我可以为我的代码使用不同的授权。

除非特别备注，

 + 所有代码为 (c) Warren Toomey 所有，并采用 GPL3 授权(GPL3 license)。
 + 所有非代码类的文档（比如英文文档、图像文件）为 (c) Warren Toomey 所有，并采用署名-非商业性使用-相同方式共享 4.0 授权(Creative Commons BY-NC-SA 4.0 license)。

## Copyrights

I have borrowed some of the code, and lots of ideas, from the 
[SubC](http://www.t3x.org/subc/) compiler written by Nils M Holm.
His code is in the public domain. I think that my code is substantially
different enough that I can apply a different license to my code.

Unless otherwise noted,

 + all source code and scripts are (c) Warren Toomey under
   the GPL3 license.
 + all non-source code documents (e.g. English documents,
   image files) are (c) Warren Toomey under the Creative
   Commons BY-NC-SA 4.0 license.
