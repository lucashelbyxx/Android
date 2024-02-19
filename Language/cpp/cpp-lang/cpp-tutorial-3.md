导读

C++程序设计 (Ⅱ) 兼谈对象模型，对象模型主要是更底层的一些东西。

切在浮沙筑高台

目标：

- 在先前基础课程所培养的正规、大器的编程素养上，继续探讨更多技术。
- 泛型编程 (Generic Programming) 和面向对象编程，(Objected-Oriented Programming) 虽然分属不同思维，但它们正是C++的技术主线，所以本课程也讨论template(模板)。
- 深入探索面向对象之继承关系(inheritance)所形成的对象模型(Object Model)，包括隐藏于底层的this指针，vptr(虚指针)，vtbl(虚表)，virtual mechanism(虚机制)，以及虚函数(virtual functions)造成的polymorphism(多态)效果。



conversion function，转换函数

谁把谁转换成什么呢？你现在设计一个class A，它所做出来的对象可不可以被转为另一种类型，这就是转换。或者另外一种类型可不可以转换为A。转出去、转进来，两个方向我们都要谈。

```cpp
class Fraction
{
public:
    Fraction(int num, int den=1) : m_numerator(num), m_denominator(den) { }
    operator double() const {	//fraction可以被转为double
        return (double) (m_numberator / m_denominator);
    }
private:
    int m_numerator;	//分子
    int m_denominator;	//分母
};


Fraction f(3,5);
double d=4+f;	//调用operator double()将 f 转为 0.6
```

现在谈的是转出去。分数，分子除以分母就是一个double。作为一个设计者，认为我设计一个fraction，它可以被当成一个double。后面任何场景如果编译器把它当做一个double，我会认为很合理。operator double() const，编译器任何时候需要把fraction转为double的时候，都可以来调用这个函数。return type，转换完应该是double，但C++说你不用写，后面写了，前面写错了就矛盾了。

我有一个double，它是4+f。编译器编译到这个地方的时候，他手上有几个筹码，他要去看看你设计了哪些函数，这些函数能不能被编译器在这一行让它通过。编译器正在找，找你有没有写一个全局函数operator +这个动作。如果你有写，而且第一个参数是整数或者浮点数，而第二个参数是fraction。如果他能找到这样一个函数，这一行就能通过。像现在这个例子，没有设计这样一个函数，于是这一条路就走不通了。编译器设法再看看能不能把 f 转为double，那就是double加double咯，这一行可以通过呀。于是他就去找找看你有没有设计这样一个函数。

你的任何一个class都可以设计好几个转换函数，只要你认为合理。

<br>

non-explicit one argument constructor

explicit，明确的

one argument，只要一个实参就够了。当然两个也可以。编译器看到一个语句，会想办法去找能不能让这个语句通过。现在看看+，有没有这个设计呢？Fraction里头有个加的动作，但是加这个动作右边是一个分数。现在调用的地方是分数加4，这个形式和我的设计不同。于是编译器去看看能不能把4转为fraction。如果4能转fraction，就是分数加分数，符合我的设计咯。由于有 non-explicit ctor 将4转为1分之4，于是就能相加了。

non-explicit one argument ctr 它的作用是可以把别的东西转换为这种，而前面的是把这种转换为别的东西，方向刚好相反。

```cpp
class Fraction
{
public:
    Fraction(int num, int den=1)
        : m_numerator(num), m_denominator(den) { }
    Fraction operator+(const Fraction& f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int denomiator;
}


Fraction f(3,5);
Fraction d2=f+4;	//调用non-explicit ctor将4转换为(4,1)
					//然后调用operator+
```

<br>

conversion function vs. non-explicit one argument ctor

```cpp
class Fraction
{
public:
    Fraction(int num, int den=1)	//1
        : m_numerator(num), m_denominator(den) { }
    operator double() const {	//2
        return (double) (m_numerator / m_denominator);
    }
    Fraction operator+(const Fraction& f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int denomiator;
};


Fraction f(3,5);
Fraction d2=f+4;	//error, ambiguous
```

