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

Lua 于 1993 年诞生于巴西里约热内卢天主教大学（PUC-Rio）的计算机图形技术组（Tecgraf）。Lua 的创造者是 Roberto Ierusalimschy、Luiz Henrique de Figueiredo 和 Waldemar Celes。Roberto 当时是 PUC-Rio 计算机科学系的助理教授，Luiz Henrique 是博士后研究员，先在 IMPA（巴西纯数学与应用数学研究所）工作，后来加入 Tecgraf，而 Waldemar 是 PUC-Rio 计算机科学系的博士生。他们三人都是 Tecgraf 的成员，在共同开发 Lua 之前，分别参与不同的项目。他们的背景不同但相关：Roberto 是一位主要对编程语言感兴趣的计算机科学家；Luiz Henrique 是一位对软件工具和计算机图形学感兴趣的数学家；Waldemar 则是一位对计算机图形学应用感兴趣的工程师。（2001 年，Waldemar 加入 Roberto 成为 PUC-Rio 的教职员工，而 Luiz Henrique 则成为 IMPA 的研究员。）

Tecgraf 是一个拥有多个工业合作伙伴的大型研发实验室。自 1987 年 5 月成立后的头十年里，Tecgraf 主要致力于构建基础软件工具，以满足其客户对交互式图形程序的需求。因此，Tecgraf 的早期产品包括图形终端、绘图仪和打印机的驱动程序、图形库以及图形界面工具包。从 1977 年到 1992 年，巴西出于民族主义情绪，认为巴西能够并且应该生产自己的硬件和软件，因此对计算机硬件和软件实施了严格的贸易壁垒政策（称为 “市场保留”）。在这种背景下，Tecgraf 的客户在政治和经济上都无法承担从国外购买定制软件的费用：根据市场保留规则，他们必须通过复杂的官僚程序证明巴西公司无法满足他们的需求。再加上巴西与其他研发中心自然的地理隔离，这些原因促使 Tecgraf 从头开始实现其所需的基础工具。

Tecgraf 最大的合作伙伴之一（至今仍是）是巴西石油公司 Petrobras。Tecgraf 的多个产品是为 Petrobras 的工程应用开发的交互式图形程序。到 1993 年，Tecgraf 已经为其中两个应用程序开发了小型语言：一个是数据输入应用程序，另一个是用于岩性剖面分析的可配置报告生成器。这两种语言分别称为 DEL 和 SOL，它们是 Lua 的前身。我们在此简要描述它们，以展示 Lua 的起源。

### 3.1 DEL

Petrobras 的工程师每天需要多次为数值模拟器准备输入数据文件。这一过程既枯燥又容易出错，因为模拟程序是遗留代码，需要严格格式化的输入文件 —— 通常是纯数字列，没有任何说明每个数字的含义，这种格式源自打孔卡时代。1992 年初，Petrobras 要求 Tecgraf 为这类数据输入创建至少十几个图形前端。数字将通过点击描述模拟的图表的相关部分进行交互式输入 —— 这对工程师来说比编辑数字列要容易得多且更有意义。模拟器所需格式的数据文件将自动生成。除了简化数据文件的创建，这些前端还提供了数据验证的机会，并可以从输入数据中计算派生量，从而减少用户需要输入的数据量，并提高整个过程的可靠性。

为了简化这些前端的开发，由 Luiz Henrique de Figueiredo 和 Luiz Cristovão Gomes Coelho 领导的团队决定以统一的方式编写所有前端，并因此设计了 DEL（“数据输入语言”），这是一种简单的声明式语言，用于描述每个数据输入任务 [^17]。DEL 是现在所谓的领域特定语言 [^43]，但在当时简称为小型语言 [^10]。

典型的 DEL 程序定义了多个 “实体”。每个实体可以有多个字段，这些字段都有名称和类型。为了实现数据验证，DEL 包含谓词语句，用于对实体的值施加限制。DEL 还包括指定数据输入和输出方式的语句。DEL 中的实体本质上就是传统编程语言中的结构体或记录。重要的区别在于 —— 这也是 DEL 适合数据输入问题的原因 —— 实体名称也出现在一个单独的图形元文件中，该文件包含相关的图表，工程师在此图表上进行数据输入。一个名为 ED（“entrada de dados” 的缩写，葡萄牙语中意为 “数据输入”）的交互式图形解释器被编写出来，用于解释 DEL 程序。Petrobras 要求的所有数据输入前端都被实现为在此单一图形应用程序下运行的 DEL 程序。

