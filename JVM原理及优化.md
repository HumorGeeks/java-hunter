#### 内存模型以及分区

- 堆

  初始化的对象，成员变量 （那种非 static 的变量），所有的对象实例和数组都要 在堆上分配

- 栈

  存储局部变量表(指针)，操作数栈，方法出口等信息

- 方法区

  类信息，常量池（static 常量和 static 变量），编译后的代码（字节码）等数据

- 本地方法栈

  主要为 Native 方法服务

- 程序计数器

  记录当前线程执行的行号

#### 堆内存

- **JAVA堆内存是如何划分**

  划分为堆内存和非堆内存，堆内存分为年轻代（Young Generation）、老年代（Old Generation），非堆内存就一个永久代（Permanent
  Generation）、JDK1.8版本废弃了永久代、替代的是元空间（放在本地内存中）、避免永久代出现的 OOM 问题

  年轻代又分为 Eden 和 Survivor 区。Survivor 区由 FromSpace 和 ToSpace 组成。Eden区占大容量，Survivor两个区占小容量，默认比例是8:1:1

- **分代概念**

  新生成的对象首先放到年轻代Eden区，当Eden空间满了，触发Minor GC、存活下来的对象移动到Survivor0（from）区、Survivor0区满后触发执行Minor
  GC、Survivor0区存活对象移动到Suvivor1区、保证了一段时间内总有一个survivor区为空、多次Minor GC仍然存活的对象移动到老年代、老年代存储长期存活的对象，占满时会触发**Major GC=Full GC**
  ，GC期间会停止所有线程等待GC完成

- **为什么survivor分为两块相等大小的幸存空间**

  为了解决碎片化。如果内存碎片化严重，也就是两个对象占用不连续的内存，已有的连续内存不够新对象存放，就会触发GC

- **JVM堆内存常用参数**

  |          参数          |                           描述                            |
    | :--------------------: | :-------------------------------------------------------: |
  |          -Xms          |                 堆内存初始大小，单位m、g                  |
  |  -Xmx（MaxHeapSize）   |       堆内存最大允许大小，一般不要大于物理内存的80%       |
  |      -XX:PermSize      | 非堆内存初始大小，一般应用设置初始化200m，最大1024m就够了 |
  |    -XX:MaxPermSize     |                   非堆内存最大允许大小                    |
  |  -XX:NewSize（-Xns）   |                    年轻代内存初始大小                     |
  | -XX:MaxNewSize（-Xmn） |            年轻代内存最大允许大小，也可以缩写             |
  |  -XX:SurvivorRatio=8   |  年轻代中Eden区与Survivor区的容量比例值，默认为8，即8:1   |
  |          -Xss          |                       堆栈内存大小                        |

#### 垃圾回收算法

- **标记-清除**

    - 标记阶段：从根集合出发，将所有活动对象及其子对象打上标记

    - 清除阶段：遍历堆，将非活动对象（未打上标记）的连接到空闲链表上

    - 优点：

      实现简单

    - 缺点：

      碎片化；分配速度不理想，每次分配都需要遍历空闲列表找到足够大的分块

- **标记-压缩**

    - 在标记阶段后它将所有活动对象紧密的排在堆的一侧（压缩），消除了内存碎片

    - 优点：

      有效利用了堆，不会出现内存碎片 也不会像复制算法那样只能利用堆的一部分

    - 缺点：

      压缩过程的开销，需要多次搜索堆

- **引用计数**

    - 记录每个对象被引用的次数、新建对象、赋值引用和删除引用的同时更新计数器，如果计数器值为0则直接回收内存

    - 优点：

      可即刻回收垃圾、最大暂停时间短、不需要开始查找

    - 缺点：

      计数器的增减处理繁重、计数器需要占用很多位、循环引用无法回收

- **复制算法**

    - 为两个大小相同的空间 From 和 To， 利用 From 空间进行分配，当 From 空间满的时候，GC将其中的活动对象复制到 To 空间，之后将两个空间互换即完成GC

    - 优点：

      高速分配； 因为分块是连续的、不会发生碎片化

    - 缺点：

      堆使用率低、与保守式GC不兼容

- **分代收集算法**

#### 垃圾收集器

- 串行收集器

  单线程、收集时，必须暂停应用的工作线程，直到收集结束

- 并行收集器

  多条垃圾收集线程并行工作，在多核CPU下效率更高，应用线程仍然处于等待状态

- CMS收集器

  缩短暂停应用时间为目标而设计的，是基于标记-清除算法实现，整个过程分为4个步骤

    - 初始标记
    - 并发标记
    - 重新标记
    - 并发清除

  初始标记、重新标记这两个步骤仍然需要暂停应用线程、初始标记只是标记一下GC
  Roots能直接关联到的对象，速度很快，并发标记阶段是标记可回收对象、重新标记阶段则是为了修正并发标记期间因用户程序继续运作导致标记产生变动的那一部分对象的标记记录，这个阶段暂停时间比初始标记阶段稍长一点，但远比并发标记时间短、整个过程中消耗最长的并发标记和并发清除过程收集器线程都可以与用户线程一起工作，所以，CMS收集器内存回收与用户一起并发执行的，大大减少了暂停时间、G1跟踪各个Region里的垃圾堆积价值大小（所获得空间大小以及回收所需时间），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region，从而保证了再有限时间内获得更高的收集效率

    - 初始标记
    - 并发标记
    - 最终标记
    - 筛选回收

