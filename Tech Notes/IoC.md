### What is IoC?
- Normally, when you write code, your classes and objects directly create or manage their own dependencies using something like:
```java
DatabaseService db = new DatabaseService();
```

```java
public class UserService { // example without IoC
    private DatabaseService db;

    public UserService() {
        db = new DatabaseService(); // tightly coupled
    }

    public void process() {
        db.query("SELECT * FROM users");
    }
}
```
- This means your class **controls** the creation of its dependencies.
- Here, `UserService` **creates its own** `DatabaseService` dependency.
- This makes it tightly coupled — you can’t easily switch `DatabaseService` to something else (like a mock for testing). That's the problem with this approach.
- But with **IoC**, **you give up that control**. Instead of creating the `DatabaseService` yourself, someone else (usually a framework or container like Spring) gives it to you. That’s why it’s called _Inversion_ of Control — because the control is **inverted**, and given to the container or framework.
```java
public class UserService { //example with IoC
    private DatabaseService db;

    public UserService(DatabaseService db) {
        this.db = db; // dependency is injected
    }

    public void process() {
        db.query("SELECT * FROM users");
    }
}
```
### Dependency Injection
- Code is cleaner with the DI principle, and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies and does not know the location or class of the dependencies. As a result, your classes become easier to test, particularly when the dependencies are on interfaces or abstract base classes, which allow for stub or mock implementations to be used in unit tests.
- Dependency Injection is a **specific way to implement IoC**. It's about **injecting dependencies** into an object, instead of the object creating them itself.
- There are 3 common types of injection:
	1. **Constructor Injection** (most preferred)
	    - Dependencies are passed through the constructor.
	2. **Setter Injection**
	    - Dependencies are set through setter methods after the object is created.    
	3. **Field Injection** (in frameworks like Spring)
	    - Dependencies are injected directly into fields (using annotations like `@Autowired` in Spring).
