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

可以看到，仅需修改轮胎（Tire）的构造函数，无需修改了所有上层（Bottom、Framework、Luggage）的构造函数，代码可维护性高。

#### 什么是IoC容器？

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework's IoC container. The `BeanFactory` interface provides an advanced configuration mechanism capable of managing any type of object. `ApplicationContext` is a sub-interface of `BeanFactory`. It adds:

- Access to messages in i18n-style, through the `MessageSource` interface.

- Access to resources, such as URLs and files, through the `ResourceLoader` interface.

- Event publication, namely to beans that implement the `ApplicationListener` interface, through the use of the `ApplicationEventPublisher` interface.

- Loading of multiple (hierarchical) contexts, letting each be focused on one particular layer, such as the web layer of an application, through the `HierarchicalBeanFactory` interface.

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

BeanWrapperImpl中注册了许多默认的`PropertyEditor`实现类：

| Class | Explanation |
| :-: | :-: |
| `ByteArrayPropertyEditor` | Editor for byte arrays. Converts strings to their corresponding byte representations. Registered by default by `BeanWrapperImpl`. |
| `ClassEditor` | Parses Strings that represent classes to actual classes and vice-versa. When a class is not found, an `IllegalArgumentException` is thrown. By default, registered by `BeanWrapperImpl`. |
| `CustomBooleanEditor` | Customizable property editor for `Boolean` properties. By default, registered by `BeanWrapperImpl` but can be overridden by registering a custom instance of it as a custom editor. |
| `CustomCollectionEditor` | Property editor for collections, converting any source `Collection` to a given target `Collection` type. |
| `CustomDateEditor` | Customizable property editor for `java.util.Date`, supporting a custom `DateFormat`. NOT registered by default. Must be user-registered with the appropriate format as needed. |
| `CustomNumberEditor` | Customizable property editor for any `Number` subclass, such as `Integer`, `Long`, `Float`, or `Double`. By default, registered by `BeanWrapperImpl` but can be overridden by registering a custom instance of it as a custom editor. |
| `FileEditor` | Resolves strings to `java.io.File` objects. By default, registered by `BeanWrapperImpl`. |
| `InputStreamEditor` | One-way property editor that can take a string and produce (through an intermediate `ResourceEditor` and `Resource`) and `InputStream` so that `InputStream` properties may be directly set as strings. Note that the default usage does not close the `InputStream` for you. By default, registered by `BeanWrapperImpl`. |
| `LocaleEditor` | Can resolve strings to `Locale` objects and vice-versa (the string format is `[country][variant]`, same as the `toString` method of `Locale`). By default, registered by `BeanWrapperImpl`. |
| `PatternEditor` | Can resolve strings to `java.util.regex.Pattern` objects and vice-versa. |
| `PropertiesEditor` | Can convert strings (formatted with the format defined in the javadoc of the `java.util.Properties` class) to `Properties` objects. By default, registered by `BeanWrapperImpl`. |
| `StringTrimmerEditor` | Property editor that trims strings. Optionally allows transforming an empty string into a `null` value. NOT registered by default - must be user-registered. |
| `URLEditor` | Can resolve a string representation of a URL to an actual `URL` object. By default, registered by `BeanWrapperImpl`. |

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

我们可以使用AddressEditor来将字符串转换为Address对象。

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

If there is a need to register other custom `PropertyEditors`, several mechanisms are available. The most manual approach, which is not normally convenient or recommended, is to use the `registerCustomerEditor()` method of the `ConfigurableBeanFactory` interface, assuming you have a `BeanFactory` reference. Another (slightly more convenient) mechanism is to use a special bean factory post-processor called `CustomEditorConfigurer`. Although you can use bean factory post-processors with `BeanFactory` implementations, the `CustomEditorConfigurer` has a nested property setup, so we strongly recommend that you use it with the `ApplicationContext`, where you can deploy it in similar fashion to any other bean and where is can be automatically detected and applied.

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

Spring detects configuration problems, such as references to non-existent beans and circular dependencies, at container load-time. Spring sets properties and resolves dependencies as late as possible, when the bean is actually created. This means that a Spring container that has loaded correctly can later generate an exception when you request an object if there is a problem creating that object or one of its dependencies - for example, the bean throws an exception as a result of a missing or invalid property. This potentially delayed visibility of some configuration issues is why `ApplicationContext` implementations by default pre-instantiate singleton beans. At the cost of some upfront time and memory to create these beans before they are actually needed, you discover configuration issues when the `ApplicationContext` is created, not later. You can still override this default behavior so that singleton beans initialize lazily, rather being pre-instantiated.

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

