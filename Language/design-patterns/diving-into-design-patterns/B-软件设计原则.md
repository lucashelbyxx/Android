# **软件设计原则**

## **优秀的设计特征**

软件架构的设计过程，了解一下需要达成目标与需要尽量避免的陷阱。



## **代码复用**

开发软件产品，成本和时间都是最重要的两个维度。

代码复用是减少开发成本时的常用方式之一。意图非常明显：与其反复从头开发，不如在新对象中重用已有代码。想法很好，但实际要让已有代码在全新的上下文中工作，通常需要付出额外努力。组件间紧密的耦合、对具体类而非接口的依赖和硬编码的行为都会降低代码的灵活性，使得复用这些代码变得更加困难。

使用设计模式是增加软件组件灵活性并使其易于复用的方式之一。但是有时也会让组件变得更加复杂。



*设计模式创始人之一的埃里希.伽马，在谈到代码复用中设计模式的角色时说：*

*复用有三个层次。在最底层，可以复用类：类库、容器，还有一些类的 “团体” （如容器、迭代器）。*

*框架位于最高层，能帮助你精简自己的设计，可以用于明确解决问题所需的抽象概念，然后用类来表示这些概念并定义其关系。例如：JUnit 是一个框架，其中定义了 Test、TestCase 和 TestSuite 这几个类及其关系。*

*框架通常比单个类的颗粒度要大。你可以通过在某处构建子类与框架建立联系。这些子类信奉 “别给我们打电话，我们会给你打电话的。” 这句所谓的好莱坞原则。框架让你可以自定义行为，并会在需要完成工作时告诉你。这和 JUnit 一样，当它希望执行测试时就会告诉你，但其他的一切仅会在框架中发生。*

*还有一个中间层次。这也是我认识中的模式所处位置。设计模式比框架更小且更抽象。它们实际上是对一组类的关系及其互动方式的描述。当你从类转向模式，并最终到达框架过程中，复用程度会不断增加。*

*中间层次的有点在于模式提供的复用方式要比框架的风险小。创建框架是一项投入重大且风险很高的工作。模式则让你能独立于具体代码来复用设计思想和理念。*



## **扩展性**

**变化是程序员生命中唯一不变的事情。**

首先，在开始着手解决问题后才能更好地理解问题。通常在完成了第一版的程序后，你就做好了从头开始重写代码的准备，因为现在你已经能在很多方面更好地理解问题了，同时在专业水平上也有所提高，所以之前的代码现在看上去可能很糟糕。其次可能是在你掌控之外的某些事情发生了变化。第三个原因是需求的改变。如果有人要求你对程序进行修改，至少说明还有人关心它。

因此在设计程序架构时，所有有经验的开发者会尽量选择支持未来任何可能变更的方式。



# **设计原则**

什么是优秀的软件设计？如何对其进行评估？你需要遵循哪 些实践方式才能实现这样的方式？如何让你的架构灵活、稳 定且易于理解？ 这些都是很好的问题。但不幸的是，根据应用类型的不同， 这些问题的答案也不尽相同。不过对于你的项目来说，有几个通用的软件设计原则可能会对解决这些问题有所帮助。绝大部分设计模式都是基于这些原则的。



## **封装变化的内容**

找到程序中变化的内容并将其与不变的内容区分开。

该原则的主要目的是将变更造成的影响最小化。将程序变化部分放入独立的模块中，保护其他代码不受负面影响。你只需要花较少时间就能让程序恢复正常工作，或是实现并测试修改的内容。修改程序的时间越少，就有更多时间实现功能。



**方法层面的封装**

一个 getOrderTotal 方法，用于计算订单的总价（包括税金）。未来可能会修改与税金相关的代码。税率会根据国家、省甚至城市或其他原因而有所不同。需要经常修改 getOrderTotal 方法。看方法名称，它都在暗示不关心税金是如何计算出来的。



```
method getOrderTotal(order) is
    total = 0
    foreach item in order.lineItems
        total +=  item.price * item.quantity
        
    if (order.country == "US")
        total += total * 0.07 // 美国营业税
    else if (order.country == "EU"):
        total += total * 0.20 // 欧洲增值税
        
    return total
  
      
// 修改后
method getOrderTotal(order) is
    total = 0
    foreach item in order.lineItems
        total +=  item.price * item.quantity
    
    total += total * getTaxRate(order.country)    
            
    return total


method getTaxRate(country) is
    if (order.country == "US")
        return 0.07 // 美国营业税
    else if (order.country == "EU"):
        return 0.20 // 欧洲增值税
    else
        return 0
```





