# 容器 set，multiset

set/multiset 以 rb_tree 为底层结构，因此有元素自动排序特性。排序的依据是key，而set/multiset元素的value和key合一：value就是key。

set/multiset提供遍历操作及iterators。按正常规则 (++ite) 遍历，便能获得排序状态 (sorted)。

我们无法使用set/multiset的iterators改变元素值 (因为key有其严谨排列规则)。set/multiset的iterator是其底部的RB tree的const-iterator，就是为了禁止user对元素赋值。

set元素的key必须独一无二，因此其insert()用的是rb_tree的insert_unique()

multiset元素的key可以重复，因此其insert()用的是rb_tree的insert_equal()



![image-20230327185523811](assets/image-20230327185523811.png)

当你从set拿迭代器拿到的是const_iterator，这种迭代器是不允许改内容的。



# 容器 set, in VC6

没有identity，要自己写。写成一个inner class内部类。

![image-20230327190123221](assets/image-20230327190123221.png)



# 使用容器 multiset

multiset 放的是字符串，随机数转换成的字符串。

::find 是泛化的，而 c.find 是容器的特化版本，比较快。

![image-20230327190147368](assets/image-20230327190147368.png)



# 容器 map, multimap

map/multimap 以 rb_tree 为底层结构，因此有元素自动排序特性。排序的依据是key。

map/multimap 提供遍历操作及 iterators。按正常规则 (++ite) 遍历，便能获得排序状态 (sorted)。

我们无法使用 map/multimap 的 iterators 改变元素值 (因为key有其严谨排列规则)，但可以使用它来改变元素的data。因此 map/multimap 内部自动将 user 指定的 key type 设为 const，如此便能禁止 user 对元素的 key 赋值。

map 元素的 key 必须独一无二，因此其 insert() 用的是 rb_tree 的 insert_unique()

multimap 元素的 key 可以重复，因此其 insert() 用的是 rb_tree 的 insert_equal()

![image-20230327190228136](assets/image-20230327190228136.png)



# 容器 map, in VC6

![image-20230327190246175](assets/image-20230327190246175.png)

select1st() 重载了一个小括号()，它本身是一个仿函数。



# 使用容器 multimap

每写一个小测试，都把它包在一个命名空间里，非常完整独立，不会跟其他的测试交杂在一起。

![image-20230327190310268](assets/image-20230327190310268.png)

# 容器 map, 独特的 operator[ ]

![image-20230327190338093](assets/image-20230327190338093.png)

# 使用容器 map

使用 [] 安插元素，更加直观。

![image-20230327190358675](assets/image-20230327190358675.png)



