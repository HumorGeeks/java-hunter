#### 大厂面试之Spring

##### 环境搭建过程需要注意

- Gradle 版本（高），JDK版本（高）、配置阿里镜像拉取 jar 包

##### Spring IOC

- xml 配置文件 或者 java配置类；BeanDefinitionReader 处理配置信息，配置并解析（成

  BeanDefinition：关于 Bean 的生命周期、销毁、初始化等操作；XML 中定义的各种属性都会先加载到 BeanDefinition 上，然后通过 BeanDefinition 来生成一个 Bean）注册到容器里边

    - Bean 的 Class 全路径、id、别名信息、properties 属性
        - 正常情况下注册，向容器中保存 BeanDefinition 信息的map中添加一个键值对，key 是 beanName，value是 BeanDefinition 对象，使用一个 map 来保存映射关系，get
          的时候需要重定向一次
    - Bean 是否懒加载
    - Bean 是不是单例的
    - Bean 的作用域
    - Bean 的初始化方法、销毁方法

- 注册和执行 BeanFactory 容器后处理器实例

- BeanFactoryPostProcessor 包含一个接口方法可以将当前容器的引用传递进去，之后就可以手动注册 BeanDefinition，修改删除现有的 BeanDefinition 信息，面向扩展开放，修改关闭

- BeanFactoryPostProcessor 如何提供给框架
    - 实现 BeanFactoryPostProcessor 接口，然后将实现的类，通过 xml 配置 bean 的方式写到配置文件
    - 容器启动的时候 会检查 BeanDefinition 信息，将“容器后处理器"接口（实现）的配置类抽取出来，反射创建实例，执行接口方法
        - Bean 后处理器 和 容器后处理器 区别
            - 对容器本身进行处理，并总是在容器实例化其他任何Bean之前读取配置文件的元数据并可能修改这些数据
            - Bean 后处理器 实例化每个实例的过程中穿插执行，有两个接口方法，分别是调用实例初始化方法的前后；还有一些子接口，提供了在实例化前后增强的方法入口，是Spring容器的核心扩展点技术，Spring 提供了 很多
              “Bean后处理器”接口的实现类
                - 获取 ApplicationContext 实例的话，实现 ApplicationContextAware 接口；某一个后处理器完成，当前 BeanDefinition 定义的 class ，创建完实例
                  调用 初始化方法前后，会执行后处理器的方法
                - 解析 自动装配的注解 @Autowire @Value
                - AOP 面向切面编程

- Spring 创建一个事件传播器对象（可以提供扩展）

    - 事件传播器 会接收相应事件 并传播
    - BeanDefinition 创建的单实例保存到缓存中以后；事件到事件传播器（可以注册许多监听者）然后广播出去；其实就是 监听者模式
    - 监听者 实现 Spring 的监听者接口，并在xml中 配置这个bean，启动过程中，会根据类型获取 BeanDefinition 信息，并且实例化后注册到事件传播器内

