# uniform initialization

before C++11, programmers, especially novices, could easily become confused by the question of how to initialize a variable or an object. initialization could happen with parentheses, braces, and/or assignment operators.

for this resion, C++11 introduced the concept of uniform initialization, which means that for any initialization, you can use one common syntax. this syntax uses braces, so the following is possible now:

```cpp
int value[] {1,2,3};
vector<int> v {2,3,5,7,11,13,17 };
vector<string> cities {
    "Berlin","New York","London","Braunschweig","Cairo","Cologne"
};
complex<double> c{4.0,3.0};	//equivalent to c(4.0,3.0)
Rect r1 = { 3,7,20,25,&area,&print };
Rect r1(3,,7,20,25);
int ia[6] = { 27,210,12,47,109,83 }
```

其实是利用一个事实：编译器看到 { t1, t2... tn} 便做出一个 initializer_list<T>，它关联至一个 array<T,n>。调用函数 (例如 ctor) 时该 array 内的元素可被编译器分解逐一传给函数。但若函数参数是个 initializer_list<T>，调用者却不能给予数个 T 参数然后以为它们会被自动转为一个 initializer_list<T> 传入。

这形成一个 initializer_lsit<string>，背后有个 array<string, 6>。调用 vectir<string> ctors 时编译器找到了一个 vector<string> ctor 接受 initializer_list<string>。所有容器皆有如此 ctor。

这形成一个 initializer_list<double>，背后有个 array<double, 2>。调用 complex<double> ctor 时该 array 内的2个元素被分解传给 ctor。complex<double> 并无任何 ctor 接受 initialization。



# initializer lists

an initializer list foreces so-called value initialization, which means that even local variables of fundamental data types, which usually have an underfined initial value, are initialized by zero (or nullptr, if it is a pointer):

```cpp
int i;		//i has undefined value
int j{};	//j is initialized by 0
int* p;		//p has undefined value
int* q{};	//q is initialized by nullptr
```

note, however, that narrowing initializations, those that reduce precision or where the supplied value gets modified, are not possible with braces. for example:

```cpp
int x1(5.3);	//OK, but OUCH: x1 becomes 5
int x2 = 5.3;	//OK, but OUCH: x2 becomes 5
int x3{5.0};	//ERROR: narrowing
int x4 = {5.3};	//ERROR: narrowing
char c1{7};		//OK: even though 7 is an int, this is not narrowing
char c2{99999};	//ERROR: narrowing (if 99999 doesn't fit into a char)
std::vector<int> v1 { 1, 2, 4, 5 };		//OK
std::vector<int> v2 { 1, 2.3, 4, 5.6}	//ERROR: narrowing
```



# initializer_list<>

to support the concept of initializer lists for user-defined types, C++11 provides the class template std::initializer_list<>. it can be used to support initializations by a list of values or in any other place where you want to process just a list of values. for example:

```cpp
void print (std::initializer_list<int> vals)
{
    for (auto p = vals.begin(); p != vals.end(); ++p)
}
```



```cpp
class P
{
  public:
    P(int a, int b)		//1, complex<T> 就是这种情况(below A)
    {
        cout << "P(int, int), a=" << a << ", b=" << b << endl;
    }
    P(initializer_list<int> initlist)
    {
        cout << "P(initializer_list<int>), values= ";
        for (auto i : initlist)
            cout << i << ' ';
        cout << endl;
    }
};


P p(77, 5);			//P(int, int), a=77, b=5
P q{77, 5};			//A。P(initializer_list<int>), values= 77 5 
P r{77, 5, 42};		//P(initializer_list<int>), values= 77 5 42
P s={77, 5};		//P(initializer_list<int>), values= 77 5
```

when there are constructors for both 1 a specific number of arguments and 2 an initializer list, the version with the initializer list is preferred. 

without the constructor for the 2 initializer list, the 1 constructor taking two ints would be called to initialize q and s, while the initialization of r would be invalid.

initializer_list objects are automatically constructed as if an array of elements of type T was allocated, with each of the elements in the list being copy-initialized to its corresponding element in the array, using any necessary non-narrowing implicit conversions.

the initializer_list object refers to the elements of this array without containing them: copying an initializer_list object produces another object referring to the same underlying elements, not to new copies of them (reference semantics 含义).

the lifetime of this temporary array is the same as the initializer_list object.

```cpp
template<class _E> class initializer_list
{
public:
    typedef _E			value_type;
    typedef const _E&	reference;
    typedef cosnt _E&	const_reference;
    typedef size_t		size_type;
    typedef const _E*	iterator;
    typedef cosnt _E*	const_iterator;
    
private:
    iterator		_M_array;
    size_type		_M_len;
    
    // the compiler can call a private constructor.
    constexpr initializer_list(const_iterator __a, size_type __l) 
        : _M_array(__a), _M_len(__l) { }
    
public:
    constexpr initializer_list() noexcept : _M_array(0), _M_len(0) { }
    
    // number of elements.
    constexpr size_type size() const noexcept { return _M_len; }
    
    // first elements.
    constexpr const_iterator begin() const noexcept { return _M_array; }
    
    // one past the last element.
    constexpr const_iterator end() const noexcept { return begin() + size(); }  
};
```



initializer_list<>

如今所有容器都接受指定任意数量的值用于构建或复制或 insert() 或 assign(); max() 和 min() 也愿意接受任意参数。

```cpp
// vector

#include <initializer_lsit>

vector(initializer_list<value_type) __l,
	const allocator_type& __a = allocator_type())
        : _Base(__a)
{ _M_range_initialize(__l.begin(), __l.end(), random_access_iterator_tag()); }

vector& operator=(initializer_list<value_type> __l)
{ this->assign(__l.begin(), __l.end());		return *this; }

void insert(iterator __position, initializer_list<value_type> __l)
{ this->insert(__position, __l.begin(), __L.end()); }

void assign(initializer_lsit<value_type> __l)
{ this->assign(__l.begin(), __l.end()); }
```



