# Material Design 简介

大多数人可能会认为Android系统的UI并不算美观，至少没有iOS系统的美观。以至于很多IT公司在进行应用界面设计的时候，为了保证双平台的统一性，强制要求Android端的界面风格必须和iOS端一致。这种情况在现实工作当中实在是太常见了，虽然我认为这是非常不合理的。因为对于一般用户来说，他们不太可能会在两个操作系统上分别使用同一个应用，但是必定会在同一个操作系统上使用不同的应用。因此，同一个操作系统中各个应用之间的界面统一性要远比一个应用在双平台的界面统一性重要得多。

但是Android标准的界面设计风格并不是特别被大众所接受，很多公司觉得自己可以设计出更加好看的界面，从而导致Android平台的界面风格长期难以得到统一。为了解决这个问题，Google也是使出了杀手锏，在2014年Google I/O大会上重磅推出了一套全新的界面设计语言——Material Design。

# Toolbar

Toolbar的强大之处在于，它不仅继承了ActionBar的所有功能，而且灵活性很高，可以配合其他控件完成一些Material Design的效果。

这里使用xmlns:app指定了一个新的命名空间。正是由于每个布局文件都会使用xmlns:android来指定一个命名空间，我们才能一直使用android:id、android:layout_width等写法。这里指定了xmlns:app，也就是说现在可以使用app:attribute这样的写法了。但是为什么这里要指定一个xmlns:app的命名空间呢？这是由于许多Material属性是在新系统中新增的，老系统中并不存在，那么为了能够兼容老系统，我们就不能使用android:attribute这样的写法了，而是应该使用app:attribute。

# 滑动菜单

## DrawerLayout

所谓的滑动菜单，就是将一些菜单选项隐藏起来，而不是放置在主屏幕上，然后可以通过滑动的方式将菜单显示出来。这种方式既节省了屏幕空间，又实现了非常好的动画效果，是Material Design中推荐的做法。

DrawerLayout的用法。首先它是一个布局，在布局中允许放入两个直接子控件：第一个子控件是主屏幕中显示的内容，第二个子控件是滑动菜单中显示的内容。

# 悬浮按钮和可交互提示

## FloatingActionButton

# 卡片式布局

## MaterialCardView

# 下拉刷新

SwipeRefreshLayout就是用于实现下拉刷新功能的核心类，我们把想要实现下拉刷新功能的控件放置到SwipeRefreshLayout中，就可以迅速让这个控件支持下拉刷新。

# 可折叠式标题栏

## CollapsingToolbarLayout

## 充分利用系统状态栏空间

想要让背景图能够和系统状态栏融合，需要借助android:fitsSystemWindows这个属性来实现。

# kotlin