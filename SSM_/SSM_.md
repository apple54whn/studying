[TOC]



# 第三部分 Spring

> 以5.0.2版本讲解。要求JDK8及以上，Tomcat8.5及以上



## 1 Spring概述

Spring是分层的 Java SE/EE应用 **full-stack 轻量级开源框架**，**以 IoC（Inverse Of Control： 控制反转）和 AOP（Aspect Oriented Programming：面向切面编程）为内核**，提供了展现层 Spring MVC 和持久层 Spring JDBC 以及业务层事务管理等众多的企业级应用技术，还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的Java EE 企业应用开源框架。 

------

Spring的**优势**：

* **方便解耦，简化开发**

  通过Spring提供的IoC容器，可以将对象间的依赖关系交由 Spring进行控制，避免硬编码所造成的过度程序耦合。用户也不必再为单例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用

* **AOP编程**的支持

  通过 Spring的 AOP 功能，方便进行面向切面的编程，许多不容易用传统OOP实现的功能可以通过 AOP 轻松应付

* **声明式事务**的支持  

  可以将我们从单调烦闷的事务管理代码中解脱出来，通过声明式方式灵活的进行事务的管理， 提高开发效率和质量

* 方便程序的**测试**  

  可以用非容器依赖的编程方式进行几乎所有的测试工作，测试不再是昂贵的操作，而是随手可做的事情

* 方便**集成各种优秀框架**  

  Spring可以降低各种框架的使用难度，提供了对各种优秀框架（Struts、Hibernate、Hessian、Quartz 等）的直接支持

* **降低JavaEE API的使用难度**  

  Spring对 JavaEE API（如 JDBC、JavaMail、远程调用等）进行了薄薄的封装层，使这些 API 的 使用难度大为降低

* **Java源码是经典学习范例**  

  Spring的源代码设计精妙、结构清晰、匠心独用，处处体现着大师对Java 设计模式灵活运用以 及对 Java技术的高深造诣。它的源代码无意是 Java 技术的最佳实践的范例

------

**Spring的体系结构**

![](images\spring-overview.png)

## 2 IoC

引入：程序间耦合

- 在软件工程中，耦合指的就是就是**==对象之间的依赖性==**。对象之间的耦合越高，维护成本越高。因此对象的设计应使类和构件之间的耦合最小。软件设计中通常用耦合度和内聚度作为衡量模块独立程度的标准。**划分模块的一个准则**就是**==高内聚低耦合==**。

- **Bean**：计算机英语中，有**可重用组件**的含义

  JavaBean：用**Java语言编写的可重用组件**。JavaBean>实体类

- 实际开发中，应做到：**编译期不依赖**，**运行时才依赖**

- 解耦的思路（**==工厂模式==**）：
  1. **读取配置文件的key**获取要创建对象的**全限定类名value**；
  2. 使用**反射创建对象**，避免使用new关键字；
  3. 定义一个map**容器**，在静态代码块中创建所有对象并存放。获取对象时直接返回对应key的value，是单例的

------

* ==**IoC（控制反转）**：**把创建对象的权利交给容器（或工厂）**。可以**解决程序之间的耦合**，解除代码中的依赖关系。==
* DI：依赖注入。即调用类对某一接口实现类的依赖关系由第三方容器或协作类注入，移除调用类对某一接口实现类的依赖
  * 一般完成特定的业务逻辑可能会需要多个类之间进行协作。按传统的做法，每个对象负责管理与自己互相协作的对象(它所依赖的对象)的引用，这会导致高度耦合并难以测试的代码。利用依赖注入，每个对象可以直接获取所依赖的对象

------

Maven中添加Spring依赖的坐标

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.1.RELEASE</version>
</dependency>
```

上述是复合依赖，总共依赖了6项：

* Core container中四项`spring-beans`、`spring-core`、`spring-context`、`spring-expression` 和 `spring-jcl`（封装logging）以及注解时要依赖的`spring-aop`

### 2.1 基于xml的配置

步骤：

1. 在resources中创建**`spring-config.xml`或`applicationContext.xml`**文件。（约束在参考中搜索`xmlns`即可找到）

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
   </beans>
   ```

2. 在配置文件中**配置让 spring 创建的对象**（原理和引入中工厂模式一致）

   ```xml
   <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl"></bean>
   <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
   ```

3. 获取Spring IoC核心容器，并根据id获取对象

   ```java
   ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");
   AccountDao accountDao = (AccountDao) ac.getBean("accountDao"); //需要强转
   AccountService accountService = ac.getBean("accountService", AccountService.class); //传字节码文件对象，不用强转
   ```

------

Spirng中容器接口及其实现类

* ~~`BeanFactory`顶层容器接口。创建对象采用**延迟加载**方式（何时使用何时创建，多例适用）~~
* **`ApplicationContext`**容器接口。创建对象采用**立即加载**方式（==**读完配置文件**，**默认情况下就创建(可修改)**==，单例适用）
  * **`ClassPathXmlApplicationContext`**，基于xml配置的实现类。只能加载**类路径**下的配置文件
  * `FileSystemXmlApplicationContext`，基于xml配置的实现类。可以加载**磁盘任意路径**的配置文件（需有访问权限）
  * **`AnnotationConfigApplicationContext`**，基于注解配置的实现类。读取**注解**创建容器

------

#### 2.1.1 Spirng对Bean的管理细节

##### 1、bean标签

* 作用：==**配置对象让spring来创建的**==。默认情况下它调用的是类中的**==无参构造函数==**。如果没有无参构造函数则不能创建成功。 
* 属性：
  * **`id`**：给对象在容器中提供一个**唯一标识**，用于获取对象
  * **`class`**：指定**类的全限定类名**，用于**反射创建对象**，默认情况下调用无参构造函数
  * **`scope`**：==**指定对象的作用范围**==
    * **`singleton`**：**==单例==**的（默认），一个应用只有一个对象的实例，它的作用范围就是**整个应用** 
    * **`prototype`**：**==多例==**的，每次访问对象时，都会重新创建对象实例。
    * `request`：WEB 项目中，Spring 为**每个请求**创建一个bean实例
    * `session`：WEB 项目中，Spring 为**每个会话**创建一个bean实例
    * `global-session`：作用于**集群(Portlet)环境的全局会话范围**，当不是集群(Portlet)环境时，它就是session
  * `init-method`：指定类中的初始化方法名称
  * `destroy-method`：指定类中销毁方法名称。单例中需要关闭容器（使用实现类的方法）

##### 2、Bean对象的生命周期

- **==单例对象：生命周期和容器相同==**，一个应用只有一个对象的实例，它的作用范围就是整个应用
  - 出生：当容器创建时出生（读取完配置文件）
  - 存活：只要容器存在，对象一直存活
  - 死亡：容器销毁，对象消亡
- **==多例对象==**：每次访问对象时，都会重新创建对象实例。
  - **出生**：当我们**==使用对象==**时，**Spring框架为我们创建**
  - **存活**：对象只要在**使用过程中，就一直存活**
  - **死亡**：当对象**长时间不使用，且没有别的对象引用**时，由Java的**GC**机制回收

##### 3、创建Bean的三种方式

1. **使用==默认构造方法==创建**

   配置文件中使用`bean`标签以及`id`和`class`属性，且没有其他属性和标签时~。若类中没有默认构造方法，则对象无法创建

   ```xml
   <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl"></bean>
   ```

2. 使用**==普通工厂中的方法==**创建对象（也称使用某个类中的方法创建对象），并存入Spirng容器

   ```xml
   <bean id="accountDaoFactory" class="com.itheima.factory.AccountDaoFactory"></bean>
   <bean id="accountDao" factory-bean="accountDaoFactory" factory-method="getAccountDao"></bean>
   ```

3. 使用**==静态工厂中的静态方法==**创建对象（也称使用某个类中的静态方法创建对象），并存入Spirng容器

   ```xml
   <bean id="accountDao" class="com.itheima.factory.AccountDaoFactory" factory-method="getAccountDao"></bean>
   ```



#### 2.1.2 依赖注入

> 我们的业务层仍会调用持久层的方法，在使用 spring 之后，就让 **spring 来维护**了他们的**依赖关系**。就是让框架把持久层对象传入业务层，不用自己去获取。

DI（Dependency Injection）：**依赖注入**，即是**依赖关系的维护**交由Spirng管理。

能注入的数据有三类：（不适用于经常变换的数据）

* **基本类型和String**
* **其他bean类型**（在配置文件或注解中配置过的bean）
* **复杂类型/集合类型**：如Array、List、Set、Map、Properties



##### 1、构造方法注入

使用类中的构造方法给成员变量赋值。注意：赋值的操作不是我们自己做的，而是通过配置的方式，让spring框架来为我们注入

* 要求：类中需要提供一个**对应参数列表的构造方法**
* 标签：`bean`标签中**`constructor-arg`**
  * 属性：
    * ~~`index`：指定参数在构造函数参数列表的索引位置，从0开始。不常用。~~
    * ~~`type`：指定参数在构造函数中的数据类型。不常用。~~
    * **`name`**：指定**参数在构造函数中的名称**
    * **`value`**：它能赋的值是**基本数据类型**和 **String 类型** 
    * **`ref`**：它能赋的值是**其他 bean 类型**，也就是说，必须得是**配置过的 bean** 

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"> 
    <constructor-arg name="name" value=" 张三 "></constructor-arg>  <!--String类型-->
    <constructor-arg name="age" value="18"></constructor-arg>       <!--基本类型-->
    <constructor-arg name="birthday" ref="now"></constructor-arg>   <!--其他bean类型-->
</bean> 
<bean id="now" class="java.util.Date"></bean> 
```

* 优势：在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。
* 弊端：优势也是其弊端，改变了bean对象的实例化方式。创建对象时，即使用不到这些数据，也必须提供。

> c-命名空间注入（本质是构造方法注入）：无法实现装配集合



##### 2、set方法注入(使用最多)

* 要求：就是在类中**提供需要注入成**员的 **set 方法**
* 标签：`bean`标签中**`property`**
  * 属性：
    * **`name`**：找的是类中 **set 方法后面的部分** 
    * **`value`**：它能赋的值是**基本数据类型**和 **String 类型**
    *  **`ref`**：它能赋的值是**其他 bean 类型**，也就是说，必须得是**配置过的 bean** 

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">   
    <property name="name" value="test"></property> <!--String类型-->
    <property name="age" value="21"></property>    <!--基本类型-->
    <property name="birthday" ref="now"></property> <!--其他bean类型-->
</bean> 
<bean id="now" class="java.util.Date"></bean>
```

* 优势：创建对象时，注入的参数没有明确限制，可以直接使用默认构造方法
* 弊端：优势也是其弊端，若某个成员必须有值，则获取对象时有可能set方法没有执行

> p-命名空间注入（本质还是调用 set 方法）：无法实现装配集合



##### 3、注入集合属性

就是给类中的**集合成员传值**，可以采用set方法注入的方式，只不过变量的数据类型都是集合

* 注入集合数据：只要**==结构相同，标签可以互换==** 
  * List结构可以用：array、list、set
  * Map结构可以用：map、entry；props、prop。但是properties只能存储键值都是字符串的数据。

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"> 
 <!-- 在注入集合数据时，只要结构相同，标签可以互换 -->  
    <!-- 给数组注入数据 -->  
    <property name="myStrs"> 
        <set>    
            <value>AAA</value>    
            <ref bean="car"></ref><!--也可以引用类型。只是列出来参考，实际不会这样存储不同类型数据-->
        </set> 
    </property> 
    <!-- 注入 list 集合数据 -->  
    <property name="myList">   
        <array>    
            <value>AAA</value>    
            <ref bean="car"></ref><!--也可以引用类型。只是列出来参考，实际不会这样存储不同类型数据-->
        </array>  
    </property>  
    <!-- 注入 set 集合数据 -->  
    <property name="mySet"> 
        <list>    
            <value>AAA</value>    
            <ref bean="car"></ref><!--也可以引用类型。只是列出来参考，实际不会这样存储不同类型数据-->
        </list>  
    </property> 
    <!--=============================================-->
    <!-- 注入 Map 数据 -->  
    <property name="myMap">   
        <map>    
            <entry key="testA" value="aaa"></entry> 
            <entry key-ref="car" value-ref="car"></entry> <!--key,value,key-ref,value-ref  共4种组合 -->
            
            <!--value-type="cn.itcast.pojo.Car"   可以不设置，设置后值类型唯一-->
      <!--  <entry key="testB">   也可以这样写，但是麻烦  
                <value>bbb</value> 
            </entry>  --> 
        </map>
    </property> 
    <!-- 注入 properties 数据--> 
    <property name="myProps"> 
          <props> 
            <prop key="testA">aaa</prop>    
            <prop key="testB">bbb</prop>   
        </props> 
    </property> 
</bean>
<!--========================被引用的测试bean =====================================-->
<bean name="car" class="cn.itcast.pojo.Car">
    <property name="name" value="BMW"></property>
    <property name="color" value="red"></property>
</bean>
```





### 2.2 基于注解的配置

在spring-config.xml**开启对注解的支持**，并告知Spring**创建容器时扫描的包**。（约束在参考中搜索`xmlns:context`即可找到） 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 告知spring在创建容器时要扫描的包 -->
    <context:component-scan base-package="com.itheima"></context:component-scan>
    <!--配置QueryRunner-->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <!--注入数据源-->
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/eesy"></property>
        <property name="user" value="root"></property>
        <property name="password" value="1234"></property>
    </bean>
</beans>
```

注解方式查看2.3





### 2.3 Spring纯注解配置(JavaConfig)

> 写到此处，基于注解的 IoC 配置已经完成，但是大家都发现了一个问题：我们依然离不开 spring 的 xml 配 置文件，那么能不能不写这个 xml，所有配置都用注解来实现呢？ 当然，同学们也需要注意一下，我们选择哪种配置的原则是简化开发和配置方便，而非追求某种技术。 

待改造的问题：我们发现，之所以我们现在离不开 xml 配置文件，是因为我们有一句很关键的配置

* 告知Spring框架在读取配置文件创建容器时，**扫描注解**，依据注解**创建对象**，并**存入容器**中
* 另外，**数据源和 JdbcTemplate 的配置**也需要靠注解来实现

------

#### 1、`@Configuration`配置类

* **`@Configuration`**

  用于**指定当前类是一个Spring配置类**，当创建容器时会从该类上加载注解。

  - `value`用于指定配置类的字节码，一般不用
  - 细节：当**配置类**作为`AnnotationConfigApplicationContext`对象创建的**参数**时，该注解可以不写

  获取容器时使用 `AnnotationConfigApplicationContext(有@Configuration 注解的类.class)`来创建`ApplicationContext`对象。由于本身也是`@Component`，所以指定@ComponentScan时可以跳过此类（了解）

  * `String[] getBeanDefinitionNames()`：查询这个IoC容器中所有Bean的名称
  * `String[] getBeanNamesForType(Class c)`：查询这个IoC容器中指定类型的Bean的名称
  * `Map<String, Class >getBeansOfType(Class c)`：查询这个IoC容器中所有Bean的名称、和Bean对象
  * `Object getBean([String beanName ,] [Class c])`

* **`@PropertySource`**

  用于**指定properties文件的位置**，是重复注解

  - `value`指定文件的**名称和路径**数组。关键字：**classpath**，表示类路径下




#### 2、`@ComponentScan`注册组件

> 包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]

* `@ComponentScans`

  用于多个`@ComponentScan`。属性有`value`数组，可以包含多个` @ComponentScan`

* **` @ComponentScan`**

  指定Spring在**初始化容器**时**要扫描的包**（扫描4个注解的类），在JDK8之后是重复注解，直接写多个，不再使用上边那个

  * `value`或`basePackages`都是用于指定要扫描的包，都是数组类型

  * `excludeFilters`指定扫描的时候按照什么规则排除那些组件

  * `includeFilters`指定扫描的时候只需要包含哪些组件

    值为`Filter[]`，使用`@Filter`注解，属性有`type`（如下）、`classes`（Class类型）

    * `FilterType.ANNOTATION`：按照带有的**注解类型**
    * `FilterType.ASSIGNABLE_TYPE`：按照**给定的具体类型**
    * `FilterType.ASPECTJ`：使用ASPECTJ表达式
    * `FilterType.REGEX`：使用正则指定
    * `FilterType.CUSTOM`：使用自定义规则，需要自定义`TypeFilter`的实现类，重写`match`方法

  * `useDefaultFilters`在使用指定的规则时需要将这个值设置为`false`

    ```java
    public class MyTypeFilter implements TypeFilter {
        /**
    	 * metadataReader：读取到的当前正在扫描的类的信息
    	 * metadataReaderFactory:可以获取到其他任何类信息的
    	 */
        @Override
        public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
            throws IOException {
            //获取当前类注解的信息
            AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
            //获取当前正在扫描的类的类信息
            ClassMetadata classMetadata = metadataReader.getClassMetadata();
            //获取当前类资源（类的路径）
            Resource resource = metadataReader.getResource();
    
            String className = classMetadata.getClassName();
            System.out.println("--->"+className);
            if(className.contains("er")){
                return true;
            }
            return false;
        }
    }
    ```

    ```java
    @ComponentScan(value="com.atguigu",includeFilters = {
        @Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
        @Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),
        @Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class})
    },useDefaultFilters = false)	
    ```

> 作用和xml配置文件中编写`<bean>`标签实现功能一致

- **`@Component`**：用于==**把当前类对象存入Spirng容器中**==。注解在实现类上不指定value会自动指定value（删除Impl）

  - 属性：`value`：用于指定bean的id。不写时默认为当前类名，且首字母小写

- **`@Controller`**或**`@RestController`**：表现层

- **`@Service`**：业务层

- **`@Repository`**：持久层

  以上三个注解的作用和属性与`@Componet`一致，是Spring提供明确的三层架构使用的注解，使三层对象更加清晰

  也可以使用`@Scope`、`@Conditional`注解，如下：





#### 3、`@Bean`注册组件

> 导入的第三方jar包里面的组件

* **`@Bean`**

  ==用于把当前**方法的返回值**作为bean对象**注册存入Spring容器**中==。该注解**只能写在方法上**

  * `value`或`name`用于指定**bean的id**。没写则默认为**当前方法的名称**
  * 细节：当我们使用注解配置方法时，如果==**方法有参数**==，Spring框架会去容器中查找有没有可用的bean对象。查找的方式和`@Autowired`注解的作用是一样的

* **`@Scope`**：用于指定bean的作用范围。也可以放置注解在定义的4个组件上

  - `value`指定范围的取值。常用有：
    - **`singleton`**：**==单例==**【默认】，一个应用只有一个对象的实例，IoC容器启动时则会创建对象并放入容器
      - `@Lazy`：懒加载，修改单例对象创建时间，变为获取对象时才创建，但还是单例
    - **`prototype`**：**==多例==**，每次获取对象时，都会重新创建对象实例。
    - `request`：WEB 项目中，Spring 为**每个请求**创建一个bean实例
    - `session`：WEB 项目中，Spring 为**每个会话**创建一个bean实例
    - `global-session`：作用于**集群(Portlet)环境的全局会话范围**，当不是集群(Portlet)环境时，它就是session

* **`@Profile`**

  指定**组件**在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件

  * 加了环境标识的**bean**，只有这个环境被激活的时候才能注册到容器中。默认不写是default环境，即没有标注环境标识的bean在任何环境下都是加载的

  * 写在**配置类上**，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效

    ```java
    @Profile("test")
    ```

  使用：

  * 运行时使用命令行动态参数：虚拟机参数位置写上`-Dspring.profiles.active=test`

  * 代码的方式激活某种环境

    ```java
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    applicationContext.getEnvironment().setActiveProfiles("dev","test");//设置需要激活的环境
    applicationContext.register(MainConfigOfProfile.class);//注册主配置类
    applicationContext.refresh();//启动刷新容器
    ```

* **`@Conditional`**

  类中组件统一设置。**满足当前条件**，这个配置的**bean注册才能生效**。可以标在类和方法上。**Springboot中使用很多**

  * `value`为实现了`Condition`接口的数组，实现类重写`matches`方法

    ```java
    //判断是否linux系统
    public class LinuxCondition implements Condition {
    
        /**
    	 * ConditionContext：判断条件能使用的上下文（环境）
    	 * AnnotatedTypeMetadata：注释信息
    	 */
        @Override
        public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
            // TODO是否linux系统
            //1、能获取到ioc使用的beanfactory
            ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
            //2、获取类加载器
            ClassLoader classLoader = context.getClassLoader();
            //3、获取当前环境信息
            Environment environment = context.getEnvironment();
            //4、获取到bean定义的注册类
            BeanDefinitionRegistry registry = context.getRegistry();
    
            
            //可以判断容器中的bean注册情况，也可以给容器中注册bean
            boolean definition = registry.containsBeanDefinition("person");
            
            String property = environment.getProperty("os.name");
            if(property.contains("linux")){
                return true;
            }
            return false;
        }
    }
    ```

    ```java
    @Conditional(LinuxCondition.class)//可以标在类（满足条件则类中所有Bean才会注册）和方法（只限制这个方法的Bean）上
    @Bean("linus")
    public Person person02(){
        return new Person("linus", 48);
    }
    ```


#### 4、`@Import`注册组件

* **`@Import`**

  用于**导入其他的配置类**也可以理解为**直接导入第三方组件**，只能标注在类、接口、枚举类上

  * `value`用于指定其他配置类的**字节码**，是数组，被导入的其他配置类也不用加`@Configuration`注解

    注册的Bean的id为组件的全类名；若是配置类则还会注册配置的Bean（id为指定的或方法名）

    ```java
    @Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
    //@Import导入组件，id默认是组件的全类名
    public class MainConfig2 {
    ```

    * **`ImportSelector`**接口：返回需要导入的组件的全类名数组，需要自定义类实现该接口。**Springboot中使用很多**

      ```java
      //自定义逻辑返回需要导入的组件
      public class MyImportSelector implements ImportSelector {
          //返回值，就是到导入到容器中的组件全类名
          //AnnotationMetadata:当前标注@Import注解的类的所有注解信息
          @Override
          public String[] selectImports(AnnotationMetadata importingClassMetadata) {
              return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
          }
      }
      ```

    * `ImportBeanDefinitionRegistrar`接口：手动注册Bean到容器中，需要自定义类实现该接口

      ```java
      public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
          /**
      	 * AnnotationMetadata:当前标注@Import注解的类的所有注解信息
      	 * BeanDefinitionRegistry:BeanDefinition注册类；
      	 * 把所有需要添加到容器中的bean；调用BeanDefinitionRegistry.registerBeanDefinition手工注册进来
      	 */
          @Override
          public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
              boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
              boolean definition2 = registry.containsBeanDefinition("com.atguigu.bean.Blue");
              if(definition && definition2){
                  //指定Bean定义信息；（Bean的类型，Bean。。。）
                  RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
                  //注册一个Bean，指定bean名
                  registry.registerBeanDefinition("rainBow", beanDefinition);
              }
          }
      }
      ```


#### 5、FactoryBean注册组件

与`@Import`不同的是，它是调用无参构造创建Bean，而这个利用工厂获取Bean。多用于**整合Spring和其他框架**的底层代码

```java
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {
	//返回一个Color对象，这个对象会添加到容器中
	@Override
	public Color getObject() throws Exception {
		System.out.println("ColorFactoryBean...getObject...");
		return new Color();
	}
	@Override
	public Class<?> getObjectType() {
		return Color.class;
	}
	//是否为单例？
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

```java
@Bean
public ColorFactoryBean colorFactoryBean(){
    return new ColorFactoryBean();
}
```

```java
//工厂Bean获取的是工厂bean调用getObject创建的对象
Object bean1 = applicationContext.getBean("colorFactoryBean");
System.out.println("bean的类型："+bean1.getClass());//Color的全限定类名

Object bean2 = applicationContext.getBean("&colorFactoryBean");
System.out.println(bean2.getClass());//ColorFactoryBean的全限定类名
```



#### 6、Bean的生命周期方法

> 作用和xml配置文件中的`<bean>`标签编写`<init-method>`和`<destroy-method>`标签实现功能一致
>
> 一个是Bean创建并赋值完成后调用，一个是容器关闭（单例）或GC（多例）

------

1、`@Bean`指定类中的`init-method`和`destroy-method`方法。注意scope不同时区别。。

```java
@Bean(initMethod="init",destroyMethod="detory")
public Car car(){
    return new Car();
}
```

------

2、通过让Bean实现`InitializingBean`（重写初始化方法afterPropertiesSet），`DisposableBean`（重写销毁方法destroy）

------

3、JSR250定义的注解，只能定义在**方法上**

- `@PostConstruct`：在bean创建完成并且赋值完成；来执行初始化方法
- `@PreDestroy`：在销毁bean之前通知我们进行清理工作。单例中需要关闭容器（使用实现类的方法，如close）

```java
@Component
public class Dog {
	public Dog(){
		System.out.println("dog constructor...");
	}

	@PostConstruct
	public void init(){
		System.out.println("Dog....@PostConstruct...");
	}

	@PreDestroy
	public void detory(){
		System.out.println("Dog....@PreDestroy...");
	}
}
```

------

4、Spring提供的`BeanPostProcessor`接口：bean的后置处理器，在**所有bean初始化前后**进行一些处理工作

* `postProcessBeforeInitialization`：在自定义初始化之前工作
* `postProcessAfterInitialization`：在自定义初始化之后工作

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
		return bean;
	}
}
```

> 源码：
>
> 遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，一但返回null，跳出for循环，不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
>
> BeanPostProcessor原理：
>
> populateBean(beanName, mbd, instanceWrapper); //给bean进行属性赋值
>
> initializeBean：
>
> ​	applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
>
> ​	invokeInitMethods(beanName, wrappedBean, mbd); //执行自定义初始化
>
> ​	applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
>
> Spring底层对 BeanPostProcessor 的使用：bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async等等





#### 7、属性赋值

**`@Value`**：用于注入基本类型和String数据

- 属性：`value`指定数据的值。

  - 基本类型和String

  - 使用SpEL（写法：`#{表达式}`）

  - 可以取出配置文件如properties中的值（也存在于运行环境变量里面）：`${}`，结合`@PropertySource注解`

    ```java
    public class Person {
        @Value("张三")
        private String name;
        @Value("#{20-2}")
        private Integer age;
        @Value("${person.nickName}")
        private String nickName;
        ......
    }
    ```

    ```java
    ConfigurableEnvironment environment = applicationContext.getEnvironment();
    String property = environment.getProperty("person.nickName");
    ```



#### 7、自动装配

> 作用和xml配置文件中的`<bean>`标签编写`<property>`标签实现功能一致。利用DI完成依赖关系赋值

- **`@Autowired`**

  ==自动**按照类型注入**==

  首先按照类型去找，只要容器中**有唯一一个**bean对象类型和要注入的变量**类型匹配**，就注入成功，一个都没有则报错

  如果有**多个类型匹配到**，使用要注入的对象**变量名称**作为bean的id，在spring容器查找，找到了也可以注入成功，否则报错

  * 出现的位置：**属性**，方法（get、set），构造器，参数
    * 标注在方法的参数位置：@Bean+方法参数；参数从容器中获取；@Autowired可以**省略**
    * 标在构造器上：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以**省略**

  - 可以使用`required=false`指定后，注入失败也不会报错（了解）
  - `@Primary`：在Bean对象添加此注解，则首先注入这个Bean。若`@Qualifier`指定了，则这个注解失效

- **`@Qualifier`**

  在自动按照类型注入的**基础之上**，**再按照 Bean 的 id 注入**

  注意：在给**类成员注入时不能单独使用**，但是**给方法参数注入时可以单独使用**

  - 属性：`value`用于指定注入的bean的id

- **`@Resource`**JaveEE（JSR250）的注解，耦合性低

  可以和@Autowired一样实现自动装配功能，默认是**按照组件名称进行装配的**，找不到再看类型装配。可以独立使用。

  没有能支持@Primary功能没有支持@Autowired（reqiured=false）;

  - 属性：`name`用于指定注入的bean的id

- **`@Inject`**：JaveEE（JSR330）的注解，耦合性低

  - 需要导入`javax.inject`依赖。和@Autowired的功能一样，只是没有required=false的功能

  以上四个注解都**只能注入其他bean类型数据**，而基本类型和String无法注入。集合类型注入只能用xml、javaConfig实现

> AutowiredAnnotationBeanPostProcessor：解析完成自动装配功能（以上四个）；	



* 自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）

  * 自定义组件需要实现xxxAware（参考Aware接口设计）：在创建对象的时候，会调用接口规定的方法注入相关组件
  * xxxAware：功能实现使用的是xxxProcessor，如ApplicationContextAware->ApplicationContextAwareProcessor

  ```java
  @Component
  public class Red implements ApplicationContextAware,BeanNameAware,EmbeddedValueResolverAware {
  	
  	private ApplicationContext applicationContext;
  
  	@Override
  	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
  		System.out.println("传入的ioc："+applicationContext);
  		this.applicationContext = applicationContext;
  	}
  
  	@Override
  	public void setBeanName(String name) {
  		System.out.println("当前bean的名字："+name);
  	}
  
  	@Override
  	public void setEmbeddedValueResolver(StringValueResolver resolver) {
  		String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
  		System.out.println("解析的字符串："+resolveStringValue);
  	}
  }
  ```

* 



注意：`@Qualifier`给方法参数注入时可以单独使用

```java
//@Configuration
@ComponentScan("com.itheima")
@Import(JdbcConfig.class)
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {

}
```

```java
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    /**
     * 用于创建一个QueryRunner对象
     * @param dataSource
     * @return
     */
    @Bean(name="runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(@Qualifier("ds2") DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    /**
     * 创建数据源对象
     * @return
     */
    @Bean(name="devDataSource")
    public DataSource createDataSource(){
        try {
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setDriverClassName(driver);
            dataSource.setUrl(url);
            dataSource.setUsername(username);
            dataSource.setPassword(password);
            return dataSource;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    @Bean(name="testDataSource")
    public DataSource createDataSource1(){
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClassName(driver);
            ds.setJdbcUrl("jdbc:mysql://localhost:3306/eesy02");
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```

```properties
#jdbcConfig.properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy
jdbc.username=root
jdbc.password=1234
```

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(com.itheima.config.SpringConfiguration.class);
JdbcTemplate jdbcTemplate = ac.getBean("runner", QueryRunner.class);
.....
```



**注解和 XML的选择问题** 

- 注解的优势：  配置简单，维护方便（我们找到类，就相当于找到了对应的配置）
- XML的优势：  修改时，不用改源码。不涉及重新编译和部署

当bean来自第三方，可以使用xml或JavaConfig；当bean的实现类由用户自己开发，则使用注解









## 3 AOP

AOP(Aspect Oriented Programming)：**面向切面编程**

* 作用：**在程序运行期间，不修改源码对已有方法进行增强**。 
* 优势：
  * 减少重复代码     
  * 提高开发效率     
  * 维护方便
* 实现方式：**动态代理**

------

引入

之前使用`DBUtils`做的增删改查例子中，客户的业务层实现类存在的问题：**事务被自动控制了**。换言之，我们使用了 connection 对象的 `setAutoCommit(true)`  此方式控制事务，如果我们每次都执行一条 sql 语句，没有问题，但是如果业务方法一次要执行多条 sql 语句，这种方式就无法实现功能了。 

日志记录需要给每个方法都添加记录的方法。。。

权限控制需要给每个方法都添加权限校验。。。

……

------

**AOP相关术语**

* Joinpoint(连接点)：所谓连接点是指那些被拦截到的点。在 spring 中这些点指的是方法，因为 spring 只支持方法类型的连接点。 
* Pointcut(**切入点**)：所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。即**被增强的连接点**
* Advice(**通知**/增强):   所谓通知是指拦截到 Joinpoint 之后所**要做的事情**就是通知。   
  * 通知的类型：前置通知、后置通知、异常通知、最终通知、环绕通知。 根据`method.invoke`之前、之后、catch中、finally中，以及实现类重写的`invoke`方法在执行（环绕通知中有明确的切入点方法调用）
* Introduction(引介):   引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field
* Target(目标对象):   代理的目标对象
* Weaving(织入):   是指把增强应用到目标对象来创建新的代理对象的**过程**。spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。 
* Proxy（代理）：一个类被 AOP 织入增强后，就产生一个结果代理类
* Aspect(**切面**):   是**切入点和通知（引介）的结合**。 

------

**学习AOP要明确的事**

* 开发阶段（我们做的）
  * 编写核心业务代码（开发主线）：大部分程序员来做，要求熟悉业务需求。  
  * 把公用代码抽取出来，制作成通知。（开发阶段最后再做）：AOP 编程人员来做。  
  * 在配置文件中，声明切入点与通知间的关系，即切面。：AOP 编程人员来做
* 运行阶段（Spring框架完成的）
  * Spring 框架监控切入点方法的执行。一旦监控到切入点方法被运行，使用代理机制，动态创建目标对象的代理对象，根据通知类别，在代理对象的对应位置，将通知对应的功能织入，完成完整的代码逻辑运行。 



### 3.1 基于xml的AOP配置

1. 在pom.xml中添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.1</version>
   </dependency>
   ```

