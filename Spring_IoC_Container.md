#### 什么是IoC？

Inversion of Control (IoC) is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

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






























