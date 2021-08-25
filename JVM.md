# JVM

## 一、JVM整体概述

#### 1、 字节码

字节码： 各种语言进行编译，compile后得到的文件，为java字节码。一般为 **.class** 文件

#### 2、虚拟机 

一款用来执行虚拟计算机的指令的软件。

* 系统虚拟机： VMware，Visual Box。
* 程序虚拟机：JVM

#### 3、JVM的特点

* 一次编译，到处运行
* 自动分配内存，自动内存管理
* 垃圾回收机制

## 二、JVM 整体结构

#### 1、整体结构



<img src="D:\CZY\Young\文档\研究生\learn\JVM\ppt\微信截图_20210308123435.png" style="zoom: 67%;" />

**注:** 

* **classfile：**已经经过第一次编译形成的java字节码文件。

* **方法区和堆**：是多线程共享的。

* **灰色部分**是每个线程都独自拥有。

* User -----> 字节码------> JVM ------> OS -------> 硬件。 **JVM 不与硬件直接交互，得通过OS**

  

## 三、java程序执行步骤



<img src=".\picture\image-20210308145642916.png" alt="image-20210308145642916" style="zoom: 67%;" />

* 第一次编译生成 java 字节码 .class 文件。

* 第二次编译 由 执行引擎 来编译， 将 java 字节码文件 编译成 机器码

  ------

  

## 四、 JVM的架构模型

#### 1、 基于栈

* 设计与实现较为简单，适用于资源受限系统（嵌入式系统）
* 指令小而多，指令中大部分为零地址指令
* 不需要硬件支持，可移植性好
* 避开了寄存器分配资源的难题，使用零地址指令分配

#### 2、基于寄存器

* x86 二进制指令集

* 可移植性差

* 性能优秀

* 一，二，三，地址指令为主

* 花费更少的指令完成操作

  

## 五、JVM的生命周期

1. **JVM 启动**：JVM 启动是由引导类加载器（bootstrap class initial）创建一个初始类来完成，这个类由JVM的具体实现指定。
2. **JVM 执行**：JVM执行java程序，程序运行时，JVM执行，程序结束时，JVM停止
3. **JVM 退出**: 
   * 程序正常运行结束
   * 出现异常，或错误而异常终止
   * OS出现错误，JVM进程终止
   * Runtime类，System类中的exit方法，或者runtime的halt方法 

## 六、内存结构---类加载

#### 1、内存结构概述

<img src=".\picture\image-20210308203005752.png" alt="image-20210308203005752" style="zoom:67%;" />



<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：自己写JVM，主要考虑那些结构？**</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:  类加载器， 执行引擎</span>



#### 2、类加载过程和加载器

![image-20210308204004998](.\picture\image-20210308204004998.png)

* 类加载器子系统负责从文件系统或者网络中加载 class 文件， class文件再文件开头有标识 **CA FE BA BE** 
* ClassLoader 只负责加载类，是否可行，由执行引擎判断
* 加载的类信息存放在 ：**方法区**， 方法区还会存储常量池信息。
* 自定义类使用 **系统类加载器** 加载

<img src=".\picture\image-20210308212002443.png" alt="image-20210308212002443" style="zoom:67%;" />

<img src=".\picture\image-20210308212052267.png" alt="image-20210308212052267"  />

<img src=".\picture\image-20210308212034327.png" alt="image-20210308212034327"  />

![image-20210308212516215](.\picture\image-20210308212516215.png)

**A .加载 .class 文件的方式**

* 从本地加载
* 通过网络加载
* 从zip中读取，jar包
* 运行时计算生成，动态代理
* 其他文件生成，JSP
* 从专有数据库提取
* 从加密文件中获取，预防 Class 文件被反编译

**B. Linking(链接)**

1. 验证：验证 Class 文件中的字节流符合 JVM 的规范，保证类的加载正确性
          	四种验证：**文件格式验证，元数据验证，字节码验证，符号引用验证**
2. 准备：为类变量分配内存设置为默认值，即零。
              这里不包含 **final** 修饰的 常量值， final 修饰的 在编译时 就初始化了
              这里也不能为实例变量实例化，实例变量随着对象分配 java 堆中
3. 解析：将常量池内的符号引用转化为直接引用

**C. 初始化**

* 就是执行类构造器方法 **clinit**
* 此方法不需要定义，javac 编译器自动收集类中的所有变量赋值动作，以及静态代码块
* **clinit 按照源文件中的代码出现顺序执行**

```java
public class CLASSInit{
    public static int num = 1;
    
    static{
        num = 2;
        number = 10;
        System.out.print(num);
        System.out.ptint(number);
    }
    
    private static int number = 20;  // Linking 的 prepare number = 0 --->initial 20 ---> 10
}
```

* clinit() 与 类的构造器（构造函数） 不一样， 在JVM视角下 构造器时 init（）

* 若该类有父类，则会先运行父类的 clinit()

* JVM 必须保证 一个类的 clinit（）同步加锁

* <span style='color:yellow;background:背景颜色;font-size:文字大小;font-family:字体;'>若类中没有赋值操作和静态代码块，则不会调用 clinit() </span>

  

#### 3、类加载器的分类

![image-20210309092450094](.\picture\image-20210309092450094.png)

* JVM 支持两种类加载，引导类加载器（Bootstrap ClassLoader），自定义类加载器。 
* 凡是继承于 ClassLoader 的类都为自定义类加载器， Extension ， System Classloader 也是。

**引导类加载器**

使用C++/C语言实现，嵌套在JVM内部

用来加载核心类的

**扩展类加载器（Extension ClassLoader）**

继承Classloader类

父类加载器是启动类加载器

从指定的类库加载类，jre/lib/ext中加载

**应用程序类加载器（AppClassLoader）** 

派生于ClassLoader

父类加载器为扩展类加载器

是程序中默认的类加载器

**用户自定义类加载器**

<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：为什么要用户自定义类加载器**</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:  1.隔离加载类  2. 修改类的加载方式 3.扩展加载源  4.防止源码泄露</span>



#### 4、 双亲委派机制

<img src=".\picture\image-20210309120754583.png" alt="image-20210309120754583" style="zoom:67%;" />

**原理：**

1. 一个类加载器收到类加载的请求，不会自己加载，先委托父类类加载器
2. 若父类类加载器还有父类，那么再次委托给上级
3. 若父类可以加载成功，则返回，父类无法加载子类才会自己加载

[双亲委派的例子，String类的加载]: https://www.bilibili.com/video/BV1PJ411n7xZ?p=35&amp;spm_id_from=pageDriver

![image-20210309122337696](.\picture\image-20210309122337696.png)

![image-20210309122438384](.\picture\image-20210309122438384.png)

因为 是双亲委派机制，所以会先在父类类加载器中加载，会优先加载 系统中的核心类 String 而不是自定义的类

**优点：**

