# Spring注解驱动开发

> 学习注解开发之前，先学习Spring5开发

Bean 的生命周期概括起来就是 **4 个阶段**：

- 实例化（Instantiation）
- 属性赋值（Populate）
- 初始化（Initialization）
- 销毁（Destruction）



![img](springAnnotatioin.assets/v2-a2cb36aabe9b6b044ade2a4f5bcaa759_720w.jpg)

![preview](springAnnotatioin.assets/v2-14dd69d3b86097a8ce5178c86276627d_r.jpg)

## 参考

[如何记忆 Spring Bean 的生命周期](https://juejin.cn/post/6844904065457979405)

[三问Spring事务：解决什么问题？如何解决？存在什么问题？](https://juejin.cn/post/6844904190385324045)

[聊聊spring的那些扩展机制](https://juejin.cn/post/6844903682673229831#heading-6)

[Spring 工厂方法与FactoryBean(实例化Bean)](https://blog.csdn.net/w_linux/article/details/80063062)

[Spring-Bean生命周期](https://zhuanlan.zhihu.com/p/158468104)

[Spring夺命连环10问](https://mp.weixin.qq.com/s/Xbua0iCg-QaWbq54EZN7vA)



# 1.容器 

**导入jar包**

```xml
//Spring核心包 包含bean
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```

## 1.组件注册

### 1. @configuration  @Bean注解

> @configuration  告诉Spring这是一个配置类
>
> @Bean   给容器中注册一个Bean

#### 1. xml配置开发

```xml
<bean id="person" class="com.atguigu.bean.Person"  scope="prototype" >
    <property name="age" value="10"></property>
    <property name="name" value="zhangsan"></property>
</bean>
```

 测试类

```java
public class MainTest {
	
	@SuppressWarnings("resource")
	public static void main(String[] args) {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
		Person bean = (Person) applicationContext.getBean("person");
		System.out.println(bean);
    }
}
```



#### 2.注解开发

配置类【相当于配置文件】

```java
//配置类==配置文件
@Configuration  //告诉Spring这是一个配置类
public class MainConfig {
	
	//给容器中注册一个Bean;类型为返回值的类型，id默认是用方法名作为id
    //也可以改名，命名为“person”
	@Bean("person")
	public Person person01(){
		return new Person("lisi", 20);
	}
}

```

测试类  

```java
public class MainTest {
	
	@SuppressWarnings("resource")
	public static void main(String[] args) {		
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
		Person bean = applicationContext.getBean(Person.class);
		System.out.println(bean);
		
		String[] namesForType = applicationContext.getBeanNamesForType(Person.class);
		for (String name : namesForType) {
			System.out.println(name);
		}	
	}
}

```



### 2.@ComponentScan注解

> `扫描 包下标注了@Controller、@Service、@Repository，@Component的注解，将其加入容器中`
>
> //@ComponentScan  value:指定要扫描的包
> //excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
> //includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
> //FilterType.ANNOTATION：按照注解
> //FilterType.ASSIGNABLE_TYPE：按照给定的类型；
> //FilterType.ASPECTJ：使用ASPECTJ表达式
> //FilterType.REGEX：使用正则指定
> //FilterType.CUSTOM：使用自定义规则

```xml
<!--包扫描、只要标注了@Controller、@Service、@Repository，@Component 这4个注解，都会被加入容器中-->
<context:component-scan base-package="com.atguigu" use-default-filters="false"></context:component-scan>
```

```java
@ComponentScan("com.xsbl")  //value指定要扫描的包
public class MainConfig {
```

```java
ComponentScan.Filter[] includeFilters() default {};

ComponentScan.Filter[] excludeFilters() default {};

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }

```

#### 1.excludeFilters 

```java
//1.按照类型 排除某些注解
//2.按照类名 排除某些注解
//指定扫描的时候按照什么规则排除那些组件  
//不能排除主配置类的注解
@ComponentScan(value = "com.xsbl",excludeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Repository.class}),
    @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = BookService.class)
}
```

#### 2.includeFilters  

**`注意：`**  useDefaultFilters = false 禁用原来的默认扫描规则

```
@ComponentScan(value = "com.xsbl",includeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class})
},useDefaultFilters = false
)
```



#### 3.@ComponentScans 

定义多个扫描策略

```json
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScans {
	ComponentScan[] value();
}


@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)  # java 8 新特性 可重复注解 @Repeatable
public @interface ComponentScan {
```



```java
@ComponentScans(
    value = {
        @ComponentScan(value = "com.xsbl",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes =BookService.class)
        }
        ),
        @ComponentScan(value = "com.xsbl",includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Repository.class})
        },useDefaultFilters = false
        )
    }
)
```



#### 4.自定义过滤规则 FilterType.CUSTOM

```json
/** Filter candidates using a given custom
* {@link org.springframework.core.type.filter.TypeFilter} implementation.
*/
CUSTOM
```

```json
/**
 * 自定义注解 类MyTypeFilter
 */
@Configuration
@ComponentScans(
    value = {
        @ComponentScan(basePackages = "com.xsbl",includeFilters = {
            @ComponentScan.Filter(type = FilterType.CUSTOM,classes = {MyTypeFilter.class})
        },useDefaultFilters = false)
    }
)
public class MainConfig {
    
```

```json
public class MyTypeFilter implements TypeFilter {
    /**
     * @param metadataReader 读取到的当前正在扫描的类的信息
     * @param metadataReaderFactory 可以获取到其它任何类的信息
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取当前类的注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前正在扫描的类的信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源(类的路径)
        Resource resource = metadataReader.getResource();

        //URL url = resource.getURL();
        //System.out.println("路径地址：：："+url);

        String className = classMetadata.getClassName();
        System.out.println("扫描到的类信息--->"+className);

        /**
         * 类名中包含“er”字符，就匹配，并且将其加入容器中
         */
        if (className.contains("er")) {
            return true;
        }
        return false;
    }
}
```

测试方法

```json
@Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : beanDefinitionNames) {
            System.out.println(name);
        }
    }
```



### 3.@Scope 和@Lazy 注解

>  @Scope:调整作用域
>
> @Lazy：懒加载作用：`容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化`；

```json
@Configuration
public class MainConfig2 {
	
	//默认是单实例的
	/**
	 * ConfigurableBeanFactory#SCOPE_PROTOTYPE    
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON  
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST  request
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION	 sesssion
	 * @return\
	 * @Scope:调整作用域
	 * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
	 * 					每次获取[getBean]的时候才会调用方法创建对象；
	 * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
	 * 			以后每次获取就是直接从容器（map.get()）中拿，
	 * request：同一次请求创建一个实例 
	 * session：同一个session创建一个实例
	 * 
	 * 懒加载：
	 * 		单实例bean：默认在容器启动的时候创建对象；
	 * 		懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
	 * 
	 */
//	@Scope("prototype")
	@Lazy
	@Bean("person")
	public Person person(){
		System.out.println("给容器中添加Person....");
		return new Person("张三", 25);
	}
}
	
```



### 4.@Conditional注解 【重点】

> 按照条件判断给容器中注册组件

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition}s that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```

#### 4.1 @Conditional 放在方法上：

```json
@Configuration
public class MainConfig2 {

    @Bean
    public Person person(){
        return new Person("lisi",24);
    }
    /**
     * @Conditional({Condition}) : 按照一定的条件进行判断，满足条件给容器中注册bean
     * 如果系统是Windows，给容器中注册（"bill"）
     * 如果是linux系统，给容器中注册（"linus")
     */
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01(){
        return new Person("Bill Gates", 63);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02(){
        return new Person("linus", 48);
    }
}

/**
 * 判断是否是Linux系统
 */
public class LinuxCondition implements Condition {
    /**
     * @param context 判断条件能使用的上下文（环境）
     * @param metadata 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //TODO 是否是linux系统
        //1、能获取到ioc使用的beanfactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2、获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3、获取当前环境信息
        Environment environment = context.getEnvironment();
        //4、获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        String property = environment.getProperty("os.name");

        //可以判断容器中的bean注册情况，也可以给容器中注册bean
        boolean person = registry.containsBeanDefinition("person");

        if (property.contains("linux")){
            return true;
        }
        return false;
    }
}
```

#### 4.1 @Conditional 放在类上：

```json
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
@Configuration
public class MainConfig2 {
```

```json
# 测试类  设置VM options 为-Dos.name=linux
	@Test
    public void test02(){
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        String[] names = applicationContext.getBeanNamesForType(Person.class);

        Environment environment = applicationContext.getEnvironment();
        //动态的获取环境变量的值：Windows 7
        System.out.println("操作系统："+environment.getProperty("os.name"));
        for (String name : names) {
            System.out.println(name);
        }
    }
```



### 5.@Import注解 【重点】

```json
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * 可以使用 ImportSelector
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```



```json
@Configuration
//@Import导入组件，id默认是组件的全类名【com.xsbl.bean.Color】
@Import({Color.class, Red.class})
public class MainConfig2 {
```

#### 5.1 ` ImportSelector`

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

```json
@Configuration
//类中组件统一设置，满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional(WindowsCondition.class)
//@Import导入组件，id默认是组件的全类名【com.xsbl.bean.Color】
//@Import({Color.class, Red.class})
@Import({MyImportSelector.class})
public class MainConfig2 {

```

```json
//自定义逻辑,返回需要导入的组件
public class MyImportSelector implements ImportSelector {

    /**
     * 返回值，就是导入到容器中的组件全类名
     * AnnotationMetadata:当前标注@Import注解的类的所有注解信息
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        /**
         * 标注有@Import类的注解信息：
         * className: com.xsbl.config.MainConfig2
         * annotationTypes: [org.springframework.context.annotation.Configuration,
         * org.springframework.context.annotation.Conditional,
         * org.springframework.context.annotation.Import]
         */
        String className = importingClassMetadata.getClassName();
        Set<String> annotationTypes = importingClassMetadata.getAnnotationTypes();
        System.out.println("className:"+className+"\n annotationTypes:"+annotationTypes);

        return new String[]{"com.xsbl.bean.Blue","com.xsbl.bean.Yellow"};
    }
}
```

#### 5.2 ImportBeanDefinitionRegistrar

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```



使用：@Import({MyImportBeanDefinitionRegister.class})

```json
@Configuration
//类中组件统一设置，满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional(WindowsCondition.class)
//@Import导入组件，id默认是组件的全类名【com.xsbl.bean.Color】
//@Import({Color.class, Red.class})
@Import({Red.class, MyImportSelector.class, MyImportBeanDefinitionRegister.class})
public class MainConfig2 {
```



```json
public class MyImportBeanDefinitionRegister implements ImportBeanDefinitionRegistrar {
    /**
     * AnnotationMetadata：当前类的注解信息
     * BeanDefinitionRegistry:BeanDefinition注册类；
     * 		把所有需要添加到容器中的bean；调用
     * 		BeanDefinitionRegistry.registerBeanDefinition手工注册进来
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean definition = registry.containsBeanDefinition("com.xsbl.bean.Red");
        boolean definition1 = registry.containsBeanDefinition("com.xsbl.bean.Blue");
        if (definition && definition1) {
            //指定Bean定义信息;(Bean的类型，bean。。。)
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个Bean,指定bean名
            registry.registerBeanDefinition("rainBow",beanDefinition);
        }
    }
}
```



### 6.FactoryBean   

> 本质是一个能生产对象或者修饰对象的工厂Bean
>
> ##### 扩展：XML方式中有三种方式来实例化bean
>
> - 反射模式
> - 工厂方法模式
> - FactoryBean模式

```json
/**
 * 创建一个Spring定义的FactoryBean
 * @Author: dengyanwei
 * @CreateDate: 2020/12/12 23:31
 */
public class ColorFactoryBean implements FactoryBean<Color> {
    //该方法返回该FactoryBean“生产”的对象
    //我们需要实现该方法以给出自己对象实例化逻辑
    //返回一个Color对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    //是单例？
    //true：这个bean是单实例，在容器中保存一份
    //false：多实例，每次获取都会创建一个新的bean；
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

将这个类注入

```json
@Configuration
public class MainConfig2 {
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

测试

```json
@Test
    public void testFacotryBean(){
        Object bean = applicationContext.getBean("colorFactoryBean");
        System.out.println("bean的类型："+bean.getClass());

        Object bean1 = applicationContext.getBean("&colorFactoryBean");
        System.out.println("bean的类型："+bean1.getClass());
    }
#结果
bean的类型：class com.xsbl.bean.Color
bean的类型：class com.xsbl.bean.ColorFactoryBean
```



### 7.组件注册总结

**给容器中注册组件：**

> 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
> 2）、@Bean[导入的第三方包里面的组件]
> 3）、@Import[快速给容器中导入一个组件]
>  		1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
> 		2）、ImportSelector:返回需要导入的组件的全类名数组；
>  		3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
> 4）、使用Spring提供的 FactoryBean（工厂Bean）;
>  		1）、默认获取到的是工厂bean调用getObject创建的对象
>  		2）、要获取工厂Bean本身，我们需要给id前面加一个&
>       			  &colorFactoryBean



## 2.Bean 的生命周期

### 2.1 指定初始化和销毁方法

> bean的生命周期：
>
> ​			bean创建---初始化----销毁的过程
>
> 容器管理bean的生命周期；
>
> 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

>  初始化：
>
> ​				对象创建完成，并赋值好，调用初始化方法。。。
>
> 销毁：
>
> ​			单实例：容器关闭的时候
>
> ​			多实例：容器不会管理这个bean；容器不会调用销毁方法；



1. 定义bean的初始化方法和销毁方法

   ```java 
   public class Car {
   	
   	public Car(){
   		System.out.println("car constructor...");
   	}
   	
   	public void init(){
   		System.out.println("car ... init...");
   	}
   	
   	public void detory(){
   		System.out.println("car ... detory...");
   	}
   
   }
   
   ```

2. 注入bean中

   ```java
   @Configuration
   public class MainConfigOfLifeCycle {
   	//指定初始化的方法名init,和销毁方法的方法名 detory
   	//@Scope("prototype")
   	@Bean(initMethod="init",destroyMethod="detory")
   	public Car car(){
   		return new Car();
   	}
   }
   ```

3. 测试

   ```java
   @Test
   public void test01(){
       //1、创建ioc容器
       AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
       System.out.println("容器创建完成...");
   
       //applicationContext.getBean("car");
       //关闭容器
       applicationContext.close();
   }
   ```

   

### 2.2  实现InitializingBean 和DisposableBean 接口定义初始化和销毁

1. 定义bean，实现InitializingBean 和DisposableBean 接口

```json
@Component
public class Cat implements InitializingBean,DisposableBean {	
	public Cat(){
		System.out.println("cat constructor...");
	}
    #销毁方法
	@Override
	public void destroy() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("cat...destroy...");
	}
    # 初始化方法
	@Override
	public void afterPropertiesSet() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("cat...afterPropertiesSet...");
	}
}
```

2. 在配置文件中将bean注册进容器中

```java 
@ComponentScan("com.atguigu.bean")
@Configuration
public class MainConfigOfLifeCycle {
}
```

3. 测试

```java
@Test
public void test01(){
    //1、创建ioc容器
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
    System.out.println("容器创建完成...");

    //applicationContext.getBean("car");
    //关闭容器
    applicationContext.close();
}

```



### 2.3 @PostConstruct @PreDestroy 实现初始化和销毁

> JSR250标准：
> @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
>
> @PreDestroy：在容器销毁bean之前通知我们进行清理工作

```java
@Component
public class Dog{
	
	public Dog(){
		System.out.println("dog constructor...");
	}
	
	//对象创建并赋值之后调用
	@PostConstruct
	public void init(){
		System.out.println("Dog....@PostConstruct...");
	}	
	//容器移除对象之前
	@PreDestroy
	public void detory(){
		System.out.println("Dog....@PreDestroy...");
	}
}
```



### 2.4bean的后置处理器 BeanPostProcessor[接口]

> BeanPostProcessor【interface】：bean的后置处理器；
>  * 在bean初始化前后进行一些处理工作；
>
>    ​		postProcessBeforeInitialization:在初始化之前工作
>
>    ​		postProcessAfterInitialization:在初始化之后工作

```java
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```



```java
/**
 * 后置处理器：初始化前后进行处理工作
 * 将后置处理器加入到容器中
 * @author lfy
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
		return bean;
	}
}
```



### 2.5 BeanPostProcessor原理

参考：[Spring-Bean生命周期](https://zhuanlan.zhihu.com/p/158468104)

> BeanPostProcessor原理:
>
> 1.populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
>
> 2.initializeBean
>
> {
>
> ​	applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
>
> ​	invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
>
> ​	applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
> }
>
> 注：使用前面的 MyBeanPostProcessor类debug源码
>
> 
>
> 三、应用场景
>
> 　　几个典型的应用如：
>
> 　　1、解析bean的注解，将注解中的字段转化为属性
>
> 　　2、统一将属性在执行前，注入bean中，如数据库访问的sqlMap，如严重服务，这样不需要每个bean都配置属性
>
> 　　3、打印日志，记录时间等。





```java
Object wrappedBean = bean;
if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
}

try {
    invokeInitMethods(beanName, wrappedBean, mbd);
}
catch (Throwable ex) {
    throw new BeanCreationException(
        (mbd != null ? mbd.getResourceDescription() : null),
        beanName, "Invocation of init method failed", ex);
}

if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```



#### Spring底层对 BeanPostProcessor 的使用

 参考：[Spring-Bean生命周期](https://zhuanlan.zhihu.com/p/158468104)

有关应用：**与 bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor**

![image-20210108125214842](springAnnotatioin.assets/image-20210108125214842.png)

getBean--->doGetBean()---调用getSingleton()中的getObject()--->实际调用createBean()--->doCreateBean()



```json
#1. new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
#2.调用refresh方法
public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
    this();
    register(annotatedClasses);
	refresh();
}

@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		// Prepare this context for refreshing.
		prepareRefresh();

		// Tell the subclass to refresh the internal bean factory.
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		// Prepare the bean factory for use in this context.
		prepareBeanFactory(beanFactory);

		try {
			// Allows post-processing of the bean factory in context subclasses.
			postProcessBeanFactory(beanFactory);

			// Invoke factory processors registered as beans in the context.
			invokeBeanFactoryPostProcessors(beanFactory);

			// Register bean processors that intercept bean creation.
			registerBeanPostProcessors(beanFactory);

			// Initialize message source for this context.
			initMessageSource();

			// Initialize event multicaster for this context.
			initApplicationEventMulticaster();

			// Initialize other special beans in specific context subclasses.
			onRefresh();

			// Check for listener beans and register them.
			registerListeners();
            
			#3.调用 finishBeanFactoryInitialization 
			// Instantiate all remaining (non-lazy-init) singletons.
			finishBeanFactoryInitialization(beanFactory);

			// Last step: publish corresponding event.
			finishRefresh();
		}

		catch (BeansException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Exception encountered during context initialization - " +
						"cancelling refresh attempt: " + ex);
			}

			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
			cancelRefresh(ex);

			// Propagate exception to caller.
			throw ex;
		}

		finally {
			// Reset common introspection caches in Spring's core, since we
			// might not ever need metadata for singleton beans anymore...
			resetCommonCaches();
		}
	}
}

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
	// Initialize conversion service for this context.
	if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
			beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
		beanFactory.setConversionService(
				beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
	}

	if (!beanFactory.hasEmbeddedValueResolver()) {
		beanFactory.addEmbeddedValueResolver(new StringValueResolver() {
			@Override
			public String resolveStringValue(String strVal) {
				return getEnvironment().resolvePlaceholders(strVal);
			}
		});
	}

	// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
	String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
	for (String weaverAwareName : weaverAwareNames) {
		getBean(weaverAwareName);
	}

	// Stop using the temporary ClassLoader for type matching.
	beanFactory.setTempClassLoader(null);

	// Allow for caching all bean definition metadata, not expecting further changes.
	beanFactory.freezeConfiguration();
	#4.调用preInstantiateSingletons方法
	// Instantiate all remaining (non-lazy-init) singletons.
	beanFactory.preInstantiateSingletons();
}

@Override
public void preInstantiateSingletons() throws BeansException {

	// Iterate over a copy to allow for init methods which in turn register new bean definitions.
	// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
	List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);

	// Trigger initialization of all non-lazy singleton beans...
	for (String beanName : beanNames) {
		RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
		if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
			if (isFactoryBean(beanName)) {
				final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
				boolean isEagerInit;
				if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
					isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
						@Override
						public Boolean run() {
							return ((SmartFactoryBean<?>) factory).isEagerInit();
						}
					}, getAccessControlContext());
				}
				else {
					isEagerInit = (factory instanceof SmartFactoryBean &&
							((SmartFactoryBean<?>) factory).isEagerInit());
				}
				if (isEagerInit) {
					getBean(beanName);
				}
			}
			else {
                #5.调用getBean()方法
				getBean(beanName);
			}
		}
	}

	// Trigger post-initialization callback for all applicable beans...
	for (String beanName : beanNames) {
		Object singletonInstance = getSingleton(beanName);
		if (singletonInstance instanceof SmartInitializingSingleton) {
			final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
			if (System.getSecurityManager() != null) {
				AccessController.doPrivileged(new PrivilegedAction<Object>() {
					@Override
					public Object run() {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}
				}, getAccessControlContext());
			}
			else {
				smartSingleton.afterSingletonsInstantiated();
			}
		}
	}
}

@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

protected <T> T doGetBean()throws BeansException {
	try {
		final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
		checkMergedBeanDefinition(mbd, beanName, args);
		// Create bean instance.
		if (mbd.isSingleton()) {
            #获取单例
			sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
				@Override
				public Object getObject() throws BeansException {
					try {
                    	#实际调用createBean()方法
						return createBean(beanName, mbd, args);
					}
					catch (BeansException ex) {
						destroySingleton(beanName);
						throw ex;
					}
				}
			});
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
		}
	}	
}

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		try {
    		#调用doGetBean中ObjectFactory实现的getObject()方法
			singletonObject = singletonFactory.getObject();
			newSingleton = true;
		}
		return (singletonObject != NULL_OBJECT ? singletonObject : null);
	}
}

@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args){
	try {
    	#实现代理Proxy的关键步骤
    // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance. 
		Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
		if (bean != null) {
			return bean;
		}
	}
	#调用doCreateBean方法
	Object beanInstance = doCreateBean(beanName, mbdToUse, args);
	
	return beanInstance;
}
#在调用doCreateBean之前
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
	Object bean = null;
	if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
		// Make sure bean class is actually resolved at this point.
        #如果实现了InstantiationAwareBeanPostProcessor 
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			Class<?> targetType = determineTargetType(beanName, mbd);
			if (targetType != null) {
                #实例化之前
				bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
				if (bean != null) { #如果返回不为null,则说明修改了bean对象，执行实例化后置对象
                    #实例化之后
					bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
				}
			}
		}
		mbd.beforeInstantiationResolved = (bean != null);
	}
	return bean;
}
#applyBeanPostProcessorsBeforeInstantiation解读
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
	for (BeanPostProcessor bp : getBeanPostProcessors()) {
		if (bp instanceof InstantiationAwareBeanPostProcessor) {
			InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
			#注释5：只要其中一个postProcessBeforeInstantiation返回实例bean即结束回调，
             #这个bean将会直接返回给bean容器管理
			Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
			if (result != null) {
				return result;
			}
		}
	}
	return null;
}

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args){

	// Instantiate the bean.
	BeanWrapper instanceWrapper = null;
	if (mbd.isSingleton()) {
		instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
	}
	if (instanceWrapper == null) {
		instanceWrapper = createBeanInstance(beanName, mbd, args);
	}
	final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
	Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);
	mbd.resolvedTargetType = beanType;

	// Initialize the bean instance.
	Object exposedObject = bean;
	try {
		#属性赋值
		populateBean(beanName, mbd, instanceWrapper);
		if (exposedObject != null) {
			#初始化
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
	}
	return exposedObject;
}

protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
	Object wrappedBean = bean;
	if (mbd == null || !mbd.isSynthetic()) {
		#初始化之前
		wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
	}

	try {
		#主要作用有两个：
        #1、判断bean是否继承了InitializingBean，如果继承接口，执行afterPropertiesSet()方法，
		#2、获得是否设置了init-method属性，如果设置了，就执行设置的方法
		invokeInitMethods(beanName, wrappedBean, mbd);
	}
	
	if (mbd == null || !mbd.isSynthetic()) {
		#初始化之后
		wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
	}
	return wrappedBean;
}

@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName){
	Object result = existingBean;
    #遍历处理
	for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
		result = beanProcessor.postProcessBeforeInitialization(result, beanName);
		if (result == null) {
			return result;
		}
	}
	return result;
}

protected void invokeInitMethods(String beanName, final Object bean, RootBeanDefinition mbd){
	boolean isInitializingBean = (bean instanceof InitializingBean);
	if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
		if (System.getSecurityManager() != null) {
			try {
				AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
					@Override
					public Object run() throws Exception {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}
				}, getAccessControlContext());
			}
		}
		else {
            #调用实现了InitializingBean接口方法
			((InitializingBean) bean).afterPropertiesSet();
		}
	}

	if (mbd != null) {
		String initMethodName = mbd.getInitMethodName();
		if (initMethodName != null && !(isInitializingBean && 											"afterPropertiesSet".equals(initMethodName)) &&
				!mbd.isExternallyManagedInitMethod(initMethodName)) {
            #调用设置了init-method属性的方法
			invokeCustomInitMethod(beanName, bean, mbd);
		}
	}
}
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName){
	Object result = existingBean;
    #遍历处理
	for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
		result = beanProcessor.postProcessAfterInitialization(result, beanName);
		if (result == null) {
			return result;
		}
	}
	return result;
}
```

![preview](springAnnotatioin.assets/v2-14dd69d3b86097a8ce5178c86276627d_r.jpg)

```json
# XXXAware 接口为容器中注入底层bean，如ApplicationContext，BeanFactory等
@Component
public class Dog implements ApplicationContextAware {
	
	//@Autowired
	private ApplicationContext applicationContext;
	
	public Dog(){
		System.out.println("dog constructor...");
	}
	
	//对象创建并赋值之后调用
	@PostConstruct
	public void init(){
		System.out.println("Dog....@PostConstruct...");
	}
	
	//容器移除对象之前
	@PreDestroy
	public void detory(){
		System.out.println("Dog....@PreDestroy...");
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		// TODO Auto-generated method stub
		this.applicationContext = applicationContext;
	}	
}

```

![image-20210113161745487](springAnnotatioin.assets/image-20210113161745487.png)



## 3.属性赋值

### 3.1 @Value

> 使用@Value赋值；
> 	1、基本数值
> 	2、可以写SpEL； #{}
> 	3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）

```json
public class Person {
	
	//使用@Value赋值；
	//1、基本数值
	//2、可以写SpEL； #{}
	//3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）
	
	@Value("张三")
	private String name;
	@Value("#{20-2}")
	private Integer age;
	
    //需要使用PropertySource加载配置文件到环境变量中，${}才能生效
	@Value("${person.nickName}")
	private String nickName;
	
    
	public String getNickName() {
		return nickName;
	}
	public void setNickName(String nickName) {
		this.nickName = nickName;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getAge() {
		return age;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	
	public Person(String name, Integer age) {
		super();
		this.name = name;
		this.age = age;
	}
	public Person() {
		super();
		// TODO Auto-generated constructor stub
	}
	@Override
	public String toString() {
		return "Person [name=" + name + ", age=" + age + ", nickName=" + nickName + "]";
	}
}
```

配置文件：

```java
//使用@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中;加载完外部的配置文件以后使用${}取出配置文件的值
@PropertySource(value={"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {
	
	@Bean
	public Person person(){
		return new Person();
	}

}
```

测试类：

```json
public class IOCTest_PropertyValue {
	AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
	@Test
	public void test01(){
		printBeans(applicationContext);
		System.out.println("=============");
		
		Person person = (Person) applicationContext.getBean("person");
		System.out.println(person);
		
		# 将配置文件加载到环境变量中，通过 环境变量获取配置文件的值
		ConfigurableEnvironment environment = applicationContext.getEnvironment();
		String property = environment.getProperty("person.nickName");
		System.out.println(property);
		applicationContext.close();
	}
	
	private void printBeans(AnnotationConfigApplicationContext applicationContext){
		String[] definitionNames = applicationContext.getBeanDefinitionNames();
		for (String name : definitionNames) {
			System.out.println(name);
		}
	}

}
```



### 3.2@PropertySource

相当于xml配置文件 <context:property-placeholder/>

```xml
<context:property-placeholder location="classpath:person.properties"/>
```



## 4.自动装配

> 自动装配：
>
>    	Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；

### 4.1 @AutoWired/@Qualifier/@Primary

> 1）、@Autowired：自动注入：
> 		1）、**默认优先按照类型**去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
> 		2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
> 							applicationContext.getBean("bookDao")
> 		3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
> 		4）、自动装配默认一定要将属性赋值好，没有就会报错；
> 			可以使用@Autowired(required=false);
> 		5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
> 				也可以继续使用@Qualifier指定需要装配的bean的名字   【@Qualifier 和 @Primary 不要一起用】
>
>  		BookService{
> 			@Autowired
> 			BookDao  bookDao;
> 		}
>
> 建议使用：@Autowrite + @Qualifier

```java
@Controller
public class BookController {	
	@Autowired
	private BookService bookService;
}
```

```java
@Service
public class BookService {
	
	//@Qualifier("bookDao")
	@Autowired(required=false)
	private BookDao bookDao;
	
	public void print(){
		System.out.println(bookDao);
	}

	@Override
	public String toString() {
		return "BookService [bookDao=" + bookDao + "]";
	}				
}
```

```java
@Repository
public class BookDao {
	
	private String lable = "1";

	public String getLable() {
		return lable;
	}

	public void setLable(String lable) {
		this.lable = lable;
	}

	@Override
	public String toString() {
		return "BookDao [lable=" + lable + "]";
	}	
}
```



配置类：

```java
@Configuration
@ComponentScan({"com.atguigu.service","com.atguigu.dao",
	"com.atguigu.controller","com.atguigu.bean"})
public class MainConifgOfAutowired {
	
	@Primary
	@Bean("bookDao2")
	public BookDao bookDao(){
		BookDao bookDao = new BookDao();
		bookDao.setLable("2");
		return bookDao;
	}
}
```

测试类：

```java
@Test
public void test01(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConifgOfAutowired.class);

    BookService bookService = applicationContext.getBean(BookService.class);
    System.out.println(bookService);

    BookDao bean = applicationContext.getBean(BookDao.class);
    System.out.println(bean);

}
```



### 4.2 @Resource和@Inject注解

> Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
>  * @Resource:
>
>    ​	可以和@Autowired一样实现自动装配功能；**默认是按照组件名称进行装配的**；
>
>    ​	没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
>  * @Inject:
>
>    需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
>
>    @Autowired:Spring定义的； @Resource、@Inject都是java规范

bean类：

```json 
@Controller
public class BookController {	
	@Autowired
	private BookService bookService;
}

@Service
public class BookService {

	//@Resource(name="bookDao2")
	@Inject
	private BookDao bookDao;
	
	public void print(){
		System.out.println(bookDao);
	}

	@Override
	public String toString() {
		return "BookService [bookDao=" + bookDao + "]";
	}	
}

@Repository
public class BookDao {
	
	private String lable = "1";

	public String getLable() {
		return lable;
	}

	public void setLable(String lable) {
		this.lable = lable;
	}

	@Override
	public String toString() {
		return "BookDao [lable=" + lable + "]";
	}	

}
```

配置类：

```json
@Configuration
@ComponentScan({"com.atguigu.service","com.atguigu.dao",
	"com.atguigu.controller","com.atguigu.bean"})
public class MainConifgOfAutowired {
	
	@Primary
	@Bean("bookDao2")
	public BookDao bookDao(){
		BookDao bookDao = new BookDao();
		bookDao.setLable("2");
		return bookDao;
	}	
}
```



### 4.3 @Autowired 默认不写的规则

> 1. [标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
>
> 2. [标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取 【Spring推荐写法】
>
> 3. 放在参数位置：

第一种：使用@Component的方式

```json
//默认加在ioc容器中的组件，容器启动会调用无参构造器创建对象，再进行初始化赋值等操作
@Component
public class Boss {
	
	private Car car;
	
	//构造器要用的组件，都是从容器中获取
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

public class Color {
	
	private Car car;

	public Car getCar() {
		return car;
	}

	public void setCar(Car car) {
		this.car = car;
	}

	@Override
	public String toString() {
		return "Color [car=" + car + "]";
	}
}

```



第二种：使用@Bean

```json
@Configuration
@ComponentScan({"com.atguigu.service","com.atguigu.dao",
	"com.atguigu.controller","com.atguigu.bean"})
public class MainConifgOfAutowired {
	/**
	 * @Bean标注的方法创建对象的时候，方法参数的值从容器中获取
	 * @param car
	 * @return
	 */
	@Bean
	public Color color(/*@Autowired*/Car car){
		Color color = new Color();
		color.setCar(car);
		return color;
	}
	

}

#测试

	@Test
	public void test01(){
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConifgOfAutowired.class);
		
		BookService bookService = applicationContext.getBean(BookService.class);
		System.out.println(bookService);
		
		//BookDao bean = applicationContext.getBean(BookDao.class);
		//System.out.println(bean);
		
		Boss boss = applicationContext.getBean(Boss.class);
		System.out.println(boss);
		Car car = applicationContext.getBean(Car.class);
		System.out.println(car);
		
		Color color = applicationContext.getBean(Color.class);
		System.out.println(color);
		System.out.println(applicationContext);
		applicationContext.close();
	}
```





### 4.4 把spring底层组件注册到bean中

> 自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
>  * 		自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
>  * 		把Spring底层一些组件注入到自定义的Bean中；
>  * 		xxxAware：功能使用xxxProcessor；
>  * 			ApplicationContextAware==》ApplicationContextAwareProcessor[通过后置处理器完成]；
>  * 			![image-20210111125920743](springAnnotatioin.assets/image-20210111125920743.png)

```json
#例子
@Component
public class Red implements ApplicationContextAware,BeanNameAware,EmbeddedValueResolverAware {
	
	private ApplicationContext applicationContext;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("传入的ioc："+applicationContext);
		this.applicationContext = applicationContext;
	}

	@Override
	public void setBeanName(String name) {
		// TODO Auto-generated method stub
		System.out.println("当前bean的名字："+name);
	}

	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		// TODO Auto-generated method stub
		String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
		System.out.println("解析的字符串："+resolveStringValue);
	}
}
```



### 4.5 @Profile

>  * Profile：
>  * 		Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
>  * 
>  * 开发环境、测试环境、生产环境；
>  * 数据源：(/A)(/B)(/C)；
>
>  * @Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件
>  * 
>  * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。**默认是default环境**
>  * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
>  * 3）、**没有标注环境标识的bean在，任何环境下都是加载的；**

```json
# 通过@PropertySource注解引入properties文件【见下面获取properties文件值的方式】
@PropertySource("classpath:/dbconfig.properties")
@Configuration                              #3.通过实现值解析器，获取properties属性值
public class MainConfigOfProfile implements EmbeddedValueResolverAware{
	
	@Value("${db.user}") # 1 在属性上直接写 获取properties文件值
	private String user;
	
	private StringValueResolver valueResolver;
	
	private String  driverClass;
	
	@Bean
	public Yellow yellow(){
		return new Yellow();
	}
	
	@Profile("test")
	@Bean("testDataSource")
	public DataSource dataSourceTest(@Value("${db.password}")# 2 在形参上写 获取properties文件
                                     String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}
	
	@Profile("dev")
	@Bean("devDataSource")
	public DataSource dataSourceDev(@Value("${db.password}")String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_crud");
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}
	
	@Profile("prod")
	@Bean("prodDataSource")
	public DataSource dataSourceProd(@Value("${db.password}")String pwd) throws Exception{
		ComboPooledDataSource dataSource = new ComboPooledDataSource();
		dataSource.setUser(user);
		dataSource.setPassword(pwd);
		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/scw_0515");
		
		dataSource.setDriverClass(driverClass);
		return dataSource;
	}

	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		// TODO Auto-generated method stub
		this.valueResolver = resolver;
        #获取底层值解析器，解析${}
		driverClass = valueResolver.resolveStringValue("${db.driverClass}");
	}
}
```



```json
 #测试
	//1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
	//2、代码的方式激活某种环境；
	@Test
	public void test01(){
		AnnotationConfigApplicationContext applicationContext = 
				new AnnotationConfigApplicationContext();
		//1、创建一个applicationContext
		//2、设置需要激活的环境
		applicationContext.getEnvironment().setActiveProfiles("dev");
		//3、注册主配置类
		applicationContext.register(MainConfigOfProfile.class);
		//4、启动刷新容器
		applicationContext.refresh();		
		
		String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
		for (String string : namesForType) {
			System.out.println(string);
		}

		#没有标注环境标识的bean在，任何环境下都是加载的；
		Yellow bean = applicationContext.getBean(Yellow.class);
		System.out.println(bean);
		applicationContext.close();
	}
```



**激活方法**

1. 使用命令行动态参数的方法：在虚拟机参数位置加载 -Dspring.profiles.active=test

![image-20210111133416707](springAnnotatioin.assets/image-20210111133416707.png)

2. 代码的方式激活某种环境

![image-20210111131701295](springAnnotatioin.assets/image-20210111131701295.png)

## 5.AOP动态代理

>  * AOP：【动态代理】
>
>    ​	指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式
>
>    三步：
>
>    1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
>
>    2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
>
>    3）、开启基于注解的aop模式；@EnableAspectJAutoProxy

### 5.1详细步骤

1、导入aop模块；Spring AOP：(spring-aspects)

![image-20210111132326614](springAnnotatioin.assets/image-20210111132326614.png)

![image-20210111132354479](springAnnotatioin.assets/image-20210111132354479.png)

2.定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）

```json
public class MathCalculator {	
	public int div(int i,int j){
		System.out.println("MathCalculator...div...");
		return i/j;	
	}
}
```

3.定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；

 * 通知方法：

   ​	前置通知(@Before)：logStart：在目标方法(div)运行之前运行

   ​	后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）

   ​	返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行

   ​	异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行

   ​	环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）

 4、给切面类的目标方法标注何时何地运行（通知注解）；

```json
#切入点表达式
#1、本类引用       #@Before("pointCut()")
#2、其他的切面引用  #@After("com.atguigu.aop.LogAspects.pointCut()")
@Pointcut("execution(public int com.atguigu.aop.MathCalculator.*(..))")
public void pointCut(){}; #方法不用实现
```

5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;

6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)

7、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】

 * 		在Spring中很多的 @EnableXXX;

```xml
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>>
```



```json
#1.配置类
@EnableAspectJAutoProxy  #开启基于注解的AOP模式
@Configuration
public class MainConfigOfAOP {	 
	//业务逻辑类加入容器中
	@Bean
	public MathCalculator calculator(){
		return new MathCalculator();
	}
	//切面类加入到容器中
	@Bean
	public LogAspects logAspects(){
		return new LogAsects();
	}
}
```

```json
#2.日志切面类
# @Aspect：告诉Spring当前类是一个切面类
# 切面类的每一个方法称为通知方法，通过切入表达式告诉 MathCalculator#div 何时何地运行
@Aspect  
public class LogAspects {
	
	//抽取公共的切入点表达式
	//1、本类引用
	//2、其他的切面引用
	@Pointcut("execution(public int com.atguigu.aop.MathCalculator.*(..))")
	public void pointCut(){};
	
	//@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
	@Before("pointCut()")
	public void logStart(JoinPoint joinPoint){  
		Object[] args = joinPoint.getArgs();
		System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
	}
	
	@After("com.atguigu.aop.LogAspects.pointCut()")
	public void logEnd(JoinPoint joinPoint){
		System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
	}
	
	//JoinPoint一定要出现在参数表的第一位
    #需要加returning，并指定形参 为 result
	@AfterReturning(value="pointCut()",returning="result")
	public void logReturn(JoinPoint joinPoint,Object result){
		System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
	}
	
	#需要加throwing，并指定异常 为 exception
	@AfterThrowing(value="pointCut()",throwing="exception")
	public void logException(JoinPoint joinPoint,Exception exception){
		System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
	}

}
```

```json
 #测试类
	@Test
	public void test01(){
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);
		
		//1、不要自己创建对象
//		MathCalculator mathCalculator = new MathCalculator();
//		mathCalculator.div(1, 1);
		MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
		
		mathCalculator.div(1, 0);
		
		applicationContext.close();
	}
#结果

```

### 5.2 AOP原理

> AOP原理：【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】
>
>  * 		@EnableAspectJAutoProxy；

#### 1.@EnableAspectJAutoProxy是什么？

利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetion

​	**internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator**

​	给容器中注册一个AnnotationAwareAspectJAutoProxyCreator；

AnnotationAwareAspectJAutoProxyCreator架构图
![image-20210115101300905](springAnnotatioin.assets/image-20210115101300905.png)



```json
AnnotationAwareAspectJAutoProxyCreator
	-->AspectJAwareAdvisorAutoProxyCreator
		-->AbstractAdvisorAutoProxyCreator
			-->AbstractAutoProxyCreator
				implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
				关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory

 AbstractAutoProxyCreator.setBeanFactory()
 AbstractAutoProxyCreator.有后置处理器的逻辑;  
 AbstractAdvisorAutoProxyCreator.setBeanFactory()-->initBeanFactory()

 AnnotationAwareAspectJAutoProxyCreator.initBeanFactory()

```

#### 2.AOP流程

**※.创建和注册AnnotationAwareAspectJAutoProxyCreator的过程**

1. 传入配置类，创建IOC容器

2. 注册配置类，调用refresh()刷新容器

3. registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；

   1. 先获取IOC容器已经定义了的需要创建对象的所有BeanPostProcessor

      ```json
      String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
      ```

      ![image-20210115164645315](springAnnotatioin.assets/image-20210115164645315.png)

   2. 给容器中加别的 BeanPostProcessor

   3. 优先注册实现了PriorityOrdered接口的BeanPostProcessor；

   ![image-20210115130139729](springAnnotatioin.assets/image-20210115130139729.png)

   4. 再给容器中注册实现了Ordered接口的BeanPostProcessor；

   ![image-20210115130319273](springAnnotatioin.assets/image-20210115130319273.png)

   5. 注册没实现优先级接口的BeanPostProcessor；

   ```json
   for (String ppName : postProcessorNames) {
   	if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
   		BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   		priorityOrderedPostProcessors.add(pp);
   		if (pp instanceof MergedBeanDefinitionPostProcessor) {
   			internalPostProcessors.add(pp);
   		}
   	}
   	else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
   		orderedPostProcessorNames.add(ppName);
   	}
   	else {
   		nonOrderedPostProcessorNames.add(ppName);
   	}
   }
   ```

   

   6. 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；

    创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】

   ```json
   List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
   for (String ppName : orderedPostProcessorNames) {
       # getBean()方法创建bean的实例
   	BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
   	orderedPostProcessors.add(pp);
   	if (pp instanceof MergedBeanDefinitionPostProcessor) {
   		internalPostProcessors.add(pp);
   	}
   }
   sortPostProcessors(orderedPostProcessors, beanFactory);
   registerBeanPostProcessors(beanFactory, orderedPostProcessors);
   ```

   

   1. 创建Bean的实例

      getBean--->doGetBean--->getSingleton--->craeteBean()--->doCreateBean()

   2. populateBean；给bean的各种属性赋值

   3. initializeBean：初始化bean；

      1. invokeAwareMethods()：处理Aware接口的方法回调
      2. applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
      3. invokeInitMethods()；执行自定义的初始化方法
      4. applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；

   4. BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--> aspectJAdvisorsBuilder

   1. 把BeanPostProcessor注册到BeanFactory中；

      ​		beanFactory.addBeanPostProcessor(postProcessor);

**※.AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor**

4. finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean

   1. 遍历获取容器中所有的Bean，依次创建对象getBean(beanName);

       * 				getBean->doGetBean()->getSingleton()->

   2. 创建bean     **【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】**

      1. 先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；只要创建好的Bean都会被缓存起来

      2. createBean();  创建bean；

         AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例

         【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】

         【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】

         1.resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation

          * 希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续

          * 后置处理器先尝试返回对象；

            bean = applyBeanPostProcessorsBeforeInstantiation()：

            拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;就执行postProcessBeforeInstantiation

         ​				if (bean != null) {
         ​					  bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
         ​				}

         2.doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；

         

#### 3.AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用

1. **每一个bean创建之前，调用postProcessBeforeInstantiation()；**

   关心MathCalculator和LogAspect的创建

   1. 判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
   2. 判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，或者是否是切面（@Aspect）
   3. 是否需要跳过
      1. 获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
      2. 永远返回false

2. **创建对象**   

   postProcessAfterInitialization；

   ​	return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下

   1. 获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors

      1. 找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
      2. 获取到能在bean使用的增强器。
      3. 给增强器排序

   2. 保存当前bean在advisedBeans中；

   3. 如果当前bean需要增强，创建当前bean的代理对象；

      1. 获取所有增强器（通知方法）

      2. 保存到proxyFactory

      3. 建代理对象：Spring自动决定

         ​	JdkDynamicAopProxy(config);jdk动态代理；

         ​	ObjenesisCglibAopProxy(config);cglib的动态代理；

      4. 给容器中返回当前组件使用cglib增强了的代理对象；

      5. 以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；

3. **目标方法执行；**

   ![image-20210126160124083](springAnnotatioin.assets/image-20210126160124083.png)

   容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；

   1. CglibAopProxy.intercept();拦截目标方法的执行

   2. 根据ProxyFactory对象获取将要执行的目标方法拦截器链；

      List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

      1. List<Object> interceptorList保存所有拦截器 5

         一个默认的ExposeInvocationInterceptor 和 4个增强器；

      2. 遍历所有的增强器，将其转为Interceptor；

         registry.getInterceptors(advisor);

      3. 将增强器转为List<MethodInterceptor>；

         如果是MethodInterceptor，直接加入到集合中

         如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；

         转换完成返回MethodInterceptor数组；

   3. 如果没有拦截器链，直接执行目标方法;

      拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）

   4. 如果有拦截器链，把需要执行的目标对象，目标方法，

      拦截器链等信息传入创建一个 CglibMethodInvocation 对象，

      并调用 Object retVal =  mi.proceed();

   5. 拦截器链的触发过程;

      1. 果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；

      2. 链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
   
         拦截器链的机制，保证通知方法与目标方法的执行顺序；



### 5.3总结

> 总结：
> 1）、  @EnableAspectJAutoProxy 开启AOP功能
> 2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
> 3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
> 4）、容器的创建流程：
> 	1）、registerBeanPostProcessors（）注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象
> 	2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean
> 		1）、创建业务逻辑组件和切面组件
> 		2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
> 		3）、组件创建完之后，判断组件是否需要增强
> 			是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
> 5）、执行目标方法：
> 	1）、代理对象执行目标方法
> 	2）、CglibAopProxy.intercept()；
> 		1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
> 		2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
> 		3）、效果：
> 			正常执行：前置通知-》目标方法-》后置通知-》返回通知
> 			出现异常：前置通知-》目标方法-》后置通知-》异常通知



## 6.声明式事

### 6.1环境搭建

1、导入相关依赖
		数据源、数据库驱动、Spring-jdbc模块

```xml
        <!-- https://mvnrepository.com/artifact/c3p0/c3p0 -->
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.1.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.44</version>
        </dependency>

		<!--用jdbc操作数据库，导入spring-jdbc-->
		<!--用hibernate，或者Mybatis操作数据库，导入spring-orm-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.12.RELEASE</version>
        </dependency>
```



2、配置数据源、JdbcTemplate（Spring提供的简化数据库操作的工具）操作数据

3、给方法上标注 @Transactional 表示当前方法是一个事务方法；

4、 @EnableTransactionManagement 开启基于注解的事务管理功能；

```xml
<tx:annotation-driven/>
```

​		@EnableXXX

5、配置事务管理器来控制事务;
		@Bean
		public PlatformTransactionManager transactionManager()

```json
#1 配置数据源，JDBCTemplate，开启@EnableTransactionManagement 配置事务管理器来控制事务;
@EnableTransactionManagement
@Configuration
@ComponentScan("com.xsbl.tx")
public class TxConfig {

    //数据源
    @Bean
    public DataSource dataSource() throws Exception {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://192.168.220.133:3306/springboot?useSSL=false");
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception {
        #Spring对@Configuration类会特殊处理；给容器中加组件的方法，多次调用都只是从容器中找组件
        return new JdbcTemplate(dataSource());
    }

	//注册事务管理器在容器中
    @Bean
    public PlatformTransactionManager platformTransactionManager() throws Exception {
        return new DataSourceTransactionManager(dataSource());
    }
}

#2 标注@Transactional 
@Repository
public class UserDao {
	
	@Autowired
	private JdbcTemplate jdbcTemplate;
	public void insert(){
		String sql = "INSERT INTO `tbl_user`(username,age) VALUES(?,?)";
		String username = UUID.randomUUID().toString().substring(0, 5);
		jdbcTemplate.update(sql, username,19);		
	}
}
@Service
public class UserService {
	
	@Autowired
	private UserDao userDao;
	
    #加上事务注解@Transactional
	@Transactional
	public void insertUser(){
		userDao.insert();
		//otherDao.other();xxx
		System.out.println("插入完成...");
		int i = 10/0;
	}
}

#3.测试
public class IOCTest_Tx {	
	@Test
	public void test01(){
		AnnotationConfigApplicationContext applicationContext = 
				new AnnotationConfigApplicationContext(TxConfig.class);
		UserService userService = applicationContext.getBean(UserService.class);
		userService.insertUser();
		applicationContext.close();
	}
}
```





### 6.2原理

 1）、@EnableTransactionManagement
 			利用TransactionManagementConfigurationSelector给容器中会导入组件
 			导入两个组件
 				①.AutoProxyRegistrar
 				②.ProxyTransactionManagementConfiguration

 2）、AutoProxyRegistrar：
 			给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；
 			InfrastructureAdvisorAutoProxyCreator：？
 			利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；

 3）、ProxyTransactionManagementConfiguration 做了什么？
  			1、给容器中注册事务增强器；
  				1）、事务增强器要用事务注解的信息，AnnotationTransactionAttributeSource解析事务注解
  				2）、事务拦截器：
  					TransactionInterceptor；保存了事务属性信息，事务管理器；
  					他是一个 MethodInterceptor；
  					在目标方法执行的时候；
  						执行拦截器链；
  						事务拦截器：
  							1）、先获取事务相关的属性
  							2）、再获取PlatformTransactionManager，如果事先没有添加指定任何transactionmanger
  								最终会从容器中按照类型获取一个PlatformTransactionManager；
  							3）、执行目标方法
  								如果异常，获取到事务管理器，利用事务管理回滚操作；
  								如果正常，利用事务管理器，提交事务



![image-20210115202509016](springAnnotatioin.assets/image-20210115202509016.png)

