2. 在spirng-config.xml中添加AOP约束。在指南中搜索`xmlns:aop`即可

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd">
   </beans>
   ```

3. 配置 spring 的 ioc 

4. **==抽取公共代码制作成通知==** 

5. spring中基于XML的**==AOP配置==**步骤

   1. 把**通知Bean也交给spring来管理**

   2. 使用`aop:config`标签表明开始**AOP的配置**

   3. 使用`aop:aspect`标签表明配置**切面**

      - `id`属性：是给切面提供一个唯一标识
      - `ref`属性：是指定通知类bean的Id

   4. 在`aop:aspect`标签的内部使用对应标签来配置**通知的类型**。示例是让printLog方法在切入点方法执行之前之前

      - `aop:before`：表示配置前置通知。其他通知类似
        - `method`属性：用于指定Logger类中**哪个方法是前置通知**
        - `pointcut`属性：用于指定切入点表达式，该表达式的含义指的是对业务层中**哪些方法增强**
        - `ponitcut-ref`属性：用于指定切入点的表达式的引用 

   5. **切入点表达式**的写法：关键字：`execution(表达式)`

      * 表达式`访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)`

      * 标准写法`public void com.itheima.service.impl.AccountServiceImpl.saveAccount()`

      * 访问修饰符可以省略`void com.itheima.service.impl.AccountServiceImpl.saveAccount()`

      * 返回值可以使用通配符，表示任意返回值

        `* com.itheima.service.impl.AccountServiceImpl.saveAccount()`

      * 包名可以使用通配符，表示任意包。但是有几级包，就需要写几个`*.`

        `* *.*.*.*.AccountServiceImpl.saveAccount()`

      * 包名可以使用`*..`表示当前包及其子包
        `* *..AccountServiceImpl.saveAccount()`

      * 类名和方法名都可以使用`*`来实现通配

        `* *..*.*()`

      * 参数列表：

        * 可以直接写数据类型：
          * 基本类型直接写名称           int
          * 引用类型写包名.类名的方式   java.lang.String
        * 可以使用通配符表示任意类型，但是必须有参数
        * 可以使用..表示有无参数均可，有参数可以是任意类型

      * 全通配写法：`* *..*.*(..)`

      * **实际开发中切入点表达式的通常写法**：**切到业务层实现类下的所有方法**

        `* com.itheima.service.impl.*.*(..)`

      ```xml
      <!-- 配置Logger类 -->
      <bean id="logger" class="com.itheima.utils.Logger"></bean>
      
      <!--配置AOP-->
      <aop:config>
          <!-- 配置切入点表达式 id属性用于指定表达式的唯一标识。expression属性用于指定表达式内容
                    此标签写在aop:aspect标签内部只能当前切面使用。
                    它还可以写在aop:aspect外面，此时就变成了所有切面的通知可可使用此切点  -->
          <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
          
          <!--配置切面 -->
          <aop:aspect id="logAdvice" ref="logger">  <!--引入通知-->
              <!-- 配置前置通知：在切入点方法执行之前执行
              <aop:before method="beforePrintLog" pointcut-ref="pt1" ></aop:before>-->
      
              <!-- 配置后置通知：在切入点方法正常执行之后值。它和异常通知永远只能执行一个
              <aop:after-returning method="afterReturningPrintLog" pointcut-ref="pt1"></aop:after-returning>-->
      
              <!-- 配置异常通知：在切入点方法执行产生异常之后执行。它和后置通知永远只能执行一个
              <aop:after-throwing method="afterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>-->
      
              <!-- 配置最终通知：无论切入点方法是否正常执行它都会在其后面执行
              <aop:after method="afterPrintLog" pointcut-ref="pt1"></aop:after>-->
      
              <!-- 配置环绕通知 详细的注释请看Logger类中。通常情况下，环绕通知都是独立使用的 -->
              <aop:around method="aroundPringLog" pointcut-ref="pt1"></aop:around>
          </aop:aspect>
      </aop:config>
      ```

      ```java
      /**
       * 用于记录日志的工具类，它里面提供了公共的代码
       */
      public class Logger {
      
      	//前置通知
          public  void beforePrintLog(){
              System.out.println("前置通知Logger类中的beforePrintLog方法开始记录日志了。。。");
          }
      	//后置通知
          public  void afterReturningPrintLog(){
              System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志了。。。");
          }
          //异常通知
          public  void afterThrowingPrintLog(){
              System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志了。。。");
          }
          //最终通知
          public  void afterPrintLog(){
              System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志了。。。");
          }
      
          /**
           * 环绕通知
           * 问题：
           *      当我们配置了环绕通知之后，切入点方法没有执行，而通知方法执行了。
           * 分析：
           *      通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码中没有。
           * 解决：
           *      Spring框架为我们提供了一个接口：ProceedingJoinPoint。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。
           *      该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。
           *
           * spring中的环绕通知：
           *      它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。
           */
          public Object aroundPringLog(ProceedingJoinPoint pjp){
              Object rtValue = null;
              try{
                  Object[] args = pjp.getArgs();//得到方法执行所需的参数
      
                  System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");
      
                  rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）
      
                  System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");
      
                  return rtValue;
              }catch (Throwable t){
                  System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
                  throw new RuntimeException(t);
              }finally {
                  System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
              }
          }
      }
      ```

------

要想使用**注解的方式**使用Spring AOP就**必须要有源码**(因为我们要在切面类上添加注解)。如果没有源码的话，我们就得使用XML来声明切面了

| AOP配置元素                | 用途                                            |
| -------------------------- | ----------------------------------------------- |
| **< aop:config>**          | 顶层的AOP配置元素                               |
| **< aop:aspect>**          | 定义一个切面                                    |
| **< aop:pointcut>**        | 定义一个切点                                    |
| < aop:advesor>             | 定义AOP通知器                                   |
| < aop:before>              | 定义AOP前置通知                                 |
| < aop:after>               | 定义AOP后置通知（无论被通知的方法是否执行成功） |
| < aop:after-returning>     | 定义AOP返回通知                                 |
| < aop:after-throwing>      | 定义AOP异常通知                                 |
| < aop:around>              | 定义AOP环绕通知                                 |
| < aop:aspect-autoproxy>    | 启用@AspectJ注解驱动的切面                      |
| **< aop:declare-parents>** | 以透明的方式为被通知的对象引入额外的接口        |





### 3.2 基于注解的AOP配置

在spring-config.xml**开启对注解的支持**，并添加AOP依赖，告知Spring**创建容器时扫描的包**，开启注解

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置spring创建容器时要扫描的包-->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!-- 配置spring开启注解AOP的支持 -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

添加**`@Component`、`@Aspect`**注解，配置切面（切点、通知）

```java
/**
 * 用于记录日志的工具类，它里面提供了公共的代码
 */
@Component("logger")
@Aspect//表示当前类是一个切面类
public class Logger {

    @Pointcut("execution(* com.itheima.service.impl.*.*(..))")
    private void pt1(){}

    //如下注解中，value和pointCut含义一样
    //以下通知的方法的参数列表中，JoinPoint必须出现在参数列表的第一位
    
    //前置通知
    @Before("pt1()")
    public  void beforePrintLog(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"@Before:参数列表是：{"+Arrays.asList(args)+"}");
    }
    //后置通知
    @AfterReturning(value="pt1()",,returning="result")
    public  void afterReturningPrintLog(JoinPoint joinPoint,Object result){
       System.out.println(joinPoint.getSignature().getName()+"==@AfterReturning:运行结果：{"+result+"}");
    }
    //异常通知
    @AfterThrowing(value="pt1()",throwing="throwing")
    public  void afterThrowingPrintLog(JoinPoint joinPoint,Throwable t){
        System.out.println(joinPoint.getSignature().getName()+"异常。。。异常信息：{"+t+"});
    }
    //最终通知
    @After("pt1()")
    public  void afterPrintLog(JoinPoint joinPoint){
        System.out.println("joinPoint.getSignature().getName()+"结束。。。@After");
    }
 
}
```

```java
/** 省略上面代码 **/
/**
     * 环绕通知
     * 问题：
     *      当我们配置了环绕通知之后，切入点方法没有执行，而通知方法执行了。
     * 分析：
     *      通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码中没有。
     * 解决：
     *  Spring框架为我们提供了一个接口：ProceedingJoinPoint。该接口有一个方法proceed()，此方法就相当于明确调用切入点方法。
     *  该接口可以作为环绕通知的方法参数，在程序执行时，spring框架会为我们提供该接口的实现类供我们使用。
     *
     * spring中的环绕通知：
     *      它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。
     */
//若是外部类引用切点，则需要全类名+方法（但好像没意义，就两行代码）
@Around(value="pt1()") 
public Object aroundPringLog(ProceedingJoinPoint pjp){
    Object rtValue = null;
    try{
        Object[] args = pjp.getArgs();//得到方法执行所需的参数

        System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。前置");

        rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）

        System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。后置");

        return rtValue;
    }catch (Throwable t){ //只能catch Throwable 
        System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。异常");
        throw new RuntimeException(t);
    }finally {
        System.out.println("Logger类中的aroundPringLog方法开始记录日志了。。。最终");
    }
}
```

注意：基于配置的AOP，**最终通知会在后置通知或异常通知之前执行**。环绕通知没有此问题，推荐使用

`ProceedingJoinPoint`接口继承了`JoinPoint`接口。比它多了`proceed`的无参和有参方法，用于执行JointPoint（连接点）

* `Signature getSignature()`：获取签名，即方法的签名
  * `String getName()`：获取方法的名称
* `Object[] getArgs()`：获取实际参数列表
* `Object getTarget()`：获取目标对象（被代理对象？）
* `Object getThis()`：获取代理对象

### 3.3 Spring纯注解配置(JavaConfig)

spring-config.xml文件替换为如下配置类

**`@EnableAspectJAutoProxy`**

```java
@Configuration 
@ComponentScan(basePackages="com.itheima") 
@EnableAspectJAutoProxy 
public class SpringConfiguration {
}
```

切面类同上



#### 注解AOP原理

看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？

1. **`@EnableAspectJAutoProxy`**是什么？
   1. 该注解上有`@Import(AspectJAutoProxyRegistrar.class)`注解：给容器中导入AspectJAutoProxyRegistrar，利用它自定义给容器中注册bean；



未完……待续……



## 4 Spring的声明式事务

### 4.1 明确要点

- JavaEE 体系进行分层开发，**==事务处理位于业务层==**，Spring 提供了分层设计业务层的事务处理解决方案。 
- Spring 框架为我们提供了一组事务控制的**接口**，在 `spring-tx-5.*.*.RELEASE.jar` 中。具体在后面的第二小节介绍。
- Spring的事务控制都是基于 AOP 的，它既可以使用编程的方式实现，也可以使用**配置的方式实现**（重点）。

### 4.2  Spring中事务控制的 API介绍 

PlatformTransactionManager：此接口是 spring 的事务管理器，它里面提供了我们常用的操作事务的方法，如下：

- 获取事务状态信息：`TransactionStatus getTransaction(TransactionDefinition definition)`
- 提交事务：`void commit(TransactionStatus status)`
- 回滚事务：`void rollback(TransactionStatus status)`

我们在开发中都是使用它的实现类（**真正管理事务的对象** ）：

- **`org.springframework.jdbc.datasource.DataSourceTransactionManager`**：**Spring JDBC或iBatis持久化数据使用**
- `org.springframework.orm.hibernate5.HibernateTransactionManager`：用 Hibernate 版本进行持久化数据使用

------

TransactionDefinition：是事务的定义信息对象，里面有如下方法：

- 获取事务对象名称：`String getName()`
- 获取事务隔离级别：`int getIsolationLevel()`，采用数据库的默认级别
- 获取事务**传播行为**：`int getPropagationBehavior()`
  - **REQUIRED**：如果当前没有事务就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选择(默认值)
  - **SUPPORTS**：支持当前事务，如果当前没有事务，就以非事务方式执行(没有事务) 
  - MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常
  - REQUERS_NEW:新建事务，如果当前在事务中，把当前事务挂起
  - NOT_SUPPORTED:以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
  - NEVER:以非事务方式运行，如果当前存在事务，抛出异常
  - NESTED:如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行 REQUIRED 类似的操作
- 获取事务超时时间：`int getTimeout()`，默认值是-1，没有超时限制。如果有，以秒为单位进行设置
- 获取事务**是否只读**：`boolean isReadOnly()`，建议查询时设置为只读。 

------

TransactionStatus：此接口提供的是**事务具体的运行状态**，方法介绍如下：

- 刷新事务：`void flush()`
- 获取是否存在存储点：`boolean hasSavepoint()`
- 获取事务是否完成：`boolean isCompleted()`
- 获取事务是否为新的事务：`boolean isNewTransaction()`
- 获取事务是否回滚：`boolean isRollbackOnly()`
- 设置事务回滚：`void setRollbackOnly()`



### 4.3 基于xml的声明式事务控制

1. 在pom.xml中添加依赖坐标

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>5.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-tx</artifactId>
       <version>5.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.1</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-jdbc</artifactId>
       <version>5.1.1.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.12</version>
   </dependency>
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.12</version>
   </dependency>
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-test</artifactId>
       <version>5.1.1.RELEASE</version>
   ```

2. spring中基于XML的声明式事务控制配置步骤

   1. 配置**事务管理器**
   2. 配置**事务的通知**引用事务管理器 
      此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
      使用`tx:advice`标签配置事务通知
      属性：
      ​        `id`：给事务通知起一个唯一标识
      ​        `transaction-manager`：给事务通知提供一个事务管理器引用
   3. 配置**事务的属性**（在上述`tx:advice`中）
   4. 配置AOP中的通用**切入点**表达式
   5. 建立事务**通知和切入点表达式的对应关系**

   ```xml
   <!-- 配置事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource"></property>
   </bean>
   
   <!-- 配置事务的通知-->
   <tx:advice id="txAdvice" transaction-manager="transactionManager">
       <!-- 配置事务的属性
         isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
         propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
         read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
         timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
     rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
    no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
       -->
       <tx:attributes>
           <tx:method name="*" propagation="REQUIRED" read-only="false"/>
           <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
       </tx:attributes>
   </tx:advice>
   
   <!-- 配置aop-->
   <aop:config>
       <!-- 配置切入点表达式-->
       <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"></aop:pointcut>
       <!--建立切入点表达式和事务通知的对应关系 -->
       <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
   </aop:config>
   ```



### 4.4 基于注解的声明式事务控制

> aop约束，注解约束、事务约束。配置spring创建容器时要扫描的包。配置JdbcTemplate、配置数据源

1. 在pom.xml中添加依赖坐标（同xml）

2. spring中基于注解 的声明式事务控制配置步骤

   1. 配置事务管理器
   2. 开启spring对注解事务的支持
   3. 在需要事务支持的地方使用@Transactional注解

   ```xml
   <!-- 配置事务管理器 -->
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource"></property>
   </bean>
   
   <!-- 开启spring对注解事务的支持-->
   <tx:annotation-driven transaction-manager="transactionManager"/>
   ```

   ```java
   /**
    * 账户的业务层实现类
    * 事务控制应该都是在业务层
    */
   @Service("accountService")
   @Transactional(propagation= Propagation.SUPPORTS,readOnly=true)//只读型事务的配置
   public class AccountServiceImpl implements IAccountService{
   
       @Autowired
       private IAccountDao accountDao;
   
       @Override
       public Account findAccountById(Integer accountId) {
           return accountDao.findAccountById(accountId);
   
       }
       
       @Transactional(propagation= Propagation.REQUIRED,readOnly=false)//需要的是读写型事务配置，默认为那俩参数
       @Override
       public void transfer(String sourceName, String targetName, Float money) {
           System.out.println("transfer....");
           //2.1根据名称查询转出账户
           Account source = accountDao.findAccountByName(sourceName);
           //2.2根据名称查询转入账户
           Account target = accountDao.findAccountByName(targetName);
           //2.3转出账户减钱
           source.setMoney(source.getMoney()-money);
           //2.4转入账户加钱
           target.setMoney(target.getMoney()+money);
           //2.5更新转出账户
           accountDao.updateAccount(source);
   
           int i=1/0;
           //2.6更新转入账户
           accountDao.updateAccount(target);
       }
   }
   ```

   注意：比xml配置麻烦的一处在于对Service中多个方法的事务注解，除了类上的只读事务配置，其他读写事务都得一个个配



### 4.5 Spring纯注解事务配置(JavaConfig)

注册事务管理器：MyBatis、JdbcTemplate使用`PlatformTransactionManager`接口下的`DataSourceTransactionManager`实现类，传入`DataSource`

**`@EnableTransactionManagement`、`Transactional`**

```java
@Configuration
@ComponentScan("com.itheima")
@Import({JdbcConfig.class,TransactionConfig.class})
@PropertySource("classpath:jdbcConfig.properties")
@EnableTransactionManagement
public class SpringConfiguration {

}
```

```java
//连接数据库相关的配置类
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DriverManagerDataSource dataSource = new DriverManagerDataSource();//使用spring提供的DataSource
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```

```java
//和事务相关的配置类
public class TransactionConfig {
    //创建事务管理器对象
    @Bean 
    public PlatformTransactionManager transactionManager (DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

```properties
#jdbcConfig.properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql:///bxgaccount?serverTimezone=GMT%2B8
jdbc.username=root
jdbc.password=w111151
```

AccountServiceImpl和6.4中一致





## 5 Spring整合JUnit

> 在测试类中，每个测试方法都有以下两行代码：  
>
> `ApplicationContext ac = new ClassPathXmlApplicationContext("spring-config.xml");`
>
> `AccountService as = ac.getBean("accountService",IAccountService.class); `
>
> 这两行代码的作用是获取容器，如果不写的话，直接会提示空指针异常，所以又不能轻易删掉

分析：

1. 应用程序的入口：`main`方法

2. JUnit单元测试中，没有`main`方法也能执行是由于JUnit集成了一个运行器。执行带有@Test注解的方法

3. JUnit不会管我们是否采用spring框架，也就不会为我们读取配置文件/配置类创建spring核心容器

   所以当测试方法执行时，没有IoC容器，就算写了@Autowired注解，也无法实现注入

------

解决：JUnit给我们暴露 了一个注解，可以让我们替换掉它的运行器

1. 在pom.xml中添加Spring-test依赖

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-test</artifactId>
       <version>5.1.3.RELEASE</version>
       <!-- <scope>test</scope> -->
   </dependency>
   ```

2. 使用**`@RunWith`**注解**替换原有运行器** 

3. 使用**`@ContextConfiguration`** 指定 spring 配置文件的位置 

   - 属性：
     - `locations`：用于指定**配置文件**的位置。如果是类路径下，需要用 `classpath:`表明 
     - `classes`：用于指定**注解的类**。当不使用 xml 配置时，需要用此属性指定注解类的位置

4. 使用**`@Autowired`**给测试类中的变量注入数据（其他注解也行）

   ```java
   @RunWith(SpringJUnit4ClassRunner.class) 
   //@ContextConfiguration(locations= "classpath:spring-config.xml") 
   @ContextConfiguration(classes = com.itheima.config.SpringConfiguration.class)
   public class AccountServiceTest {
   	@Autowired 
   	private AccountService as ; 
   }
   ```

注意：`SpringJUnit4ClassRunner`要求的JUnit版本在**4.12及以上**





## 6 Spring的JdbcTemplate

* pom.xml中导入依赖

  ```xml
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.1.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.1.1.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.1.1.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.12</version>
  </dependency>
  ```

* 配置类

  ```java
  @Configuration
  @ComponentScan("com.itheima")
  public class SpringConfiguration {
  
      @Bean(name = "jdbcTemplate")
      public JdbcTemplate jdbcTemplate(DataSource dataSource){
          return new JdbcTemplate(dataSource);
      }
  
      @Bean
      public DataSource dataSource(){
          DriverManagerDataSource dataSource = new DriverManagerDataSource();//使用spring提供的DataSource
          dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
          dataSource.setUrl("jdbc:mysql:///bxgaccount?serverTimezone=GMT%2B8");
          dataSource.setUsername("root");
          dataSource.setPassword("w111151");
          return dataSource;
      }
  }
  ```

* **调用JdbcTemplate的方法**来完成CRUD的操作（查找语句若未找到，直接抛异常，不是返回null，所以要**try...catch...**）

  - `update()`：执行**DML**语句。增、删、改语句
  - `query()`：查询并将结果封装为==**JavaBean对象的List集合**==。JavaBean属性使用**包装类型**最好，可以存储null。
    - query的参数：RowMapper函数式接口
      - 一般我们使用BeanPropertyRowMapper实现类。可以完成数据到JavaBean的自动封装
      - **`new BeanPropertyRowMapper<>(类型.class)`**（不用在尖括号中添加泛型）
  - `queryForObject()`：查询并将结果封装为==**对象**==。
    - 参数同`query()`的参数可封装为==**JavaBean对象**==；参数传**包装类型**，可用于聚合函数的查询。
  - `queryForMap()`：查询并将结果的列名作为key(String)、值作为value(Object)封装为一个==**Map**==集合
    - 注意：这个方法查询的结果集长度只能是1
  - `queryForList()`：查询并将结果集封装为==**List**==集合
    - 注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中

* Dao层

  ```java
  @Repository("accountDao")
  public class AccountDaoImpl implements AccountDao {
      @Autowired
      private JdbcTemplate jdbcTemplate;
  
      @Override
      public void addAccount(Account a) {
          jdbcTemplate.update("insert into account (name,money) values(?,?)", a.getName(), a.getMoney());
      }
  }
  ```

* JUnit测试（其他方法可以查看数据库中笔记）

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(classes = com.itheima.config.SpringConfiguration.class)
  public class JdbcTemplateTest {
  
      @Autowired
      AccountDao accountDao;
  
      @Test
      public void test(){
          Account account = new Account("xxx",888);
          accountDao.addAccount(account);
      }
  }
  ```

* 当有多个Dao时，依赖注入会重复写在多个Dao中。此时Spring提供了**`JdbcDaoSupport`类**，可以直接**继承**。配置中也**只需要配置DataSource即可**，这个类会**自动根据DataSource创建JdbcTemplate**。由于JdbcTemplate是私有的，所以可以使用`getJdbcTemplate()`方法来获取。但是**只能xml配置时使用**，注解时由于JdbcTemplate上不能添加注入，非常麻烦。

  ```java
  public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao { }
  ```

















# 第四部分 Spring MVC

> 三层架构
>
> * 表现层（web）：接收客户端请求，向客户端响应结果。包括控制层(接收请求)、展示层(展示结果)，采用MVC模型 
> * 业务逻辑层（service）：处理具体的业务逻辑，包括事务
> * 持久层（dao）：数据持久化，即和数据库交互，对数据库表进行增删改查 
>
> MVC模型（Model View Controller），即模型视图控制器，每个部分各司其职。 
>
> * Model：数据模型，JavaBean的类，用来进行数据封装
> * View：指jsp或者html。作用一般就是展示数据的。通常视图是依据模型数据创建的
> * Controller：是应用程序中处理用户交互的部分，作用一般就是处理程序逻辑的，整个流程的控制器。可以数据校验等

SpringMVC概述

* 是一种基于**Java**实现的MVC设计模型的**请求驱动**类型的**轻量级WEB框架**。 
* Spring MVC属于SpringFrameWork的后续产品，已经**融合在Spring Web Flow**里面。Spring 框架提供 了构建 Web 应用程序的全功能 MVC 模块。 
* 使用 Spring **可插入的 MVC 架构**，从而在使用Spring进行WEB开发时，可以选择使用Spring的 SpringMVC框架或集成其他MVC开发框架，如Struts1(现在一般不用)，Struts2等。 

SpringMVC 和 Struts2 的优劣分析

* 共同点：  
  * 它们都是表现层框架，都是基于 MVC 模型编写的。  
  * 它们的底层都离不开原始 ServletAPI。  
  * 它们处理请求的机制都是一个核心控制器。 
* 区别：
  * Spring MVC 的入口是 Servlet, 而 Struts2 是 Filter 
  * Spring MVC 是基于方法设计的，而 Struts2 是基于类，Struts2 每次执行都会创建一个动作类，比Spring MVC会稍慢
  * Spring MVC 使用更加简洁,同时还支持 JSR303, 处理 ajax 的请求更方便 
  * Struts2 的 OGNL 表达式使页面的开发效率相比 Spring MVC 更高些，但执行效率并没有比 JSTL 提 升，尤其是 struts2 的表单标签，远没有 html 执行效率高



## 1 入门

### 1.1 入门程序

* pom.xml中添加依赖的坐标

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.1.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.1.1.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.1.1.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

------

- **`web.xml`**或JavaConfig配置**前端控制器`DispatcherServlet`**，拦截请求到Controller层（自己编码）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 配置SpringMVC前端控制器 -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置Servlet的初始化参数，读取spring的xml配置文件或JavaCOnfig，创建spring容器 -->
        <!-- 可以不配置，那就默认找/WEB-INF/[servlet的名称]-servlet.xml -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-config.xml</param-value>
        </init-param>
        
        <!-- 配置servlet启动时加载对象，参数必须为整数
		    1、为0或者大于0，则springMVC随着容器启动而启动；（越小权限越大）
             2、小于0，则在第一次请求进来的时候启动 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <!-- 1. /* 拦截所有 jsp js png .css 建议不使用。除非用在Filter中（若用在servlet中请求jsp会报404）
             2. *.action *.do 拦截以do action 结尾的请求 肯定能使用 ERP 
             3. / 拦截所有（只不包括jsp) 强烈建议使用 前台 面向消费者 需要配置对静态资源放行 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
    
    <!-- 以下入门不使用 -->
        <!-- 指定Spring配置文件(下部分代码由IDEA自动生成)-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/applicationContext.xml</param-value>
    </context-param>
    <!-- 配置监听器加载spring配置文件 -->
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
    
    <!-- 配置过滤器，解决post的乱码问题 -->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <!-- 是否强制Request、Response编码 -->    
        <init-param>     
            <param-name>forceEncoding</param-name>   
            <param-value>true</param-value>    
        </init-param> 
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern> <!--Filter中推荐使用/*，拦截所有-->
    </filter-mapping>
    
    
    <!--释放静态资源方式1：配置DefaultServlet的静态资源url，直接过滤静态资源 
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/css/*</url-pattern>
        <url-pattern>/img/*</url-pattern>
        <url-pattern>/plugins/*</url-pattern>
        <url-pattern>/templates/*</url-pattern>
    </servlet-mapping>  -->
    
</web-app>
```

------

* **`spring-config.xml`**或**`SpringConfiguration`**配置类。开启扫描**@Controller、@Service**等

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans    http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.itcast"/>
    
    <!-- 处理器映射器 从spring3.1版本开始默认的废除了，所以配置新的-->
    <!-- <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/> -->
    <!-- 处理器适配器 从spring3.1版本开始默认的废除了，所以配置新的-->
    <!--<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/> -->
    
    <!-- 配置spring开启注解mvc的支持  替代处理器映射器，处理器适配器的配置-->
    <mvc:annotation-driven/>
    
    <!-- 配置视图解释器 省略前后缀-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
    
    <!--（推荐）释放静态资源方式3：将SpringMVC处理不了的请求交给Tomcat，静态资源就可以访问了-->
    <mvc:default-servlet-handler/>

    <!--释放静态资源方式2： location表示路径，mapping表示文件，**表示该目录下的文件以及子目录的文件 
    <mvc:resources mapping="/static/**" location="/static/"/>   静态资源放入static目录即可只写这一句
    <mvc:resources mapping="/css/**" location="/css/" />
    <mvc:resources mapping="/images/**" location="/images/"/>
    <mvc:resources mapping="/js/**" location="/js/" /> 
    -->

</beans>
```

```java
@Configuration
@ComponentScan("cn.itcast")
@EnableWebMvc
public class SpringConfiguration {

    @Bean
    public InternalResourceViewResolver internalResourceViewResolver(){
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

* **Controller层**

```java
@Controller
public class HelloController {

    @RequestMapping("/hello")
    public String sayHello(){
        System.out.println("HelloController 的 sayHello 方法执行了。。。。");
        return "success";
    }
}
```



### 1.2 执行过程分析

1. 当启动Tomcat服务器的时候，因为配置了`load-on-startup`标签，所以会创建`DispatcherServlet`对象， 并分局初始化参数加载`spring-config.xml`配置文件或`SpringConfiguration`配置类 
2. 由于开启了注解扫描，那么`HelloController`对象就会被创建
3. 从index.jsp发送请求，请求会先到达`DispatcherServlet`前端控制器，根据配置`@RequestMapping`注解找到执行的方法
4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件 
5. Tomcat服务器渲染页面，做出响应 



### 1.3 入门案例中组件分析

在 SpringMVC 的各个组件中，**处理器映射器**、**处理器适配器**、**视图解析器**称为 SpringMVC 的三大组件。 

我们只需要编写处理具体业务的**控制器**以及**视图**

------

* DispatcherServlet

前端控制器：用户请求到达前端控制器，它就相当于 mvc 模式中的 c，dispatcherServlet 是整个流程控制的中心，由 它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性

* HandlerMapping

处理器映射器：HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的 映射方式，例如：配置文件方式，实现接口方式，注解方式等。 

* Handler

它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理。

* HandlAdapter

通过 HandlerAdapter 对处理器进行执行，这是**适配器模式**的应用，通过扩展适配器可以对更多类型的处理器进行执行

* View Resolver

View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名 即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户

* View

SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView 等。最常用的视图就是jsp

一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面



## 2 `@RequestMapping`

作用：用于**建立请求 URL** 和**处理请求方法**之间的对应**关系**

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};
    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};
    String[] params() default {};
    String[] headers() default {};
    String[] consumes() default {};
    String[] produces() default {};
}
```

出现位置： 

* **类**上：请求URL的**第一级访问目录**，模块化管理。此处不写的话，就相当于应用的根目录。写的话需要以`/`开头，如`/user`

* **方法**上：请求 URL 的**第二级访问目录**。 如`/add`。但是RESTful风格的可以使用`@***Mapping`替代

  当使用分级配置时，前端页面的请求路径中，要么写绝对路径即带`/`和项目名，要么写**相对路径不带`/`**（表示相对应用的根目录），否则404

属性：

* **`value`**：用于指定**请求的 URL**。它和 **path** 属性的作用是一样的。**大小写敏感**
* **`method`**：用于指定**请求的方式**，值为`RequestMethod`枚举类
* `params`：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样。如：
  * `params = {"accountName"}`，表示请求参数必须有 accountName 
  * `params = {"moeny!100"}`，表示请求参数中 money 不能是 100
* `headers`：用于限定发送的请求中必须包含某请求头



## 3 请求参数的绑定

绑定机制 

1. 表单提交的数据都是k=v格式的 `username=haha&password=123`
2. SpringMVC的参数绑定过程是把**表单提交的请求参数**，作为控制器中**方法的参数**进行绑定的 



### 3.1 基本数据类型和字符串类型

当**请求的参数名称**和**处理器形参名称**一致时（**区分大小写**）会将请求参数与形参进行绑定

注意：参数类型**推荐**使用**包装数据类型**，因为基础数据类型不可为null；布尔类型的参数，请求的参数值为true或false，1或0

```java
@RequestMapping("/edit")
public String queryItemById(Integer id) {
    System.out.println(id);
    return "success";
}
```



### 3.2 POJO类型及它的关联对象

要求表单中**参数名称**和 POJO 类的**属性名称**保持一致。并且控制器方法的**参数类型是 POJO 类型**

```java
public class Account implements Serializable { 
    private Integer id;  
    private String name;  
    private Float money;  
    private Address address;  
    //getters and setters 
} 
```

```java
public class Address implements Serializable {    
    private String provinceName;  
    private String cityName;  
    //getters and setters 
}
```

```html
<form action="account/saveAccount" method="post"> 
    账户名称：<input type="text" name="name" ><br/> 
    账户金额：<input type="text" name="money" ><br/>  
    账户省份：<input type="text" name="address.provinceName" ><br/>  
    账户城市：<input type="text" name="address.cityName" ><br/>  
    <input type="submit" value=" 保存 "> 
</form>
```

```java
@RequestMapping("/saveAccount") 
public String saveAccount(Account account) {  
    System.out.println("保存了账户。。。。"+account);  
    return "success"; 
}
```

#### 3.2.1 POST请求参数乱码问题

`web.xml`中配置过滤器。Tomcat8及之后的已经解决GET请求参数乱码问题

```xml
<!-- 配置过滤器，解决post的乱码问题 -->
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!-- 启动过滤器，好像不配置这个也行 -->    
    <init-param>     
        <param-name>forceEncoding</param-name>   
        <param-value>true</param-value>    
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



### 3.3 POJO 类中包含集合类型参数

第一种：（如下面代码）

* 要求集合类型的请求参数必须在 **POJO** 中。在表单中**请求参数名称**要和 POJO 中集合**属性名称相同**。
* 给 List 集合中的元素赋值，使用下标。 
* 给 Map 集合中的元素赋值，使用键值对。 

第二种

* 接收的请求参数是 json 格式数据。需要借助一个注解实现。 

```java
public class User implements Serializable {    
    private String username;  
    private String password;  
    private Integer age;  
    private List<Account> accountList;  
    private Map<String,Account> accountMap;    
    //getters and setters
}
```

```html
<form action="account/updateAccount" method="post">
    用户名称：<input type="text" name="username"><br/>
    用户密码：<input type="password" name="password"><br/>
    用户年龄：<input type="text" name="age"><br/>
    账户 1 名称：<input type="text" name="accountList[0].name"><br/>
    账户 1 金额：<input type="text" name="accountList[0].money"><br/>
    账户 2 名称：<input type="text" name="accountList[1].name"><br/>
    账户 2 金额：<input type="text" name="accountList[1].money"><br/>
    账户 3 名称：<input type="text" name="accountMap['one'].name"><br/>
    账户 3 金额：<input type="text" name="accountMap['one'].money"><br/>
    账户 4 名称：<input type="text" name="accountMap['two'].name"><br/>
    账户 4 金额：<input type="text" name="accountMap['two'].money"><br/>
    <input type="submit" value=" 保存 ">
</form>
```

```java
@RequestMapping("/updateAccount") 
public String updateAccount(User user) {  
    System.out.println("更新了账户。。。。"+user);  
    return "success"; 
}
```







### 3.4 Servlet原生API及默认支持类型

**控制器**(处理器)**形参**中添加如下类型的参数，处理适配器会默认识别并进行赋值

- `HttpServletRequest`：通过request对象获取请求信息
- `HttpServletResponse`：通过response处理响应信息
- `HttpSession`：通过session对象得到session中存放的对象

------

**默认支持的参数类型**

- `Model`/`ModelMap`

  - 除了`ModelAndView`以外，还可以使用`Model`(接口)来**向页面传递数据**，在参数里**直接声明Model即可**

    使用Model可以不使用ModelAndView对象，Model对象可以向页面传递数据，View对象可以使用String返回值替代。==**其本质都是使用Request域对象传递数据**==

    ```java
    model.addAttribute("item", item);
    return "itemEdit";
    ```

  - `ModelMap`是`Model`接口的实现类，也可以通过`ModelMap`向页面传递数据。效果一样，同上代码



### 3.5 自定义类型转换器

除了类型转换器，SpringMVC还提供了**注解`@DateTimeFormate`来转换日期格式**。查看5常用注解这一章。

SpringMVC还可以实现一些**数据类型自动转换**。内置转换器全都在`org.springframework.core.convert.support`包下。如String转Integer等等

如遇**特殊类型转换**要求，比如日期数据有很多种格式，SpringMVC没办法把带`-`字符串转换成日期类型，需要我们自己编写**自定义类型转换器**。步骤如下：

1. 定义一个类，**实现 Spring提供的 `Converter` 接口**，该接口有两个泛型。

   ```java
   //Converter<S, T>：  S:source,需要转换的源的类型；T:target,需要转换的目标类型
   @Componet
   public class StringToDateConverter implements Converter<String, Date> {
       @Override
       public Date convert(String source) {
           DateFormat format = null;
           try {
               if (StringUtils.isEmpty(source)) {
                   throw new NullPointerException("请输入要转换的日期");
               }
               format = new SimpleDateFormat("yyyy-MM-dd");
               Date date = format.parse(source);
               return date;
           } catch (Exception e) {
               throw new RuntimeException("输入日期有误");
           }
       }
   }
   ```

