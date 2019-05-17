---
title: jvm内存溢出分析
date: 2018-06-23 20:33:04
tags: [jvm]
categories: jvm 
---

## 堆内存溢出 

  - 堆内存中主要存放对象、数组等，只要不断地创建这些对象，并且保证 GC Roots 到对象之间有可达路径来避免垃圾收集回收机制清除这些对象
    当这些对象所占空间超过最大堆容量时，就会产生 OutOfMemoryError 的异常。堆内存异常示例如下：
    ```
       /**
        * 设置最大堆最小堆：-Xms20m -Xmx20m
        * 运行时，不断在堆中创建OOMObject类的实例对象，且while执行结束之前，GC Roots(代码中的oomObjectList)到对象(每一个OOMObject对象)之间有可达路径，垃圾收集器就无法回收它们，最终导致内存溢出。
        */
       public class HeapOOM {
           static class OOMObject {
           }
           public static void main(String[] args) {
               List<OOMObject> oomObjectList = new ArrayList<>();
               while (true) {
                   oomObjectList.add(new OOMObject());
               }
           }
       }

    ```
  - 运行后会报异常，在堆栈信息中可以看到
    
    java.lang.OutOfMemoryError: Java heap space 的信息，说明在堆内存空间产生内存溢出的异常。
    
    新产生的对象最初分配在新生代，新生代满后会进行一次 Minor GC，如果 Minor GC 后空间不足会把该对象和新生代满足条件的对象放入老年代，老年代空间不足时会进行 Full GC，之后如果空间还不足以存放新对象则抛出 OutOfMemoryError 异常。
    
    常见原因：内存中加载的数据过多如一次从数据库中取出过多数据；集合对对象引用过多且使用完后没有清空；代码中存在死循环或循环产生过多重复对象；堆内存分配不合理；网络连接问题、数据库问题等
  
## 虚拟机栈/本地方法栈溢出
  - StackOverflowError：当线程请求的栈的深度大于虚拟机所允许的最大深度，则抛出StackOverflowError，简单理解就是虚拟机栈中的栈帧数量过多（一个线程嵌套调用的方法数量过多）时，就会抛出StackOverflowError异常
    + 最常见就是方法无限递归调用
    ```
       /**
        * 设置每个线程的栈大小：-Xss256k
        * 运行时，不断调用doSomething()方法，main线程不断创建栈帧并入栈，导致栈的深度越来越大，最终导致栈溢出。
        */
       public class StackSOF {
           private int stackLength=1;
           public void doSomething(){
                   stackLength++;
                   doSomething();
           }
           public static void main(String[] args) {
               StackSOF stackSOF=new StackSOF();
               try {
                   stackSOF.doSomething();
               }catch (Throwable e){//注意捕获的是Throwable
                   System.out.println("栈深度："+stackSOF.stackLength);
                   throw e;
               }
           }
       }
    ```
  - 上述代码执行后抛出：Exception in thread "Thread-0" java.lang.StackOverflowError 的异常
  
  - OutOfMemoryError：如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出 OutOfMemoryError
    + 我们可以这样理解，虚拟机中可以供栈占用的空间≈可用物理内存 - 最大堆内存 - 最大方法区内存，比如一台机器内存为 4G，系统和其他应用占用 2G，虚拟机可用的物理内存为 2G，最大堆内存为 1G，最大方法区内存为 512M
      那可供栈占有的内存大约就是 512M，假如我们设置每个线程栈的大小为 1M，那虚拟机中最多可以创建 512个线程，超过 512个线程再创建就没有空间可以给栈了，就报 OutOfMemoryError 异常了
      
    ![oomStack](oomStack.png "Optional title")
    
    + 栈上能够产生 OutOfMemoryError 的示例如下：
    
    ```
       /**
        * 设置每个线程的栈大小：-Xss2m
        * 运行时，不断创建新的线程（且每个线程持续执行），每个线程对一个一个栈，最终没有多余的空间来为新的线程分配，导致OutOfMemoryError
        */
       public class StackOOM {
           private static int threadNum = 0;
           public void doSomething() {
               try {
                   Thread.sleep(100000000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
           public static void main(String[] args) {
               final StackOOM stackOOM = new StackOOM();
               try {
                   while (true) {
                       threadNum++;
                       Thread thread = new Thread(new Runnable() {
                           @Override
                           public void run() {
                               stackOOM.doSomething();
                           }
                       });
                       thread.start();
                   }
               } catch (Throwable e) {
                   System.out.println("目前活动线程数量：" + threadNum);
                   throw e;
               }
           }
       }
    ```
    + 上述代码运行后会报异常，在堆栈信息中可以看到 java.lang.OutOfMemoryError: unable to create new native thread 的信息，无法创建新的线程，说明是在扩展栈的时候产生的内存溢出异常
    
    + 总结：在线程较少的时候，某个线程请求深度过大，会报 StackOverflow 异常，解决这种问题可以适当加大栈的深度（增加栈空间大小），也就是把 -Xss 的值设置大一些，但一般情况下是代码问题的可能性较大；在虚拟机产生线程时，无法为该线程申请栈空间了，会报 OutOfMemoryError 异常，解决这种问题可以适当减小栈的深度，也就是把 -Xss 的值设置小一些，每个线程占用的空间小了，总空间一定就能容纳更多的线程，但是操作系统对一个进程的线程数有限制，经验值在 3000~5000 左右。
      在 jdk1.5 之前 -Xss 默认是 256k，jdk1.5 之后默认是 1M，这个选项对系统硬性还是蛮大的，设置时要根据实际情况，谨慎操作
      
