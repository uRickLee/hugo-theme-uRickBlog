---
title: Spring注解驱动
date: 2020-01-06 22:44:57
image: https://gitee.com/uRick/oss/raw/master/blog/Spring封面图.png
description: Spring配置Bean有多种形式，一种是XML方式，一种是@Configuration注解的方式，表示是一个配置类（同XML功能相同），作为Bean的载体。
categories:
 - Spring
 - Technology
tags:
 - Spring
---

## 1.1. Spring基础注解
### 1.1.1. @Configuration
>Spring配置Bean有多种形式，一种是XML方式，一种是@Configuration注解的方式，表示是一个配置类（同XML功能相同），作为Bean的载体。

```java
package org.springframework.context.annotation;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Component;
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
/**
* 同@Component注解功能一样，只是语言不同
*/
public @interface Configuration {
    // 组件名称（bean的名称）
    @AliasFor(annotation= Component.class)
    String value() default "";
}
```

### 1.1.2. @ComponentScan
>组件扫描注解，通过配置包、类、过滤器来扫描指定的类到spring上下文容器中，该注解在jdk8以后可以在同一个类上注解多个，可以通过basePackages、excludeFilters、includeFilters过滤注解,其中value同basePackages、Classes相当于一个属性，也可以自定义过滤方式如：

` @ComponentScan(basePackages= {"com.bdr.*"},excludeFilters = { @Filter(type = FilterType.ANNOTATION, classes = Controller.class) }) `
`@ComponentScan(value = "com.bdr.*", excludeFilters = {
		@Filter(type = FilterType.CUSTOM, value = MyTypeFilter.class) }, useDefaultFilters = false)`

```java
public class MyTypeFilter implements TypeFilter {
	/**
	 * 自定义匹配模式,使用自定义的的过滤器，需要设置@Filter type = FilterType.CUSTOM、 属性useDefaultFilters = false
	 */
	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
			throws IOException {
		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
		String[] memberClassNames = annotationMetadata.getMemberClassNames();
		for (String v: memberClassNames) {
		System.out.println("memberClassNames："+v);
		}
		return false;
	}
}
```


### 1.1.3. @Scope
> 常用于bean作用域于注解，定义Spring容器创建实例的方式。

```java
/**
 * prototype：多实例，每次获取bean都会从容器中创建一个新的实例
 * singleton：单例，容器启动就创建一个实例
 * request：一次请求创建一个实例
 * session：一个会话创建一个实例
 * @return
 */
@Bean
@Scope("prototype")
public Employee myBean() {
	Employee employee = new Employee();
	return employee;
}
```

### 1.1.4. @Lazy
>用于定义容器创建实例的时机（懒加载机制），容器创建时不实例化bean，当使用到该bean时实例化bean，该注解默认为value=true

```java
@Bean
@Lazy
@Scope("prototype")
public Employee myBean() {
	Employee employee = new Employee();
	return employee;
}
```

### 1.1.5. @Import
>用于快速向Spring容器添加Bean，该注解支持Configuration、ImportSelector、ImportBeanDefinitionRegistrar自定义导入方式和导入的类；@Import 导入时的是全类名。

- **ImportSelector**
`@Import(value = { Depart.class, MyImportSelector.class })`

```java
public class MyImportSelector implements ImportSelector{

	/**
	 * 这里的返回值，不要返回null,返回一个空数组也可以
	 * 返回的数组就是，对应需要导入的包，通过ImportSelector 自定导入需要的包
	 */
	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
		System.out.println("MyImportSelector>>>>: "+importingClassMetadata.getSuperClassName());
		System.out.println("MyImportSelector>>>>: "+importingClassMetadata.getEnclosingClassName());
		//  返回指定全类名的包，导入该指定包
		return new String[] {"com.bdr.bean.Depart4"};
	}
}
```
- **ImportBeanDefinitionRegistrar**

`@Import(value = { Depart.class, MyImportBeanDefinitionRegistrar.class })`


```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * 通过ImportBeanDefinitionRegistrar 导入指定的bean
	 */
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
// importingClassMetadata 获取注解信息，BeanDefinitionRegistry bean注入操作。
		boolean containsBeanDefinition = registry.containsBeanDefinition("myWindowBean");
		if(containsBeanDefinition) {
		RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Depart4.class);
			registry.registerBeanDefinition("depart4444", rootBeanDefinition);
		}
	}
}
```

- **FactoryBean**

