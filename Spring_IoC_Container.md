#### 什么是IoC？

Inversion of Control (IoC) is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

Code is cleaner with the DI principle, and decoupling is more effective when objects are provided with their dependencies. The object does not look up its dependencies and does not know the location or class of the dependencies. As a result, your classes become easier to test, particularly when the dependencies are on interfaces or abstract base classes, which allows for stub or mock implementations to be used in unit tests.

##### 以设计行李箱为例

###### 上层建筑依赖于下层建筑

行李箱`依赖`箱体，箱体`依赖`底盘，底盘`依赖`轮子。

上述依赖关系表现在代码层面如下所示。

```java
public class Luggage {
	private Framework framework;

	public Luggage() {
		this.framework = new Framework();
	}

	public void move() {
		System.out.println("This luggage is moving.");
	}
}
```

上述行李箱（Luggage）依赖于下述箱体（Framework）。

```java
public class Framework {
	private Bottom bottom;

	public Framework() {
		this.bottom = new Bottom();
	}
}
```

上述箱体（Framework）依赖于下述底盘（Bottom）。

```java
public class Bottom {
	private Tire tire;

	public Bottom() {
		this.tire = new Tire();
	}
}
```

上述底盘（Bottom）依赖于下述轮子（Tire）。

```java
public class Tire {
	private int size;

	public Tire() {
		this.size = 20;
	}
}
```

新建一个行李箱的方式如下。

```java
Luggage luggage = new Luggage();
luggage.move();
```

如果想要动态地改变轮子的大小，需要重构上述代码如下。

```java
public class Luggage {
	private Framework framework;

	public Luggage(int size) {
		this.framework = new Framework(size);
	}

	public void move() {
		System.out.println("This luggage is moving.");
	}
}

public class Framework {
	private Bottom bottom;

	public Framework(int size) {
		this.bottom = new Bottom(size);
	}
}

public class Bottom {
	private Tire tire;

	public Bottom(int size) {
		this.tire = new Tire(size);
	}
}

public class Tire {
	private int size;

	public Tire(int size) {
		this.size = size;
	}
}
```

新建一个行李箱的方式如下。

```java
int size = 30;
Luggage luggage = new Luggage(size);
luggage.move();
```

可以看到，仅仅修改了轮胎（Tire）的构造函数，却修改了所有上层（Bottom、Framework、Luggage）的构造函数，改动过大，代码不可维护。

###### 用依赖注入实现控制反转

把下层作为参数传递给上层，实现上层对下层的“控制”。

将轮子`注入`底盘，再将底盘`注入`箱体，最后将箱体`注入`行李箱。

上述依赖关系表现在代码层面如下所示。

```java
public class Luggage {
	private Framework framework;

	public Luggage(Framework framework) {
		this.framework = framework;
	}

	public void move() {
		System.out.println("This luggage is moving.");
	}
}

public class Framework {
	private Bottom bottom;

	public Framework(Bottom bottom) {
		this.bottom = bottom;
	}
}

public class Bottom {
	private Tire tire;

	public Bottom(Tire tire) {
		this.tire = tire;
	}
}

public class Tire {
	private int size;

	public Tire() {
		this.size = 20;
	}
}
```

新建一个行李箱的方式如下。

```java
Tire tire = new Tire();
Bottom bottom = new Bottom(tire);
Framework framework = new Framework(bottom);
Luggage luggage = new Luggage(framework);
luggage.move();
```

如果想要动态地改变轮子的大小，可以重构上述代码如下。

```java
public class Luggage {
	private Framework framework;

	public Luggage(Framework framework) {
		this.framework = framework;
	}

	public void move() {
		System.out.println("This luggage is moving.");
	}
}

public class Framework {
	private Bottom bottom;

	public Framework(Bottom bottom) {
		this.bottom = bottom;
	}
}

public class Bottom {
	private Tire tire;

	public Bottom(Tire tire) {
		this.tire = tire;
	}
}

public class Tire {
	private int size;

	public Tire(int size) {
		this.size = size;
	}
}
```

