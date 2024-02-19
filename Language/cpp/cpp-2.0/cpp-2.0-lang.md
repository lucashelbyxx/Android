# 概述

你应具备的基础：C++ 语法，语意

我们的目标：较全面的认识C++ 2.0新特性，并从实例获得体验。



C++ Standard 之演化

- C++ 98 (1.0)
- C++ 03 (TR1, Technical Report 1)
- C++ 11 (2.0)
- C++ 14



Header files

C++ 2.0 新特性包括语言和标准库两个层面，后者以 header files 形式呈现。

- C++ 标准库的 header files 不带副档名(.h)，例如 #include <vector>
- 新式 C header files 不带副名称 .h，例如 #include <cstdio>
- 旧式 C header files (带有副名称 .h) 仍可用，例如 #include <stdio.h>

```cpp
#include <type_traits>
#include <unordered_set>
#include <forward_list>
#include <array>
#include <tuple>
#include <regex>
#include <thread>

using namespace std;
```

many of the TR1 features that existed in namespace std::tr1 in the previous release (like shared_ptr and regex) are now part of the standard library under the std namespace.



重磅出击

语言：

- Variadic Templates
- move Semantics
- auto
- Range-base for loop
- Initializer list
- Lambdas
- ...

标准库

- type_traits
- Unordered 容器
- forward_list
- array
- tuple
- Con-currency
- RegEx
- ...