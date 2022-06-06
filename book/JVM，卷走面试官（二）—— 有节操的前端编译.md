# JVM，卷走面试官（二）—— 有节操的前端编译

上篇讲到，JVM就如同插线板，那能适配插线板的插头——这个字节码又是如何产生的呢？我们本篇内容会进行如同新闻联播前十分钟一样具体地阐述，深入贯彻，学习领会，并发扬光大。



首先谈点逻辑思维的问题，比如我们写了一段充满节操的代码：

```java

public class NewEra {

    public static void main(String[] args) {
        int start = 1921;
        int latestYearOfCelebration = 2021;
        System.out.println("热烈庆祝永远伟大光荣正确的xxxxx
                           	" + yearOfGlory(start, latestYearOfCelebration) + "岁生日");
    }

    /**
     * 无需注释！！！
     */
    private static int yearOfGlory(int start, int latestYearOfCelebration) {
        return latestYearOfCelebration - start;
    }

}
```

那么，每个有党性，有良知的中国人，无论你会不会代码，你也能明白这段代码的深远意义。但是，JVM是没有党性的，所以，它不明白这段代码的含义，这就造成了一个问题，我们无法感化它。但是我们深刻地知道，它认识字节码，于是，我们必须找到一种路线和方法，让这段洋溢着百年奋斗史的代码，变成也能够被JVM认识的字节码，那么我们必须找到一个耐心的翻译去把这段代码翻译成JVM听得懂的语言。那么这个过程就叫编译，这个翻译就叫做编译器。对于JAVA来说，从人能看懂的字母到JVM能看懂的字节码这个过程，我们通常叫做前端编译，编译器通常使用的就是javac。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibV0fShVEXpQ4Vuricvc6cCwYfz2zmBlmiaYs7NQRadwzIibrSrbw8EkNAg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





接下来，我们就隆重介绍一下这位翻译。首先有一个问题，这位翻译它在哪呢？找来找去找不到，领导都来了，翻译鸽了。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibqicmorGQzrQdUcoF4B9xH1N4gVpaTHx2SAYkHrPmAY1gHA8bIUp0kCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



首先，作为Java 程序员，进行JAVA开发的时候，必备条件，就是在开发环境中安装JDK，即JAVA开发工具包，它的里面又包含了JRE，即JAVA运行环境，我们的主角JVM就在JRE里面，我们的翻译官同志呢，由于职务的特殊性，放在了JRE的外围，如图红圈所示：



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibAJhOdLqpibcQHTrPyOxRCJwaYYaiaMAuibJYuPUNyibGKoCsyKfIVo4Imw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



所以，我们只需要从JDK的里面刨土豆，就能把它给找到。对于JAVA 8来说，它就藏在这里：“JDK 目录下面的tools.jar (新版本已经换地方了，本文仅以JAVA 8为例)，然后找到com.sun 目录，然后再找到tools 目录，然后就能看到javac 了”。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4Pxib4J9RCG8SSQP9sJYdy2bo8cjDibhyJVbC5lIpSvaIaEW6Mib8aEbaVGug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



随便找个文件点进去，看看热闹。。。。

```java

@jdk.Exported
public class Main {

    public static void main(String[] args) throws Exception {
        System.exit(compile(args));
    }

    public static int compile(String[] args) {
        com.sun.tools.javac.main.Main compiler =
            new com.sun.tools.javac.main.Main("javac");
        return compiler.compile(args).exitCode;
    }
    
    public static int compile(String[] args, PrintWriter out) {
        com.sun.tools.javac.main.Main compiler =
            new com.sun.tools.javac.main.Main("javac", out);
        return compiler.compile(args).exitCode;
    }
}
```



看着挺熟悉的，很像什么语言来着？哦，对了，像JAVA。

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibgfW3V7z8yzJlzRrRbrmCzJiah6R1MCAfGFntokppKyPZe87OjxdcLjA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibAavBVibghibmtuNzDCr3QS2OSXf0DqaWYMJnicFGbKxiaLrwtbQvxYo6aw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



把java代码编译成字节码的编译器，是用java 写的。。。。。。那就是说翻译的外语启蒙老师是翻译自己。这不禁让人想起，VPN人员因为登不上VPN而解决不了VPN登录不上的问题，以及自己亲自去办事需要出示自己是自己的证明的问题。



