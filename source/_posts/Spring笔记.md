---
title: Spring笔记
date: 2024-10-06 01:44:57
tags:
  - 笔记
categories:
  - Spring
cover: https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410060150642.webp
---

# :o:导学

传统Java开发的代码耦合性过强，接口与实现紧密耦合。此外，Log等日志功能也都写在业务代码中，这使得代码非常的繁琐和重复，同时也不便于维护

![Snipaste_2024-09-29_20-31-46](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409292032418.png)

为了解决这个问题，基于工厂设计模式的Ioc思想应运而生。

## :rainbow:IoC、 DI和AOP思想

\- IOC 控制反转思想的提出
**IOC思想**： Inversion of Control，翻译为“控制反转” 或“反转控制” ，强调的是原来在程序中创建Bean的权利反
转给第三方。
例如：原来在程序中手动的去 new UserServiceImpl()，手动的去new UserDaoImpl()，而根据IoC思想的指导，
寻求一个第三方去创建UserServiceImpl对象和UserDaoImpl对象。这样程序与具体对象就失去的直接联系。  

> 工厂设计模式， BeanFactory来充当第三方的角色，来产生Bean实例
> BeanFactory怎么知道产生哪些Bean实例呢？
> 可以使用配置文件配置Bean的基本信息， BeanFactory根据配置文件来生产Bean实例  

将实例类的创建交给工厂，通过BeanFactory创建对象，具体的，我们通过代码创建一个BeanFactiry工厂对象，让他来读取配置文件并创建Bean实例 

例如：

在你的业务代码中

```java
BeanFactory factory = new BeanFactory("Beans.xml");
UserService userService = (UserService)factory.getBean("userService");	//getBean方法创建的是Object类
```

在resources包下创建一个Bean.xml配置文件

```xml
<bean id="userService" class="com.example.service.impl.userServiceImpl">
    <!--假设其中有一个方法名为setUser用于设置用户名称,你可以用下面这种方式调用它-->
    <property name="user" ref = "user"></property>
</bean>
<bean id="user" class="com.example.pojo.entity.User"></bean>
```

将User在BeanFactory内部设置给userService的过程叫做注入，而UserService需要User的注入后进行工作，因此这个过程称为“**依赖注入**”也就是**DI思想**

**AOP思想：**

AOP(Aspect Oriented Programming)，即面向切面编程，它也是基于代理模式，可以在代理对象中对目标对象的方法进行增强，它采用横向切面抽取方法，可以理解为在方法的前后进行一些操作，比如可以在业务代码之前生成日志，这样，你就不需要在每一段业务代码前都加入关于输出日志的代码了。

## :rainbow:Spring容器

# :o:基于xml的Spring应用

| Xml配置方式                               | 功能描述                                                     |
| ----------------------------------------- | ------------------------------------------------------------ |
| <bean id="" class="">                     | Bean的id和全限定名配置                                       |
| <bean name="">                            | 通过name设置Bean的别名，通过别名也能直接获取到Bean实例       |
| <bean scope="">                           | Bean的作用范围， BeanFactory作为容器时取值singleton和prototype |
| <bean lazy-init="">                       | Bean的实例化时机，是否延迟加载。 BeanFactory作为容器时无效   |
| <bean init-method="">                     | Bean实例化后自动执行的初始化方法， method指定方法名          |
| <bean destroy-method="">                  | Bean实例销毁前的方法， method指定方法名                      |
| <bean autowire="byType">                  | 设置自动注入模式，常用的有按照类型byType，按照名字byName     |
| <bean factory-bean="" factory-method=""/> | 指定哪个工厂Bean的哪个方法完成Bean的创建                     |

## :rainbow:基本使用

### :one:Bean的实例化配置

Spring的实例化方式主要如下两种：

#### 构造方式实例化：

底层通过构造方法对Bean进行实例化

* 有参：在xml中加一个constructor-arg标签

```xml
<bean id="user1" class="com.example.pojo.entity.User">
    <constructor-arg name="name" value="李华"/>
    <constructor-arg name="age" value="18"/>
</bean>
```

可以通过调试查看实例化的值(首先确保你的User中有name和age这两个字段)

#### 工厂方式实例化：

底层通过调用自定义的工厂方法对Bean进行实例化

* 静态工厂实例化

