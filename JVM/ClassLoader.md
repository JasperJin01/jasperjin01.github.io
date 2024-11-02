---
title: Java类加载
layout: default
parent: JVM
---


# Java类加载
{: .no_toc }




1. TOC
{:toc}




## Java的类加载机制：





> 类的加载只能被加载一次吗？（==被同一个类的加载器加载一次==。类的加载有一个命名空间，类和类的加载器有一个双向引用的关系）
>
> Java 的类加载器使用不同的命名空间隔离不同类加载器加载的类。也就是说，**即使两个类有相同的名字**，如果它们是由**不同的类加载器**加载的，它们会被视为**不同的类**。



加载到内存中的类叫做类模板，类模板放在方法区。类的引用在栈里面，类的实例在堆里面



## JVM的启动流程



**Class类的构造方法是私有的，只有JVM能够创建。**



### JVM启动流程

最终运行的是字节码class文件 

<img src="https://github.com/jasperjin01/Photo/raw/main/java/jvmstart.png" alt="JVM Start Image">



**启动JVM**：

- 运行 `java com.tuling.jvm.Math` 命令，这个命令会触发 Java 虚拟机的启动。在 Windows 系统下，==`java.exe` 会调用底层的 `jvm.dll` 文件==，这是一个由C++实现的动态链接库，==用于启动JVM==。

**创建引导类加载器实例**：

- 启动JVM后，首先会==创建一个引导类加载器（Bootstrap ClassLoader）的实例==。引导类加载器是JVM的根类加载器，它负责加载核心Java类库，比如`java.lang`、`java.util`等，这些类存放在JDK的`rt.jar`文件中。

**创建Launcher**：

- 在创建完引导类加载器后，JVM会使用C++代码调用Java代码，创建一个`sun.misc.Launcher`类的实例。`sun.misc.Launcher`是负责启动JVM的类，由引导类加载器加载，并负责创建和管理其他类加载器（比如应用程序类加载器）。

**获取应用程序类加载器（AppClassLoader）**：

- `sun.misc.Launcher`的`getLauncher()`方法会被调用，以便获取运行类的加载器实例。在这里，得到的是应用程序类加载器的实例（AppClassLoader），它负责加载类路径下的应用程序类和第三方库。

**加载目标类**：

- 应用程序类加载器通过 ==`getClassLoader()` 返回一个类加载器实例==，用于加载目标类。在这里，目标类是 `com.tuling.jvm.Math`。
- ==使用 `loadClass("com.tuling.jvm.Math")` 方法加载目标类==，这一步会将 `Math` 类加载到JVM内存中，准备供后续执行。（**后面会详细说明加载类的过程**）

**执行目标类的 main 方法**：

- 当目标类 `Math` 被成功加载后，JVM会调用它的 `main` 方法作为程序的入口点。`main` 方法执行完毕后，Java程序运行结束。





### 类的加载

**类加载过程的五个阶段：加载 >> 验证 >> 准备 >> 解析 >> 初始化**

**加载 (Loading)**：

- 在上图中，`classLoader.loadClass("com.tuling.jvm.Math")` 就是加载阶段的开始。JVM会找到并加载目标类（这里是 `com.tuling.jvm.Math`）的字节码文件，并将其读入内存。

**验证 (Verification)**：

- 加载后，JVM会对字节码进行验证，以确保它符合Java语言规范，且不会对JVM安全性造成威胁。这个步骤是为了防止非法字节码（如被恶意修改过的类文件）导致系统崩溃。
- 虽然在上图中没有具体表现出来，但验证过程是在 `loadClass` 后自动进行的。

**准备 (Preparation)**：

- 在这个阶段，JVM会为类的静态变量分配内存，并赋予默认的初始值。例如，`int` 类型的变量会被赋值为 `0`。这一步也不是直接在图中展示出来，但它在 `loadClass` 之后紧接着发生。
- 需要注意的是，==赋值的是默认值而不是用户定义的初始值==。

**解析 (Resolution)**：

