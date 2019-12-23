# Java类加载机制

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/类加载全流程.png)



## 类编译

类编译，即 .java 文件通过javac命令编译成 .class 文件，才能在虚拟机上正常运行代码。可以通过 javap查看class文件包含信息。

重点：编译后的**字节码文件**主要包括**常量池和方法表集合**这两部分

- 常量池 ：主要记录的是字节码文件中出现的**字面量**以及**符号引用**

  - **字面常量**包括字符串常量（例如 String str=“abc”，其中”abc”就是常量），声明为 final 的属性以及一些基本类型（例如，范围在 -127-128 之间的整型）的属性。

  - **符号引用**包括类和接口的全限定名、类引用、方法引用以及成员变量引用（例如 String str=“abc”，其中 str 就是成员变量引用）等。

- 方法表集合

  - 包含一些方法的字节码、方法访问权限（public、protect、prviate 等）、方法名索引（与常量池中的方法引用对应）、描述符索引、JVM 执行指令以及属性集合等。



## 类加载流程

类从被加载到虚拟机内存到卸出内存为止，类加载的整个生命周期分为加载-连接-初始化三个阶段。

![](https://icecrea-blog-1300414836.cos.ap-beijing.myqcloud.com/blog/类加载流程.png)

### 1. 加载阶段：

**加载阶段**是指将字节码数据（类的.class文件的二进制数据）从不同的数据源读取到 JVM内存中，放在方法区内，在堆中创建java.lang.Class对象，封装方法区内的数据结构。

字节码数据源可以来自于磁盘文件 *.class，也可以是 jar 包里的 *.class，也可以来自远程服务器提供的字节流，字节码的本质就是一个字节数组 []byte，有特定的复杂的内部格式。如果输入数据不是 ClassFile 的结构，则会抛出 ClassFormatError。

加载过程完成以下三步：

1. 通过一个**类的全限定名**来获取定义此类的**二进制字节流**。
2. 将这个字节流所代表的**静态存储结构**转化为**方法区的运行时存储结构**。 
3. 在内存中生成一个代表这个**类的 Class 对象**，作为**方法区这个类的各种数据的访问入口**。

加载阶段即可以使用系统提供的类加载器在完成，也可以由用户**自定义的类加载器**来完成。加载阶段与连接阶段的部分内容(如一部分字节码文件格式验证动作)是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始。

### 2. 连接阶段
#### 2.1 验证阶段
确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。
整体分为4个阶段的校验工作：**文件格式、元数据、字节码、符号引用**

#### 2.2 准备阶段
准备阶段是正式为**类变量(static 成员变量)分配内存**并设置类变量初始值（零值）的阶段，这些变量所使用的内存都将在方法区中进行分配。
这时候进行内存分配的仅包括类变量，而不包括实例变量，**实例变量将会在对象实例化时随着对象一起分配在堆中**。
其次，这里所说的初始值“通常情况”下是数据类型的零值，假设一个类变量的定义为：

```java
public static int value = 123;
```
那么，变量value在准备阶段过后的值为0而不是123。因为这时候尚未开始执行任何java方法，而**把value赋值为123的putstatic指令是程序被编译后，存放于类构造器方法之中**，所以把value赋值为123的动作将在初始化阶段才会执行。
“特殊情况”：当类字段的字段属性是常量时，会在准备阶段初始化为指定的值，所以标注为final之后，value的值在准备阶段初始化为123而非0。

#### 2.3 解析阶段
　解析阶段是虚拟机将**常量池内的符号引用**替换为**直接引用**的过程。编译时，Java 类并不知道所引用的类的实际地址，因此只能使用符号引用来代替。类结构文件的常量池中存储了符号引用，包括类和接口的全限定名、类引用、方法引用以及成员变量引用等。如果要使用这些类和方法，就需要把它们转化为 JVM 可以直接获取的内存地址或指针，即直接引用。

### 3. 初始化阶段
#### 类何时初始化？

虚拟机规范中并没有强制约束何时进行**加载**，但是规范严格规定了有且只有下列五种情况必须对类进行**初始化**（加载、验证、准备都会随着发生）：

> 1.  遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则必须先触发其初始化。最常见的生成这 4 条指令的场景是：
		
		 a. 使用 new 关键字实例化对象的时候
		
	 b. 读取或设置一个类的静态字段的时候（被 final 修饰、已在编译器把结果放入常量池的静态字段除外)
	
	 c. 调用一个类的静态方法的时候
	
	2. 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行初始化，则需要先触发其初始化。
	
	3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
	
	4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类；
	
	5.  当使用 JDK.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为REF_getStatic, REF_putStatic, REF_invokeStatic  的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化；

