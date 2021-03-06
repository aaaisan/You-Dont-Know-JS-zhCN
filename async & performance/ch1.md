# You Don't Know JS: Async & Performance
# Chapter 1: Asynchrony: Now & Later

如何表达以及处理比较耗时的程序，是像JS这种编程语言里重要却经常被错误理解的一个知识。

这里说的不仅仅是`for`循环的执行耗时（当然它也会消耗一定时间来完成），而是说你的程序中的一部分*当前*在执行，而另一部分*过一会儿*才执行，这里会有一段时间缝隙，你的程序没有在活动。

事实上，所有重要的程序都通过了一定的方式来处理这段时间，或者等待用户输入，或者从数据库、文件系统获取数据，或者通过网络发送请求等待响应，或者周期性执行某个任务（比如动画）。不论那种方式，你的程序都需要及时的控制那段缝隙的状态切换，就像坐地铁听到的那样“请小心列车与站台之间的缝隙”。你的程序里的*当前*和*后来*的关系正是异步编程的核心。

异步编程从JS刚出现时就已经存在了，但是大多数的开发者都没有认真的思考它为何会突然出现在程序里，也没有去探索*其它*方式来处理它。一个*足够好的*方式就是用简单的回调函数，许多人直到今天依然坚持使用回调绰绰有余。

但是随着JS不断的扩展和变得复杂，为了满足不断扩大的主流编程语言的需求，比如可以运行在浏览器和服务器甚至任何合适的设备上，我们用来处理异步的方式越来越显的拙劣，我们迫切需要一种强大的途经。

尽管目前看起来很抽象，我确信随着你阅读完这本书，我们会逐渐掌握它。接下来几章我们会探究很多新兴的JS中处理异步的技巧。

但在这之前，我们先来深入了解以下异步以及它在JS中的运行原理。

## A Program in Chunks

你可能将你的JS程序只写在一个*.js*文件里，但它依然可以被划分为几块，而且在*当前*，只有其中的一个在运行，其余的都是*将*要被执行，（译者注：其实还有已经执行了的）。最常见的块就是`function`。

许多新手JS开发者遇到的问题是*后来*并不是恰好在*当前*之后立即执行。或者说，那些在*当前*无法完成的，当然是要异步的去完成，这样就不会阻塞其它你所预期的行为。

比如:

```js
// ajax(..) is some arbitrary Ajax function given by a library
var data = ajax( "http://some.url.1" );

console.log( data );
// Oops! `data` generally won't have the Ajax results
```

你可能已经知道，标准的Ajax请求不会同步完成，也就是说`ajax(..)`函数还没有值来返回给变量`data`，如果`ajax(..)`*可以*阻塞直到结果返回，那么`data = ..`的赋值就会生效。

但这也不是Ajax的用法，我们*当前*发送一个异步的Ajax请求，直到*后来*我们不会得到任何结果。最简单（但绝不是只有或最好）的方式来从*当前*等待*后来*就是函数，通常也叫回调函数：

```js
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", function myCallbackFunction(data){

	console.log( data ); // Yay, I gots me some `data`!

} );
```

**警告：** 你可能听说过创建同步Ajax请求是可能的，这个技术上可以实现，但是你应该避免在任何场景下使用，因为它会锁死浏览器的UI（按钮、菜单、滚动等），阻止任何的用户交互。这种用法很糟糕，坚决应该被避免。

在你提出异议之前，或者，你想要避免大量回调*不*应该是你使用阻塞的同步Ajax的理由。

比如，考虑下面的代码:

```js
function now() {
	return 21;
}

function later() {
	answer = answer * 2;
	console.log( "Meaning of life:", answer );
}

var answer = now();

setTimeout( later, 1000 ); // Meaning of life: 42
```

该程序有两部分：*当前*执行的，*后来*要执行的，这两部分的功能很明显，但是让我们再仔细看看：

当前:

```js
function now() {
	return 21;
}

function later() { .. }

var answer = now();

setTimeout( later, 1000 );
```

后来:

```js
answer = answer * 2;
console.log( "Meaning of life:", answer );
```