In most application scenarios, most beans in the container are singletons. When a singleton bean needs to collaborate with another singleton bean or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycle are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container creates the singleton bean A only once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

#### 什么是Lookup Method Injection？

Lookup method injection is the ability of the container to override methods on container-managed beans and return the lookup result for another named bean in the container. The lookup typically involves a prototype bean. The Spring Framework implements this method injection by using bytecode generation from the CGLIB library to dynamically generate a subclass that overrides the method.

##### Lookup Method Injection的注意点?????

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

##### websocket?????

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

#### 默认的destroy方法

You can assign the `destroy-method` attribute of a `<bean/>` element a special `(inferred)` value, which instructs Spring to automatically detect a public `close` or `shutdown` method on the specific bean class. (Any class that implements `java.lang.AutoCloseable` or `java.io.Closeable` would therefore match.) You can also set this special `(inferred)` value on the `default-destroy-method` attribute of a `<beans/>` element to apply this behavior to an entire set of beans. Note that this is the default behavior with Java Configuration.

#### bean初始化时，`@PostConstruct`注解的方法、setter注入、构造函数注入、`InitializingBean`的`afterPropertiesSet()`方法、`init-method`的执行顺序

（1）构造函数注入

（2）setter注入

（3）`@PostConstruct`注解的方法（多个`@PostConstruct`注解的方法的执行顺序是其在类中出现的顺序执行）

（4）`InitializingBean`的`afterPropertiesSet()`方法

（5）`init-method`

##### 如何开启对`@PostConstruct`的支持？

在XML文件中加入如下语句。

```xml
<context:annotation-config/>
```

##### 如果`InitializingBean`的`afterPropertiesSet()`方法被`@PostConstruct`标记，该方法只会执行一次。

#### IoC容器销毁时，`@PreDestroy`注解的方法、`DisposableBean`的`destroy()`方法、`destroy-method`的执行顺序

（1）`@PreDestroy`注解的方法。（多个`@PreDestroy`注解的方法的执行顺序是其在类中出现的顺序执行）

（2）`DisposableBean`的`destroy()`方法。

（3）`destroy-method`。

##### 如何在IoC容器销毁时，测试这些方法的执行顺序？

Spring's web-based `ApplicationContext` implementations already have code in place to gracefully shut down the Spring IoC container when the relevant web application is shut down. If you use Spring's IoC container in a non-web application environment (for example, in a rich desktop environment), register a shutdown hook with the JVM. Doing so ensures a graceful shutdown and calls the relevant destroy methods on your singleton beans so that all resources are released.

To register a shudown hook, call the `registerShutdownHook()` method that is declared on the `ConfigurableApplicationContext` interface.

```java
ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
ctx.registerShutdownHook();	// add a shutdown hook for the above context...
```

#### LifeCycle接口的作用是什么？

```java
public interface Lifecycle {
    void start();

    void stop();

    boolean isRunning();
}
```

任意Spring管理的bean实例都可以实现`Lifecycle`接口，在IoC容器加载和初始化所有bean之后、收到stop()信号之后，`LifecycleProcessor`会调用所有`Lifecycle`对应的方法。

```java
public interface LifecycleProcessor extends Lifecycle {
    void onRefresh();

    void onClose();
}
```

##### Lifecycle的所有方法不会在IoC容器加载和初始化所有bean之后、收到stop信号之后被自动调用

Note that the regular `org.springframework.context.Lifecycle` interface is a plain contract for explicit start and stop notifications and does not imply auto-startup at context refresh time. For fine-grained control over auto-startup of a specific bean (including startup phases), consider implementing `org.springframework.context.SmartLifecycle` instead.

```java
ConfigurableApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-config.xml");
applicationContext.registerShutdownHook();
//需要显式调用start()和stop()方法，Lifecycle接口的start()和stop()方法才会被调用
applicationContext.start();	
applicationContext.stop();
```

##### Lifecycle的stop()方法一定先于`@PreDestroy`注解的方法、`DisposableBean`的`destroy()`方法和`destroy-method`吗？

