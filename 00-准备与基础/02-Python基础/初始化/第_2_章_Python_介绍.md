[Download Python](https://www.python.org/downloads/)

[01-Python简介.pdf](https://www.yuque.com/attachments/yuque/0/2025/pdf/2639475/1753081352426-34c4eba0-adf7-4867-b6b1-0c5ec3705453.pdf)

# <font style="color:rgb(0,0,0);">简介 </font>
<font style="color:rgb(0,0,0);">Python 语言是少有的一种可以称得上即简单又功能强大的编程语言。你将惊喜地发现 Python 语言是多么地简单，它注重的是如何解决问题而不是编程语言的语法和结构。 </font>

<font style="color:rgb(0,0,0);">Python 语言的官方介绍是： </font>

<font style="color:rgb(0,0,0);">Python 是一种简单易学，功能强大的编程语言，它有高效率的高层数据结构，简单而有效地实现面向对象编程。 Python 简洁的语法和对动态输入的支持，再加上解释性语言的本质，使得它在大多数平台上的许多领域都是一个理想的脚本语言，特别适用于快速的应用程序开发。 </font>

<font style="color:rgb(0,0,0);">我会在下一节里详细地讨论 Python 的这些特点。 </font>

**<font style="color:rgb(0,0,0);">注释： </font>**

<font style="color:rgb(0,0,0);">Python 语言的创造者 Guido van Rossum 是根据英国广播公司的节目“蟒蛇飞行马戏”命名这个语言的，并非他本人特别喜欢蛇缠起它们的长身躯碾死动物觅食。 </font>

# <font style="color:rgb(0,0,0);">Python 的特点 </font>
1. <font style="color:rgb(0,0,0);">简单 </font>

<font style="color:rgb(0,0,0);">Python 是一种代表简单主义思想的语言。阅读一个良好的 Python 程序就感觉像是在读英语一样，尽管这个英语的要求非常严格！ Python 的这种伪代码本质是它最大的优点之一。它使你能够专注于解决问题而不是去搞明白语言本身。 </font>

2. <font style="color:rgb(0,0,0);">易学 </font>

<font style="color:rgb(0,0,0);">就如同你即将看到的一样， Python 极其容易上手。前面已经提到了， Python 有极其简单的语法。 </font>

3. <font style="color:rgb(0,0,0);">免费、开源 </font>

<font style="color:rgb(0,0,0);">Python 是 FLOSS（自由/开放源码软件）之一。简单地说，你可以自由地发布这个软件的拷贝、阅读它的源代码、对它做改动、把它的一部分用于新的自由软件中。 FLOSS 是基于一个团体分享知识的概念。这是为什么 Python 如此优秀的原因之一 , 它是由一群希望看到一个更加优秀的 Python 的人创造并经常改进着的。 </font>

4. <font style="color:rgb(0,0,0);">高层语言 </font>

<font style="color:rgb(0,0,0);">当你用 Python 语言编写程序的时候，你无需考虑诸如如何管理你的程序使用的内存一类的底层细节。 </font>

5. <font style="color:rgb(0,0,0);">可移植性 </font>

<font style="color:rgb(0,0,0);">由于它的开源本质， Python 已经被移植在许多平台上（经过改动使它能够工作在不同平台上）。如果你小心地避免使用依赖于系统的特性，那么你的所有 Python 程序无需修改就可以在下述任何平台上面运行。 </font>

<font style="color:rgb(0,0,0);">这些平台包括 Linux、Windows、FreeBSD、Macintosh、Solaris、OS/2、Amiga、 AROS、AS/400、BeOS、OS/390、z/OS、Palm OS、QNX、VMS、Psion、Acom RISC OS、VxWorks、PlayStation、Sharp Zaurus、Windows CE 甚至还有 PocketPC！ </font>

6. <font style="color:rgb(0,0,0);">解释性 </font>

<font style="color:rgb(0,0,0);">这一点需要一些解释。 </font>

<font style="color:rgb(0,0,0);">一个用</font>**<font style="color:rgb(0,0,0);">编译性语言</font>**<font style="color:rgb(0,0,0);">比如 </font>**<font style="color:rgb(0,0,0);">C 或 C++</font>**<font style="color:rgb(0,0,0);"> </font>**<font style="color:rgb(0,0,0);">(还有Java和go</font>**<font style="color:rgb(0,0,0);">) 写的程序可以从源文件（即 C 或 C++ 语言）转换到一个你的计算机使用的语言（二进制代码，即 0 和 1）。这个过程通过编译器和不同的标记、选项完成。当你运行你的程序的时候，连接/转载器软件把你的程序从硬盘复制到内存中并且运行。 </font>

<font style="color:rgb(0,0,0);">而 </font>**<font style="color:rgb(0,0,0);">Python 语言写的程序不需要编译成二进制代码。你可以直接从源代码运行程序</font>**<font style="color:rgb(0,0,0);">。在计算机内部， Python 解释器把源代码转换成称为字节码的中间形式，然后再把它翻译成计算机使用的机器语言并运行。事实上，由于你不再需要担心如何编译程序，如何确保连接转载正确的库等等，所有这一切使得使用 Python 更加简单。由于你只需要把你的 Python 程序拷贝到另外一台计算机上，它就可以工作了，这也使得你的 Python 程序更加易于移植。 </font>

7. **<font style="color:rgb(0,0,0);">面向对象 </font>**

<font style="color:rgb(0,0,0);">Python 即支持面向过程的编程也支持面向对象的编程。在面向过程的语言中，程序是由过程或仅仅是可重用代码的函数构建起来的。在面向对象的语言中，程序是由数据和功能组合而成的对象构建起来的。与其他主要的语言如 C++ 和 Java 相比， Python 以一种非常强大又简单的方式实现面向对象编程。 </font>

8. <font style="color:rgb(0,0,0);">可扩展性 </font>

<font style="color:rgb(0,0,0);">如果你需要你的一段关键代码运行得更快或者希望某些算法不公开，你可以把你的部分程序用 C 或 C++ 编写，然后在你的 Python 程序中使用它们。 </font>

9. <font style="color:rgb(0,0,0);">可嵌入性 </font>

<font style="color:rgb(0,0,0);">你可以把 </font><font style="color:rgb(0,0,0);">Python </font><font style="color:rgb(0,0,0);">嵌入你的 </font><font style="color:rgb(0,0,0);">C/C++ </font><font style="color:rgb(0,0,0);">程序，从而向你的程序用户提供脚本功能。 </font>

10. <font style="color:rgb(0,0,0);">丰富的库 </font>

<font style="color:rgb(0,0,0);">Python 标准库确实很庞大。它可以帮助你处理各种工作，包括正则表达式、文档生成、单元测试、线程、数据库、网页浏览器、CGI、FTP、电子邮件、XML、XML-RPC、HTML、WAV 文件、密码系统、GUI（图形用户界面）、Tk 和其他与系统有关的操作。记住，只要安装了 Python ，所有这些功能都是可用的。这被称作 Python 的“功能齐全”理念。 </font>

<font style="color:rgb(0,0,0);">除了标准库以外，还有许多其他高质量的库，如 wxPython 、Twisted 和 Python 图像库等等。 </font>

<font style="color:rgb(0,0,0);">Python 确实是一种十分精彩又强大的语言。它合理地结合了高性能与使得编写程序简单有趣的特色。 </font>

# <font style="color:rgb(0,0,0);">为什么不选 Perl？ </font>
<font style="color:rgb(0,0,0);">也许你以前并不知道，Perl 是另外一种极其流行的开源解释性编程语言。如果你曾经尝试过用 Perl 语言编写一个大程序，你一定会自己回答这个问题。在规模较小的时候， Perl 程序是简单的。它可以胜任于小型的应用程序和脚本，“使工作完成”。然而，当你想开始写一些大一点的程序的时候， Perl 程序就变得不实用了。我是通过为 Yahoo 编写大型 Perl 程序的经验得出这样的总结的！ </font>

<font style="color:rgb(0,0,0);">与 Perl 相比， Python 程序一定会更简单、更清晰、更易于编写，从而也更加易懂、易维护。我确实也很喜欢 Perl，用它来做一些日常的各种事情。不过当我要写一个程序的时候，我总是想到使用 Python ，这对我来说已经成了十分自然的事。 </font>

<font style="color:rgb(0,0,0);">Perl 已经经历了多次大的修正和改变，遗憾的是，即将发布的 Perl 6 似乎仍然没有在这个方面做什么改进。我感到 Perl 唯一也是十分重要的优势是它庞大的 CPAN 库——综合 Perl 存档网络。就如同这个名字所指的意思一样，这是一个巨大的 Perl 模块集，它大得让人难以置信 —— 你几乎用这些模块在计算机上做任何事情。 Perl 的模块比 Python 多的原因之一是 Perl 拥有更加悠久的历史。当然，随着 Python 包索引的增加这也在不断变化。 </font>

# <font style="color:rgb(0,0,0);">为什么不选 Ruby？ </font>
<font style="color:rgb(0,0,0);">也许你以前并不知道， Ruby 是另外一种极其流行的开源解释性编程语言。如果你已喜欢并且已经使用 Ruby ，那么我明确地给你说应该继续使用它。对那些没使用过 Ruby ，正试着判断该学 Python 还是 Ruby 的人来讲，我会推荐 Python ，纯粹是因为从易学的角度来考虑。就我个人而言，让我来喜欢 Ruby 是很困难的事情，但对理解 Ruby 的人来说，他们总是会赞赏 Ruby 的优美。非常不幸，我不是这种幸运儿。 </font>

# <font style="color:rgb(0,0,0);">程序员都说些什么 </font>
<font style="color:rgb(0,0,0);">读一下像 ESR 这样的超级电脑高手谈 Python 的话，你会感到十分有意思： </font>

+ <font style="color:rgb(0,0,0);">Eric S. Raymond 是《The Cathedral and the Bazaar》的作者、“开放源码”一词的提出人。他说 Python 已经成为了他最喜爱的编程语言。这篇文章也是促使我第一次接触 Python 的真正原动力。 </font>
+ <font style="color:rgb(0,0,0);">Bruce Eckel 著名的《Thinking in Java》和《Thinking in C++》的作者。他说没有一种语言比得上 Python 使他的工作效率如此之高。同时他说 Python 可能是唯一一种旨在帮助程序员把事情弄得更加简单的语言。请阅读完整的采访以获得更详细的内容。 </font>
+ <font style="color:rgb(0,0,0);">Peter Norvig 是著名的 Lisp 语言书籍的作者和 Google 公司的搜索质量主任（感谢Guido van Rossum 告诉我这一点）。他说 Python 始终是 Google 的主要部分。事实上你看一下 Google 招聘的网页就可以验证这一点。在那个网页上，Python 知识是对软件工程师的一个必需要求。 </font>

# <font style="color:rgb(0,0,0);">关于 Python 3.0 </font>
<font style="color:rgb(0,0,0);">Python 3.0 是该编程语言的新版。有时又被用为 Python 3000 或 Py3K 。</font>

<font style="color:rgb(0,0,0);">本版更新的主要原因就是为了除去过去几年积累的一些问题，并且使得本语言更清晰。 </font>

<font style="color:rgb(0,0,0);">如果你已有大量的 Python 2.x 的代码，那么这儿有个工具能帮助你将 2.x 转换成 3.x 的源代码。（http://docs.python.org/dev/3.0/library/2to3.html） </font>

<font style="color:rgb(0,0,0);"></font>

<font style="color:rgb(0,0,0);">更多细节： </font>

<font style="color:rgb(0,0,0);">• Guido van Rossum’s introduction </font>

<font style="color:rgb(0,0,0);">(</font>[http://www.artima.com/weblogs/viewpost.jsp?thread=208549](http://www.artima.com/weblogs/viewpost.jsp?thread=208549)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">• What’s New in Python 2.6 </font>

<font style="color:rgb(0,0,0);">(</font>[http://docs.python.org/dev/whatsnew/2.6.html](http://docs.python.org/dev/whatsnew/2.6.html)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">(features signifificantly different from previous Python 2.x versions and most likely will </font>

<font style="color:rgb(0,0,0);">be included in Python 3.0) </font>

<font style="color:rgb(0,0,0);">• What’s New in Python 3.0 </font>

<font style="color:rgb(0,0,0);">(</font>[http://docs.python.org/dev/3.0/whatsnew/3.0.html](http://docs.python.org/dev/3.0/whatsnew/3.0.html)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">• Python 2.6 and 3.0 Release Schedule </font>

<font style="color:rgb(0,0,0);">(</font>[http://www.python.org/dev/peps/pep-0361/](http://www.python.org/dev/peps/pep-0361/)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">• Python 3000 (the offificial authoritative list of proposed changes) </font>

<font style="color:rgb(0,0,0);">(</font>[http://www.python.org/dev/peps/pep-3000/](http://www.python.org/dev/peps/pep-3000/)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">• Miscellaneous Python 3.0 Plans </font>

<font style="color:rgb(0,0,0);">(</font>[http://www.python.org/dev/peps/pep-3100/](http://www.python.org/dev/peps/pep-3100/)<font style="color:rgb(0,0,0);">) </font>

<font style="color:rgb(0,0,0);">• Python News (detailed list of changes) </font>

<font style="color:rgb(0,0,0);">(</font>[http://www.python.org/download/releases/3.0/NEWS.txt](http://www.python.org/download/releases/3.0/NEWS.txt)<font style="color:rgb(0,0,0);">) </font>





