# explicit for ctors taking more than one argument

```cpp
class P
{
public:
    P(int a, int b) {
        cout << "P(int a, int b) \n";
    }
    
    P(initializer_list<int>) {
        cout << "P(initializer_list<int>) \n"
    }
    
    explicit P(int a, int b, int c) {
        cout << "explicit P(int a, int b, int c) \n"
    }
};


//explicit cotr with multi-arguments
P p1 (77, 5);			//P(int a, int b)
P p2 {77, 5};			//P(initializer_list<init>)
P p3 {77, 5, 43};		//P(initializer_list<init>)
P p4 = {77, 5};			//P(initializer_list<init>)
//! P p5 = {77, 5, 43}	//[Error] converting to 'P' from initializer list would use explicit constructor 
P p6 (77, 5, 43);		//explicit P(int a, int b, int c)

fp( {47, 11} );			//P(initializer_list<int>)
//! fp( {47,11,3})		//[Error] converting to 'P' from initializer list would use explicit constructor
fp( P{47,11} );			//P(initializer_list<init>)
fp( P{47,11,3} );		//P(initializer_list<init>)

P p11 {77,5,42,500};	//P(initializer_list<init>)
P p12 = {77,5,42,500};	//P(initializer_list<init>)
P p13 {10};				//P(initializer_list<init>)
```



# explicit for ctors taking one argument

```cpp
struct Complex
{
    int real, imag;
    
    Complex(int re, int im=0) : real(re), imag(im)
    { }
    
    Complex operator+(const Complex& x)
    { return Complex((real + x.real), (imag + x.imag)); }
};


Complex c1(12,5);
Complex c2 = c1 + 5;



struct Complex
{
    int real, imag;
    
    explicit Complex(int re, int im=0) : real(re), imag(im) 
    { }
    
    Complex operator+(const Complex& x)
    { return Complex((real + x.real), (imag + x.imag)); }
};


Complex c1(12,5);
Complex c2 = c1 + 5;
//[Error] no match for 'operator+' (operand types are 'Complex' and 'int')
```



# range-based for statement

```cpp
//1
for ( decl : coll ) {
    statement
}


for (int i : {2,3,4,,6,7} ) {
    cout << i << endl;
}


vector<double> vec;
...
for ( auto elem : vec) {
    cout << elem << endl;
}

for ( auto& elem : vec) {
    elem *= 3;
}


//1 equal this
{
    for (auto _pos=coll.begin(), _end=coll.end(); _pos!=_end; ++_pos) {
        decl = *_pos;
        statement
    }
}
//or
{
    for (auto _pos=begin(coll), _end=end(coll); _pos!=_end; ++_pos) {
        decl = *_pos;	//both begin() and end() are global
        statement
    }
}
```



```cpp
tempalte <typename T> void printElements(const T& coll)
{
    for (const auto& elem : coll) {	//2
        cout << elem << endl;
    }
}

//2 equal to this
{
    for (auto_pos=coll.begin(); _pos!=coll.end(); ++_pos) {
        const auto& elem = * _pos;
        cout << elem <<endl;
    }
}
```



no explicit type conversions are possible when elements are initialized as decl inside the for loop. thus, the following does not complie:

```cpp
class C
{
public:
    explicit C(const string& s);	//explicit(!) type conversion from strings
    ...
};

vector<string> vs;
for (const C& elem : vs) {	//ERROR, no conversion form string to C defined
    cout << elem << endl;
}
```



# =default, =delete

如果你自行定义了一个 ctor，那么编译器就不会再给你一个 default ctor。

如果你强制加上 =default，就可以重新获得并使用 default ctor。

```cpp
class Zoo
{
public:
    Zoo(int i1, int i2) : d1(i1), d2(i2) { }
    Zoo(const Zoo&) = delete;
    Zoo(Zoo&&) = default;
    Zoo& operator=(const Zoo&) = default;
    Zoo& operator=(const Zoo&&) = delete;
    virtual ~Zoo() { }
private:
    int d1, d2;
};
```



```cpp
class Foo
{
public:
    Foo(int i) : _i(i) { }
    Foo() = befault;	//11: 于是和上一个并存(ctor 可以多个并存)
    
    Foo(const Foo& x) : _i(x._i) { }
    //! Foo(const Foo&) = default;	//[Error] 'Foo::Foo(const Foo&)' cannot be overloaded
    //22: ! Foo(const Foo&) = delete;	//[Error] 'Foo::Foo(const Foo&)' cannot be overloaded
	
    Foo& operator=(const Foo& x) {_i=x._i; return *this; }
    //! Foo& operator=(const Foo& x) = default;	//[Error] 'Foo&Foo::operator=(const Foo&)' cannot be overloaded
    //33: ! Foo& operator=(const Foo& x) = delete;	//[Error] 'Foo&Foo::operator=(const Foo&)' cannot be overloaded
    
    //!void func1() = delete;	//[Error] 'void Foo::func1()' cannot be defaulted
    void func2() = delete;		//ok
    //=default; 用于 Big-Five 之外是何意义？-》无意义，编译报错。
    //=delete; 可用于任何函数身上。( =0 只能于 virtual 函数)
    
    
    //! ~Foo() = delete;	//这会造成使用Foo object时出错-》[Error] use of eleted function 'Foo::~Foo()'
    ~Foo() = default;
    
private:
    int _i;
};


Foo f1(5);
Foo f2;			//11: 如果没写出 =default 版本-》[Error] no matching function for call to 'Foo::Foo()'
Foo f3(f1);		//22: 如果 copy cotr = delete; [Error] use of deleted function 'Foo::Foo(const Foo&)'
f3 = f2;		//33: 如果 copy assign = delete; [Error] use of deleted function 'Foo&Foo::oeprator=(const Foo&)' 
```



