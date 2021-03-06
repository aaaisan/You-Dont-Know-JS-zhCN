# You Don't Know JS: Up & Going
# Chapter 3: Into YDKJS

这系列丛书都讲了些什么呢？一句话来说，就是认真对待学习 *JS所有内容* 的这件事，不是有些人所说的“好的部分”或者让你完成你的工作所需要的最小部分。

其它语言的认真的开发者会努力学习它们使用的语言的大多甚至全部，但是JS开发者好像比较特殊，他们一般不愿意学习太多这门语言，这样不好，而且我们不应该允许这样的情况泛滥。

*YDKJS* 系列丛书与一般学习JS的方法完全对立，也不像其它你读到的JS的书，它挑战你离开自己的舒适区，对遇到的每一个特性都深入的追问原因。你准备好这种挑战了吗？

我将会通过这最后一章来概述一下在本系列的其余书中的内容，以及如何最有效的通过本系列丛书建立学习JS的基础。

## Scope & Closures（作用域和闭包）

可能你需要先赶紧学习的基础就是变量的作用域在JS中如何工作，仅仅知道一些关于作用域的模糊说法是不够的，卷 *Scope & Closures* 首先揭穿一个JS是“解释型语言”所以不会被编译的错误概念。

JS引擎会在你的代码执行前（有时会在执行时）编译它，所以我们使用一些略深的编译器到我们代码的知识来理解它是如何寻找和处理变量和函数声明的。同时，我们会讲解JS变量作用域管理的典型隐喻——“变量提升”。

批判的理解“词法作用域”是我们学习本书最后一章探索闭包的基础。闭包也许是JS中最重要的概念，但如果你不先熟悉作用域工作原理，你可能一直都理解不了闭包。

闭包最重要的一个应用就是模块模式（模块化编程），本书的第二章会简单介绍，而模块模式可能是JS所有代码设计模式中最流行一个，一定要深入的理解它。

## this & Object Prototypes（this和对象原型）

也许JS中最流行且顽固的错误知识就是`this`关键字指向它所在的函数，很可怕的一个错误。

`this`关键字是动态绑定的，取决于函数到底如何被执行，我总结了四个简单的规则来理解并完全可以判断`this`的绑定关系。

与`this`比较密切的是对象的原型机制，它是属性的查找链，类似于词法作用域的变量查找原理，另一个关于JS的巨大错误就是过分沉溺于原型，比如模拟类和继承的想法。

不幸的是，想要将类和继承的设计模式思想引入到JS中是最糟糕的尝试，因为语法可能会舞蹈你想象有类这些东西存在，事实上原型机制最终表现出的行为却正好相反。

忽略它们的不同假装你实现的是“继承”是否更好，或者学习并拥抱对象原型系统的工作原理是否是合适的，这两个问题仍在争议之中，后者被命名为“行为委托”。

这不仅仅是语法偏好问题，委托是一个完全不同并且更加强大的设计模式，替代了需用类和继承来设计的必要性。但是这种断言会公然违背整个JS一生中关于这个主题的其它每一博文、书籍以及会议演讲。

我提出的关于委托对比继承的观点，不是来自于对这门语言及它语法的喜好，而是来自想要看到这门语言的真正能力被释放以及无尽的困惑和沮丧被扫除的愿景。

但是关于原型和委托的知识会比我在这里说的更多，如果你已准备好重新思考你知道的所有关于JS的“类”和“继承”，我提供给你机会去“探求真相”（*Matrix* 1999），查看卷 *this & Object Prototypes* 的第四至六章。

## Types & Grammar（类型和语法）

第三卷主要关注另一个高争议的主题：类型强转。可能带给JS开发者的沮丧最多的就是隐式强转了。

目前，传统观点是隐式强转是这门语言的“坏部分”，无论如何都应该被避免。事实上，甚至有人称它是这门语言的设计缺陷，而且还有专门的工具来检测即使是有点儿类似强转的代码。

强转真的这么令人困惑，这么糟糕，这么危险，以致于你的代码一旦使用它就被判决失败？

当然不。你在学习第一至三章并对类型和值如何工作有了一定认识，第四章会继续这个讨论，并且完整的从边边角角解释强转的工作原理。我们会看到强转的哪部分真正让人惊喜，以及哪部分真的值得花时间去学习。

但我不仅仅是暗示强转是多么容易上手，我只是说强转是一个很有用且完全被低估的 *你应该在你代码里使用* 的工具，我想说的是，当合理使用时，强转不仅仅可以正常工作，也可以让你的代码更好，那些否定、怀疑者肯定会在这里嘲笑，但是我相信这是你升级你JS游戏的关键环节。

