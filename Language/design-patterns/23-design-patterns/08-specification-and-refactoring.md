# 理论一：什么情况下要重构？到底重构什么？又该如何重构？

重构代码对一个工程师能力的要求，要比单纯写代码高得多。重构需要你能洞察出 代码存在的坏味道或者设计上的不足，并且能合理、熟练地利用设计思想、原则、模式、编 程规范等理论知识解决这些问题。很多工程师对为什么要重构、到底重构什么、什么时候重构、又该如何重构等相 关问题理解不深，对重构没有系统性、全局性的认识，面对一堆烂代码，没有重构技巧的指 导，只能想到哪改到哪，并不能全面地改善代码质量。

为了让你对重构有个清晰的认识，对于这部分知识的讲解，主要包含以下几个方面：

对重构概括性的介绍，包括重构的目的（why）、对象（what）、时机（when）、方 法（how）； 

保证重构不出错的手段，这里我会重点讲解单元测试和代码的可测试性； 

不同规模的重构，重点讲解大规模高层次重构（比如系统、模块、代码结构、类与类之间的交互等的重构）和小规模低层次重构（类、函数、变量等的重构）。

## 重构的目的：为什么要重构（why）？

重构是一种对软件内部结构的改善，目的是在不改变软件的可见行为的情况下，使其更易理解，修改成本更低。有一个值得强调的点：“重构不改变外部的可见行为”。我们可以把重构理解为，在保持功能不变的前提下， 利用设计思想、原则、模式、编程规范等理论来优化代码，修改设计上的不足，提高代码质 量。

首先，重构是时刻保证代码质量的一个极其有效的手段，不至于让代码腐化到无可救药的地 步。项目在演进，代码不停地在堆砌。如果没有人为代码的质量负责任，代码总是会往越来 越混乱的方向演进。当混乱到一定程度之后，量变引起质变，项目的维护成本已经高过重新 开发一套新代码的成本，想要再去重构，已经没有人能做到了。其次，优秀的代码或架构不是一开始就能完全设计好的，就像优秀的公司和产品也都是迭代 出来的。我们无法 100% 遇见未来的需求，也没有足够的精力、时间、资源为遥远的未来 买单，所以，随着系统的演进，重构代码也是不可避免的。最后，重构是避免过度设计的有效手段。在我们维护代码的过程中，真正遇到问题的时候， 再对代码进行重构，能有效避免前期投入太多时间做过度的设计，做到有的放矢。

重构实际上是对我们学习的经典设计思想、设计原则、设 计模式、编程规范的一种应用。重构实际上就是将这些理论知识，应用到实践的一个很好的 场景，能够锻炼我们熟练使用这些理论知识的能力。除此之外，平时堆砌业务逻辑，你可能 总觉得没啥成长，而将一个比较烂的代码重构成一个比较好的代码，会让你很有成就感。

除此之外，重构能力也是衡量一个工程师代码能力的有效手段。所谓“**初级工程师在维护代码，高级工程师在设计代码，资深工程师在重构代码**”，这句话的意思是说，初级工程师在 已有代码框架下修改 bug、修改添加功能代码；高级工程师从零开始设计代码结构、搭建 代码框架；而资深工程师为代码质量负责，需要发觉代码存在的问题，重构代码，时刻保证 代码质量处于一个可控的状态（当然这里的初级、高级、资深只是一个相对概念，并不是一 个确定的职级）。

## 重构的对象：到底重构什么（what）？

根据重构的规模，我们可以笼统地分为大规模高层次重构（以下简称为“大型重构”）和小 规模低层次的重构（以下简称为“小型重构”）。 

大型重构指的是对顶层代码设计的重构，包括：系统、模块、代码结构、类与类之间的关系等的重构，重构的手段有：分层、模块化、解耦、抽象可复用组件等等。这类重构的工具就 是我们学习过的那些设计思想、原则和模式。这类重构涉及的代码改动会比较多，影响面会 比较大，所以难度也较大，耗时会比较长，引入 bug 的风险也会相对比较大。 