2. 在 spring配置文件中**配置类型转换器**。JavaConfig暂时不会怎么配置

   spring 配置类型转换器的机制是，将自定义的转换器注册到类型转换服务中去。 

   ```xml
   <!-- 配置类型转换器工厂 --> 
   <bean id="converterService"   class="org.springframework.context.support.ConversionServiceFactoryBean"> 
   <!--FormattingConversionServiceFactoryBean可以让SpringMVC支持和@DateTimeFormat等Spring内部自定义的转换器，建议-->
    <!-- 给工厂注入一个新的类型转换器 -->      
       <property name="converters"> 
         <array> 
          <!-- 配置自定义类型转换器 -->       
             <ref bean="stringToDateConverter"/>
         </array>      
       </property> 
   </bean>
   <!-- 配置spring开启注解mvc的支持，替代处理器映射器和处理器适配器配置，并配置类型转换器 -->
   <mvc:annotation-driven conversion-service="conversionService" />
   ```



## 4 响应数据和结果视图

### 4.1 返回String

* **逻辑视图名**

  Controller中方法返回字符串可以**指定逻辑视图名**，通过**视图解析器解析为物理视图地址**。 

  > 如：`return "success"; `指定逻辑视图名，经过视图解析器解析为 jsp 物理路径如`/WEB-INF/pages/success.jsp` 

* **`forward:`转发**：`return "forward:pages/success.jsp"; `则路径必须写成实际视图 url，不能写逻辑视图

* **`redirect:`重定向**：`return "redirect:testReturnModelAndView"; `路径可以不添加项目名称，会自动添加

  需要注意的是，如果是重定向到 jsp 页面，则 jsp 页面不 能写在 WEB-INF 目录中，否则无法找到

### 4.2 void(配合Ajax)

- 在Controller方法形参上可以绑定原生ServletAPI，使用request或response**指定响应结果**。常与`@ResponseBody`一起使用

  ```java
  //使用request请求转发页面
  request.getRequestDispatcher("页面路径").forward(request, response);
  //通过response页面重定向
  response.sendRedirect("url")
  //通过response指定响应结果，例如响应json数据如下
  response.setCharacterEncoding("utf-8"); 
  response.setContentType("application/json;charset=utf-8"); 
  response.getWriter().write("{\"abc\":123}");
  ```

### 4.3 ModelAndView

* ModelAndView 是 SpringMVC 为我们提供的一个对象，该对象也可以用作控制器方法的返回值。 该对象中有两个方法： 

  ```java
  @RequestMapping("/testReturnModelAndView") 
  public ModelAndView testReturnModelAndView() {  
      ModelAndView mv = new ModelAndView();  
      mv.addObject("username", "张三");  //存储的request域中
      mv.setViewName("success"); 
      return mv; 
  }
  ```




## 5 常用注解

### 5.1 `@RequestParam`

作用：把请求中指定名称的参数给控制器中的形参赋值。 但是还不如参数名称一致好使。可以用于**分页中默认值设置**

属性：

* `value`或`name`：请求参数中的名称。若参数名称(name)和形参一致，可以不用指定value或name
* `required`：请求参数中是否必须提供此参数。默认值：true。表示必须提供，如果不提供将报错
* `defaultValue`：请求参数默认值

```html
<a href="springmvc/useRequestParam?name=test">requestParam 注解</a> 
```

```java
@RequestMapping("/useRequestParam") 
public String test( 
    @RequestParam("name")String username, 
    @RequestParam(value="age",required=false,defaultValue="88")Integer age){
    
   	System.out.println(username+","+age);  
    return "success"; 
}
```



### 5.2 `@PathVariable`

作用：用于**==绑定 url 中的占位符==**。例如：请求 url 中 `/delete/{id}`，这个`{id}`就是 url 占位符。  

​	url 支持占位符是 spring3.0 之后加入的。是 springmvc 支持 rest 风格 URL 的一个重要标志。 

属性：  

- `value`或`name`：用于指定 url 中占位符名称。若占位符名称和形参一致，可以不用指定value。尽量指定！！！
- `required`：是否必须提供占位符。 

```html
<a href="springmvc/usePathVariable/100">pathVariable 注解</a> 
```

```java
@RequestMapping("/usePathVariable/{sid}") 
public String usePathVariable(@PathVariable("sid") Integer sid){  
    System.out.println(sid);  
    return "success"; 
}
```



### 5.3 `@RequestBody`

作用：用于==**获取请求体**==内容。直接使用得到是`key=value&key=value`结构的数据。**get 请求方式不适用**。常用于**Json数据封装**

属性：`required`是否必须有请求体，默认为true。当取值为true时，get请求方式会报错；若为false，get请求得到是null

```html
<form action="springmvc/useRequestBody" method="post">
    用户名称：<input type="text" name="username" ><br/>  
    用户密码：<input type="password" name="password" ><br/>  
    用户年龄：<input type="text" name="age" ><br/> 
 <input type="submit" value=" 保存 "> </form>
```

```java
@RequestMapping("/useRequestBody") 
public String useRequestBody(@RequestBody(required=false) String body){  
    System.out.println(body);  
    return "success"; 
}
```



### 5.4 `@ResponseBody`

用于将 Controller 的方法返回的对象，通过 `HttpMessageConverter` 接口转换为指定格式的数据如：`json`,`xml` 等，通过 Response 响应给客户端 

如果需要SpringMVC支持JSON，必须加入JSON的处理**jar包**：Jackson

```xml
<dependency>            
    <groupId>com.fasterxml.jackson.core</groupId>            
    <artifactId>jackson-core</artifactId>            
    <version>2.9.0</version>        
</dependency>        
<dependency>            
    <groupId>com.fasterxml.jackson.core</groupId>            
    <artifactId>jackson-annotations</artifactId>            
    <version>2.9.0</version>        
</dependency>
<dependency>            
     <groupId>com.fasterxml.jackson.core</groupId>            
     <artifactId>jackson-databind</artifactId>            
     <version>2.9.0</version>        
</dependency>        
```

```java
@RequestMapping("/testJson")  
@ResponseBody
public Address testJson(@RequestBody Address address) {        
    System.out.println(address);        
    address.setAddressName("上海");        
    return address;    
}
```

```javascript
 $(function(){        
     // 绑定点击事件        
     $("#btn").click(function(){            
         $.ajax({                
             url:"user/testJson",  
             type:"post", 
             contentType:"application/json;charset=UTF-8",
             data:'{"addressName":"哈哈","addressNum":100}',
             success:function(data){                    
                 alert(data);                    
                 alert(data.addressName);                
             }
             dataType:"json",                                           
         });        
     });    
 }); 
```



### 5.5 `@RestController`

是`@Controller`和`@ResponseBody`的组合

### 5.6 `@Get/Post/Put/Delete/PatchMapping`

用在方法上，替代方法的`@RequestMapping`

### 5.7 `@DateTimeFormat`

直接在JavaBean**属性**上添加注释即可（在get或set方法上添加，命名规范的话字段上添加也行。由于一般自动生成，所以都行）

```java
@DateTimeFormat(pattern="yyyy-MM-dd HH:mm")
private Date creationTime;
```



### 5.8 RESTful

- RESTful是一个资源定位及资源操作的风格。使用POST、DELETE、PUT、GET，使用不同方法对资源进行操作，分别对应  添加、 删除、修改、查询

- 需求：RESTful方式实现商品信息查询，返回json数据

  - **从URL上获取参数**：根据id查询商品，使用RESTful风格开发的接口地址是：http://127.0.0.1/item/1

    - 注解**`@RequestMapping("item/{id}")`**声明请求的URL，`{xxx}`为占位符，请求的URL是“`item /1`”

    - 使用**`@PathVariable() Integer id`**获取URL上的数据

      ```java
      @RequestMapping("item/{id}")
      public @ResponseBody Item queryItemById(@PathVariable Integer id) {
          Item item = this.itemService.queryItemById(id);
          return item;
      }
      ```

      - 如果`@RequestMapping`中表示为"`item/{id}`"，id和形参名称一致，`@PathVariable`不用指定名称。如果不一致，例如"`item/{ItemId}`"则需要指定名称`@PathVariable("itemId")`

    - **注意**：

      - @PathVariable是获取url上数据的。@RequestParam获取请求参数的（包括post表单提交）
      - 如果加上@ResponseBody注解，就不会走视图解析器，不会返回页面，返回如json数据。如果不加，就走视图解析器，返回页面

* 注意：
  * 表单只支持GET、POST请求，若要发送其他请求，表单本身设置为POST请求，并需要input中属性`name="_method" value="PUT"`，Ajax中查看HTML章节
  * 后端中需要配置过滤器`org.springframework.web.filter.HiddenHttpMethodFilter`

### 5.9 其他不常用注解

#### `@RequestHeader`

一般不怎么用

* 作用：用于获取请求消息头。 
* 属性：`value`提供消息头名称。`required`是否必须有此消息头 

------

#### `@CookieValue`

一般不怎么用

* 作用：用于把指定 cookie 名称的值传入控制器方法参数。 
* 属性：`value`指定 cookie 的名称。`required`是否必须有此 cookie

------

#### `@ModelAttribute`

> 该注解是 SpringMVC4.3 版本以后新加入的。它可以用于修饰方法和参数。 
>
> - 出现在**方法上**，表示当前方法会在**控制器的方法执行之前先执行**。它可以修饰没有返回值和有具体返回值的方法
> - 出现在**参数上**，获取**指定的数据给参数赋值**

* 属性：`value`用于获取数据的 key。key 可以是 **POJO** 的属性名称，也可以是 **map** 结构的 key。 

* 应用场景：当表单提交数据不是完整的实体类数据时，保证**没有提交数据的字段**使用**数据库对象原来的数据**。

  * 我们在编辑一个用户时，用户有一个创建信息字段，该字段的值是不允许被修改的。在提交表单数据是肯定没有此字段的内容，一旦更新会把该字段内容置为 null，此时就可以使用此注解解决问题

* 基于POJO 属性的基本使用

  ```html
  <a href="springmvc/testModelAttribute?username=test">测试 modelattribute</a> 
  ```

  ```java
  @ModelAttribute  
  public void showModel(User user) {   
      System.out.println("执行了 showModel 方法"+user.getUsername());  
  } 
  @RequestMapping("/testModelAttribute") 
  public String testModelAttribute(User user) {   
      System.out.println("执行了控制器的方法"+user.getUsername());   
      return "success";  
  }
  //执行了 showModel 方法
  //执行了控制器的方法
  ```

* 基于 Map 的应用场景示例 1：ModelAttribute 修饰方法带返回值 

  ```html
  <!--需求：  修改用户信息，要求用户的密码不能修改 -->
  <form action="springmvc/updateUser" method="post"> 
      用户名称：<input type="text" name="username" ><br/>  
      用户年龄：<input type="text" name="age" ><br/>  
      <input type="submit" value=" 保存 "> 
  </form> 
  ```

  ```java
  // 模拟修改用户方法 
  @RequestMapping("/updateUser") 
  public String testModelAttribute(User user) {  
      System.out.println("控制器中处理请求的方法：修改用户："+user);  
      return "success"; 
  }
  @ModelAttribute public User showModel(String username) {  
      //模拟去数据库查询  
      User abc = findUserByName(username); 
      System.out.println("执行了 showModel 方法"+abc);  
      return abc; 
  } 
  // 模拟去数据库查询 
  private User findUserByName(String username) {  
      User user = new User();  
      user.setUsername(username);
      user.setAge(19);  
      user.setPassword("123456");  
      return user; 
  }
  //输出会给未提交的age字段赋值19，其他的使用提交的数据
  ```

* 基于 Map 的应用场景示例 2：ModelAttribute 修饰方法不带返回值 

  ```java
  @RequestMapping("/updateUser") 
  public String testModelAttribute(@ModelAttribute("abc")User user) {  
      System.out.println("控制器中处理请求的方法：修改用户："+user);  
      return "success"; 
  }
  @ModelAttribute 
  public void showModel(String username,Map<String,User> map) { 
   	//模拟去数据库查询  
      User user = findUserByName(username); 
      System.out.println("执行了 showModel 方法"+user);  
      map.put("abc",user); 
  } 
  // 模拟去数据库查询 
  private User findUserByName(String username) {  
      User user = new User();  
      user.setUsername(username);
      user.setAge(19);  
      user.setPassword("123456");  
      return user; 
  }
  //输出会给未提交的age字段赋值19，其他的使用提交的数据
  ```

------

#### `@SessionAttribute`

* 作用：用于多次执行控制器方法间的参数共享。 

* 属性：

  * `value`：用于指定存入的属性名称  
  * `type`：用于指定存入的数据类型。 

  ```java
  @Controller("sessionAttributeController") 
  @RequestMapping("/springmvc") 
  @SessionAttributes(value= {"username","password","age"},types= {String.class,Integer.class})//存入到session域 
  public class SessionAttributeController { 
      @RequestMapping("/testPut")    
      public String testPut(Model model){           
          model.addAttribute("username", "泰斯特");           
          model.addAttribute("password","123456");           
          model.addAttribute("age", 31);   
          //跳转之前将数据保存到 username、password 和 age 中，因为注解@SessionAttribute 中有这几个参数 
          return "success"; 
      }
      
      @RequestMapping("/testGet")       
      public String testGet(ModelMap model){           
          System.out.println(model.get("username")+";"+model.get("password")+";"+model.get("age"));           
          return "success";       
      } 
      
      @RequestMapping("/testClean")        
      public String complete(SessionStatus sessionStatus){         
          sessionStatus.setComplete();            
          return "success";        
      }
  }
  ```



## 6 处理Multipart数据(文件上传)

pom.xml中添加依赖

```xml
<dependency>            
    <groupId>commons-fileupload</groupId>            
    <artifactId>commons-fileupload</artifactId>            
    <version>1.3.1</version>        
</dependency>        
<dependency>            
    <groupId>commons-io</groupId>            
    <artifactId>commons-io</artifactId>            
    <version>2.4</version>        
</dependency
```

### 6.1 文件上传的回顾 

* **form表单的`enctype`取值必须是`multipart/form-data`**(默认值是`application/x-www-form-urlencoded`)。`enctype`代表表单请求正文的类型
* `method` 属性取值必须是** `Post`**
* 提供一个**文件选择域**`<input type="file" />  `

```html
<form action="user/fileupload" method="post" enctype="multipart/form-data">       
    选择文件：<input type="file" name="upload"/><br/>        
    <input type="submit" value="上传文件"/>    
</form>
```

```java
@RequestMapping(value="/fileupload")    
public String fileupload(HttpServletRequest request) throws Exception {        
    // 先获取到要上传的文件目录        
    String path = request.getSession().getServletContext().getRealPath("/uploads");        
    // 创建File对象，一会向该路径下上传文件        
    File file = new File(path);        
    // 判断路径是否存在，如果不存在，创建该路径        
    if(!file.exists()) {            
        file.mkdirs();        
    }        
    // 创建磁盘文件项工厂        
    DiskFileItemFactory factory = new DiskFileItemFactory();        
    ServletFileUpload fileUpload = new ServletFileUpload(factory);        
    // 解析request对象        
    List<FileItem> list = fileUpload.parseRequest(request);        
    // 遍历        
    for (FileItem fileItem : list) {            
        // 判断文件项是普通字段，还是上传的文件            
        if(fileItem.isFormField()) {                            

        }else {                
            // 上传文件项
            // 获取到上传文件的名称                
            String filename = fileItem.getName();               
            // 上传文件                
            fileItem.write(new File(file, filename));                
            // 删除临时文件                
            fileItem.delete();            
        }        
    }                
    return "success";    
}
```



### 6.2 SpringMVC传统方式的文件上传

> 传统方式的文件上传，指的是我们上传的文件和访问的应用存在于同一台服务器上。 并且上传完成之后，浏览器可能跳转。 

```java
@RequestMapping(value="/fileupload2")    
public String fileupload2(HttpServletRequest request,MultipartFile upload) throws Exception {        
    System.out.println("SpringMVC方式的文件上传...");        
    // 先获取到要上传的文件目录        
    String path = request.getSession().getServletContext().getRealPath("/uploads");        
    // 创建File对象，一会向该路径下上传文件        
    File file = new File(path);        
    // 判断路径是否存在，如果不存在，创建该路径        
    if(!file.exists()) {            
        file.mkdirs();        
    }        
    // 获取到上传文件的名称        
    String filename = upload.getOriginalFilename();        
    String uuid = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();        
    // 把文件的名称唯一化        
    filename = uuid+"_"+filename;        
    // 上传文件        
    upload.transferTo(new File(file,filename));        
    return "success";    
}
```

* 在**`spring-config.xml`配置文件解析器**

```xml
<!-- 配置文件上传解析器，id是固定的！！！--> 
<bean id="multipartResolver"  class="org.springframework.web.multipart.commons.CommonsMultipartResolver"> 
 <!-- 设置上传文件的最大尺寸为 5MB -->  
    <property name="maxUploadSize">   
        <value>5242880</value>  
    </property> 
</bean>
```

- JavaConfig配置**MultipartResolver**接口的实现类

  - `CommonsMultipartResolver`：使用Jakarta Commons FileUpload解析multipart请求

  - **`StandardServletMultipartResolver`**：依赖于**Servlet3.0**对multipart请求支持（**始于Spring3.1**）

    选择这个，它使用Servlet所提供的功能支持，不依赖其他项目。它**没有构造器参数和属性**

    ```java
    @Bean
    public MultipartResolver multipartResolver() throws IOException {
        return new StandardServletMultipartResolver();
    }
    ```

    如果配置DispatcherServlet的Servlet初始化类继承了**AbstractAnnotationConfigDispatcherServletInitializer**或AbstractDispatcherServletInitializer的话，通过**重载customize Registration()方法**（它会得到Dynamic参数）来配置multipart的具体细节

    ```java
    //class Config extends AbstractAnnotationConfigDispatcherServletInitializer
    @Override
    protected void customizeRegistration(Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement("/tmp/file/uploads",2097152,4194304,0));
        //location,maxFileSize,maxRequestSize,fileSizeThreshold(为0则上传文件写到磁盘)
    }
    ```



### 6.3 SpringMVC跨服务器方式的文件上传

> 在实际开发中，我们会有很多处理不同功能的服务器（不是服务器集群）。例如： 
>
> * 应用服务器：负责部署我们的应用 
> * 文件服务器：负责存储用户上传文件的服务器
> * 数据库服务器：运行我们的数据库 
> * ……
>
> 目的是让服务器各司其职，从而提高我们项目的运行效率

步骤：

1. **搭建图片服务器** 

   1. 根据文档配置tomcat9的服务器，现在是2个服务器 
   2. 导入资料中day02_springmvc5_02image项目，作为图片服务器使用 

2. 实现SpringMVC跨服务器方式文件上传

   1. 导入依赖的jar包的坐标（sun公司提供的，下面导包时注意）

      ```xml
      <dependency>            
          <groupId>com.sun.jersey</groupId>            
          <artifactId>jersey-core</artifactId>            
          <version>1.18.1</version>        
      </dependency>        
      <dependency>            
          <groupId>com.sun.jersey</groupId>            
          <artifactId>jersey-client</artifactId>            
          <version>1.18.1</version>        
      </dependency>
      ```

   2. 控制器

      ```java
      @RequestMapping(value="/fileupload3")
      public String fileupload3(MultipartFile upload) throws Exception {        
          System.out.println("SpringMVC跨服务器方式的文件上传...");                
          // 定义图片服务器的请求路径        
          String path = "http://localhost:9090/day02_springmvc5_02image/uploads/";//创建好该文件夹              
          // 获取到上传文件的名称        
          String filename = upload.getOriginalFilename();        
          String uuid = UUID.randomUUID().toString().replaceAll("-", "").toUpperCase();        
          // 把文件的名称唯一化        
          filename = uuid+"_"+filename;        
          // 向图片服务器上传文件                
          // 创建客户端对象        
          Client client = Client.create();        
          // 连接图片服务器        
          WebResource webResource = client.resource(path+filename);        
          // 上传文件        
          webResource.put(upload.getBytes());        
          return "success";    
      }
      ```

   3. **配置文件解析器**，同上



## 7 异常处理器

- SpringMVC在处理请求过程中出现异常信息交由异常处理器进行处理，自定义异常处理器可以实现一个系统的异常处理逻辑

- 思路：

  - 系统中异常包括两类：**预期异常**和运行时异常**RuntimeException**，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生
  - 系统的dao、service、controller出现都通过throws Exception向上抛出，最后由SpringMVC**前端控制器交由异常处理器**进行异常处理

- **自定义异常类(继承Exception或RuntimeException)**：为了区别不同的异常,通常根据异常类型进行区分

  ```java
  public class MyException{
      public MyException(){};
      public MyException(String msg){
          super(msg);
      };
  }
  ```

- **自定义异常处理器(实现HandlerExceptionResolver)**，并**在`spring-config.xml`中配置或使用`@Component`**

  ```java
  @Component
  public class CustomExceptionResolver implements HandlerExceptionResolver {
     	//handler:异常处理器对象。发生异常的地方，包名+类名+方法名(形参)的字符串，用于日志
      @Override
      public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,Exception exception) {
          
          ModelAndView modelAndView = new ModelAndView();
          // 定义异常信息
          String msg = "";
  
          // 判断异常类型
          if (exception instanceof MyException) {
              // 如果是自定义异常，读取异常信息
              msg = exception.getMessage();
          } else {
              //简写
              msg = "服务器访问量过大，请您稍后..."
              //或 如果是运行时异常，则取错误堆栈，从堆栈中获取异常信息
              //Writer out = new StringWriter();
              //PrintWriter s = new PrintWriter(out);
              //exception.printStackTrace(s);
              //msg = out.toString();
  
          }
          // 把错误信息发给相关人员,邮件,短信等方式
          // 返回错误页面，给用户友好页面显示错误信息
         
          modelAndView.addObject("msg", msg);
          modelAndView.setViewName("error");
  
          return modelAndView;
      }
  }
  ```



## 8 拦截器

- 类似于Servlet 开发中的过滤器Filter，用于对处理器进行预处理和后处理。AOP思想的具体应用
- 区别：
  - 过滤器是 servlet 规范中的一部分，任何 java web 工程都可以使用。 
  - 拦截器是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用。 
  - 过滤器在 url-pattern 中配置了`/*`之后，可以对所有要访问的资源拦截。 
  - **拦截器**它是**只会拦截访问的控制器方法**，如果访问的是 jsp,html,css,image 或者 js 是不会进行拦截的

使用步骤：

- **实现HandlerInterceptor接口，并重写三个默认方法**

  ```java
  public class HandlerInterceptor1 implements HandlerInterceptor {
      // Controller执行前调用此方法
      // 返回true表示放行，返回false不放行
      // 这里可以加入登录校验、权限拦截等
      @Override
      public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
          System.out.println("HandlerInterceptor1....preHandle");
          return true;
      }
  
      // controller执行后但未返回视图前调用此方法，且只有所有preHandle返回true时调用
      // 这里可在返回用户前对模型数据进行加工处理，比如这里加入公用信息以便页面显示
      @Override
      public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3) throws Exception {
          System.out.println("HandlerInterceptor1....postHandle");
      }
  
      // controller执行后且视图返回后调用此方法，且只有preHandle返回true才调用
      // 这里可得到执行controller时的异常信息
      // 这里可记录操作日志
      @Override
      public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3) throws Exception {
          System.out.println("HandlerInterceptor1....afterCompletion");
      }
  }
  ```

* **拦截器配置**

  ```xml
  //spring-config.xml
  <!-- 配置拦截器 -->
  <mvc:interceptors>
      <mvc:interceptor>
          <!-- /user/* 请求进入拦截器 -->
          <mvc:mapping path="/user/*" />
          <!-- 配置具体的拦截器 -->
          <bean class="cn.itcast.ssm.interceptor.HandlerInterceptor1" />
      </mvc:interceptor>
      <mvc:interceptor>
          <!-- 所有的请求都进入拦截器 -->
          <mvc:mapping path="/**" />
          <!-- 配置具体的拦截器 -->
          <bean class="cn.itcast.ssm.interceptor.HandlerInterceptor2" /> <!--ref也可以，但必须有@Component注解 -->
      </mvc:interceptor>
  </mvc:interceptors>
  
  //需要释放静态资源
  ```

* 总结：

  - preHandle按拦截器定义顺序调用，**返回false时后续拦截器将不调用**
  - postHandler按拦截器定义逆序调用，且只有**所有preHandle返回true时调用**
  - afterCompletion按拦截器定义逆序调用，且**只有preHandle返回true才调用**，与上一条不同



* 应用
  * 有一个登录页面，需要写一个Controller访问登录页面

  * 登录页面有一提交表单的动作。需要在Controller中处理

    * 判断用户名密码是否正确（在控制台打印）
    * 如果正确,向session中写入用户信息（写入用户名username）
    * 跳转到商品列表

  * 拦截器

    * 拦截用户请求，判断用户是否登录（登录请求不能拦截）
    * 如果用户已经登录。放行
    * 如果用户未登录，跳转到登录页面。

    ```java
    public class LoginInterceptor implements HandlerInterceptor{ 
    	@Override  
        Public boolean preHandle(HttpServletRequest request,HttpServletResponse response, Object handler) throws Exception { 
            //如果是登录页面则放行   
            if(request.getRequestURI().indexOf("login.action")>=0)
                return true;   
            HttpSession session = request.getSession(); 
      
            //如果用户已登录也放行   
            if(session.getAttribute("user")!=null)
                return true;   
            
            //用户没有登录挑战到登录页面   
            request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);   
            return false;  
        } 
    } 
    ```





## 9 Servlet 3.0及纯注解配置

详细的查看文档，搜索JCP并搜Servlet下载即可

------

### 9.1 ServletContainerInitializer

文档的8.2.4章节—Shared libraries（共享库） / runtimes pluggability（运行时可插拔性）

1. Servlet容器启动会扫描，当前应用里面每一个jar包的ServletContainerInitializer的实现
2. 提供ServletContainerInitializer的实现类，必须绑定在META-INF/services/javax.servlet.ServletContainerInitializer，文件的内容就是ServletContainerInitializer实现类的全类名

总结：容器在启动应用的时候，会扫描当前应用每一个jar包里面META-INF/services/javax.servlet.==**ServletContainerInitializer**==
指定的实现类，启动并运行这个实现类的方法；`@HandlesTypes`注解用于传入感兴趣的类型。



### 9.2 ServletContext注册三大组件

```java
//容器启动的时候会将@HandlesTypes指定的这个类型下面的子类（实现类，子接口等）传递过来；
//传入感兴趣的类型；
@HandlesTypes(value={HelloService.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {

	/**
	 * 应用启动的时候，会运行onStartup方法；
	 * 
	 * Set<Class<?>> arg0：感兴趣的类型的所有子类型；
	 * ServletContext arg1:代表当前Web应用的ServletContext；一个Web应用一个ServletContext；
	 * 
	 * 1）、使用ServletContext注册Web组件（Servlet、Filter、Listener）
	 * 2）、使用编码的方式，在项目启动的时候给ServletContext里面添加组件；
	 * 		必须在项目启动的时候来添加；
	 * 		1）、ServletContainerInitializer得到的ServletContext；
	 * 		2）、ServletContextListener得到的ServletContext；
	 */
	@Override
	public void onStartup(Set<Class<?>> arg0, ServletContext sc) throws ServletException {
		System.out.println("感兴趣的类型：");
		for (Class<?> claz : arg0) {
			System.out.println(claz);
		}
		
		//注册组件  ServletRegistration  
		ServletRegistration.Dynamic servlet = sc.addServlet("userServlet", new UserServlet());
		//配置servlet的映射信息
		servlet.addMapping("/user");
		
		
		//注册Listener
		sc.addListener(UserListener.class);
		
		//注册Filter  FilterRegistration
		FilterRegistration.Dynamic filter = sc.addFilter("userFilter", UserFilter.class);
		//配置Filter的映射信息
		filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST), true, "/*");		
	}
}
```



### 9.3 与SpringMVC整合

> 查看Spring官方文档即可

![](F:\GitHub\Studying\SSM\images\mvc-context-hierarchy.png)

**原理**

1、web容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer

2、加载这个文件指定的类SpringServletContainerInitializer

3、spring的应用一启动会加载感兴趣的WebApplicationInitializer接口的下的所有组件；

4、并且为WebApplicationInitializer组件创建对象（组件不是接口，不是抽象类）

​	1）、AbstractContextLoaderInitializer：创建根容器；createRootApplicationContext()；

​	2）、AbstractDispatcherServletInitializer：
​			创建一个web的ioc容器；createServletApplicationContext();
​			创建了DispatcherServlet；createDispatcherServlet()；
​			将创建的DispatcherServlet添加到ServletContext中；
​				getServletMappings();

​	3）、AbstractAnnotationConfigDispatcherServletInitializer：注解方式配置的DispatcherServlet初始化器
​			创建根容器：createRootApplicationContext()
​					getRootConfigClasses();传入一个配置类
​			创建web的ioc容器： createServletApplicationContext();
​					获取配置类；getServletConfigClasses();
​	
总结：以注解方式来启动SpringMVC；**继承`AbstractAnnotationConfigDispatcherServletInitializer`**；**实现抽象方法**指定DispatcherServlet等的配置信息（和在web.xml中功能一致）；

------

定制SpringMVC；

1. `@EnableWebMvc`：开启SpringMVC定制配置功能；与`<mvc:annotation-driven/>`功能一致
2. 配置组件（视图解析器、视图映射、静态资源映射、拦截器。。。） 
   1. 实现`WebMvcConfigurer`重写全部方法！！！可以配置任何东西。不如下面的实现类方便。
   2. 继承`WebMvcConfigurerAdapter`（Spring5.0后过时）
   3. **实现`WebMvcConfigurer `接口**

```java
//web容器启动的时候创建对象；调用方法来初始化容器以前前端控制器
public class WebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	//获取根容器的配置类；（Spring的配置文件）   父容器；
	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[]{RootConfig.class};
	}

	//获取web容器的配置类（SpringMVC配置文件）  子容器；
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[]{AppConfig.class};
	}

	//获取DispatcherServlet的映射信息
	//  /：拦截所有请求（包括静态资源（xx.js,xx.png）），但是不包括*.jsp；
	//  /*：拦截所有请求；连*.jsp页面都拦截；jsp页面是tomcat的jsp引擎解析的；
	@Override
	protected String[] getServletMappings() {
		return new String[]{"/"};
	}
    
    //该方法下的所有过滤器都会映射到DispatcherServlet
    @Override
    protected Filter[] getServletFilters() {
        //编码过滤器
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter("UTF-8",true);
        //解决PUT、DELETE等请求的过滤器
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();

        return new Filter[]{characterEncodingFilter,hiddenHttpMethodFilter};
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        return super.createServletApplicationContext();
    }
}
```

```java
@Configuration
@ComponentScan(value="cn.itcast",excludeFilters={
    @Filter(type=FilterType.ANNOTATION,classes={Controller.class})
})//扫描cn.itcast下的除过Controller的所有组件，不能添加useDefaultFilters
@EnableTransactionManagement//开启事务
@PropertySource("classpath:JdbcConfig.properties")//导入外部配置文件
@Configuration
public class RootConfig {

    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;


    @Bean
    public DataSource dataSource(){
        //DriverManagerDataSource dataSource = new DriverManagerDataSource();//使用spring提供的DataSource
        DruidDataSource dataSource = new DruidDataSource();//配置Druid的DataSource
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
    
    //配置事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }


   /**配置mybatis，执行后总是报NullPointerException，毫无头绪。上面的JdbcTemplate成功了

    //配置SqlSessionFactory，利用org.mybatis.spring提供的SqlSessionFactoryBean
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        
        //数据源
        sqlSessionFactoryBean.setDataSource(dataSource);
        
        //pageHelper的拦截器插件配置
        PageInterceptor pageInterceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect","mysql");
        properties.setProperty("reasonable","true");
        pageInterceptor.setProperties(properties);
        sqlSessionFactoryBean.setPlugins(new Interceptor[]{pageInterceptor});

        return sqlSessionFactoryBean.getObject();  //mybatis-spring整合的文档是这么写的
    }

    //配置Mapper接口扫描
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("cn.itcast.dao");
        return mapperScannerConfigurer;
    } */
}
```

```java
@Configuration
@ComponentScan(value = "cn.itcast.controller", includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)
}, useDefaultFilters = false)//必须添加useDefaultFilters
@EnableWebMvc
public class AppConfig implements WebMvcConfigurer  {

    @Autowired
    private ApplicationContext servletApplicationContext;

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //模板解析器
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
        templateResolver.setApplicationContext(this.servletApplicationContext);//必须有
        templateResolver.setPrefix("/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode("HTML");
        templateResolver.setCharacterEncoding("UTF-8");

        //模板引擎
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);

        //Thymeleaf视图解析器
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(templateEngine);
        viewResolver.setCharacterEncoding("UTF-8");

        registry.viewResolver(viewResolver);

        //registry.jsp("/views/",".jsp");
    }

    //释放静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyFirstInterceptor()).addPathPatterns("/**");
    }
}
```

> 以上配置，在dao层查询时会报空指针异常，暂时GG。同样的xml配置OK。我不OK了！！！
>

纯注解配置时由于Maven的原因，需要配置如下plugin

```xml
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <warSourceDirectory>WebContent</warSourceDirectory>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```









## 10 Thymeleaf

JSP、Velocity、Freemarker、Thymeleaf（Spring推荐，语法更简单，功能更强大）

![](images\template-engine.png)

### 10.1 依赖及配置

pom.xml

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```

springmvc.xml配置如下：

```xml
<bean id="applicationContext" class="org.springframework.context.support.ClassPathXmlApplicationContext"/>
<!--模板解析器-->
<bean id="templateResolver" class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
    <property name="applicationContext" ref="applicationContext"/>
    <property name="prefix" value="/templates/"/> <!--把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；-->
    <property name="suffix" value=".html"/>
    <property name="templateMode" value="HTML"/>
    <property name="characterEncoding" value="UTF-8"/><!--解决页面乱码！-->
</bean>
<!--模板引擎-->
<bean id="templateEngine" class="org.thymeleaf.spring5.SpringTemplateEngine">
    <property name="templateResolver" ref="templateResolver"/>
</bean>
<!--Thymeleaf视图解析器-->
<bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <property name="templateEngine" ref="templateEngine"/>
    <property name="characterEncoding" value="UTF-8"/><!--解决页面乱码！-->
</bean>
```

HTML中导入名称空间，有代码提示

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```





### 10.2 语法

* `th:text`：改变当前元素里面的文本内容，转移特殊字符。`th:utext`不转义。**th:任意html属性**：来替换原生属性的值。

* `th:each`：遍历

  ```html
  <tr th:each="p : ${pageInfo.list}">
      <td th:text="${p.id}"></td>
      <td th:text="${p.name}"></td>
  ```



