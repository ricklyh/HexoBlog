---
title: jvm调优工具说明
date: 2018-07-12 12:12:04
tags: [jvm]
categories: jvm
---

## JVM调优工具 
   - JVM配置以及调优是Java程序员进阶必须掌握的，一个优秀的Java程序员可以根据运行环境设置JVM参数，从而达到最优配置，合理充分的利用系统资源，避免生产环境发生一些如OOM的异常或者线程死锁、Java进程CPU消耗过高等问题
  
   - 注意！！！：使用的jdk版本是jdk8,查看本机的初始化参数：java -XX:+PrintFlagsInitial
   
## jps
   - JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程
   
   - 命令格式
     - jps [options] [hostid]
     -  option参数
     -  -l : 输出主类全名或jar路径
     -  -q : 只输出LVMID
     -  -m : 输出JVM启动时传递给main()的参数
     -  -v : 输出JVM启动时显示指定的JVM参数
     -  其中[option]、[hostid]参数也可以不写
     
   - 示例  jps -ml 或 jps -l -m
   ![jvmjps](jps1.png "Optional title")  
   
## jstat
   - jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据    
   
   - 命令格式
     - jstat [option] LVMID [interval] [count]
     - 参数
     - [option] : 操作参数
     - LVMID : 本地虚拟机进程ID
     - [interval] : 连续输出的时间间隔
     - [count] : 连续输出的次数
     
### 类加载统计
   - ![jstat](jstat0.png "Optional title") 
     - 显示字段含义
       - Loaded:加载class的数量
       - Bytes：所占用空间大小
       - Unloaded：未加载数量
       - Bytes:未加载占用空间
       - Time：时间
       
### 编译统计
   - ![jstat](jstat1.png "Optional title") 
     - 显示字段含义
       - Compiled：编译数量。
       - Failed：失败数量
       - Invalid：不可用数量
       - Time：时间
       - FailedType：失败类型
       - FailedMethod：失败的方法
       
### 垃圾回收统计
   - ![jstat](jstat2.png "Optional title") 
     - 显示字段含义
       - S0C：第一个幸存区的大小
       - S1C：第二个幸存区的大小
       - S0U：第一个幸存区的使用大小
       - S1U：第二个幸存区的使用大小
       - EC：伊甸园区的大小
       - EU：伊甸园区的使用大小
       - OC：老年代大小
       - OU：老年代使用大小
       - MC：方法区大小
       - MU：方法区使用大小
       - CCSC:压缩类空间大小
       - CCSU:压缩类空间使用大小
       - YGC：年轻代垃圾回收次数
       - YGCT：年轻代垃圾回收消耗时间
       - FGC：老年代垃圾回收次数
       - FGCT：老年代垃圾回收消耗时间
       - GCT：垃圾回收消耗总时间
          
### 堆内存统计
   - ![jstat](jstat3.png "Optional title") 
     - 显示字段含义
       - NGCMN：新生代最小容量
       - NGCMX：新生代最大容量
       - NGC：当前新生代容量
       - S0C：第一个幸存区大小
       - S1C：第二个幸存区的大小
       - EC：伊甸园区的大小
       - OGCMN：老年代最小容量
       - OGCMX：老年代最大容量
       - OGC：当前老年代大小
       - OC:当前老年代大小
       - MCMN:最小元数据容量
       - MCMX：最大元数据容量
       - MC：当前元数据空间大小
       - CCSMN：最小压缩类空间大小
       - CCSMX：最大压缩类空间大小
       - CCSC：当前压缩类空间大小
       - YGC：年轻代gc次数
       - FGC：老年代GC次数
          
### 新生代垃圾回收统计
   - ![jstat](jstat4.png "Optional title") 
     - 显示字段含义
       - S0C：第一个幸存区大小
       - S1C：第二个幸存区的大小
       - S0U：第一个幸存区的使用大小
       - S1U：第二个幸存区的使用大小
       - TT:对象在新生代存活的次数
       - MTT:对象在新生代存活的最大次数
       - DSS:期望的幸存区大小
       - EC：伊甸园区的大小
       - EU：伊甸园区的使用大小
       - YGC：年轻代垃圾回收次数
       - GCT：年轻代垃圾回收消耗时间
       
### 新生代内存统计
   - ![jstat](jstat5.png "Optional title") 
     - 显示字段含义
       - NGCMN：新生代最小容量
       - NGCMX：新生代最大容量
       - NGC：当前新生代容量
       - S0CMX：最大幸存1区大小
       - S0C：当前幸存1区大小
       - S1CMX：最大幸存2区大小
       - S1C：当前幸存2区大小
       - ECMX：最大伊甸园区大小
       - EC：当前伊甸园区大小
       - YGC：年轻代垃圾回收次数
       - FGC：老年代回收次数
       
### 老年代垃圾统计
   - ![jstat](jstat6.png "Optional title") 
     - 显示字段含义
       - MC：方法区大小
       - MU：方法区使用大小
       - CCSC:压缩类空间大小
       - CCSU:压缩类空间使用大小
       - OC：老年代大小
       - OU：老年代使用大小
       - YGC：年轻代垃圾回收次数
       - FGC：老年代垃圾回收次数
       - FGCT：老年代垃圾回收消耗时间
       - GCT：垃圾回收消耗总时间
       