静态工厂方法实例化Bean，其实就是定义一个工厂类，提供一个静态方法用于生产Bean实例，在将该工厂类及其
静态方法配置给Spring即可

```java
//工厂类
public class UserFactoryBean {
//非静态工厂方法
    public static User getUser(String name, int age){
    //可以在此编写一些其他逻辑代码
   		return new User(name, age);
    }
}  
```

```xml
<bean id="user2" class="com.example.factory.UserFactoryBean" factory-method="getUser">
	<constructor-arg name="name" value="李华"/>
	<constructor-arg name="age" value="18"/>
</bean>
```

* 实例工厂实例化

```java
public class UserFactoryBean{
    public User getUser1(String name, int age){
        return new User(name, age)
    }
}
```

需要实例化工厂Bean，通过工厂Bean的非静态方法实例化User Bean

```xml
<!--先实例化工厂-->
<bean id="MyFactoryBean" class="com.example.factory.UserFactoryBean"/>
<!--再实例化User-->
<bean id="user3" factory-bean="MyFactoryBean" factory-method="getUser1">
	<constructor-arg name="name" value="李华"/>
	<constructor-arg name="age" value="18"/>
</bean>
```

* FactoryBean工厂实例化  

Spring提供了FactoryBean的接口规范，
FactoryBean接口定义如下：

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = “factoryBeanObjectType” ;
    
    T getObject() throws Exception; //获得实例对象方法
    
    Class<?> getObjectType(); //获得实例对象类型方法
    
    default boolean isSingleton() {
    	return true;
    }
}
```

你可以实现这个接口，然后在xml中再配置User的Bean实例化

```java
//实现接口
public class MyFactoryBean3 implements FactoryBean<User>{
    public User getObject() throws Exception {
    	return new User();
    }
    public Class<?> getObjectType() {
    	return User.class;
    }
}
```

```xml
<!--配置FactoryBean交给Spring管理-->
<bean id="user4" class="com.explame.factory.MyFactoryBean3"/>
```

>通过ApplicationContext实例化的对象通常存放在：ApplicationContext->beanFactory->singletonObjects(单例池)中，但如果通过实现FactoryBean作为自定义Bean工厂配置的实例化对象并不存储在singletonObjects中，而是先保存在factiryBeanObjectCache(初沉池)中，当调用getBean方法时，才将实例化对象从初沉池中拿出放入单例池。
>
>你可以通过Debug验证它

### :two:Bean的依赖注入配置

Bean的依赖注入有两种方式：

| 注入方式                   | 配置方式                                                     | 需求                     |
| -------------------------- | ------------------------------------------------------------ | ------------------------ |
| 通过Bean的setter方法注入   | <property name="userDao" ref="userDao"/> <br /><property name="userDao" value="haohao"/> | 需要实现对应的set方法    |
| 通过构造Bean的方法进行注入 | <constructor-arg name="name" ref="userDao"/><br /> <constructor-arg name="name" value="haohao"/> | 在构造方法中包含这个参数 |

> 其中， ref 是 reference 的缩写形式，翻译为：涉及，参考的意思，用于引用其他Bean的id。 value 用于注入普通
> 属性值。
>
> 通常来说，依赖注入的数据类型有如下三种：
>
> * 普通数据类型，例如： String、 int、 boolean等，通过value属性指定。
> * 引用数据类型，例如： UserDaoImpl、 DataSource等，通过ref属性指定。
> * 集合数据类型，例如： List、 Map、 Properties等。  在下面演示

#### 集合类型数据

1. List：为userServiceImpl加入list字段，并且提供一个公开的set方法。这里以stringList、userList作为演示

```xml
<!--List集合的配置-->
<bean id="userService1" class="com.example.springlean.service.impl.UserServiceImpl">
    <property name="stringList">
        <list>
            <value>张三</value>
            <value>李四</value>
            <value>王五</value>
        </list>
    </property>
    <property name="userList">
        <list>
            <ref bean="user"/>
            <ref bean="user1"/>
            <ref bean="user2"/>
        </list>
    </property>
</bean>
```

为了方便，以下的其他集合类型的演示就默认是在userService1 Bean标签下的

2. set：它与List几乎没有区别，只是将list标签换成set标签

3. Map：使用entry标签设置键值对

```xml
<!--注入值为字符串的Map集合-->
<property name="valueMap">
    <map>
        <entry key="aaa" value="AAA" />
        <entry key="bbb" value="BBB" />
        <entry key="ccc" value="CCC" />
    </map>