DEL 在 Tecgraf 的开发人员和 Petrobras 的用户中都取得了成功。在 Tecgraf，DEL 如最初预期的那样简化了前端的开发。在 Petrobras，DEL 允许用户根据他们的需求定制数据输入应用程序。很快，用户开始要求 DEL 具备更多功能，例如用于控制实体是否处于输入状态的布尔表达式，DEL 因此变得更加复杂。当用户开始要求控制流（如条件语句和循环）时，很明显 ED 需要一个真正的编程语言，而不是 DEL。

### 3.2 SOL

大约在 DEL 创建的同时，由 Roberto Ierusalimschy 和 Waldemar Celes 领导的团队开始开发 PGM，这是一个用于岩性剖面分析的可配置报告生成器，同样是为 Petrobras 开发的。PGM 生成的报告由多个列（称为 “轨道”）组成，并且具有高度可配置性：用户可以创建和定位轨道，并选择颜色、字体和标签；每个轨道可以有一个网格，网格也有其一系列选项（对数 / 线性、垂直和水平刻度等）；每条曲线都有自己的比例尺，在溢出情况下必须自动调整；等等。所有这些配置都需要由终端用户完成，通常是 Petrobras 在石油工厂和海上平台工作的地质学家和工程师。配置需要存储在文件中以便重复使用。团队决定，配置 PGM 的最佳方式是通过一种称为 SOL（Simple Object Language 的缩写）的专用描述语言。

由于 PGM 需要处理许多不同的对象，每个对象都有许多不同的属性，SOL 团队决定不将这些对象和属性固定在语言中。相反，SOL 允许类型声明，如下面的代码所示：

```SOL
type @track{ x:number, y:number=23, id=0 }
type @line{ t:@track=@track{x=8}, z:number* }
T = @track{ y=9, x=10, id="1992-34" }
L = @line{ t=@track{x=T.y, y=T.x}, z=[2,3,4] }
```

这段代码定义了两个类型：`track` 和 `line`，并创建了两个对象：一个轨道 `T` 和一条线 `L`。track 类型包含两个数值属性 `x` 和 `y`，以及一个无类型属性 `id`；属性 `y` 和 `id` 具有默认值。`line` 类型包含一个轨道 `t` 和一个数字列表 `z`。轨道 `t` 的默认值为 `x=8`、`y=23` 和 `id=0` 的轨道。SOL 的语法深受 BibTEX [^34] 和 UIL（一种用于描述 Motif 用户界面的语言）[^39] 的影响。

SOL 解释器的主要任务是读取报告描述，检查给定的对象和属性是否类型正确，然后将信息传递给主程序（PGM）。为了实现主程序和 SOL 解释器之间的通信，解释器被实现为一个 C 库，并与主程序链接。主程序可以通过该库中的 API 访问所有配置信息。特别是，主程序可以为每种类型注册一个回调函数，SOL 解释器会调用该函数来创建该类型的对象。

## 4. Birth

SOL 团队在 1993 年 3 月完成了 SOL 的初始实现，但他们从未交付它。PGM 很快需要支持过程式编程，以允许创建更复杂的布局，而 SOL 将不得不进行扩展。与此同时，如前所述，ED 用户要求 DEL 具备更多功能。ED 还需要进一步的描述性功能来编程其用户界面。大约在 1993 年年中，Roberto、Luiz Henrique 和 Waldemar 聚在一起讨论 DEL 和 SOL，并得出结论：这两种语言可以被一种更强大的单一语言取代，他们决定设计和实现这种语言。于是，Lua 团队诞生了；自那以后，团队一直未变。

根据 ED 和 PGM 的需求，我们决定需要一种真正的编程语言，具备赋值、控制结构、子程序等功能。该语言还应提供数据描述功能，例如 SOL 所提供的功能。此外，由于该语言的许多潜在用户并非专业程序员，语言应避免晦涩的语法和语义。新语言的实现应具有高度可移植性，因为 Tecgraf 的客户拥有多样化的计算机平台。最后，由于我们预计其他 Tecgraf 产品也需要嵌入脚本语言，新语言应遵循 SOL 的范例，并作为一个具有 C API 的库提供。

当时，我们可以选择采用现有的脚本语言，而不是创建一种新语言。1993 年，唯一真正的竞争者是 Tcl [^40]，它被明确设计为嵌入应用程序中。然而，Tcl 的语法不够直观，对数据描述的支持也不够完善，并且只能在 Unix 平台上运行。我们没有考虑 LISP 或 Scheme，因为它们的语法不够友好。Python 当时还处于起步阶段。在 Tecgraf 当时盛行的自由、自己动手的氛围中，尝试开发我们自己的脚本语言是很自然的。因此，我们开始着手开发一种新语言，希望它比现有语言更简单易用。我们最初的设计决策是：保持语言简单小巧，并保持实现简单且可移植。由于新语言部分灵感来自 SOL（葡萄牙语中的 “太阳”），Tecgraf 的一位朋友（Carlos Henrique Levy）建议将其命名为 “Lua”（葡萄牙语中的 “月亮”），于是 Lua 诞生了。（DEL 并未直接影响 Lua 作为一门语言的设计。DEL 对 Lua 诞生的主要影响在于让我们意识到，复杂应用程序的很大部分可以通过嵌入式脚本语言来实现。）

