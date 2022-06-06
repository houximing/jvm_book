# JVM，卷走面试官（一）—— JVM与插线板

JAVA这门语言有年头了，已经算是编程语言里面的中年人了。但至今依旧保持着无与伦比的风光，甚至在国内的互联网圈内，稳坐大哥大的宝座。这门语言入门不难，当然，所有语言入门都不难，虽然，笔者被一个叫做Ada的语言虐过，在此不提（还是必须得提一下）。

<img src="https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSNqJzibosoRhj4xyQp2bEj4ficv4Q9LrYQNoibDM9uAtcQNHMC3x6Rew3Ew/640?wx_fmt=png" alt="img" style="zoom:33%;" />

很多程序员都知道一个亘古不变的真理——PHP 是最好的语言。

<img src="https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSNOXbklh6cCX1FjoGeHFlKNvamY0hmP1PZgnYRkLiakicibawBxEVL3sFiag/640?wx_fmt=png" alt="img" style="zoom:25%;" />

的确，要想让PHP在某个机器上运行起来，我们需要根据PHP的版本，各种扩展的版本，操作系统的版本，去安装类似于LNMP或者LAMP的各种适配版本的环境。在不依托于类似Docker, Vagrant 或者是虚拟机镜像的前提下，换一台电脑，就需要从头到尾重新配置调试一遍，为了满足各个元素版本之间网状的依赖兼容关系，PHP开发者们大都见多识广，无所不能，所以它才被称作最好的语言。

但是Java 这门语言，在创造之初，就遵循“一次编译，随处运行”的傻瓜似的特点。说透彻点，就是你按照它的语法写了个最困难的程序Hello World，并在客户端进行了编译，那么编译好的那些东西，不仅仅在你本地的机器可以运行，而且可以在任何拥有版本兼容的Java 运行环境的机器上运行。我们就拿Java 8 举个优美的例子（虽然这是一个不太年轻的版本，但由于那个年代，我们用它产生了很多支撑首富的系统至今都还在用，所以这个版本还是值得学习和怀念的）。

35岁的小明平时开发使用7000块钱的WINDOWS 电脑，不喜欢996，所以他在有一天加班到夜里2点的时候，产生了不适。由于大脑缺氧，过度劳累，产生了幻觉，加上代码本身凌乱，于是他不由自主地用Java 8 写了一个死循环，如下图：

![img](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSNgBqrjRu8ZHblNjw1NAOpWYjCFLWO5WSGxc5V5wBSJ8TK6BWhzh5chw/640?wx_fmt=png)

第二天，他毕业了。这份神奇的代码被大家敬而远之，但接下来的接任者，喜欢996的25岁的小刚，不信邪，因为他使用的是2万块钱的苹果电脑，于是直接把这份代码拿到了自己电脑上，成功的安装了Java 8的运行环境，运行了一下，也毕业了。

所以证明，只要代码是同一份，运行环境版本兼容，那么它的疗效和开发者的年龄，经验，身体状况，心理行为，运行的时间，电脑的品牌，价格，是否喜欢996都没有什么关系。

 这就是Java “一次编译，随处运行”的精髓所在——即同一份代码，同一个运行环境，同一张毕业照。

![img](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSNP73LZrtUYR05oc0O0LiclHyZKuEhpZAL6dkXhSheUOficxk7lYV28sSw/640?wx_fmt=png)

 那么让Java 产生如此魅力的东西是什么呢？那就是我们接下来的几篇文章要关注的重点——JVM（Java Virtual Machine, Java 虚拟机）能领略JVM的威力之后，不仅能更加明白Java运行的底层原理，也能在工作和找工作中切换自如。

上面提到了一些词，什么编译，还客户端编译，编译后的一坨东西，又在这个JVM上运行等等碎片化的东西，都是个啥？它又怎么依赖我们本身的计算机去运行的呢？我们会在接下来的几篇文章中进行逐步讲解。

 先从简单的讲起，“一次编译，随处执行”到底是什么意思。这就好比，家家户户都有插线板，有人说叫配电盘，也有人说叫插排，甭管怎么说，就那么个东西，上面能插不同的插头。把它的插头，往旁边墙上的插座一插，那它上面不同的这些插口，就都能用了，甭管你是两相的台灯，还是三相的电饭锅，都能用上电。这些电器用的是电是从哪来的呢？很显然源头是从墙上的插座来的，插线板起到了适配转换的作用，这样无论家里的电器插头是否能插到墙上的插座里，只要它们符合插线板上提供的一系列插口的标准，那就能用上墙上插座的电。如下图所示，对，就是这么让人心旷神怡。

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSNMjXibVkkKatJnkOBuCoicfJLudEsAg2XicxiarTMDFbAD51dQjjGMQ093g/640?wx_fmt=jpeg" alt="img" style="zoom:33%;" />

对了，JVM，就是这个插线板，电脑就是那个插座。所以我们有些同学可能听说过有些语言和Java能混合编译调用，运行什么的，比如Scala, Groovy, Kotlin 等等，原因就是因为这些语言也是基于JVM这个插线板的标准去设计的，他们就如同电水壶，电暖气和热水器。 只要插到了同一个插线板上，他们就都能运行在这个插线板上，当然他们之间也可能会产生相互的影响，就如同这些电器同时开，应该会停电一样。

 那么有些同学问了，这些语言长得也不一样，那JVM是怎么去识别他们并运行他们的呢？难道我来一种新的语言，JVM就得去添加支持一种语言并大兴土木一番？这个问题其实也很简单，还从插线板上找答案，常用插线板上就那么两三种插孔，上面能插无数种电器，那是什么原因呢？插线板不需要知道你是什么电器，只需要提供这几种插孔，让电器的插头按照这个规定去设计就好了。如果JVM是插线板，语言是电器，那插头是什么呢？答案——字节码。那字节码又是哪来的呢？这就和前面的连上了，字节码是通过客户端编译器去产生的。一说客户端编译器，可能有些同学又不清楚了，那就用简单的一个命令来说明，如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbpto6k3E67WaAUkSiaMRSN3ogPvhdJ9gS9e9G3aLpOvCic5CTH0qaH4UsEytyxzXHODz7jWdqRQzw/640?wx_fmt=png)

这个可能只要接触过Java的技术人员就都见过了。此乃大名鼎鼎的Java之火焰——Javac编译器，字节码就是这个东西生成的。那字节码又是如何生成的，按照什么标准生成的呢？我们下一章见。

各位施主，求加公众号关注一下小弟。阿弥陀佛，佛法无边。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fAFyC36wKQMbtBExxdncGDceujVg4PxibVlnTfWVdM9ibickaDoZtibApFm2eoRQiac4gmkDAzAtUzmtib9CySJvnFKg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)