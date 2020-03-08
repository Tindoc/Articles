# Spring 相关
> 简介：Spring 是一个基于 IOC 和 AOP 的结构 J2EE 系统的框架
>
> 参考资料：
>
> 1. [官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/index.html) （另：GitHub 该项目仓库主页的 Wiki 中也有）
>
> 2. [W3Cschool]()

## Spring Framework 体系结构
> 体系结构的作用：便于理解（本文大致按照体系结构的内容来编排）
- Core Container 核心容器
    - Core
        spring-core
    - Beans
        spring-beans
    - Context
        spring-context, spring-context-support
    - SpEL
        spring-expression
- Data Access/Integration 数据访问/集成
    - JDBC(Java Database Connectivity)
    - ORM(Object Relational Mapping)
    - OXM(Object XML Mapping)
    - JMS(Java Message Service)
    - Transactions
- Web(MVC/Remoting) 网络
    - Web
    - WebSocket
    - Servlet
    - Portlet
- Others
    - AOP(Aspect-Oriented Programming) 面向方面编程
    - Aspects
    - Instrumentation
    - Messaging
    - Test 测试

## 配置文件 applicationContext.xml
> 配置文件必须放在 src 目录下
以下将介绍两种形式，手动逐个配置 Bean 和 使用自动扫描。
### 1. 逐个配置 Bean
#### applicationContext.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
http://www.springframework.org/schema/context     
http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:annotation-config/> <!-- 指明使用注解方式进行配置 -->
    <bean name="categoryBean" class="com.sample.pojo.Category">
        <property name="name" value="category1" />
    </bean>

    <bean name="productBean" class="com.sample.pojo.Product">
        <property name="name" value="product1" />
        <!-- <property name="category" ref="c"> 因为使用了注解自动注入不再需要指明 -->
    </bean>
</beans>
```
##### Product.java
Product.java
``` java
package com.sample.pojo;

import org.springframework.beans.factory.annotation.Autowired;

public class Product {

    private int id;
    private String name;
    @Autowired // 自动装配，需要在 applicationContext.xml 中指明 <context:annotation-config/>
    // 或者使用 @Resource(name="categoryBean") // "categoryBean" 对应 applicationContext.xml 中的 Bean 的名字
    private Category category;
    
    // getter & setter
}
```

### 2. 自动扫描
#### applicationContext.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
   http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/aop
   http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
   http://www.springframework.org/schema/tx
   http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
   http://www.springframework.org/schema/context     
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  
    <context:component-scan base-package="com.sample.pojo"/> <!-- 自动扫描 com.sample.pojo 包，其中使用注解 @Component/@Service/@Controller 标记的类会自动配置成 Bean -->
     
</beans>
```

#### Category.java
``` java
package com.sample.pojo;

import javax.annotation.Resource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component("productBean") // 指明名字，作用：通过 (Product)context.getBean("productBean"); 获得该类，未指明的情况可以使用首字母小写的类名来获得
public class Product {

    private int id;
    private String name="product1";
    
    @Autowired
    private Category category;

    // getter & setter
}
```

## MetaData 配置元数据
配置元数据可以通过 XML，Java 注解或 Java 代码来表示。通过阅读配置元数据提供的指令，容器知道对哪些对象进行实例化，配置和组装。

## Spring 容器
两种
### Spring BeanFactory 容器
来自 org.springframework.beans.factory.BeanFactory 接口

常用接口实现类：` XmlBeanFactory() `
``` java
XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("beans.xml"));
Sample obj = (Sample)factory.getBean("beanId");
```

### Spring ApplicationContext 容器
来自 org.springframework.context.ApplicationContext 接口
> 也被称为 Spring 上下文

常用接口实现类（三种）：
- ` FileSystemXmlApplicationContext `
> 需要提供 XML 文件的完整路径
``` java
ApplicationContext context = new FileSystemXmlApplicationContext("D:proj/sample/beans.xml");
Sample obj = (Sample) context.getBean("beanId");
```

- ` ClassPathXmlApplicationContext `
> 会在 CLASSPATH 环境变量中自动搜索 bean 配置文件
``` java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
Sample obj = (Sample) context.getBean("beanId");
```

- ` WebXmlApplicationContext `
> 会在 web 应用程序的范围内加载在 XML 文件中已被定义的 bean

> ApplicationContext 容器包括 BeanFactory 容器的所有功能，推荐使用（只在移动设备或基于 applet 的轻量级的应用程序中建议使用 BeanFactory，可显著提升这些应用的数据量和速度）



## Bean
### Bean 与 Spring 容器的关系
1. Spring 容器读取 **Bean 配置信息**（即 metadata） 在容器内生成一个 **Bean 定义注册表**
2. Spring 容器根据 **Bean 定义注册表** 找到实际的类（即 class 属性指向的类）来实例化 Bean
3. Spring 容器将 Bean 实例放到 Spring 容器中的 **Bean 缓存池** 中
4. 应用程序需要使用 Bean 时，Spring 容器就会从 **Bean 缓存池** 中取出实例好的 Bean

### 一个 Bean 配置信息的例子
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- 一个简单的 bean 定义 -->
   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- 一个懒加载的 bean 定义 -->
   <bean id="..." class="..." lazy-init="true">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- 一个带初始化方法的 bean 定义 -->
   <bean id="..." class="..." init-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- 一个带销毁方法的 bean 定义 -->
   <bean id="..." class="..." destroy-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- more bean definitions go here -->

</beans>
```
其中，
1. ` xmlns="http://www.springframework.org/schema/beans" `是用于 Spring Bean 定义的默认命名空间，没有空间名。
2. ` xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" `是 xsi 命名空间。用于为每个文档中命名空间指定相应的Schema样式文件，是标准组织定义的标准命名空间
3. 初始化方法和销毁方法都是指定义该 bean 的容器被初始化和销毁时执行的方法

### Bean 的属性
#### class
强制要求
#### name/id
唯一的 bean 标识符
#### scope
Bean 作用域，共五种：
- singleton（默认）
    在 spring IoC 容器仅存在一个 Bean 实例，Bean 以 **单例方式** 存在。

    例子：
    ``` java
    // XML 主要定义如下：
    // <bean id="beanId" class="com.sample.Sample" scope="singleton"></bean>

    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    Sample objA = (Sample)context.getBean("beanId");
    objA.setMessage("I'm object A");
    objA.getMessage(); // 输出 I'm object A
    Sample objB = (Sample)context.getBean("beanId");
    objB.getMessage(); // 同样输出 I'm object A
    ```
- prototype
    每次从容器中调用 Bean 时，都返回一个新的实例，即每次调用 getBean() 时，相当于执行 newXxxBean()。

    Prototype 是 **原型类型**，它在我们创建容器的时候并没有实例化，而是当获取 bean 的时候才创建一个对象，而且我们每次获取到的对象都不是同一个对象。

    例子：
    ``` java
    // XML 主要定义如下：
    // <bean id="beanId" class="com.sample.Sample" scope="prototype"></bean>

    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    Sample objA = (Sample)context.getBean("beanId");
    objA.setMessage("I'm object A");
    objA.getMessage(); // 输出 I'm object A
    Sample objB = (Sample)context.getBean("beanId");
    objB.getMessage(); // 输出 null
    ```
- request
    每次 HTTP 请求都会创建一个新的 Bean，该作用域仅适用于 WebApplicationContext 环境
- session
    同一个 HTTP Session 共享一个 Bean，不同 Session 使用不同的 Bean，仅适用于 WebApplicationContext 环境
- global-session
    一般用于 Portlet 应用环境，该运用域仅适用于 WebApplicationContext 环境
#### constructor-arg, properties, autowiring mode 
用于注入依赖关系
#### lazy-initialization mode
懒加载，由` lazy-init="true" `指定，见 [一个 Bean 配置信息的例子]()
#### initialization, destruction
与 Bean 的生命周期有关，生命周期简单描述为：Bean的定义 -> Bean的初始化 -> Bean的使用 -> Bean的销毁

其中，正如 [一个 Bean 配置信息的例子]() 中看到的 init-method 和 destroy-method 属性，这些属性指明的方法都是 class 属性指明的类内方法（该方法无参数，且返回类型为 void）

