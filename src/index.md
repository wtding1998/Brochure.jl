# Julia编程指南

Julia语言在2018年正式发布了第一个长期支持版本（1.0版本），在这之后市面上出现了很多中文教程。但是，
为什么我还要编写本书呢？有如下的一些原因：

1. **市面上的中文书籍和教程主要都以入门Julia语言为主。**但是还没有一本**由有经验的Julia语言开发者**所编写的书籍。
2. 我们在[中文社区的论坛](https://discourse.juliacn.com)，QQ群等中文媒体上经常可以看到中文用户抱怨没有好的教材可以参考，缺乏可以学习的代码范例，并且中文网络上依然大量充斥着旧版Julia的代码（Julia 0.6甚至是更加古老的版本）。**这些旧的代码随着1.0版本的发布已经不再支持，对很多Julia学习者造成了困扰。**
3. 我发现由于Julia语言编程范式和其它流行语言（例如Python，C++，MATLAB）有很大不同，有很多Julia学习者在阅读过文档之后，即便已经学会了基本语法，**大部分Julia学习者入门之后依然无法很好的编写一个完整的，符合Julia范式的代码库。**
6. 我们有时候还是需要一些文档里不会讲的内容，比如一些经验性质的总结，一些更加和某个领域相关的完整工程展示。所以这也是这本书的目的之一：**提供编写Julia工程的实践范例**
7. 我在[知乎上写过很多零散的关于Julia语言的文章](https://zhuanlan.zhihu.com/halfinteger)，有很多人建议我将它们整理到一起，但是我不是很喜欢知乎的编辑器，它并不适合编写长篇的技术文章，对数学公式和代码的支持非常差。而微信公众号则更加糟糕，在经过调研后我觉得还是**需要使用开源工具自己来做这件事情。**
8. 大部分的书籍依然是传统的纸质媒体，或者以纸质媒体为前提进行编写。这在现在这个互联网和富文本时代是非常落后的。我希望以此为媒介，做一些现代书籍的实验。


本书使用纯Julia进行编写，除了构成静态网页的js/html/css 脚本以外，**这本书的Julia纯度为：100%**，当你下载这本书之后，可以用Julia的编译器运行下面这行命令

在命令行里

```sh
julia make.jl
```

或者打开Julia的REPL，然后运行

```jl
include("make.jl")
```

这本书就会以网页的形式挂在到 [localhost:8000](http://localhost:8000/index.html)，它使用了 [Documenter](https://github.com/JuliaDocs/Documenter.jl) 和 [LiveServer](https://github.com/asprionj/LiveServer.jl) 这两个Julia包进行编译和挂载。如果你喜欢黑夜模式（dark mode）
你还可以点击右上角的齿轮按钮选择黑夜模式。

## Julia语言的定位是什么？

Julia是主要用来做科学计算吗？是的，Julia主要用来做科学/技术计算（Scientific Computing），实际上在1.0以前Julia的REPL启动之后都有一句：A Fresh Approach To Technical Computing。这意味着社区的主要关注点在科学计算上，但这不是说Julia只能做科学计算。实际上构建网站，挖比特币，图形界面等等都有相关的package。但是我们说社区的主要关注点在科学计算，所以相比诸如Python/Go/Erlang的这些类似功能，Julia的这些功能更多的是为了服务于科学计算，而不是和已有的软件同台竞争。

但Julia本身的定位和很多其它语言一样：**一个通用编程语言。**

## 为什么要设计一个新语言？原先的解决方案有什么问题？

我知道很多人提到Julia都想到性能。但是我不想从性能讲，因为Julia并不单纯的是要解决性能问题，Julia要解决的是一个有前提的性能问题：如何在让程序足够通用（generic）的前提下，尽可能提供最好的性能。Julia为了解决这个问题成为了第一个为JIT编译设计的语言，也是第一个实现了高性能的多重派发（multiple dispatch）的语言。

当你开始使用Julia以后，你会发现不同于Python/MATLAB/C++/C的是，大部分时候我们只要关心数学对象，我们写的代码都非常的generic，于是非常多的功能通过多重派发和类型系统只要数学上讲得通，就自动工作了，不需要写额外的代码把他们组合在一起。例如曾经在社区里发生过一个很经典的事情：DifferentialEquations的作者有一天发现，有一个他完全不认识的人写了一个误差分析的库（Measurements），定义了一种数，带误差上下界，结果放进微分方程的求解器里什么都没有算，就自动能够工作用来分析误差。

所以有很多Julia用户的观点是：即便Julia的性能不好（其实挺不错的），这种新的编程范式本身就足够吸引人了。

这个问题我在之后还会更加详细的以长篇幅讨论，如果你想现在阅读也可以直接跳到这部分：

## Julia官方宣称自己很快，但是再快能有C/C++/Fortran快吗？

首先我们要解释一下误区，我遇到过很多人都持有这样的观点：如果我用一个静态编译语言实现我的功能，它就会很快。这是错误的，一段代码快不快并不取决于它使用什么语言编写，而是取决于它具体做了什么，它所实现的算法时间复杂度是多少。在很多情况下，用静态语言写的实现更快只是因为静态语言的编译机制强制你写出能够被编译器充分优化的代码（但有时候也许没有必要）。

比如 AutoTVM 可以用Python的API找出一个比MKL快的kernel，这个算不算Python更快呢？所以实际上，如果你不在乎性能，Julia可以像Python解释器一样慢，但是如果你在乎性能，不同于Python的是Julia也为优化提供了充分的可能性。例如你可以用纯Julia，以高级语言的特性实现BLAS的gemm routine，它的性能可以接近手工调整汇编指令的OpenBLAS。而注意这个矩阵乘法的实现本身是一般的，它没有用到任何特殊的语义，它是使用Julia本身的语法表达了矩阵的分块等语义。也就是说它不仅可以使用32和64位浮点，对于其它的Julia类型也能够工作。这就是我们说的在代码足够一般的前提下，依然能够获得非常好的性能。

![JuliaBLAS-benchmark](https://user-images.githubusercontent.com/17304743/53292180-c9142300-378c-11e9-9f41-8c10711af122.png)

矩阵乘法能够在一定程度上说明即便是稍微复杂一些的任务，Julia也能够拥有不错的性能。而实际上由于社区本身对科学计算的重视，我们会发现当我们实现一个完整的工程之后，往往要比同类软件的C++版本快。这往往是因为在Julia里已经有了丰富的基础生态（例如各种特殊矩阵算法的实现，各种线性代数算法的实现）同时实现各种特化的算法代价又非常的低，同样的接口组合下不同的类型就可以了，最后导致的结果就是即便编译器本身做的优化并不如GCC/Clang，最终的效果却可能更快。

当然了，说到这里有人可能还是不信，[我最近刚好在给我们的量子计算框架做benchmark](https://github.com/Roger-luo/quantum-benchmarks)，其中 [ProjectQ](https://github.com/ProjectQ-Framework/ProjectQ) 是一个非常标准的 [pybind11 C++/Python](https://github.com/pybind/pybind11) 程序，使用了C++大量优化了性能，并且是第一个实现量子霸权模拟(45 比特的全概率幅模拟）的软件。而 [qulacs](https://github.com/qulacs/qulacs) 和 [QuEST](https://github.com/quest-kit/QuEST) 则是使用了大量C++进行优化了的高性能模拟器。但是我们依然能够通过精心设计的软件架构，大量的使用通用性的特化代码，用纯Julia就实现了更好的性能和更多的功能

![parameterized](https://github.com/Roger-luo/quantum-benchmarks/raw/master/images/pcircuit_relative.png)

而实际的代码量却远远小于C++的实现。

## 既然多重派发这么好，为什么在Julia之前没有语言大规模使用？

实际上并不是这样，在Julia之前有非常多的语言社区都尝试实现过多重派发，甚至曾经出现过以多重派发为主要范式的语言。但是他们都有一个严重的问题：这些多重派发的实现性能都不好，以至于最终无法应用到实际的生产环境里去。Julia的出现也是站在这些前辈的探索之上的。

那么Julia是如何解决这个问题的呢？这要说到Julia最大的特点：对JIT编译友好的语言设计。Julia在设计之初就以JIT编译为假定，而非解释执行或者静态编译（AOT）。
完整的多重派发本身是一种动态特性，这是因为变量的类型并不一定能够完全在编译时期确定，所以我们也说类似C++的函数重载是静态的多重派发，但是C++本身并不支持动态的多重派发，实现类似的功能需要虚表。而Python社区里也有多重派发的实现，但是Python由于本身是以解释器为基础设计的，所以这些多重派发的实现本质上也都只是存储了一个动态的方法表，然后在运行时期来查找。这就导致性能甚至比使用Python自带的class的单重派发+继承还差。类似的在其它语言里也有类似的实现，他们或者因为静态编译，对动态的多重派发支持不好，或者因为本身是动态地解释执行，最终的性能很差。

而Julia通过结合JIT解决了这个问题，Julia的编译过程是动态的，每次编译发生在函数被调用之前。这就意味着当编译器能够正确推导出类型的时候，我们就可以进行静态派发，当编译器无法推导类型的时候，我们自动进行动态派发。这表现为Julia语言的类型稳定性（type stability）。而所有的编译都以函数为界。这也就是说，如果函数 f1 是类型不稳定的，但是实际的程序中它的输出输入到一个类型稳定的函数 f2里，Julia依然能够被静态编译f2，使得f2内部的函数派发变成静态的。


## Julia解决了什么问题？

想想我们一开始为什么会用Python/MATLAB，他们起初也没什么生态不是么？但是它们在十年二十年前解决了科学计算的什么痛点呢？
用Fortran95或者C/C++（除去C++11以后的版本，这个后面另外讨论）写出来的代码通用性差，往往只能针对特别的问题，编写程序的过程中，因为要不断的考虑类型的问题，我们往往花了很多的时间在思考如何写出能够让编译器通过的代码这件事上。而且非常多的场景下，我们只是需要一个类似计算器的东西，给你几个矩阵，告诉我结果就行了。我们不想去思考内存布局，类型是 `A<Float32, Int>` 还是 `B<Float32, C<Float32>>` 。甚至连面向对象都懒得管。至于具体的语法细节，什么goto，范型等其实都不是那么的重要（有很多替代方案），甚至如果你想在 Fortran95 中使用泛型（generic type）都是可能的。

Python/MATLAB 通过使用动态性，在一定程度上解决了以上问题。它们都可以简单地被当作计算器来用，快速的告诉你想知道事情，而不需要在乎类型推导是否能够通过，也不需要等待重新编译的时间。而说到底在我们编写的很多程序中，有一大部分的性能是无所谓的，我们只需要一些关键的地方足够快就行了，所以解释执行也是没有关系的。而Python本身的class实际上由于其动态性，它可以被当成鸭子类型（duck type）来使用的。每个class都可以动态的插入方法。所以当我们判断一个类型是什么的时候我们不再考虑这是什么类型，而是相对的考虑这个类型能做什么事情。这为很多代码带来很强的通用性，这意味着我不必在乎这个类型是谁的子类，反正它能做这件事（比如某个类型是具有对称性的，它是一个Hermitian的算符），那它就能够输入给任何只使用这个接口的函数。如果我写着写着，突然发现我需要用这个方法，我临时往我的类型里插入这个API也都是可以的。这是Python的动态性带来的方便之处。

再比如，如果有在诸如C++这类静态语言里实现过多位数组的同学也许会注意到，我们很多时候希望能够在模板参数里带上一个维度作为参数，这样可以利用类型推导减少很多不必要的check，以及保证程序的正确性，并且通过对类型参数的特化来派发一些针对性的优化，甚至更加激进一些，我们可以把shape也放进模板里，这样在维度不大的时候展开for循环，手动编写针对小矩阵的优化。

但是这样的静态特性在使用的时候又反而带来不方便，强制运行的类型推导有时候给一些小任务带来了很大的负担。例如我们很多人都用过Python的numpy，里面有一个einsum的功能，它提供了爱因斯坦求和标记来计算多个高维张量（也就是ndarray）的缩并，或者说求和。那么这个计算结果是一个张量，这个张量的大小和维度信息有可能是动态计算出来的，也有可能是能够在编译时期推导出来的。

这在Fortran和C/C++里都是不行的或者说想要实现类似的特性是更加麻烦的。

但是，对性能需求大的部分，我们就需要精心设计了，过于动态的语义虽然方便，但是它使得编译器能够做的假设就更少。而编译器想要优化代码，必须要有足够的假设来证明优化前后的代码是等价的才行。所以我们选择用语义上能够提供这些信息给编译器的语言来编写。

这就是著名的两语言问题。我们用一种脚本语言提供方便的接口，用一个静态语言编写有性能需求的地方。这确实在很长一段时间，甚至直到现在都能够很好的解决很多问题。但是同时也造成了很多的问题，尤其是代码的通用性。

例如，我们在MATLAB或者Python的CPU上的矩阵乘法往往是通过调用使用汇编优化的，这些代码分别针对两种浮点数（32和64）进行了优化。但是它们不能在其它类型上工作，并且组合不同类型的矩阵，由于缺乏足够一般（generic）的实现，导致我们需要大量的人工工作。但是反观Julia，一个典型的例子就是大量的自定义数组。我常常说：Julia有现在所有语言里最好的矩阵和多维数组生态。让我简单列举一下：

#### 标准库

- LinearAlgebra：定义了几个常用的矩阵类型 Hermitian，Symmetric，Diagonal

#### 特殊数组
- StaticArrays 栈上的静态数组：通过编译器提供高性能的小矩阵计算
- FillArrays 一些特殊的稀疏矩阵，例如零矩阵，全是某个数的矩阵等等
- InfiniteArrays 无穷维矩阵/数组
- BlockArrays 分块数组
- LazyArrays 懒惰求值的Array
- AxisArrays 每个维度可以带标签的数组（也就是PyTorch所谓的NamedTensor）

#### 特殊矩阵（2维数组）
- ToeplitzMatrices 
- BandedMatrices
- SpecialMatrices：这个包里实现了一些特殊矩阵包括 Cauchy，Circulant，Campanian，Frobenius等十一种特殊矩阵（太多了就不一一列举了）
- CovarianceMatrices
- LuxurySparse（这是我们自己开发的）：PermMatrix（广义交换矩阵），各种静态数组和上面的特殊矩阵组合的一些特化的实现

#### 针对分布式和多进程的数组
- 标准库里的：SharedArray，可以在进程中共享内存
- DistributedArray，分布式的（稠密）数组，支持一些基本操作例如分布式乘法

为什么会有这么多矩阵和数组，因为在Julia里实现一个特殊矩阵的代价非常低，我们很容易就能做到让一个新的矩阵类型加入到整个生态里去，然后拥有包括基本的线性代数在内的各种功能。

而这些数组和矩阵并没有特别地为很多其它算法实现什么，例如指数矩阵乘向量（expv）算法在ExponentialUtils里只假定这是一个数学上讲得通的线性映射，只要你能使用乘法，那么就可以在这个算法里运行，并且编译器会通过类型推导特化你的代码，从而获得最优的性能。而作者在实现ExponentialUtils的时候并不需要知道这个具体是一个什么矩阵。

所以多重派发是Julia高性能的基石，也是为什么大家喜欢用Julia写科学计算代码的主要原因之一。它确实大大降低了工作量。