前面两个并存，编译器不知道该怎么办了。编译器要把4转为fraction吗？1就可以把4转为fraction，fraction相加也有，加完之后是fraction，这条路走得通。但是2把fraction转为double，五分之三转为0.6，加完为4.6，要转为d2，d2是fraction。这么多条路里面，编译器发现有两条可行。只要多于一条路可行，编译器就不知道走哪条了。也没有谁比较好，谁比较坏这种判断。在这个例子，并存的话，ambiguous，歧义，报错。所以要不要1、2，你自己要去控制，是否歧义取决于你做的事情、使用者怎么使用这个class。

<br>

explicit-one-argument ctor

explicit，明白的、明确的，告诉编译器说你不要暗度陈仓了，不要帮我自动做事情了。我既然设计是构造函数，就以构造函数的形式你需要的时候再来调用它。

```cpp
class Fraction
{
public:
    explicit Fraction(int num, int den=1)	
        : m_numerator(num), m_denominator(den) { }
    operator double() const {	
        return (double) (m_numerator / m_denominator);
    }
    Fraction operator+(const Fraction& f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int denomiator;
};


Fraction f(3,5);
Fraction d2=f+4;	//error, conversion from double to fraction requested
```

加法设计在这里，左右两边需要是fraction。现在右边4转不过去，失败了。

<br>

conversion function, 转换函数

```cpp
//标准库里头的例子
template<class Alloc>
class vector<bool, Alloc>	//模板的偏特化
{
public:
    typedef _bit_reference reference;
protected:
    reference operator[] (size_type n) {
        return * (begin() + difference_type(n));
    }
...
};


struct _bit_reference {
    unsigned int* p;
    unsigned int mask;
    ...
public:
    operator bool() const { return !(!(*p & mask)); }	//转换函数
}
```

这里头设计了一个函数对中括号重载。C++操作符重载非常重要，应用很多。reference operator[] 要注意传回的是什么。vetor这个容器意思是里面放的每个元素都是固定值，都是0或1，真或假。所以要取出某个位置的真或假，传出来应该是固定值，但这里设计传出来的是reference，另外一个class。这个在设计手法上叫proxy，代理，传回来的reference来代表本来真正传回来的东西。

这个reference自然应该有一个转换函数转为boolean，非常合理。你拿A代表B，A就应该有个转换函数转为B。

<br>

pointer-like classes, 关于智能指针

接下来要谈的是，一个C++的class做出来可能会像两种东西。一种就是这种class做出来之后，所产生的出来的对象，像一个指针。下一个主题是像一个函数。

为什么要把一个class设计出来像一个指针呢？因为你想比指针再多做一些事情，所以通常这样子做出来的东西我们又把它叫做智能指针。C++2.0之前有个智能指针叫 auto pointer，2.0之后有好几个智能指针，包括现在看到的 shared pointer。

```cpp
template<class T>
class shared_ptr
{
public:
    T& operator*() const	//3
    { return *px; }
    
    T* operator->() const	//4
    { return px; }
    
    shared_ptr(T* p) : px(p) { }
    
private:
    T* px;	//pointer to T，指向谁无所谓，T是个模板
    long* pn;
...
};


struct Foo
{
    ...
	void method(void) { ... }
};


shared_ptr<Foo> sp(new Foo);

Foo f(*sp);	//1

sp->method();	//2
//等同 px->method();	5
```

它们的长相都类似。智能指针里头一定会有一个真正的指针。现在要探讨的不是智能在哪里，讨论它的语法。

pointer-like 要强调的是指针所允许的动作，它这个class做出来的对象也都要能够允许。什么样的操作符用在指针上呢？以现在的shared_ptr来讲，一个是星号，一个是箭头符号。这一类的指针它的这两个写法是固定的。

我们先看它的用法，假设现在写了一个class叫Foo。我要把new Foo()这种天然的指针包装到这个聪明的指针里去。聪明的指针一般都有shared_ptr(T* p):px(p)这个构造函数，接受天然指针。所以new出来的指针当做初值塞进去。