我们想要一种轻量级但功能完备的语言，并具备数据描述能力。因此，我们采用了 SOL 中用于记录和列表构造的语法（但不包括类型声明），并通过表（table）统一了它们的实现：记录使用字符串（字段名）作为索引；列表使用自然数作为索引。例如，像这样的赋值语句：

```SOL
T = @track{ y=9, x=10, id="1992-34" }
```

在 SOL 中有效的语法，在 Lua 中仍然有效，但含义不同：它创建了一个对象（即一个表），并包含给定的字段，然后调用函数 track 对这个表进行验证，或者为某些字段提供默认值。该表达式的最终值就是这个表。

除了其过程式数据描述结构外，Lua 并未引入新的概念：Lua 是为实际生产使用而创建的，而不是作为一种支持编程语言研究的学术语言。因此，我们只是简单借鉴了（甚至无意识地）我们在其他语言中见过或读过的内容。我们没有重新阅读旧论文来回忆现有语言的细节，而是从我们对其他语言的了解出发，并根据我们的喜好和需求进行了调整。

我们很快确定了一小组控制结构，其语法主要借鉴了 Modula（如 `while`、`if` 和 `repeat until`）。从 CLU 中我们借鉴了多重赋值和函数调用中的多重返回值。我们将多重返回值视为 Pascal 和 Modula 中使用的引用参数以及 Ada 中使用的输入输出参数的更简单替代方案；我们还希望避免使用显式指针（如 C 语言中的指针）。从 C++ 中我们借鉴了允许局部变量在需要时才声明的巧妙想法。从 SNOBOL 和 Awk 中我们借鉴了关联数组，并将其称为表（tables）；然而，在 Lua 中，表是对象，而不是像 Awk 中那样绑定到变量。

Lua 中为数不多（且相对较小）的创新之一是字符串连接的语法。自然的 `+` 运算符会引发歧义，因为我们希望在算术操作中自动将字符串强制转换为数字。因此，我们发明了 `..`（两个点）作为字符串连接的语法。

一个有争议的点是分号的使用。我们认为，对于有 Fortran 背景的工程师来说，要求使用分号可能会有些困惑，但不允许使用分号又会让有 C 或 Pascal 背景的工程师感到困惑。典型的委员会风格下，我们决定让分号成为可选项。

最初，Lua 有七种类型：数字（仅实现为实数）、字符串、表（tables）、nil、用户数据（指向 C 对象的指针）、Lua 函数和 C 函数。为了保持语言的简洁，我们最初没有包含布尔类型：与 Lisp 一样，nil 表示假，其他任何值表示真。在 13 年的持续演变中，Lua 类型的唯一变化是在 Lua 3.0（1997 年）中将 Lua 函数和 C 函数统一为单一的函数类型，以及在 Lua 5.0（2003 年）中引入布尔类型和线程类型（见 §6.1）。为了简单起见，我们选择使用动态类型而不是静态类型。对于需要类型检查的应用程序，我们提供了基本的反射功能，例如运行时类型信息和全局环境的遍历，作为内置函数（见 §6.11）。

到 1993 年 7 月，Waldemar 完成了 Lua 的第一个实现，这是由 Roberto 指导的课程项目。该实现遵循了如今成为极限编程核心的信条：“最简单的可能有效的方法”[^7]。词法分析器使用 lex 编写，解析器使用 yacc 编写，这是实现语言的经典 Unix 工具。解析器将 Lua 程序翻译为基于栈的虚拟机的指令，然后由一个简单的解释器执行。C API 使得向 Lua 添加新函数变得非常容易，因此第一个版本仅提供了一个包含五个内置函数（`next`、`nextvar`、`print`、`tonumber`、`type`）的小型库和三个小型外部库（输入输出、数学函数和字符串操作）。

尽管实现简单 —— 或者可能正是因为其简单 ——Lua 的表现超出了我们的预期。PGM 和 ED 都成功使用了 Lua（PGM 至今仍在使用；ED 被 EDG [^12] 取代，后者大部分是用 Lua 编写的）。Lua 在 Tecgraf 内部立即取得了成功，很快其他项目也开始使用它。1993 年 10 月，在第七届巴西软件工程研讨会上，我们简要报告了 Lua 在 Tecgraf 的初步使用情况 [^29]。

本文的其余部分将讲述我们在改进 Lua 过程中的历程。

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