* 避免了类的重复加载
* 保护了程序安全，防止核心的API被随意的篡改

![image-20210309125533422](.\picture\image-20210309125533422.png)

解释，SPI 核心类在 引导类加载器中加载

而 jdbc.jar 第三方的 jar 包，则下派刀系统类加载器中加载

**沙箱安全机制**

自定义的 String ，但是在加载自定义的 String 类会率先使用引导类加载，引导类会优先加载 JDK 自带的文件。这样可以保护核心代码，叫沙箱安全机制。

#### 5、判断类相同

* 类名以及包名相同

* 加载这个类的 ClassLoader 相同

  比如自定义的 String 和 JDK 中的 String 就不相同

**类的主动使用与被动使用**

主动使用：

* 创建类的实例

* 访问某个类或者接口的静态变量，或者给该变量赋值

* 反射

* 调用类的静态方法

  

## 七、运行时数据区

<img src=".\picture\image-20210309173040427.png" alt="image-20210309173040427" style="zoom:67%;" />

<img src=".\picture\image-20210309173305854.png" alt="image-20210309173305854" style="zoom:67%;" />

![image-20210311163018429](.\picture\image-20210311163018429.png)

JVM 定义了若干运行时的数据区，其中一些会随着虚拟机的创建而创建，生命周期和JVM一样，另外一些是与线程的生命周期一样

* 灰色区域为线程独有的，红色的是多线程

* 线程间共享：堆，堆外内存（方法区，源空间）

* 每个进程对应着一个 JVM，每个 JVM 只有一个 Runtime 的实例，即运行环境

  注：**JROCKIT，IBM J9虚拟机没有方法区。**

  

#### 1、PC寄存器 （PC Register）

![](.\picture\image-20210311163124315.png)

**程序计数器：** 存储指令下一条指令的地址，即即将要执行的指令代码。由执行引擎读取下一条指令。

每个线程都有 PC Register ，线程私有，占用很小的内存空间，运行速度最快。**不会有 OOM 错误**。**没有GC**

```java
public class PCRegister {
    public static void main(String[] args) {
        int i = 10;
        int j = 1;
        int k = i + j;
        String s = "abc";

        System.out.println(i);
        System.out.println(j);
        //进入 cd production/Leetcode/JVM
        //javap - verbose .class
        //进行反编译 ，就可以看到 pc寄存器，机器指令
    }
}
```

![image-20210309200025640](.\picture\image-20210309200025640.png)

<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程执行地址？**</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:  CPU在不停的切换线程，切换回来之后根据PC寄存器就知道接着从哪里开始运行</span>

<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：PC寄存器为什么被设定为线程私有？**</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答: 因为PC寄存器如果公有，则会产生错误</span>

<img src=".\picture\image-20210309212228870.png" alt="image-20210309212228870" style="zoom: 50%;" />

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>若线程共有PC寄存器，则在线程1 执行被线程2 抢占之后 PC 寄存的值会变为 5， 若这是线程3抢占了 线程2 则又被修改为 7， 再切换为线程 1 就不是 5 了。只有每个线程独自拥有 PC 寄存器 才不会冲突</span>



#### 2、虚拟机栈

![image-20210311162947341](.\picture\image-20210311162947341.png)

**内存中的栈与堆：** 栈是运行时的单位，堆是存储单位。

栈：程序该如何执行，如何处理数据。

堆：数据该放在哪一个部分，怎么放。对象大部分放在堆空间中。运行数据区默认最大。

**JVM中JAVA 虚拟机栈： ** 

* 每个线程都会拥有 java 虚拟机栈，内部保存的一个个栈帧，对应一次次的 java 方法调用。
* 生命周期：与线程一致。
* 作用：主管 java 程序的运行，保存方法的局部变量，部分结果，并参与方法的调用和返回。
* JVM 栈中只有出栈，入栈操作。
* 虚拟机栈存于 **OOM中，不存在 GC**

<img src=".\picture\image-20210309215456856.png" alt="image-20210309215456856" style="zoom: 67%;" />

<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：**虚拟机栈中可能出现的异常：怎么设置栈的大小</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:  java 栈的大小可以是固定的，也可以是动态变化的。 若固定了大小，超过了固定大小，回StackOverflowError 。若 java 栈是动态的，java栈在扩展时申请内存时失败，或者没有足够的内存来扩展，抛出OOM（OutOfMemoryError）</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:  -Xss 设置当前的线程使用的java栈的大小， 例如 -Xss 256k</span>

**虚拟机栈的内容**：每个线程都有自己的虚拟机栈，栈中存储着 栈帧，每个栈帧与对应的方法对应。

在某个时间点上，只会有一个活动的栈帧，即当前栈帧，也就是虚拟机栈的栈顶元素，对应的当前方法，定义这个方法的称为当前类。

执行引擎会根据当前栈帧运行对应的字节码。

<span style='color:pink;background:背景颜色;font-size:18px;font-family:字体;'>注: Ali 新技术，进程之间共享  GCIH 有空可以了解</span>



##### a.**栈运行原理**

* 不同线程各自的栈帧不能相互作用，即一个栈帧不可以引用另外一个线程的栈帧

* 若当前方法调用其他的方法，方法返回时，当前栈帧会把结果传回前一个帧，然后虚拟机抛弃前一个帧，使前一个帧成为当前栈帧。

* **java函数两种返回方式**：1. return 2.抛出异常（没处理异常） 无论哪一种方式都会弹出栈帧。

  ​			try catch 捕获异常的情况 为正常结束return 结束。而不是异常结束。

##### b. 栈帧的内部结构

* **局部变量表**（Local Varible）

  又称为局部变量数组，本地变量表。

  1. 定义为**数字**数组（因为基本数据类型都可以使用 数值 来表示），存储 **方法参数** 和 方法内的 **局部变量** （基本变量类型，对象引用，returnAddress）

  2. 因为线程的 虚拟机栈 是私有的，所以栈帧中的 LV 不存在安全问题。

  3. LV 的大小是在编译期确定，运行的时候不会被更改。

  4. 方法嵌套次数由 虚拟机栈大小决定，大小确定后和栈帧大小相关，栈帧大小主要和 **LV** 相关。

  5. 栈帧中 **LV** 中的值 只在当前方法有效，方法结束后栈帧被销毁，LV 也被销毁。

     <span style='color:pink;background:背景颜色;font-size:16px;font-family:字体;'>关于 slot 理解</span>

     * slot 称为 槽，LV 会为每一个 slot 分配一个访问索引，通过索引可以成功访问 slot 中的值

     * 当一个实例方法被调用时，他的方法参数和方法内部的局部变量按顺序复制到 LV 中的每个 slot 上。

     * 字节长度小于32的 占用一个 slot ，等于 64 的占用两个 slot。

     * 若当前帧是构造方法，或者实例方法创建的，**非静态的**，那么该对象的引用 this 将存放在索引为 0 的 slot 处。（在静态方法中不能使用 this 是因为 LV 中 不存在 this 变量！）

     * slot 可以重复利用，一个局部变量过了他的作用域，那么在他之后的新的局部变量会复用他的槽位。

       ```java
       public void test(){
       	int a = 10;
           {
               int b = 0;
               b = a + 1;
           }
           //该代码中 有三个 slot。 this，a，c
           // 因为局部变量b 已经过了他的作用域，新的局部变量c 会服用他的槽位
           int c = 1;
       }
       ```

       <img src=".\picture\image-20210310120719990.png" alt="image-20210310120719990" style="zoom: 67%;" />
     
  6. LV 是重要的垃圾回收根节点，只要被局部变量表中直接或者间接引用的对象不会被回收。