使用者接下来的动作，面对sp，就要把它当成一般的指针一样。当使用者1、2这样用的时候，我们作为设计者要这么呼应，这样对不对。sp现在就代表原来的那根指针，*作用上去相当于解参考(dereference)拿那个东西。怎么去解释这个行为呢？你要拿，好啊，那我就把我记录的这个指针所指的东西传给你。1是使用者想要的东西，3是智能指针对他的呼应，这是合理的。

->符号，可能没办法解释这个行为，也许会写，看多就会写。如果使用者2这样用的话是什么意思？通过sp对象调用这个函数，操作符作用在sp上面，智能指针回应把里面的指针传出来。所以把2换成5应该是可以的，因为操作符作用上去返回的是px。2转到5这个“->”符号已经被用掉了，就是它作用上去才得到的px，那怎么5还有一个箭头符号呢？箭头符号有个特殊行为，它作用下去的结果箭头符号会继续作用下去，语法规定。

<br>

pointer-like classes, 关于迭代器

这是pointer-like的第二大分类，迭代器。使用库的时候里面有一个重要的东西就是容器，容器本身就一定带着迭代器。只谈迭代器作为另外一种智能指针，我比较在意它的操作符重载。迭代器干什么呢？迭代器它就是要代表容器里面的一个元素，它要去指向一个元素。因此，它也像一个指针，也可以说它是一个智能指针。

迭代器不但要处理*、->，还要处理++、--、==、！=。迭代器之外的智能指针可能不用写++、--，因为它没有所谓这些动作。++--只适用在指针移动，++就是向前移、--向后移。迭代器就是用来遍历容器，所以它需要写出++--。

```cpp
template<class T, class Ref, class Ptr>
struct _list_iterator {
    typedef _list_iterator<T, Ref, Ptr> self;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef _list_node<T>* link_type;
    link_type node;
    bool operator==(const self& x) const { return node == x.node; }
    ...
	reference operator*() const { return (*node).data; }	//1
    pointer operator->() const { return &(operator*()); }	//2
    ...
	self& operator--() { node = (link_type) ((*node).prev); return *this; }
    self operator--(int) { self tmp = *this; --*this; return tmp; }
};


template <class T>
struct _list_node {
    void *prev;
    void *next;
    T data;
};


list<Foo>::iterator ite;
...
*ite;	//获得一个 Foo object
ite->method();
//意思是调用 Foo::method()
//相当于(*ite).method();
//相当于(&(*ite))->method;	todo?
```

对迭代器做*解参考，意思就是拿这个data。1就呼应它的需求，从node这个指针取得它的东西，deference，取得的就是一个_list_node，这一块里面的data栏目。

<br>

function-like classes, 所谓仿函数

设计一个class，让其对象的行为像一个函数。为什么要让类的对象像指针或像函数？

如何像一个函数呢？函数有一个函数名称，要用一个小括号作用上去。事实上，小括号操作符叫做function call函数调用操作符。所以任何一个东西能够接受小括号这个操作符，我们就把这个东西叫做一个函数或者叫做像函数的东西。

```cpp
template <class T>
struct identity {	//1
    const T& operator() (const T& x) const { return x; }
};


template <class Pair>
struct select1st {	//2
    const typename Pair::first_type& 
    operator() (const Pair& x) const 
    { return x.first; }
};


template <class Pair>
struct select2nd {	//3
    const typename Pair::second_type&
	operator() (const Pair& x) const
    { return x.second; }
};


template <class T1, class T2>
struct pair {
    T1 first;
    T2 second;
    pair() : first(T1()), second(T2()) {}
    pair(const T1& a, const T2& b) : first(a), second(b) {}
...
};
```

1有对小括号()的重载，我们就说它像函数，它所做出来的对象可以接受小括号。它是模板只是更弹性一些而已，接受的类型更多。

