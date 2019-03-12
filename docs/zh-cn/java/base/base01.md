# 对象与类

## 对象的创建

比如`Student s = new Student()`实例化一个对象，其实经历了如下几个过程：

1. 将`Student.class`加载到内存中
2. 在**栈内存**中给`s`开辟内存空间。
3. 在**堆内存**给`Student`类申请一个内存空间。
4. 给成员变量进行默认初始化，0 null false...
5. 自定义给成员变量初始化赋值
6. 初始化完毕，把堆内存地址赋值给栈内存的`s`变量

## Main方法剖析

```java
public static void main(String[] args) { ... }
```

* `public`: 公共的，访问权限最大，因为main方法是被JVM调用的。
* `static`：静态的，不需要创建对象，通过类名就能调用，方便JVM调用。
* `void`: 无返回值，因为main方法是被JVM调用的，所以给JVM返回数据没有意义。
* `main`: 常见的方法入口，很多语言的入口方法都是main方法。
* `String[] args`: 字符串数组，是作为命令行参数调用的。

### static关键字

`static`关键字特点：（可以修饰成员变量，也可修饰成员方法）

* 随着类的加载而加载
* 优先于对象存在
* 被类中的所有对象共享
* 可直接通过类名调用

**拓展**

静态方法中没有`this`关键字，因为`this`代表当前方法对象，但`static`优于对象存在，所以在对象还未创建完毕`static`修饰的方法就被调用，此时`this`代表的对象还未创建。

## String

`String`底层定义为`public final class String`，说明`String`是常量，一旦被创建就不能修改。可以查看如Integer Long String这些类的源码：

```java
public final class Integer {}
public final class Long {}
public final class String {}
```

这些**基本类型**，在初始化值、赋值时都是先**从常量池中**取数据，如果常量池中没有该数据，就`new`对象初始化为新数据。

比如常见的一个面试题：

```java
String s = "ab";
s = "abc";
String ss = "ab";
ss = new String("ab");
```

这个`s`和`ss`各自创建了几个对象？答案：`s`创建两个对象；`ss`创建一个对象。因为`s`的常量池中有值`ab`，而重新赋值`s = "abc"`这个`abc`在`s`的常量池中不存在，所以`new String()`创建了一个新对象。`ss`同理分析。可以通过如下方式验证：

```java
String ss = "ab";
System.out.println(ss.hashCode());
ss = "abc";
System.out.println(ss.hashCode());
```