可以看到，仅虚修改了轮胎（Tire）的构造函数，无需修改了所有上层（Bottom、Framework、Luggage）的构造函数，代码可维护性高。

#### 什么是IoC容器？

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework's IoC container. The `BeanFactory` interface provides an advanced configuration mechanism capable of managing any type of object. `ApplicationContext` is a sub-interface of `BeanFactory`. It adds:

- Easier integration with Spring's AOP features

- Message resource handling (for use in internationalization)

- Event publication

- Application-layer specific contexts such as the `WebApplicationContext` for use in web applications.

In short, the `BeanFactory` provides the configuration framework and basic functionality, and the `ApplicationContext` adds more enterprise-specific functionality. The `ApplicationContext` is a complete superset of the `BeanFactory`.

#### 什么是bean？

In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.

#### IoC容器是如何工作呢？

![](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/container-magic.png)

The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code. It lets you express the objects that compose your application and the rich interdependencies between those objects.

Several implementations of the `ApplicationContext` interface are supplied with Spring. In stand-alone applications, it is common to create an instance of `ClassPathXmlApplicationContext` or `FileSystemXmlApplicationContext`. While XML has been the traditional format for defining configuration metadata, you can instruct the container to use Java annotations or code as the metadata format by providing a small amount of XML configuration to declaratively enable support for these additional metadata formats.

#### 什么是Configuration Metadata？

Configuration metadata represents how you, as an application developer, tell the Spring container to instantiate, configure, and assemble the objects in your application.

Spring configuration consists of at least one and typically more than one bean definition that the container must manage. XML-based configuration metadata configures these beans as `<bean/>` elements inside a top-level `<beans/>` element. Java configuration typically uses `@Bean`-annotated methods within a `@Configuration` class.

#### 什么是BeanDefinition？

A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML `<bean/>` definitions). Within the container itself, these bean definitions are represented as `BeanDefinition` objects, which contain (among other information) the following metadata:

- A package-qualified class name: typically, the actual implementation class of the bean being defined.

- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).

- References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.

- Other configuration settings to set in the newly created object - for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.

When you create a bean definition, you create a recipe for creating actual instances of the class defined by that bean definition. The idea that a bean definition is a recipe is important, because it means that, as with a class, you can create many object instances from a single recipe.

#### 如何手动注册BeanDefinition？

In addition to bean definitions that contain information on how to create a specific bean, the `ApplicationContext` implementations also permit the registration of existing objects that are created outside the container (by users). This is done by accessing the ApplicationContext's BeanFactory through the `getBeanFactory()` method, which returns the BeanFactory `DefaultListableBeanFactory` implementation. `DefaultListableBeanFactory` supports this registration through the `registerSingleton(..)` and `registerBeanDefinition(..)` methods. However, typical applications work solely with beans defined through regular bean definition metadata.

##### 手动注册BeanDefinition存在哪些问题呢？

Bean metadata and manually supplied singleton instances need to be registered as early as possible, in order for the container to properly reason about them during autowiring and other introspection steps. While overriding existing metadata and existing singleton instances is supported to some degree, the registration of new beans at runtime (concurrently with live access to the factory) is not officially supported and may lead to concurrent access exceptions, inconsistent state in the bean container, or both.

#### 未显示指定name和id的bean的命名规则是怎么样的？

如果该bean是在XML中显示定义的：

If you do not supply a `name` or `id` explicitly, the container generates a unique name for that bean. （并不是首字母小写的形式） However, if you want to refer to that bean by name, through the use of the `ref` element or a Service Locator style lookup, you must provide a name. Motivations for not supplying a name are related to using inner beans and autowiring collaborators.

如果该bean是由component scanning机制在classpath中被扫描到的：