- 解析是将类、接口、字段、方法的符号引用替换为直接引用的过程。这是指把类文件中的符号（如方法名、变量名等）解析成实际内存地址的过程。
- 这一步同样没有在上图中详细表示，但它会在 `loadClass` 和 `Math.main()` 执行前完成。

**初始化 (Initialization)**：

- 这个阶段是执行类的 `<clinit>` 方法的过程。即会对静态变量赋用户定义的初始值，执行静态代码块。
- 在上图中，`Math.main()` 之前就是初始化阶段。当类完成初始化后，`main` 方法被执行，程序正式开始运行。





## Java类的加载

### 1. 加载 (Loading)

加载是类加载的第一个阶段，这一阶段的主要任务是将类的字节码文件（`.class` 文件）从硬盘加载到内存中。具体步骤如下：

- **查找并读取类文件**：当 JVM 碰到类的引用（如 `new` 一个对象或调用 `main()` 方法）时，JVM 首先在硬盘上查找该类的字节码文件（`.class` 文件）。

- 生成 Class 对象

    ：一旦找到并读取了该字节码文件，JVM 会在内存中生成一个 

    ```
    java.lang.Class
    ```

     对象，作为该类的各种信息的入口。

    - **Class 对象的作用**：`Class` 对象在 JVM 中作为一个类的代表，它保存着该类的字节码数据和相关的元数据信息，后续对该类的所有访问都是通过 `Class` 对象完成的。

- **方法区中的数据**：`Class` 对象会保存类的结构信息，包括类名、包名、字段、方法、父类、接口等，并作为访问这些数据的入口。方法区是 JVM 中专门存储类信息的区域。

在这个阶段，类已经被加载到 JVM 中，但还没有执行任何代码（如静态变量的初始化），只是简单地在内存中创建了类的 `Class` 对象。

### 2. 验证 (Verification)

验证是为了确保字节码的安全性和正确性，避免非法字节码导致 JVM 崩溃。JVM 的验证阶段包含了几个步骤：

- **文件格式验证**：检查字节码文件的基本格式，确保它符合 Java Class 文件的规范。比如，文件开头的魔数值（`0xCAFEBABE`）必须正确。
- **元数据验证**：检查类的元数据，例如确保该类不会与系统中的其他类冲突，且不会有重复的字段或方法。
- **字节码验证**：这一部分是为了确保代码逻辑是安全的。比如，方法中的字节码是否能正确使用操作数栈，类型转换是否合法等。
- **符号引用验证**：检查符号引用是否指向了合法的类、方法和字段。符号引用是在字节码中对其他类或方法的引用，但在加载时还未解析为直接引用（即具体的内存地址或句柄）。

这个阶段的目标是确保加载的字节码不会破坏 JVM 的安全性和稳定性。

### 3. 准备 (Preparation)

在准备阶段，JVM 会为类的 **静态变量** 分配内存并赋予默认值。需要注意的是，这里的默认值并不是你在代码中定义的初始值，而是数据类型的初始值。例如：

- `int` 类型的变量默认值是 `0`。
- `boolean` 类型的变量默认值是 `false`。
- 引用类型（如 `String` 或 `Object`）的默认值是 `null`。

**为什么不直接赋用户定义的初始值？** 在这个阶段，只是为静态变量分配内存和默认值，具体的初始值赋值会在后面的**初始化阶段**完成。这个阶段的分配操作只是在方法区中开辟静态变量的存储空间，并赋上默认值。

### 4. 解析 (Resolution)

解析是将**符号引用**替换为**直接引用**的过程。这个步骤在类加载时进行，也叫**静态链接**。这里是对符号引用和直接引用的简要说明：

- **符号引用**：在字节码文件中，对其他类、方法或字段的引用是以符号（如类名、方法名）表示的，它并没有具体的内存地址。
- **直接引用**：解析过程中，JVM 会将符号引用转换成实际的内存地址或方法句柄等直接引用，方便后续直接调用。

#### 静态链接 vs 动态链接

