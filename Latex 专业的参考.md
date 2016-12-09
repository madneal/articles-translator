# Latex 专业的参考

tex对于论文写作或者其他的一些需要排版的写作来说，还是非常有意义的。我在网上看到这个对于Latex的入门介绍还是比较全面的，[Arbitrary  reference](http://latex.knobs-dials.com) .所以将会翻译出来，供初学者学习。

## TeX, LaTeX以及他们的用法

### Tex:排版

Tex是Donald Knuth发明的一种排版语言。排版意味着从结构性的文本到审美的一个标准。在Tex里面，你可以控制文本的根本结构，而不是像word一样，是软件阴影的帮你管理文件的结构，而紧紧给你提供一个图形化的接口。在TeX/LaTeX里面，对于布局和样式都有着默认的合理的设置但是却是可以控制的。

Tex对于希望控制文本的人是非常友好的，一部分可能是因为它不会帮助你不能控制的东西。当然，简便也是一方面的－当TeX工作正常的时候，它运行的很好并且容易明白，但是当它有问题的时候，将会很麻烦，将会变得难以处理。（这点我深有体会）如果你稍微脱离了他的运作方式或者期望一个不太容易实现的功能，你可能需要头痛一下了。

还有其他的一些优点和缺点。TeX的公式拍版是它的强项之一。在以前它可能是唯一较为正式的选择，即使像现在可以使用MathML或者MathType的时候，TeX也被认为是使用起来还是比较便利的。假如你以前还没有接触过TeX，那么你可尝试穿件下面的公式：

![formula](http://latex.knobs-dials.com/images/906ddf37149c974d31fda057893be08362952c3a.140.png)

在TeX仅仅使用`t) = h(t) \otimes g(t) \equiv \int h(\nu) g(t-\nu) d\nu`就可以搞定了。是不是看起来挺简单的。

 它的缺点可能是将它用于不是它的目的功能的地方。比如，你希望得到一些奇特的表格，你可能会遇到问题并发现一些替代的解决方法，不过这意味着你需要用一些相应的包并且按照其相应的方式去实现了。

### LATEX：文档编制

LaTeX是围绕TeX的一种文档编制的宏命令，它是一个比较高层次的能够利用它们产生一些比较常用的文档类型。它考虑到很多方面的内容，包括页眉和页尾，表格内容生成，自动两列的样式以及其他更多的细节。LaTeX现在被广泛得应用以及于它和TeX之间可以交换使用。TeX的实现方法大多数可以直接用在LaTeX的文档上面。当然你可以直接写TeX文件，但这不一定有什么用。如果希望看到更多的介绍，你可以参考[这个](http://www.latex-project.org/)。

### 书籍，报告以及PDF

LaTeX对于写文章，报告以及书籍有些基本的设置。

它之所以受到喜爱的原因可能是对于大型的文档不容易搞混样式，于其它的一些工具截然不同，比如微软的word，随着文档长度的增长，它的处理复杂度也在逐渐增长，如果你曾经参与过大型的共同合作的文档项目，那么你将深有体会。

近些年以来，pafTeX编译器变得流行起来，因为它可以直接编译pdf文件（不需要dvi以及ps文件作为媒介），使用`pdflatex` 而不是`latex`，你可使用一些针对pdf的特点。

### 包

在TeX和LaTeX里面有各种各样的包，你可以用一些比较怪异的符号，制作一些表格包括输和公式，写活页曲谱，做CAD图，确保首字母大写以及单词拼写正确，或者仅仅只是用一些特别的地方。

### 版本变化以及实现

LaTeX最近通用![latex-v](http://latex.knobs-dials.com/images/888853eb390825fa820d0158a102daae2941ac78.90.png)经常写作是LaTeX2e。我认为你是在使用这个版本的。通常这也无关紧要，但是有一些老的命令我不会特别提及。

注意这个版本以及老版本的LaTeX209是最通用的语言以及实现规范，并不针对特别的包或者实现。

免费的TeX软件包括通常的'teTeX'也叫做'TeX Live'对于unix系列的系统，以及对于windows系统的'MikTex'。你也可以参考下面相关的软件。

你可以参考维基百科里面的[TeX](http://en.wikipedia.org/wiki/TeX)以及[LaTeX](http://en.wikipedia.org/wiki/LaTeX)。[TUG](http://www.tug.org/)是近些年的一些开发工作和相关文档。

## 基本的使用

###  TeX会产生什么

最基本的来说，你会生成一个.tex文件，即你的文档，即`youfile.tex`。

运行`latex you file.tex`可以让TeX工作并且生成`your file.dvi`，这是当下的输出。dvi是一个独立的排版语言。

因为dvi不能够存储图像， 所以它经常被用来作为媒介步骤来产生文档。

dvi也可以被转化成pdf文件，所以pdf文件经常是可以立即生成的，不要dvi文件作为媒介可以直接调用`pdflatex yourfile.tex`。

### 输出文件:

LaTeX运行一次会产生很多文件，其中很多文件产生的原因是因为LaTeX是单流的;很多文件指示文档编译的信息文件，它们也能用于下次运行，当你编译文档的时候你还可以引用它们。比如，图的引用，章节的引用，以及其他文献的引用。鉴于此，它产生的数据是有特定用途的。(`.aux`文件是引用的，`toc`是给章头用的等等)，这些数据下次运行还可以继续使用。

注意这可能是你经常需要运行LaTeX两次来确保引用正常工作从而来更新它们。在一些特别变态的情况下，你甚至要运行更多次。你可以忽略掉这些额外的文件，你可以在生成文档后删除这些文件。

如果你用的是unix的系统，我建议你可以看看[rubber](http://www.pps.jussieu.fr/~beffara/soft/rubber/)。它的目的是为了在必要的时候重新编译文档。它不是特别简单的，但是能给你带来很多便利。

一个`.log`文件也会产生，它是tex文件编译产生的一些相关信息。

注意一点，一旦这个文件生成了，你无需担心保存除了原始数据(.text, .bib)其它的任何数据。log,aux, toc文件可能在运行之后看起来比较混乱，你也可以删掉它们。

### 其他可能输出的格式

除了在unix系统用于xdvi以及打印，dvi格式并不是很有用。上文曾经提到过，通过`dvips`软件可以将dvi文件转化为PostScript。你甚至可以先转成PS,然后再转成pdf文件，或者直接转成pdf文件。但是这些间接的步骤可能只是引发新的问题，在pdf方面经常不会怎么使用。更重要的是，这还会存在一个[字体渲染的问题](http://www.utdallas.edu/~cantrell/online/tex-pdf.html)。对于pdf，我建议你使用`pdflatex`或者类似的工具从而避免字体的一系列问题。你必须将所有的`.ps/.eps`文件转化为pdf，但这不是很困难的事。你可以在图片章节找到更多的细节。

## TeX语法

### TeX语法，编辑

你可能已经注意到，(La)TeX文档是蠢笨的基本上不包含什么具有特殊意义的符号，经常是依赖环境的并且很容易就可以看得出来。下面有一段LaTeX的代码，你也不用担心你还读不懂它，因为它可能包含不少的特别的符号：

```latex
I am text. Yes.

%comment: a semi-complex table with math in it:
\begin{tabular}{|l|r|}
 \hline
 $a_1~~~b$ & $\sqrt[3]{a_1^2}$ \\
\end{tabular}
```

最终产生的表格的排版是这个样子的![tab](http://latex.knobs-dials.com/images/7d5f9100c96107ce945caf17f5f4092cc2285b92.110.png)

### 特殊符号的总结

* **{**和**}**是作为一些命令参数来定义一些小块，比如临时的粗黑体在`{\bf bold}`

* **$**是用来开始和结束数学模式的，比如一些公式啊，数字之类的。你可以在你文本的任何地方插入`$a+b=c$`，输入`$$a+b=c$$`，那么你的公式就会在段与段之间以块的形式展现。

* **%**是用来注释的，这个是单行注释。如果你要注释大段的代码的时候，为了避免插入过多的百分号，你可以把这些字符放在`\iffalse`和`\fi`里面。

* **_**和**^**分别作为下标和上标。你也可以同时使用上标和下标，比如：![formula1](http://latex.knobs-dials.com/images/16ac076820e1df6038500ee08ee65d76ed316e47.120.png)

* **~**是一个硬空格，它对于排版是有影响的，它是具有大小的，并且不可分连的空格，就像&nbsp一样的。它很有用比如：`A.~Smith`以及在引用的图表的时候`Figure~\ref{dataflow}`,这确保了作者姓名或者图片和数字之间不会在行与行之间分隔。（也可以使用其他的办法来解决这个问题，比如mbox，不会强制使用特殊的空格大小）

* 实际上，`\ `经常和`~`拿起来一起来使用。尽管这两者之间还是有区别的：`\ `是字间的空格，经常用来告诉LaTeX这不是句子的末尾，一般用于缩写或者标题。(`Dr.\ Jones`)

* **&**适用于在数组以及表格中定义列的。

* **\**用于开始一个命令。有一些可能是比较特殊的(`\\`用于换行，`\>`用于tab缩进)，一般化的话应该是这样的`\commandname`。当然这可能会有看起来不太相同的使用方法：

  * 一次效果函数，比如使用`\ss`来获得一个德国字母![\ss](http://latex.knobs-dials.com/images/9ffef8f0b5c7c54c24e674529f21ea9d238ee17a.90.png)。

  * 状态改变，比如粗体，强调，比如`text-{\em a-tron}`会产生![text-{\em a-tron}](http://latex.knobs-dials.com/images/acef73a11eb4d5db7c58a2e90c9fbf3275036f59.90.png)。（花括号是来限制作用的范围的）

  * 使用命令取得相应的值，一般是使用`{}`或者`[]`。比如：

    * `\textsc{SmallCaps}`产生![\textsc{SmallCaps}](http://latex.knobs-dials.com/images/b6ee9181b90a3df999713ea8d56ccd18b12e5a0d.90.png)
    * `\caption{Description`用于标题说明，一般用于图表。
    * 口音和发声符号，比如`\'{e} \v{o}`来产生![\'{e} \v{o}](http://latex.knobs-dials.com/images/adbf6cdf913ff33772131646daeace9f54f16083.90.png)

  * 使用`\begin`和`end`是定义环境，从而和其他内容区分处理，比如：

    ```latex
    \begin{verbatim}
    In the verbatim environment, 
     text appears with almost no treatment.
    There's also no need for manual TeX newlines (\\)
    \end{verbatim}
    ```

    会产生![ \begin{verbatim}In the verbatim environment,  text appears with almost no treatment.There's also no need for manual TeX newlines (\\)\end{verbatim}](http://latex.knobs-dials.com/images/3d054b70734c3a9a9e04195236946b4b7cfa29a8.90.png)

  这些命令有选择项和参数项，对于每一个命令都有着相应的设置。有一些命令定义后，你可以用几种方式使用，但是一般的使用时选择项在参数项之前，比如对于`\command[option1,option2]{argument}`你可以用`\comman{argument}`作为基本使用。

* **#**是在内部使用的，比如`\newcommand`


为了在文本里面使用上述的一些字符，你需要添加反斜杠使用`\$ \{ \% \} \_ \#`从而产生![\$ \{ \% \} \#](http://latex.knobs-dials.com/images/7a2cf6db5f01b41a35128ca1498d4f967c51dbf2.100.png)

这里也有几个意外情况，`\\`是一个字面上的换行，`\~`是一个插入符号。

对于反斜杠如何表示，可以使用`$\backslash$`

对于其他的一些插入符号，你可以将参数不加设置，`\~{}, \^{}`，这样也能获得你想要的比如![foo \^{} bar \~{} quu #](http://latex.knobs-dials.com/images/a63f4e5a026678d6b477830afe06134302a26aee.100.png)

对于等宽字体你也可以使用inline verbatim，比如`\verb|^|, \verb|~|, \verb|\|`

### 关于波浪字符和插入字符更多信息

为了在URLs里面使用波浪符号，你可以使用url包，这个可以为你处理任何事情：波浪符号会被当做一个波浪符号而不是TeX里面的空格，它复制一个波浪符号而不是空格，这个URLs也是可以点击的。

为了在非URL文本里面获得波浪符号，当然还有其他的办法，比如[swung dash](https://en.wikipedia.org/wiki/Dash#Swung_dash)

* 你可以获得一个不一样的波浪符号（在空格之上）通过使用`\~{}, \textasciitilde, \char \~`。这个波浪符号位置比较高，大多数人并不喜欢用。
* 如果你希望在等宽字体里面使用波浪符号，一个简单的方法是使用verbatim环境，可能没有内联的使用起来那么方便`\verb|foo/~var`![\verb|foo/~bar|](http://latex.knobs-dials.com/images/7dc8b48bcff8d1d865a6bcb1e9e5c5c9dac04171.90.png)
* `texttidlebelow`（依赖包textcomp）的位置更低，但是不能够以波浪符号粘贴复制。它在某些字体面，位置可能特别低，这个可能和字体相关。
* `$\sim$`对于大多数情况来说就不太常用了，一般在数学环境里面用的比较多。

如果你不想使用宏命令，你也可以自己创建一个波浪符号，自己来调整位置和样式。比如：

你可以提高`\sim`波浪符号的位置通过`{\raise.17ex\hbox{$\scriptstyle\sim$}}`

你也可以降低波浪符号的位置，在`\mathtt`里面看起来更精细，你可能比较偏向于在普通文本中使用。在`texttt`看起来更粗，对于等宽字体显示效果比较好。比如，你可以定义：

`\newcommand\thintilde{{\lower.92ex\hbox{\mathtt{\char`\~}}}}`

`\newcommand\thicktilde{{\lower.74ex\hbox{\texttt{\char`\~}}}}`

你可以产生`a\thintilde b\thicktilde c`看起来就是这样![a\thintilde b\thicktilde c](http://latex.knobs-dials.com/images/ddfaf8eb96829130e1b80448ad6e642a4b7ff77e.130.png)

作为对比：

![\begin{tabular}{lll} & & result \\\hline\verb#url# package                                         & & \url{foo/~bar~} \\\verb|\~{}| ~and~ \verb|\textasciitilde|                  & & foo/\~{}bar\~{} \\\verb|{\tt \~{}}| ~and~ \verb|\textt{\~{}}|           & & foo/{\tt \~{}}bar{\tt \~{}} \\simple {\hspace{-.25ex}\lower.72ex\hbox{\texttt{\~{}}}} in verbatim environment                            & & \verb|foo/~bar| \\\verb#\texttildelow#                                      & & foo/\texttildelow bar\texttildelow \\basic \verb#$\sim$#                                          & & foo/{$\sim$}bar{$\sim$} \\~\\~~~~~~~~~~~~~~~~~~~~~~~mucking about: \\tweaked \verb|\sim|                                        & & foo/{\raise.17ex\hbox{$\scriptstyle\sim$}}bar{\raise.17ex\hbox{$\scriptstyle\sim$}} \\tweaked \verb|\sim| {\small (looks in monospace context)}  & & {\tt foo/{\raise.17ex\hbox{$\scriptstyle\sim$}}bar{\raise.17ex\hbox{$\scriptstyle\sim$}} } \\lowered diacritic tilde, mathtt                             & & {foo/{\lower.92ex\hbox{\mathtt{\char`\~}}}bar{\lower.92ex\hbox{\mathtt{\char`\~}}} } \\lowered diacritic tilde, texttt                             & & {\tt foo/{\lower.74ex\hbox{\texttt{\char`\~}}}bar{\lower.74ex\hbox{\texttt{\char`\~}}} } \\\hline\end{tabular}](http://latex.knobs-dials.com/images/b65770c89060158a3aa055b4bb7b449089290ca2.130.png)

当你希望使用一个字面上的插入符号，`\^{}`是一个高的发音符号，你也可以在verbatim样式里面使用，比如：

![\verb|x=x^2|](http://latex.knobs-dials.com/images/6464cb2daeb065351c0f35657bff30fa9a38f0f9.120.png)

### 文档设置

一个小的文档看起来是这个样子的：

```latex
\documentclass{article}
\begin{document}
Text
\end{document}
```

在一些老的教材里面你可以使用`documentstyle`，这种用法是比较古老的，在LaTeX2009里面。

一个更实用一点的文档可能看起来如下：

```latex
\documentclass[a4paper]{article}
 
%package imports and document options go here
\usepackage{url}   %I occasionally use URLs in footnotes, this helps.
 
%useful if you use \maketitle, or want it to end up in the document file's metadata
\title{I am Sam}
\author{Dr. Seuss}
\date{1960}
 
 
%%%% the actual document %%%%
\begin{document}
\maketitle
 
I do not like green eggs and ham.
\end{document}
```

这个编译后就可以产生[eggs.pdf](http://latex.knobs-dials.com/eggs.pdf)

在`\begin{document}`之前的命令是前言部分，里面是一些包的导入以及命令的重新定义。

#### 文档类

文档有种类也有一个类，这是控制文档制作的基本样式，包括是否显示章类型，给你提供了这样的命令，比如`\section`。基本的文档类有：

* `article`对于几页的文章
* `proc`对于会议记录
* `report`对于篇幅较长的报告
* `book`书籍之类的
* `letter`可以添加一些命令比如地址，开头，收信人，结束语等等。可以看看[.ps](http://latex.knobs-dials.com/letter.ps)以及[.pdf](http://latex.knobs-dials.com/letter.pdf)，这些都是用[.tex](http://latex.knobs-dials.com/letter.tex)生成的。

注意像`\part, \chapter, \section, \subsection, \subsubsection, \paragraph, \subparagraph, \appenix`这些命令需要根据文档的类使用在其响应的文档中。

有很多文档类都是默认安装的，这取决于你设置的版本或者一些其他的默认设置：

* `exarticle, extreport, extbook, extletter, extproc`这些就是用来定制化的一些变量，比如让你可以改变全局字体大小。
* `amsart, amsbook, amsproc`这些都是来自于美国数学学会。大多数是用于格式的改变，可以和相应的互换使用，具体可以参考[amslatex介绍](ftp://ftp.ams.org/pub/author-info/documentation/amslatex/instr-l.pdf)
* 对于投影仪或者开销表你可以使用：
  * `slides`和一些新的浏览器可以在一起正常工作，你可以做一个类似于ppt之类的展示。
  * `beamer`
  * `prosper`是一个基于pdfTeX的幻灯片，它具有幻灯变切换，可点击导航，动画，样式可能有一点点难看。可以在[这](http://freshmeat.net/articles/view/667/)看到一些例子。
  * `foils`如果你已经安装[FoilTeX](http://www.ctan.org/tex-archive/help/Catalogue/entries/foiltex.html)
* `esam`主要是针对于考试问题来做一些布局，处理打分系统，可以参考[在档类中使用exam](http://www.google.com/search?q=using+the+exam+document+class)
* `minimal`
* `arbitrary`

#### 文档类选项

每一个文档种类都会识别一组参数。每一个文档类的选项的支持情况取决于常识以及文档对于某些选项是否做过相应的更新。命令选项通常支持包括：

* `10pt, 11pt, 12pt`来控制基本的字体大小。默认的是10pt。
* `draft`是用来找那些过满的hbox。
* 页相关的
  * `a4paper, a5paper, b5paper, letterpaper, legalpaper`页面大小
  * `lanscape`文档横向打印方向
  * `twoside`是对于左右分栏风格
  * `openany`强制章节从右边开始，这可能会长生空页。
* 公式相关的
  * `fleqn`左对齐公式(\$\$fomula\$\$)
  * `leqno`改变公式数字的对齐
  * `centertags, tbtags`

比如，如果你想要打印A5大小的小册子，你可能会使用：

`\documentclass[9pt,twoside,a5paper]{extbook}`

#### 头和尾

简单的头尾控制可以使用`\pagestyle`来控制。你也可用`\pagestyle{empty}`来禁用。你可以使用`\pagestyle{headings}`，这样就可以使用实现定义的头部样式，这也取决于文档的样式。默认的样式是`\pagestyle{plain}`一些文档类提供更多的选项。

这还有个页面样式，叫做`myheadings`，但是通常是用`fancyhdr`来代替，因为这样更加定制化。

为了改变头部附近的间隔，你可以使用`\addtolength, \headheight`来改变，但是使用几何包可能会更简。