With component scanning in the classpath, Spring generates bean names for unnamed components, following the rules: essentially, taking the simple class name and turning initial character to lower-case. However, in the (unusual) special case when there is more than one character and both the first and second characters are upper case, the original casing gets preserved.

#### 如何在XML配置文件中指定静态内部类的class属性？

If you want to configure a bean definition for a `static` nested class, you have to use the binary name of the nested class.

For example, if you have a class called `SomeThing` in the `com.example` package, and this `SomeThing` class has a `static` nested class called `OtherThing`, the value of the `class` attribute on a bean definition would be `com.example.SomeThing$OtherThing`. （`com.example.SomeThing.OtherThing`也是可以的）

Notice the use of the `$` character in the name to separate the nested class name from the outer class name.

#### PropertyEditor的作用是什么？

`*.properties`是Java程序常用的数据存储文件，Spring提供了`BeanWrapper`接口将`*.properties`文件中的数据转换成一个标准的JavaBean对象。

```java
public class Person {
	private String name;

	private int age;

	private boolean license;

	private Date birtday;

	private Address address;

	private Map<String, String> otherInfo = new HashMap<>();

    // Getter & Setter ......
}
```

对于上述实体类Person，可以通过`BeanWrapper`将`Properties`对象中的数据设置到对象中。

```java
BeanWrapper wrapper = new BeanWrapperImpl(new Person());

//使用 BeanWrapper::setPropertyValue 接口设置数据
wrapper.setPropertyValue("name", "niubility");
wrapper.setPropertyValue("age", 18);
wrapper.setPropertyValue("license", true);
System.out.println(wrapper.getWrappedInstance());

//使用 Properties对象设置数据，Properties实例可以来源于*.properties文件
Properties p = new Properties();
p.setProperty("name", "From Properties");
p.setProperty("age", "25");
p.setProperty("license", "false");
p.setProperty("otherInfo[birthday]", "2000-01-01");
wrapper.setPropertyValues(p);
System.out.println(wrapper.getWrappedInstance());
```

在JavaBean规范中定义了`PropertyEditor`，用于将字符串转换为对象。`BeanWrapper`继承了`PropertyEditorRegistry`接口用于注册`PropertyEditor`。`BeanWrapperImpl`预置了许多简单类型的`PropertyEditor`，比如上述例子中的代码`p.setProperty("age", "25");`，`age`是一个`int`类型，在设置数据时会自动启用`CustomNumberEditor`进行格式转换。除了预置的`PropertyEditor`外，我们还可以自定义`PropertyEditor`。

Person中有一个类型为Address的成员变量，我们需要为Address添加一个自定义的`PropertyEditor`，用于字符串和Address对象之间的转换。

```java
public class Address {
	private String province; //省

	private String city;  //市

	private String district;  //区

    // Getter & Setter
}

public class AddressEditor extends PropertyEditorSupport {
	private String[] SPLIT_FLAG = { ",", "-", ";", ":" };

	public void setAsText(String text) {
		int pos = -1;
		Address address = new Address();
		for (String flag : SPLIT_FLAG) {
			pos = text.indexOf(flag);
			if (-1 < pos) {
				String[] split = text.split(flag);
				address.setProvince(split[0]);
				address.setCity(split[1]);
				address.setDistrict(split[2]);
				break;
			}
		}
		if (-1 == pos) {
			throw new IllegalArgumentException("地址格式错误");
		}
		setValue(address);	//设定Address实例
	}
}
```

我们可以使用AddressEditor来讲字符串转换为Address对象。

```java
//使用预设转换工具和自定义转换工具
BeanWrapper wrapper = new BeanWrapperImpl(new Person());
// 创建AddressEditor实例
AddressEditor addressEditor = new AddressEditor();
// 注册addressEditor，将其与Address类进行绑定
wrapper.registerCustomEditor(Address.class, addressEditor);
// 设置值自动进行转化
wrapper.setPropertyValue("address", "广东-广州-白云");
System.out.println(wrapper.getWrappedInstance());
```