![](images/2018-02-04_123955.png)

上图有优先级顺序



- Simple expressions:（表达式语法）

  - **==Variable Expressions==: `${...}`**：获取变量值；底层是OGNL；

    - 获取对象的属性、调用方法。
    - 使用内置的基本对象：
      - `${#ctx}` : the context object.
      - `${#vars}`: the context variables.
      - `${#locale}` : the context locale. 如`${#locale.country}`
      - `${#request}` : (only in Web Contexts) the HttpServletRequest object.
      - `${#response}` : (only in Web Contexts) the HttpServletResponse object.
      - `${#session}`#session : (only in Web Contexts) the HttpSession object.
      - `${#servletContext}` : (only in Web Contexts) the ServletContext object.
    - 使用内置的**工具**对象（同上，放在`${...}`里）：
      - `#execInfo` : information about the template being processed.
      - `#messages` : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
      - `#uris` : methods for escaping parts of URLs/URIs
      - `#conversions` : methods for executing the configured conversion service (if any).
      - `#dates` : methods for java.util.Date objects: formatting, component extraction, etc.
      - `#calendars` : analogous to #dates , but for java.util.Calendar objects.
      - `#numbers` : methods for formatting numeric objects.
      - `#strings` : methods for String objects: contains, startsWith, prepending/appending, etc.
      - `#objects` : methods for objects in general.
      - `#bools` : methods for boolean evaluation.
      - `#arrays` : methods for arrays.
      - `#lists` : methods for lists.
      - `#sets` : methods for sets.
      - `#maps` : methods for maps.
      - `#aggregates` : methods for creating aggregates on arrays or collections.
      - `#ids` : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

  - Selection Variable Expressions: `*{...}`：选择表达式：和`${...}`在功能上是一样；配合`th:object`使用如下：

    ```html
    <div th:object="${session.user}">
        <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>	<!--${session.user.firstName}-->
        <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>	<!--${session.user.lastName}-->
        <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>	<!--${session.user.nationality}-->
    </div>
    ```

  - Message Expressions: `#{...}`：获取国际化内容

  - ==**Link URL Expressions**==: `@{...}`：定义URL；在需要参数时，放在`()`里。

    ```html
    <li><a href="" th:href="@{/user}" aria-label="Previous">首页</a></li>
    <li><a href="" th:href="@{/user(pageNum=${pageInfo.pageNum}-1)}">上一页</a></li>
    <li th:each="i:${#numbers.sequence(1,pageInfo.pages)}"><a href="" th:href="@{/user(pageNum=${i})}" th:text="${i}"></a></li> 暂时不会使用thymeleaf写页码
    <li><a href="" th:href="@{/user(pageNum=${pageInfo.pageNum}+1)}">下一页</a></li>
    <li><a href="" th:href="@{/user(pageNum=${pageInfo.pages})}" aria-label="Next">尾页</a></li>
    若使用Pagehelper，无需考虑越界
    
    <form th:action="@{/user/}+${user.id}" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="id" th:value="${user.id}">
        <input type="radio" th:name="sex" value="男" th:checked="${user.sex}=='男'?true:false">
    </form>
    
    <script th:src="@{/plugins/bootstrap/js/bootstrap.min.js}"></script>
    ```

  - Fragment Expressions: `~{...}`：片段引用表达式

  - Literals（字面量）

    - Text literals: 'one text' , 'Another one!' ,…
    - Number literals: 0 , 34 , 3.0 , 12.3 ,…
    - Boolean literals: true , false
    - Null literal: null
    - Literal tokens: one , sometext , main ,…

  - Text operations:（文本操作）

    - String concatenation: +
    - Literal substitutions: |The name is ${name}|

  - Arithmetic operations:（数学运算）

    - Binary operators: + , - , * , / , %
    - Minus sign (unary operator): -

  - Boolean operations:（布尔运算）

    - Binary operators: and , or
    - Boolean negation (unary operator): ! , not

  - Comparisons and equality:（比较运算）

    - Comparators: > , < , >= , <= ( gt , lt , ge , le )
    - Equality operators: == , != ( eq , ne )

  - Conditional operators:条件运算（三元运算符）

    - If-then: (if) ? (then)
    - If-then-else: (if) ? (then) : (else)
    - Default: (value) ?: (defaultvalue)

  - Special tokens:

    - No-Operation: _ 








### 10.3 公共页面的抽取

```html
1、抽取公共片段
<footer th:fragment="copy" id="footer1">
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

2、引入方式（每个templates下的html都是模板，其去掉后缀就是模板名）
~{templatename::selector}：模板名::选择器（CSS选择器）
~{templatename::fragmentname}:模板名::片段名
使用以下属性进行引入，可以不用写~{}
行内写法可以加上：[[~{}]];[(~{})]；一个是转义，一个不转义
<div th:insert="footer :: copy"></div> 	<!--将公共片段整个插入到声明引入的元素中-->
<div th:replace="footer :: copy"></div>	<!--将声明引入的元素替换为公共片段-->
<div th:include="footer :: copy"></div>	<!--将被引入的片段的内容（只有内容）包含进这个标签中-->

<div th:include="footer :: #footer1"></div> <!--显示效果同include-->

效果
<div>
    <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
&copy; 2011 The Good Thymes Virtual Grocery
</div>
```

引入片段的时候传入参数（动态显示高亮等用处）：

````html
<nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
                <a class="nav-link active"
                   th:class="${activeUri=='main.html'?'nav-link active':'nav-link'}"
                   href="#" th:href="@{/main.html}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                        <polyline points="9 22 9 12 15 12 15 22"></polyline>
                    </svg>
                    Dashboard <span class="sr-only">(current)</span>
                </a>
            </li>

<!--引入侧边栏;传入参数-->
<div th:replace="commons/bar::#sidebar(activeUri='emps')"></div>
````



















# 第五部分 Spring Security

