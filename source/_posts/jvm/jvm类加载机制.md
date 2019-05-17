---
title: jvm类加载机制
date: 2018-07-03 11:10:34
tags: [jvm]
categories: jvm
---

## 类加载器过程 
   - 从类被加载到虚拟机内存中开始，到卸御出内存为止，它的整个生命周期分为7个阶段，加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)、卸御(Unloading)。其中验证、准备、解析三个部分统称为连接
   ![classLoad](load1.png "Optional title")

### 加载
   - 在加载阶段，虚拟机主要完成三件事情：
     - 通过一个类的全限定名（比如 com.danny.framework.t）来获取定义该类的二进制流
     - 将这个字节流所代表的静态存储结构转化为方法区的运行时存储结构
     - 在内存中生成一个代表这个类的 java.lang.Class 对象，作为程序访问方法区中这个类的外部接口
     
### 验证

   - 验证的目的是为了确保 class 文件的字节流包含的内容符合虚拟机的要求，且不会危害虚拟机的安全
   - 文件格式验证：主要验证 class 文件中二进制字节流的格式，比如魔数是否已 0xCAFEBABY 开头、版本号是否正确等
   - 元数据验证：主要对字节码描述的信息进行语义分析，保证其符合 Java 语言规范，比如验证这个类是否有父类（java.lang.Object 除外），如果这个类不是抽象类，是否实现了父类或接口中没有实现的方法，等等
   - 字节码验证：字节码验证更为高级，通过数据流和控制流分析，确保程序是合法的、符合逻辑的
   - 符号引用验证：对类自身以外的信息进行匹配性校验，举个栗子，比如通过类的全限定名能否找到对应类、在类中能否找到字段名 / 方法名对应的字段 / 方法，如果符号引用验证失败，将抛出异常
   
### 准备
   - 正式为【类变量】分配内存并设置类变量【初始值】，这些变量所使用的内存都分配在方法区。注意分配内存的对象是“类变量”而不是实例变量，而且为其分配的是“初始值”，一般数值类型的初始值都为0，char类型的初始值为'\u0000'（常量池中一个表示Nul的字符串），boolean类型初始值为false，引用类型初始值为null
   - 但是加上final关键字比如public static final int value=123;在准备阶段会初始化value的值为123； 

### 解析
   - 解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程
     - 符号引用：简单的理解就是字符串，比如引用一个类，java.util.ArrayList 这就是一个符号引用，字符串引用的对象不一定被加载
     - 直接引用：指针或者地址偏移量。引用对象一定在内存（已经加载）
     
### 初始化
   - 在准备阶段，已经为类变量赋了初始值，在初始化阶段，则根据程序员通过程序定制的主观计划去初始化类变量的和其他资源，也可以从另一个角度来理解：初始化阶段是执行类构造器 <clinit>() 方法的过程，那 <clinit>() 到底是什么呢？
   
   - 在准备阶段，已经为类变量赋了初始值，在初始化阶段，则根据程序员通过程序定制的主观计划去初始化类变量的和其他资源，也可以从另一个角度来理解：初始化阶段是执行类构造器 <clinit>() 方法的过程，那 <clinit>() 到底是什么呢？
   
   - 下面看段代码来理解下：
   ```
      public class Parent {
          static {
              System.out.println("Parent-静态代码块执行");
          }
      
          public Parent() {
              System.out.println("Parent-构造方法执行");
          }
      
          {
              System.out.println("Parent-非静态代码块执行");
          }
      }
      
      public class Child extends Parent{
          private static int staticValue = 123;
          private int noStaticValue=456;
      
          static {
              System.out.println("Child-静态代码块执行");
          }
      
          public Child() {
              System.out.println("Child-构造方法执行");
          }
      
          {
              System.out.println("Child-非静态代码块执行");
          }
      
          public static void main(String[] args) {
              Child child = new Child();
          }
      }
   ```
   - 看下面的运行结果之前可以先猜测一下结果是什么，运行结果如下：
   ```
      Parent-静态代码块执行
      Child-静态代码块执行
      Parent-非静态代码块执行
      Parent-构造方法执行
      Child-非静态代码块执行
      Child-构造方法执行
   ```
   
   - 上面的例子中可以看到一个类从加载到实例化的过程中，静态代码块、构造方法、非静态代码块的加载顺序。无法看到静态变量和非静态变量初始化的时间，静态变量的初始化和静态代码块的执行都是在类的初始化阶段 （<clinit>()） 完成，非静态变量和非静态代码块都是在实例的初始化阶段 （<init>()） 完成

## 类加载器
 
### 类加载器的作用
   - 加载 class：类加载的加载阶段的第一个步骤，就是通过类加载器来完成的，类加载器的主要任务就是 “ 通过一个类的全限定名来获取描述此类的二进制字节流 ”，在这里，类加载器加载的二进制流并不一定要从 class 文件中获取，还可以从其他格式如zip文件中读取、从网络或数据库中读取、运行时动态生成、由其他文件生成（比如 jsp 生成 class 类文件）等
   
   - 从程序员的角度来看，类加载器动态加载class文件到虚拟机中，并生成一个 java.lang.Class 实例，每个实例都代表一个 java 类，可以根据该实例得到该类的信息，还可以通过newInstance()方法生成该类的一个对象
   
   - 确定类的唯一性：类加载器除了有加载类的作用，还有一个举足轻重的作用，对于每一个类，都需要由加载它的加载器和这个类本身共同确立这个类在 Java 虚拟机中的唯一性。也就是说，两个相同的类，只有是在同一个加载器加载的情况下才 “ 相等 ”，这里的 “ 相等 ” 是指代表类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果，也包括 instanceof 关键字对对象所属关系的判定结果
    