小型重构指的是对代码细节的重构，主要是针对类、函数、变量等代码级别的重构，比如规 范命名、规范注释、消除超大类或函数、提取重复代码等等。小型重构更多的是利用我们能 后面要讲到的编码规范。这类重构要修改的地方比较集中，比较简单，可操作性较强，耗时会比较短，引入 bug 的风险相对来说也会比较小。你只需要熟练掌握各种编码规范，就可 以做到得心应手。

## 重构的时机：什么时候重构（when）？

什么时候重构？是代码烂到一定程 度之后才去重构吗？当然不是。因为当代码真的烂到出现“开发效率低，招了很多人，天天 加班，出活却不多，线上 bug 频发，领导发飙，中层束手无策，工程师抱怨不断，查找 bug 困难”的时候，基本上重构也无法解决问题了。

我个人比较反对，平时不注重代码质量，堆砌烂代码，实在维护不了了就大刀阔斧地重构、 甚至重写的行为。有时候项目代码太多了，重构很难做得彻底，最后又搞出来一个“四不像 的怪物”，这就更麻烦了！所以，寄希望于在代码烂到一定程度之后，集中重构解决所有问 题是不现实的，我们必须探索一条可持续、可演进的方式。

所以，我特别提倡的重构策略是**持续重构**。这也是我在工作中特别喜欢干的事情。平时没有 事情的时候，你可以看看项目中有哪些写得不够好的、可以优化的代码，主动去重构一下。 或者，在修改、添加某个功能代码的时候，你也可以顺手把不符合编码规范、不好的设计重 构一下。总之，就像把单元测试、Code Review 作为开发的一部分，我们如果能把持续重 构也作为开发的一部分，成为一种开发习惯，对项目、对自己都会很有好处。

尽管我们说重构能力很重要，但持续重构意识更重要。我们要正确地看待代码质量和重构这 件事情。技术在更新、需求在变化、人员在流动，代码质量总会在下降，代码总会存在不完 美，重构就会持续在进行。时刻具有持续重构意识，才能避免开发初期就过度设计，避免代 码维护的过程中质量的下降。而那些看到别人代码有点瑕疵就一顿乱骂，或者花尽心思去构 思一个完美设计的人，往往都是因为没有树立正确的代码质量观，没有持续重构意识。

## 重构的方法：又该如何重构（how）？

按照重构的规模，重构可以笼统地分为大型重构和小型重构。对于这两种不同规模的重构，我们要区别对待。

对于大型重构来说，因为涉及的模块、代码会比较多，如果项目代码质量又比较差，耦合比 较严重，往往会牵一发而动全身，本来觉得一天就能完成的重构，你会发现越改越多、越改 越乱，没一两个礼拜都搞不定。而新的业务开发又与重构相冲突，最后只能半途而废， revert 掉所有的改动，很失落地又去堆砌烂代码了。 

在进行大型重构的时候，我们要提前做好完善的重构计划，有条不紊地分阶段来进行。每个阶段完成一小部分代码的重构，然后提交、测试、运行，发现没有问题之后，再继续进行下 一阶段的重构，保证代码仓库中的代码一直处于可运行、逻辑正确的状态。每个阶段，我们 都要控制好重构影响到的代码范围，考虑好如何兼容老的代码逻辑，必要的时候还需要写一 些兼容过渡代码。只有这样，我们才能让每一阶段的重构都不至于耗时太长（最好一天就能完成），不至于与新的功能开发相冲突。

大规模高层次的重构一定是有组织、有计划，并且非常谨慎的，需要有经验、熟悉业务的资 深同事来主导。而小规模低层次的重构，因为影响范围小，改动耗时短，所以，只要你愿意 并且有时间，随时都可以去做。实际上，除了人工去发现低层次的质量问题，我们还可以借 助很多成熟的静态代码分析工具（比如 CheckStyle、FindBugs、PMD），来自动发现代 码中的问题，然后针对性地进行重构优化。

