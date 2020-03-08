## FreeMarker 模板引擎

### 开发时实时更新

> [参考](https://blog.csdn.net/iteye_9153/article/details/82582908)

### SpringBoot 集成
如果使用 `Spring Initializr` 新建项目且勾选了 FreeMarker 模板引擎，只需要在 application.properties 中设置 `spring.freemarker.suffix=.html` 或 `spring.freemarker.suffix=.ftl` ，在 resources/templates 中新建对应后缀的文件就可以正常访问了
> 否则会导致 Circular view path 错误

如果使用 pom 导入可能需要额外设置 `spring.freemarker.tempalte-loader-path=classpath:/templates/` classpath 设置为项目中放置 view 的目录即可

- 依赖
    ``` xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>
    ```
- 完整配置（[这里](http://freemarker.foofun.cn/pgui_misc_servlet.html) 有一些，XML 版本）
    ```
    # FREEMARKER (FreeMarkerAutoConfiguration)
    #spring.freemarker.allow-request-override=false # Set whether HttpServletRequest attributes are allowed to override (hide) controller generated model attributes of the same name.
    #spring.freemarker.allow-session-override=false # Set whether HttpSession attributes are allowed to override (hide) controller generated model attributes of the same name.
    #spring.freemarker.cache=false # Enable template caching.
    #spring.freemarker.charset=UTF-8 # Template encoding.
    #spring.freemarker.check-template-location=true # Check that the templates location exists.
    #spring.freemarker.content-type=text/html # Content-Type value.
    #spring.freemarker.enabled=true # Enable MVC view resolution for this technology.
    #spring.freemarker.expose-request-attributes=false # Set whether all request attributes should be added to the model prior to merging with the template.
    #spring.freemarker.expose-session-attributes=false # Set whether all HttpSession attributes should be added to the model prior to merging with the template.
    #spring.freemarker.expose-spring-macro-helpers=true # Set whether to expose a RequestContext for use by Spring's macro library, under the name "springMacroRequestContext".
    #spring.freemarker.prefer-file-system-access=true # Prefer file system access for template loading. File system access enables hot detection of template changes.
    #spring.freemarker.prefix= # Prefix that gets prepended to view names when building a URL.
    #spring.freemarker.request-context-attribute= # Name of the RequestContext attribute for all views.
    #spring.freemarker.settings.*= # Well-known FreeMarker keys which will be passed to FreeMarker's Configuration.
    #spring.freemarker.suffix= # Suffix that gets appended to view names when building a URL.
    #spring.freemarker.template-loader-path=classpath:/templates/ # Comma-separated list of template paths.
    #spring.freemarker.view-names= # White list of view names that can be resolved.
    ```

### 其他

- Bootstarp [链接](https://getbootstrap.com/)
    
    > [中文](https://www.bootcss.com/)

    > [4.3.1 学习](https://www.runoob.com/bootstrap4/bootstrap4-tutorial.html)

- Font Awesome 字体图标 [链接](http://www.fontawesome.com.cn/faicons/)

- i18n 国际化 [链接]()

    > [Spring Boot 支持国际化](https://blog.csdn.net/u012100371/article/details/78199568)