</property>
    <!--注入值为对象的Map集合-->
<property name="objMap">
    <map>
        <entry key="ud" value-ref="user"/>
        <entry key="ud2" value-ref="user2"/>
        <entry key="ud3" value-ref="user3"/>
    </map>
</property>
```

4. properties：使用prop标签，并在标签中指定值：

```xml
<property name="properties">
    <props>
        <prop key="xxx">XXX</prop>
        <prop key="yyy">YYY</prop>
    </props>
</property>
```

### :three:Spring的其他配置标签

Spring的默认标签用到的是Spring的默认命名空间

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

该命名空间约束下的默认标签如下：

| 标签     | 作用                                                |
| -------- | --------------------------------------------------- |
| <beans>  | 一般作为 xml 配置根标签，其他标签都是该标签的子标签 |
| <bean>   | Bean的配置标签，上面已经详解了，此处不再阐述        |
| <import> | 外部资源导入标签                                    |
| <alias>  | 指定Bean的别名标签，使用较少                        |

#### 配置环境标签

在你的xml配置文件中，可以为beans指定环境

```xml
<!--配置测试环境下需要加载的bean实例-->
<beans profile="test">
    <!--一些bean或其他的配置-->
    <bean id="userService3" class="com.example.service.impl.UserServiceImpl"/>
</beans>
```

>Spring默认启用默认的配置环境，如果你想启用测试环境下的Bean配置，可以使用以下两种方式指定被激活的环境：
>
>* 使用命令行动态参数，虚拟机参数位置加载 -Dspring.profiles.active=test
>
>* 使用代码的方式设置环境变量 System.setProperty("spring.profiles.active","test")  
>
> >```java
> >System.setProperty("spring.profiles.active","test");
> >//通过ApplicationContext获取实例化对象
> >ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
> >UserService userService = (UserService) context.getBean("userService3");
> >```

#### 导入配置环境

即**Import**,通过以下方法指定导入，如果在beans2.xml中有其他的注入，也能通过ApplicationContext获取

```xml
<!--在beans.xml中-->
<import resource="classpath:application-user.xml"/>


<!--在application-user.xml中-->
<bean id="user" class="com.example.springlean.pojo.entity.User">
    <constructor-arg name="name" value="来自application-user.xml的User"/>
    <constructor-arg name="age" value="18"/>
</bean>
```

你可以在beanFactory->beanDefinitionMap的这个地方看到它的来源

![来自](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409301546670.jpg)

#### 导入非Spring的第三方标签

由于是第三方的，不能直接在Spring中使用，需要先引入项目依赖，再在根标签的name space和shemaLocation中导入

以java RPC框架 dubbo为例

1. 引入依赖

```xml
<!--dubbo-->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.7.5</version>
    </dependency>
```

2. 在bean.xml的beans根标签中添加

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://dubbo.apache.org/schema/dubbo
                           http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
</beans>
```

现在可以在beans.xml中使用dubbo的标签了

<a id="section2"></a>

###:four:实操（以druid数据库池为例）

1. 在pom.xml导入项目依赖

```xml
<!--druid池-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.49</version>
    </dependency>
    <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        <version>1.1.23</version>
    </dependency>
```

2. 通过静态工厂实例化方法实例化DateSource对象

```xml
<!--druid必备的driverName、url、username、password-->
<bean id="dateSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/sky_take_out"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
</bean>
```

```java
//通过ApplicationContext获得实例对象
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
DruidDateSource dateSource = (DruidDateSource)applicationContext.getBean("dateSource");
```

于是获得了一个DruidDataSource实例对象

## :rainbow:Bean 实例化的流程(原理)

### :one:读取配置文件并封装需要实例化的对象

Spring容器在进行初始化时，会将xml配置的 bean 的信息封装成一个 **BeanDefinition** 对象，所有的BeanDefinition存储到一个名为 **beanDefinitionMap** 的Map集合中去， Spring框架在对该Map进行遍历，使用反射创建Bean实例对象，创建好的Bean对象存储在一个名为 **singletonObjects** 的Map集合中，当调用getBean方法时则最终从该Map集合中取出Bean实例对象返回。  