对于重构这件事情，资深的工程师、项目 leader 要负起责任来，没事就重构一下代码，时 刻保证代码质量处在一个良好的状态。否则，一旦出现“破窗效应”，一个人往里堆了一些 烂代码，之后就会有更多的人往里堆更烂的代码。毕竟往项目里堆砌烂代码的成本太低了。 不过，保持代码质量最好的方法还是打造一种好的技术氛围，以此来驱动大家主动去关注代 码质量，持续重构代码。

## 重点回顾

对重构有个正确的、全局性的认知，建 立持续重构意识。因为很多技术问题本身就 不是单纯靠技术来解决的，更重要的是要有这种认知和意识。

1. 重构的目的：为什么重构（why）？对于项目来言，重构可以保持代码质量持续处于一个可控状态，不至于腐化到无可救药的地 步。对于个人而言，重构非常锻炼一个人的代码能力，并且是一件非常有成就感的事情。它 是我们学习的经典设计思想、原则、模式、编程规范等理论知识的练兵场。
2. 重构的对象：重构什么（what）？ 按照重构的规模，我们可以将重构大致分为大规模高层次的重构和小规模低层次的重构。大 规模高层次重构包括对代码分层、模块化、解耦、梳理类之间的交互关系、抽象复用组件等 等。这部分工作利用的更多的是比较抽象、比较顶层的设计思想、原则、模式。小规模低层 次的重构包括规范命名、注释、修正函数参数过多、消除超大类、提取重复代码等等编程细 节问题，主要是针对类、函数级别的重构。小规模低层次的重构更多的是利用编码规范这一 理论知识。 
3. 重构的时机：什么时候重构（when）？ 我反复强调，我们一定要建立持续重构意识，把重构作为开发必不可少的部分，融入到日常 开发中，而不是等到代码出现很大问题的时候，再大刀阔斧地重构。
4. 重构的方法：如何重构（how）？ 大规模高层次的重构难度比较大，需要组织、有计划地进行，分阶段地小步快跑，时刻让代 码处于一个可运行的状态。而小规模低层次的重构，因为影响范围小，改动耗时短，所以， 只要你愿意并且有时间，随时随地都可以去做。

# 理论二：为了保证重构不出错，有哪些非常能落地的技术手段？

据我了解，很多程序员对重构这种做法还是非常认同的，面对项目中的烂代码，也想重构一 下，但又担心重构之后出问题，出力不讨好。确实，如果你要重构的代码是别的同事开发 的，你不是特别熟悉，在没有任何保障的情况下，重构引入 bug 的风险还是很大的。 那如何保证重构不出错呢？你需要熟练掌握各种设计原则、思想、模式，还需要对所重构的 业务和代码有足够的了解。除了这些个人能力因素之外，最可落地执行、最有效的保证重构 不出错的手段应该就是单元测试（Unit Testing）了。当重构完成之后，如果新的代码仍然能通过单元测试，那就说明代码原有逻辑的正确性未被破坏，原有的外部可见行为未变，符 合上一节课中我们对重构的定义。

单元测试。今天的内容主要包含这样几个内容：

什么是单元测试？ 

为什么要写单元测试？ 

如何编写单元测试？ 

如何在团队中推行单元测试？

## 什么是单元测试？

单元测试由研发工程师自己来编写，用来测试自己写的代码的正确性。我们常常将它跟集成 测试放到一块来对比。单元测试相对于集成测试（Integration Testing）来说，测试的粒 度更小一些。集成测试的测试对象是整个系统或者某个功能模块，比如测试用户注册、登录 功能是否正常，是一种端到端（end to end）的测试。而单元测试的测试对象是类或者函 数，用来测试一个类和函数是否都按照预期的逻辑执行。这是代码层级的测试。

```java
public class Text {
private String content;
public Text(String content) {
this.content = content;
}
/**
* 将字符串转化成数字，忽略字符串中的首尾空格；
* 如果字符串中包含除首尾空格之外的非数字字符，则返回 null。
*/
public Integer toNumber() {
if (content == null || content.isEmpty()) {
return null;
}
//... 省略代码实现...
return null;
}
}
```