按照JavaBean规范，`PropertyEditor`和对应的JavaBean可以用命名规则来表示绑定关系，而无需显式调用注册方法。规则如下：如果一个JavaBean命名为A，在相同的包下由一个实现了PropertyEditor接口并且命名为AEditor的类，那么就无需调用`BeanWrapper::registerCustomEditor`方法来声明A和AEditor的绑定关系。

对于Spring的ApplicationContext而言，`BeanWrapper`、`PropertyEditor`都是相对比较底层的类，IoC容器使用`CustomEditorConfigurer`（一个`BeanFactoryPostProcessor`）来管理Bean初始化相关的`PropertyEditor`。通过`CustomEditorConfigurer`可以使用所有预置的Editor，还可以增加自定义的Editor。

```java
@Configuration
@ImportResource("classpath:com/repeat/read/propertyeditor/spring-config.xml")
public class BeanManipulationConfig {
	@Bean
	public static CustomEditorConfigurer customEditorConfigurer() {	//由于CustomEditorConfigurer是一个BeanFactoryPostProcessor，故该方法必须是static方法
		// 构建CustomEditorConfigurer
		CustomEditorConfigurer configurer = new CustomEditorConfigurer();
		
		// 注册自定义PropertyEditor的方式一

		Map<Class<?>, Class<? extends PropertyEditor>> customEditors = new HashMap<>();
		
		// 添加AddressEditor和Address的绑定
		customEditors.put(Address.class, AddressEditor.class);
		
		// 添加绑定列表
		configurer.setCustomEditors(customEditors);
		
		// 注册自定义PropertyEditor的方式二

		// 通过PropertyEditorRegistrar注册PropertyEditor
		configurer.setPropertyEditorRegistrars(new PropertyEditorRegistrar[] { new DateFormatRegistrar() });
		return configurer;
	}
}

public class DateFormatRegistrar implements PropertyEditorRegistrar {
	@Override
	public void registerCustomEditors(PropertyEditorRegistry registry) {
		DateFormat df = new java.text.SimpleDateFormat("yyyy-MM-dd");
		CustomDateEditor editor = new CustomDateEditor(df, false);
		registry.registerCustomEditor(Date.class, editor);
	}
}
```

#### 构造函数注入和setter方法注入，哪种更好呢？

It is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies. Note that use of the `@Required` annotation on a setter method can be used to make the property be a required dependency; however, constructor injection with programmatic validation of arguments is preferable.

The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not `null`. Furthermore, constructor-injected components are always returned to the client code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later.

#### 依赖解析过程

- The `ApplicationContext` is created and initialized with configuration metadata that describes all the beans. Configuration metadata can be specified by XML, Java code, or annotations.

- For each bean, its dependencies are expressed in the form of properties, constructor arguments, or arguments to the static-factory method (if you use that instead of a normal constructor). These dependencies are provided to the bean, when the bean is actually created.

- Each property or constructor argument is an actual definition of the value to set, or a reference to another bean in the container.

- Each property or constructor argument that is a value is converted from its specified format to the actual type of that property or constructor argument. By default, Spring can convert a value supplied in string format to all built-in types, such as `int`, `long`, `String`, `boolean`, and so forth.

##### bean创建时机

The Spring container validates the configuration of each bean as the container is created. However, the bean properties themselves are not set until the bean is actually created. Beans that are singleton-scoped and set to be pre-instantiated (the default) are created when the container is created. Otherwise, the bean is created only when it is requested. Creation of a bean potentially causes a graph of beans to be created, as the bean's dependencies and its dependencies' dependencies (and so on) are created and assigned. Note that resolution mismatches among those dependencies may show up late - that is ,on first creation of the affected bean.

##### 循环依赖问题

If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.