```cpp
//标准库中的仿函数的奇特模样
template <class T>
struct identity : public unary_function<T, T> {	//1
    const T& operator() (const T& x) const { return x; }
};


template <class Pair>
struct select1st : public unary_function<Pair, typename Pair::first_type> {	//2
    const typename Pair::first_type& 
    operator() (const Pair& x) const 
    { return x.first; }
};


template <class Pair>
struct select2nd : public unary_function<Pair, typename Pair::second_type>  {	//3
    const typename Pair::second_type&
	operator() (const Pair& x) const
    { return x.second; }
};


//标准库中，仿函数所使用的的奇特的 base classes
//unary一个操作数，binary两个操作数
template <class Arg, class Result>
struct unary_funciton {
    typedef Arg argument_type;
    typedef Result result_type;
};

template <class Arg1, class Arg2, class Result>
struct binary_function {
  typedef Arg1 first_argument_type;
  typedef Arg2 second_argument_type;
  typedef Result result_type;
};
```

标准库里面有很多反函数，这些反函数都是一些小小的class，里面有重载小括号()，这样它们就成为反函数了。这些反函数都有继承一些奇怪的父类，这些父类大小是0，函数没有，只有一些typedef。继承这些父类有什么用，在标准库讲。

typedef与#define两者的区别如下：

- \#define 进行简单的进行字符串替换。 #define 宏定义可以使用 #ifdef、#ifndef 等来进行逻辑判断，还可以使用 #undef 来取消定义。
- typedef 是为一个类型起新名字。typedef 符合（C语言）范围规则，使用 typedef 定义的变量类型，其作用范围限制在所定义的函数或者文件内（取决于此变量定义的位置），而宏定义则没有这种特性。

通常，使用 typedef 要比使用 #define 要好，特别是在有指针的场合里。

<br>

namespace经验谈

namespace就是把一些东西区隔开来。因为现在设计软件一般是团队作业，由于部门A写的function和部门B写的function函数名冲突了或者类的名称冲突了。所以namespace的意思是你再取一个名字，再把你的东西包起来。放在不同的命名空间，彼此不冲突。

```cpp
using namespace std;
//-----------------
#include <iostream>
#include <memory> //shared_ptr
namespace jj01
{
void test_member_template()
{ ... }
} //namespace
//------------------
#include <iostream>
#include <list>
namespace jj02
{
template<typename T>
using Lst = list<T, allocator<T>>;
void test_template_template_param()
{ ... }
} //namespace
//------------------


int main(int argc, char** argv)
{
    jj01::test_member_template();
    jj02::test_template_template_param();
}
```

<br>

class template, 类模板

class template，就是你在设计一个class的时候，你认为可以把哪些类型抽出来，允许使用者任意指定，你就把它抽出来。比如复数的实部虚部，你认为它可以是整数、浮点数、double、long。这样的话你会想把实部虚部的类型抽到最外头，告诉编译器现在类型叫T，T是将来用的时候再说。

```cpp
template<typename T>
class complex
{
public:
    complex (T r = 0, T i = 0) : re (r), im (i) { }
    complex& operator += (const complex&);
    T real () const { return re; }
    T imag () const { return im; }
private:
    T re, im;
    
    friend complex& _doapl (complex*, const complex&);
};


{
    complex<double> c1(2.5,1.5);
    complex<int> c2(2,6);
    ...
}
```

<br>

function template, 函数模板

你设计一个函数，现在要比大小。a和b来比，用 < 来比。至于它们是整数、浮点数、甚至字符串，都没有关系。这样的话你就把这个函数设计为一个function template。

函数模板在使用的时候不用指明它的type。因为函数模板使用的时候一定去调用它，调用的时候一定会放参数，于是编译器会对 function template 进行实参推导 (argument dedution)。编译器会根据你调用的参数推出 T 是什么。如果推出来是两个stone，那么stone能不能比大小呢？编译器会再去找有没有写 < 小于这个动作。

编译器在编译这段模板的时候，它实际是不知道日后真正的a、b的类型是什么？模板的编译，编译出来也只是个半成品，不能保证后头一定会成功，使用的时候会再编译一次。

```cpp
class stone
{
public:
    stone(int w, int h, int we) : _w(w), _h(h), _weight(we) { }
    bool operator< {const stone& rhs} const { return _weight < rhs._weight; }
private:
    int _w, _h, _weight;
};


template <class T>
inline
const T& min(const T& a, const T& b)
{
    return b < a ? b : a;
}


//编译器会对 function template 进行实参推导 (argument deduction)
//实参推导的结果，T为stone，于是调用stone::operator<
stone r1(2,3), r2(3,3), r3;
r3 = min(r1,r2);
```