*当前*部分在你程序运行时就立即执行，但是`setTimeout(..)`也创建了一个*后来*才发生的事件，所以`later()`函数里的内容会在一定时间（此程序中1秒）过后才执行。

无论何时你将代码的一部分封装到`function`里，然后设置它在一些事件（比如定时、鼠标点击、Ajax响应等）发生时才执行，你就在代码里创建了一个*后来*部分，从而导致程序的异步。

### Async Console

没有说明书或其它文本来说明`console.*`方法的运行原理——它们并不是JS的正式内容，而是由*主机环境*添加到JS的（见本系列书里的*Type & Grammer*那本）。

所以，不同的浏览器和JS环境实现机制不同，有时会引发让人困惑的现象。

事实上，有一些浏览器或者某些场景下`console.log(..)`不是立即输出。这种事情的发生是因为I/O是很多语言（不仅仅JS）的缓慢且阻塞的部分，所以，从页面或UI角度来讲，让浏览器处理异步的`console`的I/O，更好些。

一个不是很常见但一定会存在的场景，上面的情况可以*被观察到*（不是从代码本身，而是从外部）：

```js
var a = {
	index: 1
};

// later
console.log( a ); // ??

// even later
a.index++;
```

我们期望的是`a`对象在执行到`console.log(..)`语句时立即被打印出`{ index: 1 }`，然后`a.index++`在前面的输出语句输出后修改`a`的内容。

大多数情况下，上面的代码执行都没有问题，但是也有这种可能，当浏览器觉得需要将控制台的I/O推迟，此时，`a.index++`就有可能发生在浏览器控制台打印之前（注意，不是打印语句执行之前），从而可能会输出`{ index: 2}`。

`console`的I/O被推迟会在特定条件下发生，有时是察觉到的，所以记住I/O里的异步性，以防你在调试中遇到对象在`console.log(..)`语句后被修改却看到了预料外的修改被打印的问题。

**注意：** 如果你遇到了这种奇葩的场景，最好的方式就是使用JS调试器而不是依赖`console`输出，另一个最佳选择就是强制输出涉及对象的“快照”，将它序列化为一个`string`，比如使用`JSON.stringify(..)`。

## Event Loop

我们总结一下：尽管JS代码里的异步是被允许的（比如我们上面看到的timeout），但直到最近（ES6），JS本身并没有直接的异步概念。

**纳尼！？** 很奇葩对不对？然而事实如此。JS引擎本身除了在特定的时间去按要求执行仅仅一块代码外不会去做比的事情。

“按要求。”被谁？这是关键点。

JS引擎不是独立运行的，它需要在一个*主机环境*里运行，也就是对大多数开发者来讲的浏览器。过去几年，JS已经扩展到了浏览器之外的其它运行环境，比如服务器，通过类似Node.js的东西。事实上，JS这几年已经被嵌入到了各种设备，从机器人到电灯。

但是所有这些环境的那一个公共的“线程”（这是一个含糊其辞的玩笑，却很对），是它们都有一个机制来*随着时间*处理你的程序里的各个部分，在每一时刻，激活JS引擎，这就叫做“事件轮询”。

换言之，JS引擎自身没有*时间*知觉，但是有一个为任何JS代码待命的执行环境，周围的环境经常*调度*“事件”（JS代码的执行）。

举个栗子，当你JS程序发送一个Ajax请求去从服务器获取数据时，你在一个函数里进行“响应”的处理（通常叫做“回调”），然后JS引擎告诉主机环境，“嗨，我现在要暂停执行了，但是当你完成网络请求后，你取得了一些数据，请*返回*去*调用*这个函数。

然后浏览器监听网络的响应，当有内容返回时，它通过把回调函数插入到*事件轮询*里来通过调度执行。。

所以，什么是*事件轮询*？

我们通过伪代码来概念化：

```js
// `eventLoop` is an array that acts as a queue (first-in, first-out)
var eventLoop = [ ];
var event;

// keep going "forever"
while (true) {
	// perform a "tick"
	if (eventLoop.length > 0) {
		// get the next event in the queue
		event = eventLoop.shift();

		// now, execute the next event
		try {
			event();
		}
		catch (err) {
			reportError(err);
		}
	}
}
```