- **静态链接**：在类加载期间完成的解析，比如将 `main()` 方法的符号引用解析为实际指向该方法在内存中的位置。
- **动态链接**：在运行时将符号引用解析为直接引用。这种情况多见于方法的重写或动态代理，例如 Java 的多态机制（调用父类方法时，具体指向的子类实现是在运行时动态决定的）。

解析阶段的重点是确保在后续执行时可以快速访问方法、字段等，而不需要再查找符号引用。

### 5. 初始化 (Initialization)

初始化阶段是类加载过程的最后一步，这里 JVM 会执行类的 `<clinit>` 方法，对静态变量赋初始值，并执行静态代码块。

**静态变量赋初始值**：在这个阶段，JVM 会将用户定义的初始值赋给静态变量。例如：

```
public static int count = 10;  // 初始化时 count 被赋值为 10
```

**执行静态代码块**：如果类中定义了静态代码块，JVM 也会在此阶段执行这些代码。

```
static {
    System.out.println("Class initialized");
}
```

静态代码块通常用于类的初始化逻辑，只会在类第一次加载时执行一次。







符号（java的类、方法、变量等都会在常量池符号中。也就是javap得到的#x的内容）！符号在常量池中。符号会指向（内存？）反正就是指向一个什么东西。如果在初始化的时候就确定了指向的内容，就叫做静态链接。如果只有执行到这段代码，才能确认指向的内容，就叫做动态链接。（好像是要指向内存中代码的位置？）

```shell
javap -v Math.class
```

(base) jmjin@JmJindeMacBook-Pro JUC % cd src/class01/base/abc/safeend 
(base) jmjin@JmJindeMacBook-Pro safeend % javap -v EndRunnable.class     
错误: 找不到类: EndRunnable.class
(base) jmjin@JmJindeMacBook-Pro safeend % ls
EndRunnable.java           HasInterrputException.java SelfInterruptedExp.java
EndThread.java             SelfInterruptThread.java
(base) jmjin@JmJindeMacBook-Pro safeend % javap -v EndRunnable      
错误: 找不到类: EndRunnable
(base) jmjin@JmJindeMacBook-Pro safeend % javap -v EndRunnable.java

**<font color=red>为啥不对啊？？？</font>**





### 动态加载/懒加载

java的加载是一种懒加载机制。例如，一个jar包中有多个类，只有用到某个类时，会去加载这个类文件，而不会把全部文件全部加载。

```java
public class DynamicLoad {
    static {
        // 静态代码块，在初始化时执行的
        System.out.println("load TestDynamicLoad");
    }

    public static void main(String[] args) {
        new A();
        System.out.println("load test");
        B b = null;
    }
}
class A {
    static {
        System.out.println("load A");
    }
    public A() {
        System.out.println("init A");
    }
}
class B {
    static {
        System.out.println("load B");
    }
    public B() {
        System.out.println("init B");
    }
}

// load TestDynamicLoad
// load A
// init A
// load test
```











**永久代和元空间**

**永久代（Permanent Generation）**和**元空间（Metaspace）**是用来存储类的元数据的内存区域，但它们在JVM不同版本中。

永久代是JVM在Java 7及之前版本中用于存放类的元数据的内存区域。从Java 8开始，永久代被移除，取而代之的是元空间。





## ClassLoader类加载器

Java 中的类加载器负责将类字节码文件（`.class` 文件）从不同的路径加载到 JVM 中。Java 类加载器的层级结构遵循委派模型，即一个类加载请求会先从顶层的引导类加载器开始，逐层向下传递，如果上层加载器无法找到目标类，才会交给下层的类加载器。这种结构保证了 Java 类的安全性和稳定性。以下是主要的几种类加载器的详细介绍：

### 1. 引导类加载器（Bootstrap ClassLoader）

