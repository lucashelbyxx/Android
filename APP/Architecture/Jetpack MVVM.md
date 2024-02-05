[KunMinX/Jetpack-MVVM-Best-Practice:](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice)

[JetPack MVVM - DoubleWay的博客 | DW Blog](http://doubleway.top/2020/06/15/JetPack__MVVM/)



# Lifecycle 便于声明周期同步
对于 Activity、Fragment，开发者需要在不同的生命周期回调中做不同的事情。如在 onCreate 做初始化操作、onResume 做恢复操作等等。
但一些组件强依赖于 Activity、Fragment 生命周期，常规写法一旦疏忽就会引发安全问题。

# LiveData
# ViewModel
# DataBinding
