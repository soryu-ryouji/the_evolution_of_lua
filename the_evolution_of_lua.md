# The Evolution of Lua

| Roberto Ierusalimschy                                           | Luiz Henrique de Figueiredo                                    | Waldemar Celes                                                  |
| --------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------- |
| Department of Computer Science, PUC-Rio, Rio de Janeiro, Brazil | IMPA–Instituto Nacional de Matem´atica Pura e Aplicada, Brazil | Department of Computer Science, PUC-Rio, Rio de Janeiro, Brazil |
| [roberto@inf.puc-rio.br]                                        | [lhf@impa.br]                                                  | [celes@inf.puc-rio.br]                                          |

## Abstract

我们报告了 Lua 语言的诞生和演变，并讨论了它如何从一个简单的配置语言发展成为一种多功能、广泛使用的语言，支持可扩展的语义、匿名函数、完整的词法作用域、适当的尾调用以及协程。

分类与主题描述符 K.2 [计算机历史]: 软件；D.3 [编程语言]

## 1. Introduction

Lua 是一种脚本语言，于 1993 年诞生于巴西里约热内卢的天主教大学（PUC-Rio）。自那时起，Lua 不断发展，现已被广泛应用于各种工业领域，如机器人技术、文学编程、分布式业务、图像处理、可扩展文本编辑器、以太网交换机、生物信息学、有限元分析软件包、网络开发等 [^2]。特别是在游戏开发领域，Lua 已成为领先的脚本语言之一。

Lua 的发展远远超出了我们最乐观的预期。事实上，尽管几乎所有的编程语言都源自北美和西欧（除了来自日本的 Ruby 这一显著例外）[^4]，Lua 是唯一一个在发展中国家创建并取得全球影响力的语言。

从一开始，Lua 就被设计为简单、小巧、可移植、快速且易于嵌入应用程序中。这些设计原则至今仍然有效，我们相信它们正是 Lua 在工业领域取得成功的原因。Lua 的主要特点及其简洁性的生动体现，在于它提供了一种单一的数据结构 —— 表（table），这是 Lua 对关联数组的称呼 [^9]。尽管大多数脚本语言都提供关联数组，但在其他语言中，关联数组并未扮演如此核心的角色。Lua 的表为模块、基于原型的对象、基于类的对象、记录、数组、集合、包、列表以及许多其他数据结构提供了简单而高效的实现 [^28]。

在本文中，我们报告了 Lua 的诞生与演变。我们探讨了 Lua 如何从一个简单的配置语言发展成为一种强大（但仍保持简洁）的语言，支持可扩展的语义、匿名函数、完整的词法作用域、适当的尾调用以及协程。在 §2 中，我们概述了 Lua 的主要概念，这些概念将在其他章节中用于讨论 Lua 的演变过程。在 §3 中，我们回顾了 Lua 的 “史前史”，即导致其诞生的背景。在 §4 中，我们讲述了 Lua 的诞生过程、其最初的设计目标以及第一个版本的功能。 §5 讨论了 Lua 如何以及为何演变，而 §6 则对部分特性的演变进行了详细探讨。本文在 §7 中以对 Lua 演变的回顾作为结尾，并在 §8 中简要讨论了 Lua 成功的原因，尤其是在游戏领域中的成功。

## 2. Overview

在本节中，我们简要概述了 Lua 语言，并介绍了在 §5 和 §6 中讨论的概念。关于 Lua 的完整定义，请参阅其参考手册 [^32]。如需详细了解 Lua，请参阅 Roberto 的书籍 [^28]。为了具体化，我们将描述 Lua 5.1，这是本文撰写时（2007 年 4 月）的最新版本，但本节的大部分内容同样适用于之前的版本。

从语法上看，Lua 与 Modula 相似，并使用常见的关键字。为了展示 Lua 的语法，以下代码展示了两种实现阶乘函数的方式，一种是递归实现，另一种是迭代实现。任何具备基本编程知识的人可能无需解释就能理解这些示例。

```lua
-- 递归实现
function factorial (n)
  if n == 0 then
    return 1
  else
    return n * factorial (n - 1)
  end
end

-- 迭代实现
function factorial (n)
  local result = 1
  for i = 2, n do
    result = result * i
  end
  return result
end
```

从语义上看，Lua 与 Scheme 有许多相似之处，尽管这些相似性并不显而易见，因为两种语言在语法上差异很大。在 Lua 的演变过程中，Scheme 对其影响逐渐增强：最初，Scheme 只是背景中的一种语言，但后来它逐渐成为重要的灵感来源，尤其是在引入匿名函数和完整的词法作用域之后。

