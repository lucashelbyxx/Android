# OOP, OOD

Object Oriented Programming, Object Oriented Design

我们已经能够良好的写出单一的class，不管他有没有带指针，写法不一样，都能够应付了。这种叫基于对象的设计。对于复数、字符串这样的类，它们基本不会跟其他的类发生关系。

但是考虑复杂问题的时候，就需要让类跟类之间产生关系。这个就叫面向对象的编程，面向对象的思想。

首先要探讨的是类和类之间有些的关系。我认为，我们只要了解这三种关系，就足够解决接下来所有类跟类之间的关系。

- Inheritance (继承)
- Composition (复合)
- Delegation (委托)



Composition (复合)，表示 has-a

```cpp
template <class T, class Sequence = deque<T> >
class queue {
    ...
protected:
    Sequence c;	// 底层容器
public:
    // 以下完全利用C的操作函数完成
    bool empty() const { return c.empty(); }
    size_type size() const { return c.size(); }
    reference front() { return c.front(); }
    reference back() { return c.back(); }
    // deque 是两端可进出，queue 是末端进前端出(先进先出)
    void push(const value_type& x) { c.push_back(x); }
    void pop() { c.pop_front(); }
}
```

标准库里取出来的一个例子，queue。queue里面有一个东西，这个东西类型叫sequence。第一行告诉我们，sequence默认就是deque类型(double end queue)。刚开始这样看可能不太习惯，把deque替换进来，变成下面。

```cpp
// queue ◆——> deque
template <class T>
class queue {
    ...
protected:
    deque<T> c;		// 底层容器
public:
    // 以下完全利用 c 的操作函数完成
    bool empty() const { return c.empty(); }
    size_type size() const { return c.size(); }
    reference front() { return c.front(); }
    reference back() { return c.back(); }
    //
    void push(const value_type& x) { c.push_back(x); }
    void pop() { c.pop_front(); }
}
```

这个 queue class 里面有个变量 c，c 是 deque，它是个模板类型。这个class有一个这种东西，这样我们就叫composition。我里头有另外一种东西，我跟另外那个的关系就叫composition，表现出来的是has-a的关系，我有n个这样的东西。

这种关系在面向对象出现之前早就存在。回想C语言的结构，C的struct里面就可以放其他的结构、整数、字符串。我们要慢慢习惯用图来表现类跟类之间的关系，尤其以后系统越来越庞大。你要用图来表现，而不是直接看源代码。

黑色菱形表示有东西，白色是另外一个话题。黑色菱形这端就是容器，它容纳了、拥有了另外一个东西。这样叫composition。

queue拥有了c之后，这几乎是完整的代码，前面有次要的...被拿掉了。这意思是queue里面的所有功能它都没有自己写，它都是调用它拥有的c来完成的。所有的功能都在deque这里已经完成了，而我这个queue想借用这个已经完成的功能来实现我的功能，所以queue写法非常简单。这是一个比较特殊的例子，要讲composition，只要 A 拥有 B，这样就是composition。现在这个例子是说 A 拥有 B，并且 A 的所有功能都让 B 来做，A自己不再做新的功能了，这是特例。这种特例让我们感受到queue拥有的这个东西显然功能很强大，它可以完全满足queue所需要的功能。

queue 是队列，先进先出。deque 是双端队列，两端都可进出，功能更加强大。

我们借由这个例子讲复合，而这个例子又比较独特，表现出一种什么形式呢？你已经有一个功能蛮强大的东西了，现在只是把它改装一下，就变成了另外一个class。什么叫改装？queue里面有一个函数pop()，这个函数其实它自己没有去做事情，它只是调用它里头的那个c的pop_front()做事情。所以就是deque里头的本来的名词pop_front，现在改头换面，没有多做什么事，pop_front就变成了pop，这就是刚刚所说的换了个面貌出现。

而说不定这个deque有100个功能，但是被包进来只开放了6个功能，而且名字也可能换了。存在这种情况的话，这就表现出一个设计模式，叫Adapter (改造、适配)。用在这个地方就是这个东西已经很棒了，你手上已经有了这个东西了。现在客户需要另外一个东西，但是你手上的这个东西的功能其实完全满足，只是可能名称不一样、接口不一样，那我们就说我们来写一个adapter好了，把它改造一下，只是把名字换一下。那这里queue就是一个adapter。

