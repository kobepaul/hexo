---
title: Java核心技术——对象继承
date: 2019-06-11 23:16:22
tags: [Java，读书笔记]
categories: 核心技术读书笔记
---

1. `Java`的继承是公有继承，即派生类内部可以访问基类中`public`和`protected`成员，但是类外只能通过派生类的对象访问基类的`public`成员。

2. 在子类重写父类的方法中，要想调用父类的方法，需要加上`super`关键字，否则会**循环调用子类自己的方法**。

3. 如果子类的构造器没有显式地调用父类的构造器，则会默认调用父类的无参构造器；若父类没有无参构造器，则会报错。

4. 多态：一个对象变量可以引用多种实际类型的对象；动态绑定：在运行时能够自动地选择调用哪个方法；动态绑定过程详见（`P157`）。

5. 对于子类中没有重写父类方法的情况，用父类变量引用子类对象后，调用子类内部定义的新方法会报错；多态调用的前提是**调用的方法在父类中存在**。

6. 对于子类继承父类，并且重写方法的返回值不一致时，子类的字节码文件中会多出一个返回值类型跟父类一样的方法，该方法中调用了子类重写的方法。

   ```java
   /**
   public A getInstance() {    //类A中
   	return new A();
   }
   
   @Override
   public B getInstance() {    //类B中，称为多态返回类型的协变
   	return new B();
   }
   */
   
   public A getInstance();
       Code:
          0: aload_0
          1: invokevirtual #38    // Method getInstance:()LB;
          4: areturn
   ```

7. 数组可能引发的问题：下面这段代码编译不会报错，但是运行时会报 `java.lang.ArrayStoreException`异常。

   ```java
   public class People {
   }
   
   public class Student extends People{
   	private String name = "1";
   	public Student(){
   	}
   	public String getName() {
   		return name;
   	}
   }
   
   public class StudentTest {
   	public static void main(String[] args) {
   		Student[] b = new Student[10];
   		People[] a = b;    //数组协变    
   		a[0] = new People();
   		System.out.println(b[0].getName());
   	}
   }
   ```

