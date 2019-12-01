# Spring 相关
> 简介：Spring 是一个基于 IOC 和 AOP 的结构 J2EE 系统的框架
>
> 参考资料：
>
> [官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/index.html) GitHub 中的 Wiki 中也有

## 配置文件 applicationContext.xml
### 基础

> Product.java
> ``` java
> package com.how2java.pojo;
> 
> import org.springframework.beans.factory.annotation.Autowired;
> 
> public class Product {
> 
>     private int id;
>     private String name;
>     @Autowired // 自动装配，需要在 applicationContext.xml 中指明 <context:annotation-config/>
>     // 或者使用 @Resource(name="c") // "c" 对应 applicationContext.xml 中的 bean
>     private Category category;
>     
>     // getter & setter
> }
>```

``` java
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
    <bean name="c" class="com.how2java.pojo.Category">
        <property name="name" value="category 1" />
    </bean>

    <bean name="p" class="com.how2java.pojo.Product">
        <property name="name" value="product1" />
        // <property name="category" ref="c"> // 因为使用了注解自动注入不再需要指明
    </bean>
</beans>
```

### 配置自动扫描
``` java
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
  
    <context:component-scan base-package="com.how2java.pojo"/> <!-- 自动扫描 com.how2java.pojo 下面的包，需要使用注解标记 -->
     
</beans>
```

- Category.java
    ``` java
    package com.how2java.pojo;
    
    import javax.annotation.Resource;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Component;
    
    @Component("p") // 指明名字，可以直接使用 (Product)context.getBean("p"); 获得，未指明的情况与类名相同（首字母小写）
    public class Product {
    
        private int id;
        private String name="product 1";
        
        @Autowired
        private Category category;
    
        // getter & setter
    }
    ```

### AOP

> AOP 的作用：把功能分为核心业务功能和周边功能，分别开发，最后把周边功能和核心业务功能 "编织" 在一起，这就叫 AOP

主要分为三个方面：
- 切面（在切点附近执行的方法）
- 切点（需要在自己附近自动执行方法的方法）
- 切面方法（在切点前，后或者环绕执行切面）

#### 1. 普通方式
- ProductService.java
    ``` java
    package com.how2java.service;
    
    public class ProductService {
        public void doSomeService(){
            System.out.println("doSomeService");
        }
    }
    ```

- LoggerAspect.java
    ``` java
    package com.how2java.aspect;
    import org.aspectj.lang.ProceedingJoinPoint;
    
    public class LoggerAspect {
        public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("start log:" + joinPoint.getSignature().getName());
            Object object = joinPoint.proceed(); // 执行切点
            System.out.println("end log:" + joinPoint.getSignature().getName());
            return object;
        }
    }
    ```

- applicationContent.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
    http://www.springframework.org/schema/context     
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    
        <bean name="c" class="com.how2java.pojo.Category">
            <property name="name" value="yyy" />
        </bean>
    
        <bean name="p" class="com.how2java.pojo.Product">
            <property name="name" value="product1" />
            <property name="category" ref="c" />
        </bean>
        
        <!-- 注册 pointcut -->
        <bean name="s" class="com.how2java.service.ProductService">
        </bean>   
        
        <!-- 注册 aspect -->
        <bean id="loggerAspect" class="com.how2java.aspect.LoggerAspect"/>
        
        <!-- 主要配置 -->
        <aop:config>
            <aop:pointcut id="loggerCutpoint"
                expression=
                "execution(* com.how2java.service.ProductService.*(..)) " /> <!-- 指明在执行 ProductService 的任意方法时执行 aspect --> 
                
            <aop:aspect id="logAspect" ref="loggerAspect">
                <aop:around pointcut-ref="loggerCutpoint" method="log"/>
            </aop:aspect>
        </aop:config>
    </beans>
    ```
#### 2. 注解方式
- com.how2java.service.ProductService.java
    ``` java
    package com.how2java.service;
    
    import org.springframework.stereotype.Component;
    
    @Component("s")
    public class ProductService {
        public void doSomeService(){
            System.out.println("doSomeService");
        }
    }
    ```
- com.how2java.aspect.LoggerAspect.java
    ``` java
    package com.how2java.aspect;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.springframework.stereotype.Component;
    
    @Aspect
    @Component
    public class LoggerAspect {
        
        @Around(value = "execution(* com.how2java.service.ProductService.*(..))") // 指明切面方法为环绕
        public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
            System.out.println("start log:" + joinPoint.getSignature().getName());
            Object object = joinPoint.proceed(); // 这里的位置也很重要
            System.out.println("end log:" + joinPoint.getSignature().getName());
            return object;
        }
    }
    ```
- applicationContext.xml
    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
    http://www.springframework.org/schema/context     
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    
        <context:component-scan base-package="com.how2java.aspect"/>
        <context:component-scan base-package="com.how2java.service"/>
        <aop:aspectj-autoproxy/> <!-- 自动代理 -->
    </beans>
    ```


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

