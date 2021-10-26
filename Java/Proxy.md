## 一、来源

来自某公司某个过程中的笔试题

```java

interface MyInterface {
    String process();
}


public class BuildObject {

    public static void main(String[] args) throws Exception {
        MyInterface MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$process=Abc");
        System.out.println(MyInterface.process());
        MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$process=Bcd");
        System.out.println(MyInterface.process());
        MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$processTest=Bcd");
        System.out.println(MyInterface.process());
    }


    static Object buildObject(final String str) throws Exception {
        // TODO
        return null;
    }


```



## 二、处理

笔试的时候没有处理好，后面尝试解决答案如下：

```java

import java.lang.reflect.Proxy;

interface MyInterface {
    String process();
}


public class BuildObject {

    public static void main(String[] args) throws Exception {
        MyInterface MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$process=Abc");
        System.out.println(MyInterface.process());
        MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$process=Bcd");
        System.out.println(MyInterface.process());
        MyInterface = (MyInterface) buildObject(MyInterface.class.getName() + "$processTest=Bcd");
        System.out.println(MyInterface.process());
    }


    static Object buildObject(final String str) throws Exception {
        String className = str.substring(0, str.indexOf("$"));
        String methodName = str.substring(str.indexOf("$") + 1, str.indexOf("="));
        String methodValue = str.substring(str.indexOf("=") + 1);

        Class clazz = Class.forName(className);
        return Proxy.newProxyInstance(clazz.getClassLoader(), new Class[]{clazz}, (proxy, method, args) -> {
            if (method.getName().equals(methodName)) {
                return methodValue;
            } else {
                return null;
            }
        });
    }
}

```



## 三、知识巩固

JDK中的动态代理是通过反射类Proxy以及InvocationHandler回调接口实现的;但是，JDK中所要进行动态代理的类必须要实现一个接口，也就是说**只能对该类所实现接口中定义的方法进行代理**，这在实际编程中具有一定的局限性，而且使用反射的效率也并不是很高。

**要生成某一个对象的代理对象，这个代理对象通常也要编写一个类来生成**，所以首先要编写用于生成代理对象的类。在java中如何用程序去生成一个对象的代理对象呢，java在JDK1.5之后提供了一个"**java.lang.reflect.Proxy**"类，通过"**Proxy**"类提供的一个**newProxyInstance**方法用来创建一个对象的代理对象，如下所示：

示例业务逻辑：

1. 娱乐明星都会唱歌、演习(interface Star)

2. 有一个明星叫胡歌（class HuGe implements Star）

3. 他有两个助理(分别对应两个代理类)(class HuGeProxy1、class HuGeProxy2)

4. 如果要找胡歌唱歌、演戏，需要先找两个助理中的一个，然后助理去找胡歌唱歌、演戏（class ProxyTest）