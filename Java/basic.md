---
title: Java基础
layout: default
parent: Java核心技术
nav_order: 1
---





# 1、Exception和Error

**<font color=red>请对比 Exception 和 Error，另外，运行时异常与一般异常有什么区别？</font>**

<img src="https://static001.geekbang.org/resource/image/ac/00/accba531a365e6ae39614ebfa3273900.png">

Exception 和 Error 都是继承了 Throwable 类，在 Java 中只有 Throwable 类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。Exception 和 Error 体现了 Java 平台设计者对不同异常情况的分类。

Error 是指在正常情况下，不大可能出现的情况，绝大部分的 Error 都会导致程序（比如 JVM 自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获，常见的比如 OutOfMemoryError 之类，都是 Error 的子类。

Exception 是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。

Exception 又分为**可检查异常**（IOException / checked exception）和**不检查异常**（**运行时异常 RuntimeException**）。（声明式异常（IO）和非声明式异常（runtime））

可检查异常在源代码里必须显式地进行捕获处理，这是编译期检查的一部分。

不检查异常就是所谓的运行时异常，类似 NullPointerException、ArrayIndexOutOfBoundsException 之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译期强制要求。





**<font color=red>NoClassDefFoundError 和 ClassNotFoundException 有什么区别？</font>**

NoClassDefFoundError是**一个错误(Error)**，而 ClassNOtFoundException 是**一个异常**。

**ClassNotFoundException 产生的原因：**

Java支持使用 Class.forName 方法来动态地加载类，任意一个类的类名如果被作为参数传，递给这个方法都将导致该类被加载到 JVM 中。如果这个类在类路径中没有被找到，那么此时就会在运行时抛出 ClassNotFoundException 异常。

要解决这个问题，就要确保所需的类连同它依赖的包存在于类路径中。当 Class.forName 被调用的时候，类加载器会查找类路径中的类，如果找到了那么这个类就会被成功加载，如果没找到，那么就会抛出ClassNotFountException。

除了 Class.forName，ClassLoader.loadClass、ClassLoader.findSystemClass 在动态加载类到内存中的时候也可能会抛出这个异常。

另外还有一个导致 ClassNotFoundException 的原因就是：当一个类已经某个类加载器加载到内存中了，此时另一个类加载器又尝试着动态地从同一个包中加载这个类。

由于类的动态加载在某种程度上是被开发者所控制的，所以他可以选择 catch 这个异常然后采取相应的补救措施。

**NoClassDefFoundError 产生的原因：**

如果JVM或者ClassLoader实例尝试加载（可以通过正常的方法调用，也可能是使用new来创建新的对象）类的时候却找不到类的定义。要查找的类在编译的时候是存在的，运行的时候却找不到了。这个错误往往是你使用new操作符来创建一个新的对象但却找不到该对象对应的类。这个时候就会导致NoClassDefFoundError.

由于NoClassDefFoundError是有JVM引起的，所以不应该尝试捕捉这个错误。

解决这个问题的办法就是：查找那些在开发期间存在于类路径下但在运行期间却不在类路径下的类。



ClassNotFoundException发生在装入阶段。 
当应用程序试图通过类的字符串名称，使用常规的三种方法装入类，但却找不到指定名称的类定义时就抛出该异常。

NoClassDefFoundError： 当目前执行的类已经编译，但是找不到它的定义时

也就是说你如果编译了一个类B，在类A中调用，编译完成以后，你又删除掉B，运行A的时候那么就会出现这个错误

加载时从外存储器找不到需要的class就出现ClassNotFoundException 
连接时从内存找不到需要的class就出现NoClassDefFoundError



