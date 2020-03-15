---
title: Java核心技术——单例模式总结
date: 2019-06-11 23:16:22
tags: [Java，读书笔记]
categories: 核心技术读书笔记
---

## 单例模式总结

单例模式的特点：对于一个类，只能创建一个对象，后续所有地方获取到的都是同一个对象。

### 单例模式要点

​	①构造函数写成私有，不能在其他类中创建。

​	②类提供一个静态函数，返回该类的唯一对象。（如果有则直接返回，如果没有则先创建再返回）

### 常见实现方式

①最基础的实现方案，懒汉式：等到用到时再去创建对象。**问题**：线程不安全，可能会创建多个对象，并且获取到的不是同一个对象。**不推荐使用**

```java
public class Singleton{
    private static Singleton instance = null;
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        if(null == instance){
            instance = new Singleton();
        }
        return instance;
    }
}
```

②懒汉式改进：给方法加上`synchronized`关键字，**问题**：效率低，每次获取时都要进行同步，而我们只想在第一次创建对象的时候进行同步就好了。**不推荐使用**

```java
public class Singleton{
    private static Singleton instance = null;
    
    private Singleton(){
    }
    //synchronized关键字保证了不会发生指令重排
    public static synchronized Singleton getInstance(){
        if(null == instance){
            instance = new Singleton();
        }
        return instance;
    }
}
```

③懒汉式再改进，双重校验锁。**可以使用**

```java
public class Singleton{
    //如果这里没有加volatile关键字，创建对象时可能会存在指令重排的问题，导致其他线程获取到没有完成初始化的对象
    //什么是指令重排问题？创建对象的代码会被编译成以下三条指令：①给对象分配内存空间②初始化对象③将变量指向完成初始化的对象，有时候可能执行顺序会变成1->3->2,导致另一个线程判null失败，直接返回并没有完成初始化的对象
    private static volatile Singleton instance = null;
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        if(null == instance){
            synchronized(Singleton.class){
                if(null == instance){
                    instance = new Singleton();
                }   
            }
        }
        return instance;
    }
}
```

④基础实现方案二，饿汉式：加载类信息的时候就把对象实例初始化好，要用的时候直接返回。**问题**：加载的时候就将对象实例创建好，没有懒加载的效果，如果后续根本没用到，会造成内存浪费。**可以使用**

```java
public class Singleton{
    //这里是通过类加载器来保证线程安全的，类加载器的loadClass()中加了synchronized关键字实现同步，不会多次装载，也就不会多次创建静态变量
    private static Singleton instance = new Singleton();
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        return instance;
    }
}
```

⑤饿汉改进，内部静态类。既保留了使用饿汉式保证线程安全的优点，又实现了懒加载，只有在获取的时候才会去加载`SingletonHolder`类并创建对象实例。**问题**：通过反射和序列化仍然可以破坏单例。**可以使用**

```java
public class Singleton{
    
    private static class SingletonHolder{
        private static final Singleton instance = new Singleton();
    }
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

⑥防御反射，在构造函数中进行额外判断，第二次创建时会抛异常。**问题**：~~不能避免通过反射直接将静态内部类的变量置为`null`的操作~~，同时也无法避免序列化破坏单例的问题。**可以使用**

```java
public class Singleton{
    
    private static class SingletonHolder{
        //因为这里用了final，所以利用反射重新置null的方案行不通
        private static final Singleton instance = new Singleton();
    }
    