- **作用**：引导类加载器是 JVM 的根类加载器，它负责加载 JRE 核心类库中的类，这些类是 JVM 运行所必需的，例如 `java.lang.*`、`java.util.*` 包中的类。引导类加载器在 JVM 启动时自动加载，并且是用 **原生代码（C/C++）** 实现的。
- **加载路径**：引导类加载器会加载 `JRE/lib` 目录下的类库，特别是 `rt.jar`、`charsets.jar` 等文件，这些文件包含了 JVM 的核心类。
- **不可直接访问**：引导类加载器是一个内置的类加载器，程序员无法直接访问，也不能用 Java 代码进行操作。引导类加载器的 `ClassLoader` 实例在 Java 层次上表现为 `null`。

### 2. 扩展类加载器（Extension ClassLoader）

- **作用**：扩展类加载器用于加载扩展类库，通常是 Java 扩展的功能或库。它的主要作用是扩展 Java 核心类库的功能，通常这些类并不是 JVM 核心运行所必需的，但提供了额外的支持。
- **加载路径**：扩展类加载器加载 `JRE/lib/ext` 目录下的类库，这些类库通常存储在 `.jar` 文件中，可以通过将 `.jar` 文件放到这个目录下来扩展 JRE 的功能。
- **可以扩展和自定义**：扩展类加载器也可以通过系统属性 `java.ext.dirs` 来配置，从而指定其他扩展类库路径。程序员可以用 Java 代码间接访问扩展类加载器，通过 `ClassLoader.getSystemClassLoader().getParent()` 获取它的实例。

### 3. 应用程序类加载器（Application ClassLoader）

- **作用**：应用程序类加载器（也叫系统类加载器）是加载用户自定义类的默认类加载器，负责加载 **ClassPath** 路径下的类。开发者编写的代码和第三方库通常会由应用程序类加载器加载。
- **加载路径**：应用程序类加载器会加载 `ClassPath` 中指定的路径，例如项目的 `src` 或 `bin` 目录下的 `.class` 文件或项目的 `lib` 目录下的 JAR 包。
- **程序员可以访问**：应用程序类加载器是由 Java 代码实现的，属于 Java 程序的一部分。可以通过 `ClassLoader.getSystemClassLoader()` 获取其实例，程序员也可以通过这个加载器加载指定路径下的类文件。

### 4. 自定义类加载器（Custom ClassLoader）

- **作用**：自定义类加载器是由开发者自己定义的类加载器，用于加载一些特殊路径下的类或需要自定义加载逻辑的类。例如，动态加载网络上的类，解密后加载加密过的类文件，或热加载更新的类文件等。自定义类加载器通常用于特殊场景，如插件系统、模块化加载等。
- **实现方式**：要创建一个自定义类加载器，开发者需要继承 `java.lang.ClassLoader` 类并重写 `findClass` 和 `defineClass` 方法。通过自定义类加载器，开发者可以实现类加载过程的定制，例如从网络、数据库或加密存储中加载类文件。
- **应用场景**：自定义类加载器广泛用于 Web 应用服务器（如 Tomcat），以实现类的隔离和动态加载。同时，也可以用于游戏引擎、插件系统等需要加载第三方代码的应用场景。



### Launcher 源码解析

结合上面的图，看一下Launcher类的源码。

Launcher类的构造函数中可以看到他最核心的两件事：构造了ExtClassLoader和AppClassLoader，把AppClassLoader的parent设置为ExtClassLoader。

```java
public class Launcher {
    private static URLStreamHandlerFactory factory = new Factory();
    private static Launcher launcher = new Launcher(); // 单例
    private static String bootClassPath =
        System.getProperty("sun.boot.class.path");

    public static Launcher getLauncher() { // 获取单例
        return launcher;
    }

    private ClassLoader loader;
    
    public Launcher() {
        // Create the extension class loader
        ClassLoader extcl;
        try {
            // 获取扩展类加载器，ExtClassLoader是继承URLClassLoader的类
            extcl = ExtClassLoader.getExtClassLoader();
        } catch (IOException e) {
            throw new InternalError(
                "Could not create extension class loader", e);
        }

        // Now create the class loader to use to launch the application
        try {
            // 类加载器
            loader = AppClassLoader.getAppClassLoader(extcl);
        } catch (IOException e) {
            throw new InternalError(
                "Could not create application class loader", e);
        }

        // Also set the context class loader for the primordial thread.
        Thread.currentThread().setContextClassLoader(loader);

        // Finally, install a security manager if requested
        String s = System.getProperty("java.security.manager");
        if (s != null) {
            // init FileSystem machinery before SecurityManager installation
            sun.nio.fs.DefaultFileSystemProvider.create();


```

