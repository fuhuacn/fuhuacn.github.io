---
layout: post
title: Java 基础
categories: Knowledge
description: Java 基础
keywords: Java, 基础
---
[参考来源](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java%20基础.md)

目录

* TOC
{:toc}

# 零、JAVA 即是解释型语言又是编译形语言

JAVA 语言是一种编译型-解释型语言，同时具备编译特性和解释特性（其实，确切的说 java 就是解释型语言，其所谓的编译过程只是将 java 文件编程成平台无关的字节码 .class 文件，并不是向 C 一样编译成可执行的机器语言，在此请读者注意 Java 中所谓的“编译”和传统的“编译”的区别）。作为编译型语言，JAVA 程序要被统一编译成字节码文件——文件后缀是 class。此种文件在 java 中又称为类文件。java 类文件不能再计算机上直接执行，它需要被 java 虚拟机翻译成本地的机器码后才能执行，而 java 虚拟机的翻译过程则是解释性的。java 字节码文件首先被加载到计算机内存中，然后读出一条指令，翻译一条指令，执行一条指令，该过程被称为 java 语言的解释执行，是由 java 虚拟机完成的。而在现实中，java 开发工具 JDK 提供了两个很重要的命令来完成上面的编译和解释（翻译）过程。两个命令分别是 javac.exe 和 java.exe，前者加载 java类文件，并逐步对字节码文件进行编译，而另一个命令则对应了 java 语言的解释（java.exe）过程。在次序上，java 语言是要先进行编译的过程，接着解释执行。

# 一、数据类型

## 基本类型

除了 boolean 单位为 bit，其他都是 byte。

boolean/1 bit  
byte/8 bit/1 byte  
char/16 bit/2 byte  
short/16 bit/2 byte  
int/32 bit/4 byte  
float/32 bit/4 byte  
long/64 bit/8 byte  
double/64 bit/8 byte

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

## 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

``` java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

## 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

+ new Integer(123) 每次都会新建一个对象；
+ Integer.valueOf(123) 会使用缓存池中 Integer 对象，多次调用会取得同一个对象的引用。

``` java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
int j = 123;
System.out.println(z == k);   // true
System.out.println(z == j);   // true

System.out.println(x == j);   // true // 自动拆箱了
System.out.println(z == x);   // false // 因为 valueOf 方法返回的是 Integer，所以不等
```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

``` java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

``` java
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

``` java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true

Integer m = 128;
Integer n = 128;
System.out.println(m == n); // false
```

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。

# 二、String

## 概览

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

在 Java 8 中，String 内部使用 char 数组存储数据。

``` java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。

``` java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## 不可变得好处

### 1. 可以缓存 hash 值

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

### 2. String Pool 的需要

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

### 3. 安全性

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。

### 4. 线程安全

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## String, StringBuffer and StringBuilder

### 1. 可变性

- String 不可变
- StringBuffer 和 StringBuilder 可变

### 2. 线程安全

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

## String Pool

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得同一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

``` java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
```

如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

``` java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

## new String("abc")

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；
- 而使用 new 的方式会在堆中创建一个字符串对象。

创建一个测试类，其 main 方法中使用这种方式来创建字符串对象。

``` java
public class NewStringTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
```

``` java
// ...
Constant pool:
// ...
   #2 = Class              #18            // java/lang/String
   #3 = String             #19            // abc
// ...
  #18 = Utf8               java/lang/String
  #19 = Utf8               abc
// ...

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
// ...
```

在 Constant Pool 中，#19 存储这字符串字面量 "abc"，#3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

``` java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

# 三、运算

## 参数传递

Java 的参数是以值传递的形式传入方法中，而不是引用传递。

以下代码中 Dog dog 的 dog 是一个指针，存储的是对象的地址。在将一个参数传入一个方法时，本质上是将对象的地址以值的方式传递到形参中。

``` java
public class Dog {

    String name;

    Dog(String name) {
        this.name = name;
    }

    String getName() {
        return this.name;
    }

    void setName(String name) {
        this.name = name;
    }

    String getObjectAddress() {
        return super.toString();
    }
}
```

在方法中改变对象的字段值会改变原对象该字段值，因为引用的是同一个对象。

``` java
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());          // B
    }

    private static void func(Dog dog) {
        dog.setName("B");
    }
}
```

但是在方法中将指针引用了其它对象，那么此时方法里和方法外的两个指针指向了不同的对象，在一个指针改变其所指向对象的内容对另一个指针所指向的对象没有影响。

``` java
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());          // A
    }

    private static void func(Dog dog) {
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());          // B
    }
}
```

## float 与 double

Java 不能隐式执行向下转型，因为这会使得精度降低。

1.1 字面量属于 double 类型，不能直接将 1.1 直接赋值给 float 变量，因为这是向下转型。

``` java
// float f = 1.1; 这样是错的哈
```

``` java
float f = 1.1f;
```

## 隐式类型转换

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型。

``` java
short s1 = 1;
// s1 = s1 + 1; 这里 1 就是 int，所以 + 1 就是 int 了，s1 是 short 不能等于 int。
```

但是使用 += 或者 ++ 运算符会执行隐式类型转换。

``` java
s1 += 1;
s1++; // 这俩都 OK
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

``` java
s1 = (short) (s1 + 1);
```