<br>

从内存角度解释 Composition

```cpp
// Sizeof:40
template <class T>
class queue {
protected:
    deque<T> c;
...
};

// Sizeof:16*2+4+4
// queue ◆——> deque
template <class T>
class deque {
protected:
    Itr<T> start;
    Itr<T> finish;
    T**	map;	// 指针的指针
    unsigned int map_size;
};

// Sizeof:4*4
// deque ◆——> Itr
template <class T>
struct Itr {
	T* cur;
    T* first;
    T* last;
    T** node;
...
};
```

<br>

Composition (复合) 关系下的构造和析构

```cpp
// Container ◆——> Component
// Container object 包含 Component part

// 构造由内而外
// Container 的构造函数首先调用 Component 的 default 构造函数，然后才执行自己。
Container::Container(...) : Component() { ... };

// 析构由外而内
// Container 的析构函数首先执行自己，然后才调用 Component 的析构函数。
Container::~Container(...) { ... ~Component() };
```

构造一定要由内而外构造，这样基础才稳定。这在我们人类的生活理念中也是这样，你要做一个东西，如果它有内部也有外部，当然是把里面构造好了才扎实。所以C++也要表现出这种特性。

构造函数有它的初值列，“Component()” 表示的是外面的构造函数调用里面的构造函数，等它执行完后才做外部自己的事情。强调先后关系。我们在写外部的Container其实不管这个东西，是希望C++语言帮我们实现这个东西，因为我们觉得这样很合理。“Component()” 是编译器帮我们加上去的。注意，也许构造函数有几个，编译器不知道调哪个，只能调用默认这个。要指定哪个带参构造函数的话，需要自己写。

析构函数，从外面一层层剥开，不能先把最里面的抽掉，抽掉整个塌了。

<br>

Delegation (委托). Composition by reference.

```cpp
// String ◇——> StringRep

// file String.hpp
// Handle (pImpl)
// 1
class StringRep;
class String {
public:
    String();
    String(const char* s);
    String(const String& s);	// 拷贝构造
    String &operator=(const String& s);		// 拷贝赋值
    ~String();	// 析构
...
private:
    StringRep* rep; // pimpl, pointer to implementation
};


// file String.cpp
// Body (pImpll)
// 2
#include "String.hpp"
namespace {
class StringRep {
freind class String;
    StringRep(const char* s);
    ~Stringrep();
    int count;
    char* rep;
};
}

String::String() { ... }
...
```

这个字符串类跟我们之前设计的单一字符串类很像，由于它有指针，所以要注意 big three。之前版本的指针是指向字符，现在指向另外一个东西 StringRep。

这两个类是什么关系呢？Delegation，菱形白色，空心表示指针。就是说 String类有一个东西，但是这个东西有点虚，只是有个指针指向这个东西，至于什么时候真的拥有它，目前还不知道。

为什么叫委托呢？我拥有这个指针指向你之后，在任何一个我想要的时间点，我就可以调用你来做事情，把任务委托给你。当然，前面那个 composition 复合，我也可以把任务委托给我复合的那个东西啊，也叫委托啊？这是术语上的分配。Delegation 还有另外一个术语 Composition by reference，它也是有东西，只不过它有的是指针。怎么不讲 by pointer 呢？从前面以来，从来没有讲过 by pointer 这个术语，那是因为在学术界里面不讲 by pointer，只讲 by reference，即使你用指针在传，也叫 by reference。

什么叫委托？两个类之间用指针相连。用指针相连的话，它们的寿命就不一致。前面的 composition 是如果有一个外部的，就有了内部的，它们的生命是一起出现的。Delegation 是 String 可能先创建出来，可是里面有指针，等到需要 StringRep 的时候才去创建，不同步。这种写法非常有名，pointer to implementation，我有一根指针指向为我实现所有功能的那个类。字符串的设计不在 String 类写出来，在 StringRep 类写出来，String 只是对外的接口，至于真正的实现都在 StringRep 做。当 String 需要动作的时候都调用 StringRep 来服务，它们之间永远是这样的关系。为什么这样有名？因为如果把所有的类都写成这样的话，String 类对外不变，StringRep 是真正的实现，我们可以切换，将来这根指针可以指向不同的实现类。这具有一种弹性，2怎么变都不会影响1，也就不影响客户端。这种手法也叫编译防火墙，1永远不用再编译，要编的只是2而已。