## 方法区溢出
  - 前面说到，方法区主要用于存储虚拟机加载的类信息、常量、静态变量，以及编译器编译后的代码等数据，所以方法区溢出的原因就是没有足够的内存来存放这些数据
  - 由于在 jdk1.6 之前字符串常量池是存在于方法区中的，所以基于 jdk1.6 之前的虚拟机，可以通过不断产生不一致的字符串（同时要保证和 GC Roots 之间保证有可达路径）来模拟方法区的 OutOfMemoryError 异常；但方法区还存储加载的类信息，
    所以基于 jdk1.7 的虚拟机，可以通过动态不断创建大量的类来模拟方法区溢出
    ```
       /**
        * 设置方法区最大、最小空间：-XX:PermSize=10m -XX:MaxPermSize=10m
        * 运行时，通过cglib不断创建JavaMethodAreaOOM的子类，方法区中类信息越来越多，最终没有可以为新的类分配的内存导致内存溢出
        */
       public class JavaMethodAreaOOM {
           public static void main(final String[] args){
              try {
                  while (true){
                      Enhancer enhancer=new Enhancer();
                      enhancer.setSuperclass(JavaMethodAreaOOM.class);
                      enhancer.setUseCache(false);
                      enhancer.setCallback(new MethodInterceptor() {
                          @Override
                          public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                              return methodProxy.invokeSuper(o,objects);
                          }
                      });
                      enhancer.create();
                  }
              }catch (Throwable t){
                  t.printStackTrace();
              }
           }
       }
    ```
  - 上述代码运行后会报 java.lang.OutOfMemoryError: PermGen space 的异常，说明是在方法区出现了内存溢出的错误
  
## 本机直接内存溢出
  - 本机直接内存（DirectMemory）并不是虚拟机运行时数据区的一部分，也不是 Java 虚拟机规范中定义的内存区域，但 Java 中用到 NIO 相关操作时（比如 ByteBuffer 的 allocteDirect 方法申请的是本机直接内存），也可能会出现内存溢出的异常
  
  - DirectMemory容量可通过-XX:MaxDirectMemorySize指定，如果不指定，默认与java堆的最大值Xmx一样。
  
    ```
       /**
        * VM Args: -xmx20M -XX:MaxDirectMemorySize=10M
        */
       public class DirectMemoryOOM{
       
          private static final int _1MB = 1024 * 1024 ;
          
          public static void main(String [] args)throws Exception{
          
             Field unsafeFild = Unsafe.class.getDeclaredFields()[0];
             unsafeField.setAccessible(true);
             Unsafe unsafe = (Unsafe) unsafeFileld.get(null);
             while(true){
                unsafe.allocateMemory(_1MB);
             }
          }
       }
    ```
  - 上述代码运行后会报 java.lang.OutOfMemoryError的异常，说明是在直接内存出现了内存溢出的错误
   
## 总结
  ![jvmStackTop](jvmStackTop.png "Optional title")
  