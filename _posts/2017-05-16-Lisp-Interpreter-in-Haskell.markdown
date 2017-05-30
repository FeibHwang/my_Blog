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

不久前，在看了一堆有关Haskell的文章之后，我终于越过了理论并安装了一个Haskell编译器。我决定通过两种方式探寻这门语言的演化方式。首先我准备使用Haskell解决一个它本身擅长的问题，之后解决一个源自真实世界的问题。选择一个项目很简单，有很多“传言”说Haskell很适合实现编译器或解释器，因此我决定使用Haskell实现一个Lisp方言的解释器。对于第二个问题我选择实现一个网络应用，因为Haskell之前并没有大规模应用于这个领域。

我完成了第一个测试例子，结果也是验证有效的。我准备用188行Haskell实现一个解释器来解析我的这篇[有关Lisp的文章](http://www.defmacro.org/ramblings/lisp.html)中提到的例子。我并没有试图压缩程序的体积，作为一个Haskell菜鸟这一步一般要花费更多的时间。

毫无疑问，Haskell有着强大的表述能力。相比于其他语言，我可以使用更少的代码实现更多的功能，同时代码的可扩展性也十分良好。在得到这一结论后我决定告诉给大家，于是有了这一篇文章。我完成了第一个draft但是很快就舍弃了，因为我意识到那些已经知道Haskell的人可能觉得我说的是废话，而不知道的人不太可能从我的文章中得出这一结论。

现在这篇文章是基于我的第二个draft，它包括了一些除了代码以外的内容。这篇文章记述了我对于Haskell的经验以及项目的实现过程。我会特别记述在我实现解释器某一功能的过程中我使用了Haskell的哪些特性，以及它与面向对象编程语言的区别，为什么这样更有助于我的目的，以及最后每当我陷入死局时我是如何改写我的代码的。尽管人们宣称Haskell在其他领域的优势也很突出，我希望这篇文章可以激发出你对这门美丽的语言的兴趣，同时我会试图解释一些初学者感到困惑的地方。

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

在上面的例子中模式匹配的功能与Java中的虚函数类似——基于表达式类型执行不同的代码(在Haskell中我们可以基于类型本身或使其构造函数)。然而，这种抽象更为强大。原则上讲，虚函数只是类似于下面这种大量中转的集合：

{% highlight java %}

if(typeof(something) == someType) {
    doSomething();
} else if(typeof(something) == someOtherType) {
    doSomethingElse();
}

{% endhighlight %}

传统的面向对象语言让我们花大量时间与精力来实现上面这种中转而不是做更重要的事。Haskell也一样，但它不会仅仅限制于类型抽象——编译器允许我们基于任何抽象实现条件中转！举个例子，我们可以让编译器帮我们写基于不同条件的中转代码。我们可以使用传统语言中的`if`实现中转，但Haskell可以通过更加精简优雅的方式帮我们实现这些。毕竟，我们为何仅仅基于对象类型创建虚函数？为何我们不基于函数的某一参数类型来重新定义“虚(virtuality)”？有趣的是，这也是Haskell不是面向对象的原因之一——它的函数不需要给定一个对象，编译器与运行时环境让我们基于任何类型或结构做抽象！

在我们的实现中`eval`函数同样借用了Haskell的函数第一等级的优势。看一下我们是如何解析一个符号的——我们仅仅是从map中找到对应的值。此时符号预先被添加进map中：

{% highlight hs %}

ctx = Map.fromList [("*", BlaiseFn blaiseMul)]

{% endhighlight %}

同时在这个例子中我们使用了乘法算子。如果我们试图解析`"*"`，我们便可以从map中返回对应的函数！这就像实现一个临时的乘法对象实例，除了我们不用关心模板之类的问题——Haskell会在后台默默处理这些的。

我们同时使用了高阶方程(接收一个方程为输入参数的方程)来实现对函数的解析。仔细看一下我们使用Haskell的内置高阶函数map来进行列表迭代。我们传给`map函数`一个函数和一个列表，`map函数`会对列表中的每一个元素调用该函数。我们可以使用`for`循环或`foreach`来实现类似的效果，但这些对于Haskell是多余的，Haskell允许我们对于我们经常一遍又一遍写的代码模板构建抽象而不用等待编译器帮助我们实现(map虽然是Haskell的内建函数但如果我们想通过Haskell重新是你实现一下也是十分容易的)。

# 变量
现在我们的解释器从功能上看与一个计算器差不多了，我们可以解析简单的数学表达式，比如：

{% highlight lisp %}

(+ 1 2)
(+ 1 (+ 2 3))
(* 5 (+ 3 4))

{% endhighlight %}

这就像一个Lisp格式的计算器，而不是一个解释器。构建解释器的下一步自然就是定义变量了。鉴于Haskell原则上不允许可变数据结构(mutable data)同时又由于我们的设计仅仅限于4个元素(整数，符号，函数与列表)，我们需要处理一些问题。

第一个问题最显而易见： 既然Haskell不支持变量，我们如何通过它实现支持变量的Lisp解释器? 我们不能通过声明一个全局的map进行符号映射与修改——所有的Haskell变量仅仅可以写入`一次`。为了解决这个问题我们不得不修改我们的设计：`eval`函数需要接收解释器的当前状态(即当前的符号列表)并更新新的状态以及解析过的表达式。此外我们需要修改我们的`Expr`数据类型因为函数的功能包括修改符号表。最后的代码会像下面这样：

{% highlight hs %}

type Context = Map.Map String Expr

data Expr = BlaiseInt Integer |
        BlaiseSymbol String |
        BlaiseFn (Context->[Expr]->(Context, Expr)) |
        BlaiseList [Expr]

eval :: Context -> Expr -> (Context, Expr)
eval ctx (BlaiseInt n) = (ctx, BlaiseInt n)

{% endhighlight %}

现在我们的解释器可以处理状态了！然而，我们仍然无法处理赋值，原因比较微妙，看一下下面这行代码：

{% highlight lisp %}

(set i 5)

{% endhighlight %}

如果我们试图将`set`实现为一个函数我们的解释器就会崩溃！原因很简单——我们的代码在接受参数的时候会将它们全部解析。在上面的例子中，我们的解释器会试图解析符号i, 但由于i本是不存在于世会出现错误。因此函数并不是实现类似功能的好方法。我们需要修改我们的`Expr`数据格式并引入一种新的函数来进行解析。这样我们的`set函数`实现才会只解析第二个参数而留下第一个。Lisp一般通过某些特殊形式调取这些函数，因此我们的`Expr`可以定义为：

{% highlight hs %}

data Expr = BlaiseInt Integer |
        BlaiseSymbol String |
        BlaiseFn (Context->[Expr]->(Context, Expr)) |
        BlaiseSpecial (Context->[Expr]->(Context, Expr)) |
        BlaiseList [Expr]

{% endhighlight %}

最后一件工作是修改我们的`eval`函数，使其可以区别对待普通函数与特殊函数。与之前一样，我们可以通过模式匹配实现这一功能：

{% highlight hs %}

eval ctx (BlaiseList (x:xs)) =
        let (new_ctx, fn) = (eval ctx x)
            (last_ctx, eval_args) = mapAccumL eval new_ctx xs
            apply (BlaiseFn f) = f last_ctx eval_args
            apply (BlaiseSpecial f) = f new_ctx xs
        in apply fn

{% endhighlight %}

我们现在可以实现真正的赋值操作了。真正实现起来我们只需要3行代码：

{% highlight hs %}

blaiseSet ctx [(BlaiseSymbol s), e] =
        (Map.insert s eval_e new_ctx, eval_e)
            where (new_ctx, eval_e) = eval ctx e

{% endhighlight %}

完成！！！现在我们可以进行赋值操作并把变量应用于表达式了：

{% highlight lisp %}

(set i 5)
(+ i 1)
(set j (+ i 5))
(set j (+ j 1))

{% endhighlight %}

尽管这种改进花了我们很多文字去描述，真正的代码修改却十分少量。从中我们可以体会到Haskell的代码维护优越性——这里没有需要重定义的类，没有太多需要追中的依赖关系，而且更重要的是，我们不需要关心隐式的状态。haskell的类型系统让我们在编译期就能发现大部分修改代码引入的类型错误。我们同时不用担心打乱程序调用顺序或打乱成员状态的潜在危险——所有的状态是由我们自己管理的，所有的类型检测交由Haskell来管理。在接下来的两章的代码实现中我们更能体会到Haskell的代码维护优势。

# 使用Monads进行代码重构

之前一章可能让Java程序员和Haskell程序员同时蒙逼。我们必须要越过状态吗？答案即是是又是不是。事实上这种“对程序的每一个可能状态进行格式化”的必要性是一件好事——它实际上是让我们的程序减少了数据依赖同时增强了多态性，进而增强了程序的鲁棒性。真正的问题是在每一个函数需要时进行状态输出，这就需要花当量时间进行重复的工作，进而导致模板文件的产生。

幸运的是Haskell可以通过一种叫Monad的东西帮助我们避免这些麻烦。我不会在这篇文章里详细介绍Monad，如果你有兴趣网上有很多关于Monad的讲解。Monad的原理其实十分简单——Monad通过一种对Haskell类型系统重定义的方式来让我们对对模板代码进行抽象。例如，我们可以定义一个“状态”Monad来处理状态转换，进而我们可以把注意力放在更重要的地方。

我们可以使用`状态Monad`来对我们的解释器进行之前代码的状态清理。首先我们需要清理我们之前声明的状态传递：

{% highlight hs %}

data Expr = BlaiseInt Integer |
        BlaiseSymbol String |
        BlaiseFn ([Expr]->BlaiseResult) |
        BlaiseSpecial ([Expr]->BlaiseResult) |
        BlaiseList [Expr]

eval :: Expr -> BlaiseResult

{% endhighlight %}

我们通过`StateT`monad来定义`BlaisResult`进而处理IO：

{% highlight hs %}

type BlaiseResult = StateT Context IO Expr

{% endhighlight %}

现在我们就可以借用Monad重构我们的代码并避免状态转入的麻烦。`StateT`Monad事实上帮我们对这些麻烦事进行了抽象——我们只需对代码中的状态进行重构。当这个Monad被实现后，对其进行转入的代码只需写一次(`StateT`事实上是Haskell的标准Monad)。

{% highlight hs %}

blaiseSet [(BlaiseSymbol s), e] =
        do eval_e <- eval e
           modify (\sym_table->Map.insert s eval_e sym_table)
           return eval_e

eval (BlaiseInt n) = return (BlaiseInt n)
eval (BlaiseList (x:xs)) = do fn <- eval x
                              apply fn
    where apply (BlaiseSpecial f) = f xs
          apply (BlaiseFn f) = do args <- mapM eval xs
                                  f args

{% endhighlight %}

很多初学者都有一种误解，认为`Monads`是Haskell作为一种惰性(Lazy),无副作用的语言在处理求值运算时的让步。尽管Monad被用于处理IO, 这只是它的一种用法。当我在逐渐探询Haskell的本质时，我渐渐意识到Monad是程序员构建高级抽象的强有力工具。 Monads和高阶方程一起为我们提供了一种比模板更为强大的抽象。从这个角度看Monads与Lisp的Macro很像(尽管它们的本质很不一样)，相比较而言像Java这种语言通过冗长的类型来实现高阶方程完全缺少一种灵活的可选择性。很多语言特性(异常，接续等)都可以通过Monads或Macros实现。对于其他语言这些特性是否支持完全取决于编译器的开发人员——如果Sun不决定让Java支持接续特性我们就永远无法通过Java实现。

# 错误处理

Monads最引人入胜的语法特性可能就是`组合`了，这一特性让我们可以根据不同的目的混合，匹配不同的Monads来实现良好构建，可扩展的设计。

