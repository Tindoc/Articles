## Maven 相关
> [官网-5分钟入门](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
>
> [官网-入门教程](http://maven.apache.org/guides/getting-started/index.html)
>
> [菜鸟教程-Maven]( https://www.runoob.com/maven/maven-tutorial.html) 
>
> [搜索包坐标的网站](https://mvnrepository.com/)
1. Maven 的意义

    项目依赖管理（将多个项目用到的 jar 包统一在本地仓库，不需要重复复制/粘贴、保存） + 提供项目结构、目录结构的标准（利于项目的分析与开源）
2. Maven 的配置

    配置文件所在地址： `Maven安装目录\maven<版本号>\conf\settings.xml` **or** `C:\Users\<用户名>\.m2\settings.xml`
    
    建议：修改安装目录下的 `conf/settings.xml` 文件，并将内容复制到 `C:\Users\<用户名>\.m2\settings.xml` 中（因为多数软件利用的是这个配置文件，是全局配置文件）
    > 配置文件中比较重要的是 **本地仓库** 的地址（即下载的 jar 包存放位置）
    >
    > `<localRepository>/path/to/local/repo</localRepository>`
    > 
    > 默认为 `${user.home}/.m2/repository` 建议修改为其他路径，有建议说放置在 Maven 的安装目录下
3. pom.xml 文件的基本结构
    - 项目的基本信息 <groupId>, <artifactId> 等
    - 项目变量 <properties>
    - 项目依赖 <dependencies>
    - 编译设置 <build> (内含 <plugins>)
4. 常用命令
    - clean
    - package （需要看项目类型来判断是否打包，最根本的应该就是取决于项目的 [目录结构](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)）
    - compile
4. Maven 有项目模板（也叫 项目脚手架 or 工程模板） [参考](https://www.w3cschool.cn/maven/tz8e1htp.html)
    
    - 可以使用 `-DarchetypeArtifactId=xxxx` 来指定

        例：
        ``` xml
        $ mvn archetype:generate
          -DgroupId=com.companyname.xxx
          -DartifactId=xxxxx
          -DarchetypeArtifactId=maven-archetype-quickstart
        ```

    - 可以使用 `mvn archetype:generate` 来可视化选择需要生成的项目脚手架（据说共有 614 个，我只能查看到10个，不清楚是不是源的问题）
5. 指定私有仓库 [参考](https://www.w3cschool.cn/maven/a4m51htj.html)
    ``` xml
    <dependency>
        <groupId>xxxx</groupId>
        <artifactId>xxxx</artifactId>
        <scope>system</scope>
        <version>1.0</version>
        <systemPath>${basedir}\src\lib\xxxx.jar</systemPath>
    </dependency>
    ```