这个当然是简单化了概念表述，但是已经足够让我们更容易理解它。

如你所见，有一个通过`while`循环表示的运行循环，每一次迭代称为“tick”，每一次tick，如果在队列中有事件，它会被取出来并执行，这些事件就是你的回调函数。

值得**注意**的是`setTimeout(..)`并不是把回调放在事件轮询队列里，它只是创建一个定时器，当定时器到期时，运行环境会将回调放到事件轮询里，这样后来的tick会取出它并执行。

如果当前事件轮询里已经有20个元素了会发生什么？你的回调会等待。它在其它的后面排队，一般没有方法来插队或者跳到队首。这也解释了为何`setTimeout(..)`定时器并不总是在精确的延时后触发。你想当然到认为你的回调不会在你设置的时间*之前*触发，但是它可能会在那个时间或之后触发，取决于事件队列的状态。

所以，也就是说，你的代码实际上是被划分成了许多小块，在事件轮询队列里一个接着一个的执行，技术上讲，其它与你的程序不直接相关的事件也会被交织到队列里。

**注意：** 我们提到了“直到最近”关于ES6改变了这个事件轮询队列的本质，它是一个正式的技术，但是ES6现在定义了事件轮询如何工作，也就是说它现在已经在JS引擎的范畴里而不是*主机环境*里了。这个改变的原因是ES6 Promises的引入，我们会在第3章讨论，因为它们需要有关于事件轮询队列的调度操作的直观深入的掌握（参考在“Cooperation”章节中对`setTimetou(..0)`的讨论）。

## Parallel Threading

将术语“异步”和“并行”合并很常见，但是它们实际上区别挺大的，记住，异步是关于*当前*和*后来*的间隙，而并行是指事情同时发生。

并行计算最常用的工具就是进程和线程，进程和线程独立或同步执行：在不同的处理器甚至不同的电脑上，不同的线程可以共享一个进程的内存。

对比来看，一个事件轮询将它的工作分解为任务，然后顺序的且禁止并行访问或改变共享内存的执行它们。并行和顺序在以协作事件轮询形式在不同的线程里可以共同存在。

并行线程执行的交错和异步事件执行的交错出现在非常不同的粒度级别里。

比如：

```js
function later() {
	answer = answer * 2;
	console.log( "Meaning of life:", answer );
}
```

`later()`整个内容会仅仅被当作一个事件轮询队列入口，假设这个代码运行在一个线程上，实际上有许多不同更低层的操作，比如`answer = answer * 2`首先需要载入`answer`当前的值，然后把`2`放到某个地方，然后执行乘法，最后把结果存储到`answer`里。

在一个单线程环境里，在线程队列里是否是底层的操作无关紧要，因为没有别的东西可以中断线程，但是如果在并行系统里，如一个程序里运行着两个不同线程，你很可能会遇到不可预测的行为。

考虑：

```js
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

在JS的单线程特性里，如果`foo()`在`bar()`前执行，那么结果就是`a`等于`42`，反之等于`41`。

如果JS事件共享同一数据，并且并发执行，那么问题将会十分微妙，思考下面两个伪代码任务作为可以分别运行`foo()`和`bar()`的线程，如果它们同时运行会发生什么：

If JS events sharing the same data executed in parallel, though, the problems would be much more subtle. Consider these two lists of pseudocode tasks as the threads that could respectively run the code in `foo()` and `bar()`, and consider what happens if they are running at exactly the same time:

线程1（`X`和`Y`是临时的内存地址）：
```
foo():
  a. load value of `a` in `X`
  b. store `1` in `Y`
  c. add `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
```

线程2（`X`和`Y`是临时的内存地址）：
```
bar():
  a. load value of `a` in `X`
  b. store `2` in `Y`
  c. multiply `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