要测试 Text 类中的 toNumber() 函数的正确性，应该如何编写单元测试呢？ 实际上，写单元测试本身不需要什么高深技术。它更多的是考验程序员思维的缜密程度，看能否设计出覆盖各种正常及异常情况的测试用例，来保证代码在任何预期或非预期的情况下 都能正确运行。 为了保证测试的全面性，针对 toNumber() 函数，我们需要设计下面这样几个测试用例。

如果字符串只包含数字：“123”，toNumber() 函数输出对应的整数：123。 

如果字符串是空或者 null，toNumber() 函数返回：null。 

如果字符串包含首尾空格：“ 123”，“123 ”，“ 123 ”，toNumber() 返回对应的 整数：123。 

如果字符串包含多个首尾空格：“ 123 ”，toNumber() 返回对应的整数：123； 

如果字符串包含非数字字符：“123a4”，“123 4”，toNumber() 返回 null；

设计好测试用例之后，剩下的就是将其翻译成代码了。

```java
public class Assert {
public static void assertEquals(Integer expectedValue, Integer actualValue) {
if (actualValue != expectedValue) {
String message = String.format(
"Test failed, expected: %d, actual: %d.", expectedValue, actualVa
System.out.println(message);
} else {
System.out.println("Test succeeded.");
}
}
public static boolean assertNull(Integer actualValue) {
boolean isNull = actualValue == null;
if (isNull) {
System.out.println("Test succeeded.")
} else {
System.out.println("Test failed, the value is not null:" + actualValue);
}
return isNull;
}
}
public class TestCaseRunner {
public static void main(String[] args) {
System.out.println("Run testToNumber()");
new TextTest().testToNumber();
System.out.println("Run testToNumber_nullorEmpty()");
new TextTest().testToNumber_nullorEmpty();
System.out.println("Run testToNumber_containsLeadingAndTrailingSpaces()");
new TextTest().testToNumber_containsLeadingAndTrailingSpaces();
System.out.println("Run testToNumber_containsMultiLeadingAndTrailingSpaces
new TextTest().testToNumber_containsMultiLeadingAndTrailingSpaces();
System.out.println("Run testToNumber_containsInvalidCharaters()");
new TextTest().testToNumber_containsInvalidCharaters();
}
}
public class TextTest {
public void testToNumber() {
Text text = new Text("123");
Assert.assertEquals(123, text.toNumber());
}
public void testToNumber_nullorEmpty() {
Text text1 = new Text(null);
Assert.assertNull(text1.toNumber());
Text text2 = new Text("");
Assert.assertNull(text2.toNumber());
}
public void testToNumber_containsLeadingAndTrailingSpaces() {
Text text1 = new Text(" 123");
Assert.assertEquals(123, text1.toNumber());
Text text2 = new Text("123 ");
Assert.assertEquals(123, text2.toNumber());
Text text3 = new Text(" 123 ");
Assert.assertEquals(123, text3.toNumber());
}
public void testToNumber_containsMultiLeadingAndTrailingSpaces() {
    Text text1 = new Text(" 123");
 Assert.assertEquals(123, text1.toNumber());
 Text text2 = new Text("123 ");
 Assert.assertEquals(123, text2.toNumber());
 Text text3 = new Text(" 123 ");
 Assert.assertEquals(123, text3.toNumber());
 }
 public void testToNumber_containsInvalidCharaters() {
 Text text1 = new Text("123a4");
 Assert.assertNull(text1.toNumber());
 Text text2 = new Text("123 4");
 Assert.assertNull(text2.toNumber());
 }
}
```

## 为什么要写单元测试？ 

单元测试除了能有效地为重构保驾护航之外，也是保证代码质量最有效的两个手段之一（另 一个是 Code Review）。我在 Google 工作的时候，写了大量的单元测试代码，结合我的 这些开发经验，我总结了以下几点单元测试的好处。尽管有些听起来有点“务虚”，但如果 你认真写过一些单元测试的话，应该会很有共鸣。