与 Scheme 类似，Lua 是动态类型的：变量没有类型，只有值有类型。与 Scheme 一样，Lua 中的变量从不包含结构化值，而只是对它们的引用。与 Scheme 一样，函数名在 Lua 中没有特殊地位：它只是一个普通的变量，恰好引用了一个函数值。实际上，上面使用的函数定义语法 `function foo ()・・・end` 只是将匿名函数赋值给变量的语法糖：`foo = function ()・・・end`。与 Scheme 一样，Lua 具有词法作用域的一等函数。实际上，Lua 中的所有值都是一等值：它们可以赋值给全局变量和局部变量，存储在表中，作为参数传递给函数，并从函数中返回。

Lua 与 Scheme 之间的一个重要语义差异 —— 可能也是 Lua 的主要区别特征 —— 是 Lua 将表（table）作为其唯一的数据结构机制。Lua 的表是关联数组 [^9]，但具有一些重要特性。与 Lua 中的所有值一样，表是一等值：它们不像 Awk 和 Perl 中那样绑定到特定的变量名。表可以使用任何值作为键，并可以存储任何值。表通过使用字段名作为键，可以简单高效地实现记录；通过使用集合元素作为键，可以实现集合；还可以实现通用链表结构以及许多其他数据结构。此外，我们可以通过使用自然数作为索引，用表来实现数组。精心设计的实现 [^31] 确保这样的表与数组占用相同的内存（因为它在内部表示为实际的数组），并且在独立基准测试中表现优于类似语言中的数组 [^1]。

Lua 提供了一种表达力丰富的语法来创建表，即构造函数。最简单的构造函数是表达式 {}，它会创建一个新的空表。还有一些构造函数用于创建列表（或数组），例如：`{"Sun","Mon","Tue","Wed","Thu","Fri","Sat"}` 以及用于创建记录，例如：`{lat= -22.90, long= -43.23, city="Rio de Janeiro"}`。这两种形式可以自由混合使用。表使用方括号进行索引，例如 `t [2]`，而 `t.x` 则是 `t ["x"]` 的语法糖。

表构造函数与函数的结合使 Lua 成为一种强大的通用过程式数据描述语言。例如，一个类似于 BibTEX [^34] 格式的文献数据库可以写成一系列表构造函数，如下所示：

```lua
article {"spe96"
  authors = { "Roberto Ierusalimschy",
              "Luiz Henrique de Figueiredo",
              "Waldemar Celes" },
  title = "Lua: an Extensible Extension Language",
  journal = "Software: Practics & Experience",
  year = 1996
}
```

尽管这样的数据库看起来是一个静态的数据文件，但它实际上是一个有效的 Lua 程序：当数据库被加载到 Lua 中时，其中的每一项都会调用一个函数，因为 `article {・・・}` 是 `article ({・・・})` 的语法糖，即一个以表为唯一参数的函数调用。正是在这个意义上，这类文件被称为过程式数据文件（procedural data files）。

我们说 Lua 是一种可扩展的扩展语言 [^30]。它是一种扩展语言，因为它通过配置、宏和其他终端用户定制来帮助扩展应用程序。Lua 被设计为嵌入到宿主应用程序中，以便用户可以通过编写 Lua 程序来控制应用程序的行为，这些程序可以访问应用程序服务并操作应用程序数据。它是可扩展的，因为它提供了用户数据（userdata）值来保存应用程序数据，并提供了可扩展的语义机制以自然的方式操作这些值。Lua 作为一个小的核心提供，可以通过用 Lua 和 C 编写的用户函数进行扩展。特别是，输入输出、字符串操作、数学函数和操作系统接口都作为外部库提供。

Lua 的其他显著特征来自其实现：

**可移植性：** Lua 易于构建，因为它是用严格的 ANSI C 实现的。它在大多数平台（如 Linux、Unix、Windows、Mac OS X 等）上可以开箱即用，并且在我们所知的所有平台上（包括移动设备，如手持计算机和手机，以及嵌入式微处理器，如 ARM 和 Rabbit）只需进行少量调整即可运行。为了确保可移植性，我们努力在尽可能多的编译器下实现无警告编译。