```
// abc 是 rep 指针
a、b、c —-rep-—> 整数和指针指向 —-rep-—>  "Hello"
```

它想要做的是 reference counting，共享的特性。什么叫引用计数呢？就像上面的情况，三个字符串都在用同一个 hello。共享当然是好事情，很直观的想至少内存节省了。共享的前提是内容要一样。

要跟人家发生共享，就要注意千万不能牵一发而动全局。现在abc内容都是hello，当然它们互相不知道，a要把hello改变不能影响b、c。如何做到呢？当你要改变内容的时候，整个系统就说，我单独拿一份给你改。本来是三个人共享一个内容，a想改变内容，copy一份给a来改，剩下b、c共享原来那个东西。这个概念叫做 copy on write，写入时复制。

<br>

Inheritance (继承)，表示 is-a

有的人有误解，觉得继承才是面向对象，其他两种太简单的，过去都有了，用指针来做的composition、用实际东西来做的composition。其实这三种关系都是面向对象的一部分。

```cpp
// struct 是一种 class
// 1
struct _List_node_base
{
    _List_node_base* _M_next;
    _List_node_base* _M_prev;
};


// _List_node T ——▷ _List_node_base
// 2
template<typename _Tp> 
struct _List_node 
    : public _List_node_base
{
    _Tp _M_data;
};

```

这里有两个class，它们之间要表示一种继承关系的话。语法就是加 “: puclic _List_node_base”。子类往父类画一个空心三角形。T 表示 type。C++ 给你三种继承方式，这里是public，而在Java只有public继承，不必写。使用 public 继承就是在传达一种逻辑 is-a，是一种。

现在从内存角度看看它。现在这个设计没有函数，只有data，所以这个设计只是要利用继承的观念。数据是被继承下来的，父类的数据是完整的被继承下来的。1class 对象有两根指针。2对象有自己的一个东西，还有父类的这两个。这不是继承最有价值的一部分。

继承最有价值的是跟虚函数的搭配。



Inheritance (继承) 关系下的构造和析构

继承关系下就有父类和子类 (派生类)。从内存的角度看，子类的对象里头有父类的成分。composition也是有一个外部包着内部。

```cpp
// Derived ——▷ Base
// Derived object 包含 Base part
// base class 的 dtor (析构函数) 必须是 virtual，否则会出现 undefined behavior。

// 构造由内而外
// Derived 的构造函数首先调用 Base 的 default 构造函数，然后才执行自己。
Derived::Derived(...) : Base() { ... };

// 析构由外而内
// Derived 的析构函数首先执行自己，然后才调用 Base 的析构函数。
Derived::~Derived(...) { ... ~Base() };
```

父类的析构函数必须是 virtual，要不然不会有析构由外而内的好行为。只要你的class现在或将来会变为父类，你就把你的析构函数设为virtual。



Inheritance (继承) with virtual functions (虚函数)

刚刚谈到继承的时候，继承要搭配虚函数来使用才能达到最强而有力的效果。在语法上，只要在任何成员函数前加上 virtual 这个关键字，它就成为一个虚函数。

在继承的关系里面，所有的东西都可以继承下来。数据可以继承下来，就占用内存的一部分。函数也可以被继承下来，可是函数的继承在内存的角度怎么理解呢？不应该从内存的角度理解。函数的继承，继承的是调用权。父类跟子类，子类可以调用父类的函数，这叫继承了函数，其实是继承了调用权。子类要不要重新定义函数呢？

- non-virtual 函数：你不希望 derived class 重新定义 (override, 复写)它；

- virtual 函数：你希望 derived class 重新定义 (override, 复写)它，且你对它已有默认定义。

- pure virtual 函数：你希望 derived class 一定要重新定义 (override, 复写)它，你对它没有默认定义。

```cpp
class Shape{
public:
    // pure virtual
    virtual void draw() const = 0;
    
    // impure virtual
    virtual void error(const std::string& msg);
    
    //  non-virtual
    int objectID() const;
    ...
};

class Rectangle: public Shape { ... };
class Ellipse: public Shape { ... };
```

其实纯虚函数可以有定义，但是现在不去提它。