### Container Overview
- Spring provides an **IoC container** that manages the lifecycle and dependencies of objects (called **beans**).
- In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
- The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on the components to instantiate, configure, and assemble by reading configuration metadata.
- The configuration metadata can be represented as annotated component classes, configuration classes with factory methods(`@Configuration` class with `@Bean` methods), or external XML files or Groovy scripts.
![[Pasted image 20250413154756.png]]
- These days, many developers choose [Java-based configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html) for their Spring applications, that is, define beans external to your application classes by using Java-based configuration classes. To use these features, see the [`@Configuration`](https://docs.spring.io/spring-framework/docs/6.2.5/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/6.2.5/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/6.2.5/javadoc-api/org/springframework/context/annotation/Import.html), and [`@DependsOn`](https://docs.spring.io/spring-framework/docs/6.2.5/javadoc-api/org/springframework/context/annotation/DependsOn.html) annotations.
### Bean overview
- Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:
	- A package-qualified class name: typically, the actual implementation class of the bean being defined.
	- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
	- References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
	- Other configuration settings to set in the newly created object — for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.
- In Spring, **singleton** means:The **container creates only one instance** of the bean per Spring `ApplicationContext`.  Every time that bean is needed (e.g., autowired somewhere), **the same instance** is reused.
- In addition to bean definitions that contain information on how to create a specific bean, the `ApplicationContext` implementations also permit the registration of existing objects that are created outside the container (by users). This is done by accessing the ApplicationContext’s `BeanFactory` through the `getAutowireCapableBeanFactory()` method, which returns the `DefaultListableBeanFactory` implementation. `DefaultListableBeanFactory` supports this registration through the `registerSingleton(..)` and `registerBeanDefinition(..)` methods. However, typical applications work solely with beans defined through regular bean definition metadata.
 Example:
```java
ExternalService service = new ExternalService();
```

```java
@Autowired
private ApplicationContext applicationContext;

public void registerManuallyCreatedBean() {
    ConfigurableListableBeanFactory factory = 
        ((ConfigurableApplicationContext) applicationContext).getBeanFactory();

    factory.registerSingleton("externalService", service);
}
```

Now, this `ExternalService` instance becomes a **Spring-managed singleton bean**, and you can autowire it elsewhere:
```java
@Autowired
private ExternalService externalService;
```


### Lazy-Initialized beans
- By default, `ApplicationContext` implementations eagerly create and configure all [singleton](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-singleton) beans as part of the initialization process. Generally, this pre-instantiation is desirable, because errors in the configuration or surrounding environment are discovered immediately, as opposed to hours or even days later. When this behavior is not desirable, you can prevent pre-instantiation of a singleton bean by marking the bean definition as being lazy-initialized. A lazy-initialized bean tells the IoC container to create a bean instance when it is first requested, rather than at startup.
- However, when a lazy-initialized bean is a dependency of a singleton bean that is not lazy-initialized, the `ApplicationContext` creates the lazy-initialized bean at startup, because it must satisfy the singleton’s dependencies. The lazy-initialized bean is injected into a singleton bean elsewhere that is not lazy-initialized.
### Bean scope
- Beans can be defined to be deployed in one of a number of scopes. The Spring Framework supports six scopes, four of which are available only if you use a web-aware (which has servlet context) `ApplicationContext`. You can also create [a custom scope.](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-custom)
- |Scope|Description|
|---|---|
|[singleton](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-singleton)|(Default) Scopes a single bean definition to a single object instance for each Spring IoC container.|
|[prototype](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-prototype)|Scopes a single bean definition to any number of object instances.|
|[request](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-request)|Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`.|
|[session](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-session)|Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`.|
|[application](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-application)|Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`.|
|[websocket](https://docs.spring.io/spring-framework/reference/web/websocket/stomp/scope.html)|Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`.|

- As a rule, you should use the prototype scope for all stateful beans and the singleton scope for stateless beans.
- In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean. The container instantiates, configures, and otherwise assembles a prototype object and hands it to the client, with no further record of that prototype instance. Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold. To get the Spring container to release resources held by prototype-scoped beans, try using a custom [bean post-processor](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html#beans-factory-extension-bpp) which holds a reference to beans that need to be cleaned up. In some respects, the Spring container’s role in regard to a prototype-scoped bean is a replacement for the Java `new` operator. All lifecycle management past that point must be handled by the client. (For details on the lifecycle of a bean in the Spring container, see [Lifecycle Callbacks](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html#beans-factory-lifecycle).)
- To integrate your custom scopes into the Spring container, you need to implement the `org.springframework.beans.factory.config.Scope` interface, which is described in this section. The `Scope` interface has four methods to get objects from the scope, remove them from the scope, and let them be destroyed.
### Lifecycle Callbacks
- To interact with the container’s management of the bean lifecycle, you can implement the Spring `InitializingBean` and `DisposableBean` interfaces. The container calls `afterPropertiesSet()` for the former and `destroy()` for the latter to let the bean perform certain actions upon initialization and destruction of your beans.
- The `@PostConstruct` and `@PreDestroy` annotations are generally considered best practice for receiving lifecycle callbacks in a modern Spring application. Using these annotations means that your beans are not coupled to Spring-specific interfaces. For details, see [Using `@PostConstruct` and `@PreDestroy`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/postconstruct-and-predestroy-annotations.html).
- Internally, the Spring Framework uses `BeanPostProcessor` implementations to process any callback interfaces it can find and call the appropriate methods. If you need custom features or other lifecycle behavior Spring does not by default offer, you can implement a `BeanPostProcessor` yourself.
- The Spring container guarantees that a configured initialization callback is called immediately after a bean is supplied with all dependencies. Thus, the initialization callback is called on the raw bean reference, which means that AOP interceptors and so forth are not yet applied to the bean. A target bean is fully created first and then an AOP proxy (for example) with its interceptor chain is applied. If the target bean and the proxy are defined separately, your code can even interact with the raw target bean, bypassing the proxy. Hence, it would be inconsistent to apply the interceptors to the `init` method, because doing so would couple the lifecycle of the target bean to its proxy or interceptors and leave strange semantics when your code interacts directly with the raw target bean.