* **操作数栈**（表达式栈 /  Operand Stack）

  1. 使用数组实现。当方法开始执行，栈被创建，并且是空的，但是大小是固定的（编译时就确定了，且无法改变）。**虽然用数组实现，但是不能使用索引访问，只能出栈入栈访问。**
  2. 栈中的数据 32 位 占一个栈的深度，64位站两个。
  3. 在方法执行时，根据字节码指令，往栈中写入或提取数据。
     <img src=".\picture\image-20210310135435857.png" alt="image-20210310135435857" style="zoom:50%;" />
  4. java **JVM 执行引擎 基于操作数栈执行对数据的操作**。
  5. **作用：**保存计算的中间结果，同时作为计算过程中变量的临时存储空间。
  6. 若被调用的方法有返回值，其返回值会被压入当前栈帧的操作数栈中，并更新PC寄存器的下一条指令。

* **动态链接 （Dynamic Linking）**或 运行时**常量池**的方法引用

  栈帧内部包含一个指向运行时常量池中该帧所属方法的引用，目的是为了实现当前方法的动态链接。

  java 源文件被编译为 .class文件时， 所有变量和方法引用作为符号引用保存在 class 文件的常量池中。

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>动态链接将符号引用转化为 调用方法的直接引用</span>

  即直接链接到方法区中的，运行常量池中的方法引用，就可以通过引用调用对应的方法，节省空间。

  ![image-20210310224443738](.\picture\image-20210310224443738.png)

  <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：为什么需要常量池？**</span>

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>常量池作用：提供符号和常量，便于指令识别</span>

* **方法返回地址 （Return Address）**方法正常推出或者异常退出的定义
  保存调用该方法的PC寄存器的值。

  当程序完成正常返回时，返回 ireturn， lreturn , dreturn ,freturn, areturn (返回引用), return(void)。或者异常返回

  **异常返回与正常返回的区别就是：**正常返回就是出栈操作，此时需要恢复上层方法的LV，操作数栈，将返回值压入操作数栈中，设置PC寄存器中的值。 而异常完成出口不会给上层返回任何值。

* 一些附加信息

<img src=".\picture\image-20210310100538185.png" alt="image-20210310100538185" style="zoom:67%;" />

#### 3、本地方法栈

![image-20210311163111143](.\picture\image-20210311163111143.png)

java虚拟机栈管理 java 方法调用。 本地方法栈，管理本地方法的调用。

* 线程私有
* 可以固定长度，可以动态扩展。
* 使用 C 实现的
* 当线程调用本地方法时，和虚拟机有同样的权限：可以直接使用本地的内存，本地的寄存器，以及虚拟机的运行时数据区。

#### 4、 栈顶缓存技术

因为 基于栈架构的虚拟机使用零地址指令更加的紧凑，换句话说就是完成一项操作会需要更多的出栈入栈操作。也就会影响os的执行速度。解决该问题 hotspot 提出 ToS (Top of Stack Caching)栈顶缓存。将栈顶元素全部缓存于物理CPU中，降低对内存的读写次数，提升执行引擎的执行效率。

#### 5、方法调用

 JVM中符号引用转化为调用方法的直接引用 与绑定机制有关。**转化过程在编译期间确定，还是在运行期间确定。多态性的解释。**

* **静态链接：**一个字节码文件加载到 JVM 时，若被调用的目标方法编译器可知，且运行时保持不变，
* **动态链接：**一个字节码文件被调用的方法在编译器无法确定下来，只能在运行时将符号引用转为方法引用，则为动态链接。
* 早期绑定：绑定是一个字段，方法，类的符号引用被替换为直接引用。 若被调用的目标方法编译器可知，且运行时保持不变。
* 晚期绑定：一个字节码文件被调用的方法在编译器无法确定下来，只能在运行时将符号引用转为方法引用，则为动态链接。

**非虚方法**：在编译期间就确定不变，运行时也不可变，就是非虚方法。

​	静态方法，私有方法，final方法，实例构造器，父类方法都是非虚方法。剩下的是虚方法。**** 

**虚拟机中的调用指令：**

* invokestatic 调用 static 方法
* invokespecial 调用父类中的<init> ，私有父类方法
* invokevirtual 调用虚方法
* invokeinterface 调用接口
* invokedynamic

#### 7、栈相关面试题

* <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**举例说明栈溢出的情况？**</span>

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>StackOverFlowError , 可以通过 -Xss 设置栈的固定大小，但是当调用过多的方法时，栈的空间就不足了。 也可以采用栈的大小动变化，这样容易产生 OOM 内存不足的情况。</span>

* <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**调整栈大小，能保证不会出现溢出情况吗？**</span>

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>不能</span>

* <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**分配的栈内存越大越好吗？**</span>
  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>不是的，整个内存空间有限，给栈空间多了，其他会收到影响没，比如线程会变少。</span>

* <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**垃圾回收是否设计虚拟机栈**</span>

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>不会的，因为 在 PC寄存器中 不存在 Error 与 GC ，虚拟机栈中 存在 Error 不存在 GC（因为这两个都只是入栈，出栈操作，不需要 GC ）</span>

* <span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**方法中定义的局部变量是否线程安全？**</span>

  <span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>具体问题具体分析。只有在方法内部产生，方法内部销毁的操作才是线程安全的</span>

```java
//method1 线程安全 因为在一个方法内部完成操作，并且没有返回
    public static void method1(){
        StringBuilder str = new StringBuilder();
        str.append("1");
        str.append("2");
    }

    //method2 不安全， 因为从外部读取的数据， 没有安全机制就是不安全的
    public static void method2(StringBuilder stringBuilder){
        stringBuilder.append("a");
        stringBuilder.append("b");
    }

    //存在线程不安全的问题，因为它存在返回值，所以返回的值可能被多个线程共享，因此就是线程不安全的
    public static StringBuilder method3(){
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("a");
        return stringBuilder;
    }

    //method4是线程安全的，对s1 的操作是安全的，new 了一个String，String被多个线程
    public static String method4(){
        StringBuilder s1 = new StringBuilder();
        s1.append("a");
        return s1.toString();
    }
```