- G1收集器

  将堆内存划分多个大小相等的独立区域、预测暂停时间、避免对整个堆进行全区收集

- 垃圾收集器参数

  | -XX:+UseSerialGC                       | 串行收集器                                                   |
    | -------------------------------------- | ------------------------------------------------------------ |
  | -XX:+UseParallelGC                     | 并行收集器                                                   |
  | -XX:+UseParallelGCThreads=8            | 并行收集器线程数，同时有多少个线程进行垃圾回收，一般与CPU数量相等 |
  | -XX:+UseParallelOldGC                  | 指定老年代为并行收集                                         |
  | -XX:+UseConcMarkSweepGC                | CMS收集器（并发收集器）                                      |
  | -XX:+UseCMSCompactAtFullCollection     | 开启内存空间压缩和整理，防止过多内存碎片                     |
  | -XX:CMSFullGCsBeforeCompaction=0       | 表示多少次Full GC后开始压缩和整理，0表示每次Full GC后立即执行压缩和整理 |
  | -XX:CMSInitiatingOccupancyFraction=80% | 表示老年代内存空间使用80%时开始执行CMS收集，防止过多的Full GC |
  | -XX:+UseG1GC                           | G1收集器                                                     |
  | -XX:MaxTenuringThreshold=0             | 在年轻代经过几次GC后还存活，就进入老年代，0表示直接进入老年代 |

- 为什么会堆内存溢出

    - 老年代内存不足：java.lang.OutOfMemoryError:Javaheapspace
    - 永久代内存不足：java.lang.OutOfMemoryError:PermGenspace
    - 代码bug，占用内存无法及时回收

  可以通过添加个参数-XX:+HeapDumpOnOutMemoryError，让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后分析

    - 设置堆内存最小和最大值，最大值参考历史利用率设置
    - 设置GC垃圾收集器为G1
    - 启用GC日志，方便后期分析

#### java 类加载过程

- 加载
    - 通过一个类的全限定名来获取定义此类的二进制流
    - 将字节流所代表的静态存储结构转化为方法区的运行时数据结构
    - 内存中生成一个.class对象，作为方法区这个类的各种数据访问接口

- 验证

    - 文件格式验证
        - 是否以魔数0xCAFEBABE开头
        - 主、次版本号是否在当前Java虚拟机接受范围之内
        - 常量池是否有不支持的常亮、是否有不存在的常量
        - UTF8编码
        - Class文件中信息
    - 元数据验证
        - 是否继承了不允许继承的类
        - 非抽象类是否实现了其他类或者接口之中要实现的所有方法
        - 字段方法是否与父类产生矛盾
    - 字节码验证
        - 通过数据流分析和控制流分析，确定程序语义是合法的、符合逻辑
    - 符号引用验证
        - 该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源

- 准备
    - 分配内存空间
    - 类方法解析
    - 接口方法解析

- 初始化

    - 为静态变量赋值
    - 执行static代码块

- new一个对象之后 JVM都做了什么
    - 类加载过程（同上）
    - 创建对象
        - 在堆区分配对象需要的内存
        - 对所有实例变量赋默认值
        - 执行实例初始化代码
        - 如果有类似于Child c = new Child()形式的c引用的话，在栈区定义Child类型引用变量c，然后将堆区对象的地址赋值给它

- 类加载器

  加载过程中会先检查类是否被已加载，检查顺序是自底向上，从Custom ClassLoader到BootStrap
  ClassLoader逐层检查，只要某个classloader已加载就视为已加载此类，保证此类只所有ClassLoader加载一次。而加载的顺序是自顶向下，也就是由上层来逐层尝试加载此类

    - Bootstrap ClassLoader

      加载$JAVA_HOME中jre/lib/rt.jar里所有的class

    - Extension ClassLoader

      加载java平台中扩展功能的一些jar包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包

    - App ClassLoader

      加载classpath中指定的jar包及目录中class

    - Custom ClassLoader

      应用程序根据自身需要自定义的ClassLoader

- 双亲委托模型
    - 当前ClassLoader首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类
    - 当前classLoader的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到bootstrp ClassLoader
    - 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回

- 为何采用双亲委派机制
    - 防止类的重复加载
    - 保护程序安全、防止核心API被篡改
- 为什么要破坏双亲委托
    - 某些情况下父类加载器需要委托子类加载器去加载class文件、受到加载范围的限制、父类加载器无法加载到需要的文件
    - 由于Driver接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如mysql的就写了`MySQL Connector`
      ，那么问题就来了，DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况

#### OutOfMemoryError线上排查及其解决（文件导出）

- **top** 命令，它能够实时显示系统中各个进程的资源占用状况，经常用来监控linux的系统状况，比如cpu、内存的使用。