```java
// MyFactoryBean
public class MyFactoryBean implements FactoryBean<Depart5> {
/**
 * 返回实例
 */
public Depart5 getObject() throws Exception {
	return new Depart5();
}

/**
 * 指定类型
 */
public Class<?> getObjectType() {
	return Depart5.class;
}

/**
 * 通过FactoryBean 导入bean
 * @return
 */
@Bean
public MyFactoryBean myFactoryBean() {
	return new MyFactoryBean();
}

@Test
public void getConfig() {
	AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
	String[] beans = context.getBeanDefinitionNames();
	for (String v : beans) {
		System.out.println(v);
	}
	// 获取MyFacoryBean
	// 通过myFactoryBean获取到的是FactoryBean 创建的实例bean
	// 若需要获取工厂bean类型，需要加上前缀&
	Object bean = context.getBean("&myFactoryBean");
	System.out.println("bean:::: "+bean);
}
```

### 1.1.6. @Bean
>向容器中注入Bean，该注解可以作用于一个方法上、注解上。通过该注解可以指定注入bean的注入方式(BY_NAME,BY_TYPE)、bean名称、bean的initMethod和destroyMethod，（这个方法可以写在需要注入的Bean中）

```java
public class Employee {
	private String name;
	private String no;

	public String getNo() {
		return no;
	}

	public void setNo(String no) {
		this.no = no;
	}

	public Employee() {
		System.out.println("我被创建了。。。。。。。。。。");
	}

	public void init() {
		System.out.println("我的Bean 正在初始化。。。。。");
	}

	public void destory() {
		System.out.println("我的Bean 正在销毁。。。。。");
	}

@Bean(initMethod = "init", destroyMethod = "destory")
	@Scope("prototype")
	public Employee myBean() {
		Employee employee = new Employee();
		return employee;
	}

@Test
	public void getConfigOfLifeCycle() {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig2.class);
		String[] beans = context.getBeanDefinitionNames();
		for (String v : beans) {
			System.out.println(v);
		}
		//context.getBean("myBean");
	}
```

### 1.1.7. @Value/@PropertySource/@ImportResource/@ConfigurationProperties
> 使用这几个注解需要使用`@Component` 注解（标记是Spring的组件）
1. **@Value：** 可以对bean属性赋值，支持SqEL表达式：*#{}-在里面写SqEL表达式，${}-获取属性文件中的值*  
2. **@PropertySource：** 加载指定的配置文件 `@PropertySource(value = {"classpath:person.properties"})`
3. **@ConfigurationProperties：** 将本类中的所有属性和配置文件中相关的配置进行绑定；prefix = "person"：配置文件中哪个下面的所有属性进行一一映射；`ConfigurationProperties(prefix = "person")`默认从全局配置文件中获取值；
4. **@ImportResource：** 导入Spring的配置文件，让配置文件里面的内容生效；若在Spring Boot里想让让xml配置文件生效，可以使用`@ImportResource(locations = {"classpath:beans.xml"})`方式加载进来，然后让配置生效。

### 1.1.8. 自动装配
1. **@Autowired：** 自动注入Bean，①优先按照bean的类型匹配注入，②若有多个类型的bean，则根据bean的名称去匹配；③该注解可以标在构造器、参数、方法、属性上，并且都是从容器中获取参数组件的值；④标注在方法上，参数会从容器中获取；默认不写@Autowired效果是一样的，都能实现自动装配；⑤标在构造器上：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取。

*详情可参考：[https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-annotation-config](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-annotation-config)*

```java
//默认加在ioc容器中的组件，容器启动会调用无参构造器创建对象，再进行初始化赋值等操作
@Component
public class Boss {
	private Car car;
	//构造器要用的组件，都是从容器中获取
	// 如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
	public Boss(Car car){
		this.car = car;
		System.out.println("Boss...有参构造器");
	}
	public Car getCar() {
		return car;
	}
	//@Autowired 
	//标注在方法，Spring容器创建当前对象，就会调用方法，完成赋值；
	//方法使用的参数，自定义类型的值从ioc容器中获取
	public void setCar(Car car) {
		this.car = car;
	}
	@Override
	public String toString() {
		return "Boss [car=" + car + "]";
	}
}
```

2. **@Qualifer：** 通过bean名称指定注入bean，一般配合@Autowired使用。
3. **@Primary：** 指定首选需要注入的Bean。
4. Spring还支持使用 **@Resource** (JSR250)和 **@Inject** (JSR330)[java规范的注解]
    - **@Resource：** 可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的； 不支持@Primary、@Autowired（reqiured=false）;
    - **@Inject：** 需要导入javax.inject包，和Autowired的功能一样。没有required=false的功能；

   **注：** *@Autowired是Spring定义的， @Resource、@Inject都是java规范。*