## 八、本地方法接口

**Native Method:** 本地方法，定义一个本地方法时，并不提供方法体，因为方法内容由非java语言在外部实现。本地方法作用时融合不同的语言为java使用。

**native 与 abstract 不可以共用：**native 表示有方法体，只不过使用其他语言实现的，abstract 没有方法体。

**为什么使用native：**

* 与 Java 环境外的交互
* 与 OS 的交互: 需要和操作系统的底层交互
* Sun's Java，Sun解释器是 C 实现的

## 九、堆

#### 1、核心概述

![image-20210311163111143](.\picture\image-20210311163111143.png)

* 一个 JVM 实例对应着一个堆内存，是 Java 内存管理的核心区域

* Java 堆在 JVM 启动时创建，大小就确定了，是JVM最大的一块内存。

* 虚拟机规范中规定堆可以物理上不连续，但是逻辑上必须连续。

* 所有线程共享堆，**但是可以划分线程私有的缓冲区（Thread Local Allocation Buffer , TLAB）**

* 几乎所有的对象实例都在这里匹配空间 ------- 存在栈上分配的问题

* 数组和对象可能永远不会存在栈上，因为栈帧中保存引用，该引用指向堆中的对象或者数组。

* 方法结束后，堆中的对象不会马上移除，仅仅在垃圾收集的时候才会被移除。

  <img src=".\picture\image-20210311204622915.png" alt="image-20210311204622915" style="zoom: 67%;" />

  

##### A、<span style='color:red;'>内存细分</span>

​		现代垃圾收集器大部分基于分代收集理论，细分为

			* JDK 7以前内存逻辑分为三部分：新生区，养老区，**永久区**
			* JDK 8 之后的划分为 ： 新生区， 养老区，**元空间**

#### 2、设置堆的大小，与OOM

设置大小： 

1. ​       -Xms 表示堆区的起始大小

   2. -Xmx 表示堆空间最大内存 ， 一旦超出了堆空间的最大内存就会抛出 OOM 异常

   3. 通常设置 -Xms 与 -Xmx 相等， 目的：为了使 java 回收机制清理完堆之后不需要再重新分割计算堆的大小，提高性能，

   4. 默认初始：物理电脑内存大小 / 64

      默认最大：物理电脑内存 / 4

#### 3、年轻代和老年代

存储在JVM中的对象可以分为两类：

* 一类生命周期较短，类的对象创建销亡的很迅速
* 另一类生命周期较长。
  ![image-20210312230627151](.\picture\image-20210312230627151.png)

**设置新生代，老年代的大小：** <span style='color:yellow'>默认 -XX:NewRation=2</span>, 表示老年代/新生代 = 2（可修改 ）

![image-20210312232120855](.\picture\image-20210312232120855.png)

就是新生代的 eden区 和1区和2区的比例 8：1：1

<span style='color:yellow'>可能存在自适应的参数，使得比例不是8：1:1, -XX : UseAdaptivesSizePolicy 使用自适应， -XX : -UseAdaptivesSizePolicy不使用 自适应 的参数</span>

* 几乎所有的对象都是在eden区new的（若eden 装不下，可能直接进入 old gen）
* 80%对象都是朝生暮死
* <span style='color:yellow'>设置 eden 大小， -Xmn</span>>

#### 4、 图解对象分配过程

<img src=".\picture\image-20210313080159620.png" alt="image-20210313080159620" style="zoom: 80%;" />

**步骤1**：在 Eden 区满的时候，触发 MinorGC / YGC 回收 Eden 区和 Survival0 区。将不被回收的放入 S0，age =1。

**注意：**幸存者区不会触发 MinorGC ，只有 Eden 区满了才会触发 YGC。

​	<img src=".\picture\image-20210313074540091.png" alt="image-20210313074540091" style="zoom:67%;" />

**步骤2：** 在 GC 清空 Eden 区之后，Eden 区又满了，又调用 YGC，回收 Eden 与 S0 。

​			将 Eden 区中存活的对象放入 空的 Survival区（to区 / 谁空谁是to），同时将 S0 区中存活的对象也移动至to 区，age++。这样 S1 就存在对象，而S0成为了新的 to 区 / 空区。

<img src=".\picture\image-20210313075050519.png" alt="image-20210313075050519" style="zoom:67%;" />

**步骤三：**当 Survival 区中某些对象的 age 达到 15 超过了 阈值，则把他转移至 老年代。

![image-20210313075731233](.\picture\image-20210313075731233.png)

<span style='color:yellow'>总结： 幸存者 S0， S1，复制之后有交换，谁空谁是to。</span>

<span style='color:yellow'>关于垃圾回收，频繁的在新生代收集，很少在养老区收集，几乎不再永久区/元空间收集</span>。

![image-20210313081642001](.\picture\image-20210313081642001.png)



#### 5、Minor GC，Major GC, Full GC

GC 按照回收区域分为：部分收集（Partical GC），整堆收集（Full GC）

**部分收集：**

* 新生代收集：（Minor GC / YGC）只是对新生代的垃圾收集

* 老年代收集：（Major GC / Old GC）只对老年代的收集。**注意：目前只有 CMS GC会单独对老年区GC**

  **而且很多时候使 Major GC 与 Full GC混合使用的。**需要具体分辨使老年代回收还是整堆回收。

* 混合收集：收集整个新生代和部分老年代的 GC。**目前只有 G1 GC 会有这种行为。**

  

**整堆收集**：收集整个JAVA 堆和方法区的垃圾收集。

**年轻代（Minor GC）的触发机制：**

* 当年轻代空间不足得时候，指的是 Eden 区满了，Survival 不会触发 GC。
* java 对象一般朝生暮死。MInor GC调用的很频繁。
* Minor GC 会触发STW， 暂用其他的用户线程，等回收结束后再才恢复运行。

**老年代（Major GC /Full GC）**

* 出现 Major GC 时，经常会伴随着至少一次的 Minor GC。
* Major GC 执行速度比 Minor GC 慢10倍以上。
* 执行MajorGC 后空间还是不足就OOM

**Full GC执行的情况：**

* 调用 System.gc()
* 老年代空间不足
* 方法区空间不足
* 通过 Minor GC 进入老年代的平均大小大于老年代的可用内存
* 要尽量避免 Full GC

#### 6、堆空间分代思想

<span style='color:red;background:背景颜色;font-size:18px;font-family:字体;'>**问题：为什么要分代？**</span>

<span style='color:yellow;background:背景颜色;font-size:18px;font-family:字体;'>答:因为 java 对象的生命周期大多数都是临时对象。不分代可以正常运行，但是效率不高</span>

#### 7、内存分配策略

针对不同年龄阶段的对象分配原则