例如，我现在设计一个class叫shape，世界上并没有叫形状的形状。形状是一个非常抽象的概念，但是现在我们在抽象层面上思考。我们说需要一个形状，将来我才继承下去有四方形、椭圆形，它们都可以继承自形状。然后思考在抽象这个层次我可以写些什么？



Inheritance (继承) with virtual

举个例子，我使用 PowerPoint，当我要开启一个文件的时候，把菜单打开选下去 (档案 —> 开启旧档)，会跑出一个对话框，里面条列出来让我选择或者是我在下面这里打入 filename，按下 ok 之后，程序应该收到一个 filename。它应该去检查 filename 是否正确 (Check file name)，有没有不可以有的字符，然后它应该去硬盘去找这个 file 在不在 (Search file)，找到之后把该文件 open 打开来 (Open file)，打开之后把文件读出来.

上面这些动作，其实任何人来写这段代码都差不多，表现出来的形式也差不多，那何不有人把它先写好呢？能不能有人把它先写好，让后面的人来用。这些步骤只有最后这个步骤开启文件读的内容没办法事先写。因为有可能我写一个简报软件，你写一个文书软件，我们最后要读出来的东西不一样，这件事情没办法事先写好，除此之外，所有的都可以事先写。于是有人说，那我来开发这种产品好了。

```cpp
// CMyDoc ——▷ CDocument
// Template Method
// Application framework
CDocument::OnFileOpen()
{
    ...
	Serialize()
	...
}

// Application
class CMyDoc: public CDocument
{
    virtual Serialize() { ... }
};


main()
{
    CMyDoc myDoc;
    ...
	myDoc.OnFileOpen();
    // 等同 CDocument::OnFileOpen
}
```

CDocument 里面有一个函数叫 OnFileOpen()，为了呼应刚才要对付的事情。它就把刚才的动作全部写好。在不同平台下开启一个文件动作都是一样的。但是它有一个动作写不出来，就是读内容这个动作Serialize()。

所以根据刚刚的说法，一个父类有一些函数没有办法先写出来，要让子类去写它。这种我们就必须把它设计为 virtual function。

读这个动作 Serialize()，没有写出来，但它应该是一个虚函数。它是纯虚函数还是怎样的情况呢？你可以把它设计为纯虚函数，如果把 CDocument 是纯虚函数的话，那么 CMyDoc 一定要去写它。或者把它设计成不是纯虚函数，那你应该要有默认的定义咯，可是谁也不知到这个默认定义怎么办。也许有些人写一个空函数，但是空函数跟纯虚函数意义不一样。纯虚函数子类一定要去覆写。空函数是子类不一定要覆写，但为了功能正确起见，子类还是应该要写它，但是万一子类没有写它，是会通过编译的。

怎么使用？CDocument 是有一个团队写好的，而且它还可以拿来卖钱。我们把它买过来之后，我们写自己的子类。它可能是一年前写好的，CMyDoc 才是现在要写的。我们写 CMyDoc，里头就写父类完成不了的事情 virtual Serialize() 。然后看看怎么用，非常经典的虚函数用法。main() 里头创建子类对象，子类对象调用父类的函数 (这是面向对象语言里非常常见的动作)。myDoc.OnFileOpen() 的全名叫 CDocument::OnFileOpen(&myDoc)，所以编译器才知道你调用的是 CDocument::OnFileOpen()。CDocument::OnFileOpen() 这个函数里面执行到 Serialize() 这个虚函数的时候，它看到子类中有写这个虚函数，它就会跑过来。这正是我们想要的，在最关键的这一刻呢，跑到我们写的这个函数来，执行完毕后再跑回去，然后才回到调用端 main()。

这个做法是如此经典，以至于它有一个专属的名称。CDocument::OnFileOpen() 这个函数，它里头做了一些固定的动作，把其中的一个关键部分延缓到子类去决定。Serialize() 就是一个关键动作，父类是写不出来的，延缓也许是一年三年之后由子类写出来。我们就把CDocument::OnFileOpen() 这个函数或者这个做法，叫做 Template Method，大名鼎鼎的23个设计模式之一。不要误会 template，C++也有 template，但是这里不是C++的那个模板，这里是套用这个概念。method 是 Java 的术语，Java 语言说函数就是 method。所以 template method 就是C++里面的某一个函数，就是我们刚刚将的 CDocument::OnFileOpen() 。