![](http://cdn.tycoding.cn/2019031184233.png)

### StringBuffer

`String`是不可变的字符串，`StringBuffer`是线程安全的可变字符串，用`StringBuffer`做字符串的拼接可以避免资源的浪费，因为`String`每次拼接新的字符串都是创建一个新的String对象。

**String转换为StringBuffer**

```java
//方式一
String s = "hello";
StringBuffer sb = new StringBuffer(s);
//方式二
StringBuffer sb = new StringBuffer();
sb.append(s);
```

**StringBuffer转换成String**

```java
//方式一
StringBuffer sb = new StringBuffer("hello");
String s = new String(sb);
//方式二
String s = sb.toString();
```

### 面试题

> String, StringBuffer, StringBuilder 的区别？

* String的内容不可变，StringBuffer和StringBuilder的内容都可变。
* StringBuffer是线程同步的，数据安全，效率低；String和StringBuilder是线程不同步的，数据不安全，效率高。

> StringBuffer和数组的区别？

* 二者都是一个容器，装其他数据
* 但StringBuffer最终是一个字符串数据；而数组可以存放多种数据，但必须是用一种数据类型。

> String和StringBuffer作为参数传递

* String可理解为**特殊的引用类型**，和基本类型一样，参数传递不会改变原数据内容。
* StringBuffer作为引用类型，基本的赋值不会改变原数据内容，但是调用StringBuffer的方法去改变形式参数就会影响原数据内容。

```java
public class StringBufferDemo {
	public static void main(String[] args) {
		String s1 = "hello";
		String s2 = "world";
		System.out.println(s1 + "---" + s2);// hello---world
		change(s1, s2);
		System.out.println(s1 + "---" + s2);// hello---world

		StringBuffer sb1 = new StringBuffer("hello");
		StringBuffer sb2 = new StringBuffer("world");
		System.out.println(sb1 + "---" + sb2);// hello---world
		change(sb1, sb2);
		System.out.println(sb1 + "---" + sb2);// hello---worldworld

	}

	public static void change(StringBuffer sb1, StringBuffer sb2) {
		sb1 = sb2;
		sb2.append(sb1);
	}

	public static void change(String s1, String s2) {
		s1 = s2;
		s2 = s1 + s2;
	}
}
```



## 参数传递

Java中的参数传递：

* 基本类型：形式参数的改变对实际参数没有影响。
* 引用类型：形式参数的改变直接影响实际参数。

例如：

```java
public class Demo01_Object {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        change(a, b);
        System.out.println("main: a:" + a + ", b:" + b); //10, 20
        int[] arr = {1, 2, 3};
        change(arr);
        System.out.println("main: " + arr[0]); //2
    }

    private static void change(int a, int b) {
        a = b;
        b = a + b;
        System.out.println("change: a:" + a + ", b:" + b); //20, 40
    }

    private static void change(int[] arr) {
        arr[0] = arr[1];
        System.out.println("change" + arr[0]); //2
    }
}
```

**引入概念**： 在Java中**一个对象变量并没有实际包含一个对象，而仅仅引用一个对象**。**所有的Java对象都储存在堆内存中**。例如：`Date t = new Date()`其中的`t`就是一个对象变量，`new Date()`是在堆内存中开辟了一个空间，而`t`指向`new Date()`的堆内存地址。

因此，在上述代码中`a b`都是基本类型，而`int[]`是一个引用类型，那**基本类型形式参数改变对实际参数没有影响**；**对象类型形式参数改变直接影响实际参数**。

![](http://cdn.tycoding.cn/2019031181231.png)

### 总结

**Java程序语言总是采用按值调用**，也就是说，方法得到的是所有参数值的一个拷贝，特别的，方法不能修改传递给他的任何变量的内容。

* 一个方法不能修改一个基本数据类型的参数（即数值型和布尔型）。
* 一个方法可以改变一个对象的引用状态
* 一个方法不能让对象参数引用一个新对象

比如：下列是无意义的：

```java
public static void swap(Employee x, Employee y) {
    Employee temp = x;
    x = y;
    y = temp;
}
```

当调用`swap(e1, e2)`时并不会改变`e1`和`e2`的对象引用，swap方法的参数x,y被初始化为两个对象引用的拷贝，这个方法交换的是这两个拷贝。

* **基本类型**（包括`Integer` `String` `Long`）传递的参数是参数**值**的拷贝；

特别是对于Integer Long String这些类型数据，在初始化、赋值的时候都是从常量池中取数据，比如`IntgerCache` `LongCache`，如果常量池中没有就重新new对象，例如：

```java
public static void main(String[] args) {
    String s = "123";
    System.out.println("main: " + s.hashCode()); //48690
    change(s);
    change2(s);
}
private static void change(String s) {
    s = "123";
    System.out.println("change: " + s.hashCode()); //48690
}
private static change2(String s) {
    s = "456";
    System.out.println("change2: " + s.hashCode()); //51669
}
```

* **引用类型**传递的参数是原对象在**堆内存的地址**的拷贝

对象类型参数的传递，实际上传递这个对象堆内存地址的拷贝，所以形式参数和原参数操作的都是同一个堆内存地址，即形式参数的改变会直接影响原参数。

## 成员变量和局部变量

成员变量和局部变量的区别：

- 在类中的位置不同：
  - 成员变量：在类中方法外
  - 局部变量：在方法定义中或方法声明上
- 在内存中的位置不同：
  - 成员变量：在堆内存
  - 局部变量：在栈内存
- 声明周期不同：
  - 成员变量：随着对象的创建而存在，随着对象的消失而消失。
  - 局部变量：随着方法的调用而存在，随着方法调用完毕而消失
- 初始化值不同：
  - 成员变量：有默认初始化值
  - 局部变量：没有默认初始化值，必须定义、赋值后才能使用

## 构造方法

在Java中，当需要调用构造方法时，**若该类没有定义构造方法，系统会自动提供一个无参构造方法；如果该类定义了构造方法（带参构造），系统将不再提供无参构造，必须手动定义**。举例：

```java
public class Demo2_Construct {
    public static void main(String[] args) {
        Demo2Student student = new Demo2Student();
        student.show();
        // Demo2School school = new Demo2School(); //error
    }
}

class Demo2Student {
    public void show() {
        System.out.println("this student show");
    }
}

class Demo2School {
    private int size = 1000;
    public Demo2School(int size) {
        this.size = size;
    }
}
```

### final

`final`可以修改类、方法、变量。

**特点：**

* `final`可以修饰类，该类不能被继承。
* `final`可以修饰方法，该方法不能被重写。
* `final`可以修饰变量，该变量不能被重新赋值。

**面试题：** `final`修饰局部变量的问题

* 基本类型：被`final`修饰的基本类型的值不能被改变
* 引用类型：引用类型的地址值不能被改变，但是该对象的堆内存地址是可以改变的。

**初始化时机**

被`final`修饰的变量必须在构造方法完毕前被初始化，比如

```java
public class Demo {
    final int WIDTH = 12;
    //final int HEIGHT; //error
    final int AREA;
    {
        AREA = 120;
    }
}
```



## 继承

1. Java支持单继承不支持多继承，但Java支持多层继承
2. 子类只能继承父类非私有成员（成员变量、成员方法）
3. 子类不能继承父类的构造方法，但可以通过`super`关键字访问父类的构造方法。

### 子类和父类的关系

**子类中的所有构造方法都默认访问父类的无参构造方法**。因为子类继承父类，并可能使用父类中的数据，所以子类初始化前一定要完成父类的初始化。所以子类每一个构造方法第一行默认都是`super()`。

```java
public class Demo04_Extends {
    public static void main(String[] args) {
        Demo04Son son = new Demo04Son();
        son.show();
    }
}

class Demo04Son extends Demo04Parent{
    private int num = 10;
    public Demo04Son() {
        super();
    }

    public void show() {
        int num = 100;
        System.out.println(num);
        System.out.println(this.num);
        System.out.println(super.num);
    }
}

class Demo04Parent {
    public int num = 1;

    public Demo04Parent() {
        System.out.println("这是父类的无参构造函数");
    }
}
```

### this-super

`this`和`super`关键字的区别和使用场景？

区别：

* `this`: 代表当前类的对象引用
* `super`: 代表父类的空间标识（可以理解为父类的引用，通过他访问父类的成员）

场景：

* this.成员变量/方法
* super.成员变量/方法
* this(…)  super(…) 

### 加载顺序

```java
public class Demo04_Extends2 {
    public static void main(String[] args) {
        Demo04Zi zi = new Demo04Zi();
    }
}
class Demo04Fu {
    static {
        System.out.println("Fu 静态代码块");
    }
    {
        System.out.println("Fu 构造代码块");
    }
    public Demo04Fu() {
        System.out.println("Fu 构造方法");
    }
}
class Demo04Zi extends Demo04Fu{
    static {
        System.out.println("Zi 静态代码块");
    }
    {
        System.out.println("Zi 构造代码块");
    }
    public Demo04Zi() {
        System.out.println("Zi 构造方法");
    }
}
```

结果：

```
Fu 静态代码块
Zi 静态代码块
Fu 构造代码块
Fu 构造方法
Zi 构造代码块
Zi 构造方法
```

静态代码块 > 构造代码块 > 构造方法

 ### 动态绑定

**调用对象方法的执行过程：**

![](http://cdn.tycoding.cn/20190312122515.png)

1. 编译器首先查看对象的声明类型和方法名。如调用`change(a)`方法，由于存在多个`change()`方法，JVM会先列举该类以及其超类中访问属性为public且名为`change`的方法。
2. 接下来，JVM将查看调用方法时提供的参数类型，并且JVM会预先为每个类创建一个**方法表（method table）**，JVM会直接从这个方法表中寻找名为`change`的方法中存在一个与提供的参数类型匹配的方法，这个过程称为**重载解析**。
3. 如果是`private`、`static`、`final`方法或者构造器，那么JVM就能准确的知道调用哪个方法，我们将这种调用方式称为**静态绑定**。与此对应，调用的方法依赖于隐式参数的实际类型，并且在运行时实现**动态绑定**。
4. 当程序运行，并且采用动态绑定调用方法时，JVM就一定调用于此最适合的一个方法，否则从超类中继续寻找。

### 强制类型转换

将一个类型强制转换为另外一个类型的过程称为类型转换。数值类型直接`(int) double`这样转换；对象引用的转换也类似，实现将某个类的对象引用转换为另一个类的对象引用。

* **向上转型**：将一个子类的引用赋值给一个超类变量。
* **向下转型**：将一个超类的引用赋值给一个子类变量，且必须进行类型转换。

**注意**

* 只能在继承层次内进行类型转换。
* 在将超类转换成子类之前，应该使用`instanceof`进行检查。

### 内部类

一个类存在于另一个类中方法外，这个类就称为内部类；一个类存在于另一个类方法内，这个类称为局部内部类。

* 内部类可以直接访问外部类的成员，包括私有
* 外部类可以访问内部类的成员，必须创建对象
* 直接访问内部类的成员：`Outer.Inner in = new Outer().new Inner()`

#### 局部内部类

局部内部类可以直接访问外部类的成员，在局部位置可以创建内部类对象，通过对象调用内部类成员。

>  **局部内部类访问局部变量注意事项?**

​		局部内部类访问局部变量必须用`final`修饰。因为**局部内部类的声明周期比局部变量长**，局部变量随着方法的调用而存在，随着调用完毕而消失；但局部内部类不一定消失，他调用一个消失的变量就会报错。

```java
public class InnerClass {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.show();
    }
}

class Outer {
    public void show() {
        int num2 = 10;
        class Inner {
            private void show() {
                System.out.println(num2);
            }
        }
        Inner inner = new Inner();
        inner.show();
    }
}
```

此时调用不会报错，但并没有加`final`修饰。这个类编译后会生成`InnerClass.class`和`Outer.class`两个文件，我们来看下`Outer.class`:

```java
class Outer {
    Outer() {}
    public void show() {
        final int num2 = 10;
        class Inner {
            Inner() {}
            private void show() {
                System.out.println(num2);
            }
        }
        Inner inner = new Inner();
        inner.show();
    }
}
```

其中的`num2`被自动加上了`final`修饰（这是因为JDK1.8的原因），所以如果你再添上`num2 = 1000`就会报错。

> 解决办法

上面说过了应该将`num2`用`final`修饰。其原因就是`Inner`类的生命周期要比`num2`的声明周期长，当`show()`方法调用完毕后`num2`就已经消失了，但此时`Inner`类在堆内存中仍然存在，他调用一个不存在的变量就会报错。而用`final`修饰，这个变量成为常量，在初始化内部类的时候，`final num2`就在内部类中生成了一份拷贝，这个拷贝和这个内部类的声明周期相同，所以不会报错。