所以在 java 中很少用 float，short，byte 等复制数字，因为默认的 1 或 1.1 都是 int 和 double，也会隐式转换了。

## switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。

``` java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```

switch 不支持 long，是因为 switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

# 四、关键字

## final

### 1. 数据

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

``` java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1; //这个被引用的对象还能改
```

### 2. 方法

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

### 3. 类

声明类不允许被继承。

## static

### 1. 静态变量

- 静态变量（类变量）：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
- 实例变量（对象/成员变量）：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

``` java
public class A {

    private int x;         // 实例变量
    private static int y;  // 静态变量

    public static void main(String[] args) {
        // int x = A.x;  // Non-static field 'x' cannot be referenced from a static context
        A a = new A();
        int x = a.x;
        int y = A.y;
    }
}
```

### 2. 静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

``` java
public abstract class A {
    public static void func1(){
    }
    // 静态方法不能是抽象方法，下面这种不行
    // public abstract static void func2();  // Illegal combination of modifiers: 'abstract' and 'static'
}
```

只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因此这两个关键字与具体对象关联。

``` java
public class A {

    private static int x;
    private int y;

    public static void func1(){
        int a = x;
        // 静态方法都是类，这时候没有对象怎么可能有 this 和 super
        // int b = y;  // Non-static field 'y' cannot be referenced from a static context
        // int b = this.y;     // 'A.this' cannot be referenced from a static context
    }
}
```

### 3. 静态语句块

静态语句块在类初始化时运行一次。

``` java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
        // 仅输出一个 123
    }
}
```

### 4. 静态内部类

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类，因为非静态的就是对象的，所以有了对象才有它。而静态内部类不需要，他是类的可以直接用。

``` java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

静态内部类不能访问外部类的非静态的变量和方法。

### 5. 静态导包

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

``` java
import static com.xxx.ClassName.*
```

### 6. 初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。以下是执行顺序：

``` java
public static String staticField = "静态变量";
```

``` java
static {
    System.out.println("静态语句块");
}
```

``` java
public String field = "实例变量";
```

``` java
{
    System.out.println("普通语句块");
}
```

最后才是构造函数的初始化。

``` java
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

# 五、Object 通用方法

## 概览

通常会复写很多方法。

``` java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## equals()

### 1. 等价关系

大多数时候，大部分的封装类中，都重写了Object类的这个方法。两个对象具有等价关系，需要满足以下五个条件：

- 自反性：x.equals(x); // true
- 对称性：x.equals(y) == y.equals(x); // true
- 传递性：if (x.equals(y) && y.equals(z)) x.equals(z); // true;
- 一致性：多次调用 equals() 方法结果不变
- 与 null 的比较：对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

### 2. 等价与相等

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

``` java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true Integer 复写了 equals 方法，所以变成了判断值相等。
System.out.println(x == y);      // false
```

### 3. 等价实现

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

``` java
public class EqualExample {

    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

## hashCode()

hashCode() 返回哈希值，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值（哈希值）一定相同，但是散列值相同的两个对象不一定等价，这是因为计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象哈希值也相等。

HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。

下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象。但是 EqualExample 没有实现 hashCode() 方法，因此这两个对象的哈希值是不同的，最终导致集合添加了两个等价的对象。

``` java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2，所以就是两个不同的 hash 值，这是不对的
```

理想的哈希函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求了哈希函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。

R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。并且一个数与 31 相乘可以转换成移位和减法：31*x == (x<<5)-x，编译器会自动进行这个优化。

``` java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

## toString()

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

## clone()

### 1. cloneable

clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

``` java
public class CloneExample {
    private int a;
    private int b;
}
```

``` java
CloneExample e1 = new CloneExample();
// protected 方法没法直接调用，因为他是 Object package 的方法。
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```

重写 clone() 得到以下实现：

``` java
public class CloneExample {
    private int a;
    private int b;