这种用法你可以想想对于一个做框架的人，framework。什么叫框架呢？我先帮你想好了你想设计一个应用程序，你该有哪一些功能，这些功能是差不多的。比如说在 Windows 底下所有的程序，左边菜单拉下来是怎么样，它的旁边应该是怎样，如果开一个文件，需要出现一个对话框，然后应该怎么样，这些都是固定的。大家写程序的竞争力不在于这些一般性的动作上，而是在于你的知识领域上。所以这些一般性的动作谁来写都一样。那么就有人说，我来帮大家把这个写好，这个叫框架，应用程序的框架。那在这种应用程序的框架里面就会大量用到 template method 的手法，也就是说它先把固定可以先写的写好，留下他无法决定的函数，让它成为一个虚函数，让你的子类去定义它。这种框架当然是造福人类啦，有人帮我们把要写的一大堆东西都写好。有这么一种产品，最有名的一种在十多年前横扫全世界的一个产品叫 MFC (MicroSoft Foundation Classes)。

你可能会好奇为什么 CDocument::OnFileOpen() 中的 Serialize() 会跑到 CMyDoc？myDoc.OnFileOpen() 的全名叫 CDocument::OnFileOpen(&myDoc)，谁调用我的那个谁就会变成 this pointer。所以如果我们扮演编译器的角色，myDoc 这个调用者它的地址就被放进来，变成一个隐藏的参数，这个就是 this。this 就被传进 CDocument::OnFileOpen() ，这个函数调用 Serialize 的时候，其实编译器是看成 this -> Serialize()，是通过 this 来调用。

我们用很经典的例子来引导大家去了解继承搭配虚函数最重要的用法。

```cpp
// 把刚刚的过程模拟一次
#include <iostream>
using namespace std;

class CDocument
{
public:
    void OnFileOpen()
    {
        // 这是个算法，每个cout输出代表一个实际动作
        cout << "dialog..." << endl;
        cout << "check file status..." << endl;
        cout << "open file..." << endl;
        Serialize();
        cout << "close file..." << endl;
        cout << "update all views..." << endl;
    }
    
    virtual void Serialize() { };
};


class CMyDoc : public CDocument
{
public:
    virtual void Serialize()
    {
        // 只有应用程序本身才知道如何读取自己的文件 (格式)
        cout << "CMyDoc::Serialize()" << endl;
    }
};


int main()
{
    CMyDoc myDoc;	// 假设对应[File/Open]
    myDoc.OnFileOpen();
}
```



Inheritance+Composition 关系下的构造和析构

```cpp
// 1
// Derived ——▷ Base
//		   ◆——> Component
// Derived object 包含 Base part、Component part


// 2
// Derived ——▷ Base ◆——> Component
// Derived object 包含 Base part，Base part 包含 Component part 
```

1父类跟子类，子类里头又有一个Component。从内存角度来画，可以说子类里头有父类的part，又有Component。可是要把Base放前面，还是把Component放前面呢？不知道，没关系。我们只要知道它们存在于子类的内容物就好了。

当你去创建Derived的时候，它的构造函数应该先去调用Base，也去调用Component，可是谁先呢？这个先后影响不大，主要让我们深刻理解内存布局。可以建立三个类，并且建立继承、复合的关系，在三个类构造函数里头打印消息出来，你就可以观察谁先谁后。

2子类里头有父类的part，父类里头又有Component的成分。Derived object 包含 Base part，Base part 包含 Component part。当要创建一个子类的时候，构造函数的次序当然是最里头的最先被调用。析构时候刚好相反。



Delegation (委托) + Inheritance (继承)

三种关系中，最强大的是这种。

PowerPoint的一种功能，你可以打开一个文件，并且在窗口这个地方下拉开一个新窗口，那我开了四个。其实我这样的动作是文件只有一份，窗口有四个。所以等同于我用4个放大镜或窗户看同一个东西。或者是我有一份数据，有三种不同的view在看它。要表现出这种行为出来，储存数据和表现数据这两个class之间要具备怎样的关联性，才能表达这种状态呢？这种状态是如果这里有变化，其他也要跟着变化，因为真正的data只有一份。如果数据有变化，条状图、折线图都要跟着变化，这样才对。几乎所有写 UI (user interface, 使用者接口) 的程序员一定会碰到这种问题，你一定要去解决这种问题。

