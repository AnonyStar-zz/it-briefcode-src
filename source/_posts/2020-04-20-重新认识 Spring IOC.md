---
title: Spring(Ⅰ) - 重新认识 Spring IOC
date: 2019-09-27 09:25:00
top: true  推荐文章（文章是否置顶），如果 top 值为 true，则会作为首页推荐文章
categories: Spring
tags:
  - Spring
  - java
---
**摘要：** `spring` 框架是学习Java语言同学必学的框架，也是目前Java领域的标杆，在以前笔者更多的都在使用spring的层次，而缺乏对spring的认识，从这个文章开始我们将进一步深入的学习 `spring framework` 框架，学习它的设计理念与源码结构
<!-- more -->

--------

# Spring IOC 剖析

## 再品IOC与DI

* `IOC(Inversion of Control)` 控制反转：所谓控制反转，就是把原先我们代码里面需要实现的对象创 建、依赖的代码，反转给**容器**来帮忙实现。那么必然的我们需要创建一个容器，同时需要一种描述来让 容器知道需要创建的对象与对象的关系。这个描述最具体表现就是我们所看到的配置文件。

* `DI(Dependency Injection)` 依赖注入：就是指对象是被动接受依赖类而不是自己主动去找，换句话说就 是指对象不是从容器中查找它依赖的类，而是在容器实例化对象的时候主动将它依赖的类注入给它。

上面是 `ioc` 和 `DI` 的通俗理解，我们也可以用我们现有的知识来思考这两点的实现，其实两者主要还是依赖与反射机制来实现这些功能，那么我们为了提出一些关键问题，来跟着关键问题来看下具体的流程。

* 在 `spring` 中对象之间的关系如何来表现

在我们配置文件中或者javaconfig中均又相应的方式来提现

* 描述对象之间关系的文件或者信息存在哪里

可能存在于classpat、fileSystem、url或者context中，

* 对于不同的存储位置和文件格式，其实的描述是不相同的，如何做到统一解析和声明

我们可以想一下，将这些外部的信息按照模型，进行转换，在内部维护一个统一的模型对象

* 如何对这些信息进行不同的解析

根据各自的特性指定相应的策略来进行解析

## IOC 容器的核心类

### 1. BeanFactory
在 `spring` 中 `BeanFactory` 是顶层的容器接口，我们可以看出来其实 `spring` 中容器的本质就是工厂, 他有非常多的实现类，我们这里把主要的核心类图展示：