```

现在，让我们假设这两个线程真的在并行跑，你很快会发现问题，对吧？它们在每一步都使用共享的存储位置`X`和`Y`。

如果按照这种步骤执行，`a`的结果是多少？

```
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
1b  (store `1` in `Y`   ==> `1`)
2b  (store `2` in `Y`   ==> `2`)
1c  (add `X` and `Y`, store result in `X`   ==> `22`)
1d  (store value of `X` in `a`   ==> `22`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `44`)
2d  (store value of `X` in `a`   ==> `44`)
```

`a`的结果是`44`。那如果是这种顺序呢？

```
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
2b  (store `2` in `Y`   ==> `2`)
1b  (store `1` in `Y`   ==> `1`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `20`)
1c  (add `X` and `Y`, store result in `X`   ==> `21`)
1d  (store value of `X` in `a`   ==> `21`)
2d  (store value of `X` in `a`   ==> `21`)
```

结果是`a`等于`21`。

所以，线程编码很讲究技巧，因为如果你花费特别的步骤来阻止这种中断／交错的发生，那么你会得到很神奇的不确定的结果，让你痛苦不已。

JS从来不在线程间共享数据，也就是说*那*个不确定性不用顾虑，但并不是说JS一直都是确定性的，还记得前面的代码吗，`foo()`和`bar()`的不同执行顺序出现的两个不同结果（`41`和`42`）。

**注意：** 可能还不明显，但是不是所有的不确定性都是不好的，有时，它不相干，有时相关。我们会在接下来的几章看到。

### Run-to-Completion

由于JS的单线程特性，在`foo()`和`bar()`里的代码是原子化的，意味着一旦`foo()`开始执行，那么它内部的所有代码都会在`bar()`执行前执行完毕，反之亦然，这就是所谓的“运行直到完成”特性。

事实上，当`foo()`和`bar()`里有更多的代码时，运行直到完成的语义会更直观。

```js
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

因为`foo()`不能被`bar()`中断，`bar()`也不能被`foo()`中断，这个程序只会有两种可能的结果，取决于那个函数先执行，而如果是多线程，并且`foo()`和`bar()`内部的代码可以交错，那么会有更多的结果出现！

代码块1时同步的（发生在*当前*），但是代码块2和3时异步的（发生在*后来*），也就是说它们的执行会被时间间隙划分开。

代码块1：
```js
var a = 1;
var b = 2;
```

代码块2 (`foo()`):
```js
a++;
b = b * a;
a = b + 3;
```

代码块3(`bar()`):
```js
b--;
a = 8 + b;
b = a * 2;
```

代码块2和3按谁先执行会有两种可能结果，如下：

Outcome 1:
```js
var a = 1;
var b = 2;

// foo()
a++;
b = b * a;
a = b + 3;

// bar()
b--;
a = 8 + b;
b = a * 2;

a; // 11
b; // 22
```

Outcome 2:
```js
var a = 1;
var b = 2;

// bar()
b--;
a = 8 + b;
b = a * 2;

// foo()
a++;
b = b * a;
a = b + 3;

a; // 183
b; // 180
```

同一个代码块出现两种结果意思仍然有不确定性！但是在函数（事件）顺序级别，而不是多线程的在语句级别（或者，事实上，表达式级别），也就是说比多线程*更确定*了些。

对于JS特性而言，这种函数顺序的不确定性就是常见的术语“竞态条件“，`foo()`和`bar()`竞争谁先执行，更具体说，因为你无法预测确切的`a`和`b`的输出所以它是一个“竞态条件”。

**注意：** 如果在JS中有一个没有运行直到完成特性的函数，我们将会有更多的输出。而ES6正好引入了这个东西（见第4章"Generators"），但是不要担心，我们会讲到它。

## Concurrency

假设一个网站展示一组随着用户下滑会自动加载的状态（比如社交网络的动态展示），为了实现这种效果，（至少）需要两个独立的“进程”来*同时*执行（即在同一个时间窗，但并不是在同一时刻）。

**注意：** 给“进程”加了引号是因为它们并不是真正的计算机科学领域的操作系统级别的进程。他们是虚拟的进程，或者任务，仅表示逻辑上的关联，序列化的操作。我们选择“进程”而不是“任务”是因为从术语角度，前者更符合我们希望讨论的概念。