[Spring Security](https://projects.spring.io/spring-security/) 的前身是 Acegi Security ，是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI 和 AOP功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

安全包括两个主要操作：

* “认证”是为用户建立一个他所声明的主体。主体一般是指用户，设备或可以在你系统中执行动作的其他系统。 
* “授权”指的是一个用户能否在你的应用中执行某个操作，在到达授权判断之前，身份的主体已经由身份验证过程建立了。

基于Spring Security的数据库认证的操作方式有多种，这里我们介绍使用`UserDetails`、 `UserDetailsService`来完成操作

* **`UserDetails`**接口，作用是于**封装当前进行认证的用户信息**，我们可以对其进行实现，也可以**使用Spring Security提供的一个UserDetails的实现类User**来完成操作

  ```java
  public interface UserDetails extends Serializable {     
  	Collection<? extends GrantedAuthority> getAuthorities();    
  	String getPassword();       
  	String getUsername();       
  	boolean isAccountNonExpired();      
  	boolean isAccountNonLocked();    
  	boolean isCredentialsNonExpired();    
  	boolean isEnabled(); 
  }
  ```

  以下是User类的部分代码：

  ```java
  public class User implements UserDetails, CredentialsContainer {
      private String password;    
      private final String username;    
      private final Set<GrantedAuthority> authorities;    
      private final boolean accountNonExpired; //帐户是否过期    
      private final boolean accountNonLocked; //帐户是否锁定    
      private final boolean credentialsNonExpired; //认证是否过期    
      private final boolean enabled; //帐户是否可用
  }
  ```

* `UserDetailsService`接口，用于规范认证方法的接口

  ```java
  public interface UserDetailsService {        
      UserDetails loadUserByUsername(String username) throws UsernameNotFoundException; 
  }
  ```

------

![](images\使用数据库完成springSecurity用户登录流程分析.png)



## 1 登陆注销

SSM综合练习中用户登录来完成Spring Security的认证操作：

1. 导入spring-security相关依赖（自动导入spring-security-core/config/web三个依赖）

   ```xml
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-web</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-config</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   <!--下面资料中没有导入-->
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-taglibs</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   ```

2. `web.xml`中配置**`springSecurityFilterChain`**（必须这个名字），别忘了监听器加载`spring-security.xml`等配置文件

   ```xml
   <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath*:spring-security.xml</param-value>
   </context-param>
   <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   
   <filter>
       <filter-name>springSecurityFilterChain</filter-name> <!--名称不能改变！-->
       <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> <!--只是个代理类，拦截-->
   </filter>
   <filter-mapping>
       <filter-name>springSecurityFilterChain</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

3. `spring-security.xml`中配置如下

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!--采用如下约束配置可以在配置时不加security前缀，但在配置bean时需要加beans前缀。主要方便springsecurity配置
    采用此约束文件，没有前缀的都是springsecurity自身的配置-->
   <beans:beans xmlns="http://www.springframework.org/schema/security" 
                xmlns:beans="http://www.springframework.org/schema/beans" 
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.springframework.org/schema/beans
                                    http://www.springframework.org/schema/beans/spring-beans.xsd
                                    http://www.springframework.org/schema/security
                                    http://www.springframework.org/schema/security/spring-security.xsd">
   
       <!-- 配置不拦截的资源（静态资源及登录相关），不登录也可以访问。否则可能会有重定向次数过多错误  -->
       <http pattern="/*.html" security="none"/> <!--webapp一级目录下的html文件（包括login.html及login_error等）-->
       <http pattern="/css/**" security="none"/>
       <http pattern="/js/**" security="none"/>
       <http pattern="/img/**" security="none"/>
       <http pattern="/plugins/**" security="none"/>
       <http pattern="/seller/add.do" security="none"/> <!--注册相关的也必须放行-->
   
   
       <!--http标签主要用于配置拦截的具体的规则
        auto-config="true"	不用自己编写登录的页面，框架提供默认登录页面，不配置默认为true
        use-expressions="true"	是否启用SPEL表达式，不配置默认为true（功能强大，可以限制ip等，没学习过）-->
       <http auto-config="true" use-expressions="true">
           <!-- 具体的拦截规则 pattern="表示拦截页面"，与web.xml中不同。此处为目录规则，/**代表目录及其子目录
                              access="访问系统的角色名称（以ROLE_开头），当前用户必须有ROLE_ADMIN或...的角色" -->
           <intercept-url pattern="/**" access="hasAnyRole('ROLE_ADMIN','ROLE_USER')"/> <!--SPEL表达式-->
           <!-- 开启表单登陆功能，定义跳转的具体的页面 
       login-page="/login.html"  //必须以斜杠开头
       default-target-url="/index.html"  //指定了成功进行身份认证和授权后默认呈现给用户的页面 
       always-use-default-target="true"  //无论请求哪个页面登陆成功后都跳转至index而不是请求的那个页面
       authentication-failure-url="/login.html"  //指定了身份认证失败时跳转到的页面-->
           <form-login login-page="/login.html"  default-target-url="/index.html" always-use-default-target="true"  authentication-failure-url="/login.html"  />
   
           <!-- 注销登陆的配置
      logout-url：登出处理url，不配置默认为/logout
      logout-successurl：登出成功页面，默认不配置则跳转登陆页面
      invalidate-session 是否删除session，不配置默认为true -->
           <logout invalidate-session="true" logout-url="/logout" logout-success-url="/login.html"/>
   
           <!-- 关闭csrf（Cross-site request forgery）跨站请求伪造 ,如果不加会出现403错误。也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。HTML不能使用，JSP可以带指定头信息使用-->
           <csrf disabled="true"/>
   
           <!--不拦截内置框架页，否则即使登陆成功也会拦截内置框架页-->
           <headers>
               <frame-options policy="SAMEORIGIN"/>
           </headers>
       </http>
   
   
       <!-- 认证管理器，指定了认证需要访问的service 。使用成数据库中的用户名和密码。-->
       <authentication-manager>
           <authentication-provider user-service-ref="userService">
               <!-- 配置加密的方式（会自动解密）。若配置此项，则需要指定密码加密方式，否则使用{noop}拼接上密码-->
               <password-encoder ref="passwordEncoder"/> 
           </authentication-provider>
       </authentication-manager>
       <!-- 配置加密类 -->
       <beans:bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
   
   
       
      
       
       <!--==========================================================================================-->
       <!--【注意】由于pinyougou-shop-web消费者中UserDetailsServiceImpl是普通类，不能自动注入Dao接口。需要借助Dubbox的Service来间接调用，并且Service也不能直接注入，需要通过dubbox使用接口来注入Service实现类-->
       <!-- 引用dubbox 服务 -->
       <dubbo:application name="pinyougou-shop-web" />
       <dubbo:registry address="zookeeper://192.168.25.129:2181"/>
       <!--通过dubbox使用接口来注入实现类-->
       <dubbo:reference id="sellerService" interface="com.pinyougou.sellergoods.service.SellerService"/>
   
       <!--认证类-->
       <beans:bean id="userDetailsService" class="com.pinyougou.service.UserDetailsServiceImpl">
           <beans:property name="sellerService" ref="sellerService"/>
       </beans:bean>	
       <!--==========================================================================================-->
   
   
       <!-- 入门方式，在内存中存入用户名、密码、角色（若使用明文密码，不配置加密方式，则不用添加{noop}）
       <authentication-manager>
        <authentication-provider>
         <user-service>
          <user name="admin" password="{noop}admin" authorities="ROLE_ADMIN"/>  
          <user name="zhangsan" password="{noop}zhangsan" authorities="ROLE_ADMIN"/>
         </user-service>
        </authentication-provider>
       </authentication-manager>
       -->
   </beans:beans>
   ```

   自定义登陆页面：表单请求方法为**POST**，action为**`/login`**（由Springsecurity自动生成路径），用户名和密码的表单name为**username、password**（不乱修改则不需要在`spring-security.xml`中配置）

   ```xml
   <!--
    login-processing-url：修改表单登陆action，不配置默认为/login
    authentication-success-forward-url：不知道。。。
    username-parameter和password-parameter：表单中用户名、密码input标签的name值。不配置默认为username、password
   -->
   <form-login
               login-processing-url="/loginnnn"
               authentication-success-forward-url="/pages/main.jsp" 
               username-parameter="name"
               password-parameter="pswd"/>
   ```

   注销按钮访问`/logout`即可

   

4. 实现`UserDetailsService`接口，springsecurity会根据**方法返回的User**实现类**对比表单登陆的数据**来决定登陆成功与否

   ```java
   @Service//不使用Dubbox是可以这样写，但使用后这个类可能在web中只是个普通类，就只能xml配置
   public class UserDetailsServiceImpl implements UserDetailsService {
   
       @Autowired//同上，不使用Dubbox是可以这样写，但使用后就只能xml配置。并且需要提供set方法
       private SellerService sellerService;
       public void setSellerService(SellerService sellerService) {
           this.sellerService = sellerService;
       }
   
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
   
           TbSeller seller = sellerService.findOne(username);//查找数据库中的数据对象
           System.out.println(username+"==="+seller.getPassword()+"==="+seller.getStatus());
           if (seller != null) {
               boolean enabled = seller.getStatus().equals("1"); //账户是否可以使用
               //角色信息本应该和数据库中User对象一起查询出来，然后遍历并添加到如下authorities集合中
               List<GrantedAuthority> authorities = new ArrayList<>();
               authorities.add(new SimpleGrantedAuthority("ROLE_SELLER"));
               //返回UserDetails的实现类User（都是SpringSecurity提供的）
               return new User(username, seller.getPassword(), enabled, true, true, true, authorities);
           } else {
               return null;
           }
       }
   }
   ```

5. UserDao中查询user时还需查询user所属的权限

   ```java
   @Repository
   public interface UserDao {
   
       @Select("select * from users where username=#{username}")
       @Results({
               @Result(id = true, property = "id", column = "id"),
               @Result(column = "username", property = "username"),
               @Result(column = "email", property = "email"),
               @Result(column = "password", property = "password"),
               @Result(column = "phoneNum", property = "phoneNum"),
               @Result(column = "status", property = "status"),
               @Result(column = "id", property = "roles",
                       many = @Many(select = "com.itheima.ssm.dao.RoleDao.findRoleByUserId"))
       })
       UserInfo findByUsername(String username);
   }
   ```

   

## 2 显示登陆名

后端

```java
@RestController
@RequestMapping("/login")
public class LoginController {

    @RequestMapping("showName")
    public Map showName(){
        String name = SecurityContextHolder.getContext().getAuthentication().getName();
        Map map = new HashMap();
        map.put("loginName",name);
        return map;
    }
}
```

前端（AngularJS）

```js
app.service("loginService",function ($http) {
    this.showName = function () {
        return $http.get("../login/showName.do");
    }
});
```

```js
app.controller("loginController",function ($scope,loginService) {
    $scope.showName = function () {
        loginService.showName().success(
            function (data) {
                $scope.loginName = data.loginName;
            }
        )
    }
});
```

```html
<!--修改框架页的信息-->
<body ng-app="pinyougou" ng-controller="loginController" ng-init="showName()">
    {{loginName}}
</body>
```



## 3 BCrypt密码加密

用户表的密码通常使用MD5等不可逆算法加密后存储，为防止彩虹表破解更会先使用一个特定的字符串（如域名）加密，然后再使用一个随机的salt（盐值）加密。 特定字符串是程序代码中固定的，salt是每个密码单独随机，一般给用户表加一个字段单独存储，比较麻烦。 **BCrypt**算法将salt随机并混入最终加密后的密码，验证时也无需单独提供之前的salt，从而无需单独处理salt问题

MD5加密后的32为字符永远是一样的，但BCrypt即使密码一样，他们的60位字符串还是不一样（加随机的盐salt）！

配置查看上面的

控制层在注册时加密。配置好登陆时SpringSecurity会自动解密对比。

```java
@RequestMapping("/add")
public Result add(@RequestBody TbSeller seller){
    try {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        String encode = encoder.encode(seller.getPassword());
        seller.setPassword(encode);
        sellerService.add(seller);
        return new Result(true, "增加成功");
    } catch (Exception e) {
        e.printStackTrace();
        return new Result(false, "增加失败");
    }
}
```







* Spring Security允许我们在定义URL访问或方法访问所应有的权限时使用Spring EL表达式，在定义所需的访问权限时如果对应的表达式返回结果为true则表示拥有对应的权限，反之则无。Spring Security可用表达式对象的基类是SecurityExpressionRoot，其为我们提供了如下在使用Spring EL表达式对URL或方法进行权限控制时通用的内置表达式。

  |           **表达式**           | **描述**                                                     |
  | :----------------------------: | ------------------------------------------------------------ |
  |        hasRole([role])         | 当前用户是否拥有指定角色。                                   |
  |   hasAnyRole([role1,role2])    | 多个角色是一个以逗号进行分隔的字符串。如果当前用户拥有指定角色中的任意一个则返回true。 |
  |      hasAuthority([auth])      | 等同于hasRole                                                |
  | hasAnyAuthority([auth1,auth2]) | 等同于hasAnyRole                                             |
  |           Principle            | 代表当前用户的principle对象                                  |
  |         authentication         | 直接从SecurityContext获取的当前Authentication对象            |
  |           permitAll            | 总是返回true，表示允许所有的                                 |
  |            denyAll             | 总是返回false，表示拒绝所有的                                |
  |         isAnonymous()          | 当前用户是否是一个匿名用户                                   |
  |         isRememberMe()         | 表示当前用户是否是通过Remember-Me自动登录的                  |
  |       isAuthenticated()        | 表示当前用户是否已经登录认证成功了。                         |
  |     isFullyAuthenticated()     | 如果当前用户既不是一个匿名用户，同时又不是通过Remember-Me自动登录的，则返回true。 |



……看整合部分项目吧







# 第二部分 Spring Data

## 1 Spring Data JPA

### 1 ORM、Hibernate与JPA的概述

#### 1.1 ORM

**ORM（Object-Relational Mapping）** 表示**对象关系映射**。简单的说：ORM就是**建立实体类和数据库表之间的关系**（包括表和字段），从而达到操作实体类就相当于操作数据库表的目的。解决了可能会写特别多数据访问层的代码、从数据库保存数据、修改数据、删除数据等重复代码问题。

常见的orm框架：~~Mybatis（ibatis）~~、Hibernate



#### 1.2 Hibernate

Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常**轻量级**的对象封装，它将POJO与数据库表建立映射关系，是一个**全自动的ORM框架**。Hibernate可以自动生成SQL语句，自动执行，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。



#### 1.3 JPA

**JPA（Java Persistence API）**， 即**Java 持久化API**，是SUN公司推出的一套**基于ORM的规范**（操作ORM框架，JDBC是操作不同数据库），内部是由一系列的接口和抽象类构成。

------

JPA的优势：

- **1.** **标准化**

  JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问API，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

- **2.** **容器级特性的支持**

  JPA框架中支持大数据集、事务、并发等容器级事务，使得 JPA 超越了简单持久化框架的局限，在企业应用发挥更大作用。

- **3.** **简单方便**

  JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity进行注释，JPA的框架和接口也都非常简单，没有太多特别的规则和设计模式的要求，开发者可以很容易的掌握。JPA基于非侵入式原则设计，因此可以很容易的和其它框架或者容器集成

- **4.** **查询能力**

  JPA的查询语言是**面向对象**而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是Hibernate HQL的等价物。JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。

- **5.** **高级特性**

  JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。

------

JPA和Hibernate的关系就像JDBC和JDBC驱动的关系，JPA是规范，Hibernate除了作为ORM框架之外，它也是一种JPA实现。JPA怎么取代Hibernate呢？JDBC规范可以驱动底层数据库吗？答案是否定的，也就是说，如果使用JPA规范进行数据库操作，底层需要Hibernate作为其实现类完成数据持久化工作。



### 2 JPA的API介绍

#### 2.1 Persistence

**`Persistence`对象主要作用是用于获取`EntityManagerFactory`对象**的 。通过调用该类的`createEntityManagerFactory()`静态方法，根据配置文件中**持久化单元名称**创建`EntityManagerFactory`。

#### 2.2 EntityManagerFactory

`EntityManagerFactory` 接口主要用`createEntityManager()`来创建 `EntityManager` 实例

由于**`EntityManagerFactory`是一个==线程安全==的对象**（即多个线程访问同一个`EntityManagerFactory` 对象不会有线程安全问题），并且`EntityManagerFactory` 的创建极其浪费资源，所以在使用JPA编程时，我们可以对`EntityManagerFactory`
的创建进行优化，只需要做到**一个工程只存在一个`EntityManagerFactory`** 即可

#### 2.3 EntityManager

在 JPA 规范中, **`EntityManager`是完成持久化操作的核心对象**。实体类作为普通 java对象，只有在调用`EntityManager`将其持久化后才会变成持久化对象。**EntityManager对象在一组实体类与底层数据源之间进行 O/R 映射的管理**。它可以用来管理和更新 Entity Bean, 根椐主键查找 Entity Bean, 还可以通过JPQL语句查询实体。

我们可以通过调用`EntityManager`的方法完成**获取事务**，以及**持久化数据库**的操作

- **`getTransaction` **: 获取事务对象
- **`persist`** ： 保存操作
- **`find/getReference` **： 根据id查询
- **`merge` **： 更新操作
- **`remove`** ： 删除操作

#### 2.4 EntityTransaction

在 JPA 规范中, EntityTransaction是完成事务操作的核心对象，对于EntityTransaction在我们的java代码中承接的功能比较简单

- `begin`：开启事务
- `commit`：提交事务
- `rollback`：回滚事务



### 3 JPA的CRUD入门案例

#### 3.1 Maven坐标导入

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.hibernate.version>5.0.7.Final</project.hibernate.version>
</properties>

<dependencies>
    <!-- Mysql and MariaDB -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.12</version>
    </dependency>

    <!-- hibernate对jpa的支持包 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>${project.hibernate.version}</version>
    </dependency>

    
    <!-- c3p0 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-c3p0</artifactId>
        <version>${project.hibernate.version}</version>
    </dependency>
    <!-- junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!-- log日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

#### 3.2 persistence.xml配置文件

1. JPA的核心配置文件：`META-INF/persistence.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
   
       <!--配置持久化单元
           name：持久化单元名称
           transaction-type：事务类型
               RESOURCE_LOCAL：本地事务管理
               JTA：分布式事务管理 -->
       <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
           <!--配置JPA规范的服务提供商（这里是Hibernate） -->
           <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
           <properties>
               <!-- 数据库驱动 -->
               <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
               <!-- 数据库地址 -->
               <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3308/jpa?serverTimezone=GMT%2B8" />
               <!-- 数据库用户名 -->
               <property name="javax.persistence.jdbc.user" value="root" />
               <!-- 数据库密码 -->
               <property name="javax.persistence.jdbc.password" value="w111151" />
   
   
               <!--JPA提供者的可选配置：我们的JPA规范的提供者为hibernate，所以JPA的核心配置中兼容Hibernate的配置-->
               <!--显示sql，false | true-->
               <property name="hibernate.show_sql" value="true" />
               <property name="hibernate.format_sql" value="true" />
               <!--自动创建数据库表
                   create : 程序运行时创建数据库表（如果有表，先删除表再创建）
                   update ：程序运行时创建表（如果有表，不会创建表）
                   none   ：不会创建表  -->
               <property name="hibernate.hbm2ddl.auto" value="update" />
           </properties>
   
       </persistence-unit>
   </persistence>
   ```



#### 3.3 ==注解实体类和数据库映射关系==

1. 创建客户的数据库表（不创建也行）

2. **在实体类上使用JPA注解的形式配置映射关系**

   所有的注解都是使用JPA的规范提供的注解，导入`javax.persistence`下注解包

   ```java
   @Entity
   @Table(name = "cst_customer")
   public class Customer {
       
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       @Column(name = "cust_id")
       private Long custId; //客户的主键
   
       @Column(name = "cust_name")
       private String custName;//客户名称
   
       @Column(name = "cust_source")
       private String custSource;//客户来源
   
       @Column(name = "cust_level")
       private String custLevel;//客户级别
   
       @Column(name = "cust_industry")
       private String custIndustry;//客户所属行业
   
       @Column(name = "cust_phone")
       private String custPhone;//客户的联系方式
   
       @Column(name = "cust_address")
       private String custAddress;//客户地址
       
       //getter setter   
   }
   ```

   - **`@Entity`**

     - 作用：指定当前类是**实体类**。

   - **`@Table`**

     - 作用：指定**实体类和表之间的对应关系**。
     - 属性：
       - `name`：指定数据库**表的名称**，不指定则为类名首字母小写

   - **`@Id`**

     - 作用：指定当前字段是**主键**。

   - **`@GeneratedValue`**

     - 作用：指定**主键的生成策略**。

     - 属性：

       - **`strategy`** ：指定主键生成策略。JPA提供的四种标准用法为`TABLE`,`SEQUENCE`,`IDENTITY`,`AUTO`。

         - **`IDENTITY`**：主键由数据库**自动生成**（主要是自动增长型，必须数据库底层支持，如MySQL）

           ```java
           @Id  
           @GeneratedValue(strategy = GenerationType.IDENTITY) 
           private Long custId;
           ```

         - **`SEQUENCE`**：根据底层数据库的**序列**来生成主键，条件是数据库支持序列，如Oracle。

           ```java
           @Id  
           @GeneratedValue(strategy = GenerationType.SEQUENCE,generator="payablemoney_seq")  
           @SequenceGenerator(name="payablemoney_seq", sequenceName="seq_payment")  
           private Long custId;
           ```

           ```java
           //@SequenceGenerator源码中的定义
           @Target({TYPE, METHOD, FIELD})   
           @Retention(RUNTIME)  
           public @interface SequenceGenerator {  
               //表示该表主键生成策略的名称，它被引用在@GeneratedValue中设置的“generator”值中
               String name();  
               //属性表示生成策略用到的数据库序列名称。
               String sequenceName() default "";  
               //表示主键初识值，默认为0
               int initialValue() default 0;  
               //表示每次主键值增加的大小，例如设置1，则表示每次插入新记录后自动加1，默认为50
               int allocationSize() default 50;  
           }
           ```

         - `AUTO`：主键由程序控制（测试了下使用的是TABLE）

           ```java
           @Id  
           @GeneratedValue(strategy = GenerationType.AUTO)  
           private Long custId;
           ```

         - `TABLE`：使用一个特定的数据库表格来保存主键

           ```java
           @Id  
           @GeneratedValue(strategy = GenerationType.TABLE, generator="payablemoney_gen")  
           @TableGenerator(name = "pk_gen",  
                           table="tb_generator",  
                           pkColumnName="gen_name",  
                           valueColumnName="gen_value",  
                           pkColumnValue="PAYABLEMOENY_PK",  
                           allocationSize=1  
                          ) 
           private Long custId;
           ```

           ```java
           //@TableGenerator的定义：
           @Target({TYPE, METHOD, FIELD})   
           @Retention(RUNTIME)  
           public @interface TableGenerator {  
               //表示该表主键生成策略的名称，它被引用在@GeneratedValue中设置的“generator”值中
               String name();  
               //表示表生成策略所持久化的表名，例如，这里表使用的是数据库中的“tb_generator”。
               String table() default "";  
               //catalog和schema具体指定表所在的目录名或是数据库名
               String catalog() default "";  
               String schema() default "";  
               //属性的值表示在持久化表中，该主键生成策略所对应键值的名称。例如在“tb_generator”中将“gen_name”作为主键的键值
               String pkColumnName() default "";  
               //属性的值表示在持久化表中，该主键当前所生成的值，它的值将会随着每次创建累加。例如，在“tb_generator”中将“gen_value”作为主键的值 
               String valueColumnName() default "";  
               //属性的值表示在持久化表中，该生成策略所对应的主键。例如在“tb_generator”表中，将“gen_name”的值为“CUSTOMER_PK”。 
               String pkColumnValue() default "";  
               //表示主键初识值，默认为0。 
               int initialValue() default 0;  
               //表示每次主键值增加的大小，例如设置成1，则表示每次创建新记录后自动加1，默认为50。
               int allocationSize() default 50;  
               UniqueConstraint[] uniqueConstraints() default {};  
           } 
           /*==================================================================*/
           //这里应用表tb_generator，定义为 ：
           CREATE TABLE  tb_generator (  
               id NUMBER NOT NULL,  
               gen_name VARCHAR2(255) NOT NULL,  
               gen_value NUMBER NOT NULL,  
               PRIMARY KEY(id)  
           )
           
           ```

   - **`@Column`**

     - 作用：指定实体类属性和数据库表之间的对应关系

     - 属性：

       - `name`：指定数据库表的列名称。不指定则为属性名
       - `unique`：是否唯一  
       - `nullable`：是否可以为空 
       - `inserttable`：是否可以插入          		
       - `updateable`：是否可以更新          		
       - `columnDefinition`: 定义建表时创建此列的DDL          		
       - `secondaryTable`: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境[重点]

       

   - **`@OneToMany`**：建立一对多的关系映射

     - 属性：
       - **`mappedBy`**：指定**在从表实体类中引用主表对象的名称**，即**主表放弃外键维护**或**被维护表放弃维护中间表**
       - **`fetch`**：指定是否采用**延迟加载**，枚举类`FetchType`
         - `LAZY`（**默认**），`EAGER`
       - **`cascade`**：指定要使用的**级联**操作，枚举类`CascadeType`
         - `ALL`，`PERSIST`，`MERGE`，`REMOVE`，`REFRESH`，`DETACH`；
       - `orphanRemoval`：是否使用孤儿删除，true；false（默认）
       - `targetEntity`：指定多的多方的类的字节码，可省略会自动推倒

   - **`@ManyToOne`**：建立多对一的关系

     - 属性：
       - `fetch`：指定是否采用**延迟**加载（同上）
       - `cascade`：指定要使用的**级联**操作（同上）
       - `optional`：关联是否可选。如果设置为false，则必须始终存在非空关系。默认为true
       - `targetEntity`：指定一的一方实体类字节码，可省略会自动推导

   - **`ManyToMany`**：用于映射多对多关系

     - 属性：
       - `fetch`：指定是否采用**延迟**加载（同上）
       - `cascade`：指定要使用的**级联**操作（同上）
       - `targetEntity`：配置目标的实体类。映射多对多的时候不用写。可省略会自动推导

   - `OneToOne`

   - **`@JoinColumn`**：用于定义**主键字段和外键字段的对应关系**，主要用于**从表中定义外键的引用**

     - 属性：
       - `name`：指定**外键字段的名称**，不配置则根据被维护表中属性名和`referencedColumnName`组合
       - `referencedColumnName`：指定**引用主表的主键字段名称**
       - `unique`：是否唯一。默认false
       - `nullable`：是否允许为空。默认true。
       - `insertable`：是否允许插入。默认true。
       - `updatable`：是否允许更新。默认true。
       - `columnDefinition`：列的定义信息。

   - **`@JoinTable`**：针对**中间表的配置**，主要**用于维护表来设置**

     - 属性：
       - `name`：配置**中间表的名称**，不配置则自动根据维护表和被维护表的类名组合来生成
       - `@joinColumns`：需要中间表的外键字段**关联当前实体类所对应表的主键字段**
       - `@inverseJoinColumn`：需要中间表的外键字段**关联对方表的主键字段**

#### 3.4 基本CRUD

1. 工具类

   ```java
   public final class JPAUtil {
       // JPA的实体管理器工厂：相当于Hibernate的SessionFactory
       private static EntityManagerFactory em;
       // 使用静态代码块赋值
       static {
           // 注意：该方法参数必须和persistence.xml中persistence-unit标签name属性取值一致
           em = Persistence.createEntityManagerFactory("myJpa");
       }
   
       /**
   	 * 使用管理器工厂生产一个管理器对象
   	 */
       public static EntityManager getEntityManager() {
           return em.createEntityManager();
       }
   }
   
   ```

2. **CRUD**

   ```java
   /**
    * Jpa的操作步骤
    * 1.加载配置文件创建工厂（实体管理器工厂）对象
    * 2.通过实体管理器工厂获取实体管理器
    * 3.获取事务对象，开启事务
    * 4.完成增删改查操作
    * 5.提交事务（回滚事务）
    * 6.释放资源
    **/
   public class JPATest {
   
       private EntityManager entityManager;
       private EntityTransaction transaction;
   
       @Before
       public void init() {
           //通过工具类获取EntityManager对象
           entityManager = JPAUtil.getEntityManager();
           //获取事务对象，开启事务
           transaction = entityManager.getTransaction();
           transaction.begin();
       }
   
       @After
       public void destroy() {
           //释放资源，工厂对象不用释放，其他方法还需要使用
           entityManager.close();
       }
   
       /**
        * 保存客户
        */
       @Test
       public void testSave() {
           Customer customer = new Customer();
           customer.setCustName("博学谷");
           customer.setCustPhone("10086");
           try {
               //保存
               entityManager.persist(customer);
               //提交事务
               transaction.commit();
           } catch (Exception e) {
               //或回滚事务
               transaction.rollback();
               e.printStackTrace();
           }
       }
   
       /**
        * 根据ID查询客户
        * find为立即加载
        */
       @Test
       public void testFind() {
           try {
               //第一个参数为要封装的对象的字节码，第二个参数为主键
               Customer customer = entityManager.find(Customer.class, 3L);
               System.out.println(customer.getCustName());
               transaction.commit();
           } catch (Exception e) {
               e.printStackTrace();
               transaction.rollback();
           }
       }
   
       /**
        * 根据ID查询客户
        * getReference为延迟加载（懒加载），使用的时候才查询数据库。一般使用这个！
        * （IDEA需要去掉Debug中几个选项才可以看到）
        */
       @Test
       public void testReference() {
           try {
               //第一个参数为要封装的对象的字节码，第二个参数为主键
               Customer customer = entityManager.getReference(Customer.class, 4L);
               System.out.println(customer.getCustName());
               transaction.commit();
           } catch (Exception e) {
               e.printStackTrace();
               transaction.rollback();
           }
       }
   
       /**
        * 跟新客户信息（先根据ID查询，再修改对象并Merge更新）
        */
       @Test
       public void testMerge() {
           try {
               Customer customer = entityManager.getReference(Customer.class, 3L);
               customer.setCustIndustry("IT666");
               entityManager.merge(customer);
               transaction.commit();
           } catch (Exception e) {
               e.printStackTrace();
               transaction.rollback();
           }
       }
   
       /**
        * 删除客户（要先根据ID查询，再删除该对象）
        */
       @Test
       public void testRemove() {
           try {
               Customer customer = entityManager.getReference(Customer.class, 4L);
               entityManager.remove(customer);
               transaction.commit();
           } catch (Exception e) {
               e.printStackTrace();
               transaction.rollback();
           }
       }
   }
   
   ```

#### 3.5 JPQL查询

JPQL（Java Persistence Query Language）。基于首次在EJB2.0中引入的EJB查询语言(EJB QL)，Java持久化查询语言(JPQL)是一种可移植的查询语言，旨在以面向对象表达式语言的表达式，将SQL语法和简单查询语义绑定在一起，使用这种语言编写的查询是可移植的，可以被编译成所有主流数据库服务器上的SQL。

其特征与原生SQL语句类似，并且完全**面向对象**，通过**类名**和**属性**访问，而**不是表名和表中字段**。（不支持`SELECT *`）

```java
/**
 * 进行jpql查询
 * 1.根据jpql语句创建query查询对象
 * 2.对参数进行赋值
 * 3.查询，并得到返回结果
 **/
public class JPQLTest {

    private EntityManager entityManager;
    private EntityTransaction transaction;

    @Before
    public void init() {
        //通过工具类获取EntityManager对象
        entityManager = JPAUtil.getEntityManager();
        //获取事务对象，开启事务
        transaction = entityManager.getTransaction();
        transaction.begin();
    }

    @After
    public void destroy() {
        //释放资源，工厂对象不用释放，其他方法还需要使用
        entityManager.close();
    }

    /**
     * 查找所有
     */
    @Test
    public void testFindAll() {
        try {
            String jpql = "from cn.itcast.domain.Customer";//也可以省略掉全限定类名，只写类名
            //创建Query查询对象，这个对象才是执行jpql的对象
            Query query = entityManager.createQuery(jpql);
            List<Customer> resultList = query.getResultList();

            transaction.commit();
            for (Customer customer : resultList) {
                System.out.println(customer.getCustName());
            }
        } catch (Exception e) {
            transaction.rollback();
            e.printStackTrace();
        }
    }

    /**
     * 排序查询（根据ID倒序查询所有客户）
     */
    @Test
    public void testOrder() {
        try {
            String jpql = "from Customer order by custId desc";//也可以省略掉全限定类名，只写类名
            //创建Query查询对象，这个对象才是执行jpql的对象
            Query query = entityManager.createQuery(jpql);
            List<Customer> resultList = query.getResultList();

            transaction.commit();
            for (Customer customer : resultList) {
                System.out.println(customer.getCustName());
            }
        } catch (Exception e) {
            transaction.rollback();
            e.printStackTrace();
        }
    }

    /**
     * 统计查询
     */
    @Test
    public void testCount() {
        try {
            String jpql = "select count(custId) from Customer";//也可以省略掉全限定类名，只写类名
            //创建Query查询对象，这个对象才是执行jpql的对象
            Query query = entityManager.createQuery(jpql);
            Long count = (Long) query.getSingleResult();

            transaction.commit();
            System.out.println(count);

        } catch (Exception e) {
            transaction.rollback();
            e.printStackTrace();
        }
    }

    /**
     * 分页查询
     */
    @Test
    public void testPage() {
        try {
            String jpql = "from Customer";//也可以省略掉全限定类名，只写类名
            //创建Query查询对象，这个对象才是执行jpql的对象
            Query query = entityManager.createQuery(jpql);
            query.setFirstResult(0);//起始索引
            query.setMaxResults(2);//每页查询条数
            List<Customer> resultList = query.getResultList();//此处为查询前两条

            transaction.commit();
            for (Customer customer : resultList) {
                System.out.println(customer.getCustName());
            }
        } catch (Exception e) {
            transaction.rollback();
            e.printStackTrace();
        }
    }

    /**
     * 条件查询
     */
    @Test
    public void testCondition() {
        try {
            String jpql = "from Customer where custName like ?";//也可以省略掉全限定类名，只写类名
            //创建Query查询对象，这个对象才是执行jpql的对象
            Query query = entityManager.createQuery(jpql);
            query.setParameter(1, "传智播客%");//第1个参数：占位符索引位置，从1开始；第2个参数：值
            List<Customer> resultList = query.getResultList();//此处为查询前两条

            transaction.commit();
            for (Customer customer : resultList) {
                System.out.println(customer.getCustName());
            }
        } catch (Exception e) {
            transaction.rollback();
            e.printStackTrace();
        }
    }
}
```





### 4 Spring Data JPA

> Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现,在实际的工作工程中，推荐使用Spring Data JPA + ORM（如：Hibernate）完成操作，这样在切换不同的ORM框架时提供了极大的方便，同时也使数据库层操作更加简单，方便解耦。
>
> Spring Data JPA 极大简化了数据库访问层代码。 如何简化的呢？ 使用了Spring Data JPA，我们的dao层中只需要写接口，就自动具有了增删改查、分页查询等方法。

Spring Data JPA 封装了 JPA ，JPA 规范的一个实现为 Hibernate ，Hibernate 又封装了 JDBC ，JDBC 来操作数据库如MySQL

#### 4.1 Maven坐标导入

```xml
<properties>
    <spring.version>5.0.2.RELEASE</spring.version>
    <hibernate.version>5.0.7.Final</hibernate.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <c3p0.version>0.9.1.2</c3p0.version>
    <mysql.version>8.0.12</mysql.version>
</properties>

<dependencies>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!-- spring beg -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.6.8</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- spring对orm框架的支持包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <!-- spring end -->

    <!-- hibernate beg -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>${hibernate.version}</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>${hibernate.version}</version>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>5.2.1.Final</version>
    </dependency>
    <!-- hibernate end -->

    <!-- c3p0 beg -->
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>${c3p0.version}</version>
    </dependency>
    <!-- c3p0 end -->

    <!-- log end -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <!-- log end -->


    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>

    <!-- spring data jpa 的坐标-->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>1.9.0.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <!-- el beg 使用spring data jpa 必须引入 -->
    <dependency>
        <groupId>javax.el</groupId>
        <artifactId>javax.el-api</artifactId>
        <version>2.2.4</version>
    </dependency>

    <dependency>
        <groupId>org.glassfish.web</groupId>
        <artifactId>javax.el</artifactId>
        <version>2.2.4</version>
    </dependency>
    <!-- el end -->
</dependencies>

```



#### 4.2 Spring Data JPA 配置文件

applicationContext.xml

```xml
<!--spring 和 spring data jpa的配置-->

<!-- 1.创建entityManagerFactory对象交给spring容器管理-->
<bean id="entityManagerFactoty" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!--配置的扫描的包（实体类所在的包） -->
    <property name="packagesToScan" value="cn.itcast.domain" />
    <!-- jpa的实现厂家 -->
    <property name="persistenceProvider">
        <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
    </property>

    <!--jpa的供应商适配器 -->
    <property name="jpaVendorAdapter">
        <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
            <!--配置是否自动创建数据库表 -->
            <property name="generateDdl" value="false" />
            <!--指定数据库类型 -->
            <property name="database" value="MYSQL" />
            <!--数据库方言：支持的特有语法 -->
            <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect" />
            <!--是否显示sql -->
            <property name="showSql" value="true" />
        </bean>
    </property>

    <!--jpa的方言 ：高级的特性 -->
    <property name="jpaDialect" >
        <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
    </property>

    <!--注入jpa的配置信息。在多表操作时为了方便会使用到create和update
            加载jpa的基本配置信息和jpa实现方式（hibernate）的配置信息
            hibernate.hbm2ddl.auto : 自动创建数据库表
                create ： 每次都会重新创建数据库表
                update：有表不会重新创建，没有表会重新创建表
        -->
    <property name="jpaProperties" >
        <props>
            <prop key="hibernate.hbm2ddl.auto">create</prop>
        </props>
    </property>

</bean>

<!--2.创建数据库连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="root"/>
    <property name="password" value="w111151"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3308/jpa?serverTimezone=GMT%2B8"/>
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
</bean>

<!--3.整合spring dataJpa-->
<jpa:repositories base-package="cn.itcast.dao" transaction-manager-ref="transactionManager"
                  entity-manager-factory-ref="entityManagerFactoty" />

<!--4.配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="entityManagerFactoty"/>
</bean>

<!----------------------------5.声明式事务，暂时用不到，直接使用注解配置@Transactional ------------------------->
<!-- 4.txAdvice-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="save*" propagation="REQUIRED"/>
        <tx:method name="insert*" propagation="REQUIRED"/>
        <tx:method name="update*" propagation="REQUIRED"/>
        <tx:method name="delete*" propagation="REQUIRED"/>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!-- 5.aop-->
<aop:config>
    <aop:pointcut id="pointcut" expression="execution(* cn.itcast.service.*.*(..))" />
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut" />
</aop:config>
<!---------------------------------------暂时用不到---------------------------------------->

<!-- 6. 配置包扫描-->
<context:component-scan base-package="cn.itcast"/>
```



#### 4.3 注解实体类和数据库映射关系

同3.3一致

```java
@Entity
@Table(name = "cst_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId; //客户的主键

    @Column(name = "cust_name")
    private String custName;//客户名称

    @Column(name = "cust_source")
    private String custSource;//客户来源

    @Column(name = "cust_level")
    private String custLevel;//客户级别

    @Column(name = "cust_industry")
    private String custIndustry;//客户所属行业

    @Column(name = "cust_phone")
    private String custPhone;//客户的联系方式

    @Column(name = "cust_address")
    private String custAddress;//客户地址

    //getter setter   
}
```



#### 4.4 Spring Data JPA规范Dao接口

> 别忘了接口之间是可以多继承的！！！

```java
/**
 * JpaRepository<实体类类型，主键类型>：用来完成基本CRUD操作
 * JpaSpecificationExecutor<实体类类型>：用于复杂查询（分页等查询操作）
 */
public interface CustomerDao extends JpaRepository<Customer,Long>, JpaSpecificationExecutor<Customer> {
}
```

![](images/springdatajpa的运行过程.png)

#### 4.5 基本CRUD和复杂查询

##### 1、接口中的定义好的方法

> 以下是继承了JpaRepository接口后获得的

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")//locations可省略
public class CustomerDaoTest {

    @Autowired
    private CustomerDao customerDao;

    /**
     * 根据ID查询（立即加载，底层调用find()方法）
     */
    @Test
    public void testFindOne() {
        Customer customer = customerDao.findOne(3L);
        System.out.println(customer);
    }

    /**
     * 根据ID查询（延迟加载，底层调用getReference()方法）
     * 返回的是动态代理对象；什么时候用，什么时候查询（IDEA需要去掉Debug中几个选项才可以看到）
     */
    @Test
    @Transactional
    public void testGetOne() {
        Customer customer = customerDao.getOne(3L);
        System.out.println(customer);
    }

    /**
     * 查询所有
     */
    @Test
    public void testFindAll() {
        List<Customer> customerList = customerDao.findAll();
        System.out.println(customerList);
    }

    /**
     * 保存或更新（！！！不要直接使用这个来更新，否则会将其他没有值的数据置null。）
     * 根据传递的对象是否包含主键ID，不包含则是保存；包含则是先根据ID查询数据，并更新
     *
     * 更新需要根据ID手动查找，然后修改再保存！！！
     */
    @Test
    public void testSave() {
        Customer customer = new Customer();
        customer.setCustName("黑马程序员");
        customer.setCustAddress("西安");
        customerDao.save(customer);
    }
    @Test
    public void testUpdate() {
        Customer customer = new Customer();
        customer.setCustId(8L);
        customer.setCustName("黑马程序员牛逼");
        customer.setCustAddress("西安");
        customer.setCustLevel("VIP");
        customerDao.save(customer);
    }

    /**
     * 根据ID删除（底层是先查询，若有再删除）
     */
    @Test
    public void testDelete() {
        customerDao.delete(8L);
    }

    /**
     * 统计查询
     */
    @Test
    public void testCount() {
        long count = customerDao.count();
        System.out.println(count);
    }

    /**
     * 根据ID判断是否存在该数据（可以根据查询的对象是否为null；或统计出的数量是否大于0（这里底层用的是这个））
     */
    @Test
    public void testExists() {
        boolean exists = customerDao.exists(7L);
        System.out.println(exists);
    }
}
```



##### 2、JPQL的查询方式

- 特有的查询：需要在**Dao接口上配置方法**
- 在新添加的方法上，使用**`@Query`注解的形式配置JPQL查询语句**

```java
public interface CustomerDao extends JpaRepository<Customer,Long>, JpaSpecificationExecutor<Customer> {

    /**
     * 根据客户名称查询客户。使用JPQL查询
     * ?1代表参数的占位符，其中1对应方法中的参数索引
     */
    @Query("from Customer where custName = ?1")
    Customer findByName(String custName);

    /**
     * 根据客户名称和客户ID查询客户。使用JPQL查询
     * ?1代表参数的占位符，其中1对应方法中的参数索引。不指定则按照方法中参数位置对应
     */
    @Query("from Customer where custName = ?1 and custId=?2")
    Customer findByNameAndId(String custName,Long custId);


    /**
     * 根据ID更新客户名称。使用JPQL
     * ?1代表参数的占位符，其中1对应方法中的参数索引。不指定则按照方法中参数位置对应
     */
    @Query("update Customer set custName = ?1 where custId = ?2")
    @Modifying//代表是更新操作
    void updateNameById(String custName,Long custId);
}

```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")//locations可省略
public class JPQLTest {

    @Autowired
    private CustomerDao customerDao;

    @Test
    public void testFindByName() {
        Customer customer = customerDao.findByName("传智播客6");
        System.out.println(customer);
    }

    @Test
    public void testFindByNameAndId() {
        Customer customer = customerDao.findByNameAndId("传智播客", 5L);
        System.out.println(customer);
    }

    /**
     * Spring Data JPA中使用JPQL完成 更新、删除操作需要手动添加事务，但默认执行后会回滚事务，需要添加注解设置为不回滚
     */
    @Test
    @Transactional
    @Rollback(value = false)
    public void testUpdateNameById() {
        customerDao.updateNameById("传智播客最牛逼", 5L);
    }
}

```



##### 3、SQL 语句查询（了解）

- 特有的查询：需要在**Dao接口上配置方法**
- 在新添加的方法上，使用**`@Query`注解的形式配置SQL查询语句**，还需设置`nativeQuery`属性

```java
public interface CustomerDao extends JpaRepository<Customer,Long>, JpaSpecificationExecutor<Customer> {
    /**
     * 使用SQL查询全部客户/模糊查询
     * nativeQuery：默认false为JPQL查询，true为SQL查询
     */
    //@Query(value = "select * from cst_customer", nativeQuery = true)
    @Query(value = "select * from cst_customer where cust_name like ?1", nativeQuery = true)
    List<Object[]> findBySQL(String name);
}

```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")//locations可省略
public class JPQLTest {

    @Autowired
    private CustomerDao customerDao;
    @Test
    public void testFindBySQL() {
        List<Object[]> list = customerDao.findBySQL("传智播客%");
        for (Object[] obj : list) {
            System.out.println(Arrays.toString(obj));
        }
    }
}

```



##### 4、方法名称规则查询

方法命名规则查询就是根据方法的名字，就能创建查询，是对JPQL更深层次封装。只需要按照Spring Data JPA提供的方法命名规则定义方法的名称，不需要配置JPQL语句就可以完成查询工作。Spring Data JPA在程序执行的时候会根据方法名称进行解析，并自动生成查询语句进行查询。

按照Spring Data JPA 定义的规则，查询方法以**`findBy`**开头，涉及条件查询时，**条件的属性用条件关键字连接**，要注意的是：条件**属性首字母需大写**。框架在进行方法名解析时，会先把方法名多余的前缀截取掉，然后对剩下部分进行解析。

```java
public interface CustomerDao extends JpaRepository<Customer,Long>, JpaSpecificationExecutor<Customer> {
    /**
     * 方法名称查询（精准匹配）
     */
    Customer findByCustName(String custName);

    /**
     * 方法名称查询（like模糊匹配）
     */
    List<Customer> findByCustNameLike(String custName);

    /**
     * 方法名称查询（使用客户名称模糊匹配和客户行业精准匹配）
     */
    List<Customer> findByCustNameLikeAndCustIndustry(String custName,String custIndustry);
}

```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")//locations可省略
public class JPQLTest {

    @Autowired
    private CustomerDao customerDao;

    /**
     * 测试方法命名查询
     */
    @Test
    public void testFindByCustName(){
        Customer customer = customerDao.findByCustName("传智播客6");
        System.out.println(customer);
    }

    /**
     * 测试方法命名查询（模糊匹配）
     */
    @Test
    public void testFindByCustNameLike(){
        List<Customer> customerList = customerDao.findByCustNameLike("传智%");
        System.out.println(customerList);
    }

    /**
     * 测试方法命名查询（使用客户名称模糊匹配和客户行业精准匹配）
     */
    @Test
    public void testFindByCustNameLikeAndCustIndustry(){
        List<Customer> customerList = customerDao.findByCustNameLikeAndCustIndustry("传智%","IT");
        System.out.println(customerList);
    }
}

```



具体的关键字，使用方法和生产成SQL如下表所示

|    **Keyword**    |                 **Sample**                 |                           **JPQL**                           |
| :---------------: | :----------------------------------------: | :----------------------------------------------------------: |
|        And        |         findByLastnameAndFirstname         |        …   where x.lastname = ?1 and x.firstname = ?2        |
|        Or         |         findByLastnameOrFirstname          |        …   where x.lastname = ?1 or x.firstname = ?2         |
|     Is,Equals     | findByFirstnameIs,   findByFirstnameEquals |                  …   where x.firstname = ?1                  |
|      Between      |           findByStartDateBetween           |           …   where x.startDate between ?1 and ?2            |
|     LessThan      |             findByAgeLessThan              |                     …   where x.age < ?1                     |
|   LessThanEqual   |           findByAgeLessThanEqual           |                     …   where x.age ⇐ ?1                     |
|    GreaterThan    |            findByAgeGreaterThan            |                     …   where x.age > ?1                     |
| GreaterThanEqual  |         findByAgeGreaterThanEqual          |                    …   where x.age >= ?1                     |
|       After       |            findByStartDateAfter            |                  …   where x.startDate > ?1                  |
|      Before       |           findByStartDateBefore            |                  …   where x.startDate < ?1                  |
|      IsNull       |              findByAgeIsNull               |                   …   where x.age is null                    |
| IsNotNull,NotNull |            findByAge(Is)NotNull            |                   …   where x.age not null                   |
|       Like        |            findByFirstnameLike             |                …   where x.firstname like ?1                 |
|      NotLike      |           findByFirstnameNotLike           |              … where   x.firstname not like ?1               |
|   StartingWith    |        findByFirstnameStartingWith         | …   where x.firstname like ?1 (parameter bound with appended %) |
|    EndingWith     |         findByFirstnameEndingWith          | …   where x.firstname like ?1 (parameter bound with prepended %) |
|    Containing     |         findByFirstnameContaining          | …   where x.firstname like ?1 (parameter bound wrapped in %) |
|      OrderBy      |        findByAgeOrderByLastnameDesc        |        …   where x.age = ?1 order by x.lastname desc         |
|        Not        |             findByLastnameNot              |                  …   where x.lastname <> ?1                  |
|        In         |        findByAgeIn(Collection ages)        |                    …   where x.age in ?1                     |
|       NotIn       |       findByAgeNotIn(Collection age)       |                  …   where x.age not in ?1                   |
|       TRUE        |             findByActiveTrue()             |                  …   where x.active = true                   |
|       FALSE       |            findByActiveFalse()             |                  …   where x.active = false                  |
|    IgnoreCase     |         findByFirstnameIgnoreCase          |           …   where UPPER(x.firstame) = UPPER(?1)            |





#### 4.6 Specification 动态查询

> 以下是继承了JpaSpecificationExecutor接口后获得的

Spring Data JPA中可以通过`JpaSpecificationExecutor`接口查询。相比JPQL，其优势是类型安全，更加的面向对象。

- `T findOne(Specification<T> spec);`  //查询单个对象
- `List<T> findAll(Specification<T> spec);`  //查询列表
- `Page<T> findAll(Specification<T> spec, Pageable pageable);`
  - Pageable：分页参数
  - 返回值：Spring Data JPA提供的分页Bean
- `List<T> findAll(Specification<T> spec, Sort sort);`//排序查询
- `long count(Specification<T> spec);`//统计查询

对于`JpaSpecificationExecutor`，这个接口基本是围绕着`Specification`接口来定义的。我们可以简单的理解为，**`Specification`构造的就是查询条件**。

```java
/**
  *	root	：Root接口，代表查询的根对象，可以通过root获取实体中的属性
  *	query	：代表一个顶层查询对象，用来自定义查询（用的少，但是很强大）
  *	cb		：用来构建查询，此对象里有很多条件方法
  **/
public Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);

```

```java
/**
 * 1.实现Specification接口（提供泛型：查询的对象类型）
 * 2.实现toPredicate方法（构造查询条件）
 * 3.需要借助方法参数中的两个参数（
 * root：获取需要查询的对象属性（不是数据库中字段名！！）
 * CriteriaBuilder：构造查询条件的，内部封装了很多的查询条件（模糊匹配，精准匹配）
 **/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class SpecTest {

    @Autowired
    private CustomerDao customerDao;

    /**
     * 单条件查询，根据客户名称查询单个客户对象（equal）
     */
    @Test
    public void testFindOne1() {
        Specification<Customer> specification = new Specification<Customer>() {
            public Predicate toPredicate(Root<Customer> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                //1.获取比较的属性
                Path<Object> custName = root.get("custName");
                //2.构造查询条件
                //第1个参数为比较的属性（Path对象）；第2个参数为要比较的值
                return criteriaBuilder.equal(custName, "博学谷");
            }
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }

    /**
     * 多条件查询，根据客户名和客户所属行业查询单个客户（equal，and）
     */
    @Test
    public void testFindOne2() {

        Specification<Customer> specification = (root, criteriaQuery, criteriaBuilder) -> {
            Path<Object> custName = root.get("custName");
            Path<Object> custIndustry = root.get("custIndustry");

            Predicate predicate1 = criteriaBuilder.equal(custName, "传智播客");
            Predicate predicate2 = criteriaBuilder.equal(custIndustry, "IT");
            //组合谓词
            return criteriaBuilder.and(predicate1, predicate2);//还有or等等方法来组合
        };
        Customer customer = customerDao.findOne(specification);
        System.out.println(customer);
    }

    /**
     * 普通查询；排序查询；分页查询，根据客户名模糊匹配查询客户列表（like）；
     * equal ：直接的到path对象（属性），然后进行比较即可
     * gt，lt,ge,le,like : 得到Path对象，根据Path指定比较的参数类型，再去进行比较
     */
    @Test
    public void testFindAll1() {
        Specification<Customer> specification = ((root, criteriaQuery, criteriaBuilder) -> {
            Path<Object> custName = root.get("custName");
            return criteriaBuilder.like(custName.as(String.class), "%传智%");//或在获取Path时直接封装为String类型
        });
        //普通查询
        List<Customer> customerList1 = customerDao.findAll(specification);

        //排序查询
        //第一个参数：排序的顺序（倒序，正序）；第二个参数：排序的属性名称（不是字段名！）
        Sort sort = new Sort(Sort.Direction.DESC, "custId");
        List<Customer> customerList2 = customerDao.findAll(specification, sort);

        //分页查询
        Pageable pageable = new PageRequest(0, 2);//页码（从0开始），每页显示条数
        Page<Customer> customerPage = customerDao.findAll(specification, pageable);
        System.out.println("总记录数:" + customerPage.getTotalElements());
        System.out.println("总页数:" + customerPage.getTotalPages());
        System.out.println("分页查询的数据:" + customerPage.getContent());
    }


    /**
	 * Specification的多表查询
	 */
    @Test
    public void testFind() {
        Specification<LinkMan> specification = (root, criteriaQuery, criteriaBuilder) ->  {
                //Join代表链接查询，通过root对象获取
                //创建的过程中，第一个参数为关联对象的属性名称，第二个参数为连接查询的方式（inner，left，right）
                //JoinType.LEFT : 左外连接,JoinType.INNER：内连接,JoinType.RIGHT：右外连接
                Join<LinkMan, Customer> join = root.join("customer",JoinType.INNER);
                return criteriaBuilder.like(join.get("custName").as(String.class),"传智播客1");
            }
        };
        List<LinkMan> list = linkManDao.findAll(specification);
        for (LinkMan linkMan : list) {
            System.out.println(linkMan);
        }
    }
}

```

> 方法对应关系
>
> | 方法名称                    | Sql对应关系            |
> | --------------------------- | ---------------------- |
> | equle                       | filed =   value        |
> | gt（greaterThan ）          | filed   > value        |
> | lt（lessThan ）             | filed   < value        |
> | ge（greaterThanOrEqualTo ） | filed   >= value       |
> | le（ lessThanOrEqualTo）    | filed   <= value       |
> | notEqule                    | filed   != value       |
> | like                        | filed   like value     |
> | notLike                     | filed   not like value |



#### 4.7 多表操作

在实际开发中，我们数据库的表难免会有相互的关联关系，在操作表的时候就有可能会涉及到多张表的操作。而在这种实现了ORM思想的框架中（如JPA），可以让我们通过操作实体类就实现对数据库表的操作。所以今天我们的学习重点是：掌握配置实体之间的关联关系。

- 第一步：首先确定两张表之间的关系。（如果关系确定错了，后面做的所有操作就都不可能正确。）

- 第二步：在数据库中实现两张表的关系

- 第三步：在实体类中描述出两个实体的关系

- 第四步：配置出实体类和数据库表的关系映射（重点）



##### 1、一对多

案例：客户和联系人的案例（一对多关系，一个客户可以具有多个联系人，一个联系人从属于一家公司）

- 客户：一家公司
- 联系人：这家公司的员工

分析步骤：

1. 明确表关系：一对多关系
2. 确定表关系（外键）
   - **主表**：客户表
   - **从表**：联系人表，在从表添加**外键**
3. 编写实体类，再实体类中描述表关系（**包含关系**）
   - 客户：再客户的实体类中包含一个联系人的集合
   - 联系人：在联系人的实体类中包含一个客户的对象
4. 配置映射关系
   - 使用 **JPA 注解配置一对多映射关系**

```java
@Entity
@Table(name = "cst_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId; //客户的主键

    
    @OneToMany(mappedBy = "customer")//主表放弃维护外键
    private Set<LinkMan> linkMans = new HashSet<>();

    @Column(name = "cust_name")
    private String custName;//客户名称
    
    ...
}
```

```java
@Entity
@Table(name = "cst_linkman")
public class LinkMan implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "lkm_id")
    private Long lkmId;

    @ManyToOne
    @JoinColumn(name = "lkm_cust_id",referencedColumnName = "cust_id")
    private Customer customer;

    @Column(name = "lkm_name")
    private String lkmName;
    ...
}
```

------

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class One2ManyTest {

    @Autowired
    private CustomerDao customerDao;

    @Autowired
    private LinkManDao linkManDao;

    @Test
    @Transactional//开启事务
    @Rollback(false)//不回滚
    public void testSave() {
        Customer customer = new Customer();
        customer.setCustName("腾讯");
        LinkMan linkMan = new LinkMan();
        linkMan.setLkmName("小马");

        customer.getLinkMans().add(linkMan);
        linkMan.setCustomer(customer);

        customerDao.save(customer);
        linkManDao.save(linkMan);
    }
    
    @Test
    @Transactional
    @Rollback(false)//设置为不回滚
    public void testDelete() {
        customerDao.delete(3L);
    }
}
```

**删除操作的说明如下**

- 删除从表数据：可以随时任意删除

- 删除主表数据：

  - 没有从表数据引用：随便删

  - 有从表数据：

    - 在默认情况下，它会把外键字段置为null，然后删除主表数据。如果在数据库的表结构上，外键字段有非空约束，默认情况就会报错了

    - 如果配置了放弃维护关联关系的权利，则不能删除（与外键字段是否允许为null，没有关系）因为在删除时，它根本不会去更新从表的外键字段了

    - 如果还想删除，使用级联删除引用（在实际开发中，级联删除请慎用！(在一对多的情况下)）

      

**级联**：操作一个对象的同时操作他的关联对象

- 级联操作：
  1.需要**区分操作主体**
  2.需要在操作主体的实体类上（主表或维护表），添加级联属性`cascade`（需要添加到多表映射关系的注解上）
- 级联添加，案例：当我保存一个客户的同时保存联系人（在代码中只需保存客户！）
- 级联删除，案例：当我删除一个客户的同时删除此客户的所有联系人，有中间表会先删除中间表（在代码中只需删除客户！）



##### 2、多对多

案例：用户和角色（多对多关系）

分析步骤：

1. 明确表关系：多对多关系
2. 确定表关系（中间表）
3. 编写实体类，再实体类中描述表关系（**包含关系**）
   - 用户：包含角色的集合
   - 角色：包含用户的集合
4. 配置映射关系
   - 使用 **JPA 注解配置多对多映射关系**

```java
@Entity
@Table//不配置name则为类目首字母小写
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column//不配置字段名则为属性名
    private Long userId;

    @ManyToMany
    //维护中间表
    @JoinTable(
        name = "sys_user_role",
        //当前对象在中间表中的外键
        joinColumns = {@JoinColumn(name = "sys_user_id",referencedColumnName = "userId")},
        //对方对象在中间表中的外键
        inverseJoinColumns = {@JoinColumn(name = "sys_role_id",referencedColumnName = "roleId")}
    )
    private Set<Role> roles = new HashSet<>();
    ...
}

```

```java
@Entity
@Table
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    private Long roleId;
    
    @ManyToMany(mappedBy = "roles")//被维护表放弃维护中间表！
    private Set<User> users = new HashSet<>();
    ...
}

```

其他删除/级联等同一对多，删除时会先删除中间表数据



##### 3、对象导航查询

对象图导航检索方式是根据已经加载的对象，导航到他的关联对象。它利用类与类之间的关系来检索对象。例如：我们通过ID查询方式查出一个客户，可以调用Customer类中的getLinkMans()方法来获取该客户的所有联系人。对象导航查询的使用要求是：两个对象之间必须存在关联关系。

对象导航查询

- **从一方查询多方：默认延迟加载**
- **从多方查询一方：默认立即加载**

可以修改配置将其改为立即加载（不推荐！），**fetch配置在多表关系中主体（或一方或多方）的注解上**，查看3.3



##### 4、Specification的多表查询

代码查看4.6







## 2 Spring Data Redis

> Spring-data-redis是spring大家族的一部分，提供了在srping应用中通过简单的配置访问redis服务，对reids底层开发包(Jedis,  JRedis, and RJC)进行了高度封装，RedisTemplate提供了redis各种操作、异常处理及序列化，支持发布订阅，并对spring 3.1 cache进行了实现

spring-data-redis针对jedis提供了如下功能：

- **连接池自动管理**，提供了一个高度封装的“RedisTemplate”类

- 针对jedis客户端中大量api进行了归类封装，将同一类型操作封装为**operation接口**

  redis存储的是：**key,value格式**的数据，其中**key都是字符串**，**value有5种不同的数据结构**

  - ValueOperations：简单字符串的操作
  - ListOperations：针对List类型的数据操作
  - SetOperations：针对Sets即set类型数据操作
  - ZSetOperations：针对Sorted Sets即排序的set类型数据操作
  - HashOperations：针对Hashes即map类型的数据操作



### 1 环境搭建

- 首先要开启Redis服务

- Maven依赖引入（Spring相关、JUnit），还需要 Jedis 和 SpringDataRedis 依赖

  ```xml
  <!-- 缓存 -->
  <dependency> 
      <groupId>redis.clients</groupId> 
      <artifactId>jedis</artifactId> 
      <version>2.8.1</version> 
  </dependency> 
  <dependency> 
      <groupId>org.springframework.data</groupId> 
      <artifactId>spring-data-redis</artifactId> 
      <version>1.7.2.RELEASE</version> 
  </dependency>	
  ```

- resources/properties目录下的`redis-config.properties`配置文件

  ```properties
  # Redis settings 
  # server IP 
  redis.host=127.0.0.1
  # server port 
  redis.port=6379
  # server pass 
  redis.pass=
  # use dbIndex 
  redis.database=0
  # 控制一个pool最多有多少个状态为idle(空闲的)的jedis实例 
  redis.maxIdle=300
  # 表示当borrow(引入)一个jedis实例时，最大的等待时间，如果超过等待时间(毫秒)，则直接抛出JedisConnectionException；  
  redis.maxWait=3000
  # 在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的  
  redis.testOnBorrow=true
  ```

- resources/spring目录下的Spring配置文件`applicationContext-redis.xml`

  ```xml
  <context:property-placeholder location="classpath*:properties/*.properties" />   
  <!-- redis 相关配置 --> 
  <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">  
      <property name="maxIdle" value="${redis.maxIdle}" />   
      <property name="maxWaitMillis" value="${redis.maxWait}" />  
      <property name="testOnBorrow" value="${redis.testOnBorrow}" />  
  </bean>  
  
  <bean id="JedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" 
        p:host-name="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" p:pool-config-ref="poolConfig"/>  <!--p为p命名空间，可以不用写property。下面的就是不用p命名空间-->
  
  <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">  
      <property name="connectionFactory" ref="JedisConnectionFactory" />  
  </bean>  
  ```

### 2 不同类型value

- String

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:spring/applicationContext-redis.xml")//可以省略locations
  public class TestString {
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Test
      public void setValue(){
          redisTemplate.boundValueOps("StringName").set("itcast");
      }
  
      @Test
      public void getValue(){
          String name = (String) redisTemplate.boundValueOps("StringName").get();
          Assert.assertEquals("itcast",name);//name值为null
      }
  
      @Test
      public void deleteValue(){
          redisTemplate.delete("StringName");//还可以传入集合，删除集合中所有key对应的key-value
      }
  }
  
  ```