* 优先分配到 Eden 区
* 大对象直接分配到老年区
* 长期存活的分配到老年区
* 动态对象年龄判断：若相同年龄的所有对象的大小大于Survival空间的一半，年龄大于或等于该对象的年龄的对象可直接进入老年代。
* 空间分配担保

#### 8、为对象分配内存：TLAB

**为什么要有 TLAB （线程私有缓冲区）**

* 堆区时线程共享的，任何线程都可以访问堆区中的数据。
* JVM中创建对象十分频繁，再并发环境下从堆区划分空间是不安全的。
* 为了避免不安全，可以采用加锁，但是影响效率

**什么是TLAB？**

* 对 Eden 区进行进一步的分配，JVM 为每个线程分配了私有的缓冲区。
* 每个线程私有一份解决了线程安全问题，**快速分配策略**。

**TLAB的注意点**：

* -XX:UserTLAB 设置是否开启
* TLAB作为JVM内存分配的首选。
* TLAB占用很小，只占Eden的 1%
* 一旦在空间中TLAB分配失败，JVM 尝试加锁机制，直接在Eden
* ![image-20210313105359738](.\picture\image-20210313105359738.png)

#### 9、堆空间的参数设置

**-Xms ：** 初始化堆的大小

**-Xmx：**堆的最大大小

**-Xmn：**初始化新生代的大小

**-XX:NewRatio:** 设置老年代/新生代的比值

**-XX:+PrintFlagsInitial** 查看所有参数的默认初始值

**-XX:+PrintFlagsFinal** 查看所有参数的最终值

**-XX:NewRatio**: 新生代与老年代的比例

**-XX:SurvivorRatio:** Eden中 S0 与 S1的比例

**-XX:MaxTenuringThreshold:** 设置新生代的最大年龄

**-XX:HandlePromotionFailer：**是否启用分配担保

![image-20210313151555943](.\picture\image-20210313151555943.png)

#### 10、不分配在堆中对象的情况--代码优化

随着 **逃逸分析** 和 **标量替换**的 优化技术存在，可以在栈上分配。

* 若经过逃逸分析后，发现一个对象并没有逃逸方法的话，那么就可以分配在栈上。

* 逃逸分析：当一个对象在方法中定义之后，只在方法内部使用，没有逃逸。可以存于栈中。 

  ​					一个方法的对象，被方法外部方法引用，则发生逃逸。

  ![image-20210313173841217](.\picture\image-20210313173841217.png)

  逃逸分析的代码优化：

  * 栈上分配，不发生逃逸的可以进行栈上分配

    -----一般发生逃逸的情形：1. 成员变量赋值。 2. 方法返回值  3. 实例引用传递

  * 同步忽略，若一个对象只被一个线程访问，则可以不考虑同步。 JIT 编译器可以借助逃逸分析来判断同步块使用锁对象是否只能被一个锁对象访问。若只能一个线程访问，则可以取消同步，叫做锁消除。

  * 分离对象或者标量替换。将聚合量替换成标量

    <img src=".\picture\image-20210313193113031.png" alt="image-20210313193113031" style="zoom:67%;" />

    <img src=".\picture\image-20210313193133857.png" alt="image-20210313193133857" style="zoom:67%;" />

**TaoBaoVM :** GCIH : 将生命周较长的对象从堆中移除，并且GC不能管理GCIH中的对象。 

## 十、方法区（元空间）

#### 1、栈，堆，方法区的关系

<img src=".\picture\image-20210313195027608.png" alt="image-20210313195027608" style="zoom:67%;" />

<img src=".\picture\image-20210313195228020.png" alt="image-20210313195228020" style="zoom:67%;" />

​                                         如果在方法中 new 的 person ，则 person 在java栈中，属于局部变量。

​										 new的对象放于堆空间中。

<img src=".\picture\image-20210313195534736.png" alt="image-20210313195534736" style="zoom:67%;" />

#### 2、方法理解

* 方法区各线程共享
* 方法区在JVM被启动时被创建。可以是不连续的
* 方法区可以固定大小，也可以动态扩展
* 方法区大小决定了存放多少个类，加载jar包过多就会oom

#### 3、设置方法区的大小与OOM

JDK 7 -XX:PermSize 设置永久代初始的空间大小

​           -XX:MaxPermSize

JDK 8:   -XX:MetaSpaceSzie        Windows下默认21M

​			  -XX:MaxMetaSpaceSize Windows下默认 -1 即没有限制

注意：在默认的初始值21M中，即元空间的初始高水位线，一旦触及这个水位线，Full GC会被调用，回收没用的类，水位线会被重置。新的水位线取决于释放空间的大小。

为了避免GC 建议将初始值 设置的大一点。

**如何解决OOM？**

1. 解决 OOM 和 heap space 异常，首先需要通过内存映像分析，分析堆中的对象是否必要，区分是内存泄漏还是溢出

2. 若是内存泄漏则可通过工具查看GC roots引用，分析为什么无法回收这些对象。
3. 若是内存溢出。就应该检查设置堆大小的参数，适当调整内存大小。从代码找出某些生命周期较长的对象，尝试减少该类对象。

#### 4、方法区的内部结构

* 存储从JVM中加载的类型信息，常量，静态变量，即时编译器的代码缓存。

  1. 类型信息：对每个加载的类型（类，接口，枚举，注解），存储
     1. 完整的有效名称：包名加类名
     2. 直接父类的有效名称
     3. 这个类型的修饰符
     4. 类的接口的有序列表
  2. Field 信息
     1. JVM 在方法区保存类型的所有域的相关信息，以及声明顺序。
     2. 包括：域名称，域类型，修饰符，
  3. 方法信息：
     1. 方法名
     2. 方法参数类型
     3. 方法的修饰符
     4. 方法的字节码
     5. 异常表
  4. 运行时常量池
     1. 包含各种字面量和对应类型，域，方法的符号引用。**在Java 源文件中的接口，类编译之后的字节码会占很大的空间，不能直接存在字节码文件中，可以换一种方式存在常量池中，将引用存在常量池中**
     2. * 运行时常量池是方法区的一部分
        * 常量池（用于存放在编译器生成的各种字面量和符号引用）在经过类加载之后就存放于方法区中的运行时常量池。
        * JVM 为每个已加载的类型（类或者接口）维护一个常量池，池中的数据通过索引访问
        * 运行时常量池包含多种不同的常量，包括编译器就明确的字面量，也包括运行期解析之后得到的方法或者字段引用，即真是地址。

  <img src=".\picture\image-20210314224135884.png" alt="image-20210314224135884" style="zoom:67%;" />

<img src=".\picture\image-20210314221621442.png" alt="image-20210314221621442" style="zoom:67%;" />

#### 5、方法区的举例

#### 6、方法区的演变细节

<img src=".\picture\image-20210315104640717.png" alt="image-20210315104640717" style="zoom:67%;" />

**<img src=".\picture\image-20210315104951875.png" alt="image-20210315104951875" style="zoom:67%;" />**

