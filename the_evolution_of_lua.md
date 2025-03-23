# The Evolution of Lua

| Roberto Ierusalimschy                                           | Luiz Henrique de Figueiredo                                    | Waldemar Celes                                                  |
| --------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------- |
| Department of Computer Science, PUC-Rio, Rio de Janeiro, Brazil | IMPA–Instituto Nacional de Matem´atica Pura e Aplicada, Brazil | Department of Computer Science, PUC-Rio, Rio de Janeiro, Brazil |
| [roberto@inf.puc-rio.br]                                        | [lhf@impa.br]                                                  | [celes@inf.puc-rio.br]                                          |

## Abstract

我们报告了Lua语言的诞生和演变，并讨论了它如何从一个简单的配置语言发展成为一种多功能、广泛使用的语言，支持可扩展的语义、匿名函数、完整的词法作用域、适当的尾调用以及协程。

分类与主题描述符 K.2 [计算机历史]: 软件；D.3 [编程语言]

## 1. Introduction

Lua是一种脚本语言，于1993年诞生于巴西里约热内卢的天主教大学（PUC-Rio）。自那时起，Lua不断发展，现已被广泛应用于各种工业领域，如机器人技术、文学编程、分布式业务、图像处理、可扩展文本编辑器、以太网交换机、生物信息学、有限元分析软件包、网络开发等[^2]。特别是在游戏开发领域，Lua已成为领先的脚本语言之一。

Lua的发展远远超出了我们最乐观的预期。事实上，尽管几乎所有的编程语言都源自北美和西欧（除了来自日本的Ruby这一显著例外）[^4]，Lua是唯一一个在发展中国家创建并取得全球影响力的语言。

从一开始，Lua就被设计为简单、小巧、可移植、快速且易于嵌入应用程序中。这些设计原则至今仍然有效，我们相信它们正是Lua在工业领域取得成功的原因。Lua的主要特点及其简洁性的生动体现，在于它提供了一种单一的数据结构——表（table），这是Lua对关联数组的称呼[^9]。尽管大多数脚本语言都提供关联数组，但在其他语言中，关联数组并未扮演如此核心的角色。Lua的表为模块、基于原型的对象、基于类的对象、记录、数组、集合、包、列表以及许多其他数据结构提供了简单而高效的实现[^28]。

在本文中，我们报告了Lua的诞生与演变。我们探讨了Lua如何从一个简单的配置语言发展成为一种强大（但仍保持简洁）的语言，支持可扩展的语义、匿名函数、完整的词法作用域、适当的尾调用以及协程。在 §2 中，我们概述了Lua的主要概念，这些概念将在其他章节中用于讨论Lua的演变过程。在 §3 中，我们回顾了Lua的“史前史”，即导致其诞生的背景。在 §4 中，我们讲述了Lua的诞生过程、其最初的设计目标以及第一个版本的功能。 §5 讨论了Lua如何以及为何演变，而 §6 则对部分特性的演变进行了详细探讨。本文在 §7 中以对Lua演变的回顾作为结尾，并在 §8 中简要讨论了Lua成功的原因，尤其是在游戏领域中的成功。

## 2. Overview

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

[^9]: J. Bentley. Programming pearls: associative arrays. Communications of the ACM, 28(6):570–576, 1985.

[^10]: J. Bentley. Programming pearls: little languages. Communications of the ACM, 29(8):711–721, 1986.

[^11]: C. Bruggeman, O. Waddell, and R. K. Dybvig. Representing
control in the presence of one-shot continuations. In SIGPLAN Conference on Programming Language Design and Implementation, pages 99–107, 1996.

[^12]:  W. Celes, L. H. de Figueiredo, and M. Gattass. EDG: umaferramenta para criac¸˜ao de interfaces gr´aficas interativas. In Proceedings of SIBGRAPI ’95 (Brazilian Symposium on Computer Graphics and Image Processing), pages 241–248, 1995.