getLauncher中的Launcher是一个单例，在初始化的时候就初始化好了。

```java
public abstract class ClassLoader {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    // The parent class loader for delegation
    // Note: VM hardcoded the offset of this field, thus all new fields
    // must be added *after* it.
    private final ClassLoader parent;

```

ClassLoader中有一个叫做parent的ClassLoader属性，而上面提到的所有接加载器都是继承自ClassLoader的。所以**每一个类加载器都有一个parent属性。**

* **Bootstrap ClassLoader** 由 JVM 在启动时自动创建，并不是 Java 代码的一部分，因此不需要在 `Launcher` 中初始化。

* **ExtClassLoader** 的父类加载器是 **Bootstrap ClassLoader**，它通过 JVM 的默认机制继承，不需要显式传递 `parent` 参数。









**ClassLoader**

loadClass方法和findClass方法：



findClass方法是一个空实现，需要重写，这个是具体加载类的一个过程。findClass方法调用defineClass实现类的加载，调用了很多本地方法（native）实现类加载的逻辑。





## 双亲委派

### 类加载器的委派模型

在 Java 中，类加载器使用一种叫做 **双亲委派模型**（Parent Delegation Model）的机制来加载类。这种机制确保了安全性和防止类的重复加载，具体流程如下：

1. **向上委派**：当一个类加载器接到加载请求时，它会先将请求传递给它的父类加载器，从最顶层的引导类加载器开始查找。
2. **逐层向下**：如果上层加载器找不到该类，才会交给下层的加载器继续查找。
3. **加载过程**：如果没有上层加载器可以加载该类，最终请求会落到当前类加载器上，由它负责加载。

这样做的好处是避免了重复加载和类的安全性问题，因为核心类库只会由引导类加载器加载，而不会被应用程序或自定义类加载器覆盖。





### 双亲委派 源码解析 ClassLoader.loadClass()

**向上委派过程**

Java 的类加载器 `ClassLoader` 中有一个 `loadClass` 方法，用于加载类。双亲委派模型的核心就在于 `loadClass` 方法的实现。具体的委派逻辑如下：

1. 当类加载器接到加载请求时，会首先将请求委派给其父加载器。
2. 如果父加载器无法找到该类，则由当前加载器尝试加载。
3. 如果加载成功，返回该类；否则，抛出 `ClassNotFoundException` 异常。

ClassLoader.java中`ClassLoader.loadClass` 方法的源码解析

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // 1. 检查类是否已经被加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                // 2. 如果有父加载器，委派给父加载器
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    // 3. 如果没有父加载器，默认使用引导类加载器（Bootstrap ClassLoader）
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // 如果父类加载器找不到该类，当前类加载器将继续尝试加载
            }

            // 4. 如果父加载器找不到该类，调用当前类加载器的 findClass 方法加载
            if (c == null) {
                c = findClass(name);
            }
        }

        // 5. 解析类（链接类）
        if (resolve)  resolveClass(c);
        return c;
    }
}

```



```java
// Launcher.java launcher.getClassLoader()
public ClassLoader getClassLoader() {
    return loader;
}
```

==getClassLoader拿到loader后加载类，private ClassLoader loader; loader在构造函数中可以看到就是AppClassLoader。== 这个是干啥的



**逐层向下过程**：

ExtClassLoader和AppClassLoader没有findClass方法，但是他们继承自URLClassLoader，URLClassLoader中的findClass方法，findClass方法是实际从指定url中查找并加载类的方法。

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    String path = name.replace('.', '/').concat(".class");
    try {
        // 打开 URL 连接并读取类数据
        byte[] classData = getClassData(path); // 读取类的字节码
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        // 调用 defineClass 方法，将字节码定义为 Class 对象
        return defineClass(name, classData, 0, classData.length);
    } catch (IOException e) {
        throw new ClassNotFoundException(name, e);
    }
}
// 如果加载不成功（在ClassLoader所包含的包不包含要引入的那个类，就不会返回return的Class对象，所以就会报错
```

