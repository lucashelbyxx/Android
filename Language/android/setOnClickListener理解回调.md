

# setOnClickListener分析

## **点击事件监听是怎么实现的？**

## **为什么传入new View.OnClickListener()参数，要去实现onClick()方法？**

## **实现的onClick()方法，没有在Activity调用？是在哪里调用的？**



回调是你实现了他，他在另一个地方调用。

setOnClickListener方法

```java
public void setOnClickListener(@Nullable OnClickListener l) {
    if (!isClickable()) {
        setClickable(true);
    }
    getListenerInfo().mOnClickListener = l;
}
```

setOnClickListener方法，传入一个OnClickListener对象作为参数

```java
public interface OnClickListener {
    /**
         * Called when a view has been clicked.
         *
         * @param v The view that was clicked.
         */
    void onClick(View v);
}
```



```java
getListenerInfo().mOnClickListener = l;
```

把传入的OnClickListener对象赋值给了getListenerInfo().mOnClickListener，传入的OnClickListener对象就相当于携带了我们实现的onClick(View view)方法

```java
ListenerInfo getListenerInfo() {
    if (mListenerInfo != null) {
        return mListenerInfo;
    }
    mListenerInfo = new ListenerInfo();
    return mListenerInfo;
}
```

getListenerInfo() 方法，单例模式 

ListenerInfo 是一个内部静态类，成员包括各种事件的监听接口,其中包括

```java
public OnClickListener mOnClickListener;
```

绕了这么一大圈（我们先不管为啥绕），我们传入的持有我们实现的onClick(View view)方法的OnClickListener接口对象(还记得吗？)，被赋值到了View中的mListenerInfo中的mOnClickListener对象，也就是，我们实现的onCLick(View view) 方法，被mListenerInfo.mOnClickListener持有了

我们实现的onClick(View view)应该就是在 View中被调用了

```java
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }

    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}
```

梳理一遍流程，我们在Activity或者Fragment中调用

View.setOnClickListener方法，传入一个OnCLickListener对象，实现了onCLick(View view)方法，然后在View中的某个地方，我们实现的onCLick(View view)被调用，实现了回调，这就是回调的流程

## **异步**

回调有什么用呢，就是异步，想象一下，系统一直在监听着屏幕的点击事件，在我们触摸到屏幕的时候进行响应，这是一个线程操作，因为如果这个放在主线程，那在事件被响应之前，我们的线程都是阻塞的，因为屏幕的资源被占用了，无法进行其他操作，而在子线程中，系统监听着屏幕的活动，然后在我们触摸时，调用performClick()方法实现了点击，并且调用了onClick(View view)方法实现了点击事件的回调，我们就可以恰恰刚好在点击时间触发的时候，进行我们想要的操作，也就是我们实现的on CLick(View view)方法



## 实现回调

```java
// A.class
// 定义接口
public interface Listener {
    // 回调方法
    void 回调方法();
}

// 声明一个接口
private Listener mListener;

// set 接口方法
public void setListener(Listener listener) {
    mListener = listener;
}

// 执行某个操作
private void 某个操作() {
    // 调用回调方法
    mListener.回调方法();
}


//B.class
private A A = new A();

a.setListener(new Listener() {
    public void 回调方法() {
        // 在A中某个操作时要做的事情
    }
});
```

