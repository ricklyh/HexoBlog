---
title: ThreadLocal原理分析
date: 2019-02-2 23:30:03
tags: [java]
categories: java
---
## 什么是ThreadLocal
   - ThreadLocal提供了线程的局部变量，每个线程都可以通过set,get来对这个局部变量进行操作且不会影响其他线程的局部变量。防止冲突，实现线程的数据隔离，以到达线程的安全
   
## ThreadLocal的使用
   ```
      public class ThreadLocalTest {
      
          //(1)打印函数
          static void print(String str){
              //1.1  打印当前线程本地内存中localVariable变量的值
              System.out.println(str + ":" +localVariable.get());
              //1.2 清除当前线程本地内存中localVariable变量
              //localVariable.remove();
          }
          //(2) 创建ThreadLocal变量
          static ThreadLocal<String> localVariable = new ThreadLocal<>();
          public static void main(String[] args) {
      
              //(3) 创建线程one
              Thread threadOne = new Thread(new  Runnable() {
                  public void run() {
                      //3.1 设置线程one中本地变量localVariable的值
                      localVariable.set("threadOne local variable");
                      //3.2 调用打印函数
                      print("threadOne");
                      //3.3打印本地变量值
                      System.out.println("threadOne remove after" + ":" +localVariable.get());
      
                  }
              });
              //(4) 创建线程two
              Thread threadTwo = new Thread(new  Runnable() {
                  public void run() {
                      //4.1 设置线程one中本地变量localVariable的值
                      localVariable.set("threadTwo local variable");
                      //4.2 调用打印函数
                      print("threadTwo");
                      //4.3打印本地变量值
                      System.out.println("threadTwo remove after" + ":" +localVariable.get());
      
                  }
              });
              //(5)启动线程
              threadOne.start();
              threadTwo.start();
          }
   ```
   - 运行结果：
   ```
      threadOne:threadOne local variable
      threadTwo:threadTwo local variable
      threadOne remove after:threadOne local variable
      threadTwo remove after:threadTwo local variable
      
      解开代码 1.2 的注释后，再次运行，运行结果为：
      
      threadOne:threadOne local variable
      threadOne remove after:null
      threadTwo:threadTwo local variable
      threadTwo remove after:null
   ```
## ThreadLocal实现原理
   ![ThreadLocal](ThreadLocal.png "Optional title")
   - 如上类图可知 Thread 类中有一个 threadLocals 和 inheritableThreadLocals 都是 ThreadLocalMap 类型的变量，而 ThreadLocalMap 是一个定制化的 Hashmap，默认每个线程中这个两个变量都为 null，只有当前线程第一次调用了 ThreadLocal 的 set 或者 get 方法时候才会进行创建
   - 其实每个线程的本地变量不是存放到 ThreadLocal 实例里面的，而是存放到调用线程的 threadLocals 变量里面。也就是说 ThreadLocal 类型的本地变量是存放到具体的线程内存空间的
   - ThreadLocal 就是一个工具壳，它通过 set 方法把 value 值放入调用线程的 threadLocals 里面存放起来，当调用线程调用它的 get 方法时候再从当前线程的 threadLocals变 量里面拿出来使用
   - 如果调用线程一直不终止，那么这个本地变量会一直存放到调用线程的 threadLocals 变量里面，所以当不需要使用本地变量时候可以通过调用 ThreadLocal 变量的 remove 方法，从当前线程的 threadLocals 里面删除该本地变量。另外 Thread 里面的 threadLocals 为何设计为 map 结构呢？很明显是因为每个线程里面可以关联多个 ThreadLocal 变量
   - 下面简单分析下 ThreadLocal 的 set，get，remove 方法的实现逻辑：
   ```
      * void set(T value)
          public void set(T value) {
              //(1)获取当前线程
              Thread t = Thread.currentThread();
              //(2)当前线程作为key，去查找对应的线程变量，找到则设置
              ThreadLocalMap map = getMap(t);
              if (map != null)
                  map.set(this, value);
              else
              //(3)第一次调用则创建当前线程对应的HashMap
                  createMap(t, value);
          }
   ```
   - 如上代码（1）首先获取调用线程，然后使用当前线程作为参数调用了 getMap(t) 方法，getMap(Thread t) 代码如下：
   ```
      ThreadLocalMap getMap(Thread t) {
              return t.threadLocals;
          }
   ```
   - 可知 getMap(t) 所做的就是获取线程自己的变量 threadLocals，threadlocal 变量是绑定到了线程的成员变量里面
   - 如果 getMap(t) 返回不为空，则把 value 值设置进入到 threadLocals，也就是把当前变量值放入了当前线程的内存变量 threadLocals，threadLocals 是个 HashMap 结构，其中 key 就是当前 ThreadLocal 的实例对象引用，value 是通过 set 方法传递的值
   - 如果 getMap(t) 返回空那说明是第一次调用 set 方法，则创建当前线程的 threadLocals 变量
   ```
       void createMap(Thread t, T firstValue) {
              t.threadLocals = new ThreadLocalMap(this, firstValue);
          }
   ```
   - T get()方法
   ```
      public T get() {
              //(4) 获取当前线程
              Thread t = Thread.currentThread();
              //(5)获取当前线程的threadLocals变量
              ThreadLocalMap map = getMap(t);
              //(6)如果threadLocals不为null，则返回对应本地变量值
              if (map != null) {
                  ThreadLocalMap.Entry e = map.getEntry(this);
                  if (e != null) {
                      @SuppressWarnings("unchecked")
                      T result = (T)e.value;
                      return result;
                  }
              }
              //(7)threadLocals为空则初始化当前线程的threadLocals成员变量
                      return setInitialValue();
          }
   ```
   - 首先获取当前线程实例，如果当前线程的 threadLocals 变量不为 null 则直接返回当前线程绑定的本地变量。否者执行代码（7）进行初始化，setInitialValue() 的代码如下：
   ```
      private T setInitialValue() {
              //(8)初始化为null
              T value = initialValue();
              Thread t = Thread.currentThread();
              ThreadLocalMap map = getMap(t);
              //(9)如果当前线程的threadLocals变量不为空
              if (map != null)
                  map.set(this, value);
              else
              //(10)如果当前线程的threadLocals变量为空
                  createMap(t, value);
              return value;
          }
          protected T initialValue() {
              return null;
          }
   ```
   - 如上代码如果当前线程的 threadLocals 变量不为空，则设置当前线程的本地变量值为 null，否者调用 createMap 创建当前线程的 createMap 变量
   
   - T remove()方法
   ```
      public void remove() {
               ThreadLocalMap m = getMap(Thread.currentThread());
               if (m != null)
                   m.remove(this);
           }
   ```
   - 如果当前线程的 threadLocals 变量不为空，则删除当前线程中指定 ThreadLocal 实例的本地变量
   
   - ### 注意
     - 每个线程内部都有一个名字为 threadLocals 的成员变量，该变量类型为 HashMap，其中 key 为我们定义的 ThreadLocal 变量的 this 引用，value 则为我们 set 时候的值
     - 每个线程的本地变量是存到线程自己的内存变量 threadLocals 里面的，如果当前线程一直不消失那么这些本地变量会一直存到，所以可能会造成内存泄露
     - 所以使用完毕后要记得调用 ThreadLocal 的 remove 方法删除对应线程的 threadLocals 中的本地变量
   