<br>

member template

```cpp
template <class T1, class T2>
struct pair {
    typedef T1 first_type;
    typedef T2 second_type;
    
    T1 first;
    T2 second;
    
    pair()
        : fist(T1()), second(T2()) {}
    pair(const T1& a, const T2& b)
        : first(a), second(b) {}
    
    template <class U1, class U2>	//1
	pair(const pair<U1, U2>& p)
        : first(p.first), second(p.second) {}
};
```

1是模板里面的一个member，它自己本身又是一个template，就把这段叫做member template。外头的模板是允许变化的东西，变化 T1、T2。而在这种变化下，T1、T2如果被确定之后，里头1这段又可以变化，变化 U1、U2。

往往很多模板，它的构造函数就会设计成这种东西，成员模板。让构造函数更有弹性。

```cpp
//Derived1鲫鱼 -▷ Base1鱼类
//Derived2麻雀 -▷ Base2鸟类
class Base1 {};
class Derived1:public Base1 {};

class Base2 {};
class Derived2:public Base2 {};


pair<Derived1, Derived2> p;	//把鲫鱼和麻雀做成一对
pair<Base1, Base2> p2(p);	//把鱼类和鸟类做成一对

pair<Base1, Base2> p2(pair<Derived1, Derived2>());
//把一个有鲫鱼和麻雀构成的pair，当成初值放进(拷贝到)一个由鱼类和鸟类构成的pair中，可以吗？可以
//反之，可以吗？不行


/*由于要满足这种可以不可以，所以设计pair的人说我允许你放任意的T1、T2，并且在构造我的时候，
我还允许你放任意的U1、U2。不过这个U1、U2必须满足2赋值动作。2什么意思？把p，即现在的初值，
把初值的头尾放进来当成我本身的头尾。所以初值是鲫鱼和麻雀，可以转为对应的鱼类和鸟类。
*/
template <class T1, class T2>
struct pair {
    typedef T1 first_type;
    typedef T2 second_type;
    
    T1 first;
    T2 second;
    
    pair()
        : fist(T1()), second(T2()) {}
    pair(const T1& a, const T2& b)
        : first(a), second(b) {}
    
    template <class U1, class U2>	//1
	pair(const pair<U1, U2>& p)
        : first(p.first), second(p.second) {}	//2
};
```

如果我有一个聪明的指针指向鱼类，现在我要给它设初值，给它的指针指向鲫鱼，可以吗？new一个鲫鱼，但是我的指针是指向鱼类，这是可以的，叫up-cast。这是一定合理的。你有一根指针指向动物，将来让它指向一只猪可以吗？可以。既然指针可以，那智能指针也可以。要可以，智能指针要写出1这种member template。

```cpp
template<typename _Tp>
class shared_ptr:public _shared_ptr<_Tp>
{
...
    template<typename _Tp1>		//1
    explicit shared_ptr(_Tp1* _p)
    :_shared_ptr<_Tp>(_p) {}
...
};


Base* ptr = new Derived1;	//up-cast
shared_ptr<Base1> sptr(new Derived1);	//模拟 up-cast
```

<br>

specialization，模板特化

什么叫泛化，泛化就是模板，模板说我有一个类型，要用的时候再指定我就可以了。

特化的意思是你作为一个设计者你很可能会面对某些独特的类型要做特殊的设计。x和y两点之间会画一条直线，那直线xy做出的坐标点都是实数。你为这件事情写一个class，科学家告诉你可以针对如果这些点是整数，你可以有个非常快的算法。所以你非常有可能在设计了一个模板后，想锁定、绑定模板泛化部分，把它更局部特征化。

```cpp
template <class Key>	//1，泛化
struct hash { };

//这些是特化
template<>	//key被绑定了，就没有了
struct hash<char> {
    size_t operator() (char x) const { return x; }
};

template<>
struct hash<int> {
    size_t operator() (int x) const { return x; }
};

template<>
struct hash<long> {	//2
    size_t operator() (long x) const { return x; }
};


// 临时对象，编译器回去找，先用特化
cout << hash<long>() (1000);
```