- 预先实例化 单实例

    - 配置加载完后，基本都转化为基本的 BeanDefinition 对象，存储在容器的某个map中，默认都是单例（还可以制定其他）的

    - 预先实例化，容器启动完毕以后，get的bean实例都是从缓存中拉取过来的，例外情况就是手动配置为懒加载，才会进行懒加载；比如某些十分耗费资源；并且非常耗费资源，这时候会配置成懒加载

        - 单实例化 核心流程

            - BeanDefinition 转换成 mbd，mbd 信息, 具体：先去创建一个新的bd对象 mbd ；然后将 parent bd的信息 拷贝到 mbd中，让后再将当前的 bd 覆盖到 mdb 里(
              类似于类里边的继承)
            - 看 当前 bd 有没有配置 depends-on 依赖；如果有，先实例化 depends-on 指定的 bean
            - 根据 bean 定义中指定的 class 信息 还有构造方法信息，寻找合适的构造方法（有配置就用配置、没有就用无参的构造方法）；通过反射调用完成实例化对象的流程
            - 依赖注入（把当前对象所依赖的数据全部写入了）、初始化方法等一些逻辑的处理（bean 后处理器去执行）

        - 什么是 循环依赖

            - 假设有 A、B两个类 ，A的构造方法有一个参数是 B类型，B的构造方法有一个参数是 A类型，实例化A的时候需要B对象，实例化B的时候需要A对象，这样都没办法完成实例化
            - 假设有 A、B、C三个或者更多个类 ；A 构造方法 依赖B、B构造方法 依赖C、C构造方法 依赖A；最终形成了一个闭环

        - Spring 循环依赖有几种情况

            - 构造器参数循环依赖、只能抛出异常
            - 单例构造方法循环依赖、只能抛出异常
            - 单例setter循环依赖、可以解决

        - Spring 如何判定循环依赖

            - 构建对象
                - 根绝 bean 定义中指定的 class 信息 还有构造方法信息，寻找合适的构造方法（有配置就用配置、没有就用无参的构造方法）；通过反射调用完成实例化对象的流程
                - 依赖注入（把当前对象所依赖的数据全部写入了）、初始化方法等一些逻辑的处理（bean 后处理器去执行）
                    - 构造方法注入
                    - setter 方法注入
                - 直接从 spring容器获取 ，容器中没有就根据 依赖的bd定义去实例化依赖的对象，依赖的 beanName 没有对应的 bd 定义，直接抛错
            - 判断原型循环依赖
                - A、B闭环，容器上 提供一个 threadLocal 类型的 set ，记录当前线程正在创建的 beanName，每个对象在开始执行创建逻辑的时候，把beanName 放入到threadLocal 类型的
                  set中；A对象已经执行到依赖注入这个环节，threadLocal 类型的 set 已经包含A的字符串了，然后 A
                  依赖于B，检查是否有B的实例可以用，假设没有，就会根绝B的BeanDefinition定义创建B的实例，发现B 依赖于A，还是回去检查是否有A可用，然后会调用getBean（A） 获取A对象的时候，发现是
                  原型类型 ，所以每次都需要创建新的；又开始创建A的实例流程，检查threadLocal里边是否有 A 的 beanName ，这时候就说明当前线程陷入原型产生的循环依赖；直接抛出异常
            - 判断单实例构造方法产生的循环依赖
                - A 构造方法 依赖于 B，B 构造方法 依赖于 A，Spring 先去创建
                  A对象，先将A的beanName保存到这个set，执行创建A的过程，获取构造方法，然后反射调用构造方法，优先处理构造方法的依赖参数，发现有个参数是B类型的，执行getBean（B）的逻辑，缓存中尚且没有实例化B，去创建B，跟上边流程差不多，触发getBean（A），缓存中没有A实例，触发创建A的流程，beanName
                  添加到set中，因为已经存在所以添加失败，抛出异常

            - 如何解决 setter 方法 造成的循环依赖（三级缓存）

                - 有 A、B两个类，A中有B，B中有A，都是通过 setter 方法进行依赖注入的；假设先去实例化A的实例，根据A的 BeanDefinition定义，拿到A class 的无参构造方法，反射创建出来
                  A实例对象（尚未进行依赖注入和init-method方法调用等等逻辑处理的早期实例），进行后续加工处理之前，包装到ObjectFactory对象内，存入到三级缓存中，key 是
                  beanName，value是这个ObjectFactory对象，暴露一个早期的对象工厂，外部可以通过三级缓存拿到对象工厂ObjectFactory，通过get方法拿到早期创建的这个对象；继续处理，进行依赖注入；A通过setter的方式依赖B对象，通过getBean（B）的方式获取A的依赖对象B，但是B也未实例化，创建B的流程，反射创建出来B的早期对象，处理B的依赖；调用getBean（A），获得B依赖对象A对象的逻辑；A已经包装成了ObjectFactory对象，放到缓存中了，这时候可以拿到A的对象工厂ObjectFactory，get方法拿到早起实例A对象，然后完成B依赖A的处理；接下来处理B对象；调用B对象配置的init-method和执行一些“bean
                  后处理器逻辑”；B实例化完毕就放到一级缓存中：key 是 beanName，value是这个bean对象；A对象相应的也会被创建，然后放到一级缓存里边，把2、3级的缓存都清理掉

                - AService实例化后，在注入属性前提前曝光，将其加入三级缓存singletonFactories中，供其他bean使用；

                - AService通过populateBean注入BService，从缓存中获取BService，发现缓存中没有，开始创建BService实例；

                - BService实例也会在属性注入前提前曝光，加入三级缓存中，此时三级缓存中有AService和BService；

                - BService在进行属性注入时，发现有AService引用，此时，创建AService时，会先从缓存中获取AService(
                  先从一级缓存中取，没有取到后，从二级缓存中取，也没有取到，这时，从三级缓存中取出)
                  ，这时会清除三级缓存中的AService，将其将其加入二级缓存earlySingletonObjects中，并返回给BService供其使用；

                - BService在完成属性注入，进行初始化，这时会加入一级缓存，这时会清除三级缓存中的BService，此时，三级缓存为空，二级缓存中有AService，一级缓存中有BService；

                -
                BService初始化后注入AService中，AService进行初始化，然后通过getSingleton方法获取二级缓存，赋值给exposedObject，最后将其加入一级缓存，清除二级缓存AService；

        - Spring 三级缓存放什么（关注#如何解决 setter 方法 造成的循环依赖）

            - 包装的ObjectFactory（三级缓存）
            - 原始的对象（尚未填充属性）
            - 完全初始化好的对象

        - 三级缓存如何升级到二级缓存

          getBean，如果获取的这个Bean实例数据存放在三级缓存中，那么会拿到ObjectFactory，并调用get（）；返回实例之前（缓存升级）放入到二级缓存，清除三级缓存

        - 为什么要有三级缓存

          反射创建出来对象以后，在执行依赖注入和init-method之前酒吧早期实例包装成ObjectFactory，放入到三级缓存中，调用init-method前后，会有“Bean后处理器”的调用点；spring实现AOP功能“Bean后处理器”，就是在这一步被它调用的；spring
          AOP
          通过动态代理实现，不管是jdk动态代理还是cglib动态代理；会产生一个新的class类型，新的class类型再实例化出来不一个实例，它包装着原生对象，完成增强；代理对象是一个全新的对象；不再和原生对象是一个地址空间了；所以必须有第三级缓存，第三季缓存中这个对象工厂的get方法做了一些处理；返回之前会把AOP的后处理器实现代理增强的逻辑预先执行；会返回增强以后的实例，否则会返回原生实例

##### Spring AOP

- 为什么要引入 AOP 编程

  OOP语言来说，当需要为部分对象引入公共部分的时候，OOP就会引入大量的重复代码；Aop便是将这些横切代码封装到一个可重用模块中，继而降低模块间的耦合度

- Aop是什么东西

    - PointCut：正常表达式来定义那个范围内的类、那些接口会被当成切点
    - Advice：
        - Before 在方法被调用之前调用
        - After 在方法完成之后调用
        - After-returning 在方法成功执行之后调用
        - After-throwing 在方法抛出异常之后调用
        - Around 在被通知的方法调用之前和调用之后调用

    - JoinPoint：具体业务的连接点





