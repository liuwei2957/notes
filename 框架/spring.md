### Spring框架的七大模块

1.  Spring Core：框架的最基础部分，提供 IoC 容器，对 bean 进行管理 
2.  Spring Context：继承BeanFactory，提供上下文信息，扩展出JNDI、EJB、电子邮件、国际化等功能 
3.  Spring DAO：提供了JDBC的抽象层，还提供了声明性事务管理方法 
4.  Spring ORM：提供了JPA、JDO、Hibernate、MyBatis 等ORM映射层 
5.  Spring AOP：集成了所有AOP功能 
6.  Spring Web：提供了基础的 Web 开发的上下文信息，现有的Web框架，如JSF、Tapestry、Structs等，提供了集成 
7.  Spring Web MVC：提供了 Web 应用的 Model-View-Controller 全功能实现 

### spring ioc初始化流程

 IOC：控制反转：将对象的创建权，由Spring管理. DI（依赖注入）：在Spring创建对象的过程中，把对象依赖的属性注入到类中 

1. resource定位： 即寻找用户定义的bean资源，由 ResourceLoader通过统一的接口Resource接口来完成 
2. beanDefinition载入： BeanDefinitionReader读取、解析Resource定位的资源 成BeanDefinition 载入到ioc中（通过HashMap进行维护BD） 
3. BeanDefinition注册： 即向IOC容器注册这些BeanDefinition， 通过BeanDefinitionRegistery实现 

### BeanDefinition加载流程

1. 定义BeanDefinitionReader
2. 解析xml的document：BeanDefinitionDocumentReader解析document成beanDefinition 

### 依赖注入怎么处理bean之间的依赖关系

 其实就是通过在beanDefinition载入时，如果bean有依赖关系，通过占位符来代替，在调用getbean时候，如果遇到占位符，从ioc里获取bean注入到本实例来 

### Bean的生命周期

1. 实例化Bean：Ioc容器通过获取BeanDefinition对象中的信息进行实例化，实例化对象被包装在BeanWrapper对象中
2.  设置对象属性（DI）：通过BeanWrapper提供的设置属性的接口完成属性依赖注入 
3. 注入Aware接口（BeanFactoryAware， 可以用这个方式来获取其它 Bean，ApplicationContextAware）：Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean
4. BeanPostProcessor：自定义的处理（分前置处理和后置处理）
5. InitializingBean和init-method：执行我们自己定义的初始化方法
6. 使用
7. destroy：bean的销毁

### Spring的IOC注入方式

 构造器注入 setter方法注入 注解注入 接口注入 

### 怎么检测是否存在循环依赖

 Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明了循环依赖了 

### Spring如解决Bean循环依赖问题

#### 属性的循环依赖

1.  singletonObjects：第一级缓存，里面放置的是实例化好的单例对象 
2.  earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象 
3.  singletonFactories：第三级缓存，里面存放的是要被实例化的对象的对象工厂 
4.  创建bean的时候Spring首先从一级缓存singletonObjects中获取。如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取，如果还是获取不到就从三级缓存singletonFactories中取（Bean调用构造函数进行实例化后，即使属性还未填充，就可以通过三级缓存向外提前暴露依赖的引用值（提前曝光），根据对象引用能定位到堆中的对象，其原理是基于Java的引用传递），取到后从三级缓存移动到了二级缓存完全初始化之后将自己放入到一级缓存中供其他使用 
5.  因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决 
6. 三级缓存主要是用来解决aop的问题，同时不违背bean的生命周期

#### 构造器的循环依赖

1. 在构造函数中使用@Lazy注解延迟加载：在注入依赖时，先注入代理对象，当首次使用时再创建对象说明 
2.  使用@PostConstruct ： 在要注入的属性（该属性是一个bean）上使用 @Autowired ,并使用@PostConstruct 标注在另一个方法，且该方法里设置对其他的依赖 
3.  实现ApplicationContextAware and InitializingBean接口 ： 如果一个Bean实现了ApplicationContextAware，该Bean可以访问Spring上下文，并可以从那里获取到其他的bean。实现InitializingBean接口，表明这个bean在所有的属性设置完后做一些后置处理操作（调用的顺序为init-method后调用）;在这种情况下，我们需要手动设置依赖 

### Spring 中使用了哪些设计模式

- 工厂模式：spring中的BeanFactory就是简单工厂模式的体现，根据传入唯一的标识来获得bean对象 

-  单例模式：提供了全局的访问点BeanFactory 

- 代理模式：AOP功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）
-  装饰器模式：依赖注入就需要使用BeanWrapper 
-  观察者模式：spring中Observer模式常用的地方是listener的实现。如ApplicationListener 
- 策略模式：Bean的实例化的时候决定采用何种方式初始化bean实例（反射或者CGLIB动态字节码生成）

