# 仿函数 functors

functors 是你最可能写到融入 stl 里头去的，融入辅助你做一些事情。只为算法来服务。

为什么要把加减这些小动作设计成一个函数呢？因为你要把这些动作传到算法去告诉它这个动作，所以你要写成一个函数或一个仿函数传给算法，它拿到这些东西才能做这些动作。

仿函数就是 class 里重载 ()，这样的 class 创建出来的对象就叫做函数对象或者仿函数。

```cpp
//算数类（arithmetic）
template <class T>
    struct plus : public binary_function<T, T, T> {
        T operator()(const T& x, const T& y) const
        { return x + y; }
    };

template <class T>
    struct minus : public bianry_function<T, T, T> {
        T operator()(const T& x, const T& y) const
        { return x - y; }
    };
...
```

```cpp
//逻辑运算类（logical）
template <class T>
    struct logical_and : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x && y; }
    };
...
```

```cpp
//相对关系类（relational）
template <class T>
    struct equal_to : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x == y; }
    };

template <class T>
    struct less : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x < y; }
    };


template <typename Iterator, typename Cmp>
Algorithm(Iterator itr1, Iterator itr2, Cmp comp)
{
    ...
}
```



```cpp
//G2.9, GNU C++ 独有，非标准
template <class T>
    struct identity : public unqry_function<T, T> {
        const T& oeprator()(const T& x) const { return x; }
    };

template <class Pair>
    struct select1st : public nary_function<Pair, typename Pair::first_type> {
        const typename Pair::first_type& operator()(const Pair& x) const
        {
            return x.first;
        }
    };

template <class Pair>
    struct select2nd : public unary_function<Pair, typename Pair::first_type> {
        const typename Pair::second_type& operator()(const Pair& x) const
        {
            return x.second;
        }
    };
```

```cpp
template <class T1, class T2>
    struct pair {
        typedef T1 first_type;
        typedef T2 second_type;
        
        T1 first;
        T2 second;
        pair() : first(T1()), second(T2()) {}
        pair(const T1& a, const T2& b)
            : first(a), second(b) {}
    };
```

```cpp
//...\4.9.2\include\c++\ext
//G4.9
template <class _Tp>
    struct identity
        : public std::_Identity<_Tp> {};

template <class _Pair>
    struct select1st
        : public std::_Select1st<_Pair> {};

template <class _Pair>
    struct select2nd
        : public std::_Select2nd<_Pair> {};
```



```cpp
// using default comparison (operator <):
sort(myvec.begin(), myvec.end());

// using function as comp
sort(myvec.begin(), myvec.end(), myfunc);

// using object as comp
sort(myvec.begin(), myvec.end(), myobj);

// using explicitly default comparison (operator <):
sort(myvec.begin(), myvec.end(), less<int>());	//()创建临时对象

// using another comparison criteria (operator >):
sort(myvec.begin(), myvec.end(), greater<int>());
```

```cpp
// 没有继承，这就没有融入 STL，不能被 adapter 修饰改造
struct myclass {
    bool operator()(int i, int j) { return ( i < j ); }
} myobj;

bool myfunc(int i, int j) { return ( i < j ); }
```

```cpp
// 相对关系类 (relational)
template <class T>
    struct greater : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x > y; }
    };

template <class T>
    struct less : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x < y; }
    };
```



# 仿函数 functors 的可适配（adaptable）条件

```cpp
template <class Arg, class Result>
    struct unary_function {		// unary, 一个操作数。例如对一个东西取它的否定
        typedef Arg argument_type;
        typedef Result result_type;
    };

template <class Arg1, class Arg2, class Result>
    struct binary_funciton {
        typedef Arg1 first_argument_type;
        typedef Arg2 second_argument_type;
        typedef Result result_type;
    };
```

STL 规定每个 adapter function 都应挑选适当者继承之，因为 function adapter 将会提问。例如：

```cpp
template <class T>
    struct less : public binary_function<T, T, bool> {
        bool operator()(const T& x, const T& y) const
        { return x < y; }
    };
```

less 继承 binary_function，没有带来任何额外开销。

于是 less<int> 便有了三个 typedef，分别是：

typedef int first_argument_type;

typedef int second_argument_type;

typedef bool result_type;