- List

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:spring/applicationContext-redis.xml")//可以省略locations
  public class TestList {
  
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Test
      public void rightPushValue(){
          redisTemplate.boundListOps("ListName").rightPushAll(new Object[]{"张三", "李四"});
          redisTemplate.boundListOps("ListName").rightPush("王五");
          //一个一个压栈：[张三, 李四, 王五]
      }
  
      @Test
      public void leftPushValue(){
          redisTemplate.boundListOps("ListName").leftPushAll(new Object[]{"zhangsan", "lisi"});
          redisTemplate.boundListOps("ListName").leftPush("wangwu");
          //一个一个压栈：[wangwu, lisi, zhangsan]
      }
  
      @Test
      public void getValueByIndex(){
          String  listName = (String) redisTemplate.boundListOps("ListName").index(1);//下标从0开始
          System.out.println(listName);
      }
      
      @Test
      public void getValue(){
          List list = redisTemplate.boundListOps("ListName").range(0, -1);
          System.out.println(list);
          list.forEach(System.out::println);
      }
      
      @Test
      public void removeValue(){
          redisTemplate.boundListOps("ListName").remove(2,"李四");//第一个为要删除的数量
      }
      
      @Test
      public void deleteValue(){
          redisTemplate.delete("ListName");//删除ListName对应的List集合。直接打印显示“[]”。还可以传入集合
      }
  }
  
  ```

- Set

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:spring/applicationContext-redis.xml")//可以省略locations
  public class TestSet {
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Test
      public void addValue(){
          redisTemplate.boundSetOps("SetName").add("张三");
          redisTemplate.boundSetOps("SetName").add("李四");
          redisTemplate.boundSetOps("SetName").add("王五");
      }
  
      @Test
      public void getValue(){
          Set setName = redisTemplate.boundSetOps("SetName").members();
          System.out.println(setName);
                  setName.forEach(System.out::println);//方法引用，setName.forEach(o -> System.out.println(o))
      }
  
      @Test
      public void removeValue(){
          redisTemplate.boundSetOps("SetName").remove("张三");//移除Set其中一个
      }
  
      @Test
      public void deleteValue(){
          redisTemplate.delete("SetName");//删除SetName对应的Set集合。直接打印显示“[]”。还可以传入集合
      }
  }
  
  ```

- ZSet（方法和Set基本一致，除了查找有个`range()`方法）

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:spring/applicationContext-redis.xml")//可以省略locations
  public class TestZSet {
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Test
      public void setValue(){
          redisTemplate.boundZSetOps("ZSetName").add("张三",0);
          redisTemplate.boundZSetOps("ZSetName").add("李四",2);
          redisTemplate.boundZSetOps("ZSetName").add("王五",-2);//由小到大排序
      }
  
      @Test
      public void getValue(){
          Set setName = redisTemplate.boundZSetOps("ZSetName").range(0,-1);
          setName.forEach(System.out::println);//方法引用，setName.forEach(o -> System.out.println(o))lambda的简写
      }
  
      @Test
      public void removeValue(){
          redisTemplate.boundZSetOps("ZSetName").remove("张三");//移除ZSet其中一个
      }
  
      @Test
      public void deleteValue(){
          redisTemplate.delete("ZSetName");//删除ZSetName对应的ZSet集合。直接打印显示“[]”。还可以传入集合
      }
  }
  ```

- Hash（即map类型，用得比较多）

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration("classpath:spring/applicationContext-redis.xml")//可以省略locations
  public class TestHash {
  
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Test
      public void putValue(){
          Map<String,String> map = new HashMap<>();
          map.put("a","张三");
          map.put("b","李四");
          redisTemplate.boundHashOps("HashName").putAll(map);
          redisTemplate.boundHashOps("HashName").put("c","王五");
  
      }
  
      @Test
      public void getValueByKey(){
          String o = (String) redisTemplate.boundHashOps("HashName").get("c");
          System.out.println(o);
      }
  
      @Test
      public void getValue(){
          Map hashName = redisTemplate.boundHashOps("HashName").entries();
          System.out.println(hashName);
          hashName.forEach((key, value) -> System.out.println(key+"=="+value));
      }
  
      @Test
      public void removeValue(){
          redisTemplate.boundHashOps("HashName").delete("c");//注意，此处不再是remove方法，是delete方法
  
      }
  
      @Test
      public void deleteValue(){
          redisTemplate.delete("HashName");//删除HashName对应的Map集合。直接打印显示“{}”。还可以传入集合
      }
  }
  
  ```



## 3 全文检索技术-Lucene

### 1 什么是全文检索

#### 1.1 数据分类

- **结构化数据**：指具有固定格式或有限长度的数据，如数据库，元数据等
- **非结构化数据**：指不定长或无固定格式的数据，如邮件，word文档等磁盘上的文件

#### 1.2 结构化数据搜索

​	常见的**结构化数据**也就是**数据库中**的数据。在数据库中搜索很容易实现，通常都是使用sql语句进行查询，而且能很快的得到查询结果（数据库中的数据存储是有规律的，有行有列而且数据格式、数据长度都是固定的）。

#### 1.3 非结构化数据查询方法

- **顺序扫描法**(Serial Scanning)：如对于每一个文档，从头看到尾......

- **全文检索**(Full-text Search)：

  ​	将非结构化数据中的一部分信息提取出来，重新组织，使其变得有一定结构，然后对此有一定结构的数据进行搜索，从而达到搜索相对较快的目的。这部分**从非结构化数据中提取出的然后重新组织的信息**，我们称之**索引**。

  ​	例如：字典。字典的拼音表和部首检字表就相当于字典的索引，对每一个字的解释是非结构化的，如果字典没有音节表和部首检字表，在茫茫辞海中找一个字只能顺序扫描。然而字的某些信息可以提取出来进行结构化处理，比如读音，就比较结构化，分声母和韵母，分别只有几种可以一一列举，于是将读音拿出来按一定的顺序排列，每一项读音都指向此字的详细解释的页数。我们搜索时按结构化的拼音搜到读音，然后按其指向的页数，便可找到我们的非结构化数据——也即对字的解释。

  ​	**这种先建立索引，再对索引进行搜索的过程就叫全文检索(Full-text Search)。**虽然创建索引的过程也是非常耗时的，但是**索引**一旦创建就可以**多次使用**，全文检索主要处理的是查询，所以耗时间创建索引是值得的。

#### 1.4 如何实现全文检索

​	可以使用Lucene实现全文检索。Lucene是apache下的一个开放源代码的全文检索引擎工具包。提供了完整的查询引擎和索引引擎，部分文本分析引擎。Lucene的目的是为软件开发人员提供一个简单易用的工具包，以方便的在目标系统中实现全文检索的功能。

#### 1.5 全文检索的应用场景

​	对于**数据量大**、**数据结构不固定的数据**可采用全文检索方式搜索，比如百度、Google等**搜索引擎**、**论坛站内搜索**、**电商网站站内搜索**等。



### 2 Lucene实现全文检索的流程

![1549724395120](images/1549724395120.png)

- 绿色表示索引过程，对要搜索的原始内容进行索引构建一个索引库，索引过程包括：

  确定原始内容即要搜索的内容（原始文档）—>采集文档—>创建文档—>分析文档—>索引文档

- 红色表示搜索过程，从索引库中搜索内容，搜索过程包括：

  用户通过搜索界面—>创建查询—>执行搜索，从索引库搜索—>渲染搜索结果



#### 2.1 创建索引

> 将用户要搜索的文档内容创建索引，索引存储在索引库（index）中

1. **获得原始文档**

   原始文档是指**要索引和搜索的内容**。原始内容包括互联网上的网页（爬虫）、数据库中的数据、磁盘上的文件等

2. **创建文档对象**

   获取原始内容的目的是为了索引，在索引前需要将原始内容创建成文档（**Document**），文档中包括**一个一个的域（Field，如文件名/文件内容/大小等）**，域中存储内容（key-value类型）

   注意：每个Document可以有多个Field，不同的Document可以有不同的Field，同一个Document可以有相同的Field（域名和域值都相同）。**每个文档都有一个唯一的编号，就是文档ID**。

3. **分析文档**

   将原始内容创建为包含域（Field）的文档（Document），需要再对域中的内容进行分析，分析的过程是经过对原始文档**提取单词**、将**字母转为小写**、**去除标点符号**、**去除停用词等**过程生成最终的语汇单元，可以将语汇单元理解为一个一个的单词。

   **每个单词叫做一个Term**，不同的域中拆分出来的相同的单词是不同的Term。**Term中**包含两部分一部分是**文档的域名**，另一部分是**单词的内容**。

4. **创建索引**

   对所有文档分析得出的语汇单元进行索引，索引的目的是为了搜索，最终要实现**只搜索被索引的语汇单元**从而**找到Document（文档）**

   注意：创建索引是对语汇单元索引，==**通过词语（内容）找文档**==，这种索引的结构叫==**倒排索引结构**==。

   传统方法是根据**文件找到该文件的内容**，在文件内容中匹配搜索关键字，这种方法是顺序扫描方法，数据量大、搜索慢。

   **倒排索引结构也叫反向索引结构，包括索引和文档两部分，索引即词汇表，它的规模较小，而文档集合较大。**

   ![1549726346135](images/1549726346135.png)

#### 2.2 查询索引

> 查询索引也是搜索的过程。搜索就是用户输入关键字，从索引（index）中进行搜索的过程。根据关键字搜索索引，根据索引找到对应的文档，从而找到要搜索的内容（此处案例指磁盘上的文件）

1. **用户查询接口**

   全文检索系统提供用户搜索的界面供用户提交**搜索的关键字**，搜索完成展示搜索结果。

   > Lucene不提供制作用户搜索界面的功能，需要根据自己的需求开发搜索界面

2. **创建查询**

   用户输入查询关键字执行搜索之前需要先**构建一个查询对象**，查询对象中可以指定查询要搜索的**Field**文档域、**查询关键字**等，查询对象会生成具体的查询语法，例如：语法 “fileName:lucene”表示要搜索Field域的文件名为“lucene”的文档

3. **执行查询**

   搜索索引过程：根据查询语法**在倒排索引词典表中分别找出对应搜索词的索引**，从而找到索引所链接的**文档链表**。

   比如搜索语法为“fileName:lucene”表示搜索出fileName域中包含Lucene的文档。搜索过程就是**在索引上查找域为fileName，并且关键字为Lucene的Term，并根据Term找到文档id列表**。

4. **渲染结果**

   以一个友好的界面将查询结果展示给用户，用户根据搜索结果找自己想要的信息，为了帮助用户很快找到自己的结果，提供了很多展示的效果，比如搜索结果中将关键字高亮显示，百度提供的快照等。



### 3 Demo

> 实现一个文件的搜索功能，通过关键字搜索文件，凡是文件名或文件内容包括关键字的文件都需要找出来。还可以根据中文词语进行查询，并且需要支持多个条件查询。

1. Lucene是开发全文检索功能的工具包，从[官方网站](http://lucene.apache.org/)下载如`lucene-7.4.0`（要求JDK1.8以上），并解压

   其中core为核心包；analysis为分析包；queryparse为查询分析器

   需要使用的有：`lucene-core-7.4.0.jar` `lucene-analyzers-common-7.4.0.jar`，还需要`common-io`来操作文件

2. **创建索引**

   1. 创建一个java工程，并导入jar包

   2. 创建`Directory`对象，指定索引库的存放位置

   3. 创建一个`IndexWriterConfig`对象

   4. 创建一个`IndexWriter`对象。

   5. 创建`Document`对象

   6. 创建`Field`对象，将field**添加**到Document对象中

   7. 使用`IndexWriter`对象将`Document`对象写入索引库，此过程进行索引创建。并将索引和`Document`对象写入索引库。

   8. 关闭`IndexWriter`对象。

      ```java
      @Test
      public void createIndex() throws IOException {
          //1.指定索引库存放的路径，存入磁盘
          Directory directory = FSDirectory.open(new File("C:\\Develop\\lucene-7.4.0\\index").toPath());
          //索引库还可以存放到内存中
          //Directory directory = new RAMDirectory();
      
          //2.创建IndexWriterConfig对象，若不指定参数则为使用标准分析器
          IndexWriterConfig config = new IndexWriterConfig(new IKAnalyzer());
          //3.创建IndexWriter对象
          IndexWriter indexWriter = new IndexWriter(directory,config);
          //原始文档的路径
          File dir = new File("F:\\00000\\0000000博学谷JavaEE\\5-流行框架\\1 lucene\\02.参考资料\\searchsource");
          File[] files = dir.listFiles();
          for (File file : files) {
              //文件名
              String fileName = file.getName();
              //文件内容
              String fileContent = FileUtils.readFileToString(file,"UTF-8");
              //文件路径
              String filePath = file.getPath();
              //文件的大小
              long fileSize  = FileUtils.sizeOf(file);
              //4.创建文件名域
              //第一个参数：域的名称；第二个参数：域的内容；第三个参数：是否存储
              Field fileNameField = new TextField("filename", fileName, Field.Store.YES);
              //文件内容域
              Field fileContentField = new TextField("content", fileContent, Field.Store.YES);
              //文件路径域（不分析、不索引、只存储）
              Field filePathField = new TextField("path", filePath, Field.Store.YES);
              //文件大小域
              Field fileSizeField = new TextField("size", fileSize + "", Field.Store.YES);
      
              //5.创建document对象
              Document document = new Document();
              document.add(fileNameField);
              document.add(fileContentField);
              document.add(filePathField);
              document.add(fileSizeField);
              //6.创建索引，并写入索引库
              indexWriter.addDocument(document);
          }
          //7.关闭Indexwriter对象
          indexWriter.close();
      }
      ```

3. 可以使用luke-7.4.0（利用JavaFx开发的应用，版本和lucene对应）查看index目录中索引库的信息，软件要求JDK9.0以上

4. **查询索引库**

   1. 创建一个`Directory`对象，也就是索引库存放的位置。

   2. 创建一个`IndexReader`对象，需要指定`Directory`对象。

   3. 创建一个`Indexsearcher`对象，需要指定`IndexReader`对象

   4. 创建一个`Query`对象`TermQuery`，指定查询的**域**和查询的**关键词**。

   5. 执行查询，得到`TopDocs`对象

   6. 返回查询结果。遍历查询结果并输出。

   7. 关闭`IndexReader`对象。

      ```java
      @Test
      public void searchIndex() throws Exception {
          //1.指定索引库存放的路径
          Directory directory = FSDirectory.open(new File("C:\\Develop\\lucene-7.4.0\\index").toPath());
          //2.创建IndexReader对象
          IndexReader indexReader = DirectoryReader.open(directory);
          //3.创建IndexSearcher对象
          IndexSearcher indexSearcher = new IndexSearcher(indexReader);
          //4.创建查询，指定查询的域和查询的关键词
          Query query = new TermQuery(new Term("content", "spring"));
          //执行查询
          //第一个参数是查询对象，第二个参数是查询结果返回的最大值
          TopDocs topDocs = indexSearcher.search(query, 10);
          //查询结果的总条数
          System.out.println("查询结果的总条数："+ topDocs.totalHits);
          //遍历查询结果
          //topDocs.scoreDocs存储了document对象的id
          for (ScoreDoc scoreDoc : topDocs.scoreDocs) {
              //scoreDoc.doc属性就是document对象的id
              //根据document的id找到document对象
              Document document = indexSearcher.doc(scoreDoc.doc);
              System.out.println(document.get("filename"));
              //System.out.println(document.get("content"));
              System.out.println(document.get("path"));
              System.out.println(document.get("size"));
              System.out.println("-------------------------");
          }
          //关闭Indexreader对象
          indexReader.close();
      }
      ```



### 4 分析器

查看分析器的分词效果

1. 创建一个标准分析器对象`Analyzer`

2. 获得`TokenStream`对象

3. 添加一个引用，可以获得每个关键词

4. 将指针调整到列表的头部

5. 遍历关键词列表，通过`incrementToken`方法判断列表是否结束

6. 关闭`TokenStream`

   ```java
   @Test
   public void testTokenStream() throws Exception {
       //创建一个标准分析器对象，对中文分词只能按字分
       //Analyzer analyzer = new StandardAnalyzer();
       
       //创建一个IKAnalyzer分析器对象
       Analyzer analyzer = new IKAnalyzer();
       //获得tokenStream对象
       //第一个参数：Field，可以随便给一个（这里只是测试）；第二个参数：要分析的文本内容
       TokenStream tokenStream = analyzer.tokenStream("test", "The Spring Framework provides a comprehensive programming and configuration model.");
       //添加一个引用，可以获得每个关键词
       CharTermAttribute charTermAttribute = tokenStream.addAttribute(CharTermAttribute.class);
       //添加一个偏移量的引用，记录了关键词的开始位置以及结束位置
       OffsetAttribute offsetAttribute = tokenStream.addAttribute(OffsetAttribute.class);
       //将指针调整到列表的头部
       tokenStream.reset();
       //遍历关键词列表，通过incrementToken方法判断列表是否结束
       while(tokenStream.incrementToken()) {
           //关键词的起始位置
           System.out.println("start->" + offsetAttribute.startOffset());
           //取关键词，直接打印调用toString()方法
           System.out.println(charTermAttribute);
           //结束位置
           System.out.println("end->" + offsetAttribute.endOffset());
       }
       tokenStream.close();
   }
   ```



#### 4.1 中文分析器

- Lucene自带中文分词器

  - StandardAnalyzer：单字分词
  - SmartChineseAnalyzer：对中文支持较好，但扩展性差，扩展词库，禁用词库和同义词库等不好处理

- **IKAnalyzer**

  1. 把`IK-Analyzer-1.0-SNAPSHOT.jar`包添加到工程中

  2. 把配置文件和扩展词典和停用词词典添加到classpath下

     注意：hotword.dic和ext_stopword.dic文件的格式为UTF-8，注意是无BOM 的UTF-8 编码

  3. 使用自定义分析器`IndexWriterConfig config = new IndexWriterConfig(new IKAnalyzer());`



### 5 索引库的维护（Field及增删改）

#### 5.1 Field域的属性

- **是否分析**：是否对域的内容进行**分词**处理。前提是我们要对域的内容进行查询。

- **是否索引**：将Field分析后的词或整个Field值进行索引，只有索引方可搜索到。

  比如：商品名称、商品简介分析后进行索引，订单号、身份证号不用分析但也要索引，这些将来都要作为**查询条件**。

- **是否存储**：将Field值存储在文档中，存储在文档中的Field才可以从Document中获取。

  比如：商品名称、订单号，凡是将来要从Document中获取的Field都要存储。**是否要将内容展示给用户**。

|                           Field类                            |        数据类型        | Analyzed是否分析 | Indexed是否索引 | Stored是否存储 |                             说明                             |
| :----------------------------------------------------------: | :--------------------: | :--------------: | :-------------: | :------------: | :----------------------------------------------------------: |
|              StoredField(FieldName, FieldValue)              | 重载方法，支持多种类型 |        N         |        N        |       Y        | 这个Field用来构建不同类型Field   不分析，不索引，但要Field存储在文档中 |
|            LongPoint(String name, long... point)             |         Long型         |        Y         |        Y        |       N        | 可以使用LongPoint、IntPoint等类型存储数值类型的数据。让数值类型可以进行索引。但是不能存储数据，如果想存储数据还需要使用StoredField。 |
|       StringField(FieldName,   FieldValue,Store.YES))        |         字符串         |        N         |        Y        |      Y或N      | 这个Field用来构建一个字符串Field，但是不会进行分析，会将整个串存储在索引中，比如(订单号,姓名等)   是否存储在文档中用Store.YES或Store.NO决定 |
| TextField(FieldName, FieldValue, Store.NO)或   TextField(FieldName, reader) |    字符串   或   流    |        Y         |        Y        |      Y或N      | 如果是一个Reader, lucene猜测内容比较多,会采用Unstored的策略. |

修改之前的创建索引中方法

```java
//不同的document可以有不同的域，同一个document可以有相同的域。
document.add(new TextField("filename", "新添加的文档", Field.Store.YES));
document.add(new TextField("content", "新添加的文档的内容", Field.Store.NO));
//LongPoint创建索引
document.add(new LongPoint("size", 1000l));
//StoreField存储数据
document.add(new StoredField("size", 1000l));
//不需要创建索引的就使用StoreField存储
document.add(new StoredField("path", "d:/temp/1.txt"));

```



#### 5.2 索引库增删改

```java
public class IndexManager {

    private IndexWriter indexWriter;

    @Before
    public void init() throws Exception {
        //创建一个IndexWriter对象，需要使用IKAnalyzer作为分析器
        indexWriter =
                new IndexWriter(FSDirectory.open(new File("C:\\temp\\index").toPath()),
                        new IndexWriterConfig(new IKAnalyzer()));
    }

    @Test
    public void addDocument() throws Exception {
        //创建一个IndexWriter对象，需要使用IKAnalyzer作为分析器
        IndexWriter indexWriter =
                new IndexWriter(FSDirectory.open(new File("C:\\temp\\index").toPath()),
                new IndexWriterConfig(new IKAnalyzer()));
        //创建一个Document对象
        Document document = new Document();
        //向document对象中添加域
        document.add(new TextField("name", "新添加的文件", Field.Store.YES));
        document.add(new TextField("content", "新添加的文件内容", Field.Store.NO));
        document.add(new StoredField("path", "c:/temp/helo"));
        // 把文档写入索引库
        indexWriter.addDocument(document);
        //关闭索引库
        indexWriter.close();
    }

    //将索引目录的索引信息全部删除，直接彻底删除，无法恢复！！！！！
    @Test
    public void deleteAllDocument() throws Exception {
        //删除全部文档
        indexWriter.deleteAll();
        //关闭索引库
        indexWriter.close();
    }

    @Test
    public void deleteDocumentByQuery() throws Exception {
        indexWriter.deleteDocuments(new Term("name", "apache"));//参数为Query也可以，好像只是再封装了一层
        indexWriter.close();
    }

    //原理是先删除！！！后添加！！！
    @Test
    public void updateDocument() throws Exception {
        //创建一个新的文档对象
        Document document = new Document();
        //向文档对象中添加域
        document.add(new TextField("name", "更新之后的文档", Field.Store.YES));
        document.add(new TextField("name1", "更新之后的文档2", Field.Store.YES));
        document.add(new TextField("name2", "更新之后的文档3", Field.Store.YES));
        //更新操作
        indexWriter.updateDocument(new Term("name", "spring"), document);
        //关闭索引库
        indexWriter.close();
    }
}

```



#### 5.3 索引库查询

对要搜索的信息创建Query查询对象，Lucene会根据Query查询对象生成最终的查询语法，类似关系数据库Sql语法一样Lucene也有自己的查询语法，比如：“name:lucene”表示查询Field的name为“lucene”的文档信息。可通过两种方法创建查询对象：

- 使用Lucene提供Query子类

  - **TermQuery**，根据**域**和**关键词**进行查询，**TermQuery不使用分析器所以建议匹配不分词的Field域查询**，比如订单号
  - LongPoint.**newRangeQuery**，**数值范围查询**

- 使用**QueryParse**解析查询表达式（**先分词**，再查询，需要指定**默认查找Field**和**分析器**）

  - 通过QueryParser也可以创建Query，QueryParser提供一个Parse方法，此方法可以直接根据查询语法来查询。Query对象执行的查询语法可通过`System.out.println(query);`查询。

    需要使用到分析器。建议创建索引时使用的分析器和查询索引时使用的分析器要一致。

    需要加入queryParser依赖的`lucene-queryparser-7.4.0.jar`包

```java
public class SearchIndex {
    private IndexReader indexReader;
    private IndexSearcher indexSearcher;
    @Before
    public void init() throws Exception {
        indexReader = DirectoryReader.open(FSDirectory.open(new File("C:\\temp\\index").toPath()));
        indexSearcher = new IndexSearcher(indexReader);
    }

    @Test
    public void testTermQuery() throws Exception {
        //创建一个Query对象
        Query query = new TermQuery(new Term("content", "lucene"));
        printResult(query);
    }
    
    
    //数值范围查询
    @Test
    public void testRangeQuery() throws Exception {
        //创建一个Query对象
        Query query = LongPoint.newRangeQuery("size", 0L, 100L);
        printResult(query);
    }

    @Test
    public void testQueryParser() throws Exception {
        //创建一个QueryPaser对象，两个参数
        //参数1：默认搜索域，参数2：分析器对象
        QueryParser queryParser = new QueryParser("name", new IKAnalyzer());
        //使用QueryPaser对象创建一个Query对象
        Query query = queryParser.parse("lucene是一个Java开发的全文检索工具包");
        //执行查询
        printResult(query);
    }


//========================================================================================
    private void printResult(Query query) throws Exception {
        //执行查询
        TopDocs topDocs = indexSearcher.search(query, 10);
        System.out.println("总记录数：" + topDocs.totalHits);
        ScoreDoc[] scoreDocs = topDocs.scoreDocs;
        for (ScoreDoc doc:scoreDocs){
            //取文档id
            int docId = doc.doc;
            //根据id取文档对象
            Document document = indexSearcher.doc(docId);
            System.out.println(document.get("name"));
            System.out.println(document.get("path"));
            System.out.println(document.get("size"));
            //System.out.println(document.get("content"));
            System.out.println("-----------------寂寞的分割线");
        }
        indexReader.close();
    }
}
```













## 4 Spring Data Solr

​	大多数搜索引擎应用都必须具有某种搜索功能，问题是搜索功能往往是巨大的资源消耗，并且它们由于沉重的数据库加载而拖垮你的应用的性能。这就是为什么转移负载到一个**外部的搜索服务器**是一个不错的主意，**Apache Solr**是一个流行的开源搜索服务器，它通过使用类似**REST的HTTP API**，这就确保你能从几乎==**任何编程语言来使用solr**==。

​	Solr是一个**开源**搜索平台，用于**构建搜索应用程序**。 它建立在[Lucene](http://www.yiibai.com/lucene/)(全文搜索引擎)之上。 Solr是企业级的、快速的和高度可扩展的。 使用Solr构建的应用程序非常复杂，可提供高性能。

​	为了在CNET网络的公司网站上添加搜索功能，Yonik Seely于2004年创建了Solr。并在2006年1月，它成为Apache软件基金会下的一个开源项目。并于2016年发布最新版本Solr 6.0，支持并行SQL查询的执行。

​	**Solr可以和[Hadoop](http://www.yiibai.com/hadoop/)一起使用**。由于Hadoop处理大量数据，Solr帮助我们从这么大的源中找到所需的信息。不仅限于搜索，**Solr也可以用于存储**目的。像其他**NoSQL**数据库一样，它是一种非关系数据存储和处理技术。

​	总之，Solr是一个**可扩展**的，**可部署**，**搜索/存储引擎**，优化搜索**大量以文本为中心的数据**。



### 1 Solr 环境搭建

#### 1.1 Solr 安装

1. 安装 Tomcat（解压即可）

2. 解压 solr

3. 把 solr 下的dist目录`solr-4.10.3.war`部署到 `Tomcat\webapps`下(去掉版本号)

4. 启动 Tomcat，自动解压缩 war 包

5. 把solr下example/lib/ext 目录下的所有的 jar 包，添加到 solr 的工程中(\WEB-INF\lib目录下)

6. solrhome，solr 下的/example/solr 目录就是一个 solrhome。复制此目录到如D盘并改名为solrhome（存储数据）

7. 关联 solr 及 solrhome。需要修改 solr 工程的 web.xml 文件

   ```xml
   <env-entry>
       <env-entry-name>solr/home</env-entry-name>
       <env-entry-value>d:\solrhome</env-entry-value>
       <env-entry-type>java.lang.String</env-entry-type>
   </env-entry>
   ```

8. 重新启动该Tomcat，访问`http://localhost:8080/solr/`即可看到界面，可以查询、分析



#### 1.2 中文分析器 IK Analyzer

> IK Analyzer 是一个开源的，基亍 java 语言开发的轻量级的中文分词工具包。从 2006年 12 月推出 1.0 版开始，IKAnalyzer 已经推出了 4 个大版本。最初，它是以开源项目Luence 为应用主体的，结合词典分词和文法分析算法的中文分词组件。3.0 版本开始，IK 发展为面向 Java 的公用分词组件，独立于 Lucene 项目，同时提供了对 Lucene 的默认优化实现。在 2012 版本中，IK 实现了简单的分词歧义排除算法，标志着 IK 分词器从单纯的词典分词向模拟语义分词衍化

IK Analyzer配置：

1. 把IKAnalyzer2012FF_u1.jar 添加到 solr 工程的 lib 目录下

2. 创建WEB-INF/classes文件夹，把扩展词典、停用词词典（辅助词或敏感词）、配置文件放到 solr 工程的 WEB-INF/classes 目录下

3. 修改 Solrhome 的 schema.xml 文件，配置一个 FieldType，使用 IKAnalyzer来做中文分析器

   ```xml
   <fieldType name="text_ik" class="solr.TextField">
       <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
   </fieldType>
   ```



#### 1.3 配置域

域相当于数据库的表字段，用户存放数据，因此用户根据业务需要去定义相关的Field（域），一般来说，每一种对应着一种数据，用户对同一种数据进行相同的操作。域的常用属性：

- name：指定域的名称
- type：指定域的类型
- indexed：是否索引（要搜索的域）
- stored：是否存储
- required：是否必须
- multiValued：是否多值

##### 1.3.1 基本域

修改solrhome的`schema.xml`文件，设置业务系统 Field。根据搜索的、被搜索出的、要用到的Field来配置

```xml
<field name="item_goodsid" type="long" indexed="true" stored="true"/> <!--goodsId,商品SKU。这些都是item表数据-->
<field name="item_title" type="text_ik" indexed="true" stored="true"/><!--title,商品标题-->
<field name="item_price" type="double" indexed="true" stored="true"/><!--price,商品价格-->
<field name="item_image" type="string" indexed="false" stored="true" /><!--image,商品图片地址-->
<field name="item_category" type="string" indexed="true" stored="true" /><!--category,商品分类-->
<field name="item_seller" type="text_ik" indexed="true" stored="true" /><!--seller,商家-->
<field name="item_brand" type="string" indexed="true" stored="true" /><!--brand,品牌-->
<!--后添加的-->
<field name="item_updatetime" type="date" indexed="true" stored="true" /><!--updatetime,上架时间-->
```

##### 1.3.2 复制域

多条件查询时（同时输入标题、分类、商家、品牌），复制域的作用在于将某一个Field中的数据复制到另一个域中，不需要存储

```xml
<field name="item_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
<copyField source="item_title" dest="item_keywords"/>
<copyField source="item_category" dest="item_keywords"/>
<copyField source="item_seller" dest="item_keywords"/>
<copyField source="item_brand" dest="item_keywords"/>
```

##### 1.3.3 动态域

当我们需要动态扩充字段时，我们需要使用动态域。对于品优购，规格的值是不确定的，所以我们需要使用动态域来实现。需要实现的效果如下：

![1548005515777](images/1548005515777.png)

```xml
<dynamicField name="item_spec_*" type="string" indexed="true" stored="true" />	


```





### 2 Spring Data Solr 入门

Spring Data Solr就是为了方便Solr的开发所研制的一个框架，其底层是对SolrJ（官方API）的封装。

> 官方类库 SolrJ：原理是**RESTful的HTTP请求和响应**
>
> Spring Data Solr：是对SolrJ的封装
>
> 也可不使用上述工具，手动请求Solr，手动处理响应，httpClient

#### 2.1 环境搭建

- Maven依赖

  ```xml
  <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-solr</artifactId>
      <version>1.5.5.RELEASE</version>
  </dependency>
  <!--Demo中还会用到JUnit和Spring与JUnit整合-->
  
  
  ```

- resources下创建`applicationContext-solr.xml`

  ```xml
  <!-- solr服务器地址，其实就是配置了一个bean -->
  <solr:solr-server id="solrServer" url="http://127.0.0.1:8080/solr" />
  
  <!-- solr模板，使用solr模板可对索引库进行CRUD的操作 -->
  <bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
      <constructor-arg ref="solrServer" />
  </bean>
  
  
  ```

#### 2.2 `@Field` 、`@Dynamic`注解

> `@Field`是Solr官方的SolrJ提供的注解，`@Dynamic`是SpringDataSolr的注解

创建 cn.itcast.pojo 包，将品优购的TbItem实体类拷入本工程，属性（即Field域）使用`@Field`注解标识。如果属性与配置文件定义的域名称不一致，需要在注解中指定域名称。（上面Solr中基本域配置和这个对应，复制域不需要，动态域后面讲）

```java
public class TbItem implements Serializable{
    
    @Dynamic
    @Field("item_spec_*")
    private Map<String,String> specMap;//动态域，在数据导入时需利用fastJSON转换JSON为map

    @Field
    private Long id;

    @Field("item_title")
    private String title;

    @Field("item_price")
    private BigDecimal price;

    @Field("item_image")
    private String image;

    @Field("item_goodsid")//注意大小写对应solr中配置的基本域
    private Long goodsId;

    @Field("item_category")
    private String category;

    @Field("item_brand")
    private String brand;

    @Field("item_seller")
    private String seller;
    
    @Field("item_updatetime")
    private Date updateTime;//后添加的，排序需要
    .......
}


```

#### 2.3 增加（修改）

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-solr.xml")//可以省略locations
public class TestTemplate {
    @Autowired
    private SolrTemplate solrTemplate;

    @Test
    public void testAdd(){

        TbItem tbItem = new TbItem();
        tbItem.setId(1L);
        tbItem.setTitle("华为Mate10");
        tbItem.setCategory("手机");
        tbItem.setBrand("华为");
        tbItem.setSeller("华为旗舰店");
        tbItem.setGoodsId(10L);
        tbItem.setPrice(new BigDecimal(4500.00));
        solrTemplate.saveBean(tbItem);//增加；修改时只需对象的id（主键，非空）不变，其他修改后save即可
        //solrTemplate.saveBeans(collection);（可以传递集合）
        solrTemplate.commit();//必须commit
    }
}


```

#### 2.4 按主键删除

```java
@Test
public void deleteById(){
    solrTemplate.deleteById("1");//自动转换
    solrTemplate.commit();
}


```

#### 2.5 删除所有（条件）

```java
@Test
public void deleteAll(){
    Query query = new SimpleQuery("*:*");
    solrTemplate.delete(query);
    solrTemplate.commit();
}


```

#### 2.5 按主键查询

```java
@Test
public void getById(){
    TbItem item = solrTemplate.getById(1L, TbItem.class);
    System.out.println(item.getTitle());
}

```

#### 2.6 条件查询（含分页查询）

```java
//条件查询
@Test
public void testQuery(){
    Query query = new SimpleQuery("*:*");//查询所有字段所有信息（利用封装好的方法，不用写表达式）
    Criteria criteria = new Criteria("item_category").contains("手机")
        .and("item_title").contains("2");
    query.addCriteria(criteria);

    ScoredPage<TbItem> items = solrTemplate.queryForPage(query, TbItem.class);

    List<TbItem> content = items.getContent();//当前页所有数据
    content.forEach(o-> System.out.println(o.getTitle()+"--"+o.getBrand()+"--"+o.getPrice()));

    System.out.println("总记录数："+items.getTotalElements());
    System.out.println("总页数："+items.getTotalPages());
}


//分页
@Test
public void testQueryPage(){
    Query query = new SimpleQuery("*:*");//查询所有字段所有信息
    query.setOffset(20);//开始索引
    query.setRows(5);//每页记录数
    ScoredPage<TbItem> items = solrTemplate.queryForPage(query, TbItem.class);

    List<TbItem> content = items.getContent();//当前页所有数据
    content.forEach(o-> System.out.println(o.getTitle()+"--"+o.getBrand()+"--"+o.getPrice()));

    System.out.println("总记录数："+items.getTotalElements());
    System.out.println("总页数："+items.getTotalPages());
}


//首先利用saveBeans()方法批量添加数据
@Test
public void testAddList(){
    List<TbItem> list=new ArrayList();

    for(int i=0;i<100;i++){
        TbItem item=new TbItem();
        item.setId(i+1L);
        item.setBrand("华为");
        item.setCategory("手机");
        item.setGoodsId(1L);
        item.setSeller("华为2号专卖店");
        item.setTitle("华为Mate"+i);
        item.setPrice(new BigDecimal(2000+i));
        list.add(item);
    }
    solrTemplate.saveBeans(list);
    solrTemplate.commit();
}
```











### 3 批量数据导入

> 可以使用Solr自带插件来导入，但不灵活，推荐查询增加的方式，根据条件筛选数据并导入Solr中

```java
//审核通过才导入
TbItemExample example = new TbItemExample();
TbItemExample.Criteria criteria = example.createCriteria();
criteria.andStatusEqualTo("1");

List<TbItem> items = itemMapper.selectByExample(example);
for (TbItem item : items) {
    Map specMap = JSON.parseObject(item.getSpec(), Map.class);//fastJSON将数据库中json数据转为Map
    item.setSpecMap(specMap);
}
solrTemplate.saveBeans(items);//saveBeans()
solrTemplate.commit();
```



### 4 关键字搜索（复制域）

`ItemSearchServiceImpl`

```java
@Service(timeout = 5000)//推荐写在服务提供方。当两方同时写，以消费方为准
public class ItemSearchServiceImpl implements ItemSearchService {
    @Autowired
    private SolrTemplate solrTemplate;
    
    //前端传入的数据有多好多类型，如关键字、类型多选框之类的，前端封装为map类似对象
    //后端不仅仅返回列表数据，还要返回根据关键字搜索后的分类信息，选择框等等，所以封装为map
    @Override
    public Map search(Map searchMap) {
        Map map = new HashMap();
        //Query query = new SimpleQuery("*:*");//好像不设置"*:*"也可以
        //Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));//is匹配（利用分词实现）
        //query.addCriteria(criteria);
        Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));//is匹配（利用分词实现）
        Query query = new SimpleQuery(criteria);
        ScoredPage<TbItem> tbItems = solrTemplate.queryForPage(query, TbItem.class);

        map.put("list",tbItems.getContent());
        return map;
    }
}