可以将计算税金的逻辑抽去到一个单独的方法中，并对原始方法隐藏该逻辑。这样税率相关的修改就被隔离在单个方法内了。如果税率计算逻辑过于复杂，也能更方便地将其移动到独立类中。



**类层面的封装**

可能会在一个以前完成简单工作的方法中添加越来越多的职责。新增行为通常还会带来更多成员变量和方法，使得包含它们的类主要职责变得模糊。将这些内容抽取到一个新类中会让程序更加清晰简洁。



```
// 修改前：在 Order 类中计算税金
Order
-taxCalculator
-lineItems
-country
-state
-city
...

+getOrderTotal()
+getTaxRate(country, state, product)

// 修改后：对 Order 类隐藏税金计算
Order ————> TaxCalculator
Order
-taxCalculator
-lineItems
-country
-state
-city
...

TaxCalculator
+getTaxRate(country, state, product)
-getUSTax(state)
-getChineseTax(product)
```





## **面向接口进行开发，而不是面向实现**

面向接口进行开发，而不是面向实现；依赖于抽象类，而不是具体类。

如果无需修改已有代码就能轻松对类进行扩展，那这样的设计就是灵活的。当需要两个类进行合作时，可以让其中一个类依赖于另一个类。但是，可用更灵活的方式来设置对象之间的合作关系。

1、    确定一个对象对另一个对象的确切需求：它需执行哪些方法？

2、     在一个新的接口或抽象类中描述这些方法

3、     让被依赖的类实现该接口

4、     现在让有需求的类依赖于这个接口，而不依赖于具体的类。你仍可与原始类中的对象进行互动，但现在其连接更灵活



```
修改前：
Cat ---> Sausage
Cat
-energy
+eat(Sausage s)

Sausage
...
+getNutrition()
+getColor()

修改后：
Cat ---> <<interface>> Food
Cat
-energy
+eat(Food s)

<<interface>> Food
+getNutrition()

Sausage ---▷ Food
Sausage
...
+getNutrition()
+getColor()
```





完成修改后，可能没法马上看到好处；相反，代码会变得比以前更加复杂。



**组合优于继承**

继承通常只有在程序中已包含大量类，且修改东西非常困难时才会引起关注。问题如下：

● 子类不能减少超类的接口。必须实现父类中所有的抽象方法，即使没用

● 重写方法时，需要确保新行为与基类中的版本兼容。因为子类的所有对象都可能被传递给以超类为参数的代码。

● 继承打破了超类的封装，因为子类拥有访问父类内部详细内容的权限。

● 子类与超类紧密耦合。超类的任何修改都可能会破坏子类功能

● 通过继承复用代码可能导致平行继承体系的产生

在多个维度上扩展一个类（汽车类型 X 引擎类型 X 驾驶类型）可能会导致子类组合的数量爆炸。子类将有大量的重复代码，因为子类不能同时继承两个类。可以使用组合解决这个问题。汽车对象可将行为委派给其他对象，而不是自行实现。还有个好处是可以在运行时对行为进行替换。例如，可以通过为汽车对象分配一个不同的引擎对象来替换已连接至汽车的引擎。



```
Transport ◆——> <<interface>> Engine
Transport ◇——> <<interface>> Driver
Transport
-engine
-driver
+deliver(destination, cargo)

<<interface>> Engine
+move()

CombustionEngine ---▷ Engine
ElectricEngine ---▷ Engine

<<interface>> Driver
+navigate()

Robot ---▷ Driver
Human ---▷ Driver
```









## **SOLID 原则**

SOLID 是让软件设计更易于理解、更加灵活和更易于维护的五个原则简称。



### **S：单一职责原则 Single Responsibility Principle**

修改一个类的原因只能有一个。尽量让每个类只负责软件的一个功能，并将该功能完全封装在该类中。该原则目的是减少复杂度。不需要构思使用多少行代码来实现复杂设计，完全可以使用十几个清晰的方法。

当程序规模不断扩大、变更不断增加后，问题才会逐渐显现。类过于庞大，以至于你无法记住细节。查找代码必须浏览整个类甚至整个程序。

如果感觉在关注程序特定方面的内容时有些困难，请回忆单一职责原则并考虑现在是否应将某些类分割成几个部分。

有几个理由对 Employee 类进行修改。第一个与该类的主要工作（管理雇员数据）有关。还有另一个理由：时间表报告的格式可能随着时间而改变，从而使你需要对类中的代码进行修改。解决该问题的方法是将与打印时间表报告相关的行为移动到一个单独的类中。这个改变让你能将其他与报告相关的内容移动到一个新的类中。