```java
//singletonObjects的底层 DefaultListableBeanFactory和其中维护的beanDefinitionMap

public class DefaultListableBeanFactory extends ... implements ... {
//存储<bean>标签对应的BeanDefinition对象
//key:是Bean的beanName， value:是Bean定义对象BeanDefinition
private final Map<String, BeanDefinition> beanDefinitionMap;
}
```

Spring框架会取出beanDefinitionMap中的每个BeanDefinition信息，**反射构造方法**或**调用指定的工厂方法**生成Bean实例对象，所以只要将BeanDefinition注册到beanDefinitionMap这个Map中， Spring就会进行对应的Bean的实例化操作  

![Spring实例化流程](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409302108737.jpg)

### :two:通过反射创建对象并存放到单例池中

Bean实例及单例池**singletonObjects**， **beanDefinitionMap**中的**BeanDefinition**会被转化成对应的Bean实例对象，存储到单例池singletonObjects中去，在**DefaultListableBeanFactory**的上四级父类DefaultSingletonBeanRegistry中，维护着**singletonObjects**单例池，源码如下：

```java
//DefaultListableBeanFactory的上四级父类DefaultSingletonBeanRegistry和其中维护的singletonObjects

public class DefaultSingletonBeanRegistry extends ... implements ... {
//存储Bean实例的单例池
////key:是Bean的beanName， value:是Bean的实例对象
private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
}  
```

>:coffee:总结：Bean 实例化的基本流程
>
>* 加载xml配置文件，解析获取配置中的每个<bean>的信息，封装成一个个的BeanDefinition对象;
>* 将BeanDefinition存储在beanDefinitionMap ==> Map<String,BeanDefinition>中;
>* ApplicationContext底层遍历beanDefinitionMap，创建Bean实例对象;
>* 创建好的Bean实例对象，存储到singletonObjects ==> Map<String,Object>中;
>* 当执行applicationContext.getBean(beanName)时，从singletonObjects去匹配Bean实例返回。  

## :rainbow:后处理器

| 后处理器                            | 作用                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| BeanProcessor                       | 在Bean实例化之后，填充到单例池singletonObjects之前执行       |
| BeanFactoryPostProcessor            | 在BeanDefinitionMap填充完成后，还没开始创建实例之前执行      |
| BeanDefinitionRegistryPostProcessor | 用作向BeanDefinitionMap中注册BeanDefinition。它是BeanFactoryPostProcessor的子类,调用时机也和BeanFactoryPostProcessor相同 |

在使用后处理器时，需要实现对应的处理器接口，然后**将处理器的实现配置到xml配置文件中作为Bean**，这样Spring会自动的在相应的时间调用后处理器的方法。

### :one:Bean工厂后处理器

:tea:：以通过BeanDefinitionRegistryPostProcessor注册为例

```java
public class RegisterProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        BeanDefinition beanDefinition = new RootBeanDefinition("com.example.pojo.entity.User");
        registry.registerBeanDefinition("user4", beanDefinition);
    }
}
```

通过xml配置存储在BeanDefinitionMap中的BeanDefinition是GenericBeanDefinition，但通过后处理器注册BeanDefinition推荐使用RootBeanDefinition

![默认的BeanDefinition](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409302238650.jpg)

```xml
<!--不要忘了将后处理器配置到Spring容器中-->
<bean class="com.example.processor.RegisterProcessor"/>
```

此时，我们可以为之前的Spring实例化流程的流程图进行一些补充

![更多细节](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409302248465.jpg)

### :two:通过注解实例化

学习完bean后处理器后，我们知道了如何在Spring实例化的不同时间段加入一个新的类以将他实例化，这非常有用处。

我们很容易发现，通过配置xml文件来实例化对象**非常麻烦**，但如果将后处理器结合上`ResourcePatternResolver`（一个用来扫描包下的所有类的Spring提供的工具类和它的实现：`PathMatchingResourcePatternResolver`）就能够实现**只需要为需要实例化的类加上一个注解即可实现类的实例化。**

