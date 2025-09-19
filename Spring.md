
# IOC（Inversion of Control，控制反转）

Spring IOC（Inversion of Control，控制反转）是 Spring 框架的核心概念之一。它是通过**依赖注入（Dependency Injection）** 实现的。IOC 让对象的创建与管理职责由容器负责，而不是由对象自身控制。

- **核心思想**：控制反转意味着将对象的创建和依赖关系交由 Spring 容器管理，而不是由程序代码直接控制。这种机制使得程序更加灵活和解耦，提升了代码的可维护性和扩展性。
- **依赖注入**：通过构造器注入、setter 注入或接口注入，将对象所需的依赖传递给它，而不是让对象自行创建依赖。

<mark class="hltr-grey">理解控制和反转</mark>

首先要明确 IOC 是一种思想，而不是一个具体的技术，其次 IOC 这个思想也不是 Spring 创造的。

然后我们要理解到底**控制**的是什么，其实就是**控制对象的创建**，IOC 容器根据配置文件来创建对象，在对象的生命周期内，在**不同时期**根据不同配置进行对象的创建和改造。

那什么被**反转**了？其实就是关于创建对象且注入依赖对象的这个动作，本来这个动作是由我们程序员在代码里面指定的，例如对象 A 依赖对象 B，在创建对象 A 代码里，我们需要写好如何创建对象 B，这样才能构造出一个完整的 A。

而反转之后，这个动作就由 IOC 容器触发，IOC 容器在创建对象 A 的时候，发现依赖对象 B ，根据配置文件，它会创建 B，并将对象 B 注入 A 中。

这里要注意，注入的不一定非得是一个对象，也可以注入配置文件里面的一个值给对象 A 等等。

<mark class="hltr-grey">好处</mark>
- 对象的创建都由 IOC 容器来控制之后，对象之间就不会有很明确的依赖关系，使得非常容易设计出松耦合的程序。
    
- 因为创建对象由 IOC 全权把控，那么我们就能很方便的让 IOC 基于扩展点来“加工”对象，例如我们要代理一个对象，IOC 在对象创建完毕，直接判断下这个对象是否需要代理，如果要代理，则直接包装返回代理对象。


这等于我们只要告诉 IOC 我们要什么，IOC 就能基于我们提供的配置文件，创建符合我们需求的对象。

例如，对象 A 需要依赖一个实现 B，但是对象都由 IOC 控制之后，我们不需要明确地在对象 A 的代码里写死依赖的实现 B，只需要写明依赖一个接口，这样我们的代码就能顺序的编写下去。

然后，我们可以在配置文件里定义 A 依赖的具体的实现 B，根据配置文件，在创建 A 的时候，IOC 容器就知晓 A 依赖的 B，这时候注入这个依赖即可。

如果之后你有新的实现需要替换，那 A 的代码不需要任何改动，你只需要将配置文件 A 依赖 B 改成 B1，这样重启之后，IOC 容器会为 A 注入 B1。

这样就使得类 A 和类 B 解耦了， very nice！

# DI（Dependency Injection，依赖注入）
DI（Dependency Injection，依赖注入）普遍认为是 Spring 框架中用于实现**控制反转（IOC）** 的一种机制。DI 的核心思想是由容器负责对象的依赖注入，而不是由对象自行创建或查找依赖对象。

通过 DI，Spring 容器在创建一个对象时，会自动将这个对象的依赖注入进去，这样可以让对象与其依赖的对象解耦，提升系统的灵活性和可维护性。

<mark class="hltr-grey">依赖注入的优势</mark>
- **解耦合**：通过将对象的依赖从代码中抽离出来，由容器负责管理，降低了类之间的耦合度。
- **提高测试性**：通过注入 mock 对象或替代实现，DI 使得单元测试变得更容易。
- **提高可维护性和灵活性**：通过配置文件或注解，开发者可以在不修改代码的情况下更改依赖，增加了应用程序的可扩展性和灵活性

# Bean
任何通过 Spring 容器实例化、组装和管理的 Java 对象都可以被称为 Spring Bean。Bean 可以在 Spring 容器中被定义并且通过依赖注入来与其他 Bean 进行互相依赖。

即 Bean 可以看作是 Spring 应用中的一个对象，它的生命周期（创建、初始化、使用、销毁等过程）完全由 Spring 容器管理。