1. **单元测试能有效地帮你发现代码中的 bug** 能否写出 bug free 的代码，是判断工程师编码能力的重要标准之一，也是很多大厂面试考 察的重点，特别是像 FLAG 这样的外企。即便像我这样代码写了十几年，逻辑还算缜密、 清晰的人，通过单元测试也常常会发现代码中的很多考虑不全面的地方。
2. **写单元测试能帮你发现代码设计上的问题** 代码的可测试性是评判代码质量的一个重要标准。对于一段代码，如果很难为其编写单元测试，或者单元测试写起来很吃力，需要依靠单元测试框架里很高级的特性才 能完成，那往往就意味着代码设计得不够合理，比如，没有使用依赖注入、大量使用静态函 数、全局变量、代码高度耦合等。
3. **单元测试是对集成测试的有力补充** 程序运行的 bug 往往出现在一些边界条件、异常情况下，比如，除数未判空、网络超时。 而大部分异常情况都比较难在测试环境中模拟。而单元测试可以利用下一节课中讲到的 mock 的方式，控制 mock 的对象返回我们需要模拟的异常，来测试代码在这些异常情况 的表现。 除此之外，对于一些复杂系统来说，集成测试也无法覆盖得很全面。复杂系统往往有很多模 块。每个模块都有各种输入、输出、异常情况，组合起来，整个系统就有无数测试场景需要 模拟，无数的测试用例需要设计，再强大的测试团队也无法穷举完备。 尽管单元测试无法完全替代集成测试，但如果我们能保证每个类、每个函数都能按照我们的 预期来执行，底层 bug 少了，那组装起来的整个系统，出问题的概率也就相应减少了。
4. **写单元测试的过程本身就是代码重构的过程** 我们提到，要把持续重构作为开发的一部分来执行，那写单元测试实际上就是 落地执行持续重构的一个有效途径。设计和实现代码的时候，我们很难把所有的问题都想清 楚。而编写单元测试就相当于对代码的一次自我 Code Review，在这个过程中，我们可以 发现一些设计上的问题（比如代码设计的不可测试）以及代码编写方面的问题（比如一些边 界条件处理不当）等，然后针对性的进行重构。
5. **阅读单元测试能帮助你快速熟悉代码** 阅读代码最有效的手段，就是先了解它的业务背景和设计思路，然后再去看代码，这样代码 读起来就会轻松很多。但据我了解，程序员都不怎么喜欢写文档和注释，而大部分程序员写 的代码又很难做到“不言自明”。在没有文档和注释的情况下，单元测试就起了替代性作 用。单元测试用例实际上就是用户用例，反映了代码的功能和如何使用。借助单元测试，我 们不需要深入的阅读代码，便能知道代码实现了什么功能，有哪些特殊情况需要考虑，有哪 些边界条件需要处理。
6. **单元测试是 TDD 可落地执行的改进方案** 测试驱动开发（Test-Driven Development，简称 TDD）是一个经常被提及但很少被执行 的开发模式。它的核心指导思想就是测试用例先于代码编写。不过，要让程序员能彻底地接 受和习惯这种开发模式还是挺难的，毕竟很多程序员连单元测试都懒得写，更何况在编写代码之前先写好测试用例了。 我个人觉得，单元测试正好是对 TDD 的一种改进方案，先写代码，紧接着写单元测试，最 后根据单元测试反馈出来问题，再回过头去重构代码。这个开发流程更加容易被接受，更加 容易落地执行，而且又兼顾了 TDD 的优点。

## 如何编写单元测试？

写单元测试就是针对代码设计覆盖各种输入、异常、边 界条件的测试用例，并将这些测试用例翻译成代码的过程。 