这一类的问题，在哲学的范畴中，称为自举，其实也很常见，很好理解，解答自举问题的关键，借助自举元循环外的可能性，提供触发循环的能量。那么对于第一版的骨灰级java的编译器来说，它也必然发生在java能够编译之前，可以肯定的是，它不是用java写的，这里讲个故事：JAVA的前身叫Oak 语言，Java 之父James Gosling 最初用C语言写了一个Oak 的编译器，为了实现一个成熟语言的自举性，一个叫Arthur van Hoff 的30多岁的年轻人在一个阳光明媚的晚上用Oak 写了一个据说能编译Oak 语言的编译器，然后用最初的C写的编译器编了一下，咦，过了！于是Oak 语言的第一个自举编译器就完成了，于是后面Oak就可以自己不断地自我翻译，自我审阅，自我批评，自我改进。过了一段这门语言就被改名字为JAVA了。之后Arthur van Hoff 看着自己即将35岁的年龄，他没有选择去送外卖，而是选择了创业，如今进了苹果做了VR的高级架构师。



废话太多，收住——————————

 

那么，它是如何去翻译的呢？



众所周知，要想当好翻译官，有几个众所周知的必要因素，前提是，两种语言都是有限集合，其次那就是对自己的语言熟悉，对目标语言熟悉，以及精通两个语言的彼此语义对应的映射关系。更出色一点的翻译，是可以整合全局的逻辑，同时进行标准化模板化抽象加工，从而更有效率地进行翻译，需要补充的一点，就是翻译官所翻译的原文也必须是在一定程度上符合源语言的语法，能够基本准确表达的。所以，要想使javac 当好这个翻译官，用计算机的语言来描述，需要满足以下条件：



- Java 语言 和字节码语言的语法是有标准的，并且语义集合是有限的
- Javac 能够精确解析java 代码
- Java 代码本身必须符合Java的语法和使用规范
- Javac 需要有一个完整的从java代码到字节码的映射
- Javac 能够精确输出目标结果——字节码
- Javac 能够完整理解java 代码的整体逻辑表达



这里面的第1条是一个事实存在，然后第2，3，4，5看着倒不复杂，只需要用普通的规则去进行解析和映射就完事了，但是最后一条貌似有一些复杂。所以带着这些结论和问题，我们去javac 的源码去验证和寻找答案。

 

首先，我们进入到了javac 的源码，看到的是一片虚无，找到main 函数后开始组织专项调查组进行系统性调研，经过了逐级地排查后，发现答案的关键就都在这里了。

```java

public void compile(List<JavaFileObject> sourceFileObjects,
                        List<String> classnames,
                        Iterable<? extends Processor> processors)
    {
        if (processors != null && processors.iterator().hasNext())
            explicitAnnotationProcessingRequested = true;
        // as a JavaCompiler can only be used once, throw an exception if
        // it has been used before.
        if (hasBeenUsed)
            throw new AssertionError("attempt to reuse JavaCompiler");
        hasBeenUsed = true;

        // forcibly set the equivalent of -Xlint:-options, so that no further
        // warnings about command line options are generated from this point on
        options.put(XLINT_CUSTOM.text + "-" + LintCategory.OPTIONS.option, "true");
        options.remove(XLINT_CUSTOM.text + LintCategory.OPTIONS.option);

        start_msec = now();

        try {
            initProcessAnnotations(processors);

            // These method calls must be chained to avoid memory leaks
            delegateCompiler =
                processAnnotations(
                    enterTrees(stopIfError(CompileState.PARSE, parseFiles(sourceFileObjects))),
                    classnames);

            delegateCompiler.compile2();
            delegateCompiler.close();
            elapsed_msec = delegateCompiler.elapsed_msec;
        } catch (Abort ex) {
            if (devVerbose)
                ex.printStackTrace(System.err);
        } finally {
            if (procEnvImpl != null)
                procEnvImpl.close();
        }
    }
```

这几十行代码经过了二十年的变化，不知道还有多少保留了Arthur van Hoff那个夜晚的杰作。



经过我们深入挖掘，发现真正核心的东西是这两句：