<img src=".\picture\image-20210315105018178.png" alt="image-20210315105018178" style="zoom: 80%;" />

<img src=".\picture\image-20210315105051750.png" alt="image-20210315105051750" style="zoom:80%;" />

#### 7、方法区的一些问题

<span style='color:yellow'>为什么要使用元空间替换永久代？</span>

* 只有hotspot 有永久代，使用虚拟机内存来存储各方面。JRoKit，J9没有永久代，生命周期较长的不使用虚拟机内存使用本地内存。
* 为永久代设置大小是很难确定的。小了：频繁GC。大了：浪费。
* <span style='color:yellow'>元空间不在虚拟机中，而是用本地内存，默认情况下元空间大小无限，仅受本地内存的限制</span>
* 对永久代调优比较困难。回收判断的东西很多，Full GC比较麻烦。

<span style='color:yellow'>为什么字符串常量池（String Table 要变化？）</span>

JDK7 将 String Table放在了堆空间中。因为永久代回收效率低，Full GC才会触发。Full GC是因为老年代空间不足永久代才会触发，从而导致String Table回收效率过低。而开发中有大量的字符串被创建，放到堆中能提高回收效率。

<span style='color:yellow'>静态变量存在哪里？</span>

JDK 7 放在堆中，JDK8 也放在堆中。

<span style="color:pink">静态引用对应的对象实体始终都在堆空间中。JDK7以后的版本将静态变量与类的对象一起存放在堆中。</span>

#### 8、方法区的垃圾回收

主要回收的内容：常量池中废弃的常量，不在使用的类型。但是很难对类型进行回收，条件苛刻。

## 十一、运行时数据区总结

![image-20210315115929650](.\picture\image-20210315115929650.png)

<img src=".\picture\image-20210315120100279.png" alt="image-20210315120100279" style="zoom: 80%;" />

<img src=".\picture\image-20210315120136103.png" alt="image-20210315120136103" style="zoom:80%;" />

## 十二、对象的实例化与内存布局访问定位

#### 1、对象实例化

* **创建对象的方式：** 

  1. new 最常见，变形：单例模式（调用类的静态方法进行创建对象）。
  2. Class的 newInstance() 反射方式，只可以调用空参的方法，并且必须是 public 。
  3. Constructor 的 newIstance(Xxx), 反射方式，可以调用空参带参数的构造器，权限没有要求。
  4. Clone：不调用构造器，但是当前类实现 Cloneable接口，实现clone。
  5. 反序列化：从文件中，网络中获取一个对象的二进制流，再还原对象。

* **创建对象的步骤：**

  1. **加载类：**JVM收到 new 指令时，检查指令是否能在常量池中找到一个类的符号引用，检查这个类是否被加载，链接，初始化过。若未加载则先加载类。使用双亲委派模式，使用当前的类加载器ClassLoader + 包名 + 类名 寻找对应的 .class文件。若没找到则ClassNotFoundException。

  2. **为对象分配内存**

      * 堆空间，若内存规整，**指针碰撞**：指针后移，指针是区分空闲和非空闲的指针。**使用 Serial（串行）, ParNew（并行） 基于压缩算法带有compact过程的收集器时。指针碰撞**
      * 内存不规整：**空闲列表：**找到足够大的空间分配给对象。使用 **CMS这种基于 Mark-Sweep**的收集器，采用空闲列表。

  3. **产生安全问题时**：

     * 虚拟机采用 **CMS 加上失败重试的方法**保证更新操作的原子性
     * 将内存分配工作按照线程划分到不同的空间中进行， TLAB 本地线程分配缓冲区。哪个线程要分配，就在他自己的TLAB上分配，再TLAB使用完后，分配新的TLAB才需要同步锁定。 -XX:+/-UseTLAB。

  4. **默认初始化**，分配完 毕之后，需要进行分配的空间初始化问题（对对象的属性默认初始化为零）。使用 TLAB 可在TLAB分配时进行。

  5. **设置对象头：**对象属于哪一个类，类的元数据信息，对象的哈希码，对象的GC分代年龄信息。具体设置取决 JVM的实现

  6. **显示初始化：**经过以上五步之后，new了一个对象，接下来要进行 **init** 方法初始化。调用构造器，显示的初始化。

     ​		

#### 2、对象的内存布局

1. 对象头

   *  运行时元数据： “ Mark Word ”: 哈希码，GC分代年龄，锁状态标志，线程持有锁，偏向线程ID，偏向时间戳。 32为 32bit， 64位 64bit
   * 类型指针：确定该类的所属类型。

2. 实例数据：对象真正存储的有效信息，程序中所定义的字段内容。父类继承下来的字段，以及自己的字段。

   ​		规则：1. 父类中定义的变量出现在子类之前    2. 相同宽度的字段被分配一起   3.  +XX:CompactFields设        置为true，子类变量可以插入到父类的间隙中。

3. 对齐填充：因为 hotspot 要求自动内存管理 对象起始地址必须是8字节的整数倍，任何对象大小必须是8字节的整数倍。 对象的大小设计成正好是8字节的倍数。

   ```java
   public class Customer{
       public static void main(String args[]){
           Customer cust = new Customee();
       }
   }
   ```

   ![image-20210316081909504](.\picture\image-20210316081909504.png)

#### 3、对象访问定位

```java
public class Customer{
    public static void main(String args[]){
        Customer cust = new Customee();
    }
}
```

**在上面代码中 cust 存放于虚拟机栈中的局部变量表 LV中。怎么通过对象引用访问到对象的实例呢？**

 <img src=".\picture\image-20210316084612958.png" alt="image-20210316084612958" style="zoom:67%;" />

**栈帧中的引用记录了堆空间中的地址值。**

**访问方式**

1. 句柄访问：java 堆会划分一块空间作为 句柄池，reference存放的就是对象的句柄地址，句柄中包含了对象的示例数据与类型数据的具体地址。<span style='color:pink'>优点，在对象移动时只需要修改句柄地址就可以，不需要修改Reference 本身</span>
2. 直接指针：reference 存放的就是对象地址。<span style='color:pink'>访问快，减少了一次指针指针定位</span>

## 十三、直接内存

- 不属于JVM虚拟机运行时数据区的一部分。JDK8 云空间在直接内存（本地内存）。
- 属于java堆外的，直接向系统申请的内存区间。
- 访问速度由于Java堆，读写性能高。读写频繁考虑使用直接内存
- 分配回收成本高。不受JVM内存回收管理