第一个“进程”会响应`onscroll`事件（创建获取新内容的Ajax请求），它会在用户滚动页面的时候触发，第二个“进程”接收“Ajax响应（并将内容渲染到页面）。

明显的，如果一个用户滚动的很快，你会看到2个或者更多的`onscroll`事件在第一个响应返回以及处理前被触发，这样你会有快速触发的`onscroll`事件和Ajax响应事件交错出现。

并发就是当2个或者更多的“进程”在同一周期内同时被执行，无论它们的各自组成操作是否*并行*（在同一时刻在不同的处理器或内核）。你可以任务并发是“进程”级别的并行，与操作级别（不同的处理器线程）的并行不同。

**注意：** 并发也引入了一个问题，这些“进程”可以相互影响，我们过会儿再说。

对于给定的时间窗（用户滚动花费的几秒），我们奖每一个独立“进程”具体为一系列事件／操作：

“进程” 1 (`onscroll`事件):
```
onscroll, request 1
onscroll, request 2
onscroll, request 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
onscroll, request 7
```

“进程” 2 (Ajax响应事件):
```
response 1
response 2
response 3
response 4
response 5
response 6
response 7
```
很可能一个`onscroll`事件和一个Ajax响应事件同时准备被处理，比如，我们用时间线来描述这些事件：

```
onscroll, request 1
onscroll, request 2          response 1
onscroll, request 3          response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6          response 4
onscroll, request 7
response 6
response 5
response 7
```

然而， 让我们回顾前面关于事件轮询队列的知识，JS一次只处理一个事件，所以，要么`onscroll, request 2`在`response 1`之前执行，要么之后，它们不可能同时被执行，就好像在学校餐厅的孩子，无论他们在门外面如何拥挤，他们最终只能通过一条线来取餐！

我们形象描述所有这些事件在事件轮询队列的交错：

事件轮询队列：
```
onscroll, request 1   <--- Process 1 starts
onscroll, request 2
response 1            <--- Process 2 starts
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- Process 1 finishes
response 6
response 5
response 7            <--- Process 2 finishes
```

“进程1”和“进程2”并发执行（任务级并行），但是他们个各自内部事件按事件轮询队列里的顺序执行。

顺便一提，注意`responce 6`和`response 5`没有按照期望的顺序返回。

单线程事件轮询是并发的一种表达（肯定还有别的，我们随后再说）。

### Noninteracting

因为2个或更多“进程”在一个程序里并发的交错它们的步骤／事件，如果任务不相关，那么它们就没必要交互。**如果它们不交互，不确定性可以被接受。**

比如：

```js
var res = {};

function foo(results) {
	res.foo = results;
}

function bar(results) {
	res.bar = results;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo()`和`bar()`是两个并发“进程”，并且谁先运行是不确定的，但是我们的程序里它们谁先执行没所谓，因为它们是独立的，所以也不会相互作用。

这不是“竞态条件”的问题，因为代码总是正常运行，无论哪种顺序。

### Interaction

一般的，并发“进程”都会互相影响，间接地通过作用域或者DOM。当这种相互作用发生时，你需要调整这些作用，防止出现如前面提到的“竞态条件”。

这里是一个简单的因为隐含的顺序导致互相作用的并发“进程”的示例，*有时会出问题*：

```js
var res = [];

function response(data) {
	res.push( data );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```
并发“进程”是两个处理Ajax响应的`responce()`回调，它们会出现谁先执行的问题。

假设我们期待的是`res[0]`有请求`"http://some.url.1"`的结果，`res[1]`有`"http://some.url.2"`的结果。有时的确是这种结果，但是有时，它们可能正好相反，取决于那个请求先完成，有很大的概率这个不确定性时一个“竞态条件”bug。

**注意：** 千万小心这种场景下你可能作出的假定，比如，这种情况也很常见，开发者观察到`"http://some.url.2"`总是比`"http://some.url.1"`响应的慢，或许由于它们执行的任务不同（比如：一个获取数据库数据另一个只是获取静态文件），导致观察到的顺序似乎总是所期望的，即使两个请求都是指向同一个服务器，而*它*正好以特定的顺序响应，但是*不能*保证响应是按顺序到达浏览器的。