### findClass 方法工作流程

**将类名转换成路径**：

- 类名是以 `.` 分隔的，例如 `com.example.MyClass`。`findClass` 会将类名转换成文件路径，例如 `com/example/MyClass.class`。
- 这样做是为了能够找到文件系统中的实际文件，或通过 URL 定位类文件。

**读取类文件的字节码** (`getClassData`)：

- `getClassData` 方法用于从 URL 或文件系统中读取 `.class` 文件的内容，将其转换成字节数组 `byte[]`。
- 这个字节数组包含了类的字节码，即 `.class` 文件的内容。URLClassLoader 支持从多个源（如本地文件、远程服务器等）读取类数据，这就是它在加载外部依赖或扩展时的优势。

**调用 defineClass 方法**：

- `findClass` 方法会将读取到的字节码传递给 ==`defineClass` 方法==，==将class字节码转换成 JVM 可以理解的 `Class` 对象==。（也就是刚刚说的[类加载](#java类的加载)）
- `defineClass` 方法是类加载过程中非常关键的一步，负责将字节码定义成 JVM 中的类对象。







findClass方法调用defineClass方法



### 一个报错的问题

自己定义一个java类，然后运行时：

```java
package java.lang;
public class String {
    public static void main() {
        System.out.println("My String Class");
    }
}
```

报错内容为：

错误: 在类 java.lang.String 中找不到 main 方法, 请将 main 方法定义为:
   public static void main(String[] args)
否则 JavaFX 应用程序类必须扩展javafx.application.Application

请问这是为什么呢？



说白了，对于java已经定义好的类，双亲委派机制可以很好的确保能够加载到java本身的核心类，而不是自定义的类。原因是，机制会将URL（java.lang.String）一直向上委派给BootStrapClassLoader，他的加载路径只包含java定义的String类。

（为什么找不到main？因为java自带的String没有main函数）





## 全盘负责委托机制







## 自定义类的加载器

自己定义的类加载器，默认的parent是AppClassLoader。为什么呢？

自定义的MyClassLoader继承自ClassLoader，在初始化MyClassLoader时会首先调用ClassLoader的构造函数private ClassLoader()，可以查看下面的代码片段，可知道parent设置为getSystemClassLoader()。



```java
// ClassLoader.java片段 
protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}

private ClassLoader(Void unused, ClassLoader parent) {
    this.parent = parent;
    if (ParallelLoaders.isRegistered(this.getClass())) {
        parallelLockMap = new ConcurrentHashMap<>();
        package2certs = new ConcurrentHashMap<>();
        assertionLock = new Object();
    } else {
        // no finer-grained lock; lock on the classloader instance
        parallelLockMap = null;
        package2certs = new Hashtable<>();
        assertionLock = this;
    }
}

@CallerSensitive
public static ClassLoader getSystemClassLoader() {
    initSystemClassLoader();
    if (scl == null) {
        return null;
    }
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkClassLoaderPermission(scl, Reflection.getCallerClass());
    }
    return scl;
}

// initSystemClassLoader()方法会把系统类加载器（scl）设置为 Launcher.getLauncher().getClassLoader(), 这个可以去看上面的Launcher，呼应上了
```







## 打破双亲委派机制

如何打破双亲委派机制？如果你弄懂了上面讲解的ClassLoader代码，你就会明白，双亲委派机制就是因为ClassLoader.loadClass()方法会先调用parent的loadClass方法。所以如果想打破双亲委派机制，只需要重写loadClass方法。



沙箱安全机制：那既然如此，是否可以通过重写一个loadClass方法，通过重写的方法加载自己写的Object.class呢？答案是不可以，会触发java.lang.SecurityException