```java
// These method calls must be chained to avoid memory leaks
delegateCompiler =
  processAnnotations(
  enterTrees(stopIfError(CompileState.PARSE, parseFiles(sourceFileObjects))),
  classnames);

delegateCompiler.compile2();
```



那我们就来说道说道，这冰山的一角，是如何实现如此强大影响力的。

从方法上来看，很明显我们有先后几个方法步骤



- parseFiles
- enterTrees
- processAnnotations
- compile2



但是compile2 这个名字与其它三个的命名方式相比是与众不同的，事出反常必有妖，于是我们需要作进一步的深入调查，于是发现了这个



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibXrER00DoDTrUCMntgGQLkWzxAiauQwH0DxicTrsHEetrFVhKXWOAibwtg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



注释中已经详细交代了这个方法的内容，那就是attribute, flow, desugar 和 generation。综合以上结果，我们将整个前端编译的流程总结为以下内容：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibfTb2IZBABxSsN3ic6UGLbKXDe8d1aM1JaicyqF07s02D9952B2zAeKXg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



那对应的每个方法里面都干了啥，又能如何和我们前面的一些分析呼应起来呢？让我们逐个方法去讲解：

1. parseFiles —— 解析java 源码，分词并进行词法，语法验证及分析，说白了，就是让javac 能读明白java 源码并确认源文件符合java基本语法的关键一步。那它是怎么明白java源码里面都写了啥，并且如何验证基本语法的呢？于是逻辑上，我们就要有一个正确的标准，让它往上去靠，靠上了就OK，靠不上就有问题，那么这个正确的标准在哪呢？答案如下:

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibnHK0gBpb85icSMYqPyHoTqTo87jKt9cHeQMxyAjQvIc6wdZ9sHxoCLw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



这下子就豁然开朗了，读进去的文件，从第一个字符开始，依次按照以上关键词进行查找，即先做分词，分词后的结果，能符合这里面的关键字，那就是java的关键字类型，就能够进行后续的标准化流程，如果没有的话，那要么就是程序员自己定义的命名，要么就是手滑写错了，后续会对这类情况进行集中处理，它在抽象语法树（马上提到）中的名称就按照用户自定义的名称来组装了。对于分词的结果的处理，核心的方法就是如下所示这个红圈里面的方法，这个方法会按照java 的语法规则，按照以类为维度对类下面所有方法，属性，以及方法下面的访问限定，类型修饰，参数，方法体，返回等元素构建一个叫做抽象语法树的东西，实际上就是这个JCTree。



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibfGRCrtsaszKFfT6FIopBbunA2hcIoicyGg1jKFFztrh7G554kl3Pibug/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



就以我们那个充满党性的代码来举例子，按照这个标准分完词之后，这个树大概是个什么样子呢？

<img src="https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4Pxibib3HthKvlcEfia24eiaq99dCS41a2PheFXvpA3y6tW343jTogAzDXGASA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />



如果实在不想看图，大概意思就是javac 在文件解析语法这一步，就会把java 代码变成一个以类为单位组建的树状层级关系的数据结构列表。

 至此第一步结束。

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibWoYCcibW3VoJyyyb2gmvc0AXjOnNKicdNad68UlfibHCzbvEKqRwnukOw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



2. enterTrees —— 填充符号表（都这么说，但是这句话确实让人摸不着头脑）



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQOZQfNUpt6ralchAPtibdkqeF8ukN73V2nbXZ5uXOX6iaOFUtu4mMaBE8J13vcghEQs0vB4CxZowUbg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



那我们就来简单讲讲。首先，这些年来，让我们不理解这句话的，就是这个罪魁祸首——符号表。啥是符号表？

广义上来讲，符号表就是一个键值对的集合，可以通过键，找到对应的值。很多时候都是基于哈希表实现的。 在创建javac 的符号表的过程中，它基于之前创建的抽象语法树，将包含了包名，作用域，到其包含的所有类，方法，变量，以及其对应的类型，所属类等信息包装到Symbol对象中，并将其赋值给抽象语法树的sym 变量中，这些信息会在之后的步骤中，用于对该符号，针对不同类型处理。此外，还会在原来的树中填充默认构造函数等信息。