### 老年代内存统计
   - ![jstat](jstat7.png "Optional title") 
     - 显示字段含义
       - OGCMN：老年代最小容量
       - OGCMX：老年代最大容量
       - OGC：当前老年代大小
       - OC：老年代大小
       - YGC：年轻代垃圾回收次数
       - FGC：老年代垃圾回收次数
       - FGCT：老年代垃圾回收消耗时间
       - GCT：垃圾回收消耗总时间
       
### 元数据空间统计
   - jstat -gcmetacapacity [LVMID]
   - ![jstat](jstat8.png "Optional title") 
     - 显示字段含义
       - MCMN: 最小元数据容量
       - MCMX：最大元数据容量
       - MC：当前元数据空间大小
       - CCSMN：最小压缩类空间大小
       - CCSMX：最大压缩类空间大小
       - CCSC：当前压缩类空间大小
       - YGC：年轻代垃圾回收次数
       - FGC：老年代垃圾回收次数
       - FGCT：老年代垃圾回收消耗时间
       - GCT：垃圾回收消耗总时间
       
### 总结垃圾回收统计
   - jstat -gcutil 17063 1s 10
   - jstat -gcutil 17063 1s #一直连续输出
   - ![jstat](jstat9.png "Optional title") 
     - 显示字段含义
       - S0：幸存1区当前使用比例
       - S1：幸存2区当前使用比例
       - E：伊甸园区使用比例
       - O：老年代使用比例
       - M：元数据区使用比例
       - CCS：压缩使用比例
       - YGC：年轻代垃圾回收次数
       - FGC：老年代垃圾回收次数
       - FGCT：老年代垃圾回收消耗时间
       - GCT：垃圾回收消耗总时间
       
### JVM编译方法统计
   - ![jstat](jstat10.png "Optional title") 
     - 显示字段含义
       - Compiled：最近编译方法的数量
       - Size：最近编译方法的字节码数量
       - Type：最近编译方法的编译类型
       - Method：方法名标识
       
## jmap
   - jmap(JVM Memory Map)命令用于生成heap dump文件
   - 如果不使用这个命令，还可以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件
   - jmap不仅能生成dump文件，还可以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等
   
   - 命令格式
     - jmap [option] LVMID
     - option参数
     - dump : 生成堆转储快照(会引发full GC)
     - finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
     - heap : 显示Java堆详细信息
     - histo : 显示堆中对象的统计信息(会引发full GC)
     - permstat : to print permanent generation statistics
     - F : 当-dump没有响应时，强制生成dump快照
     
   - 导出整个JVM 中内存信息
     - jmap -dump:format=b,file=文件名 [pid] ;比如：jmap -dump:format=b,file=/Users/lyh/Desktop/dumplock.hprof 20705
     - 示例：jmap -histo pid 展示class的内存情况:
     ![jmap](jmap0.png "Optional title")
     - 显示字段含义
       - instance：对象实例个数
       - bytes：总占用的字节数
       - class name:对应的就是 Class 文件里的 class 的标识
       - B 代表 byte
       - C 代表 char
       - D 代表 double
       - F 代表 float
       - I 代表 int
       - J 代表 long
       - Z 代表 boolean
       - 前边有 [ 代表数组， [I 就相当于 int[]
       - 对象用 [L+ 类名表示
       
## jhat
   - jhat(JVM Heap Analysis Tool)命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。
   - 在此要注意，一般不会直接在服务器上进行分析，因为jhat是一个耗时并且耗费硬件资源的过程，一般把服务器生成的dump文件复制到本地或其他机器上进行分析
   
   - 命令格式
     - jhat [dump file]
     
## jstack
   - jstack用于生成java虚拟机当前时刻的线程快照 
   - 命令格式
     - jstack [ option ] pid
     - jstack [ option ] executable core
     - jstack [ option ] [server-id@]remote-hostname-or-IP
     - option参数：
     - -F : 当正常输出请求不被响应时，强制输出线程堆栈
     - -l : 除堆栈外，显示关于锁的附加信息
     - -m : 如果调用到本地方法的话，可以显示C/C++的堆栈
        
   - 示例 输出文件：jstack -l 17063 >1.txt
   
   - 线程dump的分析工具
     - IBM Thread and Monitor Dump Analyze for Java 一个小巧的Jar包，能方便的按状态，线程名称，线程停留的函数排序，快速浏览。
     - http://spotify.github.io/threaddump-analyzer Spotify提供的Web版在线分析工具，可以将锁或条件相关联的线程聚合到一起
     
## jinfo
   - jinfo(JVM Configuration info)这个命令作用是实时查看和调整虚拟机运行参数。
   - 之前的jps -v口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用jinfo口令
   - 命令格式
     - jinfo [option] [args] LVMID
     - option参数：
     - -flag : 输出指定args参数的值
     - -flags : 不需要args参数，输出所有JVM参数的值
     - -sysprops : 输出系统属性，等同于System.getProperties()
     