    @Override
    public CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
```

``` java
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
// 会抛出 java.lang.CloneNotSupportedException: CloneExample
```

以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没有实现 Cloneable 接口（用到了 super.clone()）。

Clonable 是个空接口，仅为标记和 Serizable 一样。

应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

``` java
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

### 2. 浅拷贝

拷贝对象和原始对象的引用类型引用同一个对象。

``` java
public class ShallowCloneExample implements Cloneable {

    private int[] arr;

    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}
```

``` java
ShallowCloneExample e1 = new ShallowCloneExample();
ShallowCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 222，所以说是用的同一个引用
```

### 3. 深拷贝

拷贝对象和原始对象的引用类型引用不同对象。

``` java
public class DeepCloneExample implements Cloneable {

    private int[] arr;

    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}
```

``` java
DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

### 4. clone() 的替代方案

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

``` java
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
```

``` java
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

# 六、继承

## 访问权限

Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加访问修饰符，表示包级可见。

||同一个类|同一个包|不同包的子类|不同包的非子类|
|---|---|---|---|---|
|public|√|√|√|√|
|protected|√|√|√||
|默认|√|√|||
|private|√||||

可以对类或类中的成员（字段和方法）加上访问修饰符。

- 类可见表示其它类可以用这个类创建实例对象。
- 成员可见表示其它类可以用这个类的实例对象访问到该成员；

protected 用于修饰成员，表示在继承体系中成员对于子类可见，**但是这个访问修饰符对于类没有意义。**

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例去代替，也就是确保满足里氏替换原则。

字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。例如下面的例子中，AccessExample 拥有 id 公有字段，如果在某个时刻，我们想要使用 int 存储 id 字段，那么就需要修改所有的客户端代码。

``` java
public class AccessExample {
    public String id;
}
```

可以使用公有的 getter 和 setter 方法来替换公有字段，这样的话就可以控制对字段的修改行为。

``` java
public class AccessExample {

    private int id;

    public String getId() {
        return id + "";
    }

    public void setId(String id) {
        this.id = Integer.valueOf(id);
    }
}
```

但是也有例外，如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。

``` java
public class AccessWithInnerClassExample {

    private class InnerClass {
        int x;
    }

    private InnerClass innerClass;

    public AccessWithInnerClassExample() {
        innerClass = new InnerClass();
    }

    public int getValue() {
        return innerClass.x;  // 直接访问
    }
}
```

## 抽象类与接口

### 1. 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。

``` java
public abstract class AbstractClassExample {

    protected int x;
    private int y;

    public abstract void func1();

    public void func2() {
        System.out.println("func2");
    }
}
```

``` java
public class AbstractExtendClassExample extends AbstractClassExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}
```

``` java
// AbstractClassExample ac1 = new AbstractClassExample(); // 'AbstractClassExample' 是抽象的，不能实例化
AbstractClassExample ac2 = new AbstractExtendClassExample();
ac2.func1();
```

### 2. 接口

接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。

从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法。

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。

``` java
public interface InterfaceExample {

    void func1();

    default void func2(){
        System.out.println("func2");
    }

    int x = 123;
    // int y;               // Variable 'y' might not have been initialized
    public int z = 0;       // Modifier 'public' is redundant for interface fields
    // private int k = 0;   // Modifier 'private' not allowed here
    // protected int l = 0; // Modifier 'protected' not allowed here
    // private void fun3(); // Modifier 'private' not allowed here
}
```

``` java
public class InterfaceImplementExample implements InterfaceExample {
    @Override
    public void func1() {
        System.out.println("func1");
    }
}
```

``` java
// InterfaceExample ie1 = new InterfaceExample(); // 'InterfaceExample' is abstract; cannot be instantiated
InterfaceExample ie2 = new InterfaceImplementExample();
ie2.func1();
System.out.println(InterfaceExample.x);
```

### 3. 比较

- 从设计层面上看，抽象类提供了一种 IS-A 关系，需要满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 LIKE-A 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 IS-A 关系。
- 从使用上来看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。

### 4. 使用选择

使用接口：

- 需要让**不相关的类**都实现一个方法，例如不相关的类都可以实现 Compareable 接口中的 compareTo() 方法；
- 需要使用多重继承。

使用抽象类：

- 需要在几个**相关的类**中共享代码。
- 需要能控制继承来的成员的访问权限，而不是都为 public。
- 需要继承非静态和非常量字段。

在很多情况下，接口优先于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从 Java 8 开始，接口也可以有默认的方法实现，使得修改接口的成本也变的很低。

## super

- 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。应该注意到，子类一定会调用父类的构造函数来完成初始化工作，一般是调用父类的默认构造函数，如果子类需要调用父类其它构造函数，那么就可以使用 super() 函数。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

**子类必须写父类的构造函数。**

``` java
public class SuperExample {

    protected int x;
    protected int y;

    public SuperExample(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public void func() {
        System.out.println("SuperExample.func()");
    }
}
```

``` java
public class SuperExtendExample extends SuperExample {

    private int z;

    public SuperExtendExample(int x, int y, int z) {
        super(x, y);
        this.z = z;
    }

    @Override
    public void func() {
        super.func();
        System.out.println("SuperExtendExample.func()");
    }
}
```

``` java
SuperExample e = new SuperExtendExample(1, 2, 3);
e.func();
// 输出结果：
// SuperExample.func()
// SuperExtendExample.func()
```

## 重写与重载

### 1. 重写（Override）

存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

为了满足里式替换原则，重写有以下三个限制：

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。
- 
使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件。

下面的示例中，SubClass 为 SuperClass 的子类，SubClass 重写了 SuperClass 的 func() 方法。其中：

- 子类方法访问权限为 public，大于父类的 protected。
- 子类的返回类型为 ArrayList，是父类返回类型 List 的子类。
- 子类抛出的异常类型为 Exception，是父类抛出异常 Throwable 的子类。
- 子类重写方法使用 @Override 注解，从而让编译器自动检查是否满足限制条件。

``` java
class SuperClass {
    protected List<Integer> func() throws Throwable {
        return new ArrayList<>();
    }
}

class SubClass extends SuperClass {
    @Override
    public ArrayList<Integer> func() throws Exception {
        return new ArrayList<>();
    }
}
```

在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)

``` java
/*
    A
    |
    B
    |
    C
    |
    D
 */


class A {

    public void show(A obj) {
        System.out.println("A.show(A)");
    }

    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}

class B extends A {

    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }
}

class C extends B {
}

class D extends C {
}
```

``` java
public static void main(String[] args) {

    A a = new A();
    B b = new B();
    C c = new C();
    D d = new D();

    // 在 A 中存在 show(A obj)，直接调用
    a.show(a); // A.show(A)
    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
    a.show(b); // A.show(A)
    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
    b.show(c); // A.show(C)
    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
    b.show(d); // A.show(C)

    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
    A ba = new B();
    ba.show(c); // A.show(C)
    ba.show(d); // A.show(C)
}
```

### 2. 重载（Overload）

存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。

应该注意的是，返回值不同，其它都相同不算是重载（会直接报错）。

### 分派

*静态分派实现了重载，即根据参数的静态类型决定调用方法。动态分派实现重写，根据调用者的实际类型调用。*

*静态分派在编译器就决定了，动态分派需要在运行期才知道是谁。*

分派会解释多态性特征的一些最基本的体现，**如“重载”、“重写”在Java虚拟机中是如何实现的**，当然这里的实现不是语法上该怎么写，我们关心的是虚拟机**如何确定正确的目标方法**。

#### 静态分派

所有依赖**静态类型来定位方法执行版本的分派动作称为静态分派。**静态分派的典型应用是**方法重载（根据静态类型不同决定执行哪个方法）**。

``` java
Human man = new Man();
```

**如上代码，Human 被称为静态类型，Man 被称为实际类型。**

``` java
//实际类型变化
Human man = new Man();
man = new Woman();

//静态类型变化
StaticDispatch sr = new StaticDispatch();
sr.sayHello((Human) man);
sr.sayHello((Woman) man); //这一步强转后就是实际类型了
```

可以看到的静态类型和实际类型都会发生变化，但是有区别：静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型是在**编译期可知的**，而实际类型变化的结果在**运行期才可确定**。

``` java
class Human {  
}

class Man extends Human {  
}  

class Woman extends Human {  
}  

public class StaticDispatch {  

    public void sayHello(Human guy) {  
        System.out.println("hello, guy!");  
    }  

    public void sayHello(Man guy){  
        System.out.println("hello, gentleman!");  
    }

    public void sayHello(Woman guy){  
        System.out.println("hello, lady!");  
    }  

    public static void main(String[] args){  
        Human man = new Man();  
        Human woman = new Woman();  
        StaticDispatch sr = new StaticDispatch();  
        sr.sayHello(man); // hello, guy!
        sr.sayHello(woman); // hello, guy!
        sr.sayHello((Man)man); // hello, gentleman!
    }  
}
```

如上代码与运行结果，在调用 sayHello() 方法时，方法的调用者都为 sr 的前提下，使用哪个重载版本，**完全取决于传入参数的数量和数据类型**。代码中刻意定义了两个静态类型相同、实际类型不同的变量，可见编译器（不是虚拟机，因为如果是根据静态类型做出的判断，那么**在编译期就确定了**）在重载时是通过参数的静态类型而不是实际类型作为判定依据的。并且静态类型是编译期可知的，所以在编译阶段，javac 编译器就根据参数的静态类型决定使用哪个重载版本。这就是静态分派最典型的应用。

#### 动态分派

``` java

abstract class Human {
    abstract void call();
}
 
class Father extends Human{
    @Override
    void call() {
        System.out.println("I am the Father!");
    }
}
 
class Mother extends Human{
    @Override
    void call() {
        System.out.println("I am the Mother!");
    }
}
 
public class DynamicDispatch {
    public static void main(String[] args) {
        Human father = new Father();
        Human mother = new Mother();
        father.call(); // I am the Father!
        mother.call(); // I am the Mother!
    }
}
```

动态分派与多态性的另一个重要体现——**方法重写**有着很紧密的关系。

程序执行过程的机器指令分析如下：

![动态分派](/images/posts/knowledge/jvm/20190716175713838.png)

各操作指令解析:

- 0：在 java 堆中为变量 father 分配空间，并将地址压入操作数栈顶
- 3：复制操作数栈顶值。并压入栈顶(此时操作栈上有两个连续相同的father对象地址)
- 4：从操作栈顶弹出一个 this 的引用(即两个连续father对象地址中靠近栈顶的一个)，并调用实例化方法&lt;init&gt;:()v
- 7：将栈顶的仅剩的一个 father 对象地址存入第二个本地变量表 slot(1) 中
- 8~15：重复上面的操作,创建了 mother 对象并将其地址存入第三个本地变量表 slot(2) 中
- 16：将第二个本地变量表 slot(1) 中引用类型数据 father 地址推送至操作栈顶
- 17：调用虚方法，根据father对象地址查询其 call() 方法并执行
- 20~21：重复上面的操作,根据mother对象地址查询其 call() 并执行
- 24：结束方法

总结：从上面的invokevirtual可以知道方法call()的符号引用转换是在运行时期完成的,所以可以说动态分派解释了重载

#### 单分派与多分派

先给出宗量的定义：**方法的接受者（亦即方法的调用者）与方法的参数统称为方法的宗量。**单分派是根据一个宗量对目标方法进行选择，多分派是根据多于一个宗量对目标方法进行选择。

``` java
class Eat {  
}

class Drink {  
}  

class Father {  
    public void doSomething(Eat arg) {  
        System.out.println("爸爸在吃饭");  
    }  

    public void doSomething(Drink arg) {  
        System.out.println("爸爸在喝水");  
    }  
}  

class Child extends Father {
    public void doSomething(Eat arg) {  
        System.out.println("儿子在吃饭");  
    }  

    public void doSomething(Drink arg) {  
        System.out.println("儿子在喝水");  
    }  
}  

public class SingleDoublePai {  
    public static void main(String[] args) {  
        Father father = new Father();  
        Father child = new Child();  
        father.doSomething(new Eat()); // 爸爸在吃饭
        child.doSomething(new Drink()); // 儿子在喝水
    }  
}
```

我们首先来看编译阶段编译器的选择过程，即静态分派过程。这时候选择目标方法的依据有两点：一是方法的**接受者（即调用者）的静态类型**是 Father 还是 Child，二是**方法参数类型**是 Eat 还是 Drink。因为是根据两个宗量进行选择，所以 Java 语言的静态分派属于**多分派类型**。

再来看运行阶段虚拟机的选择，即动态分派过程。由于编译期已经了确定了目标方法的参数类型（编译期根据**参数的静态类型进行静态分派**），因此唯一可以影响到虚拟机选择的因素只有此方法的**接受者的实际类型是 Father 还是 Child**。因为只有一个宗量作为选择依据，所以 Java 语言的动态分派属于单分派类型。

根据以上论证，我们可以总结如下：目前的 Java 语言（JDK1.6）是一门**静态多分派（方法重载）、动态单分派（方法重写）**的语言。

# 七、反射

每个类都有一个 Class 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 Class.forName("com.mysql.jdbc.Driver") 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- Constructor ：可以用 Constructor 的 newInstance() 创建新的对象。

## 代码示例

``` java
package reflection.bean;

/**
 * @Author: fuhua
 * @Date: 2019/9/2 4:27 下午
 */
public class Student {
    public static void main(String[] args) {
        System.out.println("我执行了！！！");
    }
    private String name="default";
    public int score;

    private Student(int score){
        this.score = score;
        System.out.println("调用了私有方法执行了。。。");
    }

    public Student(){
        System.out.println("调用了公有、无参构造方法执行了。。。");
    }

    Student(String str){
        System.out.println("(默认)的构造方法 s = " + str);
    }

    protected Student(boolean n){
        System.out.println("受保护的构造方法 n = " + n);
    }

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    private String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    private int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }
}
```

``` props
className=reflection.bean.Student
methodName=getName
```

### 三种 Class 的获取方式

``` java
package reflection.run;

import reflection.bean.Student;

/**
 * @Author: fuhua
 * @Date: 2019/9/2 4:31 下午
 */
public class Create3Class {
    public static void main(String[] args) {
        Student stu1 = new Student();
        Class stuClass = stu1.getClass();
        System.out.println(stuClass.getName());
        Class stuClass2 = Student.class;
        Class stuClass3 = null;
        try {
            stuClass3 = Class.forName("reflection.bean.Student");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        if (stuClass == stuClass2 && stuClass2 == stuClass3) {
            System.out.println("True");
        }
    }
}
```

### 构造器

``` java
package reflection.run;

import reflection.bean.Student;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

/**
 * @Author: fuhua
 * @Date: 2019/9/2 4:39 下午
 */
public class Constructors {
    public static void main(String[] args) {
        Class stuClass = Student.class;
        System.out.println("**********************所有公有构造方法*********************************");
        Constructor[] constructors = stuClass.getConstructors();
        for (Constructor constructor:constructors){
            System.out.println(constructor);
        }

        System.out.println("**********************所有构造方法*********************************");
        constructors = stuClass.getDeclaredConstructors();
        for (Constructor constructor:constructors){
            System.out.println(constructor);
        }

        System.out.println("**********************带参数的*********************************");
        Constructor con = null;
        try {
            con = stuClass.getDeclaredConstructor(String.class);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
        System.out.println(con);

        System.out.println("**********************调用私有构造方法*********************************");
        constructors = stuClass.getDeclaredConstructors();
        for (Constructor constructor:constructors){
            if(constructor.getModifiers() == 2){
                try {
                    constructor.setAccessible(true);
                    Object obj = constructor.newInstance(10);

                } catch (InstantiationException e) {
                    e.printStackTrace();
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}
```

### 反射方法

``` java
package reflection.run;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @Author: fuhua
 * @Date: 2019/9/2 7:53 下午
 */
public class CreateMethod {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Class clazz = null;
        try {
            clazz = Class.forName("reflection.bean.Student");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }

        Method[] methods = clazz.getMethods();
        for (Method method:methods){
            System.out.println(method);
        }


        Constructor con = clazz.getDeclaredConstructor(int.class);
        con.setAccessible(true);
        Object stu = con.newInstance(1);


        try {
            Method getScore = clazz.getDeclaredMethod("getScore");
            getScore.setAccessible(true);
            System.out.println(getScore.invoke(stu));
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }

        Method setName = clazz.getMethod("setName",String.class);
        setName.invoke(stu,"fuhua");

        Method getName = clazz.getDeclaredMethod("getName");
        getName.setAccessible(true);
        Object name = getName.invoke(stu);
        System.out.println(name);

        Method main = clazz.getMethod("main",String[].class);
        main.invoke(null,(Object)new String[]{"a","b","c"});

    }
}
```

### 反射变量

``` java
package reflection.run;

import reflection.bean.Student;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

/**
 * @Author: fuhua
 * @Date: 2019/9/2 8:36 下午
 */
public class CreateFields {
    public static void main(String[] args) {
        Class stu = Student.class;
        Constructor constructor = null;
        try {
            constructor = stu.getConstructor();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
        Object student = null;
        try {
            student = constructor.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        Field[] fields = stu.getFields();
        for (Field field:fields){
            System.out.println(field);
        }
        Field name = null;
        try {
            name = stu.getDeclaredField("name");
            name.setAccessible(true);
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        try {
            name.set(student,"fuhua");
            System.out.println(name.get(student));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

**反射的优点：**

- 可扩展性：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
- 类浏览器和可视化开发环境：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
- 调试器和测试工具： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

**反射的缺点：**

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- 性能开销：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
- 安全限制：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
- 内部暴露：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

# 八、异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

- 受检异常 ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

下面代码中 ceshi() 抛出了异常并将异常扔给了上一层调用者也就是 main() 处理。main() 抛出异常给 JVM。

``` java
class Solution {
    public static void main(String[] args) throws Exception{
        Solution s = new Solution();
        s.ceshi();
    }
    public void ceshi() throws Exception{
        System.out.println(this);
        throw new Exception("test");
    }
}
```

``` text
Solution@7852e922
Exception in thread "main" java.lang.Exception: test
        at Solution.ceshi(Solution.java:15)
        at Solution.main(Solution.java:11)
```

# 九、范型

## <T>

范型可以在方法、类、静态方法中设置。**静态方法必须设置专门的范型。**

``` java
package generic;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: fuhua
 * @Date: 2019/9/9 3:36 下午
 */
public class GenericClass<T> {
    private List<T> key = new ArrayList<T>();

