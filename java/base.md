# 简介
Java两大核心机制
+ Java虚拟机（Java Virtual Machine）: 具有指令集并使用不同的存储区域。负责执行指令，管理数据、内存、寄存器
+ 垃圾收集机制（Garbage Collection）: 它提供一种系统级线程跟踪存储空间的分配情况。并在JVM空闲时，检查并释放那些可被释放的存储空间

JDK(Java Development Kit )：JDK是提供给 Java 开发人员使用的，其中包含了 java 的开发工具，也包括了JRE。
JRE(Java Runtime Environment) ：包括Java虚拟机和 Java 程序所需的核心类库等

## 关键字

```sh
# 用于定义数据类型的关键字
class	interface	enum	void
byte	short int	long	float	double	char boolean				
# 用于定义数据类型值的关键字
true	false	null		
# 用于定义流程控制的关键字
if	else	switch	case	default
while	do	for	break	continue return	
# 用于定义访问权限修饰符的关键字
private	protected	public		
# 用于定义类，函数，变量修饰符的关键字
abstract	final	static	synchronized	
# 用于定义类与类之间关系的关键字
extends	implements			
# 用于定义建立实例及引用实例，判断实例的关键字
new	this	super	instanceof	
# 用于异常处理的关键字
try	catch	finally	throw	throws
# 用于包的关键字
package	import			
# 其他修饰符关键字
native	strictfp	transient	volatile	assert
```

基本数据类型和 引用类型（class,interface, 数组）

byte,short,char之间不会相互转换，他们三者在计算时首先转换为int类型 —> long -> float -> double

strictfp（strict floating-point）： 并不常见，通常用于修饰一个方法，用于限制浮点数计算的精度和舍入行为。在类、接口或方法上使用 strictfp 时，该范围内的所有浮点数计算将遵循 IEEE 754 标准的规定，以确保跨平台的浮点数计算的一致性

transient： 修饰的字段不会被序列化
volatile： 保证不同线程对它修饰的变量进行操作时的可见性

变量可以分为局部变量、成员变量、静态变量

### static
static标记的变量或方法由整个类(所有实例)共享，如访问控制权限允许，可不必创建该类对象而直接用类名加‘.’调用。
static成员也称类成员或静态成员，如：类变量、类方法、静态方法等。
在static方法内部只能访问类的static属性，不能访问类的非static属性

在静态方法里只能直接调用同类中其它的静态成员（包括变量和方法），而不能直接访问类中的非静态成员
静态方法不能以任何方式引用this和super关键字

一个类中可以使用不包含在任何方法体中的静态代码块(static block )，当类被载入时，静态代码块被执行，且只被执行一次，
静态块经常用来进行类属性的初始化

```java
public static int total;
static {
    total = 100;//为total赋初值 
}
```

### final
final标记的变量(成员变量或局部变量)即成为常量，只能赋值一次
final标记的类不能被继承。提高安全性，提高程序的可读性
final标记的方法不能被子类重写。增加安全性
final标记的成员变量必须在声明的同时或在每个构造方法中显式赋值，然后才能使用


## 数组
Java语言中声明数组时不能指定其长度

```java
int[]= a new int[5];
MyDate dates[] = { new MyDate(22, 7, 1964),};
int[][] t  = new int [4][];

int [3][2] intArray1 = {{1,2},{2,3},{4,5}};
```

# 面向对象
类是对一类事物描述，是抽象的、概念上的定义；对象是实际存在的该类事物的每个个体，因而也称实例(instance)

封装有 4 大好处：
+ 良好的封装能够减少耦合
+ 类内部的结构可以自由修改
+ 可以对成员进行更精确的控制
+ 隐藏信息，实现细节

Java 重写(Override)
+ 参数列表必须完全相同
+ 返回类型与被重写方法的返回类型可以不相同，但是必须是父类返回值的派生类
+ 访问权限不能比父类中被重写的方法的访问权限更低
+ 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为 private 和 final 的方法。
+ 子类和父类不在同一个包中，那么子类只能够重写父类的声明为 public 和 protected 的非 final 方法
+ 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。但是，重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以

重载(Overload)是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同

```java
< modifier> class < name> [extends < superclass>] [implements < interface> [,< interface>]* ] {
    < declarations>*
}

//  用于判断对象是否属于某个类型
p instanceof Person
```

## 方法

```java
public static void print(String... strs) {
    for (String s : strs)
        System.out.println(s);
}
```

## 代码初始化块
静态初始化块在类加载时执行，只会执行一次，并且优先于实例初始化块和构造方法的执行；实例初始化块在每次创建对象时执行，在构造方法之前执行

## 多态
同一个类的对象在不同情况下表现出不同的行为和状态

1.多态是方法的多态，不是属性的多态（多态与属性无关）
2.多态存在要有3个必要条件：继承、方法重写、父类引用指向子类对象。
3.父类引用指向子类对象后，用该父类引用调用子类重写的方法，此时多态就出现了。