[^13]: B. Davis, A. Beatty, K. Casey, D. Gregg, and J. Waldron. The case for virtual register machines. In Proceedings of the 2003 Workshop on Interpreters, Virtual Machines and Emulators, pages 41–49. ACM Press, 2003.

[^14]: L. H. de Figueiredo, W. Celes, and R. Ierusalimschy. Programming advanced control mechanisms with Lua coroutines. In Game Programming Gems 6, pages 357–369. Charles River Media, 2006.

[^15]: L. H. de Figueiredo, R. Ierusalimschy, and W. Celes. The design and implementation of a language for extending applications. In Proceedings of XXI SEMISH (Brazilian Seminar on Software and Hardware), pages 273–284, 1994.

[^16]: L. H. de Figueiredo, R. Ierusalimschy, and W. Celes. Lua: an extensible embedded language. Dr. Dobb’s Journal, 21(12):26–33, Dec. 1996.

[^17]: L. H. de Figueiredo, C. S. Souza, M. Gattass, and L. C. G. Coelho. Gerac¸˜ao de interfaces para captura de dados sobre desenhos.In Proceedings of SIBGRAPI ’92 (Brazilian Symposium on Computer Graphics and Image Processing), pages 169–175, 1992.

[^18]: A. de Moura, N. Rodriguez, and R. Ierusalimschy. Coroutines in Lua. Journal of Universal Computer Science, 10(7):910–925, 2004.

[^19]: A. L. de Moura and R. Ierusalimschy. Revisiting coroutines. MCC 15/04, PUC-Rio, 2004.

[^20]: R. K. Dybvig. Three Implementation Models for Scheme. PhD thesis, Department of Computer Science, University of North Carolina at Chapel Hill, 1987. Technical Report #87-011.

[^21]: M. Feeley and G. Lapalme. Closure generation based on viewing LAMBDA as EPSILON plus COMPILE. Journal of Computer Languages, 17(4):251–267, 1992.

[^22]: T. G. Gorham and R. Ierusalimschy. Um sistema de depurac¸˜ao reflexivo para uma linguagem de extens˜ao.In Anais do I Simp´osio Brasileiro de Linguagens de Programac¸˜ao, pages 103–114, 1996.

[^23]: T. Gutschmidt. Game Programming with Python, Lua, and Ruby. Premier Press, 2003.

[^24]: M. Harmon. Building Lua into games. In Game Programming Gems 5, pages 115–128. Charles River Media, 2005.

[^25]: J. Heiss. Lua Scripting f¨ur Spieleprogrammierer. Hit the Ground with Lua. Stefan Zerbst, Dec. 2005.

[^26]: A. Hester, R. Borges, and R. Ierusalimschy. Building flexible and extensible web applications with Lua. Journal of Universal Computer Science, 4(9):748–762, 1998.

[^27]: R. Ierusalimschy. Programming in Lua. Lua.org, 2003.

[^28]: R. Ierusalimschy. Programming in Lua. Lua.org, 2nd edition, 2006.

[^29]: R. Ierusalimschy, W. Celes, L. H. de Figueiredo, and R. de Souza. Lua: uma linguagem para customizac¸˜ao de aplicacoes. In VII Simp´osio Brasileiro de Engenharia de Software — Caderno de Ferramentas, page 55, 1993.

[^30]: R. Ierusalimschy, L. H. de Figueiredo, and W. Celes. Lua: an extensible extension language. Software: Practice & Experience, 26(6):635–652, 1996.

[^31]: R. Ierusalimschy, L. H. de Figueiredo, and W. Celes. The implementation of Lua 5.0. Journal of Universal Computer Science, 11(7):1159–1176, 2005.

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

[^43]: A. van Deursen, P. Klint, and J. Visser. Domain-specific languages: an annotated bibliography. SIGPLAN Notices, 35(6):26–36, 2000.

[^44]: A. Varanese. Game Scripting Mastery. Premier Press, 2002.