5. **Spring Aware**
>Spring Aware 是属于Spring内部调用的方式。若自定义实现Spring Aware相关接口，则bean会与spring耦合在一起（可以别容器管理），可以使用Spring底层资源。

|  接口名称  |  说明    |   备注   |
| ---- | ---- | ---- |
|  BeanNameAware    |  获取容器中bean的名称    |      |
|  BeanFactoryAware    |  获取当前的BeanFactory，调用容器中的服务  |      |
|  ApplicationContextAware*    |  获取spring中的所有服务    |      |
|  MessageSourceAware    |  获取messages source 可以获取相关的文本信息    |      |
|  ApplicationEvenPublisherAware    |  应用时间发布器，可以发布时间    |      |
|  ResourceLoaderAware    |  获取资源信息，可以获取外部资源    |      |

### 1.1.9. @Profile
> spring 提供开发环境、开发部署切换的一个属性

### 1.1.10. @Conditional
>满足一定条件，才向上下文容器中注入bean，在Spring中有许多拓展@Conditional 的注解，value为org.springframework.context.annotation.Condition 的实现类，如：

```java
    @Bean
	@Conditional(WindowsConditional.class)
	public Employee myWindowBean() {
		Employee employee = new Employee();
		return employee;
	}
    /**
     * 实现Condition接口完成bean条件注入
     * @author Rick
     *
     */
public class WindowsConditional implements Condition {
// 返回true 时注入
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
	// ConfigurableListableBeanFactory 可以实现注入bean，获取bean，对bean的操作
	ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
	System.out.println("==============WindowsConditional================");
	String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
	for (String beans : beanDefinitionNames) {
		System.out.println("获取到beans：" + beans);
	}
	System.out.println("==============================");
	ClassLoader classLoader = context.getClassLoader();
	System.out.println("获取到ClassLoader：" + classLoader.toString());
	System.out.println("==============================");
	Environment environment = context.getEnvironment();
	System.out.println("获取操作系统环境：" + environment.getProperty("os.name"));
	System.out.println("==============================");
	BeanDefinitionRegistry registry = context.getRegistry();
	registry.registerAlias("myBean", "employeeRegistryWindows");
	String[] registrybeans = registry.getBeanDefinitionNames();
	for (String registrybean : registrybeans) {
		System.out.println("获取注入的bean：" + registrybean);
	}
	ResourceLoader resourceLoader = context.getResourceLoader();
	System.out.println("==============================");
	// 获取到注解参数信息
	MultiValueMap<String, Object> allAnnotationAttributes = metadata
			.getAllAnnotationAttributes("org.springframework.context.annotation.Bean");
	for (String key : allAnnotationAttributes.keySet()) {
		System.out.println("获取注解：" + allAnnotationAttributes.get(key));
	}
	// 若在windows系统上则注入该bean
	String sysOs = environment.getProperty("os.name");
	if (sysOs.contains("Windows")) {
		return true;
	}
	return false;
}
```


**作用：** *必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。*  

**常见Conditional扩展注解：**

| @Conditional扩展注解                | 作用（判断是否满足当前指定条件）               |
| ------------------------------- | ------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                |
| @ConditionalOnBean              | 容器中存在指定Bean；                   |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                  |
| @ConditionalOnExpression        | 满足SpEL表达式指定                    |
| @ConditionalOnClass             | 系统中有指定的类                       |
| @ConditionalOnMissingClass      | 系统中没有指定的类                      |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                 |
| @ConditionalOnWebApplication    | 当前是web环境                       |
| @ConditionalOnNotWebApplication | 当前不是web环境                      |
| @ConditionalOnJndi              | JNDI存在指定项                      |

### 1.1.11. @RequestMapping
> 映射URL请求到某个方法上或者Controller上
> 属性说明 &#10148;  
> **value：** 请求的URL的路径，支持U也模板、正则表达式,支持Ant风格。  
> **method：** HTTP请求方法，有GET、POST、PUT等。  
>** consumes：** 允许的媒体类型（MediaTypes），如consumes＝”application/ison”，对应于请求的HTTP的Content-Type。   
> **produces：** 相应的媒体类型，如produces＝”application/json”，对应于HTTP的Accept宇段。  
> **params：** 请求的参数，如params＝”action=update”。  
> **headers：** 请求的HTTP头的值，如headers＝”myHeader=myValue”。  

```
// 简化的@RequestMapping,RESTFul
@GetMapping;
@PostMapping;
@PutMapping;
@DeleteMapping;
@PatchMapping
```

### 1.1.12. @PathVariable
> 变量占位符，可以将 URL 中的值映射到方法参数中。