# 接口
接口(interface)是抽象方法和常量值的定义的集合
+ 接口中的所有成员变量都默认是由 `public static final` 修饰的
+ 没有使用 private、default 或者 static 关键字修饰的方法是隐式抽象的, 默认 `public abstract` 修饰的。接口没有构造方法
+ 实现接口的类中必须提供接口中所有方法的具体实现内容
+ 多个无关的类可以实现同一个接口
+ 一个类可以实现多个无关的接口
+ 与继承关系类似，接口与实现类之间存在多态性
+ 接口也可以继承另一个接口，使用extends

```java
public interface Test {
    String a = "a";
    // 抽象方法
    string get();
    // 静态方法
    static boolean func1(String a) { return true;  }
    // 默认方法
    default void func2() { System.out.println(a);}
}

```

# 内部类
在Java中，允许一个类的定义位于另一个类的内部，前者称为内部类, 内部类和外层封装它的类之间存在逻辑上的所属关系
Inner class一般用在定义它的类或语句块之内，在外部引用它时必须给出完整的名称。 Inner class的名字不能与包含它的类名相同；
Inner class可以使用包含它的类的静态和实例成员变量，也可以使用它所在方法的局部变量；

广义上我们将内部类分为四种：成员内部类、静态内部类、局部（方法）内部类、匿名内部类。
+ 使用内部类最吸引人的原因是：每个内部类都能独立地继承一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响
+ 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立
+ 内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体
+ 内部类提供了更好的封装，除了该外围类，其他类都不能访问
+ 创建内部类对象的时刻并不依赖于外围类对象的创建

Inner class可以声明为抽象类 ，因此可以被其它的内部类继承。也可以声明为final的。
和外层类不同，Inner class可以声明为private或protected；
Inner class 可以声明为static的，但此时就不能再使用外层封装类的非static的成员变量；
非static的内部类中的成员不能声明为static的，只有在顶层类或static的内部类中才可声明static成员；

## 成员内部类
内部类的内部不能有静态信息, 内部类可以直接使用外部类的任何信息，如果属性或者方法发生冲突，调用外部类.this.属性或者方法

```java
public class Outer {
    private int outerVariable = 1;
    private int commonVariable = 2;
    private static int outerStaticVariable = 3;
    public void outerMethod() {}
    public static void outerStaticMethod() { }

    public class Inner {
        private int commonVariable = 20;
        public void innerShow() {
            //当和外部类冲突时，直接引用属性名，是内部类的成员属性
            System.out.println("内部的commonVariable:" + commonVariable);
            //内部类访问外部属性
            System.out.println("outerVariable:" + outerVariable);
            //当和外部类属性名重叠时，可通过外部类名.this.属性名
            System.out.println("外部的commonVariable:" + Outer.this.commonVariable);
            System.out.println("outerStaticVariable:" + outerStaticVariable);
            //访问外部类的方法
            outerMethod();
            outerStaticMethod();
        }
    }
    
    // 外部类访问内部类信息
    public void outerShow() {
        Inner inner = new Inner();
        inner.innerShow();
    }
}

//外部类对象
Outer outer = new Outer();
//创造内部类对象
Outer.Inner inner = outer.new Inner();
```

## 静态内部类
内部可以包含任意的信息
静态内部类的方法只能访问外部类的static关联的信息，可以独立存在，不依赖于其他外围类
```java
public class Outer {
    static {    System.out.println("Outer的静态块被执行了……");   }
    public static class Inner {
        static {      System.out.println("Outer.Inner的静态块执行了……");} // 先执行
        private static int innerStaticVariable = 30;
    }
    // 外部类的内部如何和内部类打交道
    public static void callInner() {
        System.out.println(Inner.innerStaticVariable);
        Inner.innerStaticShow();
    }
}
//访问静态内部类的静态方法，Inner类被加载,此时外部类未被加载，独立存在，不依赖于外围类
Outer.Inner.innerStaticShow();
//访问静态内部类的成员方法
Outer.Inner oi = new Outer.Inner();
oi.innerShow();
```

## 局部内部类
无法创造静态信息
可以直接访问方法内的局部变量和参数（final,已被赋值且始终未改变的变量），但是不能更改
可以随意的访问外部类的任何信息

```java
public class Outer {
    public void outerCreatMethod(int value) {
        // 局部内部类，类前不能有访问修饰符
        class Inner {}
        //局部内部类只能在方法内使用
        Inner inner = new Inner();
        inner.innerShow();
    }
}
```

## 匿名内部类
匿名内部类是没有访问修饰符的
使用匿名内部类时，这个new之后的类首先是要存在的，其次要重写new后的类的某个或某些方法
匿名内部类访问方法参数时也有和局部内部类同样的限制
匿名内部类没有构造方法