On regular shutdown, all `Lifecycle` beans first receive a stop notification before the general destruction callbacks are being propagated. However, on hot refresh during a context's lifetime or on aborted refresh attempts, only destroy methods are called.

##### SmartLifecycle可以实现对stop()和start()方法的自动调用，但是其stop()方法是stop(Runnable callback)方法，接受一个回调参数Runnable callback

在SmartLifetcycle的默认实现里，stop(Runnable callback)是这样的。

```java
default void stop(Runnable callback) {
	stop();
	callback.run();
}
```

The stop method defined by `SmartLifecycle` accepts a callback. Any Implementation must invoke that callback's `run()` method after that implementation's shutdown process is complete. That enables asynchronous shutdown where necessary, since the default implementation of the `LifecycleProcessor` interface, `DefaultLifecycleProcessor`, waits up to its timeout value for the group of objects within each phase to invoke that callback. The default per-phase timeout is 30 seconds. You can override the default lifecycle processor instance by defining a bean named `lifecycleProcessor` within the context. If you want only to modify the timeout, defining the following would suffice:

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

#### 什么是ServletContext？

ServletContext是Servlet与Servlet容器之间进行直接通信的接口。Servlet容器在启动一个webapp时，会为它创建一个ServletContext对象，即每个webapp都有唯一一个ServletContext对象。同一个webapp的所有Servlet对象共享一个ServletContext，Servlet对象可以通过ServletContext来访问容器中的各种资源。

##### 如何获取一个ServletContext？

```java
javax.servlet.http.HttpSession::getServletContext()
javax.servlet.jsp.PageContext::getServletContext()
javax.servlet.ServletConfig::getServletContext()
javax.servlet.ServletRequest::getServletContext()
```

##### ServletContext有哪些作用？

（1）在webapp范围内进行数据的共享

| 方法 | 作用 |
| :-: | :-: |
| setAttribute(String name, Object object) | 以key-value的形式将数据存入ServletContext，比如Spring中作用域为application的bean实例 |
| Object getAttribute(String name) | 从ServletContext中取出传入key对应的value |
| Enumeration<String> getAttributeNames() | 从ServletContext中取出所有Attribute的name |
| void removeAttribute(String name) | 移除ServletContext中指定key对应的value |

（2）访问webapp的资源

| 方法 | 作用 |
| :-: | :-: |
| String getContextPath() | 返回当前webapp的URL入口，如果不配置默认为空字符串 |
| String getInitParameter(String name) | 获取ServletContext中配置的初始化参数，即web.xml中context-param元素中配置的参数，这个元素中配置的参数是webapp范围内都有效的初始化参数 |
| Enumeration<String> getInitParameterNames() | 获取ServletContext中配置的所有初始化参数名称 |
| RequestDispather getRequestDispatcher(String path) | 返回一个用于向其他web组件转发请求的RequestDispatcher对象 |

（3）获取Servlet容器的相关信息

| 方法 | 作用 |
| :-: | :-: |
| ServletContext getContext(String uripath) | 根据参数指定的url返回当前Servlet容器中其他webapp的ServletContext对象 |
| int getMajorVersion() | 返回Servlet容器支持的Java Servlet API的主版本号 |
| int getMinorVersion() | 返回Servlet容器支持的Java Servlet API的次版本号 |
| String getServerInfo() | 返回Servlet容器的名字和版本 |

（4）访问服务器端的文件系统资源

| 方法 | 作用 |
| :-: | :-: |
| String getRealPath(String path) | 根据参数指定的虚拟路径（以webapp所在目录为根目录），返回文件系统中的一个真实的路径 |
| URL getResource(String path) | 返回一个映射到参数指定的路径的url |
| InputStream getResource(String path) | 返回一个用于读取参数指定的文件的输入流（把文件读到输入流中去） |
| String getMimeType(String file) | 返回参数指定的文件的MIME类型 |


#### 什么是ServletConfig？

Servlet容器初始化一个Servlet类型的对象时，会为这个Servlet对象创建一个ServletConfig对象。在ServletConfig对象中包含了Servlet的初始化参数信息。此外，ServletConfig对象还与ServletContext对象相关联。Servlet容器在调用Servlet对象的init(ServletConfig config) 方法时，会把ServletConfig类型的对象当作参数传递给Servlet对象。在Servlet内部提供了getServletConfig()方法来获取关联的ServletConfig对象。