所以，为了处理这个竞态条件，你可以通过调整作用顺序来解决：

```js
var res = [];

function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

无论哪个Ajax响应先返回，我们检查`data.url`来确定应该把数据存储在`res`的什么位置，`res[0]`总是持有`"http://some.url.1"`的结果，`res[1]`持有另一个。通过简单的调整，我们消除了一个竞态条件的不确定性。

这种场景的分析可以推广到多并发函数回调通过共享的DOM相互作用的场景，比如一个更新`<div>`里的内容，一个更新样式或别的属性，你一定不愿意DOM在还没有内容的时候就出现，所以调整必须保证作用顺序。

有些并发场景如果不做调整将会*总是出错*（不仅仅是*有时）：

```js
var a, b;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(y) {
	b = y * 2;
	baz();
}

function baz() {
	console.log(a + b);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

在这个例子里，无论`foo()`或着`bar()`谁先触发，都会导致`baz()`过早运行（`a`或`b`总有一个是`undefined`），但是第二次运行`baz()`不会有问题，因为`a`和`b`都有了值。有很多方式来解决这个问题，比如这种简单的方式：

```js
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log( a + b );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`baz()`里的`if (a && b)`条件判断习惯上叫做“门”，因为我们不确定`a`和`b`哪个先到，但是我们会等到它们到后再开门（调用`baz()`）。

另一个你会遇到的并发交互条件有时叫做“竞争”，但是其实准确的应该叫“锁“，它的特点就是”只有第一个会赢“。这时，不确定性是可以接受的，因为你明确地说对”竞争“来说，先到终点的只有一个赢家也没问题。

考虑这段糟糕的代码：

```js
var a;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(x) {
	a = x / 2;
	baz();
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

无论哪一个先触发，最后触发的不仅会重新给`a`赋值，而且会再次调用`baz()`（这可能不是你期望的）。

所以，我们可以通过一个锁来调整相互作用，只让第一个通过：

```js
var a;

function foo(x) {
	if (a == undefined) {
		a = x * 2;
		baz();
	}
}

function bar(x) {
	if (a == undefined) {
		a = x / 2;
		baz();
	}
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```

`foo()`或`bar()`里的`if (a == undefined)`条件只允许第一个通过，第二个（以及后面的）都会被忽略，第二位到来的没有任何好处。

**注意：** 在所有这些场景里，我们为了简单说明问题起见，使用了全局变量，但是实际上并不是说必须用全局变量。只要函数可以访问到那些变量（通过作用域）就可以。依赖词法作用域上的变量（见系列中名为*Scope & Clousures*的那本），例子中的全局变量，都是一个明显的对并发调整的不优雅做法。当我们阅读完接下来其它章节，我们会看到其它更清晰的调整方式。

### Cooperation

另一个并发调整的方式就是“协同并发”，这里的重点不在于作用域中共享值导致的互相作用（尽管那个仍然是被允许的），将长时运行的“进程”划分为步骤或者批次，从而其它并发“进程”有机会把它们的操作交叉到事件轮询队列里。

举个栗子，有一个Ajax响应处理器，它需要遍历一个长的结果列表并转换值，我们将使用`Array#map(..)`来保证代码简短：

```js
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `data` values doubled
		data.map( function(val){
			return val * 2;
		} )
	);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

如果`"http://some.url.1"`首先拿到了结果，整个列表会立刻被映射到`res`里，如果有几千的记录，没什么大不了的，但是如果有千万记录，那将会消耗一定的时间（一个强大的笔记本大概需要几秒钟，移动设备需要更多，等等）。

尽管这个“进程”在执行，但是页面上没有任何变化，包括没有其它`response(..)`被调用，没有UI更新，甚至没有用户事件比如滚动、输入、按钮点击等，这会很让人不愉快。

所以，为了更协同的并发系统，需要这样一个更加友好的，不会挂起事件轮询队列的，你可以异步分批次的处理结果，当一个完成后，通知事件轮询队列，让其它事件继续处理。

这里有一个简单的方法：

```js
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// let's just do 1000 at a time
	var chunk = data.splice( 0, 1000 );

	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `chunk` values doubled
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// anything left to process?
	if (data.length > 0) {
		// async schedule next batch
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

我们将数据集处理成最大1000个元素的块，这样，我们保证了短时“进程”，尽管这意味着许多随后的“进程”，随着交叉到事件轮询队列，为我们提供了更具有响应（性能）的网站／应用。

当然，我们没有调整交互来改变这些“进程”的顺序，所以`res`里结果的顺序也是不可预测出的，如果需要顺序，那就需要交互技巧比如我们前面提到的，活着我们在后续章节会涉及。

我们使用`setTimeout(..0)`（hack）来实现异步调度，基本上意味着“将这个函数放到当前事件轮询队列的末尾”。

**注意：** `setTimeout(..0)`严格来讲不是直接把元素放到事件轮询队列里，定时器会在它下一次被执行时插入该事件。举个栗子，两个随后的`setTimeout(..0)`调用实际上无法保证会被按照调用顺序来处理，所以你会看到像这种定时器驱动的不同条件下，事件的执行顺序时不可预测的。在Node.js里，类似的方法是`process.nextTick(..)`。不论它多么方便（通常也性能更好），没有一个直接的方法（至少现在还没有）可以跨环境保证异步事件的顺序，下一节我们会讨论这块儿的细节。

## Jobs

到了ES6，出现了在事件轮询队列基础上的新概念，叫做“作业队列“。通过使用Promise的异步特性（见第3章），你会全面接触到它。

可惜，到目前，它还只是一个没有暴露API的原理，说明它还是有点儿复杂的。所以我们将会从概念上描述它，在第3章通过Promise来讲解异步特性时，你会理解这些活动是如何计划和处理的。

所以，我发现理解“作业队列”的最好方式就是，它是在事件轮询队列里每一次tick里会挂断的队列。确定的隐含异步的活动在一次事件论序tick里，不会导致整个事件被添加到事件轮询队列，而是添加一个元素（即作业）到当前tick的作业队列。
So, the best way to think about this that I've found is that the "Job queue" is a queue hanging off the end of every tick in the event loop queue. Certain async-implied actions that may occur during a tick of the event loop will not cause a whole new event to be added to the event loop queue, but will instead add an item (aka Job) to the end of the current tick's Job queue.

就好比说：“哦，这里有一些我*一会儿*要做的事情，但是确保在它在其它事件之前发生。”
It's kinda like saying, "oh, here's this other thing I need to do *later*, but make sure it happens right away before anything else can happen."

或者，使用比喻：事件轮询队列就像游乐场，一旦你完成了一次，你必须回到队伍的最后面重新开始，而作业队列就好像完成一次，但是插入队伍，恢复正常。

一个作业可以导致更多的作业被添加到同一队列，所以，理论上一个作业可能无限“循环”（一个作业持续添加另一个作业，等等），这样就阻止了程序前进到下一个事件轮询tick。这个在概念上和长时间运行或者无限循环（比如`while(true) ..`）是相同的。

作业有点儿像`setTimeout(..0)`hack，但是实施时却是这样一种良好定义并且保证顺序的方式：**随后，但是尽快**。

让我们假设一个调度作业的API（直接的，无须hack），并且计作`schedule(..)`，如：

```js
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// theoretical "Job API"
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```

你可能会期望输出`A B C D`，但实际上输出的时`A C D B`，因为作业发生在当前事件轮询tick的末尾，当定时器触发去调度*下一个*事件轮询tick（如果有的话！）。

在第3章，我们会看到Promises的异步特性是基于作业的，所以搞清楚它和事件轮询的关系很重要。

## Statement Ordering

我们写在代码中的语句顺序不一定就是JS引擎的执行顺序，这似乎是一个很奇怪的言论，所以我们来简单的探索一下。

在这之前，我们应该搞清楚一些事情：这门语言的规则／语法（见本系列中*Types & Grammer*那本书）使得语句的顺序从编程角度来讲是可预测且可信赖的行为成为可能。所以我们将要讨论的不是你在JS程序里**看到的**。

**警告：** 如果你曾经*观察*过像我们将要说明的被编译器重排序的语句，那是对规格书的违背，毫无疑问时因为JS引擎的bug，应该立即被报告并且修复！但是很可能是你*怀疑*一些疯狂的事情发生在JS引擎，实际上只不过时你代码里的bug（可能就是一个“竞态条件”），所以仔细的看看。JS调试器，使用断点和一行行的步进，会是很强大的对*你代码*里的bug的嗅探工具。

思考：

```js
var a, b;

a = 10;
b = 30;

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

这段代码里没有异步表达（也没有像前面提到的`console`异步I/O的竞争），所以大多数可能假设，它会按行从上执行到下。

但是*可能*JS引擎，在编译这段代码（是的，JS会被编译——见系列中那本名为*Scope & Closure*的书）后，发现可以通过（安全地）重新排列语句的顺序让你的代码运行的更快。本质上，只要你无法观察重排序，所有的事情都是公平的。

举个栗子，引擎发现如果代码这么执行会更快：

```js
var a, b;

a = 10;
a++;

b = 30;
b++;

console.log( a + b ); // 42
```

或者这样：

```js
var a, b;

a = 11;
b = 31;

console.log( a + b ); // 42
```

或者甚至这样：

```js
// because `a` and `b` aren't used anymore, we can
// inline and don't even need them!
console.log( 42 ); // 42
```

在所有这些情况下，JS引擎在编译时会进行安全的优化，最终*可观察的*结果是一样的。

但是在这种场景下，上面的这些特定的优化可能并不安全，从而不被允许（废话，那样的话就不是优化了）：

```js
var a, b;

a = 10;
b = 30;

// we need `a` and `b` in their preincremented state!
console.log( a * b ); // 300

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```

其它的例子中，编译器重排序后会生成可观察到的副作用（从而不会被允许）包括任何函数调用的副作用（甚至getter函数），活着ES6代理对象（见*ES6 & Beyond*那本书）。
Other examples where the compiler reordering could create observable side effects (and thus must be disallowed) would include things like any function call with side effects (even and especially getter functions), or ES6 Proxy objects (see the *ES6 & Beyond* title of this book series).

思考：

```js
function foo() {
	console.log( b );
	return 1;
}

var a, b, c;

// ES5.1 getter literal syntax
c = {
	get bar() {
		console.log( a );
		return 1;
	}
};

a = 10;
b = 30;

a += foo();				// 30
b += c.bar;				// 11

console.log( a + b );	// 42
```

如果不是`console.log(..)`语句（只是为了方便表示可观察的形式），JS引擎很可能会或者已经重排列了代码：

```js
// ...

a = 10 + foo();
b = 30 + c.bar;

// ...
```

尽管JS机制保护我们免受*可观察*噩梦，因编译器重排序语句，理解源代码编写和编译后的运行之间的联系多么薄弱很重要。

编译器语句重排序几乎是并发和互相影响的微隐喻，作为一个普遍概念，清楚它可以帮助你更好的理解异步JS代码流里的问题。

## Review

JS程序通常划分为两个或更多的块，第一个块运行在*当前*其他的块作为事件响应在*后来*运行。即使程序是块接着块执行的，它们共享同样的程序作用域和状态，所以每一个状态地改变都是基于前一个状态。

无论何时，只要有事件在运行，那么*事件轮询*就会一直运行到队列为空。每一个事件轮询的迭代就是一个"tick"。UI、IO和定时器给事件队加入事件。

在任何给定时刻，一次只能有一个事件可以被处理，当一个事件被执行时，它可能直接或间接的触发一个或多个后续的事件。

并发是当两个或多个事件链随着时间交织到了一起，这样从高层级去看，它们似乎是*同时*在执行（即使特定的事件一次只有一个事件在被处理）。

通常很有必要针对并发“进程”（和操作系统的进程明显不同）进行一些形式的交互调整，比如为了保证执行顺序或者避免“竞态条件”，这些“进程”可以通过划分成更小的块来*协同*从而允许其它“进程”插入。