8. [协变和逆变](https://www.jianshu.com/p/2bf15c5265c5)

9. 方法调用的过程：假设调用`x.f(args)`，隐式参数`x`声明为类`C`的一个对象。

   ```java
   1.编译器查看对象的声明类型和方法名，会一一列举所有C类中名为f的方法和其超类中访问属性为public且名为f的方法（超类的私有方法是不能访问的）。
   2.编译器将查看调用方法时提供的参数类型，在上述所有的方法中寻找一个与提供的参数类型完全匹配的方法，该过程被称为重载解析。由于允许类型转换（向上转型），所以这个过程比较复杂。如果没找到对应的方法，或者找到了多个对应的方法，编译器会报错。
   3.如果是private、static、final等关键词修饰的方法或构造器方法，编译器可以准确地知道该调用哪个方法。这类方法的调用方式，称为静态绑定；对于那些调用的实际方法依赖于隐式参数实际类型的调用方式，称为动态绑定。
   4.对于动态绑定，虚拟机会调用与x所引用对象的实际类型最匹配的那个类的方法。
   ```

10. 由于每次调用方法都要进行搜索，时间开销比较大。所以虚拟机预先为每个类创建了一个方法表，进行方法调用时只需要搜索方法表即可。

11. 如果子类中没有重新定义静态变量，那么与父类共用同一个；如果子类中定义了，那么是两个不同的静态变量。

12. 子类覆盖父类的方法时，其可见性不能低于父类方法的可见性。

13. `private`、`static`和`final`修饰的方法都属于静态绑定。下面代码的输出结果均为`111`。

    ```java
    public class People {
    	public static void say(){
    		System.out.println("111");
    	}
    }
    
    public class Student extends People{
    	public static void say(){
    		System.out.println("222");
    	}
    }
    
    public class StudentTest {
    	public static void main(String[] args) {
    		sayHello(new People());
    		sayHello(new Student());
    	}
    	public static void sayHello(People people){
    		people.say();
    	}
    }
    ```

14. 将一个类声明为`final`后，其中的所有方法都默认为`final`修饰的，但是不包括成员变量。

15. 在父类中将某个方法或成员变量声明为`protected`修饰后，允许子类访问父类中的该方法或成员变量，但是子类中的方法只能访问从子类从父类中集成来的`protected`成员变量，而不能访问其他父类对象的`protected`成员变量。（`P165`）[protected案例，关键子类和父类是不是同处一个包下](https://www.cnblogs.com/zhao1949/p/5717064.html)    [protected案例](https://blog.csdn.net/someday_spark/article/details/80006215)    **在不是同一个包的情况下，`protected`修饰的变量和方法会被子类继承，但是子类也只可以访问自己继承而来的方法和变量，不可以访问父类的（同包可以访问）。重写的`protected`方法根据重写的来计算可见性，未重写的像上追溯到有方法体的父类来计算可见性。**

16. `getClass()`方法会获取对象的实际类型，而`instanceof`只要是该类或该类的子类都返回`true`。所以当`equals()`方法只定义在父类中，子类中没有时，可以用`instanceof`进行判断（此时父类中的`equal()`最好定义为`final`修饰的）；而`equals()`方法在子类中重新定义时，只能使用`getClass()`进行判断（**是写在父类中的，子类只是判断下相比父类新增的成员变量**）。（`P169`）

17. 比较两个对象相等时，使用`Objects.equals()`更好，可以避免其中一个对象为`null`导致的空指针问题。

18. 重写`Object`对象的`equals()`方法时，则**该方法的参数必须是`Object`类型，不然无法实现重写**。

19. `ArrayList.set()`在没有元素的情况下不能使用

    ```
    ArrayList<Employee> list = new ArrayList<>();
    //调用下面的方法会报错，因为此时还没有元素。
    list.set(0,new Employee());
    ```

20. 基本数据类型装箱和拆箱过程本质

    ```
    Integer n = 10；    //实际用的是Integer.valueOf()
    int n = new Integer(10);    //实际用的是Integer.intValue()
    ```

21. 在条件表达式当中，若两种结果的类型不一致，比如一个为`int`，另一个为`double`，则**首先会进行统一，将`int`值提升为`double`值**。

22. 如果需要将`int`或`Integer`等参数值传入方法中，并进行修改的情况，需要使用`IntHolder`等类，其实现原理就是包装一层，使得成员变量可变。

23. 基本数据类型和数组都有对应的`Class`对象

    ```
    Class clsInt = int.class    //返回 int
    Class clsInt[] = new int[10].class    //返回 class [Ljava.lang.Integer;
    ```

24. 使用`Class.newInstance()`的前提是该类有无参构造器，如果需要给构造器提供参数，则需要使用`Constructor`类中的`newInstance()`。

25. 使用反射编写数组复制代码

    ```java
    public static Object goodCopy(Object a,int newLength){    //适合基本数据类型组成的数组
        Class cls = a.getClass();
        if(!cls.isArray())
        	return null;
        Class componentType = cls.getComponentType();
        int length = Array.getLength(a);
        Object newArray = Array.newInstance(componentType,newLength);
        System.arraycopy(a,0,newArray,0,Math.min(length,newLength));    //多余的元素舍弃
        return newArray;
    }
    ```

26. 继承的设计技巧

    ```java
    1.将公共操作和域放在超类
    2.不要使用受保护的域（相同包下的所有类和不同包下的子类均能直接修改该值），但是受保护的方法很有用
    3.使用继承实现“is-a”关系
    4.除非所有继承的方法都有意义，否则不要使用继承
    5.在覆盖方法时，不要改变预期的行为（置换原则）
    6.使用多态，而非类型信息
    7.不要过多地使用反射
    ```

27. `Java`编译器在对类型转换进行检查后，如果没有发现违反规则的现象，会将所有的类型化数组列表转换成原始`ArrayList`对象，也就是对象并没有参数化信息。

28. 包装器对象均是不可变的，一旦初始化了一个对象，则不能再更改其中的值。

29. `getFields()`将返回所有`public`修饰的成员变量，包括父类中定义的；而`getDeclareFields()`将返回自己类中定义的所有成员变量，不管修饰符是什么类型。

30. [Java重载时的准确性原则](https://blog.csdn.net/jackpk/article/details/43453313)，优先匹配最佳的，然后再考虑自动拆装箱以及基本数据类型转换等因素。

31. 关于继承的调用链 参考[Java继承调用](https://blog.csdn.net/qq_31655965/article/details/54746235)

    ```java
    class A {
        public String show(D obj) {
            return ("A and D");
        }
    
        public String show(A obj) {
            return ("A and A");
        }
    
    }
    
    class B extends A{
        public String show(B obj){
            return ("B and B");
        }
    
        public String show(A obj){
            return ("B and A");
        }
    }
    
    class C extends B{
    }
    
    class D extends B{
    }
    
    public class Demo {
        public static void main(String[] args) {
            A a1 = new A();
            A a2 = new B();
            B b = new B();
            C c = new C();
            D d = new D();
    
            System.out.println("1--" + a1.show(b));
            System.out.println("2--" + a1.show(c));
            System.out.println("3--" + a1.show(d));
            System.out.println("4--" + a2.show(b));
            System.out.println("5--" + a2.show(c));
            System.out.println("6--" + a2.show(d));
            System.out.println("7--" + b.show(b));
            System.out.println("8--" + b.show(c));
            System.out.println("9--" + b.show(d));
        }
    }
    //结果：
    //1--A and A
    //2--A and A
    //3--A and D
    //4--B and A
    //5--B and A
    //6--A and D
    //7--B and B
    //8--B and B
    //9--A and D
    
    /*总结：
    1.当父类对象引用变量引用子类对象时，被引用对象的类型决定了调用谁的成员方法，
    引用变量类型决定可调用的方法。如果子类中没有覆盖该方法，那么会去父类中寻找。
    2.继承链中对象方法的调用的优先级：this.show(O)、super.show(O)、this.show((super)O)、
    super.show((super)O)。*/
    ```

    

