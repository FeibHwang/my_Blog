---
layout:     post
title:      "使用Haskell实现Lisp解释器"
subtitle:   "编译原理学习笔记"
date:       2017-05-16 18:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Haskell
    - 编译原理
    - 翻译
---

在翻了几章SICP之后，一直想试着实现一个Lisp解释器，在自己读master的第一年曾经试着实现了一个Lisp的子集编译器（没有看过编译原理，纯粹是凭感觉实现的，代码惨不忍睹=_=!），在学完Haskell以后，一直想找一个可以练手的项目，看到了这篇[文章](http://www.defmacro.org/ramblings/lisp-in-haskell.html)，就顺便翻译一下吧。

# 简介

不久前，在看了一堆有关Haskell的文章之后，我终于越过了理论安装了一个Haskell编译器。我决定通过两种方式探寻这门语言的演化方式。我准备先使用Haskell解决一个它本身擅长的问题，之后解决一个源自真实世界的问题。选择一个问题很简单，有很多“传言”说Haskell很适合实现编译器或解释器，因此我决定使用Haskell实现一个Lisp方言的解释器。对于第二个问题我选择实现一个网络应用，因为Haskell并没有大规模应用于这个领域。

我完成了第一个测试例子，结果也是验证有效的。我准备使用188行Haskell实现一个解释器来解析我的这篇[有关Lisp的文章](http://www.defmacro.org/ramblings/lisp.html)里的例子。我并没有试图压缩程序的体积，作为一个Haskell菜鸟这一步一般有花费更多的时间。

毫无疑问，Haskell有着强大的表述能力。相比于其他语言，我可以使用更少的代码实现更多的功能，同时代码的可扩展性也十分良好。在得到这一结论后我决定告诉给大家，于是有了这一篇文章。我完成的第一个draft但是很快就舍弃了，因为我意识到那些已经知道Haskell的人可能觉得我说的是废话，而不知道的人不太可能从我的文章中得出这一结论。

现在这篇文章是第二个draft，它包括了除了代码以外的内容。这篇文章记述了我对于Haskell的经验以及项目的实现过程。我会记述在我实现解释器某一功能的过程中我使用了Haskell的哪些特性，以及它与面向对象编程语言的区别，为什么这样更有助于我的目的，以及最后每当我陷入死局时我是如何改写我的代码的。尽管人们宣称Haskell在其他领域的优势也很突出，我希望这篇文章可以激发出你对这门美丽的语言的兴趣，同时解释一些初学者感到困惑的地方。

# 抽象语法树

实现解释器的第一步是确定解析边界。我决定紧紧处理整数，符号，函数以及列表，这些元素组成了Lisp的最小子集。之后我需要将这些idea转换成Haskell代码，这与使用任何语言实现的过程一样。同时Haskell的优势立马显示了出来。通过Haskell我仅仅需要4行代码来实现所需要的数据结构！我使用Haskell定义了一个代数数据(Algebraic Data Type)的数据格式：

{% highlight hs %}

data Expr = BlaiseInt Integer |
            BlaiseSymbol String |
            BlaiseFn ([Expr]->Expr) |
            BlaiseList [Expr]

{% endhighlight %}

这个代数数据类型简直就像黑膜法。因为它们与我们平常的编程方式有所区别，这造成了初学者的困惑，而且往往会伴随初学者很久直到我们从直觉上理解它。上面的代码仅仅表述了一个`Expr`可能是一个整数，一个符号，一个函数或一个列表，同时每一个元素包含了一个单成员域(single member field)，分别是一个整数，一个string，一个接收表达式列表并返回一个表达式的函数，一个表达式列表。如果我们用Java表示上面的定义可能是这样：

{% highlight java %}

abstract class Expr {}

final class BlaiseInt extends Expr {
    public int _value;
}

final class BlaiseSymbol extends Expr {
    public String _value;
}

interface BlaiseFunction {
    public Expr evaluateFunction(List arguments);
}

final class BlaiseFn extends Expr {
    BlaiseFunction _value;
}

final class BlaiseList extends Expr {
    List _value;
}

{% endhighlight %}

我们可以看出Java实现相比于Haskell要更长，尽管我已经尽量简化并省略了构造函数以及共享变量等方法。除此以外，当我们继续扩展这个解释器时，为Java版本的实现将会变得十分困难。我会在这篇文章中试图解释这些点。我可以立刻举出的例子是数据结构实例化。比如，我准备实现一个带有3个整数的列表，在Haskell中我可以这样实现：

{% highlight hs %}

myList = BlaiseList [BlaiseInt 1, BlaiseInt 2, BlaiseInt 3]

{% endhighlight %}

而在Java中会这样实现：

{% highlight java %}

BlaiseList myList = new BlaiseList();
myList._value.add(new BlaiseInt(1));
myList._value.add(new BlaiseInt(2));
myList._value.add(new BlaiseInt(3));

{% endhighlight %}

虽然我们我你也可以用4行实现上述功能，但一行的Haskell实现也更加清晰：你可以使用Haskell实现更短，易读且扩展性良好的程序。

# 打印

之前一节我们实现了一个数据结构来承载Lisp的抽象语法书。还有一个所有Lisp解释器都应该具有的功能，打印数据结构。Haskell使用一个多态类型的`show`函数实现打印，对应着Java中的`toString`。`show`函数默认的会基于数据输入时的样子打印出数据（着使得`show`具有了比`toString`更强大的功能，但我们暂时不会涉及这些）。这两种实现对于我们的要求并不是十分有用，因为我们需要解释器通过`s表达式`打印数据结构。因此，我们必须自己实现这一功能：

{% highlight hs %}

instance Show Expr where
        show (BlaiseInt x) = show x
        show (BlaiseSymbol x) = x
        show (BlaiseFn x) = "<function>"
        show (BlaiseList x) = "(" ++ unwords (map show x) ++ ")"

{% endhighlight %}

不要被上面的`instace`所迷惑，上面的代码与面向对象程序中的实例无关。上述代码向Haskell表明为`Expr`数据实现了一个打印的方法“实现”。这样当任何人对我们的数据结构调用`show`函数后，就可以打印出s表达式。上面的代码应用了模式匹配————`show`函数被实现了4次。当`show`被调用时，Haskell运行环境会检测有效的数据类型，并调用对应的`show`函数实现。着与Java中的虚函数的概念很像但更加强大。在Java中我们可以定义4种`toString`函数并在运行时进行匹配。但是，Haskell的类型匹配可以同时匹配不仅仅是数据类型————它还可以匹配值，bool表达式，特殊数据结构等。这一特性的巨大优越性会在之后体现出来。现在，我们使用Java实现`toString`的列表打印：

{% highlight java %}

String toString() {
    StringBuffer output("(");
    Iterator it = _value.iterator();
    if(it.hasNext()) {
        Expr value = (Expr) it.next();
	output.append(value.toString());
    }
    for (; it.hasNext(); ) {
	output.append(" ");
        Expr value = (Expr) it.next();
	output.append(value.toString());
    }

    output.append(")");
    return output.toString();

{% endhighlight %}

然而Haskell的一行实现黑膜法变成了Java的16行代码。尽管这个例子不能说明什么，但一旦你一遍又一遍地遇到这些，你就可以看到一个有趣的图景展现在你的眼前>_<

# 语法解析

我们现在可以建立一个Haskell数据结构来表示Lisp的抽象语法树并将其打印下来。下一步就是实现一个语法解析模块(Parser)将Lisp那坑爹的括号架构转换为我们自己的数据结构。实现Lisp解释器的优点是它的Parser非常好实现————大部分时候我们可以不借助于高级的parse工具直接徒手实现。这种方式有其优越性(不需要另外编译代码，不需要学习使用这些工具，debug也是十分容易的)，相比于Java或C实现。一开始使用Haskell实现这个parser也是经过了同样的步骤————不借助工具徒手实现。然而，很快我意识到Haskell的不同并不能很好的与其他语言实现方式结合起来。

Haskell带有一个标准的语法解析库`Parsec`，它实现了一个领域特定(domain specific)语言作为解析语言。如果你熟悉C++ Boost [Spirit](http://www.defmacro.org/web/20161118010028/http://spirit.sourceforge.net/)库你应该知道我说的是什么。这个库内嵌了一门语法解析语言。人们可以使用这个库直接确定parser的语法！`Parsec`库需要一些时间来掌握，但一旦你学会了你就可以永远丢掉以往的parser生成方法了。为了展示`Parsec`的语法，以下代码片段可以解析一个符号：

{% highlight hs %}

parseSymbol = do f <- firstAllowed
                 r <- many (firstAllowed <|> digit)
                 return $ BlaiseSymbol (f:r)
        where firstAllowed = oneOf "+-*/" <|> letter

{% endhighlight %}

代码的描述能力很强。它先寻找第一个合法字符(一般不是数字)，然后是很多同类型字符或数字。你可以在源代码中找到其他部分的定义。这个例子展示了领域特定语言是如何避免了使用多余的工具，多余的编译链接步骤等没必要的工作。Haskell是领域特定语言极佳的承载器。警告：学习完`Parsec`在学习`JavaCC`会十分痛苦。

# 求值

现在我们到了解释器的核心——Lisp表达式求值。这一部分使用了Haskell的几项特性：类型推导(包括可选类型声明)，模式匹配，代数类型特性，函数第一成员，高阶函数等。让我们从以下代码片段中分别体会这一系列性质：

{% highlight hs %}

eval :: Expr -> Expr
eval (BlaiseInt n) = BlaiseInt n
eval (BlaiseSymbol s) = ctx Map.! s
eval (BlaiseFn f) = BlaiseFn f
eval (BlaiseList (x:xs)) = apply (eval x) (map eval xs)
        where apply :: Expr -> [Expr] -> Expr
              apply (BlaiseFn f) args = f args

{% endhighlight %}

这段代码定义了一个函数`eval`，用于解析Lisp表达式。如果你不熟悉Haskell的话这段代码可能会比较费解，但你只需学习一些Haskell的强大特性就可以解读。我们先从类型推导开始。

第一行是`类型声明`，我们告诉编译器`eval`函数接受一个表达式并返回另一个表达式。注意到声明与函数的剩余部分是分开的。在外行眼中Haskell好象是动态类型语言但其实不是——一切都在编译阶段确定了。Haskell通过类型接口实现静态编译而且它实现得非常好。大部分时候你不需要声明格式——Haskell会自己推导出来。有时编译器会陷入混淆因此你需要添加类型声明——但这只是例外而非必须。如果你不清楚某个对象的类型，你可以通过解释器命令行输入`:t something`来打印对象的类型。大部分类型声明是没有必要的，我添加只是为了更清楚。当你熟悉了这一套类型接口之后就很难再写出下面这种Java代码了：

{% highlight java %}

ReallyLongClassName i = (ReallyLongClassName)foo.getBar();

{% endhighlight %}

下面一行是一系列模式匹配的开端。这一特性可以看作是一种基于动态数据结构的虚函数以及正则表达式的混合，或者是一种基于大型声明跳转的抽象。4行模式匹配决定了`eval`函数对于4种类型的执行策略。例如，第二行说：“如果第一个元素是一个整数，就返回自己”。