```java
public interface IAnimal{
	void speak();
}

public class Outer {
    public static IAnimal getInnerInstance(String speak){
        return new IAnimal(){
            public void speak(){   System.out.println(speak);}
        };
    }
}
```

# 异常
xception这种异常又分为两类：运行时异常和编译时异常。
除了Exception中的 RuntimeException 及其子类以外，其他的 Exception类及其子类异常就是非运行时期异常都属于受检异常

Java标准库内建了一些通用的异常，这些类以Throwable为顶层父类
+ Error:  JVM系统内部错误、资源耗尽等严重情况，程序无法恢复的异常情况
  + VirtualMachineError： OutOfMemoryError、UnknownError 、StackOverflowError、InternalError
  + LinkageError： NoClassDefFoundError、BootstrapMethodError 、ClassFormatError
  + AWTError： 发生严重的Abstract Window Toolkit错误时抛出
  + IOError：发生严重I / O错误时抛出
+ Exception: 其它因编程错误或偶然的外在因素导致的一般性问题
  + RuntimeException：是在Java虚拟机的正常操作期间可以抛出的那些异常的超类。是未经检查的异常
    + NullPointerException  、IndexOutOfBoundsException 
    + ArithmeticException、BufferOverflowException、ClassCastException 
  + ReflectiveOperationException：核心反射中反射操作抛出的常见超类异常，ClassNotFoundException
  + IOException

```java
try {
	......	//可能产生异常的代码
}
catch( ExceptionName1 e ) {
	......	//当产生ExceptionName1型异常时的处置措施
}
catch( ExceptionName2 e ) {
...... 	//当产生ExceptionName2型异常时的处置措施
}  
[ finally{
......	 //无条件执行的语句
		}  ]

public void readFile(String file)  throws FileNotFoundException {}
throw new String("java");
```

# 枚举
使用 enum 定义的枚举类默认继承了 java.lang.Enum 类
枚举类的构造器只能使用 private 访问控制符
枚举类的所有实例必须在枚举类中显式列出(, 分隔    ; 结尾). 列出的实例系统会自动添加 public static final 修饰
所有的枚举类都提供了一个 values 方法, 该方法可以很方便地遍历所有的枚举值

```java
public enum Type {
    TYPE("TYPE"),
    FIELD("FIELD");
    private String name;
    PlayerType(String name) {
        this.name = name;
    }
}
```

# 注解
Annotation 其实就是代码里的特殊标记, 这些标记可以在编译, 类加载, 运行时被读取, 并执行相应的处理.
Annotation 可以像修饰符一样被使用, 可用于修饰 包,类, 构造器, 方法, 成员变量, 参数, 局部变量的声明

三个基本的 Annotation:
+ `@Override`: 限定重写父类方法, 该注释只能用于方法
+ `@Deprecated`: 用于表示某个程序元素(类, 方法等)已过时
+ `@SuppressWarnings`: 抑制编译器警告

注解的生命周期:
+ SOURCE：在源文件中有效，被编译器丢弃
+ CLASS：在编译器生成的字节码文件中有效，但在运行时会被处理类文件的 JVM 丢弃
+ RUNTIME：在运行时有效。它允许程序通过反射的方式访问注解，并根据注解的定义执行相应的代码

ElementType 11 种: 
+ `TYPE`：用于类、接口、注解、枚举
+ `FIELD`：用于字段（类的成员变量），或者枚举常量
+ `METHOD`：用于方法
+ `PARAMETER`：用于普通方法或者构造方法的参数
+ `CONSTRUCTOR`：用于构造方法
+ `LOCAL_VARIABLE`：用于变量
+ `ANNOTATION_TYPE`：用于注解
+ `PACKAGE`：用于包
+ `TYPE_PARAMETER`：用于泛型参数
+ `TYPE_USE`：用于声明语句、泛型或者强制转换语句中的类型
+ `MODULE`：用于模块

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface JsonField {
    public String value() default "";
}

```

# 常用类

## 包装类
Integer Character
常量缓存池: `Integer.valueOf();` Byte、Short、Integer、Long(-128~127), Character(\u0000 - \u007F) Boolean：true 和 false

```java
public static Integer valueOf(int i) {
    if (i >=IntegerCache.low && i <=IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

# JNI
Java Native Interface
+ 标准的 Java 类库不支持
+ 已经用另一种语言，比如说 C/C++ 编写了一个类库，如何用 Java 代码调用呢？
+ 某些运行次数特别多的方法，为了加快性能，需要用更接近硬件的语言（比如汇编）编写

使用
+ 编写带有 native 方法的 Java 类， `javac` 生成.class 文件；
+ 使用 `javah -jni `j生成扩展名为 h 的头文件
+ 实现本地方法，并生成动态链接库


```java
public class JNI {
    static {
        System.loadLibrary("xx"); // 加载名为 xxx 的动态链接库
    }
    // 定义本地方法
    private native void test();
}
```

# IO
+ 字节流
+ 字符流
+ 缓存流
+ 对象流