例子：
- beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="beanId" class="com.sample.Sample" init-method="init" destroy-method="destroy">
        <property name="message" value="Hello World!"/> <!-- 初始化参数 -->
    </bean>

    </beans>
    ```
- Sample.java
    ``` java
    package com.sample;

    public class Sample {
    private String message;

    public void setMessage(String message){
        this.message  = message;
    }
    public void getMessage(){
        System.out.println(message);
    }
    public void init(){
        System.out.println("Bean is going through init.");
    }
    public void destroy(){
        System.out.println("Bean will destroy now.");
    }
    }
    ```
- Main.java
    ``` java
    package com.sample;

    import org.springframework.context.support.AbstractApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;

    public class Main {
    public static void main(String[] args) {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        Sample obj = (Sample) context.getBean("beanId");
        obj.getMessage();
        context.registerShutdownHook(); // 因为这是在非 web 应用程序环境中使用 Spring 的 IoC 容器，那么要在 JVM 中注册关闭 hook 才可以可以确保容器正常关闭，才会调用 bean 的 destroy-method 指定个方法

        /* 输出如下：
        * Bean is going through init.
        * Hello World!
        * Bean will destroy now.
        */
    }
    }
    ```



底层实现：
- 初始化回调
org.springframework.beans.factory.InitializingBean 接口中有一个方法` void afterPropertiesSet() throws Exception; `

如果在类中实现该接口并重新该方法，即可实现 init-method 的效果，例如：
``` java
public class Sample implements InitializingBean {
   public void afterPropertiesSet() {
      // do some initialization work
   }
}
```

- 销毁回调
org.springframework.beans.factory.DisposableBean 接口中有一个方法` void destroy() throws Exception; `

如果在类中实现该接口并重写该方法，即可实现 destroy-method 的效果，例如：
``` java
public class Sample implements DisposableBean {
   public void destroy() {
      // do some destruction work
   }
}
```
> 但不建议直接使用这两个回调方法来实现 init-method, destroy-method

另外：如果多个 Bean 有相同名字的初始化方法和销毁方法，可以在 xml 文件中指明默认初始化和销毁方法，例如：
``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy"> <!-- 注意 -->

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```

### Bean 的后置处理器
> 关键字：BeanPostProcessor 接口
Bean 后置处理器允许在调用初始化方法 **前**、**后** 对 Bean 进行额外的处理（即在 init-method 前后再做处理）

**注意：后置处理器是一个类，需要实现 BeanPostProcessor 接口且覆盖两个方法。处理器对容器内所有 Bean 都有效（无法指定对某个 Bean 生效）。如果存在多个后置处理器，可以使用该后置处理器类的属性 order （由实现的 Ordered 接口得来）来指定执行顺序。**