```java
// 如果映射名称不一致需要制定@PathVariable("userId")
@GetMapping(path="/{userId}{type}/get.do")
@ResponseBody
public User getUser(@PathVariable("userId") Long id, @PathVariable Integer type) {
}
```

### 1.1.13. @ModeIAttribute
> 注解ModelAttribute通常作用在Controller的某个方法上，此方法会首先被调用，井将方法结果作为Model的属性，然后再调用对应的Controller处理方法。

### 1.1.14. @RequsetBody
> 接受Json格式请求，意味着请求的 HTTP 消息体的内容是一个JSON

### 1.1.15. @Controller/@RestController
> 定义个springMVC 控制器，RestController是@Controller和@ResponseBody注解的组合。

### 1.1.16. @ControllerAdvice/@RestControllerAdvice
> ControllerAdvice 注解相当于是Controller层的全局配置，在标注注解ControllerAdvice 的配置可以所用所有的Controller下的每个请求中，常常用于处理全局异常，在该注解配置中可以包含@ExceptionHandler、@InitBinder和@ModelAttribute。  
> @RestControllerAdvice 同比@Controller/@RestController ，由@ControllerAdvice和@ResponseBody组成。

```java
@ControllerAdvice
public class MyControllerAdvice {

    /**
     * 应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器
     * @param binder
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {}

    /**
     * 把值绑定到Model中，使全局@RequestMapping可以获取到该值
     * @param model
     */
    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("author", "Magical Sam");
    }

    /**
     * 全局异常捕捉处理
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public Map errorHandler(Exception ex) {
        Map map = new HashMap();
        map.put("code", 100);
        map.put("msg", ex.getMessage());
        return map;
    }

}
```

## 1.2. Aop注解
### 1.2.1. 简介

|   注解   |  说明    |   备注   |
| ---- | ---- | ---- |
|   @Before   |  在目标方法执行之前通知    |      |
|   @After   |  在目标方法执行之后通知     |      |
|   @AfterRturning    |   在目标方法执行正常返回后通知   |      |
|   @AfterThrowing    |  在目标方法执行出现异常后通知    |      |
|   @Around   |  环绕通知    |      |
|   @PointCut |  定义切点    |      |
|   @EnableAspectJAutoProxy |  开启切面支持 |      |

### 1.2.2. 步骤

```txt
1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
3）、开启基于注解的aop模式；@EnableAspectJAutoProxy
```

### 1.2.3. 代码
```java
package com.bdr.bean;
import java.util.Arrays;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
// 定义切面
@Aspect
public class AopDefinition {

	// 定义切点,也就是切入到那个目标类、目标方法上，切点名就是方法名
	// 切点表达式：作用该类下所有的公共方法，相关切面表达式查看官方文档
	@Pointcut("execution(public int com.bdr.bean.AopPointCutTest.*(..))")
	public void myPointCut() {
	}

	// @Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）,上面的切点直接引用即可
	// 引入JoinPoint 用于获取方法执行参数信息
	@Before("myPointCut()")
	public void logStart(JoinPoint joinPoint) {
		Object[] args = joinPoint.getArgs();
		System.out.println("" + joinPoint.getSignature().getName() + "运行。。。@Before:参数列表是：{" + Arrays.asList(args) + "}");
	}

	@After("myPointCut()")
	public void logEnd(JoinPoint joinPoint) {
		System.out.println("" + joinPoint.getSignature().getName() + "结束。。。@After");
	}

	// JoinPoint一定要出现在参数表的第一位
	@AfterReturning(value = "myPointCut()", returning = "result")
	public void logReturn(JoinPoint joinPoint, Object result) {
		System.out.println("" + joinPoint.getSignature().getName() + "正常返回。。。@AfterReturning:运行结果：{" + result + "}");
	}

	@AfterThrowing(value = "myPointCut()", throwing = "exception")
	public void logException(JoinPoint joinPoint, Exception exception) {
		System.out.println("" + joinPoint.getSignature().getName() + "异常。。。异常信息：{" + exception + "}");
	}
}

```

```java
package com.bdr.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import com.bdr.bean.AopDefinition;
import com.bdr.bean.AopPointCutTest;

/**
 * Aop注解学习配置类
 * @author Rick
 * 配置类
 */
// @EnableAspectJAutoProxy 开启切面注解的支持
@EnableAspectJAutoProxy
@Configuration
public class MainAopConfig {

	@Bean
	public AopPointCutTest aopPointCutTest () {
		return new AopPointCutTest();
	}
	@Bean
	public AopDefinition aopDefinition () {
		return new AopDefinition();
	}
}

```