```
// 修改前：类中包含多个不同的行为
Employee
-name
+getName()
+printTimeSheetReport()

// 修改后：额外行为有了它们自己的类
TimeSheetReport ---> Employee
TimeSheetReport
...
+print(employee)

Employee
-name
+getName()
```







### **O：开闭原则 Open / closed Principle**

对于扩展，类应该是 “开放” 的；对于修改，类应是 “封闭” 的。本原则的理念是在实现新功能时能保持已有代码不变。

如果可以对一个类进行扩展，可以创建它的子类并对其做任何事情（如新增方法或成员变量、重写基类行为等），那么它就是开放的。有些语言允许通过特殊关键字（例如 final）来限制对类的进一步扩展，这样的类就不再是 “开放” 的了。如果某个类已做好了充分的准备并可供其他类使用的话（即其接口已明确定义且以后不会修改），那么该类就是封闭（完整）的。如果一个类已经完成开发、测试和审核工作，而且属于某个框架或者可被其他类的代码直接使用的话，对其代码进行修改就是有风险的。可以创建一个子类并重写原始类的部分内容已完成不同的行为，而不是直接对原始类的代码进行修改。这样既可以达成自己的目标，但又无需修改已有的原始类客户端。

该原则不能应用于所有对类进行的修改中。如果发现类中存在缺陷，直接对其进行修复即可，不要为它创建子类。子类不应该对其父类的问题负责。

程序中包含一个计算运输费用的 Order 类，该类中所有运输方法都已硬编码的方式实现。如果需要添加一个新的运输方式，那就必须承担对 Order 类造成破坏的可能风险对其进行修改。

可以通过应用策略模式来解决这个问题。首先将运输方法抽取到拥有相同接口的不同类中。当需要实现一个新的运输方式时，可以通过扩展 Shipping 接口来新建一个类，无需修改任何 Order 类的代码。当用户选择这种运输方式时，Order 类客户端代码会将订单链接到新类的运输方式对象。此外，根据单一职责原则，这个解决方案能够让你将运输时间的计算代码移动到与其相关度更高的类中。



```
// 修改前：在程序中添加新的运输方式时，必须对 Order 类进行修改
Order
-lineItems
-shipping
+getTotal()
+getTotalWeight()
+setShippingType(st)
+getShippingCost()
+getShippingDate()

getShippingCost(){
    // 大额订单免费陆运
    if (shippnig == "ground") {
        if (getTotal() > 100) {
            return 0
        }
    }
    // 每公斤1.5元，最少10元
    return max(10, getTotalWeight() * 1.5)
    
    if (shipping == "air") {
        // 每公斤3元，最少20元
        return max(20, getTotalWeight() * 3)
    }
}


// 修改后：添加新的运输方式不需要修改已有的类
Order ◇————> <<interface>> Shipping

Order
-lineItems
-shipping:Shipping
+getTotal()
+getTotalWeight()
+setShippingType(st)
+getShippingCost(){return shipping.getCost(this)}
+getShippingDate()

Ground、Air ---▷ <<interface>> Shipping

<<inteface>> Shipping
+getCost(order)
+getDate(order)

Ground
+getCost(order)
+getDate(order)

getCost(order) {
    // 大额订单免费陆运
    if (shippnig == "ground") {
        if (getTotal() > 100) {
            return 0
        }
    }
} 
```







### **L：里氏替换原则 Liskov Substitution Principle**

当扩展一个类时，应该要能在不修改客户端代码的情况下将子类对象作为父类对象进行传递。这意味着子类必须保持与父类行为的兼容。在重写一个方法时，要对基类行为进行扩展，而不是将其完全替换。

替换原则是用于预测子类是否与代码兼容，以及是否能与其超类对象协作的一组检查。这一概念在开发程序库和框架时非常重要，因为其中的类将会在他人的代码中使用——你是无法直接访问和修改这些代码的。与有着多钟解释方式的其他设计模式不同，替代原则包含一组对子类（特别是其方法）的形式要求。

 ● 子类方法的参数类型必须与其超类的参数类型相匹配或更加抽象

​	○ 假设某个类有个放阿飞用于给猫咪喂食：feed(Cat c) 。客户端代码总是将 cat 对象传递给该方法

​	○ good way：假如你创建了一个子类并重写了前面的方法，使其能够给任何 “动物 animal” 喂食：feed(Animal c) 。如果现在你将一个子类对象而非超类对象传递给客户端代码，程序仍将正常工作。该方法可用于给任何动物喂食。

​	○ bad way：创建另一个子类且限制 feed 方法仅接受 “孟加拉猫 BengalCat”：feed(BengalCat c) 。如果用它来替代链接在某个对象中的原始类，会因为无法为传递给客户端的普通猫提供服务，从而破坏相关功能。

