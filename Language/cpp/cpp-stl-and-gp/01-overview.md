**C++ 标准库体系结构与内核分析**

为什么会用到体系结构这个字眼呢？因为 C++ 标准库它不像 C 标准库一个一个单一的函数，彼此之间几乎没有什么关联。但是 C++ 标准库分为6个part，6个部件彼此之间有紧密的、重要的关系存在。所以我们要把这个体系结构弄清楚之后，我们才能够用好 C++ 标准库，甚至来强化它。

内核分析这个字眼？因为不但谈到怎么用这个标准库，还会谈到标准库里面结构是怎么写出来的，Source Code。



所谓 Generic Programming (GP，泛型编程)，就是使用 template (模板) 为主要工具来编写程序。第二讲开宗明义阐述了 GP 与 OOP (Object Oriented Programming; 面向对象编程) 的根本差异，并谈及 templates 的意义和运用。

制作本讲义时，我使用的题目是《C++标准库-体系结构与内核分析》；由于本课程之最精髓就是根据源代码分析 C++ STL 之体系结构，而 STL 正是泛型编程 (GP) 最成功的作品，所以本课程事实上就是以 STL 为目标深层次地探讨泛型编程。



你应具备的基础：C++ 基本语法 (包括如何正确使用模板，templates)

C++ 标准库主要是使用 generic programming 也就是用模板这个工具写出来的，它没有太多的面向对象的成分、观念、技巧。在早期没有太多的继承，现在比较多继承，也没有那些烦人的虚函数 virtual function。



我们的目标

- level 0：使用 C++ 标准库
- level 1：认识 C++ 标准库 (胸中自有丘壑)
- level 2：良好使用 C++ 标准库
- level 3：扩充 C++ 标准库

使用 C++ 标准库说容易也容易。困难的是你心里头要对于你使用的东西的结构有一个相当的了解，你才能够知其然也知其所以然。

深入认识 C++ 标准库，胸中自有丘壑。你在使用这些部件的时候，你非常清楚它在内存里面长什么样子，它是怎么扩展的。当你放一百万个元素进去的时候，它是形成一个怎么样的结构。你有了这个图之后，你才能够决定、判断该用那哪个部件、容器、算法能达到最好的效果。

当你对 C++ 标准库有一个相当的认识之后，你才能够良好的运用，必须对这整个体系结构有相当的了解。

扩充 C++ 标准库，做到这步的必要性不多。基本上你能够非常良好的运用 C++ 标准库已经是猴赛雷了。



C++ Standard Library vs. Standard Template Library

C++ 标准库和 STL，标准模板库是同一个东西吗？C++ 标准库，编译器会赋给你很多头文件，以这种形式给你这种标准库，而不是编译好的东西，所以你完全可以看到整个 source code。这里大致进行区分，你可以说标准库里面百分之七八十都是 STL，这个 STL 分为六大部件，所以说标准库大于 STL。那 STL 六大部件之外，还有些零零碎碎的小东西也非常重要，也很好用。六大部件就是 STL，其他零零碎碎加上 STL 就是标准库。



标准库以 header files 形式呈现

- C++ 标准库的 header files 不带副档名 (.h)，例如 #include <vector>
- 新式 C header files 不带副档名 .h，例如 #include <cstdio>
- 旧式 C header files (带有副档名 .h) 仍然可用，例如 #include <stdio.h>
- 新式 headers 内的组件封装于 namespace "std"
  - using namespace std; 
  - using std::cout; (for example)
- 旧式 headers 内的组件不封装于 namespace "std"

```cpp
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;
```