##### ServletConfig有哪些作用？

| 方法 | 作用 |
| :-: | :-: |
| ServletContext getServletContext() | 获得与ServletConfig相关联的ServletConfig对象 |
| String getServletName() | 返回Servlet的名字，即web.xml中相对应的Servlet的子元素servlet-name的值。如果没有配置这个子元素，则返回Servlet类的全局限定名 |
| String getInitParameter(String name) | 根据给定的初始化参数，返回匹配的初始化参数值，即web.xml中servlet元素的子元素init-param所配置的参数 |
| Enumeration<String> getInitParameterNames() | 返回所有配置的初始化参数名称 |

#### 什么是BeanPostProcessor？

The `BeanPostProcessor` interface defines callback methods that you can implement to provide your own (or override the container's default) instantiation logic, dependency resolution logic, and so forth. If you want to implement some custom logic after the Spring container finishes instantiating, configuring, and initializing a bean, you can plug in one or more custom `BeanPostProcessor` implementations.

##### 如何控制多个BeanPostProcessor的执行顺序？

You can configure multiple `BeanPostProcessor` instances, and you can control the order in which these `BeanPostProcessor` instances execute by setting the `order` property. You can get this property only if the `BeanPostProcessor` implements the `Ordered` interface. If you write your own `BeanPostProcessor`, you should consider implementing the `Ordered` interface, too.

##### BeanPostProcessor的作用范围是什么？

`BeanPostProcessor` instances are scoped per-container. This is relevant only if you use container hierarchies. If you define a `BeanPostProcessor` in one container, it post-processes only the beans in that container. In other words, beans that are defined in one container are not post-processed by a `BeanPostProcessor` defined in another container, even if both containers are part of the same hierarchy.

##### BeanPostProcessor的执行时机是什么？

Object postProcessBeforeInitialization(Object bean, String beanName)发生在构造函数注入和setter方法注入之后，在`@PostConstruct`注解的方法、`InitializingBean`的`afterPropertiesSet()`方法、`init-method`执行之前。

Object postProcessAfterInitialization(Object bean, String beanName)发生在`@PostConstruct`注解的方法、`InitializingBean`的`afterPropertiesSet()`方法、`init-method`执行之后。

The `org.springframework.beans.factory.config.BeanPostProcessor` interface consists of exactly two callback methods. When such a class is registered as a post-processor with the container, for each bean instance that is created by the container, the post-processor gets a callback from the container both before container initialization methods (such as `InitializingBean.afterPropertiesSet()` or any declared `init` method) are called, and after any bean initialization callbacks. The post-processor can take any action with the bean instance, including ignoring the callback completely. A bean post-processor typically checks for callback interfaces, or it may wrap a bean with a proxy. Some Spring AOP infrastructure classes are implemented as bean post-processor in order to provide proxy-wrapping logic.

##### BeanPostProcessor是如何注册到IoC容器中的？

An `ApplicationContext` automatically detects any beans that are defined in the configuration metadata that implements the `BeanPostProcessor` interface. The `ApplicationContext` registers these beans as post-processors so that they can be called later, upon bean creation. Bean post-processor can be deployed in the container in the same fashion as any other beans.

Note that, when declaring a `BeanPostProcessor` by using an `@Bean` factory method on a configuration class, the return type of the factory method should be the implementation class itself or at least the `org.springframework.beans.factory.config.BeanPostProcessor` interface, clearly indicating the post-processor nature of that bean. Otherwise, the `ApplicationContext` cannot autodetect it by type before fully creating it. Since a `BeanPostProcessor` needs to be instantiated early in order to apply to the initialization of other beans in the context, this early type detection is critical.

###### 如何用编程的形式注册BeanPostProcessor到IoC容器中？

While the recommended approach for `BeanPostProcessor` registration is through `ApplicationContext` auto-detection, you can register them programmatically against a `ConfigurableBeanFactory` by using the `addBeanPostProcessor` method. This can be useful when you need to evaluate conditional logic before registration or even for copying bean post processors across contexts in a hierarchy. Note, however, that `BeanPostProcessor` instances added programmatically do not respect the `Ordered` interface. Here, it is the order of registration that dictates the order of execution. Note also that `BeanPostProcessor` instances registered programmatically are always processed before those registered through auto-detection, regardless of any explicit ordering.

```java
Resource resource = new ClassPathResource("com/repeat/read/lifecycle/spring-config.xml");
ConfigurableBeanFactory configurableBeanFactory = new DefaultListableBeanFactory();
BeanDefinitionReader bdr = new XmlBeanDefinitionReader((BeanDefinitionRegistry) configurableBeanFactory);
bdr.loadBeanDefinitions(resource);
configurableBeanFactory.addBeanPostProcessor(new BeanPostProcessorTest());
configurableBeanFactory.getBean("lifeCycleTest", LifeCycleTest.class);
```

#### BeanPostProcessor与AOP自动代理 ?????

Classes that implement the `BeanPostProcessor` interface are special and are treated differently by the container. All `BeanPostProcessor` instances and beans that they directly reference are instantiated on startup, as part of the special startup phase of the `ApplicationContext`. Next, all `BeanPostProcessor` instances are registered in a sorted fashion and applied to all further beans in the container. Because AOP auto-proxying is implemented as a `BeanPostProcessor` itself, neither `BeanPostProcessor` instances nor the beans they directly reference are eligible for auto-proxying and, thus, do not have aspects woven into them.

For any such bean, you should see an informational log message: `Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`.

If you have beans wired into your `BeanPostProcessor` by using autowiring or `@Resource` (which may fall back to autowiring), Spring might access unexpected beans when searching for type-matching dependency candidates and, therefore, making them ineligible for auto-proxying or other kinds of bean post-processing. For example, if you have a dependency annotated with `@Resource` where the field or setter name does not directly correspond to the declared name of a bean and no name attribute is used, Spring accesses other beans for matching them by type.

#### BeanPostProcessor和BeanFactoryPostProcessor的区别是什么？

`BeanPostProcessor` instances operate on bean (or object) instances. That is, the Spring IoC container instantiates a bean instance and then `BeanPostProcessor` instances do their work.

To change the actual bean definition (that is, the blueprint that defines the bean), you instead need to use a `BeanFactoryPostProcessor`.

`BeanFactoryPostProcessor` operates on the bean definition metadata. That is, the Spring IoC container lets a `BeanFactoryPostProcessor` read the configuration metadata and potentially change it before the container instantiates any beans other than `BeanFactoryPostProcessor` instances.

#### PropertySourcesPlaceholderConfigurer和PropertyOverrideConfigurer的区别是什么？

##### PropertySourcesPlaceholderConfigurer的用法

有一个配置文件database.properties。

```
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mydb
jdbc.username=root
jdbc.password=password
```

```xml
<bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
    <property name="location">
        <value>database.properties</value>
    </property>
</bean>
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

经过上述配置后，id为dataSource的配置和下述配置效果相同。

```xml
<bean id="dataSource"  class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
    <property name="username" value="root" />
    <property name="password" value="password" />
</bean>
```

##### PropertyOverrideConfigurer的用法

有一个配置文件myproperties.properties。

```
person.age=40
person.name=Stanis
```

```xml
<bean class="org.springframework.beans.factory.config.PropertyOverrideConfigurer">
    <property name="location" value="classpath:myproperties.properties" />
</bean>
<bean id="person" class="com.sample.Employee" >
       <property name="name" value="Dugan"/>
       <property name="age" value="50"/>       
</bean> 
```

经过上述配置后，IoC容器中的id为person的bean实例的`name`是`Stanis`，`age`是`40`。

#### 什么是FactoryBean？

You can implement the `org.springframework.beans.factory.FactoryBean` interface for objects that are themselves factories.

The `FactoryBean` interface is a point of pluggability into the Spring IoC container's instantiation logic. If you have complex initialization code that is better expressed in Java as opposed to a (potentially) verbose amount of XML, you can create your own `FactoryBean`, write the complex initialization inside that class, and then plug your custom `FactoryBean` into the container.

#### 如何从IoC容器中获取FactoryBean？

When you need to ask a container for an actual `FactoryBean` instance itself instead of the bean it produces, preface the bean's `id` with the ampersand symbol (`&`) when calling the `getBean()` method of the `ApplicationContext`. So, for a given `FactoryBean` with an `id` of `myBean`, invoking `getBean("myBean")` on the container returns the product of the `FactoryBean`, whereas invoking `getBean("&myBean")` returns the `FactoryBean` instance itself.

#### 注解配置和XML配置的顺序先后

Annotation injection is performed before XML injection. Thus, the XML configuration overrides the annotations for properties wired through both approaches.

#### 如何开启注解配置？

在XML配置文件中定义如下配置。

```xml
<context:annotation-config/>
```

上述XML配置向IoC容器注册了4个BeanPostProcessor，分别是`AutowireAnnotationBeanPostProcessor`, `CommonAnnotationBeanPostProcessor`, `PersistenceAnnotationBeanPostProcessor`和`RequiredAnnotationBeanPostProcessor`。

#### @Autowired注解用在setter方法和构造函数上的区别是什么？

- @Autowired注解用在setter方法上时，如果该属性是一个map，且找不到该map的value对应类型的bean，那么就会抛出异常。

- @Autowired注解用在构造函数上时，如果该属性是一个map，且找不到该map的value对应类型的bean，那么该map就会为空（不是null）。

This allows for a common implementation pattern where all dependencies can be declared in a unique multi-argument constructor - for example, declared as a single public constructor without an `@Autowired` annotation.

Only one constructor of any given bean class may declare `@Autowired` with the `required` attribute set to `true`, indicating the constructor to autowire when used as a Spring bean. As a consequence, if the `required` attribute is left at its default value `true`, only a single constructor may be annotated with `@Autowired`. If multiple constructors declare the annotation, they will all have to declare `required=false` in order to be considered as candidates for autowiring. The constructor with the greatest number of dependencies that can be satisfied by matching beans in the Spring container will be chosen. If none of the candidates can be satisfied, then a primary/default constructor (if present) will be used. If a class only declares a single constructor to begin with, it will always be used, even if not annotated. Note that an annotated constructor does not have to be public.

#### @Autowired注解的自引用问题

As of 4.3, `@Autowired` also considers self references for injection (that is, references back to the bean that is currently injected). Note that self injection is a fallback. Regular dependencies on other components always have precedence. In that sense, self references do not participate in regular candidate selection and are therefore in particular never primary. On the contrary, they always end up as lowest precedence. In practice, you should use self references as a last resort only (for example, for calling other methods on the same instance through the bean's transactional proxy). Consider factoring out the effected methods to a separate delegate bean in such a scenario. Alternatively, you can use `@Resource`, which may obtain a proxy back to the current bean by its unique name.

Trying to inject the results from `@Bean` methods on the same configuration class is effectively a self-reference scenario as well. Either lazily such references in the method signature where it is actually needed (as opposed to an autowired field in the configuration class) or declare the affected `@Bean` methods as `static`, decoupling them from the containing configuration class instance and its lifecycle. Otherwise, such beans are only considered in the fallback phase, with matching beans on other configuration classes selected as primary candidates instead (if available).

#### CustomAutowireConfigurer的作用是什么？

@Qualifier可以实现自定义修饰注解的功能。

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {}
```

为了避免每个自定义注解上都用@Qualifier去修饰，可以在Spring中提供一个CustomAutowireConfigurer的bean定义，并直接注册所有自定义注解类型。

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.Offline</value>
        </set>
    </property>
</bean>
```

#### AutowireCandidateResolver是如何确定自动注入哪个对象的？

- The `autowire-candidate` value of each bean definition （如果某个bean的定义为`autowire-candidate=false`，那么这个bean不会被自动注入到其他对象中）

- Any `default-autowire-candidates` patterns available on the `<beans/>` element

- The presence of `@Qualifier` annotations and any custom annotations registered with the `CustomAutowireConfigurer`

When multiple beans qualify as autowire candidates, the determination of a "primary" is as follows: If exactly one bean definition among the candidates has a `primary` attribute set to `true`, it is selected.

#### `@Resource`注解是根据名称查找bean的，但如果没有找到相匹配的bean，将会根据类型自动注入。

#### 在`@ComponentScan`注解中使用过滤器`includeFilters`选项时，需要禁用默认规则（`useDefaultFilters = false`）才会生效。

#### `InjectionPoint`的作用是什么？

As of Spring Framework 4.3, you may also declare a factory method parameter of type `InjectionPoint` (or its specific subclass: `DependencyDescriptor`) to access the requesting injection point that triggers the creation of the current bean. Note that this applies only to the actual creation of bean instances, not to the injection of existing instances. As a consequence, this feature makes most sense for beans of prototype scope. For other scopes, the factory method only ever sees the injection point that triggered the creation of the provided injection point metadata with semantic care in such scenarios.

#### 在`@Component`中定义的bean和`@Configuration`中定义的bean有什么区别？

The `@Bean` methods in a regular Spring component are processed differently than their counterparts inside a Spring `@Configuration` class. The difference is that `@Component` classes are not enhanced with CGLIB to intercept the invocation of methods and fields. CGLIB proxying is the means by which invoking methods or fields within `@Bean` methods in `@Configuration` classes creates bean metadata references to collaborating objects. Such methods are not invoked with normal Java semantics but rather go through the container in order to provide the ususal lifecycle management and proxying of Spring beans, even when referring to other beans through programmatic calls to `@Bean` methods. In contrast, invoking a method or field in a `@Bean` method within a plain `@Component` class has standard Java semantics, with no special CGLIB processing or other constraints applying.

##### Full `@Configuration` vs "lite" `@Bean` mode

When `@Bean` methods are declared within classes that are not annotated with `@Configuration`, they are referred to as being processed in a "lite" mode. Bean methods declared in a `@Component` or even in a plain old class are considered to be "lite", with a different primary purpose of the containing class and a `@Bean` method being a sort of bonus there. For example, service components may expose management views to the container through an additional `@Bean` method on each applicable component class. In such scenarios, `@Bean` methods are a general-purpose factory method mechanism.

Unlike full `@Configuration`, lite `@Bean` methods cannot declare inter-bean dependencies. Instead, they operate on their containing component's internal state and, optionally, on arguments that they may declare. Such a `@Bean` method should therefore not invoke other `@Bean` methods. Each such method is literally only a factory method for a particular bean reference, without any special runtime semantics. The positive side-effect here is that no CGLIB subclassing has to be applied at runtime, so there are no limitations in terms of class design (that is, the containing class may be `final` and so forth).

In common scenarios, `@Bean` methods are to be declared with `@Configuration` classes, ensuring that "full" mode is always used and that cross-method references therefore get redirected to the container's lifecycle management. This prevents the same `@Bean` method from accidentally being invoked through a regular Java call, which helps to reduce subtle bugs that can be hard to track down when operating in "lite" mode.

#### 什么时候适合将`@Bean`注解的方法标记为`static`方法？

You may declare `@Bean` methods as `static`, allowing for them to be called without creating their containing configuration class as an instance. This makes particular sense when defining post-processor beans (for example, of type `BeanFactoryPostProcessor` or `BeanPostProcessor`), since such beans get initialized early in the container lifecycle and should avoid triggering other parts of the configuration at that point.

Calls to static `@Bean` methods never get intercepted by the container, not even within `@Configuration` classes, due to technical limitations: CGLIB subclassing can override only non-static methods. As a consequence, a direct call to another `@Bean` method has standard Java semantics, resulting in an independent instance being returned straight from the factory method itself.

#### `@Bean`注解的方法有什么特殊要求？

The Java language visibility of `@Bean` methods does not have an immediate impact on the resulting bean definition in Spring's container. You can freely declare your factory methods as you see fit in non-`@Configuration` classes and also for static methods anywhere. However, regular `@Bean` methods in `@Configuration` classes need to be overridable - that is, they must not be declared as `private` or `final`.

`@Bean` methods are also discovered on base classes of a given component or configuration class, as well as on Java 8 default methods declared in interfaces implemented by the component or configuration class. This allows for a lot of flexibility in composing complex configuration arrangements, with even multiple inheritance being possible through Java 8 default methods as of Spring 4.2.

Finally, a single class may hold multiple `@Bean` methods for the same bean, as an arrangement of multiple factory methods to use depending on available dependencies at runtime. This is the same algorithm as for choosing the "greediest" constructor or factory method in other configuration scenarios: The variant with the largest number of satisfiable dependencies is picked at construction time, analogous to how the container selects between multiple `@Autowired` constructors.

#### XML配置和注解配置的本质区别是什么？

As with most annotation-based alternatives, keep in mind that the annotation metadata is bound to the class definition itself, while the use of XML allows for multiple beans of the same type to provide variations in their qualifier metadata, because that metadata is provided per-instance rather than per-class.

#### Environment是什么？

The `Environment` interface is an abstraction integrated in the container that models two key aspects of the application environment: profiles and properties.

A profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active. Beans may be assigned to a profile whether defined in XML or with annotations. The role of the `Environment` object with relation to profiles is in determining which profiles (if any) are currently active, and which profiles (if any) should be active by default.

Properties play an important role in almost all applications and may originate from a variety of sources: properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc `Properties` objects, `Map` objects, and so on. The role of the `Environment` object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them.

#### PropertySource是什么？

Spring's `Environment` abstraction provides search operations over a configurable hierarchy of property source.

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

We see a high-level way of asking Spring whether the `my-property` property is defined for the current environment. To answer this question, the `Environment` object performs a search over a set of `PropertySource` objects. A `PropertySource` is a simple abstraction over any source of key-value pairs, and Spring's `StandardEnvironment` is configured with two PropertySource objects - one representing the set of JVM system properties (`System.getProperties()`) and one representing the set of system environment variables (`System.genenv()`).

##### System.getenv()和System.getProperties()的区别是什么？

System.getenv()方法是获取指定的环境变量的值。它有两个重载方法，一个是接收参数为任意字符串，当存在指定环境变量时即返回环境变量的值，否则返回null。另一个是不接受参数，返回所有的环境变量。

System.getProperties()用于获取所有的系统相关属性，包括文件编码、操作系统名称、区域、用户名等，此属性一般由JVM自动获取，不能设置。

#### Spring提供的几个标准Event

##### ContextRefreshedEvent

Published when the `ApplicationContext` is initialized or refreshed (for example, by using the `refresh()` method on the `ConfigurableApplicationContext` interface). Here, "initialized" means that all beans are loaded, post-processor beans are detected and activated, singletons are pre-instantiated, and the `ApplicationContext` object is ready for use. As long as the context has not been closed, a refresh can be triggered multiple times, provided that the chosen `ApplicationContext` actually supports such "hot" refreshes. For example, `XmlWebApplicationContext` supports hot refreshes, but `GenericApplicationContext` does not.

##### ContextStartedEvent

Published when the `ApplicationContext` is started by using the `start()` method on the `ConfigurableApplicationContext` interface. Here, "started" means that all `Lifecycle` beans receive an explicit start signal. Typically, this signal is used to restart beans after an explicit stop, but it may also be used to start components that have not been configured for autostart (for example, components that have not already started on initialization).

##### ContextStoppedEvent

Published when the `ApplicationContext` is stopped by using the `stop()` method on the `ConfigurableApplicationContext` interface. Here, "stopped" means that all `Lifecycle` beans receive an explicit stop signal. A stopped context may be restarted through a `start()` call.

##### ContextClosedEvent

Published when the `ApplicationContext` is being closed by using the `close()` method on the `ConfigurableApplicationContext` interface or via a JVM shutdown hook. Here, "closed" means that all singleton beans will be destroyed. Once the context is closed, it reaches its end of life and cannot be refreshed or restarted.

##### RequestHandledEvent

A web-specific event telling all beans that an HTTP request has been serviced. This event is published after the request is complete. This event is only applicable to web applications that use Spring's `DispatcherServlet`.

##### ServletRequestHandledEvent

A subclass of `RequestHandledEvent` that adds Servlet-specific context information.

#### `BeanFactory`和`ApplicationContext`的区别是什么？

You should use an `ApplicationContext` unless you have a good reason for not doing so, with `GenericApplicationContext` and its subclass `AnnotationConfigApplicationContext` as the common implementations for custom bootstrapping. These are the primary entry points to Spring's core container for all common purposes: loading of configuration files, triggering a classpath scan, programmatically registering bean definitions and annotated classes, and (as of 5.0) registering functional bean definitions.

Because an `ApplicationContext` includes all the functionality of a `BeanFactory`, it is generally recommended over a plain `BeanFactory`, except for scenarios where full control over bean processing is needed. Within an `ApplicationContext` (such as the `GenericApplicationContext` implementation), several kinds of beans are detected by convention (that is, by bean name or by bean type - in particular, post-processors), while a plain `DefaultListableBeanFactory` is agnostic about any special beans.

For many extended container features, such as annotation processing and AOP proxying, the `BeanPostProcessor` extension point is essential. If you use only a plain `DefaultListableBeanFactory`, such post-processors do not get detected and ectivated by default. This situation could be confusing, because nothing is actually wrong with your bean configuration. Rather, in such a scenario, the container needs to be fully bootstrapped through additional setup.

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```



















