**易于嵌入：** Lua 的设计目标是易于嵌入到应用程序中。Lua 的一个重要部分是一个定义良好的应用程序编程接口（API），它允许 Lua 代码与外部代码之间进行完全通信。特别是，通过从宿主应用程序导出 C 函数来扩展 Lua 非常容易。该 API 不仅允许 Lua 与 C 和 C++ 接口，还可以与其他语言（如 Fortran、Java、Smalltalk、Ada、C#（.Net））甚至其他脚本语言（如 Perl 和 Ruby）进行交互。

**小巧的体积：** 将 Lua 添加到应用程序中不会使其变得臃肿。整个 Lua 发行版，包括源代码、文档和一些平台的二进制文件，始终可以轻松地存放在一张软盘上。Lua 5.1 的压缩包（包含源代码、文档和示例）压缩后为 208K，解压后为 835K。源代码包含约 17,000 行 C 代码。在 Linux 下，使用所有标准 Lua 库构建的 Lua 解释器大小为 143K。大多数其他脚本语言的相应数字要大一个数量级以上，部分原因是 Lua 主要设计为嵌入应用程序中，因此其官方发行版仅包含少量库。而其他脚本语言则设计为独立使用，并包含许多库。

**高效性：** 独立基准测试 [^1] 显示，Lua 是解释型脚本语言领域中最快的语言之一。这使得应用程序开发者可以用 Lua 编写整个应用程序的相当大一部分。例如，Adobe Lightroom 中有超过 40% 的代码是用 Lua 编写的（约 10 万行 Lua 代码）。

尽管这些是特定实现的特点，但它们之所以可能，完全归功于 Lua 的设计。特别是，Lua 的简洁性是实现小巧高效的关键因素 [^31]。

## 3. Prehistory

## 4. Birth

## 5. History

## 6. Feature evolution

## 7. Retrospect

## 8. Conclusion

## Acknowledgments

## References