### AOP

 传统oop开发代码逻辑自上而下的，这个过程中会产生一些横切性问题，这些问题与我们主业务逻辑关系不大，会散落在代码的各个地方，造成难以维护，aop思想就是把业务逻辑与横切的问题进行分离，达到解耦的目的，提高代码重用性和开发效率 

#### 应用场景

记录日志，监控性能，权限控制，事务管理

#### 源码分析

1.  @EnableAspectJAutoProxy给容器（beanFactory）中注册一个AnnotationAwareAspectJAutoProxyCreator对象
2.  AnnotationAwareAspectJAutoProxyCreator对目标对象进行代理对象的创建，对象内部，是封装JDK和CGlib两个技术，实现动态代理对象创建的（创建代理对象过程中，会先创建一个代理工厂，获取到所有的增强器（通知方法），将这些增强器和目标类注入代理工厂，再用代理工厂创建对象）  
3.  代理对象执行目标方法，得到目标方法的拦截器链，利用拦截器的链式机制，依次进入每一个拦截器进行执行 

#### AOP使用哪种动态代理

1.  当bean的是实现中存在接口或者是Proxy的子类，---jdk动态代理；不存在接口，spring会采用CGLIB来生成代理对象 
2.  JDK 动态代理主要涉及到 java.lang.reflect 包中的两个类：Proxy 和 InvocationHandler 
3.  Proxy 利用 InvocationHandler（定义横切逻辑） 接口动态创建 目标类的代理对象 

##### jdk动态代理

1. 通过bind方法建立代理与真实对象关系，通过Proxy.newProxyInstance（target）生成代理对象
2. 代理对象通过反射invoke方法实现调用真实对象的方法

##### 动态代理与静态代理区别

-  静态代理，程序运行前代理类的.class文件就存在了 
- 动态代理：在程序运行时利用反射动态创建代理对象<复用性，易用性，更加集中都调用invoke>

### springMVC流程

 ->DispatcherServlet->HandlerMapping->Handler ->DispatcherServlet->HandlerAdapter处理handler->ModelAndView ->DispatcherServlet->ModelAndView->ViewReslover->View ->DispatcherServlet->返回给客户 

1.  用户请求发送给DispatcherServlet，DispatcherServlet调用HandlerMapping处理器映射器 
2.  HandlerMapping根据xml或注解找到对应的处理器，生成处理器对象返回给DispatcherServlet 
3.  DispatcherServlet会调用相应的HandlerAdapter 
4.  HandlerAdapter经过适配调用具体的处理器去处理请求，生成ModelAndView返回给DispatcherServlet 
5.  DispatcherServlet将ModelAndView传给ViewReslover解析生成View返回给DispatcherServlet 
6.  DispatcherServlet根据View进行渲染视图 

### SpringBoot启动流程

1.  new springApplication对象，利用spi机制加载applicationContextInitializer， applicationLister接口实例（META-INF/spring.factories） 
2. 调run方法准备Environment，加载应用上下文（applicationContext），发布事件 很多通过lister实现
3.  创建spring容器， refreshContext（） ，实现starter自动化配置，spring.factories文件加载， bean实例化 

### Gateway原理分析

1.  请求到达DispatcherHandler， DispatchHandler在IOC容器初始化时会在容器中实例化HandlerMapping接口 
2.  用handlerMapping根据请求URL匹配到对应的Route，然后有对应的filter做对应的请求转发最终response返回去 

### zoo与eur区别

1. zoo采用半数存活原则（避免脑裂），eur采用自我保护机制来解决分区问题
2. eur本质是个工程，zoo只是一个进程 ZooKeeper基于CP，不保证高可用，如果zookeeper正在选主，或者Zookeeper集群中半数以上机器不可用，那么将无法获得数据。Eureka基于AP，能保证高可用，即使所有机器都挂了，也能拿到本地缓存的数据。作为注册中心，其实配置是不经常变动的，只有发版（发布新的版本）和机器出故障时会变。对于不经常变动的配置来说，CP是不合适的，而AP在遇到问题时可以用牺牲一致性来保证可用性，既返回旧数据，缓存数据。所以理论上Eureka是更适合做注册中心。而现实环境中大部分项目可能会使用ZooKeeper，那是因为集群不够大，并且基本不会遇到用做注册中心的机器一半以上都挂了的情况。所以实际上也没什么大问题 

### Hystrix原理

 通过维护一个自己的线程池，当线程池达到阈值的时候，就启动服务降级，返回fallback默认值 