3. processAnnotations——注解处理，这一步应该比较好理解，从JAVA 6开始，JAVA生态里面增加了可以在前端编译阶段就可以生效的注解式的插件处理，说一大堆，直接举个最简单的例子，大名鼎鼎的@Lombok，就是代表之一

```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Setter {
    AccessLevel value() default AccessLevel.PUBLIC;

    Setter.AnyAnnotation[] onMethod() default {};

    Setter.AnyAnnotation[] onParam() default {};

    /** @deprecated */
    @Deprecated
    @Retention(RetentionPolicy.SOURCE)
    @Target({})
    public @interface AnyAnnotation {
    }
}
```



此处的这个@Retention(RetentionPolicy.SOURCE) 就意味着它会在前端编译阶段会去根据注解的定义，对抽象语法树进行修改，比如这个，就是在类中增加setter 方法。一旦这一步改变了语法树，则javac 会将前面两步，即parseFile 和 enterTrees 重新来过一遍，再进入注解处理，直到没有语法树的改变。归纳起来如下图



![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQOZQfNUpt6ralchAPtibdkqe141POEPJCTYCxxm2TwnFzoAzXGkXTJymKvnC1IbUFibNCXxzmzIwxPg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

\4. Attribute —— 标注检查，该步骤为语义分析的第一步。通过上面的几步，我们已经生成了一棵囫囵个的语法树，结构是有了，合情合理合法，但是内容呢？是不是符合核心价值观？是不是符合规范呢？那么就需要以下两步去分析了。比如说，企图用int 加上 一个 bool 得到一个字符串，那是万万不能的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQOZQfNUpt6ralchAPtibdkqe7sj8oqqLzWwDiaIVibxfqtn19icoHDic3OajOKeTR1JZCjmT0Wpjm1Appw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这个方法里会进行如下的操作和验证

- 类型规则是否符合规范
- 赋值之前是否被声明
- 查询继承关系中是否存在循环引用和环状关系
- 枚举类是否被继承
- 非抽象类中是否存在抽象方法，或者是由于本身集成其它接口或者抽象类，但存在没有实现的方法
- 检查所有继承的类都可被编译
- 检查泛型类没有继承Throwable
- 检查继承或实现的方法都符合java的语法
- 检查并限制实现AutoCloseable 接口的资源类抛出InterruptedException
- 检查内部类的声明是否存在static
- 检查默认构造函数和注解调用关系中是否存在环
- 检查类中是否正确使用serialVersionUID
- 检查类型注解
- 检查标注@Deprecated的注解
- 检查Functional Interface
- 此外，在这一步，还会进行诸如常量折叠的优化，优化后，以下语句等价：

```java
int a = 5;
int a = 1 + 4;
```

编译器会自动把 a 变成 a = 5。

经过了第一轮审查，看看里面有没有不守规矩乱蹦蹬的，看完没问题，接下来是第二轮检查，看看内容是否正能量

 5. Flow —— 数据控制流分析，这一步根据之前那棵树中的各种依赖关系，检查程序的上下文（就是前前后后），从开始到结尾的赋值，比如局部变量是否在使用前赋值，返回等是否符合规范，所有的受检查异常是否被正确处理等。

6. Desugar —— 解语法糖。又是一个翻译的天花板，类似的还有鲁棒性和缺省，个人觉得荔枝味的棒棒糖最好吃。所谓解语法糖，就是把JDK工具包里面让程序员写着更省事，更人性的东西，给还原回它原本的样子，这些糖的种类在此列举几个常见的：泛型擦除、自动拆箱和装箱、for-each增强for循环、方法变长参数，对了，还有我们的Lambda。

7. Generate —— 生成字节码，终于到你了，差点都忘了。总的来说，这一步，就是依据经过层层审查后的那棵树还有那个表，以及里面的所有内容，关系，生成字节码的过程。好了，这一步，以我们最开始的推测，需要有一个到字节码的映射，那我们找找它在哪。

```java

public interface ByteCodes {
    /** 
     * Byte code instruction codes.
     */
    int illegal         = -1,
        nop             = 0,
        aconst_null     = 1,
        iconst_m1       = 2,
        iconst_0        = 3,
        iconst_1        = 4,
        iconst_2        = 5,
        iconst_3        = 6,
        iconst_4        = 7,
        iconst_5        = 8,
        lconst_0        = 9,
        lconst_1        = 10,
        fconst_0        = 11,
        fconst_1        = 12,
        fconst_2        = 13,
        dconst_0        = 14,
        dconst_1        = 15,
        bipush          = 16
        // 还有很多。。。
}
```

以及

```java

public class ClassFile {

    public final static int JAVA_MAGIC = 0xCAFEBABE;

    // see Target
    public final static int CONSTANT_Utf8 = 1;
    public final static int CONSTANT_Unicode = 2;
    public final static int CONSTANT_Integer = 3;
    public final static int CONSTANT_Float = 4;
    public final static int CONSTANT_Long = 5;
    public final static int CONSTANT_Double = 6;
    public final static int CONSTANT_Class = 7;
    public final static int CONSTANT_String = 8;
    public final static int CONSTANT_Fieldref = 9;
    public final static int CONSTANT_Methodref = 10;
    public final static int CONSTANT_InterfaceMethodref = 11;
    public final static int CONSTANT_NameandType = 12;
    public final static int CONSTANT_MethodHandle = 15;
    public final static int CONSTANT_MethodType = 16;
    public final static int CONSTANT_InvokeDynamic = 18;

    public final static int REF_getField = 1;
    public final static int REF_getStatic = 2;
    public final static int REF_putField = 3;
    public final static int REF_putStatic = 4;
    public final static int REF_invokeVirtual = 5;
    public final static int REF_invokeStatic = 6;
    public final static int REF_invokeSpecial = 7;
    public final static int REF_newInvokeSpecial = 8;
    public final static int REF_invokeInterface = 9;

    public final static int MAX_PARAMETERS = 0xff;
    public final static int MAX_DIMENSIONS = 0xff;
    public final static int MAX_CODE = 0xffff;
    public final static int MAX_LOCALS = 0xffff;
    public final static int MAX_STACK = 0xffff;
}
```

一个是字节码对应的指令集，一个是字节码常量池。这些东西，我们下一次会详细讲解。

关于字节码生成的过程我们讲完了，让我们回顾一下之前我们的那些推测

- Java 语言 和字节码语言的语法是有标准的，并且语义集合是有限的**(客观事实)**
- Javac 能够精确解析java 代码 **(parseFile)**
- Java 代码本身必须符合Java的语法和使用规范**（各个环节的校验）**
- Javac 需要有一个完整的从java代码到字节码的映射 **(generate)**
- Javac 能够精确输出目标结果——字节码 **(generate)**
- Javac 能够完整理解java 代码的整体逻辑表达 **(基于抽象语法树和符号表的语义分析和数据流验证流程)**

闭环了！

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQOZQfNUpt6ralchAPtibdkqeCEYaSgGHLAicDPK0oBt7jfyz4LZ89T1G7KbhFtwrNyM2t4UKDFgPgeg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

最后，它终于翻译完了之后，翻译稿放哪了？

这个让我们随随便便来利用javac 编译一下文章开始的那篇充满能量的代码，走你——

![图片](https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibHoW3EziaLxQNUeLBd2ABzlenADmPyUbG4aKBUehLlbs6IztsyyrVs9w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们使用-verbose参数来展示更多的信息，里面能看到执行前端编译的时候的一些步骤，最终生成了一个NewEra.class文件。那么这个新时代的类文件，就是我们的最终翻译稿，这篇气势恢弘的翻译稿，随后就会被拿去给JVM进行深入学习领会提升了。

最后，我们来一观翻译稿里面究竟都有什么？

<img src="https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4PxibibaFbAqIwXRtLs7L1ORKVoxibJYwjBicksK9icLnLTiclPNhWCjCiabWm77w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:33%;" />

算了，太长了，还是下一篇见吧。

<img src="https://mmbiz.qpic.cn/mmbiz_png/fAFyC36wKQMbtBExxdncGDceujVg4Pxibp99gOpcpoa5J7CX6v486TMVia1U5odjaVO1wSdfKuAGBhPZ8LQ6Utqw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:33%;" />



各位施主，求加公众号关注一下小弟。阿弥陀佛，佛法无边。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/fAFyC36wKQMbtBExxdncGDceujVg4PxibVlnTfWVdM9ibickaDoZtibApFm2eoRQiac4gmkDAzAtUzmtib9CySJvnFKg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