你是想要继续随波逐流还是想要将所有的假设放到一边，用一个新视角来探索强转呢？卷 *Types & Grammar* 会转换你的思维。

## Async & Performance（异步和性能）

前三卷主要关注这门语言的核心机制，但是第四卷稍微拓展了一下，涵盖了在掌握这门语言的基础上控制异步编程的设计模式。异步不仅仅是我们应用的性能关键部分，也是逐渐成为编写、维护代码的关键点。

这本书开始先阐明了一些容易混淆的概念和术语，比如“异步”、“并行”以及“并发”，并且深入解释了这些东西在JS中的适用和不适用。

然后我们会接触到触发异步的基本方法：回调，但是只有回调完全不能满足现代异步编程的需求，我会指出只使用回调编程的两个主要缺陷： *Inversion of Control* (IoC) trust loss and lack of linear reason-ability.【没理解，先不翻译。】

为了强调这两个主要缺陷，ES6引入了两种新的机制（实际上是模式）：promises和generators。

Promises是对“未来值”的独立于时间的封装，它允许你推论、组合它们而无视值是否已经准备好。而且，借助可信、可组合的promise机制，通过路由不同的回调，有效的解决了IOC的可信度问题。

generator引入了一种新的JS函数执行方式，generator可以在`yield`出现的地方暂停，然后异步继续。这种暂停-继续的能力让看起来同步、顺序的代码在generator里可以被异步在后台的处理。如此，我们解决了回调带来的非线性、非本地跳转的困惑，从而让我们的异步代码更像同步代码更合理。

但是是promise和generator的组合“产生”了如今JS中的有效异步编码模式。事实上，更多在ES7以及之后的未来成熟的异步都会建立在这个基础上，为了认真对待在异步世界中有效编程，你需要熟悉promise和generator的组合。

如果promise和generator是关于如何让我们的程序并发从而一定时间内完成更多的处理，JS还有许多其它关于性能优化的方面值得我们探索。

第五章是探究关于使用多线程的程序并行，以及单指令多数据（SIMD）的数据并行的主题，以及低水平的优化技术比如ASM.js。第六章是从合理的基准测试技术角度看一下性能优化，包括什么样的性能需要考虑什么可以忽略。

编写有效JS代码意味着你的代码可以跳出各种浏览器或其它环境的限制，这需要很多复杂且详细的规划以及我们自身编写的程序从“正常运行”到“有效运行”的努力。

卷 *Async & Performance* 提供给你编写合理且高性能的JS代码所需要的所有工具和技能。

## ES6 & Beyond（ES6和ES6+）

至此，无论你感觉自己有多么熟练的掌握JS，事实上，JS一直没有停止演进，而且演进的越来越快，这个事实几乎也是本系列丛书的精神的隐喻，坦然接受我们永远也无法完全 *掌握* JS的每一个部分，因为当你掌握全部时，新的内容又会来到，你又需要再去学习。

此卷是这门语言发展方向的短中期展望，不仅仅是ES6 *已知* 的东西，还有以后 *可能会* 有的东西。

这个系列丛书使用了编写时JS的最新特性，即ES6中期，本系列丛书到目前关注更多的是ES5，所以现在我们需要关注ES6、ES7，以及……

由于ES6在本书编写时基本已经完成，*ES6和ES6+* 开始就将ES6的内容划分为几个大类，包括新语法、新数据结构（集合）、新处理能力以及新API。我们分不同的层次来讲解这几个ES6的新特性，包括回顾那些在本系列其它卷中提到的。

一些让人兴奋的ES6的东西期待我们阅读：解构、参数默认值、符号、简明方法、计算属性、箭头函数、块作用域、promise、generator、模块、代理、弱引用map以及更多~哇咔咔，ES6真给力！

本书的第一部分是关于所有你要学习的路线图，它指引你为后续几年你将会编写和探索的新的改进的JS做好准备。

后续部分主要是关于未来JS中可能会出现的东西，很重要的一点是ES6之后，JS很可能会以特性演进，而再是通过版本迭代，这意味着你可能会以无法想象的速度看到新特性的出现。

JS的前途很光明，我们赶快开始它的学习之路吧（译者注：道路是曲折的）！

## Review（回顾）

*YDKJS* 系列致力于提议：所有JS开发者都能且应该学习这门伟大语言的所有部分。别人的意见、框架的使用以及项目deadline都不足以成为你不去深入学习并理解JS的理由。

我们选取了这门语言的每一个重要内容，并且贡献一本短小精悍的书来全面探索它所有你以为你知道但其实并不的部分。

“不其实不懂JS”并不是批评或侮辱谁，它是我们所有人（当然包括我）都必须接受的事实，学习JS不是目的而是过程，我们还不懂JS，但是我们将会懂！
