#  WebAssembly介绍




-------------------

[TOC]

## 简介


>WebAssembly or wasm is a new portable, size- and load-time-efficient format suitable for compilation to the web.

以上是[WebAssemby](http://webassembly.org/)官网对WebAssemby的定义，翻译过来的大概是这个意思

>WebAssembly(简称wasm)是一种精简的、大小和加载时间高效的数据格式，适合编译到Web上运行。

这句话的几个关键词概括出了wasm的一些特点。

加载高效：WebAssembly被设计成体积很小的二进制格式，所以加载高效。 

Web上运行：WebAssembly 是除了 JavaScript 以外，另一种可以在浏览器中执行的编程语言。

总结起来，WebAssembly是一种二进制格式的类汇编语言，可以被浏览器加载，并进一步编译成可执行的机器码，从而在客户端运行，因此的执行效率比javascript高很多。
##为什么会出现WebAssembly
随着现代WEB应用的复杂度逐渐变高，对JavaScript的执行效率提出了更高的要求，而JavaScript作为一门解释性语言，性能是存在瓶颈的。这也是WebAssembly出现的主要原因。下面我们分析下JavaScript的性能瓶颈主要在哪几个点上。

高级语言一般可以分为解释性和编译性语言两种，解释性语言通过解释器翻译成机器码，而编译性语言通过编译器翻译成机器码。

解释器启动和执行的更快。不需要等待所有代码解释完成就可以运行你的代码。从第一行开始翻译，就可以依次继续执行了。但是当你运行一个循环，同样的代码需要多次运行时，解释器会一次一次的解释这段代码，导致其效率低下。

而编译器和解释器正好相反，它会首先将整个文件的源代码编译成目标代码，所以并不会重复去编译循环中的代码。并且编译器在编译的过程中可以对代码进行优化，所以执行效率会高很多。但是其启动执行时间会慢一些。

为了解决JavaScript的低效问题，很多JavaScript解析引擎将Just-in-time（简称JIT）编译器引入了JS引擎之中，形成了一种混合引擎，综合解释器和编译器的优点。

主要的一个原理是，JavaScript引擎中会引入一个监视器，当监视器发现同一行代码被运行了很多次之后，就会对这行代码做上标记，并把这行代码送到基线编译器里去编译，把结果存储起来。下一次当监视器发现同一行代码再次执行时，就不需要对它进行解析，而是将已经编译好的结果传给浏览器执行即可。通常这样的做法都可以提高代码的执行效率。

上文我们提到，编译器是可以对代码进行一定程度的优化的，当一段代码被执行了多次执行之后，就可能被送到优化编译器里进行优化，并将结果存储起来。优化后的代码段再次执行之前，编译器会检查此次代码是否与优化代码一致，如果发现变量类型变化的情况，那么编译器便会把优化的代码丢弃，重新对代码进行解析，这个过程叫做“去优化”。

总的来说，虽然浏览器引入JIT可以大大提高JavaScript的执行性能，但是也会增加一些开销：
1、优化和去优化开销；
2、监视器记录信息对内存的开销；
3、发生去优化情况时恢复信息的记录对内存的开销；
4、对基线版本和优化后版本记录的内存开销。

所以JavaScript还是有优化的空间，这是WebAssembly出现的一个主要原因。

##WebAssembly的工作原理
我们首先介绍下高级语言到汇编语言的一个翻译过程。
高级语言翻译成汇编语言的流程如下图（出处见底部）：

![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/02/04-01-langs09.png)

编译器的front-end部分首先会将高级语言翻译成一种中间代码（intermediate representation，IR），而后编译器的back-end部分将IR翻译成对应目标机器的汇编代码。不用的机器结构会对应不同的汇编语言的结构（x86、ARM）。所以一种汇编语言是不能在不同机构的机器上执行的，一种汇编语言对应一种机器结构，也就是说在代码到达客户端之前，我们不能将其编译成汇编代码，因为需要知道客户端的机器结构才能够正确的将代码编译成对应的汇编代码。

而WebAssembly 与其他的汇编语言不太一样，它不依赖于具体的物理机器。可以抽象地理解成它 是概念机器的机器语言，而不是实际的物理机器的机器语言。浏览器把 WebAssembly 下载下来后，可以迅速地将其转换成对应机器汇编代码。 WebAssembly 在代码编译过程的位置如下图(图片来源见底部)所示：

![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/02/04-02-langs08.png)

生成 WebAssembly 的方式有多种，可以直接手写，因为 WebAssembly 提供了文本形式，写起来跟汇编差不多（如果你愿意写的话233）。更通行的方式是将用其它语言——目前主要是静态语言（C、C++、Rust等）编写的代码编译成 WebAssembly（.wasm），编译工具最主要的是 LLVM。

LLVM 编译的基本工作机制是：首先使用一种针对特定语言的插件（类似于 webpack 中的 loader）将该语言编译为一种中间态形式（IR），然后再由 LLVM 对 IR 进一步编译、优化，从而得到.wasm。当然也有其它的编译工具，如 Emscripten、Binaryen 等。

目前 .wasm文件 需要由 JS 引入后才能运行，JS 中有一个用于操作二进制代码的 API：ArrayBuffer，JS 使用 ArrayBuffer 加载 .wasm文件，然后调用编译方法，然后再创建实例。WebAssembly 还没有集成 Web API，要调用 Web API，就必须借助 JS。未来计划允许 WebAssembly 直接调用 Web API，并且让 .wasm文件模块像 ES6 模块一样易于使用。

##为什么WebAssembly会更快
JavaScript解析执行过程大致如下图所示：
![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/02/05-01-diagram_now01.png)

1、Parsing——表示把源代码变成解释器可以运行的代码所花的时间；
2、Compiling + optimizing——表示基线编译器和优化编译器花的时间。一些优化编译器的工作并不在主线程运行，不包含在这里。
3、Re-optimizing——当 JIT 发现优化假设错误，丢弃优化代码所花的时间。包括重优化的时间、抛弃并返回到基线编译器的时间。
4、Execution——执行代码的时间。
5、Garbage collection——垃圾回收，清理内存的时间。

WebAssembly的执行过程与JS执行过程对比图如下图：
![](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2017/02/05-03-diagram_future01.png)

**文件获取**：图中没有展示这个过程，压缩后的 WebAssembly 的二进制代码比压缩后的JS代码小，使得其在服务端和客户端之间的传输时间会短一些。

**解析**：JS在被执行之前是需要先转化成中间代码的，而WebAssembly本身就是就是一种中间代码，所以并不需要这个过程。

**编译和优化**：JS是弱类型语言，当传入的变量发生类型变化时，同样的代码是需要编译成不同的版本的，而WebAssembly不存在这个问题，此外在编译优化代码之前，它不需要提前运行代码以知道变量都是什么类型。这都使得WebAssembly编译和优化过程比JS快很多。

**重优化**：前面提到过，JS在优化的过程中是存在“去优化”的过程，所以去优化之后的代码可能会被“重优化”，反复做几次无用功，导致优化资源被占用，降低执行效率。在 WebAssembly 中，类型都是确定了的，不会存在这样的问题。

**垃圾回收**：JavaScript 中，开发者不需要手动清理内存中不用的变量。JS 引擎会自动地做这件事情，这个过程叫做垃圾回收。目前的大多数浏览器已经能给垃圾回收安排一个合理的启动时间，不过这还是会增加代码执行的开销。而WebAssembly 不支持垃圾回收。内存操作都是手动控制的（像 C、C++一样）。这对于开发者来讲确实增加了些开发成本，不过这也使代码的执行效率更高。不过这一点仁者见仁了。

以上就是为什么WebAssembly比JS执行效率更高的原因。

**那么WebAssembly究竟有多快呢？**

Kamaron Peterson做了一个测试([链接](https://github.com/sessamekesh/wasm-3d-animation-demo))
他分别使用WebAssembly和JS写了一个3D的动画效果
![](https://d2slcw3kip6qmk.cloudfront.net/marketing/blog/2017Q2/WASM1.png)

然后在不同浏览器和不同操作系统上测试了动画的运行时间，测试结果如下:
![](https://d2slcw3kip6qmk.cloudfront.net/marketing/blog/2017Q2/WASMTable.png)

从结果中可以看出，WebAssembly的执行效率大概是JS的十倍。

##WebAssembly的发展情况
WebAssembly是由W3C牵头，四大浏览器厂商（谷歌、火狐、微软、苹果）共同参与的项目，可以说是JavaScript性能问题的终极解决方案，前途可谓是一片光明。

2015.04  [WebAssembly Community Group](WebAssembly Community Group started) 成立
2015.06  第一个版本发布
2016.03  具有多个可互操作实现的核心特征的定义[[1]](https://blogs.windows.com/msedgedev/2016/03/15/previewing-webassembly-experiments)[[2]](https://v8project.blogspot.com/2016/03/experimental-support-for-webassembly.html)[[3]](https://hacks.mozilla.org/2016/03/a-webassembly-milestone/)
2016.10  浏览器preview宣布多个可互操作实现[[1]](https://blogs.windows.com/msedgedev/2016/10/31/webassembly-browser-preview/)[[2]](http://v8project.blogspot.com/2016/10/webassembly-browser-preview.html)[[3]](https://hacks.mozilla.org/2016/10/webassembly-browser-preview)
2017.02  官方logo发布
2017.03 [ Cross-browser consensus ](https://lists.w3.org/Archives/Public/public-webassembly/2017Feb/0002.html)以及浏览器preview结束

随着浏览器preview的结束，各大浏览器厂商分别宣布MVP版本的正式上线，这就表明浏览器已经可以稳定运行WebAssembly了，开发者可以在浏览器上加载WebAssembly。

WebAssembly未来的[规划](http://webassembly.org/roadmap/)

##WebAssembly会代替JavaScript嘛？
这个问题应该是很多JavaScript开发者非常关心的一个问题。
>No! WebAssembly is designed to be a complement to, not replacement of, JavaScript. While WebAssembly will, over time, allow many languages to be compiled to the Web, JavaScript has an incredible amount of momentum and will remain the single, privileged (as described above) dynamic language of the Web.

以上是官方给出的回答，大致意思是说WebAssembly的出现是对JS的一种补充，并不会也不可能替代JS，随着时间的推移，WebAssembly将允许将许多语言编译到Web上，而JavaScript作为一种运行在Web上的动态语言，仍然会保持着它的优势。

WebAssembly的出现其实是希望在Web应用程序的关键性能点对Web程序进行优化，采用WebAssembly编码对可能存在性能热点的地方进行重构，从而整体提高WEB程序的效率。例如，正在开发 React 程序的团队可以把虚拟 DOM、diff算法部分的代码替换成 WebAssembly 的版本，从而提高react渲染的效率。这样的一种混合方式并不会改变以JS为主的前端开发方式，但是可以大大提高性能。并且这样的WebAssembly代码之后会是以模块的形式出现，只需要在项目中引用模块即可，就像引入一个ES6的模块一样方便。

所以WebAssembly并不会也不可能取代JS。
##WebAssembly仅仅适用于C/C++开发者嘛？
虽然WebAssembly目前仅仅支持C/C++/Rust这几种语言进行编译，但是这么做的目的仅仅是为了尽快出了一个稳定的MVP版本而做出的折衷的选择。之后会继续支持其他语言的编译（[官方说明](http://webassembly.org/docs/high-level-goals/)）。


##参考链接##
1、http://webassembly.org/
2、[Creating and working with WebAssembly modules](https://hacks.mozilla.org/2017/02/creating-and-working-with-webassembly-modules/)
3、[What makes WebAssembly fast?](https://hacks.mozilla.org/2017/02/what-makes-webassembly-fast/)
4、[A crash course in just-in-time (JIT) compilers](https://hacks.mozilla.org/2017/02/a-crash-course-in-just-in-time-jit-compilers/)
5、[WEBASSEMBLY OVERVIEW: SO FAST! SO FUN! SORTA DIFFICULT!](https://www.lucidchart.com/techblog/2017/05/16/webassembly-overview-so-fast-so-fun-sorta-difficult/)
6、[WebAssembly 系列 知乎专栏](https://zhuanlan.zhihu.com/p/25800318)