在把测试用例翻译成代码的时候，我们可以利用单元测试框架，来简化测试代码的编写。比 如，Java 中比较出名的单元测试框架有 Junit、TestNG、Spring Test 等。这些框架提供 了通用的执行流程（比如执行测试用例的 TestCaseRunner）和工具类库（比如各种 Assert 判断函数）等。借助它们，我们在编写测试代码的时候，只需要关注测试用例本身的编写即可。针对 toNumber() 函数的测试用例，我们利用 Junit 单元测试框架重新实现一下，具体代码如下所示。

```java
import org.junit.Assert;
import org.junit.Test;
public class TextTest {
@Test
public void testToNumber() {
Text text = new Text("123");
Assert.assertEquals(new Integer(123), text.toNumber());
}
@Test
public void testToNumber_nullorEmpty() {
Text text1 = new Text(null);
Assert.assertNull(text1.toNumber());
Text text2 = new Text("");
Assert.assertNull(text2.toNumber());
}
@Test
public void testToNumber_containsLeadingAndTrailingSpaces() {
Text text1 = new Text(" 123");
Assert.assertEquals(new Integer(123), text1.toNumber());
Text text2 = new Text("123 ");
Assert.assertEquals(new Integer(123), text2.toNumber());
Text text3 = new Text(" 123 ");
Assert.assertEquals(new Integer(123), text3.toNumber());
}
@Test
public void testToNumber_containsMultiLeadingAndTrailingSpaces() {
Text text1 = new Text(" 123");
Assert.assertEquals(new Integer(123), text1.toNumber());
Text text2 = new Text("123 ");
Assert.assertEquals(new Integer(123), text2.toNumber());
Text text3 = new Text(" 123 ");
Assert.assertEquals(new Integer(123), text3.toNumber());
}
@Test
public void testToNumber_containsInvalidCharaters() {
Text text1 = new Text("123a4");
Assert.assertNull(text1.toNumber());
Text text2 = new Text("123 4");
Assert.assertNull(text2.toNumber());
}
}
```

关于如何编写单元测试，我更希望传达给你一些我的经验总结。具体包括以下几点。

1. **写单元测试真的是件很耗时的事情吗？**尽管单元测试的代码量可能是被测代码本身的 1～2 倍，写的过程很繁琐，但并不是很耗 时。毕竟我们不需要考虑太多代码设计上的问题，测试代码实现起来也比较简单。不同测试 用例之间的代码差别可能并不是很大，简单 copy-paste 改改就行。
2. **对单元测试的代码质量有什么要求吗？** 单元测试毕竟不会在产线上运行，而且每个类的测试代码也比较独立，基本不互相依赖。所 以，相对于被测代码，我们对单元测试代码的质量可以放低一些要求。命名稍微有些不规 范，代码稍微有些重复，也都是没有问题的。 
3. **单元测试只要覆盖率高就够了吗？** 单元测试覆盖率是比较容易量化的指标，常常作为单元测试写得好坏的评判标准。有很多现 成的工具专门用来做覆盖率统计，比如，JaCoCo、Cobertura、Emma、Clover。覆盖率 的计算方式有很多种，比较简单的是语句覆盖，稍微高级点的有：条件覆盖、判定覆盖、路 径覆盖。 不管覆盖率的计算方式如何高级，将覆盖率作为衡量单元测试质量的唯一标准是不合理的。 实际上，更重要的是要看测试用例是否覆盖了所有可能的情况，特别是一些 corner case。 我来举个简单的例子解释一下。
4. **写单元测试需要了解代码的实现逻辑吗？** 单元测试不要依赖被测试函数的具体实现逻辑，它只关心被测函数实现了什么功能。我们切 不可为了追求覆盖率，逐行阅读代码，然后针对实现逻辑编写单元测试。否则，一旦对代码 进行重构，在代码的外部行为不变的情况下，对代码的实现逻辑进行了修改，那原本的单元 测试都会运行失败，也就起不到为重构保驾护航的作用了，也违背了我们写单元测试的初衷。
5. **如何选择单元测试框架？** 写单元测试本身不需要太复杂的技术，大部分单元测试框架都能满足。在公司内部，起码团 队内部需要统一单元测试框架。如果自己写的代码用已经选定的单元测试框架无法测试，那 多半是代码写得不够好，代码的可测试性不够好。这个时候，我们要重构自己的代码，让其 更容易测试，而不是去找另一个更加高级的单元测试框架。