#### init 和 clinit 方法区别？

- init是对象实例构造器方法，即在程序执行 new 一个对象调用该对象类的构造方法时才会执行init方法，对非静态变量解析初始化。
- clinit是类构造器方法，在jvm类加载的加载-连接-初始化的初始化阶段调用。class类构造器对静态变量，静态代码块进行初始化。

初始化阶段才真正开始执行类中的定义的 Java 程序代码。初始化阶段即虚拟机执行**类构造器方法**的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。从另一个角度表达，初始化阶段即为执行类构造器方法`<clinit>`的过程。

`<clinit>`方法是由编译器自动收集类中的所有**类变量的赋值动作**和**静态语句块**（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的。

特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。

```java
  static {
        i = 0;
        //静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。
        //编译报错： Illegal forward reference
        System.out.println(i);
    }

    static int i =1;
```
#### 被动引用：不触发初始化

注意：所有引用类的方式都不会触发初始化称为被动引用，下面是3个被动引用例子（对比上文"类何时初始化"进行理解）：

1. 通过子类引用父类静态字段，不会导致子类初始化；
2. 通过数组定义引用类，不会触发此类的初始化
3. 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用定义常量的类，因此不会触发定义常量的类的初始化

```java
public class SuperClass {
    public static final String HELLO = "hello";

    static {
        System.out.println("super static class");
    }

    public static int value = 123;

    public SuperClass(){
        System.out.println("super class 初始化");
    }
}


class SubClass extends SuperClass{
    static {
        System.out.println("sub static class");
    }

    public SubClass(){
        System.out.println("sub class 初始化");
    }
}

 class Main {
     /**
      * 输出结果：
      * super static class
      * 123
      * hello
      */
    public static void main(String[] args) {
        // 被动引用1 ：子类加载父类静态字段，初始化父类，但不会导致子类初始化；
        System.out.println(SubClass.value);

        // 被动引用2 ： 通过数组定义引用类，不会触发此类的初始化
        // 触发的是一个虚拟机自动生成的，直接继承于java.lang.Object的子类，创建动作由字节码指令newarray触发。并没有触发真正的SuperClass的初始化
        SuperClass[] sca = new SuperClass[10];

        // 被动引用3 ： 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用定义常量的类，因此不会触发定义常量的类的初始化
        System.out.println(SuperClass.HELLO);
    }
}
```

#### 类加载顺序解析

分析如下代码，控制台输出顺序是什么？类变量count1，count2最终输出为多少？


```java
public class Singleton {

    private static Singleton singleton = new Singleton();

    public static int count1;
    public static int count2 = 5;

    private Singleton() {
        count1++;
        count2++;
        System.out.println("构造函数 count1 = " + count1 + " | count2 =" + count2);
    }

    // 对象一建立就运行构造代码块了，而且优先于构造函数执行。有对象建立，才会运行构造代码块
    {
        System.out.println("构造代码块 count1 = " + count1 + " | count2 =" + count2);
    }

    static {
        System.out.println("静态代码块 count1 = " + count1 + " | count2 =" + count2);
    }

    public static Singleton getInstance() {
        return singleton;
    }

    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        System.out.println(Singleton.count1);
        System.out.println(Singleton.count2);
    }
}
```

分析一下流程：

主方法中调用了类的静态方法 getInstance()，会触发类的初始化。前文介绍过类初始化即调用`clinit`方法，该方法是编译器自动**按序**收集合并类中的所有**类变量的赋值动作**和**静态语句块**。代码中有三个类变量和一个静态语句块，按顺序合并生成`clinit`方法，最先执行的为类变量`singleton`的赋值动作。调用构造函数，同时构造代码块在构造函数之前。此时count1 , count2 还未赋值，输出结果为0，++后变成1。再继续执行`clinit`方法给count1，count2赋值，便覆盖了之前的++操作。最终结果为1和5。

最终结果如下。可以试着改变代码顺序，自己分析不同的输出结果。

```java
构造代码块 count1 = 0 | count2 =0
构造函数 count1 = 1 | count2 =1
静态代码块 count1 = 1 | count2 =5
1
5
```