[^1]: The computer language shootout benchmarks. [http://shootout.alioth.debian.org/].

[^2]: Lua projects. [http://www.lua.org/uses.html].

[^3]: The MIT license. [http://www.opensource.org/licenses/mit-license.html].

[^4]: Timeline of programming languages. [http://en.wikipedia.org/wiki/Timeline_of_programming_languages].

[^5]: Which language do you use for scripting in your gameengine? [http://www.gamedev.net/gdpolls/viewpoll.asp?ID=163], Sept. 2003.

[^6]: Which is your favorite embeddable scripting language? [http://www.gamedev.net/gdpolls/viewpoll.asp?ID=788], June 2006.

[^7]: K. Beck. Extreme Programming Explained: Embrace Change. Addison-Wesley, 2000.

[^8]: G. Bell, R. Carey, and C. Marrin.
The Virtual Reality Modeling Language Specification—Version 2.0. [http://www.vrml.org/VRML2.0/FINAL/], Aug. 1996. (ISO/IEC CD 14772).

[^9]: J. Bentley. Programming pearls: associative arrays. Communications of the ACM, 28 (6):570–576, 1985.

[^10]: J. Bentley. Programming pearls: little languages. Communications of the ACM, 29 (8):711–721, 1986.

[^11]: C. Bruggeman, O. Waddell, and R. K. Dybvig. Representing
control in the presence of one-shot continuations. In SIGPLAN Conference on Programming Language Design and Implementation, pages 99–107, 1996.

[^12]:  W. Celes, L. H. de Figueiredo, and M. Gattass. EDG: umaferramenta para criac¸˜ao de interfaces gr´aficas interativas. In Proceedings of SIBGRAPI ’95 (Brazilian Symposium on Computer Graphics and Image Processing), pages 241–248, 1995.

[^13]: B. Davis, A. Beatty, K. Casey, D. Gregg, and J. Waldron. The case for virtual register machines. In Proceedings of the 2003 Workshop on Interpreters, Virtual Machines and Emulators, pages 41–49. ACM Press, 2003.

[^14]: L. H. de Figueiredo, W. Celes, and R. Ierusalimschy. Programming advanced control mechanisms with Lua coroutines. In Game Programming Gems 6, pages 357–369. Charles River Media, 2006.

[^15]: L. H. de Figueiredo, R. Ierusalimschy, and W. Celes. The design and implementation of a language for extending applications. In Proceedings of XXI SEMISH (Brazilian Seminar on Software and Hardware), pages 273–284, 1994.

[^16]: L. H. de Figueiredo, R. Ierusalimschy, and W. Celes. Lua: an extensible embedded language. Dr. Dobb’s Journal, 21 (12):26–33, Dec. 1996.

[^17]: L. H. de Figueiredo, C. S. Souza, M. Gattass, and L. C. G. Coelho. Gerac¸˜ao de interfaces para captura de dados sobre desenhos.In Proceedings of SIBGRAPI ’92 (Brazilian Symposium on Computer Graphics and Image Processing), pages 169–175, 1992.

[^18]: A. de Moura, N. Rodriguez, and R. Ierusalimschy. Coroutines in Lua. Journal of Universal Computer Science, 10 (7):910–925, 2004.

[^19]: A. L. de Moura and R. Ierusalimschy. Revisiting coroutines. MCC 15/04, PUC-Rio, 2004.

[^20]: R. K. Dybvig. Three Implementation Models for Scheme. PhD thesis, Department of Computer Science, University of North Carolina at Chapel Hill, 1987. Technical Report #87-011.

[^21]: M. Feeley and G. Lapalme. Closure generation based on viewing LAMBDA as EPSILON plus COMPILE. Journal of Computer Languages, 17 (4):251–267, 1992.

[^22]: T. G. Gorham and R. Ierusalimschy. Um sistema de depurac¸˜ao reflexivo para uma linguagem de extens˜ao.In Anais do I Simp´osio Brasileiro de Linguagens de Programac¸˜ao, pages 103–114, 1996.

[^23]: T. Gutschmidt. Game Programming with Python, Lua, and Ruby. Premier Press, 2003.

[^24]: M. Harmon. Building Lua into games. In Game Programming Gems 5, pages 115–128. Charles River Media, 2005.

[^25]: J. Heiss. Lua Scripting f¨ur Spieleprogrammierer. Hit the Ground with Lua. Stefan Zerbst, Dec. 2005.

[^26]: A. Hester, R. Borges, and R. Ierusalimschy. Building flexible and extensible web applications with Lua. Journal of Universal Computer Science, 4 (9):748–762, 1998.

[^27]: R. Ierusalimschy. Programming in Lua. Lua.org, 2003.

[^28]: R. Ierusalimschy. Programming in Lua. Lua.org, 2nd edition, 2006.

[^29]: R. Ierusalimschy, W. Celes, L. H. de Figueiredo, and R. de Souza. Lua: uma linguagem para customizac¸˜ao de aplicacoes. In VII Simp´osio Brasileiro de Engenharia de Software — Caderno de Ferramentas, page 55, 1993.

[^30]: R. Ierusalimschy, L. H. de Figueiredo, and W. Celes. Lua: an extensible extension language. Software: Practice & Experience, 26 (6):635–652, 1996.

[^31]: R. Ierusalimschy, L. H. de Figueiredo, and W. Celes. The implementation of Lua 5.0. Journal of Universal Computer Science, 11 (7):1159–1176, 2005.

[^32]: R. Ierusalimschy, L. H. de Figueiredo, and W. Celes. Lua 5.1 Reference Manual. Lua.org, 2006.

[^33]: K. Jung and A. Brown. Beginning Lua Programming. Wrox, 2007

[^34]: L. Lamport. LATEX: A Document Preparation System. Addison-Wesley, 1986.

[^35]: M. J. Lima and R. Ierusalimschy. Continuac¸˜oes em Lua. In VI Simp´osio Brasileiro de Linguagens de Programac¸˜ao, pages 218–232, June 2002.

[^36]: D. McDermott. An efficient environment allocation scheme in an interpreter for a lexically-scoped LISP. In ACM conference on LISP and functional programming, pages 154–162, 1980.

[^37]: I. Millington. Artificial Intelligence for Games. Morgan Kaufmann, 2006.

[^38]: B. Mogilefsky. Lua in Grim Fandango. [http://www.grimfandango.net/?page=articles&pagenumber=2], May 1999.

[^39]: Open Software Foundation. OSF/Motif Programmer’s Guide. Prentice-Hall, Inc., 1991.

[^40]: J. Ousterhout. Tcl: an embeddable command language. In Proc. of the Winter 1990 USENIX Technical Conference. USENIX Association, 1990.

[^41]: D. Sanchez-Crespo. Core Techniques and Algorithms in Game Programming. New Riders Games, 2003.

[^42]: P. Schuytema and M. Manyen. Game Development with Lua. Delmar Thomson Learning, 2005.

[^43]: A. van Deursen, P. Klint, and J. Visser. Domain-specific languages: an annotated bibliography. SIGPLAN Notices, 35 (6):26–36, 2000.

[^44]: A. Varanese. Game Scripting Mastery. Premier Press, 2002.