* 参考：[博客](https://cloud.tencent.com/developer/article/1153789)、[博客](https://www.cnblogs.com/kaleidoscope/p/13516648.html)



**<font color=blue>try-catch-finally 块，throw、throws 关键字</font>**







**<font color=blue>try-with-resources 和 multiple catch</font>**







设计原则一、**尽量不要捕获类似 Exception 这样的通用异常，而是应该捕获特定异常**





设计原则二、**不要生吞（swallow）异常**



* ==输出到日志（Logger）的举例？在分布式系统？==





**Throw early, catch late 原则**





**Checked Exception是什么**



- try-catch 代码段会产生额外的性能开销，或者换个角度说，它往往会影响 JVM 对代码进行优化，所以建议仅捕获有必要的代码段，尽量不要一个大的 try 包住整段的代码；与此同时，==利用异常控制代码流程，也不是一个好主意，远比我们通常意义上的条件语句（if/else、switch）要低效==。
- ==Java 每实例化一个 Exception，都会对当时的栈进行快照==，这是一个相对比较重的操作。如果发生的非常频繁，这个开销可就不能被忽略了。







# 2、final、finally、finalize

**谈谈 final、finally、 finalize 有什么不同？**

final 可以用来修饰类、方法、变量，分别有不同的意义，final 修饰的 class 代表不可以继承扩展，final 的变量是不可以修改的，而 final 的方法也是不可以重写的（override）。

finally 则是 Java 保证重点代码一定要被执行的一种机制。我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。

finalize 是基础类 java.lang.Object 的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。



- 我们可以将方法或者类声明为 final，这样就可以明确告知别人，这些行为是不许修改的。

- 使用 final 修饰参数或者变量，也可以清楚地避免意外赋值导致的编程错误，甚至，有人明确推荐将所有方法参数、本地变量、成员变量声明成 final。
- final 变量产生了某种程度的不可变（immutable）的效果，所以，可以用于保护只读数据，尤其是在并发编程中，因为明确地不能再赋值 final 变量，有利于减少额外的同步开销，也可以省去一些防御性拷贝的必要。



* 需要关闭的连接等资源，更推荐使用 Java 7 中添加的 try-with-resources 语句，因为通常 Java 平台能够更好地处理异常情况，编码量也要少很多。
* 一个finally的例子





利用上面的提到的 try-with-resources 或者 try-finally 机制，是非常好的回收资源的办法。如果确实需要额外处理，可以考虑 Java 提供的 Cleaner 机制或者其他替代方法



final 不是 immutable。final 只能约束 strList 这个引用不可以被赋值。





**如何实现 immutable 的类？**







实践中，因为 finalize 拖慢垃圾收集，导致大量对象堆积，也是一种典型的导致 OOM 的原因。





```java
 private void runFinalizer(JavaLangAccess jla) {
 //  ... 省略部分代码
 try {
    Object finalizee = this.get(); 
    if (finalizee != null && !(finalizee instanceof java.lang.Enum)) {
       jla.invokeFinalize(finalizee);
       // Clear stack slot containing this variable, to decrease
       // the chances of false retention with a conservative GC
       finalizee = null;
    }
  } catch (Throwable x) { }
    super.clear(); 
 }
```

Throwable生吞？





**Cleaner**



# 3、强引用、软引用、弱引用、幻象引用

**<font color=red>强引用、软引用、弱引用、幻象引用有什么区别？具体使用场景是什么？</font>**

不同的引用类型，主要体现的是**对象不同的可达性（reachable）状态和对垃圾收集的影响**。

所谓强引用（“Strong” Reference），就是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以被垃圾收集的了，当然具体回收时机还是要看垃圾收集策略。

软引用（SoftReference），是一种相对强引用弱化一些的引用，可以让对象豁免一些垃圾收集，只有当 JVM 认为内存不足时，才会去试图回收软引用指向的对象。JVM 会确保在抛出 OutOfMemoryError 之前，清理软引用指向的对象。软引用通常用来实现内存敏感的缓存，如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

==弱引用（WeakReference）==并不能使对象豁免垃圾收集，仅仅是提供一种访问在弱引用状态下对象的途径。这就可以用来构建一种没有特定约束的关系，比如，维护一种非强制性的映射关系，如果试图获取时对象还在，就使用它，否则重现实例化。它同样是很多缓存实现的选择。

对于幻象引用，有时候也翻译成虚引用，你不能通过它访问对象。幻象引用仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制，比如，通常用来做所谓的 ==Post-Mortem 清理机制==，我在专栏上一讲中介绍的 Java 平台自身 Cleaner 机制等，也有人利用幻象引用监控对象的创建和销毁。



<img src="https://static001.geekbang.org/resource/image/36/b0/36d3c7b158eda9421ef32463cb4d4fb0.png">





==如果我们错误的保持了强引用（比如，赋值给了 static 变量），那么对象可能就没有机会变回类似弱引用的可达性状态了，就会产生内存泄漏。==







**引用队列（ReferenceQueue）**





**reachability fence**







# 4、String、StringBuffer、StringBuilder

**<font color=red>理解 Java 的字符串，String、StringBuffer、StringBuilder 有什么区别？</font>**

**String** 是 Java 语言非常基础和重要的类，提供了构造和管理字符串的各种基本逻辑。它是典型的 Immutable 类，被声明成为 final class，所有属性也都是 final 的。也由于它的不可变性，类似拼接、裁剪字符串等动作，都会产生新的 String 对象。由于字符串操作的普遍性，所以相关操作的效率往往对应用性能有明显影响。（字符串操作不当可能会产生大量临时字符串，以及线程安全方面的区别）

**StringBuffer** 是为解决上面提到拼接产生太多中间对象的问题而提供的一个类，我们可以用 append 或者 add 方法，把字符串添加到已有序列的末尾或者指定位置。StringBuffer 本质是一个**线程安全**的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销，所以除非有线程安全的需要，不然还是推荐使用它的后继者，也就是 StringBuilder。

**StringBuilder** 是 Java 1.5 中新增的，在能力上和 StringBuffer 没有本质区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下进行字符串拼接的**首选**。

StringBuffer 和 StringBuilder 底层都是利用可修改的（char，JDK 9 以后是 byte）数组，二者都继承了 AbstractStringBuilder，里面包含了基本操作，区别仅在于最终的方法是否加了 synchronized。





**由于不可变，Immutable 对象在拷贝时不需要额外复制数据**









==我们粗略统计过，把常见应用进行堆转储（Dump Heap），然后分析对象组成，会发现平均 25% 的对象是字符串，并且其中约半数是重复的。如果能避免创建重复字符串，可以有效降低内存消耗和对象创建开销。==







**String 相关类的演进，比如 Java 9 中实现的巨大变化**

Java 9 中紧凑字符的改进









# 5、动态代理

**谈谈 Java 反射机制，动态代理是基于什么原理？**

反射机制是 Java 语言提供的一种基础功能，赋予程序在运行时**自省**（introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装 RPC 调用、面向切面的编程（AOP）。

实现动态代理的方式很多，比如 JDK 自身提供的动态代理，就是主要利用了上面提到的反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类似 ASM、cglib（基于 ASM）、Javassist 等。



这个题目给我的第一印象是稍微有点诱导的嫌疑，可能会下意识地以为动态代理就是利用反射机制实现的，这么说也不算错但稍微有些不全面。功能才是目的，实现的方法有很多。总的来说，这道题目考察的是 Java 语言的另外一种基础机制： 反射，它就像是一种魔法，引入运行时自省能力，赋予了 Java 语言令人意外的活力，通过运行时操作元数据或对象，Java 可以灵活地操作运行时才能确定的信息。而动态代理，则是延伸出来的一种广泛应用于产品开发中的技术，很多繁琐的重复编程，都可以被动态代理机制优雅地解决。

从考察知识点的角度，这道题涉及的知识点比较庞杂，所以面试官能够扩展或者深挖的内容非常多，比如：

- 考察你对反射机制的了解和掌握程度。
- 动态代理解决了什么问题，在你业务系统中的应用场景是什么？
- JDK 动态代理在设计和实现上与 cglib 等方式有什么不同，进而如何取舍？



**我们可以在运行时修改成员访问限制**







**绕过 API 访问控制**











# 6、int和Integer







- 我在专栏第 1 讲中介绍的 Java 使用的不同阶段：编译阶段、运行时，自动装箱 / 自动拆箱是发生在什么阶段？
- 我在前面提到使用静态工厂方法 valueOf 会使用到缓存机制，那么自动装箱的时候，缓存机制起作用吗？
- 为什么我们需要原始数据类型，Java 的对象似乎也很高效，应用中具体会产生哪些差异？
- 阅读过 Integer 源码吗？分析下类或某些方法的设计要点。





# 对比Vector、ArrayList、LinkedList









# 对比Hashtable、HashMap、TreeMap









# ConcurrentHashMap和线程安全



## 如何保证集合是线程安全的？





## ConcurrentHashMap如何实现高效地线程安全？







# IO和NIO，多路复用







# Java文件拷贝









# 接口和抽象类