![beanFactory类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729398715-9caa768f-9951-40e3-b847-8dfafa0ee2ce.png#align=left&display=inline&height=1174&margin=%5Bobject%20Object%5D&name=beanFactory%E7%B1%BB%E5%9B%BE. png&originHeight=1174&originWidth=2858&size=1571591&status=done&style=none&width=2858)

对上图做简单的说明：

* BeanFactory 是顶层的容器接口，主要有三个子类接口 `HierarchicalBeanFactory` 、 `AutowireCapableBeanFactory` 、 `ListableBeanFactory` 

* 在继承的关系中我们可以看到都是接口和抽象类为主，多层次的封装，最终的实现类如 `DefaultListableBeanFactory` ，还有类似 `AbstractApplicationContext` 的抽象子类，在spring中这些接口都有自己特定的使用场景，对每种场景中不同对象的创建传递到转化的过程中都进行了相应的控制限制，有很强的领域划分，职责单一可扩展性极强

``` java
public interface BeanFactory {
	/**
	 * 主要勇于区分beanFactory与factoryBean，FactoryBean是spring内部生成对象的工厂即容器，
     * 在我们通过过getBean获取对象时得到的是真实对象的代理对象，如果我们要获取产生对象代理的
     * 工厂则需要加该前缀
	 */
	String FACTORY_BEAN_PREFIX = "&";
	/**
	 * 返回一个instance的实列 通过beanName
	 */
	Object getBean(String name) throws BeansException;
	/**
	 * 通过BeanName与class类型来获取容器中的对象，多层限制校验
	 */
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
    /**
	 * 通过BeanName 同时指定相应的构造函数或者工厂方法的参数列表
	 */
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
    
    /*
     * 校验是否在IOC中存在
     */
	boolean containsBean(String name);
    /*
     * 校验是单例或者原型模式
     */
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    /* 
     * 判断是IOC中bean的类型是否是typTomatch的类型
     */
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
    //获取指定bean的类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
    // 获取bean的 别名
	String[] getAliases(String name);
}
```

* 在spring中 `BeanFactory` 定义了 容器的行为，但是并不会去实现这些，顶级接口制作了高度的抽象化处理，具体的容器的创建和运行们都是交给子类来实现，所以我们要知道IOC是如何运转的需要从spring中ico的实现子类来入门，如我们读取xml配置方式时的 `ClasspathXmlApplicationContext` 或者时在注解中使用的 `AnnotationConfigApplicationContext` 等，在这些具体的实现类中有容器初始化的具体流程

* 在上面类图中 `ApplicationContext` 类是非常重要的一个接口，这是spring提供的一个高级接口，也是我们以后接触最多的容器

### 2. BeanDefinition

beandefinition 是spring中对对象关系，对象创建等一系列的定义模型，其本质其实是一个Map集合，其类图我们可以看一下：

![BeanDefinition类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729427552-9b1d53cb-4fae-4c9e-99d0-256b35ceb66f.png#align=left&display=inline&height=1427&margin=%5Bobject%20Object%5D&name=BeanDefinition%E7%B1%BB%E5%9B%BE. png&originHeight=1427&originWidth=2801&size=1823380&status=done&style=none&width=2801)

### 3. BeanDefinitionReader

在我们创建初始化容器时，也就是bean工厂时，会根据这个工厂创建相应的 BeanDefinitionReader 对象，这个reader对象是一个资源解析器，这个解析的过程是复杂的在我们后边的解析中会具体来看各自的实现

### 4. ResourceLoader

所属包 `org.springframework.core.io.ResourceLoader` ，这是spring用来进行统一资源加载的顶级接口，里面定义行为，实现让具体的子类实现，类图我们可以看一下

![ResourceLoader类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729447296-daff1742-2e8b-46c9-8a49-c95cae4eed96.png#align=left&display=inline&height=842&margin=%5Bobject%20Object%5D&name=ResourceLoader%E7%B1%BB%E5%9B%BE. png&originHeight=842&originWidth=3267&size=1229123&status=done&style=none&width=3267)

类图中展示的是 `ResourceLoader` 的核心实现， 在 `spring` 中容器也有实现该接口，关于统一资源加载的运转后期会专门说明

### 5. Resource

所属包 `org.springframework.core.io.Resource` , 该类是 `spring` 中资源加载的策略实现顶层接口，该类的每个实现类都是对某一种资源的访问策略，类图：

![resource类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729477073-e181263e-1e8c-4888-adda-ba76c2076659.png#align=left&display=inline&height=849&margin=%5Bobject%20Object%5D&name=resource%E7%B1%BB%E5%9B%BE. png&originHeight=849&originWidth=3455&size=1324564&status=done&style=none&width=3455)

## Web IOC 容器初识

我们在springMvc中很熟悉一个核心控制器 `DispatcherServlet` , 这个类做了一个集中分发和 `web` 容器初始的功能，首先我们来看一下类图

![dispatcherServlet类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729490549-ca104ba7-bab6-4d5b-a1b2-a49145c977c9.png#align=left&display=inline&height=1333&margin=%5Bobject%20Object%5D&name=dispatcherServlet%E7%B1%BB%E5%9B%BE. png&originHeight=1333&originWidth=2684&size=1519219&status=done&style=none&width=2684)

* 我们可以看到 `DispatcherServlet` 继承了 `HttpServlet` ，我们熟悉 `HttpServlet` 是属于Servlet的 ，那么它必然有个 `init()` 的初始化方法，我们通过查看，可以看到在 `HttpServletBean` 中重写了 `init` 方法

``` JAVA
/**
	 * 重写了init方法，对ServletContext初始化
	 * Map config parameters onto bean properties of this servlet, and
	 * invoke subclass initialization.
	 * @throws ServletException if bean properties are invalid (or required
	 * properties are missing), or if subclass initialization fails.
	 */
	@Override
	public final void init() throws ServletException {
		// Set bean properties from init parameters.
		//读取初始化参数  如web.xml中 init-param
		PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
		if (!pvs.isEmpty()) {
			try {
				BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
				ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
				bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
				initBeanWrapper(bw);
				bw.setPropertyValues(pvs, true);
			}
			catch (BeansException ex) {
				if (logger.isErrorEnabled()) {
					logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
				}
				throw ex;
			}
		}
		// Let subclasses do whatever initialization they like.
		//初始化容器 是让子类FrameworkServlet具体实现的
		initServletBean();
	}
```

* 我们可以看到， `init` 方法中具体的容器实例方法 `FrameworkServlet` 来实现的，我们跟进去看一下 `initialServletBean` 的具体实现

``` JAVA
/**
	 * Overridden method of {@link HttpServletBean}, invoked after any bean properties
	 * have been set. Creates this servlet's WebApplicationContext.
	 * 构建 web 上下文容器
	 */
	@Override
	protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
		if (logger.isInfoEnabled()) {
			logger.info("Initializing Servlet '" + getServletName() + "'");
		}
		long startTime = System.currentTimeMillis();
		try {
			//容器初始化
			this.webApplicationContext = initWebApplicationContext();
			initFrameworkServlet();
		}
		catch (ServletException | RuntimeException ex) {
			logger.error("Context initialization failed", ex);
			throw ex;
		}
		if (logger.isDebugEnabled()) {
			String value = this.enableLoggingRequestDetails ?
					"shown which may lead to unsafe logging of potentially sensitive data" :
					"masked to prevent unsafe logging of potentially sensitive data";
			logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
					"': request parameters and headers will be " + value);
		}
		if (logger.isInfoEnabled()) {
			logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
		}
	}
```

* 我们看到 `initWebApplicationContext()` 正是容器初始化的方法，我们继续跟进，我们现在是看容器初始化，其他暂时过掉，后面讲springmvc时在系统讲解

``` java
/**
	 *  初始化web容器  WebApplicationContext
	 * Initialize and publish the WebApplicationContext for this servlet.
	 * <p>Delegates to {@link #createWebApplicationContext} for actual creation
	 * of the context. Can be overridden in subclasses.
	 * @return the WebApplicationContext instance
	 * @see #FrameworkServlet(WebApplicationContext)
	 * @see #setContextClass
	 * @see #setContextConfigLocation
	 */
	protected WebApplicationContext initWebApplicationContext() {
		//从ServletContext根容器中获取父容器 WebApplicationContext
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		//声明子容器
		WebApplicationContext wac = null;
		//构建父子容器的关系，这里判断当前容器是否有，
		// 若存在则作为子容器来给他设置父容器rootcontext
		if (this.webApplicationContext != null) {
			// A context instance was injected at construction time -> use it
			wac = this.webApplicationContext;
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent -> set
						// the root application context (if any; may be null) as the parent
						cwac.setParent(rootContext);
					}
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		//判断子容器是否有引用，在ServletContext根容器中寻找，找到则赋值
		if (wac == null) {
			// No context instance was injected at construction time -> see if one
			// has been registered in the servlet context. If one exists, it is assumed
			// that the parent context (if any) has already been set and that the
			// user has performed any initialization such as setting the context id
			wac = findWebApplicationContext();
		}
		//若上面寻找也没找到，则这里进行容器的赋值 构建一个容器，但是这个容器并没有初始化 只是建立了引用
		if (wac == null) {
			// No context instance is defined for this servlet -> create a local one
			wac = createWebApplicationContext(rootContext);
		}
		//这里触发onRefresh方法进行容器真真初始化
		if (!this.refreshEventReceived) {
			// Either the context is not a ConfigurableApplicationContext with refresh
			// support or the context injected at construction time had already been
			// refreshed -> trigger initial onRefresh manually here.
			synchronized (this.onRefreshMonitor) {
				onRefresh(wac);
			}
		}
		if (this.publishContext) {
			// Publish the context as a servlet context attribute.
			String attrName = getServletContextAttributeName();
			getServletContext().setAttribute(attrName, wac);
		}
		return wac;
	}
```

* 我们看到真真初始化容器的方法 `onRefresh()` 方法, 跟进找到 `DispatcherServlet` 中的实现类，其中又调用了 `initStrategies()` 方法，继续进入

``` java
/**
	 * This implementation calls {@link #initStrategies}.
	 */
	@Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
	/**
	 * 初始化容器，进行springmvc的9大组件初始化
	 * Initialize the strategy objects that this servlet uses.
	 * <p>May be overridden in subclasses in order to initialize further strategy objects.
	 */
	protected void initStrategies(ApplicationContext context) {
		//多文件上传组件
		initMultipartResolver(context);
		//国际化组件，也就是本地语言环境
		initLocaleResolver(context);
		//初始化主题模板处理器
		initThemeResolver(context);
		//初始化HandMapping映射
		initHandlerMappings(context);
		//初始化HandlerAdapters参数适配器
		initHandlerAdapters(context);
		//初始化一场拦截组件
		initHandlerExceptionResolvers(context);
		//初始化视图预处理解析器，
		initRequestToViewNameTranslator(context);
		//初始化视图解析器
		initViewResolvers(context);
		//初始化FlashMap
		initFlashMapManager(context);
	}
```

## IOC容器初始化

IOC容器的初始化有多种方式，可以是配置文件也可以为 `Javaconfig` 的方式，常见的如 `ClassPathXmlApplicationContext` 

![ioc类图. png](https://cdn.nlark.com/yuque/0/2020/png/1292220/1587729520953-a356ec5f-4a13-4796-8afa-34c10fc47f47.png#align=left&display=inline&height=1558&margin=%5Bobject%20Object%5D&name=ioc%E7%B1%BB%E5%9B%BE. png&originHeight=1558&originWidth=3708&size=2257912&status=done&style=none&width=3708)

* IOC中主要过程可以概述为_定位_、_加载_、_注册_ 三个基本过程，我们常见的容器都是 `ApplicationContext` , `ResourceLoader` 是所有资源加载的基类，我们可以发现所有的IOC容器都是继承了 `BeanFactory` ，这也说明了所有的容器本质上都是一个bean工厂
* 我们可以通过下面的代码获取容器

``` JAVA
ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext(CONTEXT_WILDCARD);
```

那么在这个创建容器的内部具体是如何构建加载容器的，我们可以进入看一下

``` java
//调用的构造函数
	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}
```

这里有调用的一个构造函数, 这个才是真真执行的过程，我们发现内部执行力 `refresh()` 方法

``` java
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {
		//调用父类的构造函数进行资源加载设置
		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
```

* 大家可以自己看一下，其实像 `AnnotationConfigApplicationContext` 、 `FileSystemXmlApplicationContext` 、 `XmlWebApplicationContext` 这些类都调用了 `refresh()` 方法，这个方法是他们父类 `AbstractApplicationContext` 实现的 ，这里应用的了装饰器模式策略模式
