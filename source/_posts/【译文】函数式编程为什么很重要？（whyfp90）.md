---
title: 【译文】函数式编程为什么很重要？（whyfp90）
tags:
  - 文章翻译
categories:
  - 编程思想
date: 2020-06-20 23:42:21
---

> 作者： John Hughes [原文地址](https://www.cs.kent.ac.uk/people/staff/dat/miranda/whyfp90.pdf)
>
> 此论文作于1984年，作为查麦兹大学的备忘录流传了多年，经过小幅度修订的版本出现于1989年与1990年，即［Hug89］与［Hug90］。此版本基于原查麦兹大学备忘录的nroff源码，为LaTeX做了改动,使其更接近于印刷版本并纠正了少许错误。
>
> 本文由CloudiDust在[CSDN](http://blog.csdn.net/ddwn/article/details/984390) 最初翻译，由[YShaw](http:blog.yshaw.cn)根据原文重新校对、整理和排版，并添加部分注脚。

<!-- more -->

## 摘要

软件正在变得越来越复杂，因此良好的软件构架也越来越重要。结构良好的软件易于编写，易于除错，同时提供可复用组件库以降低未来开发的成本。传统型语言在程序模块化方面具有理念上的局限性，而函数式语言超越了局限。在本文中我们指出，函数式语言的两大特性，**高阶函数**与**惰性求值**[^2]，能够极大地促进模块化。作为例证，我们处理了列表和树，编写了一些数值算法，并实现了alpha-beta启发式搜索（一个人工智能算法，用于游戏系统中）。既然模块化是程序设计成功的关键，那么函数式语言对现实世界而言便极其重要了。

## 1.引言

本论文试图证明，对“现实世界”而言，函数式程序设计是极其重要的。同时，本文也试图明确指出函数式语言的长处，以帮助使用函数式语言的程序员们将这些长处发挥到极致。

函数式语言之所以被如此称呼，是因为程序完全是由函数组成的。主程序本身也是一个函数，以程序的输入为参数，并返回其输出。典型地，主函数通过其他函数定义，而这些函数又同样以更多的其他函数来定义，直到最低层的语言原生函数为止。这些函数与数学中的函数很相像，因此在本文中将以普通等式来定义它们。本文使用了Turner的程序语言Miranda中的表示方法，但对于之前没有函数式语言相关知识的读者，本文仍然是可读的。（Miranda是Research Software Ltd.的商标。）

函数式程序设计的特性与优点通常总结为类似这样：函数式程序**不包含任何赋值语句**，因此变量一旦被赋予一个值，就不再改变。更一般地说，函数式程序不包含任何副作用：**一个函数除了计算它本身的值以外，不产生任何作用**。这一特性消灭了“Bug”的一个主要来源，同时也使执行顺序不再重要——没有副作用能够改变一个表达式的值，故它可以在任何时刻被求值。这一特性将程序员从决定控制流的重担之下拯救出来。由于表达式可以在任何时刻被求值，程序员便可以随心所欲地使用变量的值来代替变量，反之亦然——也就是说，程序是“引用透明”的。这一自由使得函数式程序与它们传统的对应物相比，更容易数学化地控制。

这样的“优点”列表很不错，但如果说外行人不把它当回事，这也并不会令人惊讶。它列出了很多理由说明函数式程序设计“没有”什么（它没有赋值，没有副作用，没有控制流）但却没多说它“有”什么。函数式程序员听起来很像是中世纪的僧侣似的，他们禁绝了尘世中的种种乐趣并且期望这能使自己变得高洁。对于那些更关心物质利益的人而言，这些“优点”并没有多大的说服力。

函数式程序员们争辩说，函数式程序设计确实有巨大的物质利益——一个函数式程序员拥有比他传统型的同行高得多的生产力，因为函数式程序短得多。但这有什么道理吗？在这些“优点”的基础之上，唯一的很靠不住的借口就是，传统的程序中有90％是赋值语句，而在函数式程序中这些全都可以省略！这真是太荒唐了，如果省略赋值语句可以带来如此巨大的好处，那么FORTRAN程序员们早该这样干了二十年了。**通过省略特性来使语言更加强大在逻辑上是不可能的，不论这种特性是多么糟糕。**

函数式编程的程序员甚至应该对这些所谓的“优点”表示不满意，因为它们对于发掘函数式语言的威力毫无帮助，不可能据此写出一个完全缺少了赋值语句或**引用透明**[^1]的程序。因为没有衡量程序质量的标准，所以过于关注这些并不是一个理想的方式。

很明显，对函数式程序设计的特性的描述是不完备的。我们必须找出一些东西来填补——它们不但要解释函数式程序设计的威力，更要给函数式编程一个清楚完备的未来。

## 2.与结构化编程的类比

指出函数式与结构化程序设计之间的相似性是很有帮助的。过去，结构化程序设计的特性与优点被总结为类似这样：结构化程序不包含`goto`语句；结构化程序中的语句块没有多入口与多出口；结构化程序与它们传统的对应物相比，更容易数学化地控制。这些结构化程序设计的“优点”与我们之前所谈到的函数式程序设计的“优点”在本质上很相似。这些叙述本质上都是否定式的，从而导致了诸如“不可或缺的`goto`”之类一大堆徒劳的争论。

事后诸葛亮式地说，很明显地，结构化程序设计的这些特性，尽管很有用，但没有触及问题的核心。结构化程序与非结构化程序之间最重要的区别就是，**结构化程序是用模块化的方法设计的**。模块化设计带来了生产力的巨大提升：首先，小模块可以很快很容易地编写；其次，通用模块可以被重用，使以后的程序可以更快地开发；再次，程序的模块可以被独立测试，减少了除错的时间。

“不使用goto”等等这一类特性，对于这一提升没什么作用。这些特性促进了“程序设计的小改良”，然而模块化设计却促进了“程序设计的大进化”。因此，程序员在FORTRAN或汇编语言中都可以享受结构化程序设计带来的好处，哪怕那需要一点额外的工作。

**模块化设计是成功的程序设计的关键**，这一观点现在已经被普遍地接受了，而诸如Modula-II[Wir82]，Ada[oD80]以及Standard ML[MTH90]之类的程序语言都内置了语言特性以促进模块化。然而，有一点非常重要，却常常被忽略。当编写一个模块化程序以解决问题的时候，程序员首**先把这个问题分解为子问题，而后解决这些子问题并把解决方案合并**。程序员能够以什么方式分解问题，直接取决于他能以什么方式把解决方案粘起来。因此，为了能在观念上提升程序员将问题模块化的能力，必须在编程语言中提供各种新的黏合剂。复杂的作用域规则与对分块编译的支持只对文本层面的细节有帮助，它们没有提供能表达新观念的工具以分解问题。

通过与木匠行业的类比可以认识到黏合剂的重要性。先制作椅子的各部分——坐垫，椅子腿，靠背，等等——而后用正确的方法钉起来，那么制作一把椅子是很容易的。但这取决于将木板与插接口结合起来的能力。如果缺乏这种能力，那么制作椅子的唯一方式，就是将它从一大块木头里整个地切割出来，这是一项艰巨得多的任务。这个例子同时表明了模块化的非凡威力与拥有合适的黏合剂的重要性。

现在让我们回到函数式程序设计上来。在这篇论文余下的部分里，我们将指出，函数式语言提供了两种新的、非常重要的黏合剂。我们将给出许多可以使用新方法模块化的示例程序，它们因此变得很简洁。这就是函数式程序设计威力的关键——它允许了大幅改进的模块化设计。这也正是函数式程序员必须追求的目标——更小、更简洁、更通用的模块，用我们将要描述的新黏合剂黏合起来。

## 3.把函数粘起来

两种黏合剂中的第一种，使简单的函数可以聚合起来形成复杂的函数。以一个简单的处理问题来说明：将列表中的元素累加起来。我们用下面的语句定义列表：

```
listof X :: = nil | cons X (listof X)
```

这说明，一个元素类型为`X`的列表（不论`X`是什么），或是`nil`，代表一个没有元素的空列表，或是一个`X`与另一个X的列表的`cons`。`cons`代表一个列表，其首元素为`X`，而第二个以及后续元素即是另一个`X`的列表的元素。此处的`X`可以代表任何类型——例如，如果`X`是一个`Integer`（整数类型），那么这个定义就是说，一个整数列表，或者是空的，或者是一个整数与另一个整数列表的`cons`。依照通常的实践，我们写列表时，只是简单地将其元素包含在方括号里，而不是将`cons`和`nil`显式地写出来。方便起见，这是一个简单的速记法。例如：

```c++
[] 表示 nil
[1] 表示 cons 1 nil
[1 2 3] 表示 cons 1 ( cons 2 ( cons 3 nil ))
```

列表中的元素可以通过一个递归函数`sum`进行累加。`sum`必须为两类参数进行定义：一个空列表（`nil`），以及一个`cons`。由于没有数字存在时，累加结果是0，因此我们定义：

```c++
sum nil = 0
```

又因为`cons`的累加和可以通过将列表的第一个元素加到其余元素的累加和上的方式进行计算，所以可以定义：

```c++
sum (cons num list) = num + sum list
```

检查定义可以发现计算`sum`时，只有下面用粗体标出的部分是特化的：

> sum nil = **0**
>
> sum (cons num list) = num **+** sum list

这说明`sum`的计算可以通过一个通用的递归模式和特化的部分来模块化。这个通用的递归模式习惯上被称为“**递减**”（reduce），因此`sum`可以表达为：

```c++
sum = reduce add 0
```

方便起见，`reduce`传递了一个二元函数`add`。`add`是这样定义的：

```c++
add x y = x + y
```

只要将`sum`的定义参数化，我们便可以得到`reduce`的定义，即：

```c++
(reduce f x) nil = x
(reduce f x)(cons num list) = f num ((reduce f x) list)
```

这里我们写出了`(reduce f x)`两边的括号以强调它代替了`sum`。习惯上括号是省略的，因此 `((reduce f x) list)`写作`(reduce f x list)`。一个三元函数如`reduce`，当只提供两个参数时，将成为关于那个余下参数的一元函数。一般地，对一个N元函数，提供了M(< N)个参数后，该函数便成为了关于余下的N-M个参数的函数。我们在下文中将遵守这一约定。

用这种方式将`sum`模块化之后，我们就可以通过对这部分的重用来收获福利了。最有趣的部分就是，`reduce`可以（直接）用于编写一个函数来计算列表中元素的累乘积，而不需要更多的编程步骤：

```c++
product = reduce multiply 1
```

它也可以用来测试一个布尔值的列表中是否至少有一个元素为真：

```c++
anytrue = reduce or false
```

或者它们是否都为真：

```c++
alltrue = reduce and true
```

理解(reduce f a)的一种方式是，**将其看作一个将列表中的所有`cons`替换为`f`，将所有`nil`替换为`a`的函数**。以列表`[1,2,3]`为例，既然它表示：

```c++
cons 1 (cons 2 (cons 3 nil))
```

那么`(reduce add 0)`将其转换为：

```c++
add 1 (add 2 (add 3 0)) = 6
```

而`(reduce multiply 1)`将其转换为：

```c++
multiply 1 (mulitiply 2 (mulitiply 3 1)) = 6
```

于是，很明显地，`(reduce cons nil)`将复制列表本身。既然将一个列表追加到另一个列表上的方式是将前一个列表的元素`cons`到后一个列表前部，我们便得到：

```c++
append a b = reduce cons b a
```

例如：

```c++
append [1,2] [3,4] = reduce cons [3,4] [1,2]
= (reduce cons [3,4]) (cons 1 ( cons 2 nil ))
= cons 1 ( cons 2 [3,4]))
= [1,2,3,4] （将cons替换为cons，将nil替换为[3,4]）
```

一个用于将列表中全部元素翻倍的函数可以写作：

```c++
doubleall = reduce doubleandcons nil
doubleandcons num list = cons ( 2*num ) list
```

`doubleandcons`可以进一步模块化，首先分解为：

```c++
doubleandcons = fandcons double
double n = 2*n
fandcons f elem list = cons (f elem) list
```

继续分解：

```c++
fandcons f = cons . f
```

其中“.”（函数复合，一个标准运算符）定义为：

```c++
(f . g) h = f(g h)
```

为证明`fandcons`的定义是正确的，我们代入一些参数：

```c++
fandcons f elem
= (cons . f) elem
= cons (f elem)
```

因此

```c++
fandcons f elem list = cons (f elem) list
```

最终得到的版本是：

```c++
doubleall = reduce (cons . double) nil
```

继续模块化，我们得到：

```c++
doubleall = map double
map f = reduce (cons . f) nil
```

其中`map`使任意的函数作用于列表的全部元素之上。`map`是另一个很通用的函数。

我们甚至可以写出一个函数累加矩阵中的所有元素，该矩阵用列表的列表表示。这个函数是：

```c++
summatrix = sum . map sum
```

`map sum`使用函数`sum`分别计算所有行的元素之和，而后最左边的`sum`将每一行的元素之和累加起来，从而得到整个矩阵的累加和。

这些例子应该已经足以使读者确信，一点模块化的努力可以产生很大的效果。通过将一个简单的函数（sum）模块化为一个“高阶函数”与一些简单参数的聚合，我们得到了一个部件（reduce），它可以用于编写与列表有关的许多函数，而又不再需要（更多的）编程努力。不止是对有关列表的函数可以这么干，举另外一个例子，考虑数据类型“有序标记树”，其定义是：

```c++
treeof X ::= node X (listof (treeof X))
```

这个定义表明，一棵X的树，由一个标记类型为`X`的结点（node），以及一个子树列表组成，而这些子树也是`X`的树。例如，树：

           1 o
            /\
           /  \
          /    \
        2 o     o 3
                |
                |
                |
                o 4

可以被表示成：

```c++
node 1(cons
  (node 2 nil)
  (cons (node 3
    (cons (node 4 nil) nil)
  )
nil))
```

我们不再给出一个函数例子并将它抽象为高阶函数，取而代之的是，直接给出一个类似于`reduce`的函数`redtree`。回忆一下，`reduce`有两个参数，一个用于取代`cons`，另一个用于取代`nil`。既然树由`node`，`cons`和`nil`组成，那么`redtree`必须有三个参数——用于分别取代上述三者。由于树和列表不是同一种类型，我们得定义两个函数分别处理它们。因此我们定义：

```c++
redtree f g a (node label subtrees) = f label (redtree' f g a subtrees )
redtree' f g a (cons subtree rest) = g (redtree f g a subtree) (redtree' f g a rest)
redtree' f g a nil = a
```

［这相当于`f`取代`node`，`g`取代`cons`，`a`取代`nil`。］

很多有趣的函数都可以通过把`redtree`和其他函数粘起来的方法来定义。例如，要把一棵数字树上的所有标记累加起来，可以使用：

```c++
sumtree = redtree add add 0
```

以我们刚才表述的那棵树为例，`sumtree`展开成：

```c++
add 1
(add (add 2 0)
(add (add 3
(add (add 4 0) 0))
0))
= 10
```

要生成一个包含树中全部标记的列表，可以用：

```c++
labels = redtree cons append nil
```

仍然是那个例子，得到：

```c++
cons 1
(append (cons 2 nil)
(append (cons 3
(append (cons 4 nil) nil))
nil))
= [1,2,3,4]
```

最后，可以定义一个类似于`map`的函数，此函数使函数f作用于树中的全部标记上：

```c++
maptree f = redtree (node . f) cons nil
```

以上这些操作之所以可行，是因为函数式语言允许将传统型语言中不可分解的函数表达为一些部件的聚合，也就是一个泛化的高阶函数与一些特化函数的聚合。这样的高阶函数一旦定义，便使得很多操作都可以很容易地编写出来。不论何时，只要一个新的数据类型被定义，就应当同时定义用于处理这种数据的高阶函数。这样就简化了对数据类型的处理，同时也将与它的表示细节相关的知识局部化了。与函数式语言最相像的传统程序语言是可扩展语言，只要有需求，这种程序语言就好像随时都可以扩展出新的控制结构一样。

## 4.把程序粘起来

函数式语言提供的另一种黏合剂使得所有程序都可以粘在一起。回忆一下，一个完整的函数式程序只不过是一个从输入映射到输出的函数。如果`f`和`g`是这样的程序，那么对程序`(g.f)`当提供了输入参数`input`之后，得到：

```c++
g (f input)
```

程序`f`计算自身的输出，此输出被用作程序`g`的输入。传统上，这是通过将f的输出储存在临时文件中实现的。这种方法的毛病是，临时文件可能会占用太大的空间，以至于将程序黏合起来变得很不现实。函数式语言提供了一种解决方案。程序`f`和`g`严格地同步运行，只有当`g`试图读取输入时，`f`才启动，并且只运行足够的时间，恰好可以提供`g`需要读取的输出数据。而后`f`将被挂起，`g`将继续执行，直到它试图读取另一个输入。一个额外的好处是，如果`g`没有读取完f的全部输出就终止了，那么`f`也将被终止。`f`甚至可以是一个不会自行终止的程序，它可以产生无穷多的输出，因为当`g`运行结束时，`f`也将被强行终止。这就使得终止条件可以与循环体分离——一种强大的模块化形式。

这种求值方式使得f尽可能地少运行，因此被称为“**惰性求值（lazy evaluation）**”。它使得将程序模块化为一个产生大量可能解的生成器与一个选取恰当解的选择器的方案变得可行。有些其他的系统也允许程序以这种方式运行，但只有函数式语言对每一个函数调用都一律使用惰性求值，使得程序的每个部分都可以用这种方式模块化。惰性求值也许是函数式程序员的拿手利器中威力最大的模块化工具。

## 4.1.牛顿－拉夫森求根法

我们将编写一些数值算法以展现惰性求值的威力。首先，考虑用于**求解平方根的牛顿－拉夫森**算法。该算法从一个初始的近似值`a0`开始计算数`N`的平方根，为了求得更好的解，它使用下述规则：

```c++
a(n+1) = (a(n) + N/a(n)) / 2
```

如果近似值序列趋近于某一个极限`a`，那么

```c++
a = (a + N/a) / 2
2a = a + N/a
a = N/a
a*a = N
a = squareroot(N)
```

事实上，这个近似值序列确实迅速地趋近于一个极限。平方根算法取一个允许误差`eps`为参数，当两个相邻的近似值之差的绝对值小于`eps`时，算法便终止了。

这个算法通常被编写为类似下面这样：

```fortran
C N IS CALLED ZN HERE SO THAT IT HAS THE RIGHT TYPE
X = A0
Y = A0 + 2.*EPS
C THE VALUE OF Y DOES NOT MATTER SO LONG AS ABS(X-Y).GT.EPS
100 IF (ABS(X-Y).LE.EPS) GOTO 200
Y = X
X = (X + ZN/X) / 2
200 CONTINUE
C THE SQUARE ROOT OF ZN IS NOW IN X
```

［这是一段FORTRAN的程序，C代表注释行。.LE.是“Less than or Equal to”（小于或等于）的缩写，同理.GT.是“大于”的意思。]

在传统型语言中，这个程序是不可分解的。我们将利用惰性求值将其化为更加模块化的形式，而后演示所生成部件的一些其他用途。

由于牛顿－拉夫森算法计算的是一个近似值的序列，故将它写作一个使用近似值列表的程序就再自然不过了。每个近似值都可以通过下面的函数从前一个值计算得到：

```c++
next N x = (x + N/x) / 2
```

因此(next N)是从一个近似值映射到下一个值的函数。调用函数f，得到近似值序列：

```c++
[a0, f a0, f(f a0), f(f(f a0)), ...]
```

我们可以定义一个函数来计算：

```c++
repeat f a = cons a (repeat f (f a))
```

因此近似值序列可以这样计算：

```c++
repeat (next N) a0
```

`repeat`是一个具有“无穷”输出的函数的例子——但这没关系，因为超出程序其余部分需求的近似值并不会被计算。无穷性只是潜在的：它只说明，只要有需求，就可以计算出任意数量的近似值，`repeat`本身不会强加任何限制。

求根函数的剩余部分是函数`within`，它取一个允许误差与一个近似值列表作为参数，并在列表中查找差值不超过允许误差的一对相邻的近似值。这个函数可以定义为：

```c++
within eps (cons a (cons b rest)) = {
= b, 如果 abs(a-b) <= eps
= within eps (cons b rest), 其他情况
}
```

将这两个部件结合起来，

```c++
sqrt a0 eps N = within eps (repeat (next N) a0)
```

现在我们得到了求根函数的两大部件，便可以尝试用不同的方式组合它们。将要进行的修改之一，是将判断条件改为“相邻近似值的比趋近1”而不是“差趋近0”。这对于非常小的数字而言更加合适（当初始的相邻近似值之间的差值很小时），对非常大的数字也是如此（当舍尾产生的误差比允许误差大很多时）。我们只需要定义一个函数来替换`within`，而并不需要改写生成近似值的部件：

```c++
relative eps (cons a (cons b rest)) =
= b, 如果 abs(a-b) <= eps*abs b
= relative eps (cons b rest), 其他情况
```

［注意：`relative`里的`eps`与`within`里的`eps`定义是不同的！前者是绝对误差后者是相对误差！］

## 4.2.数值微分

我们已经重用了平方根近似值序列，当然，对函数`within`和`relative`的重用也是可能的，它们能够与任何一个生成近似值序列的数值算法配合。我们将这样来编写数值微分算法。

函数在某一点的微分，便是其图象在该点的斜率。通过分别计算函数在该点与一个临近点处的取值，而后计算两点连线斜率的方法，可以很容易地估计出微分的值。这基于一个假定：如果这两点靠得足够近，那么函数图象在两点之间不会弯曲得很厉害。于是有下述定义：

```
easydiff f x h = (f(x+h)-f x) / h
```

为了得到良好的近似值，`h`应该很小。不幸的是，如果`h`太小，那么`f(x+h)`与`f(x)`会相当接近，因此在相减过程中产生的舍尾误差可能会掩盖了计算结果。如何为h选取恰当的值呢？解决这个矛盾一种合理的方案是：从一个较大取值开始，不断减小`h`的值，并求出一个近似值序列。这个序列将趋近于该点的导数，但最终会由于舍尾误差的存在而不可救药地变得不精确。如果我们用`(within eps)`来选取第一个足够精确的近似值，那么舍尾误差影响结果的风险将会大大降低。我们需要一个函数来计算这个序列：

```c++
differentiate h0 f x = map (easydiff f x) (repeat halve h0)
halve x = x/2
```

此处`h0`是`h`的初值，而后继取值是通过不断减半得到的。通过这个函数，任意点处的导数可以这样计算：

```c++
within eps (differentiate h0 f x)
```

但是，甚至这个方案也不是那么令人满意的，因为近似值序列收敛得相当慢。解决这个问题需要一点数学知识，序列中的元素可以记为：

```c++
微分的精确值 ＋ 一个关于h的误差项
```

理论表明，该误差项与`h`的某一次幂大致成正比，因此当`h`减小时，误差也会减小。设精确值为`A`，而误差项为`B*h**n` [**是求幂运算符］。由于计算每个近似值时所用的h取值是下一个的两倍，故任意两个连续的近似值可以表示成：

```
a(i)   = A + B*(2**n)*(h**n)
a(i+1) = A + B*(h**n)
```

现在就可以消去误差项了，我们得到：

```
A=(a(i+1)*(2**n) - a(i))/(2**n - 1)
```

当然，误差项只不过“大致”与`h`的某一次幂（成正比），因此这个结论也是近似的。但这是一个好得多的近似。这一改进可以通过下述函数作用于所有相邻的近似值对之上：

```
elimerror n (cons a (cons b rest)) =
= cons ((b*(2**n)-a)/(2**n-1)) (elimerror n (cons b rest))
```

从一个近似值序列中消除误差项的操作产生了另一个收敛速度快得多的序列。

使用`elimerror`之前还有一个问题需要解决——我们必须知道n的正确值。通常这个值很难预测，但却很容易衡量。不难验证，下述函数能够正确地消除误差项，但在此我们并不给出证明。

```
order (cons a (cons b (cons c rest))) =
= round(log2( (a-c)/(b-c) - 1 ))
round x = 最接近x的整数
log2 x  = x以2为底的对数
```

现在，一个通用的近似值序列优化函数可以定义为：

```
improve s = elimerror (order s) s
```

使用improve能够更加高效地计算函数f的导数，如下：

```
within eps (improve (differentiate h0 f x))
```

`improve`只对利用一个不断减半的参数`h`计算得到的近似值序列适用。但是，如果`improve`作用于这样的序列，那么其结果也是一个这样的序列！这意味着一个近似值序列可以优化不止一次。每一次优化的过程中，都有一个不同的误差项被消除，因此优化产生的序列收敛得越来越快。因此，可以非常高效地计算导数：

```
within eps (improve (improve (improve (differentiate h0 f x))
```

从数值分析的角度讲，这似乎是一个“第四阶方法”(fourth order method)，可以迅速地给出准确的结果。甚至可以定义：

```
super s = map second (repeat improve s)
second (cons a (cons b rest)) = b
```

`super`函数使用`repeat improve`来生成一个不断被优化的近似值的序列的序列。［就是说，生成一个序列，其中每一个元素是一个近似值序列，而这个元素是用前一个元素优化得到的。］同时，`super`提取出每个近似值序列中的第二个元素，构造出一个新的序列（已经确认，第二元素是最佳选择——它比首元更精确，而且不需要额外的计算）。这个算法的确非常复杂——更多的近似值被计算的同时，它使用了不断优化的数值方法。可以用下面的程序非常非常高效地计算导数：

```
within eps (super (differentiate h0 f x))
```

这个案例可能就像是用大锤敲碎坚果一样（大材小用），但关键是，甚至一个像`super`一样复杂的函数，当被惰性求值的方法模块化时，也会变得很容易表达。

## 4.3.数值积分

在这一部分我们将讨论的最后一个例子是数值积分。问题的描述很简单：给出一个返回实数，并有一元实数参数的函数，以及两个端点`a`和`b`，估算两点之间曲线f下方的面积。估算面积的最简单方法是假定`f`趋近于直线，此时面积就是：

```
easyintegrate f a b = (f a + f b)*(b-a)/2
```

不幸的是，除非`a`与`b`足够接近，否则这个估算似乎非常不精确。更好的估算方法是，将a与b之间的区间分为两段，分别估算子区间上的面积，再将结果加起来。我们可以定义一个不断趋近于准确值的积分近似值序列，首先使用上述方程进行第一次近似，而后将分别趋近于两个子区间上的子积分准确值的（两个）近似值累加起来以得到新的（积分总体的）近似值。计算这个序列可以使用函数：

```
intergrate f a b = cons (easyintergrate f a b)
(map addpair (zip (intergrate f a mid)
(intergrate f mid b)))
式中 mid = (a+b)/2
```

`zip`是另一个标准的表处理函数。它读取两个列表，并返回一个有序对的列表，每个有序对由两个输入列表中对应的元素组成。从而第一对由列表一和列表二的首元组成，第二对由列表一和列表二的第二个元素组成，以此类推。`zip`可以定义为：

```
zip (cons a s) (cons b t) = cons (pair a b) (zip s t)
```

在函数`intergrate`中，`zip`用于生成由两个子区间上相对应的积分近似值对组成的列表，而`map addpair`用于将有序对中的元素相加，从而生成一个原积分的近似值列表。

实际上，这个版本的`intergrate`函数相当低效，因为它持续不断地重复计算f的值。就像所写的一样，`easyintergrate`计算了`f`在`a`和`b`两处的值，而对`intergrate`的递归调用将重复计算它们。同样的，`(f mid)`也在递归调用中重复计算了。因此，更可取的是下述从不重复计算f的版本：

```
intergrate f a b = interg f a b (f a) (f b)
integ f a b fa fb = cons ((fa+fb)*(b-a)/2)
(map addpair (zip (interg f a m fa fm)
(interg f m b fm fb)))
式中 m = (a+b)/2
fm = f m
```

`integrate`给出了一个不断趋近准确值的积分近似值列表，正如`differentiate`在上一小节中所做的一样。因此可以写出计算式以求出所需任意精度的积分值，如下：

```
within eps (intergrate f a b)
relative eps (integrate f a b)
```

这个积分算法与上一小节中的第一个微分算法有着同样的缺点——它收敛得相当慢。序列中的第一个近似值仅仅用了两个相距`(a-b)`的点来计算（通过`easyintergrate`）。第二个近似值也（除了`a`、`b`之外）用到了中点，因此相邻两点之间的间距仅为`(b-a)/2`。第三个近似值在两个子区间上作同样的处理，因此间距仅为`(b-a)/4`。很清楚，每个近似值对应的相邻两点之间的间距在计算下一个值时被减半了。将这一间距看作`h`，那么这个序列就可以成为上一小节中定义的`improve`函数的优化对象了。因此我们可以写出（函数来计算）快速收敛的积分近似值序列，例如：

```
super (intergrate sin 0 4)
improve (intergrate f 0 1)
式中 f x = 1/(1+x*x)
```

（后一个序列是用于计算`pi/4`的“第八阶方法”。其中的第二个近似值只需要计算5次`f`的取值，但却具有5位准确数字。）

在本节中我们选取了一些数值算法并将它们函数化地编写出来，把惰性求值当做了黏合部件的黏合剂。由于惰性求值的存在，使得我们可以用很多新的方式来模块化这些算法，从而产生用途广泛的函数，例如`within`，`relative`和`improve`。通过这些部件的不同组合，我们简单而明了地编写出了一些相当不错的数值算法。

## 5.人工智能中的例子

我们已经指出，函数式语言威力强大主要是因为它们提供了两种新的黏合剂：高阶函数和惰性求值。在本节中，我们将讨论人工智能中一个大一点的实例，并演示如何使用这两种黏合剂来十分简单地编写它。

我们选取的实例是`alpha-beta`“启发式搜索”，一个用于估计游戏者所处形势好坏的算法。该算法预测游戏局势的可能发展，但会避免对无意义局势的进一步探究。

令游戏局势使用`position`类型的对象来表示。这个类型依据游戏的不同而不同，我们不对此作任何假定。必然有一种方法可以知晓对某一个局势能够采取的行动：假定有一个函数：

```
moves: position -> listof position
```

该函数以一个游戏局势为参数，并返回一个可以由自变量出发，通过一步行动而形成的`position`的列表。以noughts and crosses游戏（tic-tac-toe）为例：

![](/Users/hexo_images/fp1.png)

这个函数假定通过当前局势总是可以判定现在是哪位游戏者的回合。在noughts and crosses中，可以通过数出`0`与`X`的数目来做到这一点。在类似于象棋的游戏中，可能必须在`position`类型中显式包含这一信息。

利用函数`moves`，第一步是构造一棵博弈树。这棵树的结点都用局势来标记，而一个结点的子结点用从该结点一步便可到达的局势标记。也就是说，如果一个结点标记为局势`p`，那么它的子结点将使用`(moves p)`中的局势来标记。一棵博弈树完全有可能是无穷的，如果这个游戏可以在双方都不胜的情形下永远进行下去的话。博弈树与第2节中讨论的树完全类似——每个结点都有一个标记（它所代表的局势）与一个子结点列表。因此我们可以使用相同的数据类型来表示它们。

博弈树是通过反复运用`moves`而构造出来的。构造从根局势开始，`moves`用于生成根结点处子树的标记，而后`moves`被用于生成子树的子树，依此类推。这一递归模式可以用一个高阶函数表示：

```
reptree f a = node a (map (reptree f) (f a))
```

使用这个函数可以定义另一个函数，该函数从一个特定的局势开始生成博弈树：

```
gametree p = reptree moves p
```

例如图1所示。此处使用的高阶函数`reptree`与上一节中用于构造无穷列表的函数`repeat`是类似的。

![](/Users/hexo_images/fp2.png)

图1： 一棵博弈树的实例

alpha-beta算法从一个给定的局势出发，就游戏的发展将会是有利还是不利作出判断。然而，要做到这一点，它必须能够在不考虑下一步的情况下粗略地估计某一个局势的“价值”。在后继局势不可预测时必须使用这一函数，它也可以用来对算法进行先期引导。静态估价的结果是从计算机的角度考虑的，是对该局势的前途的度量（假设在游戏中计算机与人对抗）。结果越大，局势对计算机而言越好。结果越小，局势越糟。最简单的此类函数将会，比如说，对计算机确定胜利的局势返回+1，对计算机确定失败的局势返回-1，而对其它的局势返回0。在现实中，静态估价函数会衡量各种使局势“看上去不错”的因素。例如，具体的好处，以及象棋中对中心的控制。假定有这样一个函数：

```
static: position -> number
```

既然一棵博弈树是一个(treeof position)，那么它就可以被函数(maptree static)转换为一个(treeof number)，该函数对树中所有的（也许是无穷多个）局势进行静态估价。此处使用了第2节中定义的函数`maptree`。

给出一棵静态估价树之后，其中各个局势的真值究竟是多大？特别地，对根局势应该赋予什么值？不是它的静态值，因为那只是一个粗略的猜测。一个结点被赋予的值，必须由其子结点的真值决定。这一过程的完成，基于每个游戏者都会选择对自己最有利的行动的假定。回忆一下，高值意味着计算机的有利形势。很明显，当计算机从任意的局势开始下一步行动时，它将选择通往真值最高的子结点的行动。类似地，对手将会选择通往真值最低的子结点的行动。假定计算机与其对手轮流行动，那么当轮到计算机行动时，节点的真值用函数`maximise`计算，反之用`minimise`计算。

［所谓“真值”（true value），可能是我翻译得不好，此处理解为类似“真正的价值”的意思吧，是一个量度，不是逻辑学里的0和1哦。］

```
maximise (node n sub) = max (map minimise sub)
minimise (node n sub) = min (map maximise sub)
```

此处`max`和`min`是关于列表的函数，分别返回列表中元素的最大值与最小值。上述定义是不完整的，因为它们将永远递归下去——没有给出边界情形。我们必须定义没有后继的结点的值（其标记）。因此静态估价用于任一游戏者胜利或者后继局势不可预测的情况下。`maximise`与`minimise`的完整定义是：

```
maximise (node n nil) = n
maximise (node n sub) = max (map minimise sub)
maximise (node n nil) = n
maximise (node n sub) = min (map minimise sub)
```

在这个阶段，几乎已经可以写出一个取一个局势作为参数并返回其真值的函数了。可能是：

```
evaluate = maximise . maptree static . gametree
```

这个定义有两个问题。首先，它不适用于无穷树。`maximise`不断地递归直到找到一个没有子树的结点——树的端点。如果没有端点那么`maximise`就不会返回结果。第二个问题与第一个有关——甚至有穷的博弈树（如noughts and crosses里的那棵）事实上也可能相当大。估价整棵博弈树是不现实的——搜索必须被限定在接下去的几步之内。为此可以将树剪至一个固定的深度：

```
prune 0 (node a x) = node a nil
prune n (node a x) = node a (map (prune (n-1)) x)
```

`(prune n)`取一棵树作为参数并“剪去”与根结点的距离超过n的所有结点。如果一棵博弈树被剪枝，那么将强制`maximise`对深度为`n`的结点执行静态估价而不是进一步递归。因此`evaluate`可以被定义为：

```
evaluate = maximise . maptree static . prune 5 . gametree
```

这将考虑其后（比如说）5步的形势。

在此开发过程中我们已经使用了高阶函数与惰性求值。高阶函数`reptree`和`maptree`使得我们能够很容易地构造与处理博弈树。更重要的是，惰性求值确保了我们可以使用这种方式模块化`evaluate`。由于博弈树具有潜在的无穷结果，在没有惰性求值的情况下，程序将永远不会终止。我们将不能写：

```
prune 5 . gametree
```

而不得不将这两个函数整合成一个只构造树的前五层的函数。更糟糕的是，甚至那前五层都可能已经太大以至于无法在同一时间内存储于内存中。而在我们所写的程序中，函数

```
maptree static . prune 5 . gametree
```

只是构造出了树中`maximise`所需的部分。由于每一部分都可以在被`maximise`处理完之后丢弃（被垃圾收集器回收），故完整的树从来没有存储于内存中。只有树的一小部分在某一段时间内被储存着。因此这个惰性程序很有效率。这一效率取决于`maximise`（组合链上的最后一个函数）与`gametree`（第一个函数）的相互作用，因此在没有惰性求值的情况下，要完成任务，只能将组合链上的所有函数整合成一个大函数。这是对模块化的强烈破坏，但也是通常的做法。通过单独修补每个部件，我们就可以优化估价算法——这相对简单。而一个传统型程序员必须把整个程序作为一个单元来修改，这就困难多了。

到目前为止，我们只是描述了简单的对最大最小值的处理（minimaxing）。但alpha-beta算法的核心是“计算`maximise`与`minimise`的值时常常不需要考虑整棵树”这一观察结果。考虑树：

```
          max
          / \
         /   \
        /     \
       /       \
     min       min
     / \       / \
    /   \     /   \
   1     2   0     ?
```

相当奇怪地，为了估价这棵树，并不需要知道问号处的值。左子树的最小值是1，但右子树的最小值显然是一个小于或等于0的值。因此这两个最小值的最大值必然是1。这一观察结果可以被泛化并内建到`maximise`和`minimise`之中。

第一步是将`maximise`拆分成`max`对一个数字列表的作用。也就是，将`maximise`分解为：

```
maximise = max . maximise'
```

（`minimise`可以用类似的方法分解。由于`maximise`和`minimise`是完全对称的，故我们将只讨论`maximise`，而假定`minimise`也照此处理。）一旦这样分解之后，`maximise`可以使用`minimise'`来发现`minimise`将对哪些数字求最小值，并且不再使用`minimise`本身。而后便可以在不查看某些数字的情况下便将它们丢弃。由于惰性求值的存在，如果`maxmise`并不会查看所有的数字列表，那么一部分列表将不会被计算，这是对计算机时间的潜在节约。

将`max`从`maximise`中“约分出来”是很简单的，得到：

```
maximise' (node n nil) = cons n nil
maximise' (node n l) = map minimise l
= map (min . minimise') l
= map min (map minimise' l)
= mapmin (map minimise' l)
式中 mapmin = map min
```

由于`minimise' `返回一个数字列表，而这个列表的最小值是`minimise`的结果，故`(map minimise' l)`返回一个数字列表的列表。`Maximise'`应该返回这些列表中每个列表的最小值组成的列表，但只有其中［`Maximise`的返回值中］的最大值才有用。我们应该定义一个`mapmin`的新版本以忽略那些最小值不重要的列表［在`(map minimise' l)`的返回值中］的最小值。

```
mapmin (cons nums rest) =
= cons (min nums) (omit (min nums) rest)
```

函数`omit`传递一个“潜在的最大值”——当前所发现的最小值中最大的一个——并忽略任何比该值小的最小值。

```
omit pot nil = nil
omit pot (cons nums rest) =
= omit pot rest, if minleq nums pot
= cons (min nums) (omit (min nums) rest), otherwise
```

`minleq`以一个数字列表和一个潜在最大值为参数，如果列表的最小值小于或等于潜在最大值就返回真。要完成这一工作，它并不需要扫描整个列表！如果列表中有任意一个元素小于或等于潜在最大值，那么列表的最小值肯定也是如此。该特别元素之后的所有元素都是无关紧要的——它们就像是上面例子中的问号一样。因此`minleq`可以被定义为：

```
minleq nil pot = false
minleq (cons num rest) port = true, if numn2
```

此处`sort`是多用途排序函数。现在估价函数定义为：

```
evaluate = max . maximise' . highfirst . maptree static . prune 8 . gametree
```

也可能认为，为了限制搜索，只要考虑计算机或者对手的前三个最佳行动也已经足够了。要编写这样的程序，只需要把`highfirst`换成`(taketree 3 . highfirst)`，其中：

```
taketree n = redtree (nodett n) cons nil
nodett n label sub = node label (take n sub)
```

`taketree`将树上所有的结点替换为最多有n个子结点的结点，它使用了函数`(take n)`，而该函数返回列表的前n个元素（如果列表比n短，那么返回的元素就少一些）。

另一种优化是对剪枝的改良。上述程序甚至在局势非常dynamic的情形下也会向前搜索固定的深度——（但是，）例如在国际象棋中，一旦皇后被威胁，也许就可以决定不再搜索了。通常可以定义某些“dynamic”的形势，并在遇到这样的结点之一时，不再继续搜索而停止。假定有函数“dyramic”用以确定这样的形势，那么只需要为`prune`追加一个定义等式：

```
prune 0 (node pos sub) = node pos (map (prune 0) sub), if dynamic pos
```

在像这个程序一样模块化的程序里，作出这样的改动是很简单的。如前所述，这个程序的效率，关键是由链中的最后一个函数`maximise`与第一个函数`gametree`的相互作用决定的，因此若没有惰性求值，就只能写成一个单一的程序。这样的程序难于编写，难于修改，而且，非常难于理解。

## 6.结论

在本文中，我们指出模块化是成功的程序设计的关键。以提高生产力为目标的程序语言，必须良好地支持模块化程序设计。但是，新的作用域规则和分块编译的技巧是不够的——“模块化”不仅仅意味着“模块”。我们分解程序的能力直接取决于将解决方案粘在一起的能力。为了协助模块化程序设计，程序语言必须提供优良的黏合剂。函数式程序语言提供了两种新的黏合剂——**高阶函数**与**惰性求值**。利用这些黏合剂可以将程序用新的、令人激动的方式模块化，对此我们举出了很多实例。越小、越通用的模块越可能被广泛地重用，使后续的程序设计工作变得简单。这解释了为什么函数式程序与传统型程序比较，要小得多，也容易编写得多。它也为函数式程序员提供了一个追求目标。如果程序的任何部分是杂乱或者复杂的，那么程序员就应当尝试将其模块化并泛化其部件。他应当期望把高阶函数和惰性求值用作他做此事的工具。

当然，我们并不是指出高阶函数与惰性求值的力与美的第一人。例如，Turner展示了这两者如何在一个生成化学结构的程序里大显身手［Tur81］。Abelson和Sussman强调“流”（惰性列表）是构架程序的强大工具［AS86］。Henderson使用了流来构架函数式操作系统［P.H82］。本论文的主要贡献是，断言了模块化自身，便是函数式语言强大威力的关键。

这与当前有关惰性求值的论战也有关联。有些人认为函数式语言应当是惰性的，而其他人认为不是这样。有些人走折衷路线，只提供惰性列表以及用于构造它们的特殊语法（例如，在SCHEME中［AS86］）。本论文提供了更进一步的证据，证明惰性求值非常重要以至于不能被降为二等公民。这也许是函数式程序员所拥有的最强大的黏合剂。人们不应当阻碍对这样一个极为重要的工具的使用。

## 致谢

在牛津程序设计研究组与Phil Wadler和Richard Bird的多次交谈对本论文的写作帮助甚大。约特堡查麦兹大学的Magnus Bondesson指出了一个数值算法的早期版本中的严重错误，同时也协助了很多其他算法的开发。本论文在英国科学与工程研究评议会提供的研究基金赞助下发表。

## 参考文献

- [AS86] H.Abelson,G.J.Sussman. 计算机程序的构造与解释. 麻省理工学院出版社,波士顿,1986
- [Hug89] J.Hughes. 函数式程序设计为什么至关重要. 计算机月刊,32(2),1989
- [Hug90] John Hughes. 函数式程序设计为什么至关重要. D.Turner主编,函数式编程的研究主题. Addison Wesley,1990
- [MTH90] R.Milner,M.Tofte,R.Harper. Standard ML的定义. 麻省理工学院出版社,1990
- [oD80] 美利坚合众国国防部. 程序语言Ada参考手册. Springer-Verlag,1980
- [P.H82] P.Henderson. 纯函数式操作系统. 1982
- [Tur81] D.A.Turner. 应用语言在语义上的优雅性. 1981年度函数式语言与计算机架构会议会报,海边温特渥,普茨茅斯,新汉普夏郡,1981
- [Tur85] D.A.Turner. Miranda: 拥有多态类型的非严格语言. 1985年度函数式语言与计算机架构会议会报,1-16页,南锡,法国,1985
- [Wir82] N.Wirth. Modula-II程序设计. Springer-Verlag,1982

[^1]: 函数的返回值只依赖于其输入值，这种特性就称为引用透明性（referential transparency）

[^2]: 即懒加载