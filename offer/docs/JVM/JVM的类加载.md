## 1.JVM在什么情况下会加载一个类？

在**你的代码中用到这个类的时候**。

首先你的代码中包含“main()”方法的主类一定会在JVM进程启动之后被加载到内存，开始执行你的“main()”方法中的代码。

接着遇到你使用了别的类，比如“ReplicaManager”，此时就会从对应的“.class”字节码文件加载对应的类到内存里来。

**（1）验证阶段**

简单来说，这一步就是根据Java虚拟机规范，来校验你加载进来的“.class”文件中的内容，是否符合指定的规范。

**（2）准备阶段**

我们写好的那些类，其实都有一些类变量，这个准备工作，其实就是给这个“ReplicaManager”类分配一定的内存空间。然后给他里面的类变量（也就是static修饰的变量）分配内存空间，来一个默认的初始值。

**（3）解析阶段**

这个阶段干的事儿，实际上是把**符号引用替换为直接引用**的过程。

## 2.类从加载到使用过程

加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载

## 3.JVM的类加载阶段

JVM的类加载分为 5个阶段：加载、验证、准备、解析、初始化。在类初始化完成后就可以使用该类的信息，在一个类不再被需要时可以从JVM中卸载。

![](D:\workspace\java\images\JVM011.png)

### 1.加载

指JVM读取Class文件，并且根据Class文件描述创建java.lang.Class对象的过程。类加载过程主要包含将Class文件读取到运行时区域的方法区内，在堆中创建java.lang.Class对象，并封装类在方法区的数据结构的过程，在读取Class文件时既可以通过文件的形式读取，也可以通过jar包、war包读取，还可以通过代理自动生成Class或其他方式读取。

### 2.验证

主要用于确保Class文件符合当前虚拟机的要求，保障虚拟机自身的安全，只有通过验证的Class文件才能被JVM加载。

### 3.准备

主要工作是在方法区中为类变量分配内存空间并设置类中变量的初始值。初始值指不同数据类型的默认值，这里需要注意final类型的变量和非final类型的变量在准备阶段的数据初始化过程不同。

比如一个成员变量的定义如下：

```
public static int value=1000;
```

在以上代码中，静态变量value在准备阶段的初始值是0，将value设置为1000的动作是在对象初始化时完成的，因为JVM在编译阶段会将静态变量的初始化操作定义在构造器中。但是，如果将变量value声明为final类型：

```
public static final int value=1000;
```

则JVM在编译阶段后会为final类型的变量value生成其对应的ConstantValue属性，虚拟机在准备阶段会根据ConstantValue属性将value赋值为1000。

### 4.解析

JVM会将常量池中的符号引用替换为直接引用。

符号引用就是class文件中的：

1.CONSTANT_Class_info

2.CONSTANT_Field_info

3.CONSTANT_Method_info

等类型的常量。

#### 符号引用

符号引用与虚拟机实现的布局无关，引用的目标并不一定要已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须是一致的，因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中。

#### 直接引用

直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有了直接引用，那引用的目标必定已经在内存中存在。

### 5.初始化

初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载器以外，其它操作都由JVM主导。到了初始阶段，才开始真正执行类中定义的Java程序代码。

主要通过执行类构造器的<client>方法为类进行初始化。<client>方法是在编译阶段由编译器自动收集类中静态语句块和变量的赋值操作组成的。JVM规定，只有在父类的<client>方法都执行成功后，子类中的<client>方法才可以被执行。在一个类中既没有静态变量赋值操作也没有静态语句块时，编译器不会为该类生成<client>()方法。

在发生以下几种情况时，JVM不会执行类的初始化流程。

- 常量在编译时会将其常量值存入使用该常量的类的常量池中，该过程不需要调用常量所在的类，因此不会触发该常量类的初始化。 
- 在子类引用父类的静态字段时，不会触发子类的初始化，只会触发父类的初始化。
- 定义对象数组，不会触发该类的初始化。
- 在使用类名获取Class对象时不会触发类的初始化。
- 在使用Class.forName加载指定的类时，可以通过initialize参数设置是否需要对类进行初始化。
- 在使用ClassLoader默认的loadClass方法加载类时不会触发该类的初始化。

## 4.类加载器

JVM提供了 3种类加载器，分别是启动类加载器、扩展类加载器和应用程序类加载器。

![](D:\workspace\Java-Interview-Offer\images\JVM012.png)

（1）启动类加载器(Bootstrap ClassLoader)

主要是负责加载我们在机器上安装的Java目录下的核心类的。负责加载Java_HOME/lib目录中的类库，或通过Xbootclasspath参数指定路径中被虚拟机认可的类库。

（2）扩展类加载器(Extension ClassLoader)

负责加载Java_HOME/lib/ext目录中的类库，或通过java.ext.dirs系统变量加载指定路径中的类库。

（3）应用程序类加载器(Application ClassLoader)

负责加载用户路径（classpath）上的类库。

除了上述 3种类加载器，我们也可以通过继承java.lang.ClassLoader实现自定义的类加载器。

## 5.双亲委派机制

JVM通过双亲委派机制对类进行加载。双亲委派机制指一个类在收到类加载请求后不会尝试自己加载这个类，而是把该类加载请求向上委派给其父类去完成，其父类在接收到该类加载请求后又会将其委派给自己的父类，以此类推，这样所有的类加载请求都被向上委派到启动类加载器中。若父类加载器在接收到类加载请求后发现自己也无法加载该类（通常原因是该类的Class文件在父类的类加载路径中不存在），则父类会将该信息反馈给子类并向下委派子类加载器加载该类，直到该类被成功加载，若找不到该类，则JVM会抛出ClassNotFoud异常。

采用双亲委派的一个好处是比如加载位于rt.jar包中的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个Object对象。

**双亲委派类加载机制的类加载流程如下**。

（1）将自定义加载器挂载到应用程序类加载器。

（2）应用程序类加载器将类加载请求委托给扩展类加载器。

（3）扩展类加载器将类加载请求委托给启动类加载器。

（4）启动类加载器在加载路径下查找并加载Class文件，如果未找到目标Class文件，则交由扩展类加载器加载。

（5）扩展类加载器在加载路径下查找并加载Class文件，如果未找到目标Class文件，则交由应用程序类加载器加载。

（6）应用程序类加载器在加载路径下查找并加载Class文件，如果未找到目标Class文件，则交由自定义加载器加载。

（7）在自定义加载器下查找并加载用户指定目录下的Class文件，如果在自定义加载路径下未找到目标Class文件，则抛出ClassNotFoud异常。

![](D:\workspace\Java-Interview-Offer\images\JVM013.png)

双亲委派机制的核心是保障类的唯一性和安全性。例如在加载rt.jar包中的java.lang.Object类时，无论是哪个类加载器加载这个类，最终都将类加载请求委托给启动类加载器加载，这样就保证了类加载的唯一性。如果在JVM中存在包名和类名相同的两个类，则该类将无法被加载，JVM也无法完成类加载流程。

## 6.自定义类加载器如何实现？

**答：**自己写一个类，继承ClassLoader类，重写类加载的方法，然后在代码里面可以用自己的类加载器去针对某个路径下的类加载到内存里来。