For example: Class A requires an instance of class B through constructor injection, and class B requires an instance of class A through constructor injection. If you configure beans for classes A and B to be injected into each other, the Spring IoC container detects this circular reference at runtime, and throws a `BeanCurrentlyInCreationException`.

One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.

##### ApplicationContext为什么选择在IoC容器创建的时候就初始化作用域为Singleton的bean实例？

Spring detects configuration problems, such as references to non-existent beans and circular dependencies, at container load-time. Spring sets properties and resolves dependencies as late as possible, when the bean is actually created. This means that a Spring container that has loaded correctly can later generate an exception when you request an object is there is a problem creating that object or one of its dependencies - for example, the bean throws an exception as a result of a missing or invalid property. This potentially delayed visibility of some configuration issues is why `ApplicationContext` implementations by default pre-instantiate singleton beans. At the cost of some upfront time and memory to create these beans before they are actually needed, you discover configuration issues when the `ApplicationContext` is created, not later. You can still override this default behavior so that singleton beans initialize lazily, rather being pre-instantiated.

#### 关于Inner Beans的一些注意点

An inner bean definition does not require a defined ID or name. If specified, the container does not use such a value as an identifier. The container also ignores the `scope` flag on creation, because inner beans are always anonymous and are always created with the outer bean. It is not possible to access inner beans independently or to inject them into collaborating beans other than into the enclosing bean.

As a corner case, it is possible to receive destruction callbacks from a custom scope - for example, for a request-scoped inner bean contained within a singleton bean. The creation of the inner bean instance is tied to its containing bean, but destruction callbacks let it participate in the request scope's lifecycle. This is not a common scenario. Inner beans typically simply share their containing bean's scope.

#### depends-on不仅能控制bean的初始化顺序，而且能控制bean的销毁顺序

The `depends-on` attribute can specify both an initialization-time dependency and, in the case of singleton beans only, a corresponding destruction-time dependency. Dependent beans that define a `depends-on` relationship with a given bean are destroyed first, prior to the given bean itself being destroyed. Thus, `depends-on` can also control shutdown order.

#### 自动注入的优缺点

##### 自动注入的优点

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template are also valuable in this regard.)

- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.

##### 自动注入的缺点

- Explicit dependencies in `property` and `constructor-arg` settings always override autowiring. You cannot autowire simple properties such as primitives, `Strings`, and `Classes` (and arrays of such simple properties). This limitation is by-design.

- Autowiring is less exact than explicit wiring. Although, Spring is careful to avoid guessing in case of ambiguity that might have unexpected results. The relationships between your Spring-managed objects are no longer documented explicitly.

- Wiring information may not be available to tools that may generate documentation from a Spring container.

- Multiple bean definitions within the container may match the type specified by the setter method or constructor argument to be autowired. For arrays, collections, or `Map` instances, this is not necessary a problem. However, for dependencies that expect a single value, this ambiguity is not arbitrarily resolved. If no unique bean definition is available, an exception is thrown.

#### 利用`autowire-candidate`排除一个bean的局限性

The `autowire-candidate` attribute is designed to only affect type-based autowiring. It does not affect explicit references by name, which get resolved even if the specified bean is not marked as an autowire candidate. As a consequence, autowiring by name nevertheless injects a bean if the name matches.

#### 为什么需要Method Injection？

In most application scenarios, most beans in the container are singletons. When a singleton bean needs to collaborate with another singleton bean or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycle are different. Suppost singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container creates the singleton bean A only once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

#### 什么是Lookup Method Injection？

Lookup method injection is the ability of the container to override methods on container-managed beans and return the lookup result for another named bean in the container. The lookup typically involves a prototype bean. The Spring Framework implements this method injection by using bytecode generation from the CGLIB library to dynamically generate a subclass that overrides the method.

##### Lookup Method Injection的注意点

- For this dynamic subclassing to work, the class that the Spring bean container subclasses cannot be `final`, and the method to be overridden cannot be `final`, either.