    private Singleton(){
    	if(null != SingletonHolder.instance){
            throw new RuntimeException("单例模式正在被反射攻击！");
    	}
    }
    
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

⑦防御序列化，对于需要实现`Serializable`接口的情况，可以在类中定义`readResolve()`来避免序列化破坏单例。**问题**：~~不能避免反射破坏单例~~，并且比较复杂。**可以使用**

```java
public class Singleton{
    
    private static class SingletonHolder{
        private static final Singleton instance = new Singleton();
    }
    
    private Singleton(){
    	if（null != SingletonHolder.instance）{
            throw new RuntimeException("单例模式正在被反射攻击！");
    	}
    }
    
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
    
    private Object readResolve() {
        return SingletonHolder.instance;
    }
}
```

⑧终极方案，使用枚举实现单例。可以同时避免线程安全问题和序列化破坏单例的问题。**强烈推荐使用**

```java
public enum Singleton {  
    INSTANCE; 
}  
```

#### 为什么枚举类能保证线程安全呢？

```java
public enum T {    
	SPRING,SUMMER,AUTUMN,WINTER;
}
//上述枚举类经过反编译后，其代码如下所示
public final class T extends Enum{   //final表示枚举类不能被继承 
    
    private T(String s, int i) {
        super(s, i);
    }
    
    public static T[] values() {
        T at[];
        int i;
        T at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new T[i = at.length], 0, i);
        return at1;
    }

    public static T valueOf(String s) {
        return (T)Enum.valueOf(demo/T, s);
    }

    public static final T SPRING;    
    public static final T SUMMER;    
    public static final T AUTUMN;    
    public static final T WINTER;  
    
    private static final T ENUM$VALUES[];    
    
    static {        
        SPRING = new T("SPRING", 0);        
        SUMMER = new T("SUMMER", 1);        
        AUTUMN = new T("AUTUMN", 2);        
        WINTER = new T("WINTER", 3);        
        ENUM$VALUES = (new T[] {SPRING, SUMMER, AUTUMN, WINTER});    
    }
    //枚举类加载时ClassLoader类的loadClass()默认保证线程安全，所以枚举类天生就是线程安全的。
}
```

#### 为什么枚举类能保证序列化安全呢？

为了保证枚举类型像`Java`规范中所说的那样，每一个枚举类型极其定义的枚举变量在`JVM中`都是唯一的`Java`语言规范中定义：在序列化的时候`Java`仅仅是**将枚举对象的`name`属性输出到结果中，反序列化的时候则是通过`java.lang.Enum`的`valueOf`方法来根据名字查找枚举对象**。同时，编译器是不允许任何对这种序列化机制的定制的，因此禁用了`writeObject、readObject、readObjectNoData、writeReplace`和`readResolve`等方法。

从代码中可以看到，代码会尝试从调用`enumType`这个`Class`对象的`enumConstantDirectory()`方法返回的`map`中获取名字为`name`的枚举对象，如果不存在就会抛出异常。再进一步跟到`enumConstantDirectory()`方法，就会发现到最后会以反射的方式调用`enumType`这个类型的`values()`静态方法，也就是上面我们看到的编译器为我们创建的那个方法，然后用返回结果填充`enumType`这个`Class`对象中的`enumConstantDirectory`属性。**所以，`JVM`对序列化有保证。**

#### 为什么枚举类能避免反射破坏呢？

1.如果利用反射获取无参构造器，然后调用`newInstance`方法，会抛`NoSuchMethodException`异常，因为枚举类没有无参构造器，只有父类有一个参数为`（String.class,int.class）`的构造器。

2.如果利用反射获取父类构造器，然后调用`newInstance`方法，会抛`IllegalArgumentException`异常，因为反射在通过`newInstance`创建对象时，**会检查该类是否`ENUM`修饰**，如果是则抛出异常，反射失败。

```java
@CallerSensitive
public T newInstance(Object ... initargs) throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException {
               
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)   //这里是关键
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

[枚举实现单例模式的好处](https://www.cnblogs.com/chiclee/p/9097772.html)

#### 额外补充：如何不使用锁保证线程安全呢？

借助`CAS`+循环操作可以实现。**问题**：该算法的问题是不能避免初始化多个实例，对于初始化方法比较复杂的场景，可能会影响性能。

```java
public class Singleton {
    
    private static final AtomicReference<Singleton> INSTANCE = new AtomicReference<Singleton>(); 

    private Singleton() {}

    public static Singleton getInstance() {
        for (;;) {
            Singleton singleton = INSTANCE.get();
            if (null != singleton) {
                return singleton;
            }
            
            singleton = new Singleton();
            if (INSTANCE.compareAndSet(null, singleton)) {
                return singleton;
            }
        }
    }
}
```