### 类加载器的分类

   - 以开发人员的角度来看，类加载器分为如下几种：启动类加载器（Bootstrap ClassLoader）、扩展类加载器（Extension ClassLoader）、应用程序类加载器（Application ClassLoader）和自定义类加载器（User ClassLoader），其中启动类加载器属于 JVM 的一部分，其他类加载器都用 java 实现，并且最终都继承自 java.lang.ClassLoader
   
   - 启动类加载器（Bootstrap ClassLoader）是由 C/C++ 编译而来的，看不到源码，所以在 java.lang.ClassLoader 源码中看到的 Bootstrap ClassLoader 的定义是 native 的 private native Class findBootstrapClass(String name);。启动类加载器主要负责加载 JAVA_HOME\lib 目录或者被 -Xbootclasspath 参数指定目录中的部分类，具体加载哪些类可以通过 System.getProperty("sun.boot.class.path") 来查看
   
   - 扩展类加载器（Extension ClassLoader）由 sun.misc.Launcher.ExtClassLoader 实现，负责加载 JAVA_HOME\lib\ext 目录或者被 java.ext.dirs 系统变量指定的路径中的所有类库，可以用通过 System.getProperty("java.ext.dirs") 来查看具体都加载哪些类
   
   - 应用程序类加载器（Application ClassLoader）由 sun.misc.Launcher.AppClassLoader 实现，负责加载用户类路径（我们通常指定的 classpath）上的类，如果程序中没有自定义类加载器，应用程序类加载器就是程序默认的类加载器
   
   - 自定义类加载器（User ClassLoader），JVM 提供的类加载器只能加载指定目录的类（jar 和 class），如果我们想从其他地方甚至网络上获取 class 文件，就需要自定义类加载器来实现，自定义类加载器主要都是通过继承 ClassLoader 或者它的子类来实现，但无论是通过继承 ClassLoader 还是它的子类，最终自定义类加载器的父加载器都是应用程序类加载器，因为不管调用哪个父类加载器，创建的对象都必须最终调用 java.lang.ClassLoader.getSystemClassLoader() 作为父加载器，getSystemClassLoader() 方法的返回值是 sun.misc.Launcher.AppClassLoader 即应用程序类加载器
   
### ClassLoader 与双亲委派模型

   - 下面看一下类加载器 java.lang.ClassLoader 中的核心逻辑 loadClass() 方法：
   ```
      protected Class<?> loadClass(String name, boolean resolve)
              throws ClassNotFoundException
          {
              synchronized (getClassLoadingLock(name)) {
                  // 检查该类是否已经加载过
                  Class c = findLoadedClass(name);
                  if (c == null) {
                      long t0 = System.nanoTime();
                      try {
                          if (parent != null) {//如果父加载器不为空，就用父加载器加载类
                              c = parent.loadClass(name, false);
                          } else {//如果父加载器为空，就用启动类加载器加载类
                              c = findBootstrapClassOrNull(name);
                          }
                      } catch (ClassNotFoundException e) {
                      }
      
                      if (c == null) {//如果上面用父加载器还没加载到类，就自己尝试加载
                          long t1 = System.nanoTime();
                          c = findClass(name);
                          sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                          sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                          sun.misc.PerfCounter.getFindClasses().increment();
                      }
                  }
                  if (resolve) {
                      resolveClass(c);
                  }
                  return c;
              }
          }
   ```
   - 这段代码的主要意思就是当一个类加载器加载类的时候，如果有父加载器就先尝试让父加载器加载，如果父加载器还有父加载器就一直往上抛，一直把类加载的任务交给启动类加载器，然后启动类加载器如果加载不到类就会抛出 ClassNotFoundException 异常，之后把类加载的任务往下抛，如下图：
   ![classLoad](ClassLoad.png "Optional title")
   
   - 通过上图的类加载过程，就引出了一个比较重要的概念——双亲委派模型，如下图展示的层次关系，双亲委派模型要求除了顶层的启动类加载器之外，其他的类加载器都应该有一个父类加载器，但是这种父子关系并不是继承关系，而是像上面代码所示的组合关系
   ![classLoad](ClassLoad1.png "Optional title")
   
   - 双亲委派模型的工作过程是，如果一个类加载器收到了类加载的请求，它首先不会加载类，而是把这个请求委派给它上一层的父加载器，每层都如此，所以最终请求会传到启动类加载器，然后从启动类加载器开始尝试加载类，如果加载不到（要加载的类不在当前类加载器的加载范围），就让它的子类尝试加载，每层都是如此
   
   - 那么双亲委派模型有什么好处呢？最大的好处就是它让 Java 中的类跟类加载器一样有了 “ 优先级 ”。前面说到了对于每一个类，都需要由加载它的加载器和这个类本身共同确立这个类在 Java 虚拟机中的唯一性，比如 java.lang.Object 类（存放在 JAVA_HOME\lib\rt.jar 中），如果用户自己写了一个 java.lang.Object 类并且由自定义类加载器加载，那么在程序中是不是就是两个类？所以双亲委派模型对保证 Java 稳定运行至关重要
   
## 思维导图
   ![classLoad](load2.png "Optional title")