例子：
- beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

        <bean id="beanId" class="com.sample.Sample" init-method="init" destroy-method="destroy">
            <property name="message" value="Hello World!"/>
        </bean>

        <bean class="com.sample.InitSample" /> <!-- 注意 -->
    </beans>
    ```
- Sample.java
    ``` java
    package com.sample;

    public class Sample {
    private String message;

    public void setMessage(String message){
        this.message  = message;
    }
    public void getMessage(){
        System.out.println(message);
    }
    public void init(){
        System.out.println("Bean is going through init.");
    }
    public void destroy(){
        System.out.println("Bean will destroy now.");
    }
    }
    ```
- InitSample.java 即后置处理器类
    ``` java
    package com.sample;

    import org.springframework.beans.factory.config.BeanPostProcessor;
    import org.springframework.beans.BeansException;

    public class InitSample implements BeanPostProcessor {
        // 初始化前
        public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("BeforeInitialization : " + beanName);
            return bean;  // you can return any other object as well
        }
        // 初始化后
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("AfterInitialization : " + beanName);
            return bean;  // you can return any other object as well
        }
    }
    ```
- Main.java 不变
    ``` java
    /* 输出如下：
     * BeforeInitialization : beanId
     * Bean is going through init.
     * AfterInitialization : beanId
     * Hello World!
     * Bean will destroy now.
    ```

### Bean 定义继承
> 关键字：parent 属性

Spring Bean 定义的继承与 Java 类的继承无关，但是继承的概念是一样的。你可以定义一个父 bean 的定义作为模板和其他子 bean 就可以从父 bean 中继承所需的配置。

**作用：继承父 bean 的配置（参数，作用域，初始化方法，销毁方法等）**

例子：
- 简单例子
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

        <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
            <property name="message1" value="Hello World!"/>
            <property name="message2" value="Hello Second World!"/>
        </bean>

        <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="helloWorld">
            <property name="message1" value="Hello India!"/> <!-- 这里覆盖了父 bean 的属性 -->
            <!-- 其中 message2 的属性将直接使用父 bean 的 -->
            <property name="message3" value="Namaste India!"/> <!-- 这是新增的属性 -->
        </bean>

    </beans>
    ```
- 抽象父 bean 例子
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

        <bean id="beanTeamplate" abstract="true"> <!-- 指明为抽象，自身不能被实例化 -->
            <property name="message1" value="Hello World!"/>
            <property name="message2" value="Hello Second World!"/>
            <property name="message3" value="Namaste India!"/>
        </bean>

        <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
            <property name="message1" value="Hello India!"/> <!-- 这里覆盖了父 bean 的属性 -->
            <!-- 其中 message2 的属性将直接使用父 bean 的 -->
            <property name="message3" value="Namaste India!"/> <!-- 这里覆盖了父 bean 的属性 -->
        </bean>

    </beans>
    ```

## 依赖注入
### 简单解释控制反转
- 传统方式
    ``` java
    public class A {
        private B b;  
        public A() {
            b = new B(); // 由 A 类控制生成 B 类的对象
        }
    }
    ```
- 控制反转
    ``` java
    public class A {
        private B b;  
        public A(B b) {
            this.b = b; // A 类只拿生成好的 B 类对象，自身不生成
        }
    }
    ```

### 依赖注入的两种方式
最佳实践是 **有强制性依存关系的使用构造函数来注入，有可选依赖关系使用 setter 函数来注入**
#### 1. 基于构造函数的依赖注入（Constructor-based dependency indection）
主要是在配置元数据中指明` constructor-arg `元素

例子：
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <!-- Definition for textEditor bean -->
    <bean id="textEditor" class="com.tutorialspoint.TextEditor"> <!-- TextEditor 类的构造函数参数为 SpellChecker 类的对象 -->
        <constructor-arg ref="spellChecker"/> <!-- 使用 constructor-arg 来指明构造函数参数 -->
        <!-- 传递引用，使用 ref；传递值，使用 value，需指明类型 -->
        <!-- <constructor-arg type="int" value="2001"/> -->
    </bean>

    <!-- Definition for spellChecker bean -->
    <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"></bean>
</beans>
```

如果构造函数的参数不止一个，可以按照参数的顺序来声明 constructor-arg，或者使用 index 显式指明属性顺序，例如
``` xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="2001"/>
    <constructor-arg index="1" value="Zara"/>
</bean>
```

#### 2. 基于设置函数的依赖注入（Setter-based dependency injection）
主要在配置元数据中指明` property `元素

例子：
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
      <property name="spellChecker" ref="spellChecker"/> <!-- 使用 property 来指明 setter 函数参数 --> 
      <!-- 上面的 name 指的是 TextEditor 类内某个属性的名字，ref 指的是某个 bean 的 id/name（这里恰巧一样了） -->
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"></bean>
</beans>
```

### 依赖注入 XML 的简便写法
关键字：使用 p-namespace

即使用` p:属性名="属性值" `来注入依赖。如果注入的引用类型，在属性名后面加上 ref 即可，即` p:属性名-ref="Bean的Id或Name" `

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
<!-- 上面引入了 xmlns:p="http://www.springframework.org/schema/p" -->

   <bean id="john-classic" class="com.example.Person"
      p:name="John Doe"
      p:spouse-ref="jane"/>
   </bean>
   <!-- 上面等同于：
   <bean id="john-classic" class="com.example.Person">
      <property name="name" value="John Doe"/>
      <property name="spouse" ref="jane"/>
   </bean> -->

   <bean name="jane" class="com.example.Person"
      p:name="Jane Doe"/>
   </bean>
   <!-- 上面等同于：
   <bean name="jane" class="com.example.Person">
      <property name="name" value="John Doe"/>
   </bean> -->

</beans>
```

### 依赖注入的类型
依赖注入的类型大致可以分为三类：
- 常规类型
    - 值类型（value）
    - 引用类型（ref）
- 集合类型
    - list
    - set
    - map
    - props
        与 map 类似，map 的键/值可以是任意类型，而 props 的键/值都是字符串类型
- null 和空字符串

#### 集合类型
例子：
- JavaCollection.java
    ``` java
    package com.tutorialspoint;

    import java.util.*;
    public class JavaCollection {

        List addressList;
        Set  addressSet;
        Map  addressMap;
        Properties addressProp; // Properties 类主要用来读取配置文件，其中配置文件默认为键/值都为字符串类型的值

        // AddressList
        public void setAddressList(List addressList) {
            this.addressList = addressList;
        }
        public List getAddressList() {
            System.out.println("List Elements :"  + addressList);
            return addressList;
        }
        
        // AddressSet
        public void setAddressSet(Set addressSet) {
            this.addressSet = addressSet;
        }
        public Set getAddressSet() {
            System.out.println("Set Elements :"  + addressSet);
            return addressSet;
        }
        
        // AddressMap
        public void setAddressMap(Map addressMap) {
            this.addressMap = addressMap;
        }  
        public Map getAddressMap() {
            System.out.println("Map Elements :"  + addressMap);
            return addressMap;
        }

        // AddressProp
        public void setAddressProp(Properties addressProp) {
            this.addressProp = addressProp;
        } 
        public Properties getAddressProp() {
            System.out.println("Property Elements :"  + addressProp);
            return addressProp;
        }
    }
    ```

- beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

        <!-- Definition for javaCollection -->
        <bean id="javaCollection" class="com.tutorialspoint.JavaCollection">
            <property name="addressList">
                <list> <!-- 使用 list -->
                    <value>INDIA</value>
                    <value>Pakistan</value>
                    <value>USA</value>
                    <value>USA</value>
                    <!-- 如果需要使用引用类型的项，如下（其中，list/set 相同） -->
                    <!-- <ref bean="beanId"> -->
                </list>
            </property>

            <property name="addressSet">
                <set> <!-- 使用 set -->
                    <value>INDIA</value>
                    <value>Pakistan</value>
                    <value>USA</value>
                    <value>USA</value>
                </set>
            </property>

            <property name="addressMap">
                <map> <!-- 使用 map -->
                    <entry key="1" value="INDIA"/> <!-- 这里使用 entry -->
                    <entry key="2" value="Pakistan"/>
                    <entry key="3" value="USA"/>
                    <entry key="4" value="USA"/>
                    <!-- 如果需要使用引用类型的项，如下（在 value 后加 -ref） -->
                    <!-- <entry key="4" value-ref="beanId"/> -->
                </map>
            </property>

            <property name="addressProp">
                <props> <!-- 使用 props -->
                    <prop key="one">INDIA</prop> <!-- 这里使用 prop，格式与 map 的有差别 -->
                    <prop key="two">Pakistan</prop>
                    <prop key="three">USA</prop>
                    <prop key="four">USA</prop>
                    <!-- props 只有值类型，不能传递引用类型 -->
                </props>
            </property>

        </bean>
    </beans>
    ```

#### null 和空字符串类型
- null
    ``` xml
    <bean id="..." class="exampleBean">
        <property name="email"><null/></property>
    </bean>
    ```
    等同于` exampleBean.setEmail(null) `

- 空字符串
    ``` xml
    <bean id="..." class="exampleBean">
        <property name="email" value=""/>
    </bean>
    ```
    等同于` exampleBean.setEmail("") `

### 注入内部 bean
将 bean 定义放到了 property 元素里面，即该 bean 的存在只为了依赖注入，不会使用 xxx.getBean("...") 来单独使用

例子：
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <!-- Definition for textEditor bean using inner bean -->
    <bean id="textEditor" class="com.tutorialspoint.TextEditor">
        <property name="spellChecker">
            <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"/> <!-- 内部 bean -->
        </property>
    </bean>
</beans>
```

### 自动装配
当一个 bean 添加了 autowire 属性之后，该 bean 会根据指定的 autowire 属性的值自动去配置元数据（XML 文件）中寻找符合的 bean 来注入，如果找不到，将会抛出异常。

**不支持简单类型的注入，例如字符串和类，仅支持引用类型和引用类型的集合（只有 byType 和 constructor 类型才支持集合）**

autowire 的五种取值：
- no
    默认
- [byName](#byName)
    根据属性名
- [byType](#byType)
    根据属性类型
- [constructor](#constructor)
    与 byType 类似，但是适用于带参数的构造函数（如果没有带参数的构造函数将导致严重错误）
- autodetect
    Spring 首先尝试通过 constructor 使用自动装配来连接，如果它不执行，Spring 尝试通过 byType 来自动装配

#### 1. byName<a id="byName"></a>
要求：bean 的 id/name 必须和注入类的属性名字完全一致（即 bean 的 id/name 与属性名相同）

例子
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="byName"> <!-- autowire="byName" -->
        <property name="name" value="Generic Text Editor" />
    </bean>

    <bean id="spellChecker" class="com.tutorialspoint.SpellChecker"></bean>

    <!-- TextEditor 类内有 spellChecker 属性，bean 的 id 为 spellChecker。必须一样 -->

</beans>
```
#### 2. byType<a id="byType"></a>
例子
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="byType"> <!-- autowire="byType" -->
        <property name="name" value="Generic Text Editor" />
    </bean>

    <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker"></bean>

    <!-- TextEditor 类内有 SpellChecker 类型的属性，必须注册一个 class 属性为 SpellChecker 的 bean -->

</beans>
```
#### 3. constructor<a id="constructor"></a>
要求：与 byType 类似

例子
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="textEditor" class="com.tutorialspoint.TextEditor" autowire="constructor"> <!-- autowire="byType" -->
        <constructor-arg value="Generic Text Editor"/>
    </bean>

    <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker"></bean>

</beans>
```

### 使用注解来配置依赖注入
从 Spring 2.5 开始就可以使用注解来配置依赖注入。

但不可以完全脱离 XML 配置文件，至少还需要在配置文件中启用注解，如下
``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/> <!-- 启用注解配置 -->
</beans>
```

几个常用的注解：
- @Required
    @Required 注释应用于 bean 属性的 setter 方法，它表明受影响的 bean 属性在配置时必须放在 XML 配置文件中，否则容器就会抛出一个 BeanInitializationException 异常。

    例子
    ``` java
    // Student.java

    package com.tutorialspoint;
    import org.springframework.beans.factory.annotation.Required;

    public class Student {
        private Integer age;
        private String name;

        @Required
        public void setAge(Integer age) {
            this.age = age;
        }
        public Integer getAge() {
            return age;
        }

        @Required
        public void setName(String name) {
            this.name = name;
        }
        public String getName() {
            return name;
        }
    }
    ```

    ``` xml
    <!-- beans.xml -->

    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:annotation-config/>

    <!-- Definition for student bean -->
    <bean id="student" class="com.tutorialspoint.Student">
        <property name="name"  value="Zara" /> <!-- 必须指明，否则出错 -->
        <property name="age"  value="11"/> <!-- 必须指明，否则出错 -->
    </bean>

    </beans>
    ```

- @Autowired
    可以出现在多个位置，不同位置有不同的作用。
    > 貌似只能给引用类型使用

    - Setter 方法中的 @Autowired
        设置了之后会执行 byType 类型的自动装配。
    - 属性中的 @Autowired
        在类内属性上添加了该注解之后，可以不写该属性的 setter 方法
    - 构造函数中的 @Autowired

    还可以使用` @Autowired(required=false) `来关闭强制注入依赖

- @Qualifier
    使用场景：在对同一个类注册了多个 bean 的情况下选择某一个 bean 来注入
    
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

        <context:annotation-config/>

        <!-- Definition for profile bean -->
        <bean id="profile" class="com.tutorialspoint.Profile"></bean>

        <!-- Definition for student1 bean -->
        <bean id="student1" class="com.tutorialspoint.Student">
            <property name="name"  value="Zara" />
            <property name="age"  value="11"/>
        </bean>

        <!-- Definition for student2 bean -->
        <bean id="student2" class="com.tutorialspoint.Student">
            <property name="name"  value="Nuha" />
            <property name="age"  value="2"/>
        </bean>
    </beans>
    ```

    ``` java
    package com.tutorialspoint;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;

    public class Profile {
    @Autowired
    @Qualifier("student1") // 指明具体注入哪个 bean
    private Student student;
    public Profile(){
        System.out.println("Inside Profile constructor." );
    }
    public void printAge() {
        System.out.println("Age : " + student.getAge() );
    }
    public void printName() {
        System.out.println("Name : " + student.getName() );
    }
    }
    ```

- @Resource
    可以在字段中或者 setter 方法中使用 @Resource 注释，会执行 byName 类型的自动装配

    ``` java
    package com.tutorialspoint;
    import javax.annotation.Resource;

    public class TextEditor {
    private SpellChecker spellChecker;
    @Resource(name= "spellChecker") // 注意，可以不指明 name
    public void setSpellChecker( SpellChecker spellChecker ){
        this.spellChecker = spellChecker;
    }
    public SpellChecker getSpellChecker(){
        return spellChecker;
    }
    public void spellCheck(){
        spellChecker.checkSpelling();
    }
    }
    ```

> 更多注解可以参考：JSR-250

### 基于 java 的配置
就是另外使用一个 java 类来代替 XML 配置文件，这个 java 类仅仅用来做配置，不包含实际意义。

#### 简单例子
例如：
``` java
package com.tutorialspoint;

import org.springframework.context.annotation.*;
@Configuration // 该注解说明该类是配置元数据
public class HelloWorldConfig {
    @Bean // 该注解说明注册一个 id/name 为 helloWorld，class 为 HelloWorld
    public HelloWorld helloWorld(){
        return new HelloWorld();
    }
}
```

等同于
``` xml
<beans>
    <bean id="helloWorld" class="com.tutorialspoint.HelloWorld" />
</beans>
```

使用
``` java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class); // 注意，这里使用的是 AnnotationConfigApplicationContext
    HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
}
```

#### 较为复杂的例子
- 依赖其他 bean 的 bean
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.annotation.*;
    @Configuration
    public class TextEditorConfig {
        @Bean
        public TextEditor textEditor(){
            return new TextEditor( spellChecker() ); // 依赖于 spellChecker 的 bean，注意这里参数的写法
        }
        @Bean
        public SpellChecker spellChecker(){
            return new SpellChecker( );
        }
    }
    ```
- 初始化方法和销毁方法
    ``` java
    // Foo.java
    public class Foo {
        public void init() {
            // initialization logic
        }
        public void cleanup() {
            // destruction logic
        }
    }

    // AppConfig.java
    @Configuration
    public class AppConfig {
        @Bean(initMethod = "init", destroyMethod = "cleanup" ) // 在这里指明
        public Foo foo() {
            return new Foo();
        }
    }
    ```

可以使用 @Import 注解引用其他的配置类，然后可以在，例如：
``` java
// ConfigA.java
@Configuration
public class ConfigA {
    @Bean
    public A a() {
        return new A(); 
    }
}

// ConfigB.java
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B b() {
      return new B(); 
   }
}

// 使用
public static void main(String[] args) {
   ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
   A a = ctx.getBean(A.class); // 因为 ConfigB.java 导入了 ConfigA.java，所以可以直接用
   B b = ctx.getBean(B.class);
}
```

### Spring 事件处理
> 需要注意，由于 Spring 的事件处理是单线程的，所以如果一个事件被发布，直至并且除非所有的接收者得到的该消息，该进程被阻塞并且流程将不会继续。

#### 标准事件：
- ContextRefreshedEvent
    ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生。

- ContextStartedEvent
    当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接收到这个事件后重启任何停止的应用程序。

- ContextStoppedEvent
    当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接收到这个事件后做必要的清理的工作。

- ContextClosedEvent
    当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。

- RequestHandledEvent
    这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。

使用：
``` java
// 事件处理类1：CStartEventHandler.java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStartedEvent;
public class CStartEventHandler implements ApplicationListener<ContextStartedEvent>{ // 实现接口 ApplicationListener，而且类型为 ContextStartedEvent 类
    public void onApplicationEvent(ContextStartedEvent event) {
        System.out.println("ContextStartedEvent Received");
    }
}

// 事件处理类2：CStopEventHandler.java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStoppedEvent;
public class CStopEventHandler implements ApplicationListener<ContextStoppedEvent>{ // 实现接口 ApplicationListener，而且类型为 ContextStoppedEvent 类
    public void onApplicationEvent(ContextStoppedEvent event) {
        System.out.println("ContextStoppedEvent Received");
    }
}

// MainApp.java
package com.tutorialspoint;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml"); // 注意这里使用的是 ConfigurableApplicationContext 类

      // Let us raise a start event.
      context.start(); // 注意

      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");

      obj.getMessage();

      // Let us raise a stop event.
      context.stop(); // 注意
   }
}
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
        <property name="message" value="Hello World!"/>
    </bean>

    <!-- 需要注册这两个事件处理类 -->
    <bean id="cStartEventHandler" class="com.tutorialspoint.CStartEventHandler"/>
    <bean id="cStopEventHandler" class="com.tutorialspoint.CStopEventHandler"/>
</beans>
```

#### 自定义事件
步骤：
1. 编写事件类
2. 编写事件发布类
3. 编写事件处理类
4. 在配置元数据中配置（只需注册事件处理类和事件发布类）

具体步骤：
- 事件类：CustomEvent.java
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.ApplicationEvent;
    // 需要实现 ApplicationEvent 接口
    public class CustomEvent extends ApplicationEvent{ 
        public CustomEvent(Object source) {
            super(source);
        }
        public String toString(){
            return "My Custom Event";
        }
    }
    ```
- 事件发布类：CustomEventPublisher.java
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.ApplicationEventPublisher;
    import org.springframework.context.ApplicationEventPublisherAware;
    // 需要实现 ApplicationEventPublisherAware 接口
    public class CustomEventPublisher implements ApplicationEventPublisherAware {
        private ApplicationEventPublisher publisher;
        public void setApplicationEventPublisher
                    (ApplicationEventPublisher publisher){
            this.publisher = publisher;
        }
        public void publish() {
            CustomEvent ce = new CustomEvent(this);
            publisher.publishEvent(ce);
        }
    }
    ```
- 事件处理类：CustomEventHandler.java
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.ApplicationListener;
    // 需要实现 ApplicationListener 接口，且使用 CustomEvent 事件类
    public class CustomEventHandler implements ApplicationListener<CustomEvent>{
        public void onApplicationEvent(CustomEvent event) {
            System.out.println(event.toString()); // 处理：输出事件类
        }
    }
    ```
- beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>

    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

        <!-- 需要注册事件发布类和事件处理类 -->
        <bean id="customEventHandler" class="com.tutorialspoint.CustomEventHandler"/>
        <bean id="customEventPublisher" class="com.tutorialspoint.CustomEventPublisher"/>
    </beans>
    ```
- MainApp.java
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.ConfigurableApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    public class MainApp {
        public static void main(String[] args) {
            ConfigurableApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");    
            CustomEventPublisher cvp = (CustomEventPublisher) context.getBean("customEventPublisher");
            cvp.publish(); // 输出 My Custom Event
            cvp.publish(); // 输出 My Custom Event
        }
    }
    ```

---

## AOP(Aspect-Oriented Programming) 面向方面编程
### aop作用
把功能分为核心业务功能和周边功能，分别开发，最后把周边功能和核心业务功能 "编织" 在一起，这就叫 AOP

### 概念
> 前三个比较重要
>
> [参考1](https://blog.csdn.net/anurnomeru/article/details/79798659)
- Aspect
    一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。

    > **至少包括一个 [pointcut](#pointcut) 和 [advice](#advice)，用来表示整个 Aspect**
- Pointcut<a id="pointcut"></a>
	这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。

    > 指明在哪个/哪几个方法被调用的时候发送通知给 AOP（通知包括 1. 调用什么方法 2. 什么时候调用（5种之一））（@未证实，暂时这样理解）
- Advice<a id="advice"></a>
	这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。

    > 某个方法，AOP 在收到特定通知的时候会去自动执行该方法
    - 有 5 种调用方式：
        - before
        - after
        - after-returning
        - after-throwing
        - around
        > 具体见 [使用](#使用（两种方式）) 的 1.普通方式（XML 配置）-Beans.xml 和 2. 注解方式-Logging.java
- Join point
    在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。

    > 在使用` around `方式的 [Advice](#advice) 时使用，见 [这里](#loggingjava) 的 log 方法
- Introduction
	引用允许你添加新方法或属性到现有的类中。
- Target object
    被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。
- Weaving
    Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时，类加载时和运行时完成。

### 使用（两种方式）
#### 1. 普通方式（XML 配置）
- Student.java
    ``` java
    package com.tutorialspoint;

    public class Student {
        private Integer age;
        private String name;
        public void setAge(Integer age) {
            this.age = age;
        }
        public Integer getAge() {
            System.out.println("Age : " + age );
            return age;
        }
        public void setName(String name) {
            this.name = name;
        }
        public String getName() {
            System.out.println("Name : " + name );
            return name;
        }  
        public void printThrowException(){
            System.out.println("Exception raised");
            throw new IllegalArgumentException();
        }
    }
    ```

- Logging.java<a id="loggingjava"></a>
    ``` java
    package com.tutorialspoint;

    public class Logging {
        public void beforeAdvice(){
            System.out.println("Going to setup student profile.");
        }

        public void afterAdvice(){
            System.out.println("Student profile has been setup.");
        }

        public void afterReturningAdvice(Object retVal){
            System.out.println("Returning:" + retVal.toString() );
        }

        public void AfterThrowingAdvice(IllegalArgumentException ex){
            System.out.println("There has been an exception: " + ex.toString());   
        }

        public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("start log:" + joinPoint.getSignature().getName());
            Object object = joinPoint.proceed(); // 注意：joinPoint。上面的代码在执行 pointCut 之前执行，下面的代码在执行 pointCut 之后执行
            System.out.println("end log:" + joinPoint.getSignature().getName());
            return object;
        }
    }
    ```

- Beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">
        <!-- 因为使用 aop，所以添加了三个与 aop 相关的内容 -->

        <!-- Definition for student bean -->
        <bean id="student" class="com.tutorialspoint.Student">
            <property name="name"  value="Zara" />
            <property name="age"  value="11" />      
        </bean>

        <!-- Definition for logging aspect -->
        <bean id="logging" class="com.tutorialspoint.Logging"/> 
        
        <!-- 主要配置 -->
        <aop:config>
            <!-- 声明 aspect -->
            <aop:aspect id="log" ref="logging">
                <!-- 声明 pointcut -->
                <aop:pointcut id="pointCutId" expression="execution(* com.tutorialspoint.*.*(..))" /> <!-- 这里的 * 号代替具体的类或者方法名，execution 后面的第一个 * 指返回值 -->

                <!-- 声明 advice -->
                <aop:before pointcut-ref="pointCutId" method="beforeAdvice"/> <!-- 在 pointCut 之前执行 -->
                <aop:after pointcut-ref="pointCutId" method="afterAdvice"/>  <!-- 在 pointCut 之后执行 -->
                <aop:after-returning pointcut-ref="pointCutId" returning="retVal" method="afterReturningAdvice"/>  <!-- 在 pointCut 返回时执行（要求该 pointCut 有返回类型） -->
                <aop:after-throwing pointcut-ref="pointCutId" throwing="ex" method="AfterThrowingAdvice"/> <!-- 在 pointCut 抛出异常时执行 -->
                <aop:around pointcut-ref="pointCutId" method="log"/> <!-- 环绕 pointCut 执行，类似即在之前又在之后，前后执行的内容都在一个函数里，需要使用 joinpoint 划分前后执行的内容 -->
            </aop:aspect>
        </aop:config>
    </beans>
    ```

- MainApp.java
    ``` java
    package com.tutorialspoint;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;

    public class MainApp {
        public static void main(String[] args) {
            ApplicationContext context = 
                    new ClassPathXmlApplicationContext("Beans.xml");
            Student student = (Student) context.getBean("student");
            student.getName();
            student.getAge();      
            student.printThrowException();

            // 输出如下：
            // Going to setup student profile.  // beforeAdvice()
            // Name : Zara                      // getName()
            // Student profile has been setup.  // afterAdvice()
            // Returning:Zara                   // afterReturningAdvice(Object retVal)

            // Going to setup student profile.  // beforeAdvice()
            // Age : 11                         // getAge()
            // Student profile has been setup.  // afterAdvice()
            // Returning:11                     // afterReturningAdvice(Object retVal)

            // Going to setup student profile.  // beforeAdvice()
            // Exception raised                 // printThrowException()
            // Student profile has been setup.  // afterAdvice()
            // There has been an exception: java.lang.IllegalArgumentException  // AfterThrowingAdvice(IllegalArgumentException ex)

            // .....
            // other exception content
        }
    }
    ```

#### 2. 注解方式
- Student.java
    与 1. 普通方式（XML 配置）的 Student.java 一样

- Logging.java
    ``` java
    package com.tutorialspoint;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Pointcut;

    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.annotation.After;
    import org.aspectj.lang.annotation.AfterThrowing;
    import org.aspectj.lang.annotation.AfterReturning;
    import org.aspectj.lang.annotation.Around;

    @Aspect // 指明为 Aspect 类
    public class Logging {
        @Pointcut("execution(* com.tutorialspoint.*.*(..))") // 指明 pointcut
        private void selectAll(){} // 这个只是一个名字，与切面类的实际无关，只是为了方便下面使用（类似 XML 中 aop:pointcut 的 id）

        @Before("selectAll()") // 使用 pointcut 的名字
        public void beforeAdvice(){
            System.out.println("Going to setup student profile.");
        }

        @After("selectAll()")
        public void afterAdvice(){
            System.out.println("Student profile has been setup.");
        }

        // 当方法在返回的时候执行（方法要有返回类型）
        @AfterReturning(pointcut = "selectAll()", returning="retVal")
        public void afterReturningAdvice(Object retVal){
            System.out.println("Returning:" + retVal.toString() );
        }

        // 当方法抛出异常时执行
        @AfterThrowing(pointcut = "selectAll()", throwing = "ex")
        public void AfterThrowingAdvice(IllegalArgumentException ex){
            System.out.println("There has been an exception: " + ex.toString());   
        }  
    }
    ```

- Beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">

        <bean id="student" class="com.tutorialspoint.Student">
            <property name="name"  value="Zara" />
            <property name="age"  value="11"/>      
        </bean>

        <bean id="logging" class="com.tutorialspoint.Logging"/> 

        <aop:aspectj-autoproxy/> <!-- 注意 -->
    </beans>
    ```

- MainApp.java
    与 1. 普通方式（XML 配置）的 MainApp.java 一样

## JDBC(Java Database Connectivity) Java数据库连接
### 涉及的包
- org.springframework.jdbc.jar
- org.springframework.transaction.jar
    > 与事务执行有关
- mysql-connector-java.jar
    > 根据具体的数据库按需使用，这里使用的是 mysql

### 两个概念
#### JdbcTemplate 类
JdbcTemplate 类执行 SQL 查询、更新语句和存储过程调用，执行迭代结果集和提取返回参数值。它也捕获 JDBC 异常并转换它们到 org.springframework.dao 包中定义的通用类、更多的信息、异常层次结构。

JdbcTemplate 类的实例是线程安全配置的。所以你可以配置 JdbcTemplate 的单个实例，然后将这个共享的引用安全地注入到多个 DAOs 中。

使用 JdbcTemplate 类时常见的做法是在你的 Spring 配置文件中配置数据源，然后共享数据源 bean 依赖注入到 DAO 类中，并在数据源的设值函数中创建了JdbcTemplate。
> 其他类似的类：NamedParameterJdbcTemplate, SimpleJdbcTemplate（@todo，查看是什么）

SQL 语句执行举例：
- 查
    - 查询一个整数类型` queryForInt `
        ``` java
        String SQL = "select count(*) from Student";
        int rowCount = jdbcTemplateObject.queryForInt(SQL);
        ```
    - 查询一个 long 类型` queryForLong `
        ``` java
        String SQL = "select count(*) from Student";
        long rowCount = jdbcTemplateObject.queryForLong(SQL);
        ```
    - 一个使用绑定变量的简单查询` queryForInt `
        ``` java
        String SQL = "select age from Student where id = ?";
        int age = jdbcTemplateObject.queryForInt(SQL, new Object[]{10});
        ```
    - 查询字符串` queryForObject `
        ``` java
        String SQL = "select name from Student where id = ?";
        String name = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, String.class); // 第三个参数指明 SQL 的返回类型
        ```
    - 查询并返回一个对象` queryForObject `<a id="RowMapper"></a>
        ``` java
        String SQL = "select * from Student where id = ?";
        Student student = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, new StudentMapper()); // 第三个参数指明 SQL 的返回类型，用了一个内部类来映射结果
        public class StudentMapper implements RowMapper<Student> {
            public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
                Student student = new Student();
                student.setID(rs.getInt("id"));
                student.setName(rs.getString("name"));
                student.setAge(rs.getInt("age"));
                return student;
            }
        }
        ```
    - 查询并返回多个对象` query `
        ``` java
        String SQL = "select * from Student";
        List<Student> students = jdbcTemplateObject.query(SQL, new StudentMapper()); // 第三个参数指明 SQL 的返回类型，用了一个内部类来映射结果
        public class StudentMapper implements RowMapper<Student> {
            // ...
            // 和上面一样
        }
        ```
- 增、删、改
    - 在表中插入一行` update `
        ``` java
        String SQL = "insert into Student (name, age) values (?, ?)";
        jdbcTemplateObject.update(SQL, new Object[]{"Zara", 11} );
        ```
    - 从表中删除一行` update `
        ``` java
        String SQL = "delete Student where id = ?";
        jdbcTemplateObject.update(SQL, new Object[]{20});
        ```
    - 更新表中的一行` update `
        ``` java
        String SQL = "update Student set name = ? where id = ?";
        jdbcTemplateObject.update(SQL, new Object[]{"Zara", 10} );
        ```
- 执行 DDL 语句` execute `
    ``` java
    String SQL = "CREATE TABLE Student( " +
        "ID   INT NOT NULL AUTO_INCREMENT, " +
        "NAME VARCHAR(20) NOT NULL, " +
        "AGE  INT NOT NULL, " +
        "PRIMARY KEY (ID));"
    jdbcTemplateObject.execute(SQL);
    ```
> **总结：**
> 1. DML 的 “增、删、改” 全部使用` update `方法，不同在于 SQL 语句中
>
> 2. SQL 中的缺省值使用占位符` ? `表示，可以使用两种方式给出具体值：1. 在增删改查方法中的第二个参数使用` Object[] `来给出（` Object[] `的数组长度等于缺省值个数）；2. 直接传参，从第二个参数开始
>
> 3. DML 的 “查” 可以使用` queryForInt `, ` queryForLong `, ` queryForObject `, ` query `方法来达到目的，除了前两个给出了明确的返回类型，后两个都需要在方法的第三个参数指明返回的类型，可能需要使用实现` RowMapper `接口的类来映射结果，见 [这里](#RowMapper) 的例子，具体的数据库类型和 Java 类型转换要参考文档
#### 数据访问对象（DAO）
> 非必须，一般声明为接口
DAO 代表常用的数据库交互的数据访问对象。DAOs 提供一种方法来读取数据并将数据写入到数据库中，它们应该通过一个接口显示此功能，应用程序的其余部分将访问它们。

### 使用
- script.sql
    > 数据库脚本
    ``` sql
    CREATE TABLE Student(
    ID   INT NOT NULL AUTO_INCREMENT,
    NAME VARCHAR(20) NOT NULL,
    AGE  INT NOT NULL,
    PRIMARY KEY (ID)
    );

    <!-- 存储过程，TEST 为表名，getRecord 为存储过程名 -->
    DELIMITER $$
    DROP PROCEDURE IF EXISTS `TEST`.`getRecord` $$
    CREATE PROCEDURE `TEST`.`getRecord` (
    IN in_id INTEGER,
    OUT out_name VARCHAR(20),
    OUT out_age  INTEGER)
    BEGIN
    SELECT name, age
    INTO out_name, out_age
    FROM Student where id = in_id;
    END $$
    DELIMITER ;
    ```
- Student.java
    > 实体类
    ``` java
    package com.tutorialspoint;

    public class Student {
        private Integer age;
        private String name;
        private Integer id;

        // getter & setter
    }
    ```
- StudentDAO.java
    > DAO 类
    > 使用 DAO 接口的意义是可以方便切换各种数据库（datasource）
    >
    > 在 Spring 中，数据访问对象(DAO)支持很容易用统一的方法使用数据访问技术，如 JDBC、Hibernate、JPA 或者 JDO
    ``` java
    package com.tutorialspoint;
    import java.util.List;
    import javax.sql.DataSource;

    public interface StudentDAO { // 接口
        // This is the method to be used to initialize database resources ie. connection.
        public void setDataSource(DataSource ds); // 初始化数据库资源，一般由配置文件来初始化

        // 插入一条记录
        public void create(String name, Integer age);

        // 根据 id 获取一条记录
        public Student getStudent(Integer id);

        // 获取所有记录
        public List<Student> listStudents();

        // 根据 id 删除一条记录
        public void delete(Integer id);

        // 根据 id 更新一条记录
        public void update(Integer id, Integer age);
    }
    ```
- StudentMapper.java
    > 查询结果映射类
    > 用于 queryForObject, query 方法结果的映射
    ``` java
    package com.tutorialspoint;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import org.springframework.jdbc.core.RowMapper;

    public class StudentMapper implements RowMapper<Student> {
        public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
            Student student = new Student();
            student.setId(rs.getInt("id"));
            student.setName(rs.getString("name"));
            student.setAge(rs.getInt("age"));
            return student;
        }
    }
    ```
- StudentJDBCTemplate.java
    ``` java
    package com.tutorialspoint;
    import java.util.List;
    import javax.sql.DataSource;
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;  // 用于执行存储过程
    import org.springframework.jdbc.core.namedparam.SqlParameterSource;     // 用于执行存储过程
    import org.springframework.jdbc.core.simple.SimpleJdbcCall;             // 用于执行存储过程

    public class StudentJDBCTemplate implements StudentDAO { // 实现 StudentDAO 接口
        private DataSource dataSource;
        private JdbcTemplate jdbcTemplateObject; // 用于执行增删改查 SQL
        private SimpleJdbcCall jdbcCall; // 用于执行存储过程

        
        public void setDataSource(DataSource dataSource) { // 由配置文件 Beans.xml 来初始化
            this.dataSource = dataSource;
            this.jdbcTemplateObject = new JdbcTemplate(dataSource);
            this.jdbcCall =  new SimpleJdbcCall(dataSource).withProcedureName("getRecord"); // 指明存储过程的名字
        }

        public void create(String name, Integer age) {
            String SQL = "insert into Student (name, age) values (?, ?)";
            jdbcTemplateObject.update(SQL, name, age);
            return;
        }

        public Student getStudent(Integer id) {
            String SQL = "select * from Student where id = ?";
            Student student = jdbcTemplateObject.queryForObject(SQL, new Object[]{id}, new StudentMapper()); // 注意第三个参数 StudentMapper 类
            return student;
        }

        // 执行存储过程
        public Student getStudentFromProc(Integer id) {
            SqlParameterSource in = new MapSqlParameterSource().addValue("in_id", id); // in_id 为存储过程的入参
            Map<String, Object> out = jdbcCall.execute(in); // 使用 execute 方法

            Student student = new Student();
            student.setId(id);
            student.setName((String) out.get("out_name"));  // out_name 为存储过程的出参，注意类型转换
            student.setAge((Integer) out.get("out_age"));   // out_age  为存储过程的出参，注意类型转换

            return student;
        }

        public List<Student> listStudents() {
            String SQL = "select * from Student";
            List <Student> students = jdbcTemplateObject.query(SQL, new StudentMapper());
            return students;
        }

        public void delete(Integer id){
            String SQL = "delete from Student where id = ?";
            jdbcTemplateObject.update(SQL, id);
            return;
        }
        
        public void update(Integer id, Integer age){
            String SQL = "update Student set age = ? where id = ?";
            jdbcTemplateObject.update(SQL, age, id);
            return;
        }
    }
    ```
- Beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd ">

        <!-- Initialization for data source -->
        <bean id="dataSourceBean" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/TEST"/>
            <property name="username" value="root"/>
            <property name="password" value="password"/>
        </bean>

        <!-- Definition for studentJDBCTemplate bean -->
        <bean id="studentJDBCTemplate" class="com.tutorialspoint.StudentJDBCTemplate">
            <property name="dataSource"  ref="dataSourceBean" /> <!-- 注意：在这里初始化类内 dataSource 属性 -->
        </bean>

    </beans>
    ```
- MainApp.java
    ``` java
    package com.tutorialspoint;

    import java.util.List;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import com.tutorialspoint.StudentJDBCTemplate;

    public class MainApp {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
            StudentJDBCTemplate studentJDBCTemplate = (StudentJDBCTemplate)context.getBean("studentJDBCTemplate");    

            System.out.println("------Records Creation--------" );
            studentJDBCTemplate.create("Zara", 11);
            studentJDBCTemplate.create("Nuha", 2);
            studentJDBCTemplate.create("Ayan", 15);

            System.out.println("------Listing Multiple Records--------" );
            List<Student> students = studentJDBCTemplate.listStudents();
            for (Student record : students) {
                System.out.print("ID : " + record.getId() );
                System.out.print(", Name : " + record.getName() );
                System.out.println(", Age : " + record.getAge());
            }

            System.out.println("----Updating Record with ID = 2 -----" );
            studentJDBCTemplate.update(2, 20);
            
            System.out.println("----Listing Record with ID = 2 -----" );
            Student student = studentJDBCTemplate.getStudent(2);
            System.out.print("ID : " + student.getId() );
            System.out.print(", Name : " + student.getName() );
            System.out.println(", Age : " + student.getAge());      
        }
    }
    ```

---

## Transactions 事务管理
### 涉及的包
- org.springframework.jdbc.jar
- org.springframework.transaction.jar
    > 与事务执行有关
- mysql-connector-java.jar
    > 根据具体的数据库按需使用，这里使用的是 mysql

### 事务分类（两种类型）
> 局部事务特定于一个单一的事务资源，如一个 JDBC 连接；全局事务可以跨多个事务资源事务，如在一个分布式系统中的事务。
- 局部事务

    局部事务管理在一个集中的计算环境中是有用的，该计算环境中应用程序组件和资源位于一个单位点，而事务管理只涉及到一个运行在一个单一机器中的本地数据管理器。
    
    局部事务更容易实现。
- 全局事务

    全局事务管理需要在分布式计算环境中，所有的资源都分布在多个系统中。在这种情况下事务管理需要同时在局部和全局范围内进行。
    
    分布式或全局事务跨多个系统执行，它的执行需要全局事务管理系统和所有相关系统的局部数据管理人员之间的协调。

### 相关接口（三个）
#### 1. PlatformTransactionManager
``` java
import org.springframework.transaction;

public interface PlatformTransactionManager {
    // 根据指定的传播行为，下面方法返回当前活动事务或创建一个新的事务
    TransactionStatus getTransaction(TransactionDefinition definition);
    throws TransactionException;
    // 下面方法提交给定的事务和关于它的状态
    void commit(TransactionStatus status) throws TransactionException;
    // 下面方法执行一个给定事务的回滚
    void rollback(TransactionStatus status) throws TransactionException;
}
```

#### 2. TransactionDefinition
``` java
public interface TransactionDefinition {
    // 下面方法返回传播行为。Spring 提供了与 EJB CMT 类似的所有的事务传播选项
    int getPropagationBehavior();
    // 下面方法返回该事务独立于其他事务的工作的程度
    int getIsolationLevel();
    // 下面方法返回该事务的名称
    String getName();
    // 下面方法返回以秒为单位的时间间隔，事务必须在该时间间隔内完成
    int getTimeout();
    // 下面方法返回该事务是否是只读的
    boolean isReadOnly();
}
```
- PropagationBehavior “传播行为” 取值
    1. ` TransactionDefinition.PROPAGATION_MANDATORY `
        支持当前事务；如果不存在当前事务，则抛出一个异常
    2. ` TransactionDefinition.PROPAGATION_NESTED `
        如果存在当前事务，则在一个嵌套的事务中执行
    3. ` TransactionDefinition.PROPAGATION_NEVER `
        不支持当前事务；如果存在当前事务，则抛出一个异常
    4. ` TransactionDefinition.PROPAGATION_NOT_SUPPORTED `
        不支持当前事务；而总是执行非事务性
    5. ` TransactionDefinition.PROPAGATION_REQUIRED `
        支持当前事务；如果不存在事务，则创建一个新的事务
    6. ` TransactionDefinition.PROPAGATION_REQUIRES_NEW `
        创建一个新事务，如果存在一个事务，则把当前事务挂起
    7. ` TransactionDefinition.PROPAGATION_SUPPORTS `
        支持当前事务；如果不存在，则执行非事务性
    8. ` TransactionDefinition.TIMEOUT_DEFAULT `
        使用默认超时的底层事务系统，或者如果不支持超时则没有

- IsolationLevel “隔离级别” 取值
    1. ` TransactionDefinition.ISOLATION_DEFAULT `
        默认的隔离级别
    2. ` TransactionDefinition.ISOLATION_READ_COMMITTED `
        表明能够阻止误读；可以发生不可重复读和虚读
    3. ` TransactionDefinition.ISOLATION_READ_UNCOMMITTED `
        表明可以发生误读、不可重复读和虚读
    4. ` TransactionDefinition.ISOLATION_REPEATABLE_READ `
        表明能够阻止误读和不可重复读；可以发生虚读
    5. ` TransactionDefinition.ISOLATION_SERIALIZABLE `
        表明能够阻止误读、不可重复读和虚读

#### 3. TransactionStatus
``` java
public interface TransactionStatus extends SavepointManager {
    // 下面方法返回该事务内部是否有一个保存点，也就是说，基于一个保存点已经创建了嵌套事务。
    boolean isNewTransaction();
    // 下面方法返回该事务是否完成，也就是说，它是否已经提交或回滚。
    boolean hasSavepoint();
    // 下面当前事务时新的情况下，该方法返回 true。
    void setRollbackOnly();
    // 下面方法返回该事务是否已标记为 rollback-only。
    boolean isRollbackOnly();
    // 下面方法设置该事务为 rollback-only 标记。
    boolean isCompleted();
}
```

### 使用（两种方式）
> 声明式事务管理比编程式事务管理更可取，尽管它不如编程式事务管理灵活，但它允许你通过代码控制事务。但作为一种横切关注点，声明式事务管理可以使用 AOP 方法进行模块化。Spring 支持使用 Spring AOP 框架的声明式事务管理。

#### 1. 编程式事务管理
> “编程式事务管理” 意味着你在编程的帮助下有管理事务。这给了你极大的灵活性，但却很难维护

基本步骤：
0. 定义一个` PlatformTransactionManager `的实例
1. 定义一个` TransactionDefinition `的实例（使用适当 transaction 属性）
2. 使用` PlatformTransactionManager实例.getTransaction(TransactionDefinition实例) `开始事务（使用` TransactionStatus `的一个实例来接收该方法的返回）
3. 执行 SQL 操作
4. 使用` PlatformTransactionManager实例.commit(TransactionStatus实例) `提交事务
5. 如果出错，使用` PlatformTransactionManager实例.rollback(TransactionStatus实例) `回滚事务

编码：
- script.sql
    > 数据库脚本
    ``` sql
    -- Student 表
    CREATE TABLE Student(
        ID   INT NOT NULL AUTO_INCREMENT,
        NAME VARCHAR(20) NOT NULL,
        AGE  INT NOT NULL,
        PRIMARY KEY (ID)
    );
    -- Marks 表
    CREATE TABLE Marks(
        SID INT NOT NULL,
        MARKS  INT NOT NULL,
        YEAR   INT NOT NULL
    );
    ```
- StudentMarks.java
    > 实体类
    ``` java
    package com.tutorialspoint;

    public class StudentMarks {
        private Integer age;
        private String name;
        private Integer id;
        private Integer marks;
        private Integer year;
        private Integer sid;

        // getter & setter
    }
    ```
- StudentDAO.java
    > DAO 类
    ``` java
    package com.tutorialspoint;

    import java.util.List;
    import javax.sql.DataSource;

    public interface StudentDAO {
        public void setDataSource(DataSource ds);
        
        // 在 Student, Marks 表中插入数据
        public void create(String name, Integer age, Integer marks, Integer year);

        public List<StudentMarks> listStudents();
    }
    ```
- StudentMarksMapper.java
    > 结果映射类
    ``` java
    package com.tutorialspoint;

    import java.sql.ResultSet;
    import java.sql.SQLException;
    import org.springframework.jdbc.core.RowMapper;

    public class StudentMarksMapper implements RowMapper<StudentMarks> { // 实现 RowMapper 接口
        public StudentMarks mapRow(ResultSet rs, int rowNum) throws SQLException {
            StudentMarks studentMarks = new StudentMarks();
            studentMarks.setId(rs.getInt("id"));
            studentMarks.setName(rs.getString("name"));
            studentMarks.setAge(rs.getInt("age"));
            studentMarks.setSid(rs.getInt("sid"));
            studentMarks.setMarks(rs.getInt("marks"));
            studentMarks.setYear(rs.getInt("year"));
            return studentMarks;
        }
    }
    ```
- StudentJDBCTemplate.java
    > DAO 实现类
    ``` java
    package com.tutorialspoint;

    import java.util.List;
    import javax.sql.DataSource;
    import org.springframework.dao.DataAccessException;
    import org.springframework.jdbc.core.JdbcTemplate;
    import org.springframework.transaction.PlatformTransactionManager;          // 用于执行事务
    import org.springframework.transaction.TransactionDefinition;               // 用于执行事务
    import org.springframework.transaction.TransactionStatus;                   // 用于执行事务
    import org.springframework.transaction.support.DefaultTransactionDefinition;// 用于执行事务

    public class StudentJDBCTemplate implements StudentDAO {
        private DataSource dataSource;
        private JdbcTemplate jdbcTemplateObject;
        private PlatformTransactionManager transactionManager; // 注意

        public void setDataSource(DataSource dataSource) { // 数据源初始化，由配置文件初始化
            this.dataSource = dataSource;
            this.jdbcTemplateObject = new JdbcTemplate(dataSource);
        }
        public void setTransactionManager(PlatformTransactionManager transactionManager) { // 事务管理器初始化，由配置文件初始化
            this.transactionManager = transactionManager;
        }

        public void create(String name, Integer age, Integer marks, Integer year){
            TransactionDefinition def = new DefaultTransactionDefinition(); // 定义一个 TransactionDefinition 实例
            TransactionStatus status = transactionManager.getTransaction(def); // 开始事务 PlatformTransactionManager.getTransaction() （定义一个 TransactionStatus 实例用来接收事务状态）
            try {
                String SQL1 = "insert into Student (name, age) values (?, ?)";
                jdbcTemplateObject.update(SQL1, name, age);
                String SQL2 = "select max(id) from Student";
                int sid = jdbcTemplateObject.queryForInt(SQL2);
                String SQL3 = "insert into Marks(sid, marks, year) values (?, ?, ?)";
                jdbcTemplateObject.update( SQL3, sid, marks, year);

                transactionManager.commit(status); // 提交事务 PlatformTransactionManager.commit() 
            } catch (DataAccessException e) {
                System.out.println("Error in creating record, rolling back");
                transactionManager.rollback(status); // 回滚事务 PlatformTransactionManager.rollback() 
                throw e;
            }
            return;
        }

        public List<StudentMarks> listStudents() {
            String SQL = "select * from Student, Marks where Student.id=Marks.sid";
            List <StudentMarks> studentMarks = jdbcTemplateObject.query(SQL, new StudentMarksMapper());
            return studentMarks;
        }
    }
    ```
- Beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd ">

        <!-- Initialization for data source -->
        <bean id="dataSourceBean" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/TEST"/>
            <property name="username" value="root"/>
            <property name="password" value="password"/>
        </bean>

        <!-- Initialization for TransactionManager -->
        <bean id="transactionManagerBean" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource"  ref="dataSourceBean" />    
        </bean>

        <!-- Definition for studentJDBCTemplate bean -->
        <bean id="studentJDBCTemplateBean" class="com.tutorialspoint.StudentJDBCTemplate">
            <property name="dataSource"  ref="dataSourceBean" />
            <property name="transactionManager"  ref="transactionManagerBean" />
        </bean>
    </beans>
    ```
- MainApp.java
    ``` java
    package com.tutorialspoint;

    import java.util.List;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import com.tutorialspoint.StudentJDBCTemplate;

    public class MainApp {
        public static void main(String[] args) {
            ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
            StudentJDBCTemplate studentJDBCTemplate = (StudentJDBCTemplate)context.getBean("studentJDBCTemplate");

            System.out.println("------Records creation--------" );
            studentJDBCTemplate.create("Zara", 11, 99, 2010);
            studentJDBCTemplate.create("Nuha", 20, 97, 2010);
            studentJDBCTemplate.create("Ayan", 25, 100, 2011);

            System.out.println("------Listing all the records--------" );
            List<StudentMarks> studentMarks = studentJDBCTemplate.listStudents();
            for (StudentMarks record : studentMarks) {
            System.out.print("ID : " + record.getId() );
            System.out.print(", Name : " + record.getName() );
            System.out.print(", Marks : " + record.getMarks());
            System.out.print(", Year : " + record.getYear());
            System.out.println(", Age : " + record.getAge());
            }
        }
    }
    ```

#### 2. 声明式事务管理
> “声明式事务管理” 意味着你从业务代码中分离事务管理。你仅仅使用注释或 XML 配置来管理事务

基本步骤：

只需要在配置文件中配置即可，其他的和正常的 JDBC 过程没有变化

编码：
- script.sql
    和 [1. 编程式事务管理] 相同
- StudentMarks.java
    和 [1. 编程式事务管理] 相同
- StudentDAO.java
    和 [1. 编程式事务管理] 相同
- StudentMarksMapper.java
    和 [1. 编程式事务管理] 相同
- StudentJDBCTemplate.java
    > 和 [1. 编程式事务管理] 的区别：**没有涉及到任何 transaction 的操作，由配置文件来处理**
    ``` java
    package com.tutorialspoint;

    import java.util.List;
    import javax.sql.DataSource;
    import org.springframework.dao.DataAccessException;
    import org.springframework.jdbc.core.JdbcTemplate;

    public class StudentJDBCTemplate implements StudentDAO{
        private JdbcTemplate jdbcTemplateObject;
        
        public void setDataSource(DataSource dataSource) {
            this.jdbcTemplateObject = new JdbcTemplate(dataSource);
        }

        public void create(String name, Integer age, Integer marks, Integer year){
            try {
                String SQL1 = "insert into Student (name, age) values (?, ?)";
                jdbcTemplateObject.update( SQL1, name, age);
                String SQL2 = "select max(id) from Student";
                int sid = jdbcTemplateObject.queryForInt( SQL2 );
                String SQL3 = "insert into Marks(sid, marks, year) values (?, ?, ?)";
                jdbcTemplateObject.update( SQL3, sid, marks, year);

                System.out.println("Created Name = " + name + ", Age = " + age); // 只回滚数据库操作，输出一定会输出
                // to simulate the exception.（模拟出错）
                throw new RuntimeException("simulate Error condition") ;
            } catch (DataAccessException e) {
                System.out.println("Error in creating record, rolling back");
                throw e;
            }
        }

        public List<StudentMarks> listStudents() {
            String SQL = "select * from Student, Marks where Student.id=Marks.sid";
            List <StudentMarks> studentMarks=jdbcTemplateObject.query(SQL, 
            new StudentMarksMapper());
            return studentMarks;
        }
    }
    ```
- Beans.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">

        <!-- Initialization for data source -->
        <bean id="dataSourceBean" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/TEST"/>
            <property name="username" value="root"/>
            <property name="password" value="cohondob"/>
        </bean>

        <!-- Initialization for TransactionManager -->
        <bean id="transactionManagerBean"class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource"  ref="dataSourceBean" />
        </bean>

        <!-- Definition for studentJDBCTemplate bean -->
        <bean id="studentJDBCTemplateBean" class="com.tutorialspoint.StudentJDBCTemplate">
            <property name="dataSource"  ref="dataSourceBean" />
        </bean>

        <!--  -->
        <tx:advice id="txAdvice"  transaction-manager="transactionManagerBean">
            <tx:attributes>
                <tx:method name="create"/> <!-- @todo：意义不明 -->
            </tx:attributes>
        </tx:advice>

        <!-- 使用 AOP 来操作事务 -->
        <aop:config>
            <aop:pointcut id="createOperation" expression="execution(* com.tutorialspoint.StudentJDBCTemplate.create(..))"/> <!-- 定义哪些方法是要以事务方式来执行的。这里是在执行 StudentJDBCTemplate.create() 方法时发送通知给 AOP -->
            <aop:advisor advice-ref="txAdvice" pointcut-ref="createOperation"/> <!-- AOP 接收到通知之后，执行 txAdvice -->
        </aop:config>
    </beans>
    ```
- MainApp.java
    和 [1. 编程式事务管理] 相同

---

## MVC















































## SSM 整合
1. 三层架构

    - 表现层：SpringMVC
        - 单独使用
            - SpringMVC 的 jar 包
            - 配置文件：spring-mvc.xml

    - 业务层：Spring
        - 单独使用
            - spring-ioc, spring-aop 等 jar 包
            - 配置文件：applicationContext.xml
        - SSM 整合使用
            - 配置文件：添加关于 mybatis 的配置

    - 持久层：MyBatis
        - 单独使用
            - mybatis 自身的核心包
            - 数据库驱动 mysql-connector-java-x.x.x-bin.jar
            - 配置文件：mybatis-config.xml, customer.xml
        - SSM 整合使用
            - mybatis-spring 包
2. 单独使用 MyBatis

    详细见 [MyBatis](./MyBatis.md)
3. Spring + MyBatis 整合

    详细见 [MyBatis](./MyBatis.md)

4. SSM 整合


## SpringMVC
> Spring 的发展太快导致内容变化太快，而网络上流传了不同版本的实现，是否需要追本溯源？
### 概述
1. 三个处理器
    HandlerMapping（决定处理请求的 URL）
    HandlerAdapter（）
    ViewResolver（渲染 JSP 文件）
2. 处理流程
### 编码
#### 用到的包
- Spring 包
- Spring-webmvc 的包
#### 概述
1. 配置核心控制器 web.xml
2. 配置处理器控制器 springmvc.xml （实际与 spring 的配置文件一致）

    注册 Controller
3. 编写 Controller（实现 Controller 接口的类）

#### 具体步骤

### Controller 获取请求中的参数的多种方式
[参考](https://blog.csdn.net/a909301740/article/details/80411114)



## 其他
### Spring 框架二进制文件下载
[http://repo.spring.io/release/org/springframework/spring](http://repo.spring.io/release/org/springframework/spring)

### Spring 的体系结构意味着什么？
> 回想一个问题？原来使用 Eclipse 创建 Spring 项目时会新建了一个 lib 文件夹用以存放 “拷贝的” 的 jar 包，可是为什么我们需要这 **6个** jar 包呢？（使用 AOP 之后还增加了几个包）
    ![拷贝的 jar 包](./img/importJar.png)

- 答案：要用 Spring 提供的功能需要至少导入核心包，即这六个包

- 解释：

先节选两张 [W3Cschool](https://www.w3cschool.cn/wkspring/dcu91icn.html) 关于 Spring 体系结构的图

![Spring 体系结构](https://atts.w3cschool.cn/attachments/image/wk/wkspring/arch1.png)

从体系结构图可以看出核心容器（Core Container）包含了四个模块，分别对应了4个 jar 包（spring-beans, spring-core, spring-context, spring-expression）

![Spring Core 依赖](https://atts.w3cschool.cn/attachments/image/20181023/1540290875453691.png)

从 spring-core 的依赖可以看出还必须带上一个 commons-logging 的包。另，特殊一点的是 spring-context-support 包，这里 [W3Cschool](https://www.w3cschool.cn/wkspring/dcu91icn.html) 的解释是 **该包提供了对第三方库集成到 Spring 上下文的支持** 

- 总结：Spring 的体系结构告诉我们要使用什么功能需要导入什么包，整个 Spring 框架的包位于 [Spring-Github](https://github.com/spring-projects/spring-framework/tree/master)
- 延申：
    1. 在使用 Spring Initializr 创建一个 “Hello, World!” 的 Web 项目时，在 pom.xml 的 <dependencies> 节点中没有明显看到之前导入的 Spring 6个包，但可看到 "spring-boot-starter-web" 这样的依赖，实际上多次追踪就可以发现 spring-boot 中就包含了 spring-core 等 spring 的核心模块，所以说 SpringBoot 的其中之一作用就是简化配置
        > 如何追踪？使用 IDEA 的话可以按住 Ctrl 键的同时使用鼠标左键点击` <artifactId> `的内容，会跳转到另一个 xxx.pom 文件，找到该文件的` <dependencies> `节点再用同样的方法继续追踪下去
    2. Maven 的作用避免了我们多次复制/粘贴，保存 jar 包

### Spring 的 Core Container
> 来自 [W3Cschool](https://www.w3cschool.cn/wkspring/dcu91icn.html)
核心容器由` spring-core `，` spring-beans `，` spring-context `，` spring-context-support `和` spring-expression `（SpEL，Spring 表达式语言，Spring Expression Language）等模块组成，它们的细节如下：
- spring-core 模块提供了框架的基本组成部分，包括 IoC 和依赖注入功能。
> 依赖于` commons-logging `包
- spring-beans 模块提供 BeanFactory（工厂模式）的微妙实现，它移除了编码式单例的需要，并且可以把配置和依赖从实际编码逻辑中解耦。
- spring-context 模块建立在由 core 和 beans 模块的基础上建立起来的，它以一种类似于JNDI注册的方式访问对象。Context 模块继承自 Bean 模块，并且添加了国际化（比如，使用资源束）、事件传播、资源加载和透明地创建上下文（比如，通过Servelet容器）等功能。Context 模块也支持 Java EE 的功能，比如 EJB、JMX 和远程调用等。ApplicationContext 接口是 Context 模块的焦点。
- spring-context-support 提供了对第三方库集成到 Spring 上下文的支持，比如缓存（EhCache, Guava, JCache）、邮件（JavaMail）、调度（CommonJ, Quartz）、**模板引擎**（FreeMarker, JasperReports, Velocity）等。
- spring-expression 模块提供了强大的表达式语言，用于在运行时查询和操作对象图。它是JSP2.1规范中定义的统一表达式语言的扩展，支持set和get属性值、属性赋值、方法调用、访问数组集合及索引的内容、逻辑算术运算、命名变量、通过名字从Spring IoC容器检索对象，还支持列表的投影、选择以及聚合等。

### 依赖注入（DI）和控制反转（IoC）的关系
控制反转（IoC）是一个通用的概念，它可以用许多不同的方式去表达，依赖注入仅仅是控制反转的一个具体的例子。
