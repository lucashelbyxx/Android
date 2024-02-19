Java JNI的本意是Java Native Interface（Java本地接口），它是为了方便Java 调用C、C++等本地代码所封装的一层接口。

NDK是Android所提供的一个工具集合，通过NDK可以在Android中更加方便地通过JNI来访问本地代码，比如C或者C++。

# 1、JNI的开发流程

首先需要在Java中声明native方法，接着用C或者C++实现native方法

\1. 在Java中声明native方法

创建一个类，这里叫做JniTest.java

```java
package com.ryg;
import java.lang.System;
public class JniTest {
    static {
    	System.loadLibrary("jni-test");
    }
    public static void main(String args[]) {
        JniTest jniTest = new JniTest();
        System.out.println(jniTest.get());
        jniTest.set("hello world");
    }
    public native String get();
    public native void set(String str);
}
```

声明了两个native方法：get和set(String)，这两个就是需要在JNI中实现的方法。在JniTest的头部有一个加载动态库的过程，其中 jni-test是so库的标识，so库完整的名称为libjni-test.so，这是加载so库的规范。

\2. 编译Java源文件得到class文件，然后通过javah命令导出JNI的头文件

\3. 实现JNI方法

\4. 编译so库并在Java中调用