<mark class="hltr-red">Spring Bean 的生命周期</mark>
- **实例化**：Spring 容器根据配置文件或注解实例化 Bean 对象。
- **属性注入**：Spring 将依赖（通过构造器、setter 方法或字段注入）注入到 Bean 实例中。
- **初始化前的扩展机制**：如果 Bean 实现了 BeanNameAware 等 aware 接口，则执行 aware 注入。
- **初始化前**（BeanPostProcessor）：在 Bean 初始化之前，可以通过 BeanPostProcessor 接口对 Bean 进行一些额外的处理。
- **初始化**：调用 InitializingBean 接口的 afterPropertiesSet() 方法或通过 init-method 属性指定的初始化方法。
- **初始化后**（BeanPostProcessor）：在 Bean 初始化后，可以通过 BeanPostProcessor 进行进一步的处理。
- **使用** Bean：Bean 已经初始化完成，可以被容器中的其他 Bean 使用。
- **销毁**：当容器关闭时，Spring 调用 DisposableBean 接口的 destroy() 方法或通过 destroy-method 属性指定的销毁方法。

<mark class="hltr-red">定义 Spring Bean 的方式</mark>
- **XML 配置**：早期的 Spring 应用通常通过 XML 文件定义 Bean，使用 `<bean>` 标签来指定类、构造器参数和依赖关系。
- **基于注解的配置**：使用 @Component、@Service、@Repository、@Controller 等注解可以将类标记为 Spring Bean，Spring 会自动扫描这些类并将其注册为 Bean。
- **Java 配置类**：通过 @Configuration 和 @Bean 注解，可以在 Java 类中手动定义 Bean。相比 XML 配置，这种方式更加简洁和类型安全。

# 循环依赖

循环依赖（Circular Dependency）是指两个或多个模块、类、组件之间相互依赖，形成一个闭环。简而言之，模块A依赖于模块B，而模块B又依赖于模块A，这会导致依赖链的循环，无法确定加载或初始化的顺序。
```java
@Service
public class A {
    @Autowired
    private B b;
}

@Service
public class B {
    @Autowired
    private A a;
}

//或者自己依赖自己
@Service
public class A {
    @Autowired
    private A a;
}

```

## 如何解决
关键就是**提前暴露未完全创建完毕的 Bean**。

在 Spring 中主要是使用**三级缓存**来解决了循环依赖：

- 一级缓存（Singleton Objects Map）: 用于存储完全初始化完成的单例Bean。
- 二级缓存（Early Singleton Objects Map）: 用于存储尚未完全初始化，但已实例化的Bean，用于提前暴露对象，避免循环依赖问题。
- 三级缓存（Singleton Factories Map）: 用于存储对象工厂，当需要时，可以通过工厂创建早期Bean（特别是为了支持AOP代理对象的创建）。

**解决步骤**：

- Spring 首先创建 Bean 实例，并将其加入三级缓存中（Factory）。
- 当一个 Bean 依赖另一个未初始化的 Bean 时，Spring 会从三级缓存中获取 Bean 的工厂，并生成该 Bean 的对象（若有代理则是代理对象）。
- 代理对象存入二级缓存，解决循环依赖。
- 一旦所有依赖 Bean 被完全初始化，Bean 将转移到一级缓存中。

# Spring 一共有几种注入方式？ 
- 构造器注入，**Spring 倡导构造函数注入**，因为构造器注入返回给客户端使用的时候一定是完整的。
- setter 注入，可选的注入方式，好处是在有变更的情况下，可以重新注入。
- 字段注入，就是平日我们用 @Autowired **标记字段**
- 方法注入，就是平日我们用 @Autowired **标记方法**
- 接口回调注入，就是实现 Spring 定义的一些内建接口，例如 BeanFactoryAware，会进行 BeanFactory 的注入

# AOP（Aspect-Oriented Programming，面向切面编程）
是一种编程范式，用于将跨领域的关注点（如日志记录、安全检查、事务管理等）与业务逻辑分离开来。它允许开发者通过“切面”（Aspect）将这些通用功能模块化，并将其应用到应用程序中的多个地方，从而避免代码重复。

- **核心思想**：AOP 的核心思想是将与业务逻辑无关的横切关注点抽取出来，通过声明的方式动态地应用到业务方法上，而不是将这些代码直接嵌入业务逻辑中。
- **主要组成部分**：AOP 包括几个关键概念：切面（Aspect）、连接点（Join Point）、通知（Advice）、切入点（Pointcut）和织入（Weaving）。

* **消除重复代码**：避免了在每个方法中都手动添加日志、事务等代码。
* **提高模块化和可维护性**：业务逻辑更纯粹，次要功能与核心功能分离，使代码更清晰。

<mark class="hltr-grey">Spring AOP中有哪几种通知（Advice）类型？</mark>
- **`@Before` (前置通知):** 在目标方法执行之前执行。
    
- **`@AfterReturning` (后置通知/返回通知):** 在目标方法**正常执行完成并返回结果后**执行。
    
- **`@AfterThrowing` (异常通知):** 在目标方法抛出异常后执行。
    
- **`@After` (最终通知):** 无论目标方法是正常执行完成还是抛出异常，**都会执行**。类似于`finally`代码块。
    
- **`@Around` (环绕通知):** 最强大的通知类型。它集成了前四种通知的功能，可以**完全控制**目标方法的执行。你可以决定方法何时执行、是否执行，甚至修改参数和返回值。你项目中的`Re-Reading Advisor`就是典型的环绕通知。