```java
// 配置加载测试类
@Test
	public void getAopConfigTest(){
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainAopConfig.class);
			AopPointCutTest bean = context.getBean(AopPointCutTest.class);
			int sum = bean.sum(1, 3);
			System.out.println(sum);
	}
```

### 1.2.4. @EnableAspectJAutoProxy 原理
1. @EnableAspectJAutoProxy 开启AOP功能;
2. @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator;
3. AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
4. 容器的创建流程：
    - registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象；
    - finishBeanFactoryInitialization（）初始化剩下的单实例bean；
      1. 创建业务逻辑组件和切面组件；
      2. AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程；
      3. 组件创建完之后，判断组件是否需要增强；
5. 执行目标方法：
    - 代理对象执行目标方法；
	- CglibAopProxy.intercept()；
    	 1. 得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）；
       	 2. 利用拦截器的链式机制，依次进入每一个拦截器进行执行；
      	 3. 效果：
    		正常执行：前置通知-->目标方法->后置通知->返回通知
        	出现异常：前置通知->目标方法->后置通知->异常通知

## 1.3. 事务
1. **@Transactional**
> 除了基于xml的事务配置声明方法外，还可以使用基于注释的方法，当然若使用@Transactional可以是xml方式如下代码，亦可以使用@EnableTransactionManagement注解开启事务。

```xml
<tx:annotation-driven />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
```

> @Transactional作用于需要实现事务机制方法上，而且必须是public类型方法；若@Transactional 注解添加到类级别上，表示所有该类的公共方法都配置相同的事务属性信息；当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务，即方法级别的事务属性信息会覆盖类级别的相关配置信息。

@Transaction是基于AOP代理方式实现的，通过代理来实现目标方法的拦截，实现事务操作。*可参考：[详情](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)*

*@Transactional属性说明：*


```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
    // 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。
	@AliasFor("transactionManager")
	String value() default "";
	@AliasFor("value")
	String transactionManager() default "";
	// 事务的传播行为，默认值为 REQUIRED
	Propagation propagation() default Propagation.REQUIRED;
	// 事务的隔离度，默认值采用 DEFAULT
	Isolation isolation() default Isolation.DEFAULT;
	// 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
	// 	指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。
	boolean readOnly() default false;
	// 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。
	Class<? extends Throwable>[] rollbackFor() default {};
	// 指定那些异常类回滚事务（需要是Throwable的子类）
	String[] rollbackForClassName() default {};
	// 抛出noRollbackFor指定的异常类型，不回滚事务。
	Class<? extends Throwable>[] noRollbackFor() default {};
	// 指定那些类异常类型不回滚事务（需要是Throwable的子类）
	String[] noRollbackForClassName() default {};
}
```

```java
@Transactional(rollbackFor=Exception.class)
@Service(value ="employeeService")
public class EmployeeService
{
…………
}
```

2. 事务的隔离级别
> 事务的隔离级别是为了解决多事务并发情况下保证数据一致性问题的，数据库隔离级别有：[详见MySQL基础](../../数据库/MySQL基础.md#1.3.7)
3. 事务的传播特性
> 事务传播是Spring中方法之间调用事务采取的策略；如当执行一个批处理事务时，当批处理事务中的单个独立事务执行失败，不需要整个批处理事务回滚，就可以采用事务传播的策略实现这种机制。

![](https://gitee.com/uRick/IMG/raw/master/1580647047_20190227134046274_20159.png)

```java
public enum Propagation {
    /**
     * 默认传播行为，如果当前存在事务，就沿用当前事务
     * 去否则新建一个事务运行子方法
     */
    REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

    /**
     * 如果当前存在事务，就沿用当前事务
     * 如果不存在，则继续采用无事务的方式运行子方法
     */
    SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

    /**
     * 必须使用事务，如果当前没有事务，则会抛出异常，
     * 如果存在当前事务 就沿用当前事务
     */
    MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

    /**
     * 无论当前事务是否存在，都会创建新事务运行方法
     * 新事务可以拥有新的锁和隔离级别等特性，与当前事务相互独立
     */
    REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

    /**
     * 不支持事务，当前存在事务时，将挂起事务，运行方法
     */
    NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

    /**
     * 不支持事务，如果当前方法存在事务，则抛出异常，否则继续使用无事务机制运行
     */
    NEVER(TransactionDefinition.PROPAGATION_NEVER),

    /**
     * 在当前方法调用子方法时，如果子方法发生异常，
     * 只因滚子方法执行过的 SQL ，而不回滚当前方法的事务
     */
    NESTED(TransactionDefinition.PROPAGATION_NESTED);

    private final int value;

    Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```