```

`ItemSearchController`

```java
@RestController
@RequestMapping("/itemSearch")
public class ItemSearchController {
    @Reference(timeout = 5000)//默认为1秒
    private ItemSearchService itemSearchService;

    @RequestMapping("/search")
    public Map search(@RequestBody Map searchMap){ //@RequestBody接收前端json数据
        return itemSearchService.search(searchMap);
    }
}

```

`ItemSearchService.js`

```js
app.service("itemSearchService",function ($http) {
    this.search = function (searchMap) {
        return $http.post("/itemsearch/search.do",searchMap);
    }
})

```

`ItemSearchController.js`

```js
app.controller("itemSearchController",function ($scope,itemSearchService) {
    $scope.search = function () {
        itemSearchService.search($scope.searchMap).success(function (data) {
            $scope.resultMap = data;
        })
    }
})

```

HTML遍历展示即可





### 5 高亮显示关键字搜索（复制域）

> 高亮显示标题title中的字符

需要修改关键字搜索中的service方法，其他不变

```java
@Override
public Map search(Map searchMap) {
    Map<String,Map> map = new HashMap<>();
    map.putAll(searchList(searchMap));//1.查询列表数据
    map.put("categoryList",searchCategoryList(searchMap));//2.分组查询商品分类列表


    return map;
}

/**
 * 列表查询
 */
private Map searchList(Map searchMap) {
    Map map = new HashMap();
    //关键字的条件查询（利用复制域），相当于where
    Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));
    HighlightQuery query = new SimpleHighlightQuery(criteria);
	//高亮选项
    HighlightOptions highlightOptions = new HighlightOptions()
        .addField("item_title")//高亮Filed域
        .setSimplePrefix("<span style='color: red;'>")//HTML前缀
        .setSimplePostfix("</span>");//HTML后缀
    query.setHighlightOptions(highlightOptions);//设置高亮设置
    HighlightPage<TbItem> items = solrTemplate.queryForHighlightPage(query, TbItem.class);

    //设置高亮
    List<HighlightEntry<TbItem>> highlighted = items.getHighlighted();//高亮entry集合（每条记录）
    for (HighlightEntry<TbItem> entry : highlighted) {
        //高亮列表（高亮Filed域个数可能多个）。并且每个Filed域可能存储多值（此处没有）
        TbItem item = entry.getEntity();//获取源实体类
        List<HighlightEntry.Highlight> highlights = entry.getHighlights();
        if (highlights.size() >= 0 && highlights.get(0).getSnipplets().size() >= 0) {
            item.setTitle(highlights.get(0).getSnipplets().get(0));
        }
    }
    map.put("list", items.getContent());
    return map;
}
```

但是此时HTML中显示的是HTML代码，不是解析后的页面。这是AngularJS为了防止html攻击采取的安全机制。此时需要用到AngularJS的`$sce`服务中的`trustAsHtml`方法。由于该方法具有通用性，定义在`filter`过滤器中（放入base.js）

```js
app.filter("trustHtml",['$sce',function ($sce) {
    return function (data) {
        return $sce.trustAsHtml(data);
    }
}])
```

HTML中就不能使用`{{}}`来绑定了，需要使用`ng-bing-html`

```html
<div class="attr" ng-bind-html="item.title | trustHtml"></div>
```



### 6 分组查询

service中添加私有方法并调用

```java
/**
 * 分组查询商品分类列表
 */
private List searchCategoryList(Map searchMap){
    List<String> list = new ArrayList<>();
    //关键字的条件查询（利用复制域），相当于where
    Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));
    Query query = new SimpleQuery(criteria);
    //设置分组选项，相当于group by。可能有多个分组（继续addGroupByField即可）
    GroupOptions groupOptions = new GroupOptions().addGroupByField("item_category");
    query.setGroupOptions(groupOptions);
    //分组页
    GroupPage<TbItem> items = solrTemplate.queryForGroupPage(query, TbItem.class);
    //分组结果
    GroupResult<TbItem> item_category = items.getGroupResult("item_category");
    //分组entry页
    Page<GroupEntry<TbItem>> groupEntries = item_category.getGroupEntries();
    //分组entryj集合，便利获取值
    List<GroupEntry<TbItem>> content = groupEntries.getContent();
    for (GroupEntry<TbItem> tbItemGroupEntry : content) {
        list.add(tbItemGroupEntry.getGroupValue());
    }
    return list;
}
```

HTML中遍历

```html
<div ng-if="resultMap.categoryList!=null" >
    <a ng-repeat="category in resultMap.categoryList" href="#{{$index}}">{{category}} </a>
</div>
```



### 7 过滤查询（以上所有代码）

```java
@Service(timeout = 5000)//推荐写在服务提供方。所两方同时写，以消费方为准
public class ItemSearchServiceImpl implements ItemSearchService {

    @Autowired
    private SolrTemplate solrTemplate;

    /**
     * 搜索
     */
    @Override
    public Map search(Map searchMap) {
        //去掉搜索关键字中的空格
        String  keywords = (String) searchMap.get("keywords");
        searchMap.put("keywords",keywords.replace(" ",""));
        
        Map map = new HashMap<>();
        //1.查询列表数据
        map.putAll(searchList(searchMap));
        //2.分组查询商品分类列表
        List<String> categoryList = searchCategoryList(searchMap);
        map.put("categoryList", categoryList);
        //3.根据分类名称查询品牌和规格列表
        String category = (String) searchMap.get("category");
        if ("".equals(category)) { //若没选择分类，按第一个分类查询
            if (categoryList.size() > 0) {
                map.putAll(searchBrandAndSpecList(categoryList.get(0)));
            }
        } else { //若选择了分类，按选择的分类名称查询
            map.putAll(searchBrandAndSpecList(category));
        }
        return map;
    }

    /**
     * 列表查询（过滤查询等等）
     */
    private Map searchList(Map searchMap) {
        Map map = new HashMap();

        //1.1关键字查询（利用复制域）
        Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));
        HighlightQuery query = new SimpleHighlightQuery(criteria);
        //高亮选项设置
        HighlightOptions highlightOptions = new HighlightOptions()
            .addField("item_title")//高亮Filed域
            .setSimplePrefix("<span style='color: red;'>")//HTML前缀
            .setSimplePostfix("</span>");//HTML后缀
        query.setHighlightOptions(highlightOptions);//设置高亮设置

        //1.2按照商品分类过滤
        if (!"".equals(searchMap.get("category"))) { //若用户选择了分类
            Criteria filterCriteria = new Criteria("item_category").is(searchMap.get("category"));
            FilterQuery filterQuery = new SimpleFilterQuery(filterCriteria);
            query.addFilterQuery(filterQuery);
        }

        //1.3按照品牌过滤
        if (!"".equals(searchMap.get("brand"))) { //若用户选择了品牌
            Criteria filterCriteria = new Criteria("item_brand").is(searchMap.get("brand"));
            FilterQuery filterQuery = new SimpleFilterQuery(filterCriteria);
            query.addFilterQuery(filterQuery);
        }

        //1.4按照规格过滤
        if (searchMap.get("spec") != null) { //若用户选择了规格

            Map<String, String> specMap = (Map<String, String>) searchMap.get("spec");
            Set<Map.Entry<String, String>> entries = specMap.entrySet();
            for (Map.Entry<String, String> entry : entries) {
                System.out.println(entry.getKey() + "====" + entry.getValue());
                Criteria filterCriteria = new Criteria("item_spec_" + entry.getKey()).is(entry.getValue());
                FilterQuery filterQuery = new SimpleFilterQuery(filterCriteria);
                query.addFilterQuery(filterQuery);
            }
        }
        
        //1.5按照价格过滤
        if (!"".equals(searchMap.get("price"))){
            String price = (String) searchMap.get("price");
            String[] split = price.split("-");

            Criteria filterCriteria = new Criteria("item_price");
            if (!split[0].equals("0")){ //起始条件
                filterCriteria.greaterThanEqual(split[0]);//大于等于
            }
            if (!split[1].equals("*")){ //终止条件
                filterCriteria.lessThanEqual(split[1]); //小于等于
            }
            FilterQuery filterQuery = new SimpleFilterQuery(filterCriteria);
            query.addFilterQuery(filterQuery);
        }
        
        //1.6分页
        Integer pageNum = (Integer) searchMap.get("pageNum");//当前页
        if (pageNum==null){
            pageNum = 1;
        }
        Integer pageSize = (Integer) searchMap.get("pageSize");//每页记录数
        if (pageSize==null){
            pageSize = 20;
        }
        query.setOffset((pageNum-1)*pageSize);//起始索引（不是页码）
        query.setRows(pageSize);//每页记录数
        //在最后返回getContent()后还需传递pages、total；

        //1.7排序（根据传入的Field字段）
        String  sortValue = (String) searchMap.get("sort");//升序或降序
        String sortField = (String) searchMap.get("sortField");//排序的Field字段
        if (sortValue!=null && !sortValue.equals("")){
            if (sortValue.equals("ASC")){
                Sort sort = new Sort(Sort.Direction.ASC,"item_"+sortField);
                query.addSort(sort);
            } else if (sortValue.equals("DESC")){
                Sort sort = new Sort(Sort.Direction.DESC,"item_"+sortField);
                query.addSort(sort);
            }
        }
        
        //高亮页对象
        HighlightPage<TbItem> items = solrTemplate.queryForHighlightPage(query, TbItem.class);

        //遍历并设置高亮
        List<HighlightEntry<TbItem>> highlighted = items.getHighlighted();//高亮entry集合（每条记录）
        for (HighlightEntry<TbItem> entry : highlighted) {
            //高亮列表（高亮Filed域个数可能多个）。并且每个Filed域可能存储多值（此处没有）
            TbItem item = entry.getEntity();//获取源实体类
            List<HighlightEntry.Highlight> highlights = entry.getHighlights();
            if (highlights.size() > 0 && highlights.get(0).getSnipplets().size() > 0) {
                item.setTitle(highlights.get(0).getSnipplets().get(0));
            }
        }

        map.put("list", items.getContent());//当前页所有数据
        map.put("pages",items.getTotalPages());//总页数
        map.put("total",items.getTotalElements());//总记录数
        return map;
    }

    /**
     * 分组查询商品分类列表
     */
    private List searchCategoryList(Map searchMap) {
        List<String> list = new ArrayList<>();
        //关键字的条件查询（利用复制域），相当于where
        Criteria criteria = new Criteria("item_keywords").is(searchMap.get("keywords"));
        Query query = new SimpleQuery(criteria);
        //设置分组选项，相当于group by。可能有多个分组（继续addGroupByField即可）
        GroupOptions groupOptions = new GroupOptions().addGroupByField("item_category");
        query.setGroupOptions(groupOptions);
        //分组页
        GroupPage<TbItem> items = solrTemplate.queryForGroupPage(query, TbItem.class);
        //分组结果
        GroupResult<TbItem> item_category = items.getGroupResult("item_category");
        //分组entry页
        Page<GroupEntry<TbItem>> groupEntries = item_category.getGroupEntries();
        //分组entryj集合，便利获取值
        List<GroupEntry<TbItem>> content = groupEntries.getContent();
        for (GroupEntry<TbItem> tbItemGroupEntry : content) {
            list.add(tbItemGroupEntry.getGroupValue());
        }
        return list;
    }

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * Redis查询品牌和规格列表
     */
    private Map searchBrandAndSpecList(String categoryName) {
        Map map = new HashMap();
        //1.根据商品分类名称得到模板id
        Long categoryId = (Long) redisTemplate.boundHashOps("itemCat").get(categoryName);
        if (categoryId != null) {
            //2.根据模板id获取品牌列表
            List brandList = (List) redisTemplate.boundHashOps("brandList").get(categoryId);
            map.put("brandList", brandList);
            //3.根据模板id获取规格列表
            List specdList = (List) redisTemplate.boundHashOps("specList").get(categoryId);
            map.put("specList", specdList);
        }
        return map;
    }
    
    //审核通过时导入SKU数据
    @Override
    public void importList(List list){
        solrTemplate.saveBeans(list);
        solrTemplate.commit();
    }

    //删除商品时同时删除solr中数据
    @Override
    public void deleteByGoodsIds(List goodsIds) {
        Criteria criteria = new Criteria("item_goodsid").in(goodsIds);
        Query query = new SimpleQuery(criteria);
        solrTemplate.delete(query);
        solrTemplate.commit();
    }
}
```



### 8 更新索引库

> 实时更新solr库，而不总是批量导入。还需实现删除方法。查看7中代码。理解即可





# 第七部分 整合SSM

整合的思路：

1. 先搭建整合的**环境** 
2. 先把**Spring**的配置搭建完成 
3. 再使用**Spring整合SpringMVC**框架 
4. 最后使用**Spring整合MyBatis**框架 

## 1 xml+注解方式

### 1.1 创建数据库和表结构

```sql
create database ssm; 
use ssm; 
create table account(    
    id int primary key auto_increment,    
    name varchar(20),    
    money double
);
```

### 1.2 创建maven_parent父工程

会使用到工程的聚合和拆分的概念，这个技术在maven高级会讲

打包方式选择**pom**（有子模块默认使用pom方式打包），只需要`pom.xml`文件，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    
    <!--统一管理jar包版本-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>

        <spring.version>5.1.1.RELEASE</spring.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <shiro.version>1.2.3</shiro.version>
        <mysql.version>8.0.12</mysql.version>
        <mybatis.version>3.4.6</mybatis.version>
        <spring.security.version>5.1.1.RELEASE</spring.security.version>
    </properties>

    <!-- 锁定jar包版本 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${mybatis.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <!--项目依赖jar包-->
    <dependencies>
        <!-- spring -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.8</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!--SpringMVC核心-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
	   
        <!--使用Spring的JdbcTemplate。使用mybatis时可以不导入-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--JUnit测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope><!--手动测试时需要修改scope-->
        </dependency>
	    <!--mysql连接驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
	    <!--servlet包-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
		<!--jsp-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
        </dependency>
		<!--jstl-->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!-- log start -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!-- log end -->
        
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
	    <!--mybatis和spring整合包-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.2</version>
        </dependency>
        <!--Druid连接池，也可以用Spring自带的-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.9</version>
        </dependency>
        <!--c3p0连接池，也可以用Spring自带的-->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
            <type>jar</type>
            <scope>compile</scope>
        </dependency>
        
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.2</version>
        </dependency>
        <!--springsecurity-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>${spring.security.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>${spring.security.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
            <version>${spring.security.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-taglibs</artifactId>
            <version>${spring.security.version}</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
             <!-- 设置编译版本为 1.8 -->     
            <plugin>       
                <groupId>org.apache.maven.plugins</groupId>       
                <artifactId>maven-compiler-plugin</artifactId>       
                <version>3.1</version >       
                <configuration>         
                    <source>1.8</source>         
                    <target>1.8</target>         
                    <encoding>UTF-8</encoding>       
                </configuration>     
            </plugin>  
            <!--添加tomcat7插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
            </plugin>
        </plugins>
    </build>
    
    <!--下面为IDEA自动生成的maven插件，可以删除掉-->
    <build> 
        <finalName>ssm</finalName>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.0.0</version>
                </plugin>
                ...还有好多
            </plugins>
        </pluginManagement>
    </build>
</project>
```

### 1.3 创建maven_dao子模块

打包方式为**jar**包（默认为jar方式打包），domain也可以从中脱离出来。`applicationContext-dao.xml`如下：

整合MyBatis思路：

1. 数据库的连接以及数据库连接池交给spring容器来完成
2. SqlSessionFactory对象应该放到spring容器中作为单例存在
3. 代理模式开发，应该从spring容器中直接获得mapper的动态代理对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
			    http://www.springframework.org/schema/beans/spring-beans.xsd
			    http://www.springframework.org/schema/context
			    http://www.springframework.org/schema/context/spring-context.xsd">
    <!--组件扫描配置。不扫描由SpringMVC控制的Controller注解-->
    <context:component-scan base-package="com.itheima.dao"/>
    <!--也可以用如下中配置方法
    <context:component-scan base-package="com.itheima">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan> -->
    
    <!-- 加载配置文件 -->
    <context:property-placeholder location="classpath:jdbcConfig.properties"/>

    <!--配置Druid DataSource连接池，交给IoC管理-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    
    <!-- 也可以配置c3p0连接池，交给IoC管理。 
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean> -->

    <!--配置SqlSession工厂，交给IoC管理-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>  <!--注入DataSource-->
        <property name="plugins"> <!--PageInterceptor配置-->
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <value>
                            helperDialect=mysql
                            reasonable=true
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>

    <!--扫描Mapper代理模式接口所在包-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>
</beans>
```

```properties
#JdbcConfig.properties
jdbc.driver=com.mysql.cj.jdbc.Driver 
jdbc.url=jdbc:mysql://localhost:3306/bxgtest?serverTimezone=GMT%2B8&useSSL=false
jdbc.username=root
jdbc.password=w1111
```





### 1.4 创建maven_service子模块

打包方式是**jar**包（默认为jar方式） ，`applicationContext-service.xml`如下：

整合思路：除了业务，**事务**也应该交由Spring的声明式事务管理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!--组件扫描配置。不扫描由SpringMVC控制的Controller注解。-->
    <context:component-scan base-package="com.itheima.service"/>

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    
    <!--也可以开启对事务的注解支持-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    
    <!--以下为xml配置，暂不推荐-->
    <!--配置事务通知-->
    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/> <!--可以不配置propagation，默认为REQUIRED-->
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置切面（切点和通知的关系）-->
    <aop:config>
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
    </aop:config>
</beans>
```



### 1.5 创建maven_web子模块

打包方式是**war**包

思路：

* `web.xml`配置：在项目服务器启动时创建**`DispatcherServlet`前端控制器**的并加载`springmvc.xml`配置的容器。
* `web.xml`配置：**`ContextLoaderListener`监听器**，在项目服务器启动时加载所有`applicationContext*.xml`的容器。
  * `classpath`和`classpath*`都是加载类路径下的资源和依赖的jar包中的资源（先后顺序）
    * `classpath`只会返回第一个匹配的资源（有误！！！）
    * `classpath*`会返回路径下匹配的所有资源，可以使用通配符
