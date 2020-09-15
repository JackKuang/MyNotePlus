# Object类

Object类是一个特殊的类，是所有类 的父类，如果一个类没有extends明确指出于某个类，那么它默认集成Object类 。

## 一、clone()

```java
protected native Object clone() throws CloneNotSupportedException;
```

保护方法，实现对象的浅拷贝，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。

主要是JAVA里除了8中基本类型传参数是值传递，其他的类对象传参数都是引用传递，我们有时候不希望再方法里讲参数改变，这里就需要再类中复写clone方法（实现深复制）。

## 二、toString()

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

Object类的toString方法返回一个字符出阿奴，该字符串有类名（对象是该类的一个示例）、@标记符和此对象哈希码的无符号十六进制组成。

## 三、getClass()

```java
public final native Class<?> getClass();
```

返回此Object运行时类型。

不可重写，要调用的话，一般和getName()联合使用，如getClass().getName()；

## 四、finalize()

```java
protected void finalize() throws Throwable { }
```

该方法用于释放资源。因为无法确定该方法什么时候被调用，很少使用。 

Java允许在类中定义一个名为finalize()方法。他的工作原理时：一旦垃圾回收器准备好释放对象占用的存储空间，讲首先调用其finalize()方法。并且在下一次垃圾回收动作发生时，才会正真回收对象占用的内存。

关于垃圾回收，需要注意以下几点：

* 对象可能不被垃圾回收。主要程序没有濒临存储空间用完的那一刻，对象占用的空间就不是释放。

* 垃圾回收只与内存有关。使用垃圾回收的唯一原因是为了回收程序不在使用的内存。

finalize()的用途：

​	无论对象是如何创建的，垃圾回收器都会负责释放对象占用的所有内容。这就对finalize()的需求限制到一种特殊情况，即通过某种创建对象的方式以外的方式为对象分配了存储空间。

​	不过这种情况都发生在使用"本地方法"的情况下，本地方法时一种在Java中调用非Java代码的方式。

## 五、equals()

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

Object的equals方法是直接判断this和obj本身的值是否相等，即用来判断equals的对象和形参obj所引用的对象是否是同一对象。

所谓同一对象就是指内存的同一块存储单元，如果this和obj指向同一块内存对象，则返true；如果this和obj指向的不是同一块内存，则返回false。

如果希望不同内存但是内容相同的两个对象equals时返回true，则我们需要重写父类的equals方法，String类就是一个很好的例子，它就重写了Object的equals方法。

## 六、hashCode()

```java
public native int hashCode();
```

返回该对象的哈希码值。

该方法用于哈希查找，可以减少在查找中使用equals的次数，重写了equals方法一般都要重写hashCode方法，这个方法在一些据有哈希功能的Collection中用到。

如果 obj1.equals(obj2) == true，可以推导obj1.hashCode == obj2.hashCode()，但是反过来却不一定。但是为了提高效率，应该尽量使两者可以相互推导。

## 七、wait()

```java
public final void wait() throws InterruptedException {
	wait(0);
}

public final native void wait(long timeout) throws InterruptedException;

/*
* @param      timeout   the maximum time to wait in milliseconds.
* 最长超时等待时间(毫秒)
* @param      nanos      additional time, in nanoseconds range 0-999999.
* 额外时间(微秒)
*/
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}
```

wait方法就是使当前线程等待该对象的锁，当前线程必须时该对象的拥有者，也就是据有该对象的锁。wait()方法一致等待，知道获得锁或者中断。wait(long timeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。

调用该方法后当前线程进入睡眠状态，直到以下事件发生：

1. 其他线程调用该对象的notify方法。
2. 其他线程调用该对象的notifyAll方法。
3. 其他线程调用了interrupt中断该线程。
4. 时间间隔到了。

此时该线程就可以被调度了，如是被中断的话就会抛出InteruptedException异常。

## 八、notify()

```java
public final native void notify();
```

该方法随机唤醒该对想上等待的某个线程。

## 九、notifyAll()

```java
public final native void notifyAll();
```

该方法唤醒在该对象上等待的所有线程。