[![image-20210316091340848](.\picture\image-20210316091340848.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

[![image-20210316091420240](.\picture\image-20210316091420240.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

设置直接内存大小 ： -XX:DirectMemorySize / 默认与 -Xmx 一样

## 十四、执行引擎

[![image-20210316122049423](.\picture\image-20210316122049423.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

#### 1、解释器VS编译器

**解释器：**一条一条解释执行代码。

**编译器：**将整个源代码都翻译为机器码（二进制，且每个系统的二进制码不一致），执行时不需要编译器，可以直接在支持目标代码的平台上运行，如此执行效率比较高。

#### 2、JIT

JIT Hotspot中的分类

- -Client : 使用 C1 编译器，对字节码进行简单的优化，耗时短。
- -Server ： 64位机器默认的是 C2 编译器，耗时长，激进优化。

C1优化策略：

- 方法内联：将引用的函数代码编译到引用点处，减少栈帧生成，减少参数传递以及跳转过程。
- 去虚拟化：对唯一的实现类进行内联
- 冗余消除：运行时把一些不会执行的代码消除

C2优化策略（基于逃逸分析）--c++：

- 栈上分配：将未逃逸的对象分配在栈中。
- 标量替换：用标量替换聚合对象的属性值。
- 同步消除：清楚同步操作，synchronized。

JVM 主要任务 加载字节码文件。但是字节码文件不能直接运行于OS上，执行引擎的任务将字节码解释/编译到对应的平台上，转化为机器指令。

[![image-20210316123108116](.\picture\image-20210316123108116.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

什么时解释器?什么JIT 编译器？

答: JVM启动时，根据预定义的规范对字节码文件逐条的解释，将字节码翻译为机器指令。JIT 就是虚拟机直接将源代码编译成本地平台相关的机器指令。

为什么java是半编译半解释性语言？

答: JVM执行代码时，通常会把解释执行和编译执行二者结合

#### 3、 热点代码

- 一个被多次调用的方法，或者方法中多次被调用的循环体。称为热点代码。
- 对于循环体出发的JIT，编译器会以整个方法座位编译对象，由于这种编译方法发生在方法执行过程中，又称之为栈上替换。
- 热点阈值由热点探测功能实现，Hotspot VM 采用基于计数式的热点探测。（采样热点探测）
  1. 方法调用计数器：统计方法被调用次数。Client 1500次，Server 10000 超过阈值触发JIT。 修改 -XX:CompileThreadhold
  2. 回边计数器：循环体执行次数

[![image-20210316140620840](.\picture\image-20210316140620840.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

- **热度衰减：**不加限制所有方法都能成为热点代码。所以统计一段时间内方法被调用的次数。当超过时间限制，次数未超过阈值，则该方法的调用计数器就会减少一半,称为热度衰减，半衰周期。 -XX:UseCounterDecay

[![image-20210316141034281](.\picture\image-20210316141034281.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

#### 4、总结

JDK10以后出了新的编译器 Graal 即时编译器

AOT 编译器， 在程序运行之前将字节码转化位机器码。

AOT好处：不必等待即使编译器的预热，减少了JAVA 应用给人带来的第一次运行慢的体验。

缺点：破坏了“一次编译，到处运行”，必须为每个硬件，os编译发行包。

 降低了Java链接过程的动态性。

## 十五、String Table

#### 1、基本特性

- String s1 = "czy"; // 字面量定义

  String s2 = new String("czy"); //对象定义

- String 声明为**final**不可被继承。

- 实现了 Serializable（序列化，可以跨进程传输）, Comparable, CharSequence 接口

- String 在 **JDK8 之前使用 final char[ ] value**存储字符串 ，**JDK9 ,11 使用 byte[ ]** 底层。大部分对象都是存储拉丁文，而拉丁文使用 1 byte就可以存储，所以将 UTF-16 char（占两个字节）替换为 byte数组，外加 encoding-flag field (其他字符集)

- 不可变

- 通过字面量的方式（区别于new）给字符串赋值，字符串值在字符串常量池中。

- 字符串常量池（String pool）不会存储相同的字符串。

  1. String 的 String Pool 是一个固定大小的String table，默认大小1009（JDK6）。若放进 String Pool 的 String 过多，造成hash冲突。导致链表长度过长。
  2. JDK7 默认长60013. -XX:StringTableSize 设置默认长度。
  3. JDK8 要求最小值是1009。

#### 2、String内存分配

- Java中 8 种数据类型，以及String。为了提高访问速度，提供常量池。

- 直接使用字面量声明的String对象会直接存储在常量池中。

- intern方法存于常量池。

- JDK 6 字符串常量池在永久代，JDK7将字符串常量池放在堆空间，JDK8 元空间，字符串常量在堆。

- [![image-20210316201255989](.\picture\image-20210316201255989.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

- [![image-20210316201359925](.\picture\image-20210316201359925.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

  **问题：为什么StringTable 要调整？**

  答: 1.permSize（永久代）默认比较小 2. 永久代回收频率低

#### 3、基本操作

```java
public static void main(String[] args){
    System.out.println("1");
    System.out.println("2");
    System.out.println("3");
    //一下操作不会在常量池中创建一样的字符串，具有相同的unicode
    System.out.println("1");
    System.out.println("2");
}
package JVM;

public class Memory {
    public static void main(String[] args) {
        int i = 1;
        Object obj = new Object();
        Memory mem = new Memory();
        mem.foo(obj);
    }

    private void foo(Object param) {
        //这里的toString调用的是Objectde的，底层通过StringBuilder进行拼接，会在堆内存中new一个String对象，因此这时的字			符串常量池没字符串对象。str指向堆String对象
        String str = param.toString();
        System.out.println(str);
    }
}
```

[![image-20210316205227289](.\picture\image-20210316205227289.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

#### 4、字符串拼接原理

1. 常量与常量拼接结果在常量池，原理是编译器优化。

2. 常量池不存在相同常量。

3. 拼接中只要有一个是变量，结果就在堆（ 堆空间中除去常量池的区域）中。变量拼接原理是StringBuilder。

4. 如果拼接的结果调用 intern() 则主动将常量池还没有的字符串对象放入池中，并返回此对象地址。

   ```java
   public void test4(){
           String s1 = "a" + "b" + "c";   //拼接结果在常量池，等同于"abc",编译器优化
           //一定在字符串常量池，因为s1的缘故常量池已经存在，将此地址返回给s2
           String s2 = "abc";
   
           System.out.println(s1 == s2);         //true
           System.out.println(s1.equals(s2));   //true
       }
   ```

   ```java
   public void test5(){
           String s1 = "JavaEE";
           String s2 = "hadoop";
   
           String s3 = "JavaEEhadoop";
           String s4 = "JavaEE" + "hadoop"; //拼接直接存常量池，编译器优化
           String s5 = s1 + "hadoop";
           String s6 = "JavaEE" + s2;
           String s7  = s1 + s2;
   
           System.out.println(s3 == s4);  //true 拼接直接存在常量池
           //false 因为有变量，new String()了结果存在了字符串常量池之外的堆空间中
           System.out.println(s3 == s5);
           System.out.println(s3 == s6);  //false 同理
           System.out.println(s3 == s7);  //false 同理
   
           System.out.println(s5 == s6);  //false
           System.out.println(s5 == s7);  //false
           System.out.println(s6 == s7);  //false
   
           //intern()在字符串常量池中是否存在该值，若存在则返回地址，不存在则在常量池加载一份，返回地址。
           String s8 = s6.intern();
           System.out.println(s3 == s8); //true
       }
   ```

   **底层原理：**

   ```java
   public void test3(){
           String s1 = "a";
           String s2 = "b";
           String s3 = "a" + "b";
           /*
           * 如下s1 + s2的执行细节：
           * 1. StringBuilder s = new StrinfBuilder();
           * 2. s.append("a"); //从LV中取得"a"的地址，根据地址在字符串常量池中找到"a"
           * 3. s.append("b");
           * 4. s.toString(); ----> 约等于于new String
           * */
           String s4 = s1 + s2;
           System.out.println(s3 == s1); //false
       }
   ```

   JDK5.0之后引入 StringBuilder(线程不安全),  5.0之前 StringBuffer

   ```java
   public void test4(){
           final String s1 = "a";
           final String s2 = "b";
           String s3 = "a" + "b";
   		//因为是常量引用，所以在编译期优化，非StringBuilder
       	//final 修饰类方法，基本数据类型，引用数据类型结构式尽量使用
           String s4 = s1 + s2;
           System.out.println(s3 == s1); //true
       }
   ```

   **拼接与append效率**

   ```java
   /*
       * 1. append方式自始至终只有一个对象，而拼接方式不断地创建了StringBuiler，和String
       * 2. 拼接创建了过多的对象，浪费空间，而且GC 花费时间
       * 3. 改进空间： 在实际开发中，若基本确定String的长度不高于 highLevel，建议使用构造器
       *    StringBuilder s = new StringBuilder(highlevel), 因为这样减少了频繁扩容的操作。
       * */    
   public void test6(){
           long start = System.currentTimeMillis();
   //        method1(100000);//3931
           method2(100000);//0
   
           long end = System.currentTimeMillis();
           System.out.println(end - start);
       }
   
       private void method1(int highlevel) {
           String src = "";
           for(int i = 0; i < highlevel;i++){
               src += "a"; //每次循环创建一个StringBuilder，还会创建String，因为src是变量
           }
       }
   
       private void method2(int i) {
           StringBuilder sre = new StringBuilder("");
           sre.append("a");\
       }
   ```

   append方式自始至终只有一个对象，而拼接方式不断地创建了StringBuiler，和String

   - 1. 拼接创建了过多的对象，浪费空间，而且GC 花费时间

   - 1. **改进空间**： 在实际开发中，若基本确定String的长度不高于 highLevel，建议使用构造器

     StringBuilder s = new StringBuilder(highlevel), 因为这样减少了频繁扩容的操作。

#### 5、intern（）

若不是双引号声明的对象，则可以使用 intern()， 从字符串常量池中判断是否存在，不存在放入常量池，存在返回 reference。

**如何保证 字符串s 指向 字符串常量池？**

- 字面量定义
- 调用 intern() 

**问题 new String("ab") 创建了几个对象？：**

答: 两个，第一个new 在堆空间中创建，第二个对象在常量池中创建

```java
    public void test8(){
         /*  0 new #24 <java/lang/String> //在堆中创建对象
             3 dup
             4 ldc #14 <ab>  //在常量池中创建 "ab"，取出
             6 invokespecial #26 <java/lang/String.<init>>
             9 astore_1
            10 return
            在常量池中有"ab"
        */
        String s1 = new String("ab");
         /*
            对象1 new StringBuilder()
            对象2 new String("a")
            对象3 常量池中的 "a"
            调用append(),对象1.append(a)
            对象4 new String("b")
            对象5 常量池中的"b"
            对象1.append("b")
            
            最后返回 toString()
            深入： 对象6 -- new Sring("ab")， toString() 字符串常量池中没有生成"ab"
        * */
        String str = new String("a") + new String("b");
    }

 
 // intern 方法
    public void test7(){
        String s = new String("1");
        s.intern();
        String s2 = "1";  //常量池中的引用
        //JDK6 false
        //JDK78 false
        /*
        new String("1") 堆空间创建对象"1"，常量池中"1" ,s 引用指向堆中对象引用
        s.intern() 在常量池中存在 "1"
        若修改了 s = s.intern() 则 s == s2; true 
        * */
        System.out.println(s == s2);
		
        //先有 “11” 但是不在字符串常量池中，因此，s3.intern() 会将字符串常量池中的 "11" 引用指向堆中对象s3的地址。
        String s3 = new String("1") + new String("1");
        //上行代码执行执行结束后，常量池不存在"11"
        s3.intern(); // 在字符串常量池中生成 "11"，JDK6中创建新的对象，新的地址。
                    // JDK78中， 字符串常量池在堆空间，没有创建 "11" 
                    // new String("11")存在，为了节省空间，常量池中"11"记录new String("11")地址
        String s4 = "11";  //使用上一行代码在常量池中生成的"11 "地址
        //JDK6 false
        //JDK78 true
        System.out.println(s3 == s4);
    }
```

[![image-20210316222831922](.\picture\image-20210316222831922.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

<span style='color:pink'>下图先有 “11” 对象且不在字符串常量池中（因为最后调用 toString() 方法，没有在字符串常量池中创建对象）， 所以在 调用 s3.intern() 时，字符串常量池中 " 11 " 指向 对象 s3 的地址</span>

[![image-20210316223820954](.\picture\image-20210316223820954.png)](https://github.com/Superczy1/JVMLearning/blob/main/JVM.md)

**intern()总结：**

* JDK6中，将字符串对象尝试放入常量池中，若已有则不放入，返回常量池中的地址；若没有则复制一份对象放入常量池中，并返回常量池中的地址。

* JDK78中，若已有则不会放入，返回已有的地址；若没有则把**对象的引用地址复制一份**，放入字符串常量池中，并返回**引用地址**。 

* 对于程序中大量存在的字符串，尤其是存在很多重复的字符串，使用 intern() 方法， 可以节省空间

  ```java
  arr[i] = new String(String.valueof("XXX"))
      //下面的方法因为，返回的时字符串常量池中的引用，这样可以释放很多字符串对象的引用，进而可以回收走。
  arr[i] = new String(String.valueof("XXXX")).intern();
  ```

```java
public void test8(){
        String s1 = new String("a") + new String("b");
        s1.intern();
        // inter() 方法 因为字符串常量池中没有 "ab" 所以常量池中直接指向对象s1的地址。
    	String s2 = "ab";  //s2 获取"ab"的地址。
        System.out.println(s1 == s2);
    }
```



#### 6、StringTable垃圾回收

**G1去重操作**![image-20210317171100628](.\picture\image-20210317171100628.png)