# know what fnctions C++ silently writes and calls

什么时候 empty class 不再是 empty 呢？当C++处理过它之后。如果你自己没声明，编译器就会为它声明一个 copy ctor、一个 copy assignment operator 和一个 dtor（都是所谓 synthesized version）。如果你没有声明任何 ctor，编译器也会为你声明一个车 default ctor。所有这些函式都是 public 且 inline。

```cpp
class Empty {};

{
    Empty e1;
	Empty e2(e1);
    e2 = e1;
}

//只有当这些函式被调用，它们才会被编译器合成。以上造成下面每个函式被编译器合成
class Empty {};

class Empty {
public:
    Empty() {...}
    Empty(const Empty& rhs) {...}
    ~Empty() {...}
    
    Empty& operator=(const Empty& rhs) {...}
};
```

这些函式做了什么呢？default ctor 和 dotr 主要是给编译器一个地方用来放置藏身幕后的 code，像是唤起 base classes 以及 non-static members 的 ctors 和 dtors。

编译器产出的 dtor 是 non-virtual，除非这个 class 的 base class 本身宣告有 virtual dtor。

至于 copy ctor 和 copy assignment operator，编译器合成版只是单纯将 source object 的每一个 non-static data members 拷贝到 destination object。



# complex<T> 有所谓 Big-Three 吗？

```cpp
template<typename _Tp>
struct complex
{
    //value typedef
    typedef _Tp value_type;
    
    //default constructor. first parameter is x, second parameter is y.
    //unspecified parameters default to 0.
    _GLIBCXX_CONSTEXPR complex(const _Tp& __r = _Tp(), const _Tp& __i = _Tp())
        : _M_eral(__r), _M_imag(__i) { }
    
    //lets the compiler synthesize the copy constructor
    //complex (const complex<_Tp>&);
    //copy constructor.
    template<typename _Up>
    	_GLIBCXX_CONSTEXPR complex(cosnt complex<_Up>& __z)
    : _M_real(__z.real()), _M_imag(__z.imag()) { }	//其实多此一举
    
    //没有 operator=(const complex<...>&)
    //没有 ~complex()
    
    private:
    	_Tp _M_real;
    	_Tp _M_imag;
};
```

string 有所谓 Big-Three 吗？当然有，而且有 Big-Five。



# no-copy and private-copy

```cpp
struct NoCopy {
    NoCopy() = default;		//use the synthesized(合成的) default constructor
    NoCopy(cosnt NoCopy&) = delete;		//no copy
    NoCopy &operator=(const NoCopy&) = delete;	//no assignment
    ~NoCopy() = defualt;	//use the synthesized destructor
    //other members
};

//= delete 告诉编译器不要定义它。必须出现在声明式。
//适用于任何成员函数。但若用于 dtor 后果自负。

struct NoDtor {
    NoDtor() = default;		//use the synthesized default constructor
    ~NoDtor() = delete;		//we can't destroy objects of type NoDtor
};
NoDtor nd;					//error: NoDtor destructor is deleted
NoDtor *p = new NoDtor();	//ok: but we can't delete p
delete p;					//error: NoDtor destructor is deleted


class PrivateCopy {
private:
    //no access specifier; following members are private by default;
    //copy control is private and so is inaccessible to ordinary user code
    PrivateCopy(cosnt PrivateCopy&);
    PrivateCopy &operator=(cosnt PrivateCopy&);
    //other members
public:
    PrivateCopy() = default;	//use the synthesized default constructor
    ~PrivateCopy();	//users can define objects of this type but not copy them
};
//此 class 不允许被 ordinary user code copy，但仍可被 friends 和 members copy。
//若要完全禁止，不但必须把 copy controls 放到 private 内且不可定义之。
```

...\boost\noncopyable.hpp:

```cpp
// Foo -▷ noncopyable

namespace boost {
//private copy constructor and copy assignment ensure classes derived from
//class noncopyable cannot be copied.
//contributed by Dave Abrahams
    
namespace noncopyalbe_	//protection from unintended ADL
{
    class noncopyable
    {
	protected:
        noncopyable() { }
        ~noncopyable() { }
	private: //emphasize the following members are private
        noncopyable(const noncopyable&);
        const noncopyalbe& operator=(const noncopyable&);
    };
}
    
typedef noncopyalbe_::noncopyable noncopyable;
}	//namespace boost
```

