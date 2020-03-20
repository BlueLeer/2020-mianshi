# Java基础知识

## 1.你能说说Java中类是如何被加载的嘛?

1. 加载

   将外部的.class文件加载到Java虚拟机，.class文件代表了Java类的字节流，Java虚拟机会将这个静态字节流转化为方法区运行时数据结构也就是java.lang.Class对象，这个对象描述了这个类创建出来的对象的所有信息，比如有哪些构造方法，都有哪些成员方法和变量等等。在Java虚拟机中只存在唯一的该类型的Class对象，并作为访问该类的唯一入口。

2. 链接

   连接包括验证、准备及解析三个阶段

   * 验证阶段：保证被加载的类（.class文件的字节流）满足Java虚拟机规范，不会造成安全错误。
   * 准备阶段：负责为该类的的静态成员分配内存，并设置默认初始值。
   * 解析阶段：将类的的二进制数据中的符号引用替换成直接引用

3. 初始化

   初始化类变量和静态语句块。如果初始化类的时候它的父类还没有初始化，则先初始化其父类。如果包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。



## 2. 你能简单说下对象创建的过程吗？

1. 比如遇到了 new 关键字，需要创建一个对象
2. 类加载检查，检查类是否已经被加载到Java虚拟机了，如果还没有被加载，则先需要被加载进来
3. 为对象分配内存空间
4. 将内存空间初始化为零值
5. 对对象进行必要的设置（比如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的hash码、对象的分代年龄等，这些信息放在对象的对象头中）



## 3. 你能说说静态内部类和普通内部类的加载时机嘛？

静态内部类和普通内部类都是延时加载的，也就是说只有在明确使用到内部类的时候才会去加载，只使用内部类的时候不加载。我们熟知的单例模式之静态内部类：

```java
public class Singleton{
    private Singleton(){}
    private static class InstanceHolder{
        private static final INSTANCE = new Singleton();
    }
    public static Singleton getInstance(){
        return InstanceHolder.Instance;
    }
}
```

在调用`getInstance()`方法的时候才会去加载InstanceHolder静态内部类，加载`InstanceHolder`类的时候会初始化`INSTANCE`实例，并且我们知道在JVM中任何时候最多只能有一个类的`Class`实例，也就是说`InstanceHolder`的`Class`对象只有一个，那么它内部所持有的`INSTANCE`也就只有一个。



## 4.你能说说如何解决单例模式中的反序列问题嘛？

我们将一个单例序列化以后，然后反序列化，会生成另一个对象，这就破坏了单例模式。解决方式是，在需要单例的类中加入一个`readResolve()` 方法，同时返回该单例即可，代码如下：

```java
public class Singleton implements Serializable{
    private Singleton(){}
    private static class InstanceHolder{
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance(){
        return InstanceHolder.INSTANCE;
    }
    public Object readResolve(){
    	return InstanceHolder.INSTANCE;
    }
}
```

**那么这个readResolve()方法是从哪来的，为什么加上之后就能返回同一实例了呢？**

我们知道，对象反序列时通过ObjectInputStream来通过对象字节流创建对象的，它里面有个`readOrdinaryObject()`方法，回去类中是否有这个方法，如果有的话，它会调用这个方法拿到一个返回值，并将该返回值作为最终的反序列结果（大致意思是这个，可能有出入）。