## MySQL

### MySQL 的安装
1. [官网](https://dev.mysql.com/downloads/)
    
    下载界面右侧可选其他版本
2. 安装时只需选择安装 'Server only' 即可

    因为可以通过 IDEA 来管理，不需要像以前装 sqldevelper(Oracle), SSMS(SQL Server) 这些 GUI 的管理工具
3. 安装时请创建一个用户，默认 root 用户是不能远程连接的，但是可以修改，关键词是 “开启远程访问”、“host” 等
    > 如果用不到远程连接可以不用新建，但是正常的项目为了安全不应该使用 root 账户。
    我的习惯：创建一个名为 dev 的账户
4. 如果在 IDEA 中测试连接数据库提示 serverTimeZone 类似错误，可以按照错误提示设置时区为 `GMT-8` (东八区)
    > 这只是途径之一，还可以 [这样](https://blog.csdn.net/cy12306/article/details/97259049) 修改, 但其实一步也能解决问题。
    原因大概在数据库默认安装的时候使用的是时区是 SYSTEM (从链接中可以看出)，而这个值是美国时区，出现了不匹配或 IDEA 不清楚 SYSTEM 的定义
5. IDEA 的 Database 操作可以看 [这里](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/database-introduce.md)
    > 十分推荐上面这个 IDEA 的教程 
6. 卸载旧版本/重装新版本 [这个](https://zhuanlan.zhihu.com/p/68190605) 
    > 按照里面有的就操作，没有的就不管（我是成功了，但不一定都可以 :no_mouth: ）
7. 添加用户失败，报错 `ERROR 1396 (HY000): Operation CREATE USER failed for 'XXXX'@'XXXX'`
    
    原因：数据库删除用户后需要刷新权限才可以实现删除（或重启服务。`drop user 'xxxx'@’xxxx’;` + `flush privileges;`，再创建新用户 
    > `flush privileges` 本质上是将当前 user 和 privilege 表中的用户信息/权限设置从 MySql 库（MySQL数据库的内置库）中提取到内存里

### 编码问题
- 容易引发的问题
    
    - 英文插入成功，中文无法插入，提示 `\xE4\xF5` 之类的
- 使用 UTF8 编码
    ``` sql
    CREATE TABLE tableName (
        id int(11),
        name varchar(25)
    ) CHARSET=utf8;
    ```

- 查询/修改编码

    - 数据库 
        
        `show variables like 'character_set_database';`

        `alter database <db_name> character set utf8`

    - 表

        `show create table <table_name>`

        `alter table <table_name> character set utf8;`
    
    - 字段

        `alter table <表名> change <字段名> <字段名> <类型> character set utf8;`

        > 例子：`alter table user change username username varchar(20) character set utf8 not null;`

### 存储过程的使用
``` sql
create PROCEDURE proc_sel(
    IN id integer,
    out outname varchar(20))
BEGIN
    select name into outname from testtable where id=id limit 1;
END
```

在 console 中调用：
``` sql
call proc_sel(1, @outname);
select @outname;
```

> 解释：需要声明一个变量来保存 out 的结果，需要显示的话还需要再额外选择一次

