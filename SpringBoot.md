# SpringBoot

## 创建 SpringBoot 项目
> 参考：[稀客大大博客](https://zed058.cn/code/dev/springboot-02%E5%86%8D%E4%BD%93%E9%AA%8C.html#_2%E3%80%81%E5%85%B6%E4%BB%96%E5%88%9B%E5%BB%BA%E6%96%B9%E5%BC%8F)

下面将介绍三种方式
### 1. 在 IDEA 中创建一个 Spring Initializr 项目（必须联网）

> 实际上就是第三种方法的一种便捷方式，在创建项目时可看到` Choose Initializr Service URL `指向的就是 [start.spring.io](https://start.spring.io/)

其中，在` 项目/src/main/java/包名/xxxxApplication `中由` @SpringBootApplication `注解指明了本 Web 项目的启动类，并在` main `函数中调用` SpringApplication.run() `启动项目

一个控制器类的例子
```java
package com.sample.springbootstart;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController // 处理http请求；等同于 @Controller + @ResponseBody
public class HelloWorld {
    @GetMapping ("/hello") // 映射 /hello 的 GET 方法（默认为 GET 方法）
    public String hello() {
        return "Hello, SpringBootStart_initializr.";
    }
}
```

> Bug，使用这种模式下创建的项目的 artifactId 不允许含有大写字母，原因不详
>
> 使用这种方式创建的项目默认使用的是 Spring Boot 项目中内嵌的 Tomcat 来运行（SpringBoot 2.2.0 Release 对应的是 Tomcat 9.0）
            
### 2. 在 IDEA 中创建一个 Maven 项目，手动加入 Spring 的依赖
> 共3步
#### 2.1 在 IDEA 中创建一个 Maven 项目
此时的项目结构如下图：（左边为创建完 Spring Initializr 项目的项目结构，右边为 Maven 项目的项目结构）

![区别](./img/diff.jpg)

可以看出差异还是很大的，比较重要的是
- 缺少启动类 xxxxApplication.java
- resources 文件夹缺少 application.properties 配置文件
- 在 External Libraries 中只有 java 1.8 的 jar 包
- pom.xml 中缺少了很多配置

#### 2.2 在文件 pom.xml 中添加 SpringBoot 的引用，至少需要以下四个
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

<!-- 手动添加 Spring Boot Maven 插件，用于应用打包 -->
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
编写完 pom.xml 之后刷新一下依赖或者 reimport 就会引入指明的 jar 包到 External Libraries 中

#### 2.3 添加启动器类
```java
package com.sample.java;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 指明启动类
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);   // 说明运行哪一个类
    }
}
```
    
### 3. 在 [start.spring.io](https://start.spring.io/) 中下载模板，再在 IDEA 中导入该项目
    
与第一种方式基本一致，要注意：

如果在导入的最后一步指定了其他的项目文件夹的话，IDEA 不会自动将导入的项目复制到新的文件夹，而是靠一个链接文件夹链接起来，**所以不能删除原导入项目，最好的做法就是不要修改文件夹**
    
> 可以理解为，最后一步并不能将文件移动到一个新的文件夹，只是复制了一份 .idea 这个文件夹到新的文件夹

## SpringBoot 启动
1. 如果使用 Spring Initializr 创建的项目默认使用 Spring Boot 内置的 Tomcat 来运行，不需要设置 Tomcat
2. 如果想使用非 Spring Boot 内置的 Tomcat 来运行的话，需要完成以下步骤（外部 Tomcat 的版本为 9.0 可成功运行，其他版本不行，原因不详）
    [参考](https://blog.csdn.net/fanshukui/article/details/80258793)（1. 修改启动类；2. 在 pom.xml 中去掉内嵌 Tomcat 的依赖和指明打包类型）

## SpringBoot 打包
> 基于使用 Maven 构建项目的前提
两种方式：
### 1. JAR 包
1. 在 pom.xml 中指定` <package>jar</package> `
2. 使用` mvn package `打包
3. 使用` java -jar xxx.jar `运行即可
> 因为 spring-boot-starter-web 依赖内嵌了一个 Tomcat，所以可以直接启动
>
> 缺点：需要保证第三步中使用的终端持续运行，不可关闭（解决：守护进程）
> 
> windows 下使用该方法运行，可能不能正常关闭项目（使用` netstat -aon|findstr 8080 `来查看监听端口号 8080 的进程，再根据 PID 使用` tskill xxxPID `来强制关闭或使用任务管理器关闭）
### 2. WAR 包
1. 在 pom.xml 中指定` <package>war</package> `
2. 使用` mvn package `打包
3. 将打好的 war 包放置在 tomcat 的 webapps 目录下，启动 tomcat 即可（Tomact 会自动解压 war 包）
> 多版本 tomcat 共存设置：
>
> 需要使用 IDE 调用的可以参考 [这里](https://blog.csdn.net/Jessica_xingxing/article/details/78483143)
>
> 不需要的可以参考 [这里](https://yq.aliyun.com/articles/255884)