<mark class="hltr-grey">AOP 的主要应用场景</mark>

- **日志记录**：通过 AOP 可以将日志逻辑分离到切面中，使日志代码与业务代码解耦。
- **事务管理**：可以通过 AOP 实现事务管理，确保在特定方法执行时开启事务，并在方法执行成功或失败后提交或回滚事务。
- **安全检查**：AOP 可以用于权限验证，在方法执行前检查用户是否具有相应权限。
- **性能监控**：通过环绕通知（Around advice），可以记录方法的执行时间，帮助监控应用性能。

## AOP的原理

# 你了解的 Spring 都用到哪些设计模式？ 
工厂模式，从名字就能看出来是 BeanFactory，整个 Spring IOC 就是一个工厂。

模板方法，例如 JdbcTemplate、RestTemplate，名字是 xxxTemplate 的都是模板。

代理模式，AOP 整个都是代理模式。

单例模式，默认情况下 Bean 都是单例的。

责任链模式，比如 Spring MVC 中的拦截器，多个拦截器串联起来就形成了责任链。

观察者模式，在 Spring 中的监听器实现。

适配器模式，在 Spring MVC 中提到的 handlerAdapter 其实就是适配器。

# Spring的流程、生命周期
1. **容器启动与Bean定义加载 (Scanning)**    
    - **触发**：通过 `new ApplicationContext(...)` 启动Spring容器。        
    - **动作**：Spring会扫描你指定的包路径，寻找带有 `@Component`, `@Service`, `@Repository` 等注解的类，或者解析XML配置。        
    - **产物**：此时，Spring并**没有创建对象**，而是将这些类的信息解析成一个个的`BeanDefinition`对象（可以理解为标准化的“零件蓝图”），然后将这些`BeanDefinition`注册到一个叫做`BeanDefinitionMap`的集合中。        
2. **Bean的实例化 (Instantiation)**    
    - **触发**：容器在准备好所有`BeanDefinition`后，开始遍历并创建Bean实例。默认情况下，所有单例（Singleton）作用域的Bean都会在这个阶段被创建。        
    - **动作**：Spring通过反射机制（`Constructor.newInstance()`）创建出一个“原始”的对象实例。此时，这个对象仅仅是一个空壳，它的属性都还是`null`。        
3. **属性填充 (Populate Properties / Dependency Injection)**    
    - **触发**：对象实例化之后。        
    - **动作**：Spring检查这个对象，发现其中有被`@Autowired`等注解标记的属性。Spring会去容器中查找对应的依赖Bean（如果依赖的Bean还没创建，就会先去创建那个Bean），然后通过反射将依赖Bean的引用注入到当前对象的属性中。        
    - **产物**：此时，Bean的依赖关系已经建立，但它可能还没有完全准备好工作。
        
4. **初始化 (Initialization)**    
    - 这是Bean生命周期中**最复杂、扩展点最多**的阶段，让Bean从“半成品”变为“完全体”。        
    - **`Aware`接口回调**：如果Bean实现了`BeanNameAware`、`ApplicationContextAware`等接口，Spring会回调这些接口的方法，让Bean能“感知”到自己在容器中的ID、所在的容器等环境信息。        
    - **`BeanPostProcessor`前置处理**：调用所有`BeanPostProcessor`的`postProcessBeforeInitialization`方法。这是一个非常重要的扩展点，允许我们在Bean的自定义初始化逻辑执行前，对Bean进行加工。
        
    - **自定义初始化**：        
        - 如果Bean类中定义了用`@PostConstruct`注解的方法，Spring会调用它。            
        - 如果Bean实现了`InitializingBean`接口，Spring会调用它的`afterPropertiesSet`方法。          
    - **`BeanPostProcessor`后置处理**：调用所有`BeanPostProcessor`的`postProcessAfterInitialization`方法。这**是Spring AOP实现的关键**。Spring就是在这里，将原始的目标对象（Target）包装成一个代理对象（Proxy），然后返回这个代理对象。我们后续从容器中获取的其实是这个代理对象。
        
5. **Bean投入使用 (Ready for Use)**    
    - 经过了完整的初始化阶段，这个Bean现在是一个功能完备、随时可以使用的对象了。它被存放在Spring的单例池（Singleton Cache）中，等待被其他地方调用。
        
6. **Bean的销毁 (Destruction)**    
    - **触发**：当Spring容器关闭时（调用`context.close()`）。        
    - **动作**：        
        - 如果Bean类中定义了用`@PreDestroy`注解的方法，Spring会调用它。            
        - 如果Bean实现了`DisposableBean`接口，Spring会调用它的`destroy`方法。            
    - **目的**：让Bean有机会释放它所持有的资源，比如关闭数据库连接、关闭文件流等。