* `web.xml`配置：**`CharacterEncodingFilter`编码过滤器**，设置Request、Response编码
* `springmvc.xml`中配置处理器映射器、处理器适配器、视图解析器、释放静态资源

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                             http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">

 
    <!-- 配置ContextLoaderListener监听器，加载所有applicationContext.xml并创建spring容器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 配置参数加载类路径的配置文件，不配置默认扫描WEB-INF下的applicationContext -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:applicationContext*.xml</param-value>
    </context-param>


    <!-- 配置SpringMVC的前端控制器：服务器启动后创建，并加载springmvc.xml配置文件创建springmvc容器 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置初始化参数，用于读取 springmvc 的配置文件 --> 
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 配置 servlet 的对象的创建时间点：应用加载时创建。取值只能是非 0 正整数，表示启动顺序 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern> 
    </servlet-mapping>
    
    
     <!--配置编码过滤器-->    
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 设置过滤器中的属性值 --> 
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param> 
        <!-- 是否强制Request、Response编码 -->   
        <init-param> 
            <param-name>forceEncoding</param-name>    
            <param-value>true</param-value> 
        </init-param> 
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern> <!-- 过滤所有请求 --> 
    </filter-mapping>
    
    <!--RESTful开发时，处理DELETE、PUT等请求。前端提交信息需要_method-->
    <filter>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>hiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!--释放静态资源方式1：配置DefaultServlet的静态资源url，直接过滤静态资源 
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/css/*</url-pattern>
        <url-pattern>/img/*</url-pattern>
        <url-pattern>/plugins/*</url-pattern>
        <url-pattern>/templates/*</url-pattern>
    </servlet-mapping>  -->
</web-app>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
			    http://www.springframework.org/schema/beans/spring-beans.xsd
			    http://www.springframework.org/schema/context
			    http://www.springframework.org/schema/context/spring-context.xsd
			    http://www.springframework.org/schema/mvc
			    http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--组件扫描，只扫描controller包下的类。也可以用applicationContext中配置方法-->
    <context:component-scan base-package="com.itheima.controller"/>

    <!--开启对SpringMVC注解的支持，替代处理器映射器、处理器适配器-->
    <mvc:annotation-driven/>
    
    <bean id="applicationContext" class="org.springframework.context.support.ClassPathXmlApplicationContext"/>

    <!--模板解析器-->
    <bean id="templateResolver" class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
        <property name="applicationContext" ref="applicationContext"/>
        <property name="prefix" value="/templates/"/>
        <property name="suffix" value=".html"/>
        <property name="templateMode" value="HTML"/>
        <property name="characterEncoding" value="UTF-8"/><!--解决页面乱码！-->
    </bean>
    <!--模板引擎-->
    <bean id="templateEngine" class="org.thymeleaf.spring5.SpringTemplateEngine">
        <property name="templateResolver" ref="templateResolver"/>
    </bean>
    <!--Thymeleaf视图解析器-->
    <bean class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="templateEngine" ref="templateEngine"/>
        <property name="characterEncoding" value="UTF-8"/><!--解决页面乱码！-->
    </bean>


    <!--JSP视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--（推荐）释放静态资源方式3：将SpringMVC处理不了的请求交给Tomcat，静态资源就可以访问了-->
    <mvc:default-servlet-handler/>

    <!--释放静态资源方式2： location表示路径，mapping表示文件，**表示该目录下的文件以及子目录的文件 
    <mvc:resources mapping="/static/**" location="/static/"/>   静态资源放入static目录即可只写这一句
    <mvc:resources mapping="/css/**" location="/css/" />
    <mvc:resources mapping="/images/**" location="/images/"/>
    <mvc:resources mapping="/js/**" location="/js/" /> 
    -->
    
    <!--aop注解支持-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
</beans>
```

日志输出

```properties
#log4j.properties
# Configure logging for testing: optionally with log file
log4j.rootLogger=WARN, stdout
# log4j.rootLogger=WARN, stdout, logfile

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```





### 1.6 各子模块间依赖

web依赖于service；service依赖于dao（如下）

```xml
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>maven_dao</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### 1.7 运行项目

方法一：

- 在maven_web子模块的 pom.xml 中配置 tomcat 插件运行

  运行 ssm_web 工程它会**从本地仓库下载依赖的 jar 包**（需要将maven_web所依赖的所有模块发布至本地仓库，直接install父工程也行），所以当 ssm_web 依赖的 jar 包内容修改了必须及时发布到本地仓库，比如：ssm_web 依赖的 ssm_service 修改了，需要及时将ssm_service 发布到本地仓库。 

方法二：

- **在maven_parent父工程的 pom.xml 中配置 tomcat插件运行**，自动聚合并执行（**推荐**）

  如果子工程都在本地，采用方法2则不需要子工程修改就立即发布到本地仓库，父工程会自动聚合并使用最新代码执行。 

  注意：如果子工程和父工程中都配置了tomcat 插件，运行的端口和路径以子工程为准。 

方法三：

- 使用本地Tomcat部署

  ![](images\本地Tomcat部署项目.PNG)

















## 2 SSM综合练习

### 2.1 Oracle数据库及表结构

这只是其中两个表，后续还有其他表

1. Oracle 为每个项目创建单独user，oracle数据表存放在表空间下，每个用户有独立表空间。创建用户并授权：

   ```sql
   -- Create the user 
   create user SSM
     identified by itheima
     default tablespace USERS
     temporary tablespace TEMP
     profile DEFAULT;
   -- Grant/Revoke role privileges 
   grant connect to SSM;
   grant resource to SSM;
   -- Grant/Revoke system privileges 
   grant unlimited tablespace to SSM;
   ```

2. 创建order表

   ```sql
   CREATE TABLE product(  
   	id varchar2(32) default SYS_GUID() PRIMARY KEY,  -- 主键uuid，无实际意义。default SYS_GUID()为默认获取UUID
   	productNum VARCHAR2(50) NOT NULL,  
   	productName VARCHAR2(50),  
   	cityName VARCHAR2(50),  
   	DepartureTime timestamp,  
   	productPrice Number,  
   	productDesc VARCHAR2(500),  
   	productStatus INT,  
   	CONSTRAINT product UNIQUE (id, productNum) 
   )
   ```

3. 插入数据

   ```sql
   insert into ORDERS (ordernum, ordertime, peoplecount, orderdesc, paytype, orderstatus, productid, memberid)
   values (to_timestamp('02-03-2018 12:00:00.000000', 'dd-mm-yyyy hh24:mi:ss.ff'), 2, '没什么', 0, 1, '676C5BD1D35E429A8C2E114939C5685A', 'E61D65F673D54F68B0861025C69773DB');......
   ```

### 2.2 SSM整合

1. 利用Maven的父子工程来创建各模块，父工程的pom.xml文件节选如下：

   ```xml
   <properties>
           <spring.version>5.1.1.RELEASE</spring.version>
           <slf4j.version>1.6.6</slf4j.version>
           <log4j.version>1.2.12</log4j.version>
           <oracle.version>10.2.0.1.0</oracle.version>
           <mybatis.version>3.4.6</mybatis.version>
           <spring.security.version>5.1.1.RELEASE</spring.security.version>
       </properties>
   
       <dependencies>        <!-- spring -->
           <dependency>
               <groupId>org.aspectj</groupId>
               <artifactId>aspectjweaver</artifactId>
               <version>1.6.8</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-aop</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context-support</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-web</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-orm</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-beans</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-test</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>${spring.version}</version>
   
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-tx</artifactId>
               <version>${spring.version}</version>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
   
           <dependency>
               <groupId>javax.servlet</groupId>
               <artifactId>javax.servlet-api</artifactId>
               <version>3.1.0</version>
               <scope>provided</scope>
           </dependency>
           <dependency>
               <groupId>javax.servlet.jsp</groupId>
               <artifactId>jsp-api</artifactId>
               <version>2.0</version>
               <scope>provided</scope>
           </dependency>
           <dependency>
               <groupId>jstl</groupId>
               <artifactId>jstl</artifactId>
               <version>1.2</version>
           </dependency>        <!-- log start -->
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>${log4j.version}</version>
           </dependency>
           <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-api</artifactId>
               <version>${slf4j.version}</version>
           </dependency>
           <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
               <version>${slf4j.version}</version>
           </dependency>        <!-- log end -->
   
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>${mybatis.version}</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>1.3.0</version>
           </dependency>
           <dependency>
               <groupId>com.mchange</groupId>
               <artifactId>c3p0</artifactId>
               <version>0.9.5.2</version>
               <type>jar</type>
               <scope>compile</scope>
           </dependency>
           <dependency>
               <groupId>com.github.pagehelper</groupId>
               <artifactId>pagehelper</artifactId>
               <version>5.1.2</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.security</groupId>
               <artifactId>spring-security-web</artifactId>
               <version>${spring.security.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.security</groupId>
               <artifactId>spring-security-config</artifactId>
               <version>${spring.security.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.security</groupId>
               <artifactId>spring-security-core</artifactId>
               <version>${spring.security.version}</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.security</groupId>
               <artifactId>spring-security-taglibs</artifactId>
               <version>${spring.security.version}</version>
           </dependency>
   
           <!--oracle的依赖需要自己上传至本地仓库-->
           <dependency>
               <groupId>com.oracle</groupId>
               <artifactId>ojdbc14</artifactId>
               <version>${oracle.version}</version>
           </dependency>
   
           <dependency>
               <groupId>javax.annotation</groupId>
               <artifactId>jsr250-api</artifactId>
               <version>1.0</version>
           </dependency>
       </dependencies>
       <build>
           <pluginManagement>
               <plugins>
                   <plugin>
                       <groupId>org.apache.maven.plugins</groupId>
                       <artifactId>maven-compiler-plugin</artifactId>
                       <version>3.2</version>
                       <configuration>
                           <source>1.8</source>
                           <target>1.8</target>
                           <encoding>UTF-8</encoding>
                           <showWarnings>true</showWarnings>
                       </configuration>
                   </plugin>
               </plugins>
           </pluginManagement>
       </build>
   ```

2. 编写实体类

   ```java
   private Date departureTime; // 出发时间
   private String departureTimeStr;
   
   private Integer productStatus; // 状态 0 关闭 1 开启
   private String productStatusStr;
   ```

   1. `productStatus`在数据库中存储的是数字，但是展示时需要变为字符串，所以在其`get()`方法中添加判断即可。

      ```java
      public String getProductStatusStr() {
          if (productStatus!=null){
              if (productStatus.equals(0)) {
                  this.productStatusStr = "关闭";
              }
              else if (productStatus.equals(1)) {
                  this.productStatusStr = "开启";
              }
          }
          return productStatusStr;
      }
      ```

   2. 日期在展示时也是要变为字符串，否则比较麻烦。同样的可以改`get()`方法，在方法中调用DateUtils中静态方法即可。

      ```java
      public class DateUtils {
          public static String date2String(Date date, String pattern) {
              SimpleDateFormat format = new SimpleDateFormat(pattern);
              return format.format(date);
          }
      
          public static Date String2Date(String s, String pattern) throws ParseException {
              SimpleDateFormat format = new SimpleDateFormat(pattern);
              return format.parse(s);
          }
      }
      ```

      ```java
      public String getDepartureTimeStr() {
          if (departureTime!=null){
              departureTimeStr = DateUtils.date2String(departureTime,"yyyy年MM月dd日 HH:mm:ss");
          }
          return departureTimeStr;
      }
      ```

3. 编写持久层接口

4. 编写业务层接口

5. ...

6. `applicationContext.xml`配置、`db.properties`配置

   ```properties
   jdbc.driver=oracle.jdbc.driver.OracleDriver
   jdbc.url=jdbc:oracle:thin:@Conanan:1521:orcl
   jdbc.username=ssm
   jdbc.password=itheima
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context
                              http://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/tx
                              http://www.springframework.org/schema/tx/spring-tx.xsd">
   
   
       <!-- 开启注解扫描，管理service和dao -->
       <context:component-scan base-package="com.itheima.ssm.service"/>
       <context:component-scan base-package="com.itheima.ssm.dao"/>
       
   	<!-- 加载配置文件 -->
       <context:property-placeholder location="classpath:db.properties"/>
   
       <!--配置Druid DataSource连接池，交给IoC管理-->
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
           <property name="driverClassName" value="${jdbc.driver}"/>
           <property name="url" value="${jdbc.url}"/>
           <property name="username" value="${jdbc.username}"/>
           <property name="password" value="${jdbc.password}"/>
       </bean>
       
       <!-- 也可以配置c3p0连接池，交给IoC管理。
       <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <property name="driverClass" value="${jdbc.driver}" />
           <property name="jdbcUrl" value="${jdbc.url}" />
           <property name="user" value="${jdbc.username}" />
           <property name="password" value="${jdbc.password}" />
       </bean>  -->
   
       <!--配置sqlSessionFactory，交给IoC管理-->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource" /> <!--注入DataSource-->
       </bean>
   
       <!--扫描接口的包路径，生成所有接口的代理对象，并放入Spring容器中，交给IoC管理-->
       <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
           <property name="basePackage" value="com.itheima.ssm.dao"/>
           <!-- <property name="typeAliasesPackage" value="com.itheima.domain"/>  JavaBean别名，注解时用不到-->
       </bean>
   
   
       <!-- 配置Spring的声明式事务管理 -->
       <!-- 配置事务管理器 -->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>
       <!--开启事务的注解支持。也可以使用xml配置事务，此处不再叙述，xml+注解方式中有写到-->
       <tx:annotation-driven transaction-manager="transactionManager"/>
   
   </beans>
   ```

7. `springmvc.xml`配置

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                     http://www.springframework.org/schema/beans/spring-beans.xsd
                     http://www.springframework.org/schema/mvc
                     http://www.springframework.org/schema/mvc/spring-mvc.xsd
                     http://www.springframework.org/schema/context
                     http://www.springframework.org/schema/context/spring-context.xsd
                     http://www.springframework.org/schema/aop
                     http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- 扫描controller的注解，别的不扫描 -->
       <context:component-scan base-package="com.itheima.ssm.controller"/>
   
       <!-- 开启对SpringMVC注解的支持 -->
       <mvc:annotation-driven/>
   
       <!-- 配置视图解析器 -->
       <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <!-- JSP文件所在的目录 -->
           <property name="prefix" value="/pages/"/>
           <!-- 文件的后缀名 -->
           <property name="suffix" value=".jsp"/>
       </bean>
   
       <!--释放静态资源，不拦截静态资源。将SpringMVC处理不了的请求交给Tomcat，静态资源就可以访问了-->
       <mvc:default-servlet-handler/>
       <!-- 也可以这样设置
       <mvc:resources location="/css/" mapping="/css/**"/>
       <mvc:resources location="/img/" mapping="/img/**"/>
       <mvc:resources location="/js/" mapping="/js/**"/>
       <mvc:resources location="/plugins/" mapping="/plugins/**"/> -->
   
       <!-- 支持AOP的注解支持，AOP底层使用代理技术
            JDK动态代理，要求必须有接口
            cglib代理，生成子类对象，proxy-target-class="true" 默认使用cglib的方式 -->
       <aop:aspectj-autoproxy proxy-target-class="true"/>
   </beans>
   ```

8. `web.xml`配置

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
            version="3.1">
   
     <!-- 配置加载类路径的配置文件 -->
     <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath*:applicationContext.xml</param-value>
     </context-param>
   
     <!-- 配置监听器 -->
     <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
   
       
     <!-- 前端控制器（加载classpath:springmvc.xml 服务器启动创建servlet） -->
     <servlet>
       <servlet-name>dispatcherServlet</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <!-- 配置初始化参数，创建完DispatcherServlet对象，加载springmvc.xml配置文件 -->
       <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:springmvc.xml</param-value>
       </init-param>
       <!-- 服务器启动的时候，让DispatcherServlet对象创建 -->
       <load-on-startup>1</load-on-startup>
     </servlet>
     <servlet-mapping>
       <servlet-name>dispatcherServlet</servlet-name>
       <url-pattern>*.do</url-pattern>
     </servlet-mapping>
   
       
     <!-- 解决中文乱码过滤器 -->
     <filter>
       <filter-name>characterEncodingFilter</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
         <param-name>encoding</param-name>
         <param-value>UTF-8</param-value>
       </init-param>
     </filter>
     <filter-mapping>
       <filter-name>characterEncodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
     </filter-mapping>
   
     <welcome-file-list>
       <welcome-file>index.html</welcome-file>
       <welcome-file>index.htm</welcome-file>
       <welcome-file>index.jsp</welcome-file>
       <welcome-file>default.html</welcome-file>
       <welcome-file>default.htm</welcome-file>
       <welcome-file>default.jsp</welcome-file>
     </welcome-file-list>
   </web-app>
   ```



### 2.3 产品管理

#### 1、查询所有产品

略

#### 2、添加产品

商品添加页面会有出发**时间**，但是格式是用`-`隔开，所以需要**转换器**，SpringMVC中3.5有介绍**自定义转换器**；还有更方便的注解**`@DateTimeFormat`**可以解决，只需放在JavaBean属性上即可（包括get、set方法和字段(若命名规范)），缺点为只能针对被注解的属性做处理。

```java
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
private Date departureTime; // 出发时间
```



### 2.3 订单管理

#### 1、订单分页展示（PageHelper）

1. **展示**的时间**字符串**需要用工具类从Date类型**转换**为String；订单状态也需要从0、1转换；支付类型也需从0、1、2转换；都是修改其`get`方法即可

2. 查询订单的同时要查询订单中的产品信息（**一对一**），利用注解开发，需要分别提供两个Dao方法

   ```java
   @Repository
   public interface OrdersDao {
       @Select("select * from orders")
       @Results({
           @Result(id = true, property = "id", column = "id"),
           @Result(property = "orderNum", column = "orderNum"),
           @Result(property = "orderTime", column = "orderTime"),
           @Result(property = "orderStatus",column = "orderStatus"),
           @Result(property = "peopleCount",column = "peopleCount"),
           @Result(property = "payType",column = "payType"),
           @Result(property = "orderDesc",column = "orderDesc"),
           @Result(property = "product",column = "productId",
                   one = @One(select = "com.itheima.ssm.dao.ProductDao.findById") )
       })
       List<Orders> findAll();
   }
   ```

   ```java
   @Repository
   public interface ProductDao {
   
       @Select("select * from product where id=#{id}")
       Product findById(String  id);
   }
   ```

3. **带分页的订单展示**

   GitHub中[PageHelper项目地址](https://github.com/pagehelper/Mybatis-PageHelper)。是一个MyBatis的分页插件，使用步骤如下：

   1. pom.xml中**导入依赖**

      ```xml
      <dependency>
          <groupId>com.github.pagehelper</groupId>
          <artifactId>pagehelper</artifactId>
          <version>最新版本</version>
      </dependency>
      ```

   2. 配置拦截器插件

      注意：新版拦截器是 `com.github.pagehelper.PageInterceptor`。  `com.github.pagehelper.PageHelper` 现在是一个特殊的 `dialect` 实现类，是分页插件的默认实现类，提供了和以前相同的用法。

      在 Spring 配置文件中配置拦截器插件：

      ```xml
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource"/>
          <property name="plugins">
              <array>
                  <bean class="com.github.pagehelper.PageInterceptor">
                      <!-- 这里的几个配置主要演示如何使用，如果不理解，一定要去掉下面的配置 -->
                      <property name="properties">
                          <value>
                              helperDialect=mysql
                              reasonable=true
                          </value>
                      </property>
                  </bean>
              </array>
          </property>
      </bean>
      ```

      分页插件参数介绍

      * **`helperDialect`**：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。 你可以配置`helperDialect`属性来指定分页插件使用哪种方言。配置时，可以使用下面的缩写值：
        `oracle`,`mysql`,`mariadb`,`sqlite`,`hsqldb`,`postgresql`,`db2`,`sqlserver`,`informix`,`h2`,`sqlserver2012`,`derby`
        **特别注意：**使用 SqlServer2012 数据库时，需要手动指定为 `sqlserver2012`，否则会使用 SqlServer2005 的方式进行分页。
        你也可以实现 `AbstractHelperDialect`，然后配置该属性为实现类的全限定名称即可使用自定义的实现方法。
      * `offsetAsPageNum`：默认值为 `false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为 `true` 时，会将 `RowBounds` 中的 `offset` 参数当成 `pageNum` 使用，可以用页码和页面大小两个参数进行分页。
      * `rowBoundsWithCount`：默认值为`false`，该参数对使用 `RowBounds` 作为分页参数时有效。 当该参数设置为`true`时，使用 `RowBounds` 分页会进行 count 查询。
      * `pageSizeZero`：默认值为 `false`，当该参数设置为 `true` 时，如果 `pageSize=0` 或者 `RowBounds.limit = 0` 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 `Page` 类型）。
      * **`reasonable`**：分页合理化参数，默认值为`false`。当该参数设置为 `true` 时，`pageNum<=0` 时会查询第一页，`pageNum>pages`（超过总数时），会查询最后一页。默认`false` 时，直接根据参数进行查询。
      * `params`：为了支持`startPage(Object params)`方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 `pageNum,pageSize,count,pageSizeZero,reasonable`，不配置映射的用默认值， 默认值为`pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero`。
      * `supportMethodsArguments`：支持通过 Mapper 接口参数来传递分页参数，默认值`false`，分页插件会从查询方法的参数值中，自动根据上面 `params` 配置的字段中取值，查找到合适的值时就会自动分页。 使用方法可以参考测试代码中的 `com.github.pagehelper.test.basic` 包下的 `ArgumentsMapTest` 和 `ArgumentsObjTest`。
      * `autoRuntimeDialect`：默认值为 `false`。设置为 `true` 时，允许在运行时根据多数据源自动识别对应方言的分页 （不支持自动选择`sqlserver2012`，只能使用`sqlserver`），用法和注意事项参考下面的**场景五**。
      * `closeConn`：默认值为 `true`。当使用运行时动态数据源或没有设置 `helperDialect` 属性自动获取数据库类型时，会自动获取一个数据库连接， 通过该属性来设置是否关闭获取的这个连接，默认`true`关闭，设置为 `false` 后，不会关闭获取的连接，这个参数的设置要根据自己选择的数据源来决定。
      * `aggregateFunctions`(5.1.5+)：默认为所有常见数据库的聚合函数，允许手动添加聚合函数（影响行数），所有以聚合函数开头的函数，在进行 count 转换时，会套一层。其他函数和列会被替换为 count(0)，其中count列可以自己配置。
      * 分页插件可选参数：dialect。默认情况下会使用 PageHelper 方式进行分页，如果想要实现自己的分页逻辑，可以实现 `Dialect`(`com.github.pagehelper.Dialect`) 接口，然后配置该属性为实现类的全限定名称

   3. **`PageHelper.startPage` 静态方法调用**

      调用 `PageHelper.startPage` 静态方法即可，紧跟在这个方法后的第一个**MyBatis 查询方法**会被进行分页。

      ```java
      //获取第1页，10条内容，默认查询总数count
      PageHelper.startPage(1, 10);
      //紧跟着的第一个select方法会被分页
      List<Country> list = countryMapper.selectIf(1);
      ```

   4. **`PageInfo`**的用法：本质就是一个分页Bean

      ```java
      public class PageInfo<T> extends PageSerializable<T> {
          private int pageNum; //当前页
          private int pageSize; //每页展示数据的数量
          private int pages; //总页数
          protected long total; //总记录数。继承父类的。
          private List<T> list; //封装的查询出的数据。继承父类的。
          
          private int size; //当前页的数量，指的是实际展示的数据的数量
          
          
          private int prePage; //前一页。若当前页为首页，则其值为0。查看reasonable参数
          private int nextPage; //下一页。若当前页为尾页，则其值为0。查看reasonable参数
          private boolean isFirstPage = false; //是否为第一页
          private boolean isLastPage = false; //是否为最后一页
          private boolean hasPreviousPage = false; //是否有前一页
          private boolean hasNextPage = false; //是否有下一页
          
          private int navigatePages; //导航页码数
          private int[] navigatepageNums; //所有导航页号
          private int navigateFirstPage; //导航条上的第一页
          private int navigateLastPage; //导航条上的最后一页
          
          //由于startRow和endRow不常用，这里说个具体的用法：可以在页面中"显示startRow到endRow 共size条数据"
          private int startRow; //当前页面第一个元素在数据库中的行号
          private int endRow; //当前页面最后一个元素在数据库中的行号
          
          public PageInfo(List<T> list) {
              this(list, 8);
          }
          ......
      }
      ```

   ------

   ```html
   <!--分页页码处理查看Java Web中黑马旅游网的分页-->
   <a href="${pageContext.request.contextPath}/orders/findAll.do?pageNum=1&pageSize=4"></a> 
   
   <!--通过获取select标签的value属性并在changePageSize设置location.href请求并改变pageSize即可-->
   <select class="form-control" onchange="changePageSize()">
       <option>1</option>
   </select>
   ```

   ```java
   @RequestMapping("/findAll.do")
   public String findAll(Model model, @RequestParam(defaultValue="1") Integer pageNum, @RequestParam(defaultValue="4") Integer pageSize){
       List<Orders> list = ordersService.findAll(pageNum,pageSize);
       PageInfo pageInfo = new PageInfo(list);
       model.addAttribute("pageInfo",pageInfo);
       return "orders-list";
   }
   ```

   ```java
   public List<Orders> findAll(Integer pageNum, Integer pageSize) {
       PageHelper.startPage(pageNum,pageSize);
       return ordersDao.findAll();
   }
   ```


#### 2、订单详情查询

订单查询时需要查询产品信息（多对一）、旅客信息（多对多）、会员信息（多对一）

**多对多查询时**，需要中间表。首先根据订单的id在中间表中查询查询出traveller的id，再根据traveller的多个id查询多个traveller

```html
<button type="button" onclick="location.href='${pageContext.request.contextPath}/orders/findById.do?id=${orders.id}'">详情</button>
```

```java
@RequestMapping("/findById.do")
public String findById(Model model, String id){
    Orders orders = ordersService.findById(id);
    model.addAttribute("orders",orders);
    return "orders-show";
}
```

```java
@Repository
public interface OrdersDao {
    @Select("select * from orders where id=#{id}")
    @Results({
        @Result(id = true, property = "id", column = "id"),
        @Result(property = "orderNum", column = "orderNum"),
        @Result(property = "orderTime", column = "orderTime"),
        @Result(property = "orderStatus",column = "orderStatus"),
        @Result(property = "peopleCount",column = "peopleCount"),
        @Result(property = "payType",column = "payType"),
        @Result(property = "orderDesc",column = "orderDesc"),
        @Result(property = "product",column = "productId",
                one = @One(select = "com.itheima.ssm.dao.ProductDao.findById") ),
        @Result(property = "member",column = "memberId",
                one = @One(select = "com.itheima.ssm.dao.MemberDao.findById") ),
        @Result(property = "travellers",column = "id",
                many = @Many(select = "com.itheima.ssm.dao.TravellerDao.findByOrdersId"))
    })
    Orders findById(String id);
}

@Repository
public interface TravellerDao {
   @Select("select * from TRAVELLER where id in(select TRAVELLERID from ORDER_TRAVELLER where orderId=#{orderId})")
   List<Traveller> findByOrdersId(String orderId);
}
```





### 2.4 用户管理

#### 1、登录(Spring Security数据库认证)

> 只需要根据username查询所有信息并封装，然后由Spring security自动完成登录的校验

用户表（UserInfo）、角色表（Role）、资源权限表（Permission）。其中用户与角色是多对多；角色与资源权限是多对多。

基于Spring Security的数据库认证的操作方式有多种，这里我们介绍使用`UserDetails`、 `UserDetailsService`来完成操作

- **`UserDetails`**接口，作用是于**封装当前进行认证的用户信息**，我们可以对其进行实现，也可以**使用Spring Security提供的一个UserDetails的实现类User**来完成操作

  ```java
  public interface UserDetails extends Serializable {     
  	Collection<? extends GrantedAuthority> getAuthorities();    
  	String getPassword();       
  	String getUsername();       
  	boolean isAccountNonExpired();      
  	boolean isAccountNonLocked();    
  	boolean isCredentialsNonExpired();    
  	boolean isEnabled(); 
  }
  ```

  以下是User类的部分代码：

  ```java
  public class User implements UserDetails, CredentialsContainer {
      private String password;    
      private final String username;    
      private final Set<GrantedAuthority> authorities;    
      private final boolean accountNonExpired; //帐户是否过期    
      private final boolean accountNonLocked; //帐户是否锁定    
      private final boolean credentialsNonExpired; //认证是否过期    
      private final boolean enabled; //帐户是否可用
  }
  ```

- `UserDetailsService`接口，用于规范验证方法的接口

  ```java
  public interface UserDetailsService {        
      UserDetails loadUserByUsername(String username) throws UsernameNotFoundException; 
  }
  ```

------

![](images\使用数据库完成springSecurity用户登录流程分析.png)

SSM综合练习中用户登录来完成Spring Security的认证操作：

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-web</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-config</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   <!--下面不认资料中没有导入-->
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-core</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-taglibs</artifactId>
       <version>${spring.security.version}</version>
   </dependency>
   ```

2. `web.xml`中配置**`springSecurityFilterChain`**（必须这个名字），别忘了监听器加载`spring-security.xml`配置文件

   ```xml
   <filter>
       <filter-name>springSecurityFilterChain</filter-name>
       <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
   </filter>
   <filter-mapping>
       <filter-name>springSecurityFilterChain</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
   
   <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath*:spring-security.xml</param-value>
   </context-param>
   <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
   ```

3. `spring-security.xml`中配置如下

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:security="http://www.springframework.org/schema/security"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                              http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/security
                              http://www.springframework.org/schema/security/spring-security.xsd">
   
       <!-- 配置不拦截的资源（静态资源及登录相关）  -->
       <security:http pattern="/login.jsp" security="none"/>
       <security:http pattern="/failer.jsp" security="none"/>
       <security:http pattern="/css/**" security="none"/>
       <security:http pattern="/img/**" security="none"/>
       <security:http pattern="/plugins/**" security="none"/>
   
   
       <!--配置授权的具体的规则
        auto-config="true"	不用自己编写登录的页面，框架提供默认登录页面
        use-expressions="true"	是否使用SPEL表达式-->
       <security:http auto-config="true" use-expressions="true">
           <!-- 配置具体的拦截的规则 pattern="请求路径的规则" access="访问系统的人，必须有ROLE_USER等角色" -->
           <security:intercept-url pattern="/**" access="hasAnyRole('ROLE_ADMIN','ROLE_USER')"/>
           <!-- 定义跳转的具体的页面 -->
           <security:form-login
                                login-page="/login.jsp"
                                login-processing-url="/login.do"
                                default-target-url="/index.jsp"
                                authentication-failure-url="/failer.jsp"
                                authentication-success-forward-url="/pages/main.jsp"
                                />
           <!-- 关闭跨域请求 -->
           <security:csrf disabled="true"/>
           <!-- 退出，invalidate-session 是否删除session logout-url：登出处理url logout-successurl：登出成功页面-->
           <security:logout invalidate-session="true" logout-url="/logout.do" logout-success-url="/login.jsp"/>
       </security:http>
   
   
       <!-- 认证管理器，指定了认证需要访问的service 。切换成数据库中的用户名和密码。-->
       <security:authentication-manager>
           <security:authentication-provider user-service-ref="userService">
               <!-- 配置加密的方式，以便框架来解密。   否则使用{noop}拼接上密码（不推荐）-->
               <security:password-encoder ref="passwordEncoder"/> 
           </security:authentication-provider>
       </security:authentication-manager>
   
   
       <!-- 配置加密类，注入后直接调用即可加密、解密 -->
       <bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
   
   
       <!-- 提供了入门的方式，在内存中存入用户名和密码
       <security:authentication-manager>
        <security:authentication-provider>
         <security:user-service>
          <security:user name="admin" password="{noop}admin" authorities="ROLE_USER"/>
         </security:user-service>
        </security:authentication-provider>
       </security:authentication-manager>
       -->
   </beans>
   ```

4. 实现`UserDetailsService`接口

   ```java
   public interface UserService extends UserDetailsService {}
   ```

   ```java
   @Service("userService")
   @Transactional
   public class UserServiceImpl implements UserService {
       @Autowired
       private UserDao userDao;
   
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
   
           UserInfo userInfo = userDao.findByUsername(s);//UserInfo为和数据库对应的实体类
           //User user = new User(userInfo.getUsername(), "{noop}" + userInfo.getPassword(), getAuthorities(userInfo.getRoles()));
           //添加用户状态的判断。封装到UserDetails实现类User。{noop}是明文密码暂时解决方案，后续框架利用配置的加密bean自动对其解密
           User user = new User(userInfo.getUsername(), userInfo.getPassword(),userInfo.getStatus()==0?false:true,true,true,true, getAuthorities(userInfo.getRoles()));
           return user;
       }
   
       public List<SimpleGrantedAuthority> getAuthorities(List<Role> roles) {
           List<SimpleGrantedAuthority> list = new ArrayList<>();
           for (Role role : roles) {
               list.add(new SimpleGrantedAuthority("ROLE_" + role.getRoleName()));
           }
           return list;
       }
   }
   ```

5. UserDao中查询user时还需查询user所属的角色Role表（多对多）

   ```java
   @Repository
   public interface UserDao {
   
       @Select("select * from users where username=#{username}")
       @Results({
               @Result(id = true, property = "id", column = "id"),
               @Result(column = "username", property = "username"),
               @Result(column = "email", property = "email"),
               @Result(column = "password", property = "password"),
               @Result(column = "phoneNum", property = "phoneNum"),
               @Result(column = "status", property = "status"),
               @Result(column = "id", property = "roles",
                       many = @Many(select = "com.itheima.ssm.dao.RoleDao.findRoleByUserId"))
       })
       UserInfo findByUsername(String username);
   }
   ```

6. login页面：表单请求方法为POST，action为login.do，用户名和密码的表单name为username、password（不乱修改则不需要在spring-security.xml中配置）

#### 2、退出(Spring Security)

在spring-security.xml配置中，只要前端页面访问logout.do即可退出

```html
<a href="${pageContext.request.contextPath}/logout.do">注销</a>
```



#### 3、查询所有用户

略

#### 4、添加用户(BCryptPasswordEncoder加密)

后台管理系统，一般是不提供注册功能，由管理员直接分配。

spring-security.xml中配置了认证管理器的加密方式和BCryptPasswordEncoder的Bean，可以用于加密、解密（每次加密后数据不同，但是前缀一致），只需在service中调用dao的save方法前加密即可（删掉之前添加的{noop}）

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/save.do")
    public String save(Model model,UserInfo userInfo){
        userService.save(userInfo);
        return "redirect:findAll.do";
    }
}
```

```java
@Service("userService")
@Transactional
public class UserServiceImpl implements UserService {
    @Autowired
    private BCryptPasswordEncoder bCryptPasswordEncoder;
    
    @Override
    public void save(UserInfo userInfo) {
        userInfo.setPassword(bCryptPasswordEncoder.encode(userInfo.getPassword()));//加密
        userDao.save(userInfo);
    }
}
```

#### 5、用户详情(用户、角色、资源多表联查)

通过id查询用户及其角色信息（多对多）、以及角色包含的资源权限信息（多对多）

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/findById.do")
    public String findById(Model model,String id){
        UserInfo userInfo = userService.findById(id);
        model.addAttribute("user",userInfo);
        return "user-show";
    }
}
```

```java
@Repository
public interface UserDao {    
    @Select("select * from users where id=#{id}")
    @Results({
        @Result(id = true, property = "id", column = "id"),
        @Result(column = "username", property = "username"),
        @Result(column = "email", property = "email"),
        @Result(column = "password", property = "password"),
        @Result(column = "phoneNum", property = "phoneNum"),
        @Result(column = "status", property = "status"),
        @Result(column = "id", property = "roles",
                many = @Many(select = "com.itheima.ssm.dao.RoleDao.findRoleByUserId"))
    })
    UserInfo findById(String id);
}

@Repository
public interface RoleDao {
    @Select("select * from role where id in (select roleid from USERS_ROLE where userId=#{userId})")
    @Results({
        @Result(id = true,property = "id",column = "id"),
        @Result(property = "roleName",column = "roleName"),
        @Result(property = "roleDesc",column = "roleDesc"),
        @Result(property = "permissions",column = "id",
                many = @Many(select = "com.itheima.ssm.dao.PermissionDao.findByRoleId"))
    })
    List<Role> findRoleByUserId(String userId);
}

@Repository
public interface PermissionDao {
    @Select("select * from PERMISSION where id in(
            select PERMISSIONID from ROLE_PERMISSION where roleId=#{roleId})")
    List<Permission> findByRoleId(String roleId);
}
```

注意：在用AdminLTE展示用户、角色、权限时树表格的问题需要注意`data-tt-id`和`data-tt-parent-id`的值要对应

```html
<c:forEach items="${user.roles}" var="role" varStatus="vs">
    <tr data-tt-id="${vs.index+1}" data-tt-parent-id="0">
        <td>${role.roleName }</td>
        <td>${role.roleDesc }</td>
    </tr>
    <c:forEach items="${role.permissions}" var="permission">
        <tr data-tt-id="1-1" data-tt-parent-id="${vs.index+1}">
            <td>${permission.permissionName}</td>
            <td>${permission.url}</td>
        </tr>
    </c:forEach>
</c:forEach>
```



### 2.5 角色管理

#### 1、查询所有角色

仅仅只有角色信息，不包含其他表信息。类似2.4的第3小节

#### 2、添加角色

仅仅只有角色信息，不包含其他表信息。类似2.4的第4小节，但不用加密



### 2.6 资源权限管理

#### 1、查询所有权限

仅仅只有权限信息，不包含其他表信息。类似2.4的第3小节

#### 2、添加权限

仅仅只有权限信息，不包含其他表信息。类似2.4的第4小节，但不用加密



### 2.7 权限关联

#### 1、用户关联角色

多对多关系，先查出这个用户没有的角色信息；再向user_role表中插入数据

- 信息的回显

  1. 根据用户id查询用户信息（包括已有角色信息）

  2. 根据用户id查询用户没有的角色信息

  3. 将1和2的信息封装并跳转至添加页面

     ```java
     @Controller
     @RequestMapping("/user")
     public class UserController {
         @RequestMapping("/findUserByIdAndAllRole.do")
         public String findUserByIdAndAllRole(Model model,String id){
             UserInfo userInfo = userService.findById(id); //dao中省略的，查看之前的代码
             List<Role> list = userService.findOtherRoles(id);
             model.addAttribute("user",userInfo);
             model.addAttribute("roleList",list);
             return "user-role-add";
         }
     }
     ```

     ```java
     @Repository
     public interface UserDao {
        	//省略了findById
         @Select("select * from role where id not in (select roleId from USERS_ROLE where userId = #{userId})")
         List<Role> findOtherRoles(String userId);
     }
     ```

- 用户角色的添加：需要提供用户的id和角色的id数组。（若也面不显示用户，则使用hidden隐藏）

  dao中多参数添加时需要`@Param`注解

  ```html
  <input type="hidden" name="userId" value="${user.id}">
  <input name="ids" type="checkbox" value="${role.id}"><!--使用了类似foreach标签，所以有多个值-->
  ```

  ```java
  @RequestMapping("/addRoleToUser.do")
  public String addRoleToUser(String userId,String[] ids){//获取的id是数组
      userService.addRoleToUser(userId,ids);
      return "redirect:findAll.do";
  }
  ```

  ```java
  @Insert("insert into USERS_ROLE values(#{userId},#{roleId})")
  void addRoleToUser(@Param("userId") String userId, @Param("roleId") String roleId);
  ```


#### 2、角色关联权限

多对多关系，先查出这个角色没有的权限信息；再向role_permission表中插入数据。类似用户管理角色



### 2.8 权限控制

#### 2.8.1 服务器端方法级权限控制

在服务器端我们可以通过Spring security提供的注解对方法来进行权限控制。Spring Security在方法的权限控制上
支持三种类型的注解，JSR-250注解、@Secured注解和支持表达式的注解，这三种注解默认都是没有启用的，需要
单独通过`<global-method-security>`元素的对应属性进行**启用**：

* spring-security.xml配置文件

  ```xml
  <security:global-method-security jsr250-annotations="enabled" secured-annotations="enabled" pre-post-annotations="enabled"/>
  ```

* 注解开启

  `@EnableGlobalMethodSecurity` ：Spring Security默认是禁用注解的，要想开启注解，需要在继承
  `WebSecurityConfigurerAdapter`的类上加该注解，并在该类中将`AuthenticationManager`定义为Bean

##### 1、JSR-250注解

* pom.xml中添加依赖

  ```xml
  <dependency>
      <groupId>javax.annotation</groupId>
      <artifactId>jsr250-api</artifactId>
      <version>1.0</version>
  </dependency>
  ```

* `@RolesAllowed`表示访问对应方法时所应该具有的角色

  ```java
  @RolesAllowed({"USER", "ADMIN"}) 
  ```

  该方法只要具有"USER", "ADMIN"任意一种权限就可以访问。这里可以省略前缀ROLE_，实际的权限可能是ROLE_ADMIN

  `@PermitAll`表示允许所有的角色进行访问，也就是说不进行权限控制

  `@DenyAll`是和PermitAll相反的，表示无论什么角色都不能访问

##### 2、@Secured注解

* `@Secured`注解标注的方法进行权限控制的支持，其值默认为disabled

  ```java
  @Secured("ROLE_ADMIN")
  public Account readAccount(Long id);
  ```

##### 3、支持Sp表达式的注解

* `@PreAuthorize` 在方法调用之前,基于表达式的计算结果来限制对方法的访问

  ```java
  @PreAuthorize("#userId == authentication.principal.userId or hasAuthority(‘ADMIN’)")
  void changePassword(long userId ){ }
  ```

  这里表示在changePassword方法执行之前，判断方法参数userId的值是否等于principal中保存的当前用户的userId，或者当前用户是否具有ROLE_ADMIN权限，两种符合其一，就可以访问该方法。

* `@PostAuthorize` 允许方法调用，但是如果表达式计算结果为false，将抛出一个安全性异常

  ```java
  @PostAuthorize
  User getUser("returnObject.userId == authentication.principal.userId orhasPermission(returnObject, 'ADMIN')");
  ```

* `@PostFilter` 允许方法调用,但必须按照表达式来过滤方法的结果
  `@PreFilter` 允许方法调用,但必须在进入方法之前过滤输入值





#### 2.8.2 页面上权限控制

* pom.xml中依赖导入

  ```xml
  <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
      <version>version</version>
  </dependency>
  ```

* 页面导入

  ```html
  <%@taglib uri="http://www.springframework.org/security/tags" prefix="security"%>
  ```

* **authentication**：代表的是当前认证对象，可以获取当前**认证对象信息**，例如用户名

  ```html
  <security:authentication property="" htmlEscape="" scope="" var=""/>
  ```

  `property`：只允许指定Authentication所拥有的属性，可以进行属性的级联获取，如“principle.username”，
  不允许直接通过方法进行调用

  `htmlEscape`：表示是否需要将html进行转义。默认为true。

  `scope`：与var属性一起使用，用于指定存放获取的结果的属性名的作用范围，默认我pageContext。Jsp中拥
  有的作用范围都进行进行指定

  `var`： 用于指定一个属性名，这样当获取到了authentication的相关信息后会将其以var指定的属性名进行存
  放，默认是存放在pageConext中

* **authorize**：用来判断普通**权限**的，通过判断用户是否具有对应的权限而控制其所包含内容的显示

  ```html
  <security:authorize access="" method="" url="" var=""></security:authorize>
  ```

  `access`：需要使用表达式来判断权限，当表达式的返回结果为true时表示拥有对应的权限

  `method`：method属性是配合url属性一起使用的，表示用户应当具有指定url指定method访问的权限，method的默认值为GET，可选值为http请求的7种方法

  `url`：url表示如果用户拥有访问指定url的权限即表示可以显示authorize标签包含的内容

  `var`：用于指定将权限鉴定的结果存放在pageContext的哪个属性中

* accesscontrollist：用于鉴定ACL权限的。其一共定义了三个属性：hasPermission、domainObject和var，前两必须指定

  ```html
  <security:accesscontrollist hasPermission="" domainObject="" var=""></security:accesscontrollist>
  ```

  `hasPermission`：hasPermission属性用于指定以逗号分隔的权限列表

  `domainObject`：domainObject用于指定对应的域对象

  `var`：var则是用以将鉴定的结果以指定的属性名存入pageContext中，以供同一页面的其它地方使用





### 2.9 AOP日志

记录每次访问者ip、名称、执行时间等信息

* 数据库表的创建

  ```sql
  CREATE TABLE sysLog(
      id VARCHAR2(32) default SYS_GUID() PRIMARY KEY,
      visitTime timestamp,
      username VARCHAR2(50),
      ip VARCHAR2(30),
      url VARCHAR2(50),
      executionTime int,
      method VARCHAR2(200)
  )
  ```

* 对应的实体类

  ```java
  public class SysLog implements Serializable {
  
      private String id;
      private Date visitTime;
      private String visitTimeStr;
      private String username;
      private String ip;
      private String url;
      private Long executionTime;
      private String method;
  }
  ```

基于AOP的日志处理

* **切面类**（可以放在Controller包中）

  > 课程中所写的利用`getArgs()`获取的并不是方法的参数，也不能继续用getClass获取Class类型的参数列表

  ```java
  @Component
  @Aspect
  public class LogAspect {
  
      @Autowired
      private HttpServletRequest request;//需要在web.xml中配置Request监听器，如下面代码
      @Autowired
      private LogAspectService logAspectService;
  
      private Date startTime; // 访问时间
      private Class executionClass;// 访问的类
      private String methodName; // 访问的方法
      private String url = ""; // 它的值应该是类上的@RequestMapping的value+方法上的 @RequestMapping的value
      private Long executionTime;//访问的时长
      private String ip; //ip
      private String username;//访问的人
  
      @Pointcut("execution(* com.itheima.ssm.controller.*.*(..))")
      private void pointCut() {
      }
  
      @Around("pointCut()")
      public Object around(ProceedingJoinPoint pjp) {
          Object obj = null;
          Object[] args = pjp.getArgs();//执行的参数
          try {
  
              startTime = new Date(); // 访问时间
  
              /*==================================*/
              obj = pjp.proceed(args);
              /*==================================*/
  
              executionClass = pjp.getTarget().getClass();// 获取访问的类
              methodName = pjp.getSignature().getName();// 获取访问的方法的名称
              url = request.getRequestURI();//URI
              ip = request.getRemoteAddr();//ip
              // 可以通过securityContext获取，也可以从request.getSession中获取
              SecurityContext context = SecurityContextHolder.getContext(); //
              username = ((User) (context.getAuthentication().getPrincipal())).getUsername();
              executionTime = System.currentTimeMillis() - startTime.getTime();//访问的时长
  
              SysLog sysLog = new SysLog();
              sysLog.setVisitTime(startTime);
              sysLog.setMethod("[类名]" + executionClass.getName() + "[方法名]" + methodName);
              sysLog.setUrl(url);
              sysLog.setIp(ip);
              sysLog.setUsername(username);
              sysLog.setExecutionTime(executionTime);
              logAspectService.save(sysLog);//调用service、继而调用dao保存日志
  
          } catch (Throwable throwable) {
              throwable.printStackTrace();
          }
          return obj;
      }
  }
  ```

  ```xml
  <!--Request监听器-->
    <listener>
      <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
    </listener>
  ```