● 子类方法的返回值类型必须与超类方法的返回值类型或是其子类别相匹配

​	○ 对于返回值类型的要求与对于参数类型的要求相反

​	○ 类中有一个方法 buyCat():Cat 。客户端代码执行该方法后的预期返回结果是任意类型的 “猫” 。

​	○ gw：子类将该方法重写为：buyCat():BangalCat。客户端将获得一只“孟加拉”猫。

​	○ bw：子类将该方法重写为：buyCat():Animal。客户端代码会出错，因为它获得的是自己未知的动物种类，不适用为一只 “猫” 而设计的结构

​	○ 编程语言中的一个反例是动态类型，基础方法返回一个字符串，但重写后的方法返回一个数字。

● 子类中的方法不应抛出基础方法预期之外的异常类型

​	○ 换句话说，异常类型必须与基础方法能抛出的异常或是其子类别相匹配。这条规则来源一个事实：客户端代码的 try-catch 代码块针对的是基础方法可能抛出的异常类型。因此，预期之外的异常可能会穿透客户端的防御代码，从而使整个应用崩溃。对于绝大部分现代编程语言，特别是静态类型的编程语言，这些规则已内置其中。如果违反了这些规则，将无法对程序进行编译。

● 子类不应该加强其前置条件

​	○ 假设基类方法有一个 int 类型的参数。如果子类重写该方法时，要求传递给该方法的参数值必须为正数（如果该值为负则抛出异常），这就是加强了前置条件。客户端现在使用子类的对象会使程序出错。

● 子类不能削弱其后置条件

​	○ 假如某个类中有个方法需要使用数据库，该方法应该在接收到返回值后关闭所有活跃的数据库连接。创建一个子类并对其进行了修改，使得数据库保持连接以便重用。但客户端可能对你的意图一无所知。由于它认为该方法会关闭所有的连接，因此可能会在对用该方法后就马上关闭程序，使得无用的数据库连接对系统造成 “污染” 。

● 超类的不变量必须保留

​	○ 这是所有规则中最不 “形式” 的一条。不变量是让对象有意义的条件。例如，猫的不变量是有四条腿、一条尾巴和能够喵喵叫等。不变量让人疑惑的地方在于它们既可通过接口契约或方法内的一组断言来明确定义，又可暗含在特定的单元测试和客户代码预期中。

​	○ 不变量的规则是最容易违反的，因为可能会无解或没有意识到一个复杂类中的所有不变量。因此，扩展一个类的最安全做法是引入新的成员变量和方法，而不要去招惹超类中已有的成员。但在实际中，这并非总是可行。

● 子类不能修改超类中的私有成员变量的值 

​	○ 有些编程语言允许通过反射机制来访问类的私有成员。还有一些语言（Python、JavaScript）没有对私有成员进行保护。



```
// 违反替换原则的文档类层次结构例子
// 修改前：只读文件中的保存行为没有任何意义，因此子类试图在重写后的方法中重置基础行为来解决这个问题
Document
-data
-filename
+open()
+save()

ReadOnlyDocument ————▷ Document
ReadOnlyDocument
+save(){throw new Exception("无法保存只读文件。")}

Project ◆————> Document
Project
-documents
+openAll() {
    foreach (doc in documents){
        doc.open()
    }
}
+saveAll() {
    if (!doc instanceof ReadOnlyDocument){
        doc.save()
    }
}

// 修改后：当把只读文档类作为层次结构中的基类后，这个问题得到了解决
Document
-data
-filename
+open()

WritableDocument ————▷ Document
WritableDocument
+save()

Project ◆————> Document
Project
-allDocs
-writableDocs
+openAll(){
    foreach(doc in allDocs){
        doc.open()
    }
}
+saveAll(){
    doc.save()
}

// 可以通过重新设计类层次结构来解决这个问题：一个子类必须扩展其超类的行为，因此只读文档变成层次结构的基类。可写文件变成了子类，对基类进行扩展并添加了保存行为。
```







### **I：接口隔离原则 Interface Segregation Principle**

客户端不应被强迫依赖于其不使用的方法。尽量缩小接口的范围，使得客户端的类不必实现其不需要的行为。

根据接口隔离原则，必须将 “臃肿” 的方法拆分为多个颗粒度更小的具体方法。客户端必须仅实现其实际需要的方法。否则，对于 “臃肿” 接口的修改可能会导致程序出错，即使客户端根本没有使用修改后的方法。