```cpp
class Subject
{
    int m_value;
    vector<Observer*> m_views;
public:
    void attach(Observer* obs)
    {
        m_views.push_back(obs);
    }
    
    void set_val(int value)
    {
        m_value = value;
        notify();
    }
    
    void notify()
    {
        for (int i = 0; i < m_views.size(); i++)
            m_views[i]->update(this,m_value);
    }
}；
    

// Observer
class Observer
{
public:
    virtual void update(Subject* sub, int value) = 0;
}


// Subject ◇——>n Observer
//  ...      ——▷
```

设计一个 class 叫 Subject ，放数据在这里。再设计一个 class 叫 Observer 去观察 Subject 的，就是刚刚4个窗口去观察它。Subject 可以拥有很多个 Observer，因为使用者可以开出很多个 Observer。所以应该怎样准备数据呢？我们要准备一个容器，里头放指针指向 Observer 这种东西。这就是 Delegation，指针嘛。我们有一个例子说左边是字符串，右边是字符串的的实现，当时我说那个用途不是非常大，因为左跟右是一对一，也不能变化。现在右边 Observer 可以被继承，所以 Observer 只是个父类，将来派生下去观察按一秒一次的或者是什么情况观察一次的，下面的子类都是一种 Observer。那既然都是一种 Observer，下面这些东西创建出来之后统统可以放到这个容器里头。

作为内容物，应该提供一个注册、注销的动作。你们谁想观察我，你们都要过来注册啊。所以 Subject 应该要提供一个函数叫 attach，要附着谁呢？要附着 Observer 这种东西，就把它放到容器，这个容器叫 m_views。还应该有注销的功能，但是这里没有呈现出来。比如我是一个设计电子报的人，我允许被登记、被注册、被注销。还应该有一个函数叫 notify，把容器里所有的观察者去遍历询访一遍，去通知它。那通知这个动作要怎样完成，这个要双方左右说好，Observer 说我有一个函数叫 update，Subject 说我要调用 update，通知它们更新数据。

我们现在进行的是面向对象设计，而面向对象里面我们就要考虑需要准备哪些 class 来解决某一个问题。在谈到 OOD (object oriented design)，最经典的是23个设计模式。



# 委托相关设计

我们整个课程叫C++面向对象程序设计，有基于对象部分，有面向对象部分。在面向对象类的关系上面，我们有三把大刀，继承、复合、委托。举一些例子，处理这些问题的话，你该怎样设计你的类，应该用这三把大刀的那些刀子组合起来。

刚刚提到的问题和解法有 Adapter、Handle/Body (pImpl)、Template Method、Observer。面对我们的问题，应该怎样去构思我们的类，用这三把刀把它串联起来。

现在面对的问题是请你写一个 file system 或者 window system。如果想 file system，你可以想到的是有文件、目录，目录里可以放文件，目录还可以与其他目录结合在一起再放到另外一个目录里头。现在我们面对的就是这种奇特的形式的问题。该怎么办呢？我该准备哪些 class 才能解决这些问题？我该使用哪些武器把它们关联起来？



Delegation (委托) + Inheritance (继承)

就以file system来讲好了，我认为我需要写出一个代表file的class，这里叫Primitive，也许叫file更具体一些，但是现在命名是比较泛化的。Primitive是基本的个体，不是组合物。另外我还要准备一个class叫Composite，就是组合物。

作为Composite它应该是一个容器，因为它应该可以容纳很多个Primitive。但是这个问题是如此特别，以致于Composite容器还可以容纳它自己这种东西。我们有一个容器可以放Primitive，也可以放Composite，怎么办呢？当我们要选C++的容器的时候，我们要声明，怎么声明才好呢？为Primitive和Composite写一个父类的话，Primitive is a Component，Composite 也是 is a Component。这样就好办了，Composite 里的容器，不写死为Primitive、Composite，我说我要放的是Component这种东西，而且放的是指针。因为在C++里面必须放指针，你不能放真正的东西，在容器里放的东西一定要一样大小，那现在这些不一样大小。你要放指针，指针一定是一样大小。这个基本雏形就形成了。

然后Composite是一个组合物，它应该具备一个函数叫add，参数怎么设计呢？加这个动作又要可以加Primitive、又要可以加Composite，怎么办？让它的参数是一个pointer，指向父类。那么传进来的可能是Primitive指针，也可能是Composite指针，都接受。