> [什么是ResourcePatternResolver](https://springdoc.cn/spring/core.html#resources-resourcepatternresolver)

1. 首先自定义一个注解，随便叫什么名字，我这里叫做`MyComponent`

```java
@Target(ElementType.TYPE)	//指定注解作用的类型，包括 Type、Field、Method、Package等，这里指定为Type，表示作用于类
@Retention(RetentionPolicy.RUNTIME)	//注解生效的时限（注解的生命周期），这里配置为运行时仍生效
public @interface MyCommponent{
    String value();	//这将用来在给类添加注解时指定类名
}
```

2. 为任何一个你想要实例化的类添加@MyComponent注解

```java
@MyComponent("oneBean")
public class oneBean {
}
```

3. 接下来，编写一段代码来**读取指定包（Package）下的所有添加了注解的类**

```java
package com.example.springlean.utils;

import com.example.anno.MyComponent;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;
import org.springframework.core.type.classreading.CachingMetadataReaderFactory;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.util.ClassUtils;

import java.util.HashMap;
import java.util.Map;

public class BaseClassScanUtils {

    //设置资源规则
    private static final String RESOURCE_PATTERN = "/**/*.class";

    public static Map<String, Class> scanMyComponentAnnotation(String basePackage) {

        //创建容器存储使用了指定注解的Bean字节码对象
        Map<String, Class> annotationClassMap = new HashMap<String, Class>();

        //spring工具类，可以获取指定路径下的全部类
        ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
        try {//拼接模式匹配字符串
            String pattern = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX + //	即"classpath*:"
                    ClassUtils.convertClassNameToResourcePath(basePackage) + 
                RESOURCE_PATTERN;//	即"/**/*.class"
            //读取所有的类
            Resource[] resources = resourcePatternResolver.getResources(pattern);
            //MetadataReader 的工厂类
            MetadataReaderFactory refractory = new CachingMetadataReaderFactory(resourcePatternResolver);
            for (Resource resource : resources) {
                //MetadataReader：用于读取类信息
                MetadataReader reader = refractory.getMetadataReader(resource);
                //扫描到的class
                String classname = reader.getClassMetadata().getClassName();
                Class<?> clazz = Class.forName(classname);
                //判断是否属于指定的注解类型，按照个人的注解名字更改
                if(clazz.isAnnotationPresent(MyComponent.class)){
                    //获得注解对象
                    MyComponent annotation = clazz.getAnnotation(MyComponent.class);
                    //获得属value属性值
                    String beanName = annotation.value();
                    //判断是否为""
                    if(beanName!=null&&!beanName.equals("")){
                        //存储到Map中去
                        annotationClassMap.put(beanName,clazz);
                        continue;
                    }

                    //如果没有为"",那就把当前类的类名作为beanName
                    annotationClassMap.put(clazz.getSimpleName(),clazz);

                }
            }
        } catch (Exception exception) {
        }

        return annotationClassMap;
    }
}
```

scanMyComponentAnnotation的参数表示源代码包，按照你的包结构传递对应的参数就可以获取对应包下所有加入了MyComponent注解的类。

4. 在这之后就可以借助**bean工厂后处理器**将这些添加了注解的类加入到BeanDefinitionMap中了

```java
//
package com.example.springlean.processor;

import com.example.springlean.utils.BaseClassScanUtils;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;

import java.util.Map;

public class RegisterProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        //获取所有加入了@MyComponent注解的类 
        Map<String, Class> classMap = BaseClassScanUtils.scanMyComponentAnnotation("com.example");//这是我的软件包的目录

        //将类注册到BeanDefinitionMap中
        classMap.forEach((beanName, clazz) ->{
            BeanDefinition beanDefinition = new RootBeanDefinition(clazz);
            beanDefinition.setBeanClassName(clazz.getName());//指定全限定名，相当于配置xml标签时的class=""
            registry.registerBeanDefinition(beanName, beanDefinition);
        });
    }
}
```

需要在开始实例化之前将BeanDefinition注册，因此需要使用**BeanFactoryPostProcessor**或**BeanDefinitionRegistryPostProcessor**

>```xml
><!--不要忘了将后处理器配置到Spring容器中-->
><bean class="com.example.processor.RegisterProcessor"/>
>```

之后就可以在spring容器中获取到oneBean了！🎉🎉🎉

```java
public static void main(String[] args){
    	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
		Object oneBean = applicationContext.getBean("oneBean");
        System.out.println(oneBean);
}
```

### :three:bean后处理器

beanPostProcessor

>前面已经讲过，Bean被实例化后，到最终缓存到名为singletonObjects单例池之前，中间会经过Bean的初始化过程，例如：属性的填充、初始方法init的执行等，其中有一个对外进行扩展的点BeanPostProcessor，我们称为Bean后处理。跟上面的Bean工厂后处理器相似，它也是一个接口，实现了该接口并被容器管理的BeanPostProcessor，会在流程节点上被Spring自动调用。

BeanPostProcessor的接口定义如下：

```java
public interface BeanPostProcessor {
    @Nullable
    //在属性注入完毕，init初始化方法执行之前被回调
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws 
    BeansException {
    	return bean;	//返回bean对象放入单例池中
    }
    @Nullable
    //在初始化方法执行之后，被添加到单例池singletonObjects之前被回调
    default Object postProcessAfterInitialization(Object bean, String beanName) throws 
    BeansException {
   		return bean;	//返回bean对象放入单例池中
    }
}
```

可以通过Bean后处理器+动态代理增强方法实现日志打印

```java
package com.example.springlean.processor;

import org.springframework.beans.factory.config.BeanPostProcessor;
import java.lang.reflect.Proxy;
import java.time.LocalDateTime;

public class TimeLogBeanPostProcessor implements BeanPostProcessor{
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName){
        //对bean进行动态代理，此时单例池中存放的不再是原来的Bean实例，而是方法被增强的代理实例
        Object proxyBean = Proxy.newProxyInstance(
                bean.getClass().getClassLoader(),
                bean.getClass().getInterfaces(),
                (proxy, method, args) -> {
                        System.out.println("开始时间：" + LocalDateTime.now());
                        Object invoked = method.invoke(bean, args);
                        System.out.println("结束时间：" + LocalDateTime.now());
                        return invoked;
                }
        );
        return proxyBean;	//将代理对象返回放入单例池
    }
}
```

bean实例化的初始阶段的全过程：

![Snipaste_2024-10-02_20-19-05](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410022019724.jpg)

## :rainbow: ​xml配置第三方框架

### :one: mybatisMapper

之前有通过Spring来xml配置来获取sqlSession的实例：​[跳转](#section2)。现在演示另一种：sqlSessionFactoryBean+MapperScannerConfigurer的获取Mapper来操作数据库。

```xml
<!--在 ApplicationContext.xml 中-->
<!--基本配置-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
<!--配置SqlSessionFactoryBean，将SqlSessionFactory存储到spring容器-->
<bean class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--MapperScannerConfigurer,扫描指定的包，产生Mapper对象存储到Spring容器-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="mapper"></property>
</bean>
```

然后在mapper包下创建了一个UserMapper：

```java
public interface UserMapper{
    public selectById();
}
```

为所需的Mapper配置一个xml文件，同时命名空间中指向这个Mapper。由于前面配置了扫描包，他会自己读取到其中的配置文件，并将SQL操作代理给Mapper的代理类，然后放到Spring容器中。

在resource包的mapper下新建一个UserMapper.xml配置文件,我这里随意写一个select语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <select id="select" resultType="com.example.pojo.User">
        select * from user where id = #{id}
    </select>
</mapper>
```

你可以通过Spring容器直接拿到UserMapper对象(实际上是一个代理对象)

![userMapper代理](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410042301785.jpg)

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

UserMapper userMapper = applicationContext.getBean(UserMapper.class);
```

这样就可以使用UserMapper的select方法从user表中读取数据了。

之后需要其他的Mapper，不需要做基本配置，在mapper包下新建一个对应的Mapper。然后在resources包下的mapper中也新建一个对应的.xml文件并将SQL操作写在配置文件中。

> 实际上开发中都是使用SpringBoot+MyBatisPlus的配置方法。但是这里是学习Spring，所以以此为例子而已。
>
> > SpringBoot+MyBatisPlus
> >
> > 首先确保导入了SpringBoot的MyBatisPlus依赖
> >
> > ```xml
> > <dependency>
> >  <groupId>org.mybatis.spring.boot</groupId>
> >  <artifactId>mybatis-spring-boot-starter</artifactId>
> > </dependency>
> > ```
> >
> > 在ApplicationContext中配置：
> >
> > ```yml
> > mybatis:
> > #mapper配置文件
> > mapper-locations: classpath:mapper/*.xml
> > type-aliases-package: com.example.entity
> > ```
> >
> > > mapper-locations 指定了mapper配置文件的路径
> > >
> > > type-aliases-package 让你可以在这个包下不用使用全限定名
> >
> > 此后就只需要在对应的 mapper.xml 的命名空间中指向对应的类即可

# :o:Bean的生命周期

————————————

>本节参阅：
>
>**[Spring Bean 的生命周期 - spring 中文网 (springdoc.cn)](https://springdoc.cn/the-life-cycle-of-a-spring-bean/#:~:text=本文介绍了 Spri)**
>
>[47-SpringBean的生命周期_哔哩哔哩_ bilibili](https://www.bilibili.com/video/BV1rt4y1u7q5?p=48&vd_source=3b2548ca76d1728cf98fff60e44d3e0a)
>
>[Bean的生命周期 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2184201)

Spring Bean的生命周期是从 Bean 实例化之后，即通过反射创建出对象之后，到Bean成为一个完整对象，存储到单例池中，到Bean最后被销毁，这个过程被称为Spring Bean的生命周期。 Spring Bean的生命周期大体上分为四个阶段：实例化阶段、初始化阶段、完成阶段、销毁阶段。

* Bean的实例化阶段： Spring框架会取出BeanDefinition的信息进行判断当前Bean的范围是否是singleton的，是否不是延迟加载的，是否不是FactoryBean等，最终通过反射进行实例化singleton的Bean；
* Bean的初始化阶段： Bean创建之后还仅仅是个"半成品"，还需要对Bean实例的属性进行填充、执行一些Aware接口方法、执行BeanPostProcessor方法、执行InitializingBean接口的初始化方法、执行自定义初始化init方法等。

> 该阶段是Spring最具技术含量和复杂度的阶段， Aop增强功能，Spring的注解功能等、Bean的循环引用问题都是在这个阶段体现的；

* Bean的完成阶段：经过初始化阶段， Bean就成为了一个完整的Spring Bean，被存储到单例池singletonObjects中去了，即完成了Spring Bean的整个生命周期。  
* Bean销毁阶段

下图是Spring在创建一个Bean时的方法调用流程图，可以帮助理解Bean的生命周期

![SpringBean生命周期](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410041639236.png)

**更多细节：**

![Spring构建bean时的方法调用](https://s2.loli.net/2024/10/02/iuOqJWrm2h9tcMl.png)

## :rainbow: Aware接口

前面两个关于Spring生命周期的图中都涉及到了一些Aware接口，Aware接口提供了一个让程序员获得框架的一些对象的机会。Aware接口在不同的阶段被调用，并且将对应的对象通过参数注入给你，你所需要做的，就是实现对应的接口，然后在重新方法中实现编写预期逻辑。

以下是一些常见的Aware接口：

| Aware接口               | 回调方法                                                     | 作用                                                       |
| ----------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| ServletContextAware     | setServletContext(ServletContext context)                    | Spring框架回调方法注入ServletContext对象， web环境下才生效 |
| BeanFactoryAware        | setBeanFactory(BeanFactory factory)                          | Spring框架回调方法注入beanFactory对象                      |
| BeanNameAware           | setBeanName(String beanName)                                 | Spring框架回调方法注入当前Bean在容器中 的beanName          |
| ApplicationContextAware | setApplicationContext(ApplicationContext applicationContext) | Spring框架回调方法注入applicationContext 对象              |

现在，终于知道了Spring构建Bean的完整流程：

![Spring构建Bean完整流程](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410041650431.jpg)



# :o: 基于注解的Spring应用

通过注解能够实现和xml配置相同的功能，而且更便捷更容易

## :rainbow: 基本注解

### :one: 和实例化相关的

这实际上相当于在xml中配置一个bean标签

@Component 用于注册到Spring容器，它的其他配置信息也可以通过注解来实现

| xml配置                  | 注解           | 描述                                                         |
| ------------------------ | -------------- | ------------------------------------------------------------ |
| <bean scope="">          | @Scope         | 在类上或使用了@Bean标注的方法上，标注Bean的作用范围，取值为 singleton或prototype |
| <bean lazy-init="">      | @Lazy          | 在类上或使用了@Bean标注的方法上，标注Bean是否延迟加载，取值为 true和false |
| <bean init-method="">    | @PostConstruct | 在方法上使用，标注Bean的实例化后执行的方法                   |
| <bean destroy-method=""> | @PreDestroy    | 在方法上使用，标注Bean的销毁前执行方法                       |

此外，还有几个注解和它作用相同，只是用于不同的应用常见。

| @Component衍生注解 | 描述                |
| ------------------ | ------------------- |
| @Repository        | 在Dao层类上使用     |
| @Service           | 在Service层类上使用 |
| @Controller        | 在Web层类上使用     |

### :two: 依赖注入

注入属性值的注解有：

| 属性注入注解 | 描述                                                   |
| ------------ | ------------------------------------------------------ |
| @Value       | 使用在字段或方法上，用于注入普通数据                   |
| @Autowired   | 使用在字段或方法上，用于根据类型（byType）注入引用数据 |
| @Qualifier   | 使用在字段或方法上，结合@Autowired， 根据名称注入      |
| @Resource    | 使用在字段或方法上，根据类型或名称进行注入             |

**这些注解可以用在方法的参数上**，为参数自动注入对象或值

> 注：Value还可以通过$符指向在properties中配置的键值对，像前面xml中一样。前提是在配置了
>
> <context:property-placeholder location="classpath:jdbc.properties"/>  标签
>
> 这类标签也可以通过注解进行配置：[注解配置非默认标签](#section3), 现在可以先在xml中配置验一下
>
> ```java
> @Value("${jdbc.username}")
> private String username;
> 
> //同时，在jdbc.properties中
> jdbc.username=root
> 
> @Resource
> private UserDao userDao;
> @Resource(name = "userDao2")	//会自动将配置了@Component("userDao2")的类注入到方法的参数中
> public void setUserDao(UserDao userDao){
> System.out.println(userDao);
> }
> ```

### :three: 非自定义Bean的注解

@Bean注解

```java
//以MyBatis的dataSource举例
@Bean("dataSource")
public DataSource dataSource(){
DruidDataSource dataSource = new DruidDataSource();
dataSource.setDriverClassName("com.mysql.jdbc.Driver");
dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis");
dataSource.setUsername("root");
dataSource.setPassword("root");
return dataSource;
}
```

忘记的可以点击：[跳转](#section2)

**之前自动注入的注解可以用在参数上**

```java
@Bean("dataSource")
public DataSource dataSource(@Value("${jdbc.driver}") String Driver,
                            @Value("${jdbc.url}") String url,
                            @Value("${jdbc.username}") String username,
                            @Value("${jdbc.password}") String password,){
DruidDataSource dataSource = new DruidDataSource();
dataSource.setDriverClassName(Driver);
dataSource.setUrl(url);
dataSource.setUsername(username);
dataSource.setPassword(password);
return dataSource;
}
```

在方法上配置了@Bean时，如果方法中的参数也在Spring容器中，这个参数会被自动注入

```java
@Bean
public void show(User User){
    System.out.println(user);
}
```

### :four: @Configuration注解

**@Configuration** 注解标识的类为配置类，替代原有xml配置文件，该注解第一个作用是标识该类是一个配置类，第
二个作用是具备@Component作用  

**@ComponentScan** 组件扫描配置，替代原有xml文件中的<context:component-scan base-package=""/>

<a id="section3"></a>

**@PropertySource** 注解用于加载外部properties资源配置，替代原有xml中的 <context:propertyplaceholder location=“” /> 配置  

```java
@Configuration
@ComponentScan({"com.example.service","com.example.pojo"})
@PropertySource({"classpath:jdbc.properties"})	//它的value是一个String数组，所以用{}包含，表示数组中的元素
public class ApplicationContextConfig {}
```

此时获取Spring容器就不再通过ClassPathXmlApplicationContext获取，而是AnnotationConfigApplicationContext

```java
ApplicationContext applicationContext = new  AnnotationConfigApplicationContext(ApplicationContextConfig.class);
```

你可以通过它获取对应包下加入了@Component相关注解的实例对象

### :five: 更多...

还有：

| 注解     | 作用         |
| -------- | ------------ |
| @profile | 变量的环境   |
| @import  | 导入其他配置 |
| ...      | ...          |

注解太多了，使用也不难，和xml方式的不同就是读取的是配置的注解而不是配置文件，其他的更底层的逻辑都是类似的。

> 不同注解的读取器也是通过Bean工厂后处理器等注册到Spring容器当中的，
>
> ```java
> //比如Autowired注解，通过实现Bean后处理器，将这个注解加入到Spring容器当中
> public class AutowiredAnnotationBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor, PriorityOrdered, BeanFactoryAware {
>  ...
> }
> ```

