# MyBatis 相关
> 参考资料
>
> [HOW2J.CN](https://how2j.cn/k/mybatis/mybatis-tutorial/1087.html) 强烈推荐（一对多、多对一可能比较难理解，可以看看 [这里](#whyonemul)）
> 
> [官方文档](https://mybatis.org/mybatis-3/zh/index.html) 与上面教程目录类似，缺少实际的操作，参考方面更完整
> 
> [官方文档-翻译优化](https://blog.csdn.net/guanhang89/article/details/79121332)
>
> [MyBatis-Spring](http://mybatis.org/spring/zh/getting-started.html)
>
> [MyBatis Generator](http://mybatis.org/generator/) 自动生成 MyBatis 映射类、配置文件等
> 
> [MyBatis-Plus](https://mp.baomidou.com/) 简化开发
>
> [多对多（未测试）](https://www.yiibai.com/mybatis/mybatis-many2many.html#article-start)

## 编码（传统）
> 非传统：可使用 MBG(MyBatis Generator) 来自动生成

### 使用到的包
- org.mybatis.mybatis-x.x.x.jar
- mysql.mysql-connector-java-x.x.x.jar

### 概述
- 创建 POJO 类（用于映射表的实体类）
- 创建 Mapper
- 编写 mybatis-config.xml

### 具体步骤

#### 1. 创建 POJO 类
> 新建 pojo 文件夹来存放（或 model/entity 文件夹）

表映射类，一表一类，存在外键的直接使用对应的对象（原因见 [为什么会出现一对多，多对一，多对多？](#whyonemul)）
#### 2. 创建 Mapper

三种方式：<a id="tway"></a>
- [单纯 Mapper XML 文件](#purexml)
    ```
    目录结构
    - src
        - mapper
            - xxxMapper.xml
        - pojo
            - xxx.java
        mybatis-config.xml
    ```
- [单纯 Mapper 类（使用注解）](#pureclass)
    ```
    目录结构
    - src
        - mapper
            - xxxMapper.java
        - pojo
            - xxx.java
        mybatis-config.xml
    ```
- [Mapper XML 文件 + Mapper 类（不使用注解）](#mix)
    ```
    目录结构
    - src
        - mapper
            - xxxMapper.xml
            - xxxMapper.java
        - pojo
            - xxx.java
        mybatis-config.xml
    ```

> XML 配置文件比注解能实现更为复杂的 SQL 语法 [参考](https://mybatis.org/mybatis-3/zh/java-api.html#a.E6.98.A0.E5.B0.84.E5.99.A8.E6.B3.A8.E8.A7.A3)

1. 单纯 Mapper XML 文件<a id="purexml"></a>

    > 新建 mapper 文件夹来存放

    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="com.how2java.pojo"> <!-- 指明实体类的位置 -->
        <cache/>    <!-- 二级缓存：开启二级缓存 -->
        <insert id="addCategory" parameterType="Category" >
                insert into category_ ( name ) values (#{name})
        </insert>

        <delete id="deleteCategory" parameterType="Category" >
                delete from category_ where id= #{id}
        </delete>

        <select id="getCategory" parameterType="_int" resultType="Category">
                select * from   category_  where id= #{id}
        </select>

        <update id="updateCategory" parameterType="Category" >
                update category_ set name=#{name} where id=#{id}
        </update>

        <select id="listCategory" resultType="Category">
                select * from   category_
        </select>

        <select id="listCategoryByPage" resultType="Category">
            select * from   category_
            <if test="start!=null and count!=null">-->
                limit #{start},#{count}
            </if>
        </select>

        <select id="listCategoryByName"  parameterType="string" resultType="Category">
                select * from   category_  where name like concat('%',#{0},'%')
            </select>

        <select id="listCategoryByIdAndName"  parameterType="map" resultType="Category">
            select * from   category_  where id> #{id}  and name like concat('%',#{name},'%')
        </select>

        <!-- 这个 resultMap 的 id 给后面的 select 使用 -->
        <resultMap type="Category" id="categoryBean">
            <!-- property 对应的是类内属性的名字，column 与数据库表的列名相同，多表查询同名时需要特殊指明；-->
            <!-- 总结：当实体类和数据库表列名不一致时用于消除差别，将 column 列的数据填充到类 property 的属性中去 -->
            <!-- 高级用法：https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#Result_Maps -->
            <id column="cid" property="id" /> <!-- id
                和 result 的区别是 id 是唯一标识结果的属性，加速查询，可以使用 result 代替但不建议 -->
            <result column="cname" property="name" />

            <!-- 一对多的关系 -->
            <!-- property: 指的是集合属性的值, ofType：指的是集合中元素的类型 -->
            <collection property="products" ofType="Product">
                <id column="pid" property="id" />
                <result column="pname" property="name" />
                <result column="price" property="price" />
            </collection>
        </resultMap>

        <!-- 关联查询分类和产品表 -->
        <select id="listCategory_o2m" resultMap="categoryBean">
                select c.*, p.*, c.id 'cid', p.id 'pid', c.name 'cname', p.name 'pname' from category_ c left join product_ p on c.id = p.cid
        </select>

        <!-- 多对一，使用 association -->
        <!-- <resultMap type="Post" id="resultPostsMap">
            <result property="id" column="post_id" />
            <result property="title" column="title" />
            <result property="content" column="content" />
            <association property="user" javaType="User">
                <id property="id" column="userid"/>   
                <result property="username" column="username"/>   
                <result property="mobile" column="mobile"/>   
            </association> 
        </resultMap>
        <select id="getPosts" resultMap="resultPostsMap" parameterType="int">
            SELECT u.*,p.*
            FROM user u, post p
            WHERE u.id=p.userid AND p.post_id=#{post_id} 
        </select> -->
    </mapper>
    ```
    > 有 resultType, resultMap, parameterType, parameterMap

2. 单纯 Mapper 类（使用注解）<a id="pureclass"></a>
    > 新建 mapper 文件夹来存放（或 dao 文件夹）

    两种方式：

    1. 基本方式（这些接口构成 DAO 层）
        ``` java
        public interface CategoryMapper {
            @Insert(" insert into category_ ( name ) values (#{name}) ")
            public int add(Category category);

            @Delete(" delete from category_ where id= #{id} ")
            public void delete(int id);

            @Select("select * from category_ where id= #{id} ")
            public Category get(int id);

            @Update("update category_ set name=#{name} where id=#{id} ")
            public int update(Category category);

            @Select(" select * from category_ ")
            public List<Category> list();
        }
        ```

    2. 使用语句构造器
        > 使用添加一个额外的类，使用方法没有区别。优势：不用直接书写 SQL 语句 [参考](https://mybatis.org/mybatis-3/zh/statement-builders.html)

        - 文件 CategoryDynaSqlProvider.java（这就是额外的类）

            ``` java
            public class CategoryDynaSqlProvider {
                public String list() {
                    return new SQL()
                            .SELECT("*")
                            .FROM("category_")
                            .toString();

                }

                public String get() {
                    return new SQL()
                            .SELECT("*")
                            .FROM("category_")
                            .WHERE("id=#{id}")
                            .toString();
                }

                public String add(){
                    return new SQL()
                            .INSERT_INTO("category_")
                            .VALUES("name", "#{name}")
                            .toString();
                }
                public String update(){
                    return new SQL()
                            .UPDATE("category_")
                            .SET("name=#{name}")
                            .WHERE("id=#{id}")
                            .toString();
                }
                public String delete(){
                    return new SQL()
                            .DELETE_FROM("category_")
                            .WHERE("id=#{id}")
                            .toString();
                }

            }
            ```

        - 文件 CategoryMapper.java

            ``` java
            public interface CategoryMapper() {
                @InsertProvider(type=CategoryDynaSqlProvider.class,method="add")
                public int add(Category category);

                @DeleteProvider(type=CategoryDynaSqlProvider.class,method="delete")
                public void delete(int id);

                @SelectProvider(type=CategoryDynaSqlProvider.class,method="get")
                public Category get(int id);

                @UpdateProvider(type=CategoryDynaSqlProvider.class,method="update")
                public int update(Category category);

                @SelectProvider(type=CategoryDynaSqlProvider.class,method="list")
                public List<Category> list();

                @Select(" select * from category_ limit #{start},#{count}")
                public List<Category> listByPage(@Param("start") int start, @Param("count")int count);
            }
            ```
3. Mapper XML 文件 + Mapper 类（不使用注解）<a id="mix"></a>

    > 推荐使用，理由：既拥有 xml 配置的复杂可实现性，又拥有 mapper 类方法的智能提示
    
    **注意：这种方式靠 XML 文件的 mapper 节点的 id 名和类文件方法名来实现联系，所以必须一致**

    - CategoryMapper.xml 文件
        ``` xml
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper
                PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

        <mapper namespace="com.how2java.pojo"> <!-- 指明实体类的位置 -->
            <insert id="addCategory" parameterType="Category" >
                    insert into category_ ( name ) values (#{name})
            </insert>

            <delete id="deleteCategory" parameterType="Category" >
                    delete from category_ where id= #{id}
            </delete>

            <select id="getCategory" parameterType="_int" resultType="Category">
                    select * from   category_  where id= #{id}
            </select>

            <update id="updateCategory" parameterType="Category" >
                    update category_ set name=#{name} where id=#{id}
            </update>

            <select id="listCategory" resultType="Category">
                    select * from   category_
            </select>
        </mapper>
        ```
    - CategoryMapper.java 文件
        ``` java
        public interface CategoryMapper {
            public int addCategory(Category category);

            public void deleteCategory(Category category);

            public Category getCategory(int id);

            public int updateCategory(Category category);

            public List<Category> listCategory();
        }
        ```

#### 3. mybatis-config.xml 文件

至少需要 environments, mappers 两个节点，具体 [参考](https://mybatis.org/mybatis-3/zh/configuration.html)
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 延迟加载：打开延迟加载的开关 -->
        <setting name="lazyLoadingEnabled" value="true" />
        <!-- 延迟加载：将积极加载改为消息加载即按需加载 -->
        <setting name="aggressiveLazyLoading" value="false"/>
        <!-- 二级缓存：开启二级缓存，针对同一个 sessionFactory 下的缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>

    <typeAliases>
        <!-- 指明别名，在 xxxMapper.xml 中指明 resultType 时不需要指明完整包名，注意对应-->
        <package name="com.how2java.pojo"/>
    </typeAliases>

    <plugins>
        <!-- 分页用 -->
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
        </plugin>
    </plugins>

    <environments default="development">
        <environment id="development"> <!-- 用来配置多个数个数据库，例如开发环境和生产环境 -->
            <transactionManager type="JDBC"/>
            <!-- 不使用连接池 -->
<!--            <dataSource type="POOLED">-->
<!--                <property name="driver" value="com.mysql.jdbc.Driver"/>-->
<!--                <property name="url" value="jdbc:mysql://localhost:3306/how2java?characterEncoding=UTF-8&amp;allowMultiQueries=true"/>-->
<!--                <property name="username" value="dev"/>-->
<!--                <property name="password" value="bLk37FXPAIMf"/>-->
<!--            </dataSource>-->

            <!-- 使用 C3P0 连接池 -->
            <dataSource type="org.mybatis.c3p0.C3P0DataSourceFactory">
                <property name="driverClass" value="com.mysql.jdbc.Driver" />
                <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/how2java?useUnicode=true&amp;characterEncoding=UTF-8&amp;autoReconnect=true&amp;failOverReadOnly=false"/>
                <property name="user" value="root" />
                <property name="password" value="root" />
                <!-- 连接池初始化大小为3 -->
                <property name="initialPoolSize" value="3"/>
                <!-- 连接池最大为10 -->
                <property name="maxPoolSize" value="10"/>
                <!-- 连接池最小为3 -->
                <property name="minPoolSize" value="3"/>
                <!-- 连接池在无空闲连接可用时一次性最多创建的新数据库连接数  -->
                <property name="acquireIncrement" value="5"/>
                <!-- 连接的最大空闲时间，如果超过这个时间（秒），某个数据库连接还没有被使用，则会断开掉这个连接。如果为0，则永远不会断开连接,即回收此连接 -->
                <property name="maxIdleTime" value="30"/>

                <!-- 最大的Statement数量 -->
                <property name="maxStatements" value="500"/>
                <!-- 每个连接启动的最大Statement数量 -->
                <property name="maxStatementsPerConnection" value="50"/>
                <!-- 同时运行的线程数 -->
                <property name="numHelperThreads" value="5"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!-- 添加 XML 格式的 Mapper，使用 resource 属性，注意对应 -->
        <mapper resource="com/how2java/pojo/CategoryMapper.xml"/>
        <mapper resource="com/how2java/pojo/ProductMapper.xml"/>
        <mapper resource="com/how2java/pojo/OrderMapper.xml"/>
        <mapper resource="com/how2java/pojo/OrderItemMapper.xml"/>
        <!-- 添加 Mapper 类的支持，使用 class 属性，注意对应 -->
        <mapper class="com.how2java.mapper.CategoryMapper"/>
        <mapper class="com.how2java.mapper.ProductMapper"/>
        <mapper class="com.how2java.mapper.OrderItemMapper"/>
        <mapper class="com.how2java.mapper.OrderMapper"/>
    </mappers>
</configuration>
```

## 运行
### 1. 初始化数据库连接（读取 mybatis-config.xml 文件）

具体步骤：
1. 创建 SqlSessionFactoryBuilder 对象（一般直接使用匿名对象）
2. 从 SqlSessionFactoryBuilder 创建 SqlSessionFactory 对象（需要引入配置文件）
3. 调用 SqlSessionFactory 的方法打开 SqlSession

代码：
``` java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource); // Resources 是 MyBatis 提供的实用工具类
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

SqlSession session=sqlSessionFactory.openSession();
```

### 2. 执行 SQL 语句
1. 根据 id 执行（[对应：单纯 mapper xml 文件](#tway)）
    1. 根据 `session.xxxx('id')` 寻找 id 所在的 xxxMapper.xml 文件

        ``` java
        List<Category> cs = session.selectList("listCategory"); // 通过指定执行的数据库操作类型选择 session 的方法，再选择具体 mapper 配置里的 id
        for (Category c : cs) {
            System.out.println(c.getName());
        }
        ```
        > session 的方法（对应不同的数据库操作）：
        > - session.insert()
        > - session.delete()
        > - session.update()
        > - session.selectOne()
        > - session.selectList()
    2. 通过 xxxMapper.xml 文件执行对应的 SQL 语句

    3. 根据 xxxMapper.xml 文件把返回的数据库记录封装在 resultType 中（查询操作需要指定 resultType，有参数时需要指明 parameterType ）

2. 根据 Mapper 类的方法执行（[对应：含有 mapper 类的方法](#tway)）
    1. 使用 `session.getMapper(xxxMapper.class)` 初始化 Mapper 类

        ``` java
        CategoryMapper mapper = session.getMapper(CategoryMapper.class);

        Category c = new Category();
        c.setName("新增加的Category");
        mapper.add(c); // 调用 mapper 的方法，方法名取决于 mapper 类
        ```

    2. 根据调用的 mapper 接口的方法找到并执行对应的 SQL 语句
    
### 3. **提交事务和关闭连接**

> 重要：增/删/改 的情况下一定需要提交事务

``` java
session.commit(); // 提交事务
session.close(); // 关闭连接
```

## 动态 SQL（xxxMapper.xml 文件的标签）
- **if** 用于判断执行
    ``` xml
    select * from product_
    <if test="name!=null">
        and name like concat('%',#{name},'%')
    </if>
    ```
- **where** 用于 where 子句的多重组合

    如果任何条件都不成立，那么就在sql语句里就不会出现where关键字;如果有任何条件成立，会自动去掉多出来的 and 或者 or。

    ``` xml
    select * from product_
    <where>
        <if test="name!=null">
            and name like concat('%',#{name},'%')
        </if>
        <if test="price!=null and price!=0">
            and price > #{price}
        </if>
    </where>
    ```
- **set** 用于 update 语句的 set 子句的多重组合

    类似 where 标签，一般还得搭配 if 使用

    ``` xml
    update product_
    <set>
        <if test="name != null">name=#{name},</if>
        <if test="price != null">price=#{price}</if>        
    </set>
    where id=#{id}
    ```

- **trim** 用于实现 where 和 set 标签的功能（可能是 where 和 set 的基础实现）

    ``` xml
    select * from product_
    <trim prefix="WHERE" prefixOverrides="AND | OR ">
        <if test="name!=null">
            and name like concat('%',#{name},'%')
        </if>
        <if test="price!=null and price!=0">
            and price > #{price}
        </if>
    </trim>
    ```

- **choose** 实现 if...else 功能

    ``` xml
    SELECT * FROM product_
    <where>
        <choose>
            <when test="name != null">
            and name like concat('%',#{name},'%')
            </when>          
            <when test="price !=null and price != 0">
            and price > #{price}
            </when>

            <otherwise><!-- else 字段，当前面都不匹配时才选择 -->
            and id >1
            </otherwise>
        </choose>
    </where>
    ```xml
- **foreach** 实现 foreach ，多用于 in 子句中
    ``` xml
    SELECT * FROM product_
    WHERE ID in
    <foreach item="item" index="index" collection="list"
                open="(" separator="," close=")">
        #{item}
    </foreach>
    ```
- **bind** 实现字符串连接，多用于模糊查询上
    ``` xml
    <select id="listProduct" resultType="Product"> -->
        select * from   product_  where name like concat('%',#{0},'%') -->
    </select>
    <!-- 上面为原始方式，下面为使用 bind 的方式 -->
    <select id="listProduct" resultType="Product">
        <bind name="likename" value="'%' + name + '%'" />
        select * from   product_  where name like #{likename}
    </select>
    ```

## 为什么会出现一对多，多对一，多对多？<a id="whyonemul" />

具体场景：
- 一对多：首先根据用户 ID 读取一个用户信息，然后再读取这个用户所发布贴子
- 多对一：首先根据帖子 ID 读取一个帖子信息，然后再读取这个帖子所属的用户信息
- 多对多：假设项目中存在用户和用户组，从一个用户读取出它所在的用户组，从一个用户组也知道这个组内的所有用户信息（**一个用户在多个用户组，一个用户组包含多个用户**）

> 大概是属于 ORM 的思想吧。POJO 类的属性不是一一对应基本数据类型，个别是直接对应其他的 POJO 类（外键什么的），这个时候查询的时候就不是只是查询出 A 库的外键，而是直接将 A 库外键对应 B 库的整个条目的数据都查出来（使用懒加载的话可以延迟这部分数据的加载）

例如：
- Product.java

    ``` java
    public class Product {
        private int id;
        private String name;
        private float price;
        private Category category;
        
        // getter & setter
    }
    ```

product 表中有个 cid 外键对应 category 表的主键，所以在 Product 类中有个 Category 类的属性

> 使用注解方式可以比 XML 方式更为深刻体会到一对多，多对一，多对多的实际意义

> 多对一用 association(关联)，一对多用 collection(集合)。具体参考见 [这里](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#a.E9.AB.98.E7.BA.A7.E7.BB.93.E6.9E.9C.E6.98.A0.E5.B0.84)

## MyBatis + Spring 整合<a id="ms"></a>
### 使用到的包
- mybatis 包
- mysql-connector-java 包
- mybatis-spring 包
- spring 相关的包（core, jdbc）

### 编码
1. MyBatis 基础的 **POJO 类 + Mapper 类 + Mapper XML 配置** 

    ```
    目录结构
    - src
        - mapper
            - xxxMapper.xml
            - xxxMapper.java
        - pojo
            - xxx.java
        applicationContext.xml
    ```
    
    注意：
    - **Mapper 类中的方法名必须和 Mapper XML 配置中的 id 名一致**
    - 无需配置 mybatis-config.xml

2. 配置 Spring 的 applicationContext.xml

    ``` xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
        xmlns:context="http://www.springframework.org/schema/context"
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xsi:schemaLocation="
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.0.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

        <context:annotation-config />

        <!--配置数据库连接-->
        <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
            <property name="driverClassName">
                <value>com.mysql.jdbc.Driver</value>
            </property>
            <property name="url">
                <value>jdbc:mysql://localhost:3306/database?characterEncoding=UTF-8</value>

            </property>
            <property name="username">
                <value>root</value>
            </property>
            <property name="password">
                <value>root</value>
            </property>
        </bean>

        <!--配置 sqlSession -->
        <bean id="sqlSession" class="org.mybatis.spring.SqlSessionFactoryBean">
            <!-- 指明 pojo 类位置 -->
            <property name="typeAliasesPackage" value="com.how2java.pojo" />
            <!-- 引用上面的 datasource 的 bean -->
            <property name="dataSource" ref="dataSource"/> 
            <!-- 指明 mapper XML 文件的位置 -->
            <property name="mapperLocations" value="classpath:com/how2java/mapper/*.xml"/>
        </bean>

        <!--配置 Mapper XML 自动扫描-->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.how2java.mapper"/>
        </bean>
    </beans>
    ```

### 运行
``` java
import org.springframework.test.context.ContextConfiguration;

import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class MybaitsTest {
    @Autowired
    private CategoryMapper categoryMapper;

    @Test
    public void testAdd() {
        Category category = new Category();
        category.setName("new Category");
        categoryMapper.add(category);
    }

    @Test
    public void testList() {
        System.out.println(categoryMapper);
        List<Category> cs=categoryMapper.list();
        for (Category c : cs) {
            System.out.println(c.getName());
        }
    }
}
```