- **dmesg**。dmesg可以用来查看开机之后的系统日志，其中可以捕捉到一些系统资源与进程的变化信息。dmesg |grep -E ‘kill|oom|out of memory’ 来搜索内存溢出的信息挺实用

- **查看JVM状态**

    - **ps -aux|grep java** 当服务重新部署后，可以找出当前Java进程的PID
    - **jstat -gcutil pid interval** 用于查看当前GC的状态,它对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控
    - **jmap -histo:live pid** 可用统计存活对象的分布情况，从高到低查看占据内存最多的对象

- **Java dump分析问题**

    - **jmap -dump:format=b,file=文件名 [pid]** 利用Jmap dump

    - 用gcore [pid]直接保留内存信息

        - 先生成core dump

        - 把core文件转换成hprof

        - 使用性能分析工具对hprof进行分析

            - 一个是 *MAT*，一个基于Eclipse的内存分析工具，它可以快速的计算出在内存中对象的占用大小，看看是谁阻止了垃圾收集器的回收工作，并可以通过报表直观的查看到可能造成这种结果的对象。第二个是*
              zprofile*(阿里内部工具)

              ![这里写图片描述](https://img-blog.csdn.net/20161124235638127)

              ![这里写图片描述](https://img-blog.csdn.net/20161124235705765)

            - 于是我们推测可能是ali-tomcat的StandardClassLoader的类加载时出了问题，导致引入的byte[]数组占用的堆大小过多,而Full GC回收不过来，导致了OOM

            - 通过tomcat查明真相

需要把它`dump`到本地，然后通过一些工具进行分析，才能大概的发现问题出在哪里

- 使用`java`自带的`jmap`命令，生成 `hprof`文件
- 使用mat工具分析 hprof文件
    - 当前对内存中有两个类型的对象几乎占满了整个堆内存，一个是`byte[]`数组，一个是`Http11OutputBuffer`对象。查看`details`,发现这两个问题都来自于`tomcat`
    - 进一步打开`Histogram`，找到占用内存大的对象
- 修改 `server.xml`的配置（maxHttpHeaderSize）、通过`-Xms`和`-Xmx`直接调大了堆内存的大小

#### JVM 内存泄漏及其解决

程序在向系统申请分配内存空间后(new)，在使用完毕后未释放。结果导致一直占据该内存单元，我们和程序都无法再使用该内存单元，直到程序结束

- 如果长生命周期的对象持有短生命周期的引用，就很可能会出现内存泄露

  ~~~java
  public class Simple {
   
      Object object;
   
      public void method1(){
          object = new Object();
      //...其他代码
      }
  }
  ~~~

可以改为这样：

~~~java
public class Simple {
 
    Object object;
 
    public void method1(){
        object = new Object();
        //...其他代码
        object = null;
    }
}
~~~

在内存对象明明已经不需要的时候，还仍然保留着这块内存和它的访问方式（引用），这是所有语言都有可能会出现的内存泄漏方式

**备注：尽量减小对象的作用域**

**容易发生内存泄露的例子和解决方法**

- **null值的手动设置**

  ~~~java
  public E pop(){
      if(size == 0)
          return null;
      else{
          E e = (E) elementData[--size];
          elementData[size] = null;
          return e;
      }
  }
  ~~~

- **容器使用时的内存泄露**

  ~~~java
  void method(){
      Vector vector = new Vector();
      for (int i = 1; i<100; i++)
      {
          Object object = new Object();
          vector.add(object);
          object = null;
      }
      //...对vector的操作
      //...与vector无关的其他操作
  }
  ~~~

  对vector操作完成之后，执行下面与vector无关的代码时，如果发生了GC操作，这一系列的object是没法被回收的，而此处的内存泄露可能是短暂的，因为在整个method()方法执行完成后，那些对象还是可以被回收

  **容器时方法内的局部变量，造成的内存泄漏影响可能不算很大（但我们也应该避免），但是，如果这个容器作为一个类的成员变量，甚至是一个静态（static）的成员变量时，就要更加注意内存泄露了**

  ~~~java
  public class CollectionMemory {
      public static void main(String s[]){
          Set<MyObject> objects = new LinkedHashSet<MyObject>();
          objects.add(new MyObject());
          objects.add(new MyObject());
          objects.add(new MyObject());
          System.out.println(objects.size());
          while(true){
              objects.add(new MyObject());
          }
      }
  }
   
  class MyObject{
      //设置默认数组长度为99999更快的发生OutOfMemoryError
      List<String> list = new ArrayList<>(99999);
  }
  ~~~

- **单例模式导致的内存泄露**

  很多时候我们可以把它的生命周期与整个程序的生命周期看做差不多的，所以是一个长生命周期的对象。如果这个对象持有其他对象的引用，也很容易发生内存泄露

- **内部类和外部模块的引用**

- **Set、Map 容器使用默认 equals 方法造成的内存泄露**

  重写 equals 方法只对值进行比较

**备注：非java代码中可能会用到相应的申请内存的操作（如c的malloc()函数），而在这些非java代码中并没有有效的释放这些内存，就可以使用finalize()方法，并在里面调用本地方法的free()等函数、finalize()
并不适合用作普通的清理工作**