我们来看例子看语法。1就是一个一般的泛化，它接受一个key。key是一个符号，什么都可以。指定任意类型都会跑到1，用上这段代码。如果你指定的是char、int、long，就会用上面三段代码。它们都对小括号()做了重载。

泛化又叫全泛化，对应的就有偏特化，partial specialization。

<br>

局部特化可以从两个角度去讲，一个就是个数上的偏，一个就是范围上的偏。

partial specialization, 模板偏特化，个数的偏

1模板有两个模板参数，你想绑定其中一个。vector可以允许你指定元素类型，以及指定一个分配器(有默认值)。在设计的时候，你会认为如果这个元素是固定值的话，好像我会有一个特殊的设计。为什么呢？因为c++里面最小单元的元素类型是char，8位形成的字节。如果你让一个字节去代表一个固定值，是不是太浪费了，浪费8倍。如果客户说他要放一个真假值，0或1，我觉得我可以单独为他设计一个，而不要用泛化来搞。

因此，我要告诉编译器，这里有两个模板参数，现在我只剩一个了，因为另外一个被我绑定了，T写成bool。有5个模板参数，我绑定了135，留下24，这样不行，一定要从左边到右边。

```cpp
template<typename T, typename Alloc=...>	//1
class vector
{
    ...
};

template<typename Alloc=...>
class vector<bool, Alloc>
{
...
```

partial specialization，模板偏特化，范围的偏

如果我设计接受T，T可以是任意类型。如果我把范围缩小一点，它是一个指针指向任意类型，这样就缩小了。如果你有这种运用的话，C++支持你的想法。

先写1，支持任意的T，然后再写一个特化的版本2。你告诉编译器说，如果使用者用的是指针，就用2这套代码，不是就用1。

```cpp
template <typename T>	//1
class C
{
    ...
};

template <typename T>	//2
class C<T*>
{
    ...
};


C<string> obj1;		//使用1
C<string*> obj2;	//使用2
```

<br>

template template parameter, 模板模板参数

现在设计了一个template，里头第二个模板参数它本身又是一个template。只有在尖括号<>里面，typename和class共通。Container是拿第一个模板参数做参数。设计者允许使用者2这么用，这个用法就是传入一个容器，并传入容器的元素类型，所以使用者就变得很有弹性了。3传入的链表，链表的元素是string，可以指定任意的容器、任意的元素类型，进去后被组合起来。要让使用者这么有弹性的话，这里第二参数就必须是模板模板参数。传入的list就是一个模板，它自己都还未定，需要指定元素类型，后面还有分配器，这些都没有指定。所以list本身就是一个模板。

```cpp
//1
template<typename T, template <typename T> class Container>
class XCls
{
private:
    Container<T> c;
public:
    ...
};


template<typename T>
using Lst = list<T, allocator<T>>;

//2
XCls<string, list> mylst1;	//3，XXX，不能这样用，语法规定，容器需要好几个模板参数
XCls<string, Lst> mylst2;
```

SmartPtr只接受一个模板参数。这里1就是使用传进来的智能指针，并且以第一参数作为智能指针里面的参数。

```cpp
template<typename T, template <typename T> class SmartPtr>	//1
class XCls
{
private:
    SmartPtr<T> sp;
public:
    XCls() : sp(new T) { }
};


XCls<string, shared_ptr> p1;
XCls<double, unique_ptr> p2;	//XXX，unique_ptr的特性
XCls<int, weak_ptr> p3;		//XXX，weak_ptr的特性
XCls<long, auto_ptr> p4;
```

这不是template template parameter

它接收两个模板参数，其中第二个有默认值。代码里以Sequence为类型创建一个对象出来。

```cpp
template <class T, class Sequence = deque<T>>
class stack {
    friend bool operator== <> (const stack&, const stack&);
    friend bool operator< <> (const stack&, const stack&);
    
protected:
    Sequence c;	//底层容器
...
};


stack<int> s1;
stack<int, list<int>> s2;	//这样写已经没有任何模糊地带了，写死了，不是模板了
```