## 单元测试为何难落地执行？

还有人觉得，有了测试团队，写单元测试就是浪费时间，没有必要。程序员这一 行业本该是智力密集型的，但现在很多公司把它搞成劳动密集型的，包括一些大厂，在开发 过程中，既没有单元测试，也没有 Code Review 流程。即便有，做的也是差强人意。写好 代码直接提交，然后丢给黑盒测试狠命去测，测出问题就反馈给开发团队再修改，测不出的 问题就留在线上出了问题再修复。

在这样的开发模式下，团队往往觉得没有必要写单元测试，但如果我们把单元测试写好、做 好 Code Review，重视起代码质量，其实可以很大程度上减少黑盒测试的投入。我在 Google 的时候，很多项目几乎没有测试团队参与，代码的正确性完全靠开发团队来保障， 线上 bug 反倒非常少。

## 重点回顾 

1. 什么是单元测试？ 单元测试是代码层面的测试，由研发自己来编写，用于测试“自己”编写的代码的逻辑的正 确性。单元测试顾名思义是测试一个“单元”，有别于集成测试，这个“单元”一般是类或 函数，而不是模块或者系统。 
2. 为什么要写单元测试？ 写单元测试的过程本身就是代码 Code Review 和重构的过程，能有效地发现代码中的 bug 和代码设计上的问题。除此之外，单元测试还是对集成测试的有力补充，还能帮助我们快速 熟悉代码，是 TDD 可落地执行的改进方案。 
3. 如何编写单元测试？写单元测试就是针对代码设计各种测试用例，以覆盖各种输入、异常、边界情况，并将其翻 译成代码。我们可以利用一些测试框架来简化单元测试的编写。除此之外，对于单元测试， 我们需要建立以下正确的认知：编写单元测试尽管繁琐，但并不是太耗时； 我们可以稍微放低对单元测试代码质量的要求； 覆盖率作为衡量单元测试质量的唯一标准是不合理的； 单元测试不要依赖被测代码的具体实现逻辑； 单元测试框架无法测试，多半是因为代码的可测试性不好。
4. 单元测试为何难落地执行？ 一方面，写单元测试本身比较繁琐，技术挑战不大，很多程序员不愿意去写；另一方面，国内研发比较偏向“快、糙、猛”，容易因为开发进度紧，导致单元测试的执行虎头蛇尾。最 后，关键问题还是团队没有建立对单元测试正确的认识，觉得可有可无，单靠督促很难执行 得很好。

# 理论三：什么是代码的可测试性？如何写出可测试性好的代码？

实际上，写单元测试并不难，也不需要太多技巧，相反，写出可测试的代码反倒是件非常有 挑战的事情。所以，今天，我们就再来聊一聊代码的可测试性，主要包括这样几个问题：

什么是代码的可测试性？ 

如何写出可测试的代码？ 

有哪些常见的不好测试的代码？

## 编写可测试代码案例实战

关于代码可测试性的问题，我准备通过一个实战案例来讲解。具体的被测 试代码如下所示。 其中，Transaction 是经过我抽象简化之后的一个电商系统的交易类，用来记录每笔订单交 易的情况。Transaction 类中的 execute() 函数负责执行转账操作，将钱从买家的钱包转到 卖家的钱包中。真正的转账操作是通过调用 WalletRpcService RPC 服务来完成的。除此之外，代码中还涉及一个分布式锁 DistributedLock 单例类，用来避免 Transaction 并发 执行，导致用户的钱被重复转出。