## SpringBoot相关
1. 三种创建方式，[来源：稀客大大博客](https://zed058.cn/code/dev/springboot-02%E5%86%8D%E4%BD%93%E9%AA%8C.html#_2%E3%80%81%E5%85%B6%E4%BB%96%E5%88%9B%E5%BB%BA%E6%96%B9%E5%BC%8F)
    1. 在 IDEA 中创建一个 Spring Initializr 项目（必须联网）
        
        > 实际上就是第三种方法的一种便捷方式，在创建项目时可看到 `Choose Initializr Service URL` 指向的就是 [start.spring.io](https://start.spring.io/)
        
        其中，在 `项目/src/main/java/包名/xxxxApplication` 中由 `@SpringBootApplication` 注释指明了本 Web 项目的启动类，并在 `main` 函数中调用 `SpringApplication.run()` 启动项目
        
        一个控制器类的例子
        ```java
        package com.tindoc.springbootstart;
        
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController // 处理http请求；等同于 @Controller + @ResponseBody
        public class HelloWorld {
            @GetMapping ("/hello") // 映射 /hello 的 get 方法
            public String hello() {
                return "Hello, SpringBootStart_initializr.";
            }
        }
       ```
        
        > 小问题，使用这种模式下创建的项目的 artifactId 不允许含有大写字母，原因不详
        >
        > 使用这种方式创建的项目默认使用的是 Spring Boot 项目中内嵌的 Tomcat 来运行（SpringBoot 2.2.0 Release 对应的是 Tomcat 9.0）
             
    2. 在 IDEA 中创建一个 Maven 项目，手动加入 Spring 的依赖
        
        创建完项目之后的区别如下图：（左边为创建完 Spring Initializr 项目的项目结构，右边为 Maven 项目的项目结构）
        
        ![区别](./img/diff.jpg)
        
        可以看出差异还是很大的，比较重要的是
        - 缺少启动类 xxxxApplication.java
        - resources 文件夹缺少 application.properties 配置文件
        - 在 External Libraries 中只有 java 1.8 的 jar 包
        - pom.xml 中缺少了很多配置
        
        **那如何使一个 Maven 项目变成一个 SpringBoot 项目？**
        1. 在文件 pom.xml 中添加 SpringBoot 的引用，至少需要以下四个
            ```xml
            <!-- 指定打包格式 -->
            <packaging>war</packaging>

            <!-- 添加 java 版本属性 -->
            <properties>
               <java.version>1.8</java.version>
            </properties>
           
            <!-- 添加 SpringBoot 的父项目 -->
            <parent>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-parent</artifactId>
               <version>2.2.0.RELEASE</version>
            </parent>
       
           <!-- 手动添加依赖 -->
           <dependencies>
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-web</artifactId>
               </dependency>
           </dependencies>
       
           <!-- 手动添加 Spring Boot Maven 插件，用于将应用打包为一个可执行的 jar 包 -->
               <!-- 可以使用 Maven->LifeCycle->package 来打包为 jar/war 文件 -->
           <build>
               <plugins>
                   <plugin>
                       <groupId>org.springframework.boot</groupId>
                       <artifactId>spring-boot-maven-plugin</artifactId>
                   </plugin>
               </plugins>
           </build>
            ```
           编写完 pom.xml 之后 IDEA 就会引入指明的 jar 包到 External Libraries 中
        2. 添加启动器类
            ```java
           package appStart;
           
           import org.springframework.boot.SpringApplication;
           import org.springframework.boot.autoconfigure.SpringBootApplication;
           
           @SpringBootApplication // 指明启动类
           public class Start {
               public static void main(String[] args) {
                   SpringApplication.run(Start.class, args);   // 说明运行哪一个类
               }
           }
            ```
        
    3. 在 [start.spring.io](https://start.spring.io/) 中下载模板，再在 IDEA 中导入该项目
        
        与第一种方式基本一致，要注意：
        
        如果在导入的最后一步指定了其他的项目文件夹的话，IDEA 不会自动将导入的项目复制到新的文件夹，而是链接了起来，**所以不能删除原导入项目，最好的做法就是不要修改文件夹**
            
        > 可以理解为，最后一步并不能将文件移动到一个新的文件夹，只是复制了一份 .idea 这个文件夹到新的文件夹 
2. 如果使用 Maven 打包为 jar 包，再使用 `java -jar xxx.jar` 运行时，需要注意可能不能正常关闭项目

    windows 使用 `netstat -aon|findstr "8080` 来查看监听端口号8080的进程，就可以根据 PID 使用 `tskill` 来强制关闭或使用任务管理器关闭
3. 启动方式
    1. 如果使用 Spring Initializr 创建的项目默认使用 Spring Boot 内置的 Tomcat 来运行，不需要设置 Tomcat
    2. 如果想使用非 Spring Boot 内置的 Tomcat 来运行的话，需要完成以下步骤（外部 Tomcat 的版本为 9.0 可成功运行，其他版本不行，原因不详）
        [参考](https://blog.csdn.net/fanshukui/article/details/80258793)（1. 修改启动类；2. 在 pom.xml 中去掉内嵌 Tomcat 的依赖和指明打包类型）

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


## 其他
1. Spring 的体系结构意味着什么？
    > 回想一个问题？原来我们使用 Eclipse 创建 Spring 项目时会新建了一个 lib 文件夹用以存放 “拷贝的” 的 jar 包，可是为什么我们需要这 **6个** jar 包呢？（使用 AOP 之后还增加了几个包）
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
            > 如何追踪？按住 Ctrl 键同时鼠标左键点击 `<artifactId>` 的内容，会跳转到另一个 xxx.pom 文件，找到该文件的 `<dependencies>` 节点再用同样的方法继续追踪下去
        2. Maven 的作用避免了我们多次复制/粘贴，保存 jar 包