我们发现在很多经典的问题里面，经典的解法都是要用Delegation、Inheritance这两把武器。这样的一个结构，或者说这个结构所要解决的问题，这就是一个解法，我们就把这个解法叫做Composite，设计模式之一。

至于这里头除了add把元素加进去之外，这个Composite还应该具备哪些功能呢？以 file system 而言，可能有dir，把里面所有东西都条列出来，那就不在这个考量之中了。我们把这三个class以这种关系结合在一起，已经可以描述这个问题所在，去解决这个问题了。

```cpp
// 我们开始习惯使用这种图，用这种图来讲解，在讲解的过程中相当于把代码讲了
Component
-int:int
add(e:component*)

// Primitive ——▷ Component

// Composite ——▷ Component
// Composite ◇——>n Component
Composite
-c:vector<Component*>
add(e:Component*)


// code
class Component
{
    int value;
public:
    Component(int val) { value=val; }
    virtual void add( Component* ) { }
};


class Primitive:public Component
{
public:
    Primitive(int val):Component(val) {}
};


class Composite:public Component
{
    vector<Component*>c;
public:
    Composite(int val):Component(val) { }
    
    void add(Component* elem) {
        c.push_back(elem);
    }
...
};
```

如果你不是非常习惯的话，还是看下代码。Component里头的add应该要被下面的子类重新定义，所以这个add是虚函数。如果你把它设计为纯虚函数，那又不行。设计为纯虚函数表示你没有定义，子类一定要去定义它。Primitive基础物没有办法去做加的动作。你的语法又要它去定义，但在现实中又不能去定义，所以这里add不能写成纯虚函数。add写成一个空函数，它没有做事情，Primitive可以继承自Component而不改变，继承下来不做事情是对的。你在一个文件系统里，有文件，有目录，你对于文件加东西是加不进去的。

<br>

Prototype

最后一个例子，题目说我需要一个树状继承体系，我想要创建未来才会出现的子类，怎么办？就像我在设计一个框架，这是子类是未来才会派生下去的，是被客户买下才派生子类，这个时候class名称才出现。而我可能是三年前写的Image，我不知道未来的class名称是什么，我怎么去创建它呢？不可能，我也不能读到一个字符串，然后这个字符串里面是一个class名称，然后我new这个字符串。不行，new后面一定要加class name，但现在class name还没出现。怎么办？

可不可以让下面的子类你们都创建一个自己，当成原型，让我框架这边可以有办法看到你们创建出来的原型放在什么位置上。我反正可以看到它，这样我就可以复制它，就等于我在创建了。不止在Prototype会出现这个问题，其他设计模式也会出现，就是我现在要去创建未来的class对象。解法就是前面那样。

```
Image
prototypes[10]:Image*
clone():Image*
findAndClone(i):Image*	{ return prototypes[i]->clone(); }
addPrototype(p:Image*):void

--- abstraction ---
LandSatImage ——▷ Image
LandSatImage
_LSAT:LandSatImage	//静态
-LandSatImage()	{ addPrototype(this); }	//- 表示 private，+ 表示 public(默认)
#LandSatImage(int)	//# 表示 protected
clone():Image*	{ return new LandSatImage; }

SpotImage ——▷ Image
SpotImage
_SPOT:SpotImage
-SpotImage()	{ addPrototype(this); }
#SpotImage(int)
clone():Image*	{ return new SpotImage; }
```

刚刚提到的想法，子类自己去创建自己。拿LandSatImage来讲，我在class里面安排一个静态的对象，故意让_LSAT是它自己。所以每一个子类这样写的话，它们就创建了自己。这就是所谓的原型，而这个原型必须登记到以前写好的框架端，要让它能看得到，怎么办呢？应该让框架端准备一个空间，让子类放上去，框架端父类就可以看见去做处理。

这个静态_LSAT的自己创建出来的时候会调用起构造函数。故意吧构造函数设为私有的，刚刚自己创建的时候私有函数会被调用起来吗？构造函数应该被调用起来，但是目前它是私有的，这样可以吗？可以，不是外界来调用这个函数，而是类自己。借用私有的构造函数调用addPrototype(this)把自己add上去。addPrototype是父类写，把得到的指针放到容器里面，这是个数组。