    void add(T t){
        key.add(t);
    }

    List<T> getKey(){
        return key;
    }

    // 范型方法，可以用 <> 方式设置与范型类不同的字母
    public <N> List<N> methodGeneric(N string){
        List<N> res = new ArrayList<>();
        res.add(string);
        return res;
    }

    // 静态范型方法，必须设置范型，因为静态的属于 .class 的还没有对象
    public static <S> void print(S s){
        System.out.println(s);
    }
}
```

范型可以向下继承，例如下面的例子 GoodStudent 继承 Student。

``` java
Student stu1 = new Student("aaa",100);
GoodStudent stu2 = new GoodStudent("bbb",95);
GenericClass<Student> genericClass = new GenericClass<Student>(); //这个地方如果范型里是 GoodStudent 就会报错了
genericClass.add(stu1);
genericClass.add(stu2);
```

范型接口，同时在实现的类中也可以设定范型的继承要求。

``` java
public interface GenericInterface<T> {
    public void print(T t);
}

public class GenericImplentation<T extends Student> implements GenericInterface<T> {

    @Override
    public void print(T t) {
        System.out.println(t);
    }
}
```

## <?>

**声明 ? 的地方，里面的元素要把 ? extends/super T 当作一个整体。? 可以理解为可能的某一个。List<? extends Fruit> 指的是里面的元素是一个继承自 Fruit 的元素，但继承千变万化所以就不能加了。List<? super Fruit> 指得是里面元素是 Fruit 的父类，所以加 Fruit 或 Fruit 的子类还是那个父类的子类。里面还可以添加元素**

### 上界通配符

上届：用 extends 关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的子类。

List<Number> list = ArrayList<Integer> 这样的语句是无法通过编译的，尽管 Integer 是 Number 的子类型。那么如果我们确实需要建立这种 “向上转型” 的关系怎么办呢？这就需要通配符来发挥作用了。

利用 <? extends Fruit> 形式的通配符，可以实现泛型的向上转型：

``` java
public class GenericsAndCovariance {
    public static void main(String[] args) {
        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<>();
        // Compile Error: can’t add any type of object:
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());
        flist.add(null); // Legal but uninteresting
        // We know that it returns at least Fruit:
        Fruit f = flist.get(0);
    }
}
```

上面的例子中， flist 的类型是 List<? extends Fruit>，我们可以把它读作：一个类型的 List， 这个类型可以是继承了 Fruit 的某种类型。注意，这并不是说这个 List 可以持有 Fruit 的任意类型。通配符代表了一种特定的类型，它表示 “某种特定的类型，但是 flist 没有指定”。这样不太好理解，具体针对这个例子解释就是，flist 引用可以指向某个类型的 List，只要这个类型继承自 Fruit，可以是 Fruit 或者 Apple，比如例子中的 new ArrayList<Apple>，但是为了向上转型给 flist，flist 并不关心这个具体类型是什么。

如上所述，通配符 List<? extends Fruit> 表示某种特定类型 ( Fruit 或者其子类 ) 的 List，但是并不关心这个实际的类型到底是什么，反正是 Fruit 的子类型，Fruit 是它的上边界。那么对这样的一个 List 我们能做什么呢？其实如果我们不知道这个 List 到底持有什么类型，怎么可能安全的添加一个对象呢？在上面的代码中，**向 flist 中添加任何对象，无论是 Apple 还是 Orange 甚至是 Fruit 对象，编译器都不允许，唯一可以添加的是 null。**所以如果做了泛型的向上转型 (List<? extends Fruit> flist = new ArrayList<Apple>())，**那么我们也就失去了向这个 List 添加任何对象的能力**，即使是 Object 也不行。

另一方面，如果调用某个返回 Fruit 的方法，这是安全的。因为我们知道，在这个 List 中，**不管它实际的类型到底是什么，但肯定能转型为 Fruit，所以编译器允许返回 Fruit。**

了解了通配符的作用和限制后，好像任何接受参数的方法我们都不能调用了。其实倒也不是，看下面的例子：

``` java
public class CompilerIntelligence {
    public static void main(String[] args) {
        List<? extends Fruit> flist =
        Arrays.asList(new Apple());
        Apple a = (Apple)flist.get(0); // No warning
        flist.contains(new Apple()); // Argument is ‘Object’
        flist.indexOf(new Apple()); // Argument is ‘Object’
        
        //flist.add(new Apple());   无法编译

    }
}
```

### 下界通配符

下界: 用 super 进行声明，表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object

通配符的另一个方向是　“超类型的通配符“: ? super T，T 是类型参数的下界。使用这种形式的通配符，我们就可以 ”传递对象” 了。还是用例子解释：

``` java
public class SuperTypeWildcards {
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new BigApple());
        // apples.add(new Fruit()); // Error
    }
}
```

writeTo 方法的参数 apples 的类型是 List<? super Apple>，它表示某种类型的 List，这个类型是 Apple 的基类型。也就是说，我们不知道实际类型是什么，但是这个类型肯定是 Apple 的父类型。因此，我们可以知道向这个 List 添加一个 Apple 或者其子类型的对象是安全的，这些对象都可以向上转型为 Apple。但是我们不知道加入 Fruit 对象是否安全，因为那样会使得这个 List 添加跟 Apple 无关的类型。

### 无边界通配符

还有一种通配符是无边界通配符，它的使用形式是一个单独的问号：List<?>，也就是没有任何限定。不做任何限制，跟不用类型参数的 List 有什么区别呢？

List<?> list 表示 list 是持有某种特定类型的 List，但是不知道具体是哪种类型。那么我们可以向其中添加对象吗？当然不可以，因为并不知道实际是哪种类型，所以不能添加任何类型，这是不安全的。而单独的 List list ，也就是没有传入泛型参数，表示这个 list 持有的元素的类型是 Object，因此可以添加任何类型的对象，只不过编译器会有警告信息。

# 十、代理

## 代理模式介绍

代理模式是一种设计模式，提供了对目标对象额外的访问方式，即**通过代理对象访问目标对象，这样可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。**

简言之，代理模式就是设置一个中间代理来控制访问原目标对象，以达到增强原对象的功能和简化访问方式。

代理模式UML类图：

![UML](/images/posts/knowledge/javaBasic/代理UML.png)

## 静态代理

这种代理方式需要**代理对象和目标对象实现一样的接口**。都写为静态的代码。

优点：可以在不修改目标对象的前提下扩展目标对象的功能。

缺点：

1. 冗余。由于代理对象要实现与目标对象一致的接口，会产生过多的代理类。
2. 不易维护。一旦接口增加方法，目标对象与代理对象都要进行修改。

举例：保存用户功能（UserDao）的静态代理实现

- 接口类：IUserDao
  
  ``` java
  public interface IUserDao {
      public void save();
  }
  ```

- 目标对象：UserDao

  ``` java
  public class UserDao implements IUserDao{

      @Override
      public void save() {
          System.out.println("保存数据");
      }
  }
  ```

- 静态代理对象：UserDapProxy 需要实现IUserDao接口！

  ``` java
  public class UserDaoProxy implements IUserDao{
      // 实现相同对象的接口才能赋值，与装饰器类似
      private IUserDao target;
      public UserDaoProxy(IUserDao target) {
          this.target = target;
      }
      
      @Override
      public void save() {
          System.out.println("开启事务");//扩展了额外功能
          target.save();
          System.out.println("提交事务");
      }
  }
  ```

- 测试类

  ``` java
  import org.junit.Test;

  public class StaticUserProxy {
      @Test
      public void testStaticProxy(){
          //目标对象
          IUserDao target = new UserDao();
          //代理对象
          UserDaoProxy proxy = new UserDaoProxy(target);
          proxy.save();
      }
  }
  ```

## JDK 动态代理

*使用 Proxy.newProxyInstance 生成接口，主要需要实现方法中的 InvocationHandler 接口。该接口需要实现 invoke 方法。*

动态代理利用了JDK API，**动态地在内存中构建代理对象，从而实现对目标对象的代理功能。**动态代理又被称为 JDK 代理或接口代理。

### 与静态代理区别

静态代理与动态代理的区别主要在：

- 静态代理在编译时就已经实现，编译完成后代理类是一个实际的 class 文件
- 动态代理是在**运行时动态生成的**，即编译完成后没有实际的 class 文件，而是在运行时动态生成类字节码，并加载到 JVM 中

**特点**：**动态代理对象不需要实现接口**，但是要求**目标对象必须实现接口**，否则不能使用动态代理。

### 主要类

JDK 中生成代理对象主要涉及的类有：

+ java.lang.reflect.Proxy，主要方法为

  ``` java
  static Object newProxyInstance(ClassLoader loader,  //指定当前目标对象使用类加载器
   Class<?>[] interfaces,    //目标对象实现的接口的类型
   InvocationHandler h      //事件处理器
  ) 
  //返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
  ```

  该方法会返回代理的类的接口

+ java.lang.reflect.InvocationHandler，主要方法为

  ``` java
  Object    invoke(Object proxy, Method method, Object[] args) // 在代理实例上处理方法调用并返回结果。
  ```

  该方法返回的是代理类方法的返回值。

### 举例

jdk 动态代理实现

+ 接口类： IUserDao

  ``` java
  public interface IUserDao {
      public String find(String name);
  }
  ```

+ 目标对象：UserDao，需要实现接口

  ``` java
  public class UserDao implements IUserDao {
      @Override
      public String find(String name) {
          System.out.println("程序运行中！");
          return name;
      }
  }
  ```

+ 动态代理对象，可通用的工厂类

  ``` java
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Method;
  import java.lang.reflect.Proxy;

  public class ProxyFactory {
      Object userDao;
      public ProxyFactory(Object userDao){
          this.userDao = userDao;
      }
      // 返回的是代理对象的接口
      public Object getDaoProxy(){
          return Proxy.newProxyInstance(userDao.getClass().getClassLoader(), userDao.getClass().getInterfaces(), new InvocationHandler() {
              @Override
              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                  System.out.println("开启代理");

                  // 执行目标对象方法
                  Object returnValue = method.invoke(userDao, args);

                  System.out.println("结束代理");
                  return returnValue;
              }
          });
      }
  }
  ```

+ 测试类

  ``` java
  public class Test {
      public static void main(String[] args) {
          ProxyFactory proxyFactory = new ProxyFactory(new UserDao());
          IUserDao iUserDao = (IUserDao) proxyFactory.getDaoProxy();
          System.out.println(iUserDao.find("程序返回值！"));
      }
  }
  ```

  输出为：

  ``` java
  开启代理
  程序运行中！
  结束代理
  程序返回值！
  ```

## cglib 代理

*主要实现 MethodInterceptor 中的 intercept 方法，返回的直接是代理类本身*

cglib（Code Generation Library）是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。

### 特点：

- JDK 的动态代理有一个限制，就是使用动态代理的对象必须实现一个或多个接口。**如果想代理没有实现接口的类，就可以使用 CGLIB 实现。**
- CGLIB 是一个强大的高性能的**代码生成包，它可以在运行期扩展 Java 类与实现 Java 接口。**它广泛的被许多 AOP 的框架使用，例如 Spring AOP 和 dynaop，为他们提供方法的 interception（拦截）。
- CGLIB 包的底层是通过使用一个小而快的字节码处理框架 ASM，来转换字节码并生成新的类。不鼓励直接使用 ASM，因为它需要你对 JVM 内部结构包括 class 文件的格式和指令集都很熟悉。

### 与 jdk 动态代理的区别

- 使用动态代理的对象必须实现一个或多个接口
- 使用cglib代理的对象则**无需实现接口，达到代理类无侵入**。

使用 cglib 需要引入 cglib 的 jar 包，如果你已经有 spring-core 的 jar 包，则无需引入，因为 spring 中包含了 cglib。

### 举例

+ 目标对象，无需接口

  ``` java
  import proxy.jdk.IUserDao;

  public class UserDao implements IUserDao {
      @Override
      public String find(String name) {
          System.out.println("程序运行中！");
          return name;
      }
  }
  ```

+ 代理对象：ProxyFactory，可通用的工厂类

  ``` java
  import net.sf.cglib.proxy.Enhancer;
  import net.sf.cglib.proxy.MethodInterceptor;
  import net.sf.cglib.proxy.MethodProxy;

  import java.lang.reflect.Method;

  public class ProxyFactory implements MethodInterceptor {
      Object o;
      public ProxyFactory(Object o) {
          this.o = o;
      }

      // 直接获取的是代理后的对象，不是接口了
      public Object getInstance(){
          Enhancer enhancer = new Enhancer();
          enhancer.setSuperclass(o.getClass());
          //Callback 可以理解成生成的代理类的方法被调用时执行的逻辑，即拦截器。我们这里拦截器就是 ProxyFactory，它具有此逻辑
          enhancer.setCallback(this);
          return enhancer.create();
      }

      @Override
      public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
          System.out.println("开始代理");
          // 这块的 o 一定是原始对象的 o，最开始写成了 object 导致一直在代理死循环
          Object res = method.invoke(o,args);
          System.out.println("结束代理");
          return res;
      }
  }
  ```

+ 测试

  ``` java
  public class Test {
      public static void main(String[] args) {
          UserDao userDao = new UserDao();
          UserDao proxyDao = (UserDao)new ProxyFactory(userDao).getInstance();
          System.out.println(proxyDao.find("程序返回值！"));

      }
  }
  ```

## 总结

1. 静态代理实现较简单，只要代理对象对目标对象进行包装，即可实现增强功能，但静态代理只能为一个目标对象服务，如果目标对象过多，则会产生很多代理类。
2. jdk 动态代理**需要目标对象实现业务接口**，代理类只需实现 InvocationHandler 接口。
3. jdk 动态代理生成的类为 com.sun.proxy.$Proxy0，cglib代理生成的类为 proxy.cglib.UserDao$$EnhancerByCGLIB$$44247402。
4. 静态代理在编译时产生 class 字节码文件，可以直接使用，效率高。
5. jdk 动态代理必须实现 InvocationHandler 接口，通过反射代理方法，比较消耗系统性能，但可以减少代理类的数量，使用更灵活。
6. cglib 代理无需实现接口，通过生成类字节码实现代理，比反射稍快，不存在性能问题，但 cglib 会继承目标对象，需要重写方法，所以目标对象不能为 final 类。

# 十一、注解

Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。