继承只允许类拥有一个车超类，但是并不限制类可同时实现的接口的数量。不要将大量无关的类塞进单个接口，可将其拆分为更精细的接口，如有需要可在单个类中实现所有接口，某些类也可只实现其中的一个接口。

假如创建了一个程序库，能让程序方便地与多种云计算供应商进行整合。尽管最初版本仅支持阿里云服务，但它也覆盖了一套完整的云服务与功能。假设所有云服务供应商都与阿里云一样提供相同种类的功能。但当你着手为其他供应商提供支持时，程序中绝大部分的接口会显得过于宽泛。其他云服务供应商没有提供部分方法所描述的功能。



```
// 修改前：不是所有客户端能满足复杂接口的要求
// 尽管可以去实现这些方法并放入一些桩代码，但这不是优良的解决方案。更好的方法是将接口拆分为多个部分。能够实现原始接口的类现在只需改为实现多个精细接口即可。其他类可仅实现对自己有意义的接口
<<interface>> CloudProvider
+storeFile(name)
+getFile(name)
+createServer(region)
+listServer(region)
+getCDNAddress()

AlibabaCloud ---▷ CloudProvider
+storeFile(name)
+getFile(name)
+createServer(region)
+listServer(region)
+getCDNAddress()

TencentCloud ---▷ CloudProvider
+storeFile(name)
+getFile(name)
---未实现---
+createServer(region)
+listServer(region)
+getCDNAddress()

修改后：一个复杂的接口被拆分为一组颗粒度更小的接口
<<interface>> CloudHostingProvider
+createServer(region)
+listServers(region)

<<interface>> CDNProvider
+getCDNAddress()

<<interface>> CloudStorageProvider
+storeFile(name)
+getFile(name)

AlibabaCloud ---▷ CloudHostingProvider、CDNProvider、+storeFile(name)
+getFile(name)
+createServer(region)
+listServer(region)
+getCDNAddress()

TencentCloud ---▷ CloudStorageProvider
+storeFile(name)
+getFile(name)

// 与其他原则一样，不要过度使用这条原则。不要进划分已经非常具体的接口。创建的接口越多，代码就越复杂。
```







### **D：依赖倒置原则 Dependency Inversion Principle**

高层次的类不应该依赖于低层次的类。两者都应该依赖于抽象接口。抽象接口不应依赖于具体实现。具体实现应该依赖于抽象接口。

通常在设计软件软件时，可以辨别出不同层次的类。

● 低层次的类实现基础操作（例如磁盘操作、传输网络数据和连接数据库等）

● 高层次类包含复杂业务逻辑以知道低层次类执行特定操作

有时人们会先设计低层次的类， 然后才会开发高层次的类。 当你在新系统上开发原型产品时，这种情况很常见。由于低层次的东西还没有实现或不确定，你甚至无法确定高层次类能实现哪些功能。如果采用这种方式，业务逻辑类可能会更依赖于低层原语类。

依赖倒置原则建议改变这种依赖方式。

1、     最好使用业务术语来对高层次类依赖的低层次操作接口进行描述。例如，业务逻辑应该调用名为 openReport(file) 的方法，而不是 openFile(x)、readBytes(n)等一系列方法。这些接口被视为是高层次的。

2、     现在可基于这些接口创建高层次类，而不是基于低层次的具体类。这比原始的依赖关系灵活很多。

3、     一旦低层次的类实现了这些接口，他们将依赖于业务逻辑层，从而倒置了原始的依赖关系。

依赖倒置原则通常和开闭原则共同发挥作用：你无需修改已有类就能用不同的业务逻辑类扩展低层次的类。

示例，高层次的预算报告类 (BudgetReport) 使用低层次的数据库类 (MySQLDatabase) 来读取和保存其数据。这意味着低层次类中的任何改变（如数据库服务器发布新版本时）都可能会影响到高层次的类，但高层次的类不应关注数据存储的细节。



```
// 修改前：高层次的类依赖于低层次的类
BudgetReport ————> MySQLDatabase

高层次
BudgetReport
-database
+open(date)
+save()

MySQLDatabase
+insert()
+update()
+delete()

// 要解决这个问题，可以创建一个描述读写操作的高层接口，并让报告类使用该接口代替低层次的类。然后可以修改或扩展低层次的原始类来实现业务逻辑声明的读写接口
// 修改后：低层次的类依赖于高层次的抽象
高层次
BudgetReport ————> <<interface>> Database
BudgetReport
-database
+open(date)
+save()

<<interface>> Database
+insert()
+update()
+delete()

低层次
MySQL、MongoDB ---▷ Database
+insert()
+update()
+delete()

// 其结果是原始的依赖关系被倒置：现在低层次的类依赖于高层次的抽象
```