然后这些子类你们都应该各自准备一个函数clone()，就是new自己。刚刚已经有一个原型了，框架端可以通过这个原型对象调用clone函数做出一个副本。如果没有这个原型的话，就没有办法通过对象调用clone函数。这样的话，如果我不要原型，让clone是一个静态函数，那不也是调得到吗？不需要对象就能调啊，它是个静态函数。静态函数的调用需要class name，可是现在讲的是我们不知道未来的class。

反省一下，这么说子类是不是带着一种任务、开销(overhead)呢？本来不知道这个事情的时候，我自由自在地派生下去子类写东西，现在却告诉我说必须像这样有个静态的自己，必须写一个私有的构造函数，必须写一个clone函数。这就是我的负担，合理吗？合理，世界上没有白吃的午餐。你要搭配事先写好的框架，要让它知道你自己，就必须有一些额外的开销。这只是一种做法，在其他做法可以看到不可能你什么都不做就能和框架搭配在一起。

```cpp
// 原作者写法
// 父类
#include<iostream.h>
enum imageType
{
	LSAT, SPOT
};

class Image
{
public:
    virtual void draw() = 0;
    static Image *findAndClone(imageType);
protected:
    virtual imageType returnType() = 0;
    virtual Image *clone() = 0;
    // As each subclass of Image is declared, it registers its prototype
    static void addPrototype(Image *image)
    {
        _prototypes[_nextSlot++] = image;
    }
    private:
    // addPrototype() saves each registered prototype here
    static Image *_prototypes[10];
    static int _nextSlot;
};
Image *Image::_prototypes[];
int Image::_nextSlot;

// Client calls this public static member function when it needs an instance
// of an Image subclass
Image *Image::fineAndClone(imageType type)
{
    for (int i = 0; i < _nextSlot; i++)
        if (_prototypes[i]->returnType() == type)
            return _prototypes[i]->clone();
}


// 子类
class LandSatImage:public Image
{
public:
    imageType returnType() {
        return LSAT;
    }
    void draw() {
        cout << "LandSatImage::draw"<<_id<<endl;
    }
    // When clone() is called, call the one_argument ctor with a dummy(仿造物) arg
    Image *clone() {
        return new LandSatImage(1);
    }
protected:
    // This is only called from clone()
    LandSatImage(int dummy) {
        _id=_count++;
    }
private:
    //Mechanism for initialization an Image subclass-this causes the
    //default ctor to be called, which registers the subclass's prototype
    static LandSatImage _landSatImage;
    //This is only called when the privae static data member is inited
    LandSatImage() {
        addPrototype(this)
    }
    //Nominal "state" per instance mechanism
    int _id;
    static int _count;
};
//Register the subclass's prototype
LandSatImage LandSatImage::_landSatImage;
//Initialize the "state" per instance mechanism
int LandSatImage::_count=1;
```

父类Image有个纯虚函数clone，它自己不知道怎么clone，但要求子类一定要写出clone函数。有一个容器_prototypes放所有的原型，这个容器比较粗糙，一个数组。不要忘记，class里头静态的data，一定要在class本体的外头定义，给内存。

子类里的clone是new自己。原型自己，静态的东西，_landSatImage。静态的自己创建出来的时候，会调用构造函数，这个构造函数是放在私有区域里头，但是自己人调用ok，调用时再把自己放到框架addPrototype。

findAndClone函数当有需要的时候去找并clone。clone做的动作是new自己，会调用构造函数，调用LandSatImage()的话，会把自己再加到框架放原型的空间里。该空间只能放原来那个，不能再做一次。怎么办呢？再写一个构造函数让现在这种情况去调用，因此写出第二个构造函数。不可以放在public，因为现在这些东西不准备让外界去创建，是通过要原型来创建，可以是private、protected，只要跟第一个区别开来。这里用了小技巧加一个参数区别开来，但这个参数根本用不到。



进行到这个地方，已经把基于对象、单一的类，具备了良好的基础，也学习到了面向对象的三个武器，然后也学到了设计模式里的经典做法，从中学习类和类之间如何关联起来。其实，面向对象设计是一个比较长远的路，因为它不像那种传统的编程语言，谁调用谁，线索非常清楚。面向对象不是这样子，所以这里面有很多抽象的思考，需要大家在学习的过程不断去想，为什么这样，该怎么构思，如何去写你的class，不断去写代码，在这个过程里面你才能有真正的进步。
