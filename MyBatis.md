## MyBatis 相关
> 参考资料
>
> [HOW2J.CN](https://how2j.cn/k/mybatis/mybatis-tutorial/1087.html)

1. 编码流程
    1. POJO

        表映射类，一表一类，存在外键的直接使用对应的对象（原因见“为什么会出现一对多，多对一，多对多？”）
        > 新建 pojo 文件夹存放
    2. Mapper
        > 新建 mapper 文件夹存放
        两种方式：
        - XML 方式

            一般会和 POJO 类放在一起

            ``` xml
            <?xml version="1.0" encoding="UTF-8"?>
            <!DOCTYPE mapper
                    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

            <mapper namespace="com.how2java.pojo">
                <cache/>
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

                <resultMap type="Category" id="categoryBean">
                <!--property 对应的是类内属性的名字，column 与数据库表的列名相同，多表查询同名时需要特殊指明；-->
                <!--总结：将 column 列的数据填充到类 property 的属性中去 @t-->
                    <id column="cid" property="id" />
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
            </mapper>
            ```
        - 类方式（使用注解）
            1. 基本方式
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
            2. Provider 模式

                文件 CategoryDynaSqlProvider.java
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

                文件 CategoryMapper.java
                ``` java
                public interface Category() {
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
    3. mybatis-config.xml 文件
        ``` xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>
            <settings>
                <!-- 打开延迟加载的开关 -->
                <setting name="lazyLoadingEnabled" value="true" />
                <!-- 将积极加载改为消息加载即按需加载 -->
                <setting name="aggressiveLazyLoading" value="false"/>
                <!-- 二级缓存，针对同一个 sessionFactory 下的缓存 -->
                <setting name="cacheEnabled" value="true"/>
            </settings>

            <typeAliases>
                <package name="com.how2java.pojo"/>
            </typeAliases>

            <plugins>
                <plugin interceptor="com.github.pagehelper.PageInterceptor">
                </plugin>
            </plugins>

            <environments default="development">
                <environment id="development">
                    <transactionManager type="JDBC"/>
        <!--            <dataSource type="POOLED">-->
        <!--                <property name="driver" value="com.mysql.jdbc.Driver"/>-->
        <!--                <property name="url" value="jdbc:mysql://localhost:3306/how2java?characterEncoding=UTF-8&amp;allowMultiQueries=true"/>-->
        <!--                <property name="username" value="dev"/>-->
        <!--                <property name="password" value="bLk37FXPAIMf"/>-->
        <!--            </dataSource>-->

                    <!--使用连接池-->
                    <dataSource type="org.mybatis.c3p0.C3P0DataSourceFactory">
                        <property name="driverClass" value="com.mysql.jdbc.Driver" />
                        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/how2java?useUnicode=true&amp;characterEncoding=UTF-8&amp;autoReconnect=true&amp;failOverReadOnly=false"/>
                        <property name="user" value="dev" />
                        <property name="password" value="bLk37FXPAIMf" />
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
                <mapper resource="com/how2java/pojo/CategoryMapper.xml"/>
                <mapper resource="com/how2java/pojo/ProductMapper.xml"/>
                <mapper resource="com/how2java/pojo/OrderMapper.xml"/>
                <mapper resource="com/how2java/pojo/OrderItemMapper.xml"/>
        <!--        添加对 mappers 的支持-->
                <mapper class="com.how2java.mapper.CategoryMapper"/>
                <mapper class="com.how2java.mapper.ProductMapper"/>
                <mapper class="com.how2java.mapper.OrderItemMapper"/>
                <mapper class="com.how2java.mapper.OrderMapper"/>
            </mappers>
        </configuration>
        ```

1. MyBatis 的基本流程（XML 方式 + 注解方式）
    1. 初始化数据库连接（读取 mybatis-config.xml 文件）
        ``` java
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);

        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession session=sqlSessionFactory.openSession();
        ```
    2. XML 方式
        1. 根据 `session.xxxx('id')` 寻找 id 所在的 xxxMapper.xml 文件
        2. 通过 xxxMapper.xml 文件执行对应的 SQL 语句
        3. 根据 xxxMapper.xml 文件把返回的数据库记录封装在 resultType 中（查询操作需要指定 resultType，有参数时需要指明 parameterType ）
    3. 注解方式
        1. 使用 `session.getMapper(表Mapper.class)` 初始化 Mapper 类
        2. 根据调用的 mapper 接口的方法找到并执行对应的 SQL 语句
2. xxxMapper.xml 文件的标签（动态 SQL ）
    - if 判断执行
        ``` xml
        select * from product_
        <if test="name!=null">
            and name like concat('%',#{name},'%')
        </if>
        ```
    - where 用于 where 子句的多重组合

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
    - set 用于 set 子句的多重组合

        类似 where 标签

    - trim 可用于实现 where 和 set 标签的功能

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

    - choose 实现 if...else 功能

        没有 if...else 标签，使用 choose...otherwise 实现

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
    - foreach 实现 foreach ，多用于 SQL 的 in 语句中
        ``` xml
        SELECT * FROM product_
        WHERE ID in
        <foreach item="item" index="index" collection="list"
                 open="(" separator="," close=")">
            #{item}
        </foreach>
        ```
    - bind 实现字符串连接，多用于模糊查询上
        ``` xml
        <select id="listProduct" resultType="Product"> -->
            select * from   product_  where name like concat('%',#{0},'%') -->
        </select>
        <!-- 替代 -->
        <select id="listProduct" resultType="Product">
            <bind name="likename" value="'%' + name + '%'" />
            select * from   product_  where name like #{likename}
        </select>
        ```
3. 为什么会出现一对多，多对一，多对多？

    数据库映射类的属性不是一一对应基本数据类型，有些是直接对应其他的数据库映射类（外键什么的），这个时候查询的时候就不是只是查询出 A 库的外键，而是直接将 A 库外键对应 B 库的数据都查出来

    > 使用注解方式可以比 XML 方式更为深刻体会到这一点