- Unit-testing a class that has an `abstract` method requires you to subclass the class yourself and to supply a stub implementation of the `abstract` method.

- Concrete methods are also necessary for component scanning, which requires concrete classes to pick up.

- A further key limitation is that lookup methods do not work with factory methods and in particular not with `@Bean` methods in configuration classes, since, in that case, the container is not in charge of creating the instance and therefore cannot create a runtime-generated subclass on the fly.

#### bean的6种作用域

##### singleton

![](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/singleton.png)

Spring's concept of a singleton bean differs from the singleton pattern as defined in the Gang of Four (GoF) patterns book. The GoF singleton hard-codes the scope of an object such that one and only one instance of a particular class is created per ClassLoader. The scope of the Spring singleton is best described as being per-container and per-bean. This means that, if you define one bean for a particular class in a single Spring container, the Spring container creates one and only one instance of the class defined by that bean definition.

##### prototype

![](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/images/prototype.png)

In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean. The container instantiates, configures, and otherwise assembles a prototype object and hands it to the client, with no further record of that prototype instance. Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype beans hold. To get the Spring container to release resources held by prototype-scoped beans, try using a custom bean post-processor, which holds a reference to beans that need to be cleaned up.

In some respects, the Spring container's role in regard to a prototype-scoped bean is a replacement for the Java `new` operator. All lifecycle management past that point must be handled by the client.

##### request

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

The Spring container creates a new instance of the `LoginAction` bean by using the `loginAction` bean definition for each and every HTTP request. That is, the `loginAction` bean is scoped at the HTTP request level. You can change the internal state of the instance that is created as much as you want, because other instances created from the same `loginAction` bean definition do not see these changes in state. They are particular to an individual request. When the request completes processing, the bean that is scoped to the request is discarded.

##### session

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

The Spring container creates a new instance of the `UserPreferences` bean by using the `userPreferences` bean definition for the lifetime of a single HTTP `Session`. In other words, the `userPreferences` bean is effectively scoped at the HTTP `Session` level. As with request-scoped beans, you can change the internal state of the instance that is created as much as you want, knowing that other HTTP `Session` instances that are also using instances created from the same `userPreferences` bean definition do not see these changes in state, because they are particular to an individual HTTP `Session`. When the HTTP `Session` is eventually discarded, the bean that is scoped to that particular HTTP `Session` is also discarded.

##### application

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

The Spring container creates a new instance of the `AppPreferences` bean by using the `appPreferences` bean definition once for the entire web application. That is, the `appPreferences` bean is scoped at the `ServletContext` level and stored as a regular `ServletContext` attribute. This is somewhat similar to a Spring singleton bean but differs in two important ways: It is a singleton per `ServletContext`, not per Spring 'ApplicationContext' (for which there may be several in any given web application), and it is actually exposed and there visible as a `ServletContext` attribute.

#### 将小作用域对象注入大作用域对象需要使用`<aop:scoped-proxy/>`

```xml
<!-- an HTTP Session-scoped bean exposed as a proxy -->
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
	<!-- instructs the container to proxy the surrounding bean -->
	<aop:scoped-proxy/> 
</bean>

<!-- a singleton-scoped bean injected with a proxy to the above bean -->
<bean id="userService" class="com.something.SimpleUserService">
	<!-- a reference to the proxied userPreferences bean -->
	<property name="userPreferences" ref="userPreferences"/>
</bean>
```

The Spring IoC container manages not only the instantiation of your objects (beans), but also the wiring up of collaborators (or dependencies). If you want to inject (for example) an HTTP request-scoped bean into another bean of a longer-lived scope, you may choose to inject an AOP proxy in place of the scoped bean. That is, you need to inject a proxy object that exposes the same public interface as the scoped object but can also retrieve the real target object from the revelant scope (such as an HTTP request) and delegate method calls onto the real object.






























