```java
public class Transaction {
    private String id;
    private Long buyerId;
    private Long sellerId;
    private Long productId;
    private String orderId;
    private Long createTimestamp;
    private Double amount;
    private STATUS status;
    private String walletTransactionId;
    // ...get() methods...
    public Transaction(String preAssignedId, Long buyerId, Long sellerId, Long p
        if (preAssignedId != null && !preAssignedId.isEmpty()) {
        	this.id = preAssignedId;
        } else {
        	this.id = IdGenerator.generateTransactionId();
        }
        if (!this.id.startWith("t_")) {
        	this.id = "t_" + preAssignedId;
        }
        this.buyerId = buyerId;
        this.sellerId = sellerId;
        this.productId = productId;
        this.orderId = orderId;
        this.status = STATUS.TO_BE_EXECUTD;
        this.createTimestamp = System.currentTimestamp();
    }
    public boolean execute() throws InvalidTransactionException {
        if ((buyerId == null || (sellerId == null || amount < 0.0) {
        throw new InvalidTransactionException(...);
        }
        if (status == STATUS.EXECUTED) return true;
        boolean isLocked = false;
        try {
        isLocked = RedisDistributedLock.getSingletonIntance().lockTransction(id)
        if (!isLocked) {
        return false; // 锁定未成功，返回 false，job 兜底执行
        }
        if (status == STATUS.EXECUTED) return true; // double check
        long executionInvokedTimestamp = System.currentTimestamp();
        if (executionInvokedTimestamp - createdTimestap > 14days) {
        this.status = STATUS.EXPIRED;
        return false;
        }
        WalletRpcService walletRpcService = new WalletRpcService();
        String walletTransactionId = walletRpcService.moveMoney(id, buyerId, sell
        if (walletTransactionId != null) {
        this.walletTransactionId = walletTransactionId;
        this.status = STATUS.EXECUTED;
        return true;
        } else {
        this.status = STATUS.FAILED;
        return false;
        }
        } finally {
        if (isLocked) {
        RedisDistributedLock.getSingletonIntance().unlockTransction(id);
        }
        }
    }
}

```

在 Transaction 类中，主要逻辑集中在 execute() 函数中，所以它是我们测试的重点对 象。为了尽可能全面覆盖各种正常和异常情况，针对这个函数，我设计了下面 6 个测试用例。

1. 正常情况下，交易执行成功，回填用于对账（交易与钱包的交易流水）用的 walletTransactionId，交易状态设置为 EXECUTED，函数返回 true。
2. buyerId、sellerId 为 null、amount 小于 0，返回 InvalidTransactionException。
3. 交易已过期（createTimestamp 超过 14 天），交易状态设置为 EXPIRED，返回 false。
4. 交易已经执行了（status==EXECUTED），不再重复执行转钱逻辑，返回 true。
5. 钱包（WalletRpcService）转钱失败，交易状态设置为 FAILED，函数返回 false。
6. 交易正在执行着，不会被重复执行，函数直接返回 false。

测试用例设计完了。现在看起来似乎一切进展顺利。但是，事实是，当我们将测试用例落实 到具体的代码实现时，你就会发现有很多行不通的地方。对于上面的测试用例，第 2 个实 现起来非常简单，我就不做介绍了。

## 重点回顾 

1. 什么是代码的可测试性？ 粗略地讲，所谓代码的可测试性，就是针对代码编写单元测试的难易程度。对于一段代码， 如果很难为其编写单元测试，或者单元测试写起来很费劲，需要依靠单元测试框架中很高级 的特性，那往往就意味着代码设计得不够合理，代码的可测试性不好。 
2. 编写可测试性代码的最有效手段。依赖注入是编写可测试性代码的最有效手段。通过依赖注入，我们在编写单元测试的时候， 可以通过 mock 的方法解依赖外部服务，这也是我们在编写单元测试的过程中最有技术挑 战的地方。 
3. 常见的 Anti-Patterns。常见的测试不友好的代码有下面这 5 种：代码中包含未决行为逻辑； 滥用可变全局变量； 滥用静态方法； 使用复杂的继承关系； 高度耦合的代码。

