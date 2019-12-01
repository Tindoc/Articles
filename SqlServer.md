## SQL Server 数据库

### 如果表名或者字段名为 SQL Server 的保留字，需要加上方括号 `[]`

### Sql 中为 varchar 字段执行增删改查时要加单引号 `'`

### SQL Server 数据库环境恢复

> 方法1：较为传统且易于管理，但要注意权限问题，默认带数据
>
> 方法2：最为便捷且文件较小，速度相对 方法1 慢一点点，默认不带数据，需要设置

1. 使用 mdf 和 ldf 文件
    1. 在选择安装 SQLServer 的磁盘中找到 SQLServer 默认存放 mdf 和 ldf 的地方，如果安装在 C盘，一般路径为 `C:\Program Files\Microsoft SQL Server\MSSQL<版本号>.<实例名>\MSSQL\DATA`（也可以存放在任意路径，只要在第3步“添加”文件时可找到这两个文件即可）

    2. 将本文件夹中的 mdf 和 ldf 的文件复制到上一步找到的文件夹中

    3. 打开 SSMS(Microsoft SQL Server Management Studio) 软件，在 SSMS 的 “对象资源管理器 - 数据库” 点击鼠标右键，选择 “附加”，点击 “添加” 按钮选择第2步中的放置两个文件，根据提示操作即可
      （要注意文件权限的问题，见 https://jingyan.baidu.com/article/e2284b2b456c93e2e7118d7b.html ，可能的解决方法：将 SQL Server 的属性的登录身份设置为内置账户）

2. 使用 sql 脚本文件

    在 SSMS 中新建一个查询，将文件 script.sql 拖到该窗口，执行即可

    > script.sql 文件是通过在 SSMS 中对特定数据库的执行 “任务 - 生成脚本” 生成
    >
    > 缺点： script.sql 中指定了 mdf 和 ldf 文件的位置，如果你的机器安装 SQLServer 的磁盘或者版本不一致的话可能导致该位置难以寻找，但也不是什么大问题

### 远程连接 SQL Server 的注意事项

> 可以使用“SQLServer<版本号>配置管理器” 软件来启动或停止 SQLServer 服务，下面的步骤 1~2 的设置应该都可以在安装的时候设置，所以不一定需要安装 SSMS

0. 确保客户机能够 ping 通 SQL Server 所在的服务器
1. 在 SSMS 的 “鼠标右键点击某个数据库连接-属性-安全性” 设置 “SQL Server 和 Windows 身份验证模式”（如果在安装时没有设置 sa 账户的密码可能需要启用该账户，即第二步）
2. 在 SSMS 的 “对象资源管理器-某个数据库连接-安全性-登录名” 中找到 sa 账户并鼠标右键点击，选择 “属性”，在 “登录” 选项下选中 “启用” （确保启用即可）
3. 在 SSMS 的 “属性-连接” 设置 “允许远程连接到此服务器”
4. 打开 “配置管理器”，“SQL Server 网络配置 => SQLEXPRESS 的协议- TCP/IP” 设置为 “已启用”（可以在属性中修改端口）
5. 在防火墙中新建 “入站规则”，其中“要创建的规则类型”选择“程序”，程序路径为“C:\Program Files\Microsoft SQL Server\MSSQL<版本号>.<数据库实例名>\MSSQL\Binn\sqlservr.exe”
6. 在 Visual Studio 中 “添加连接” ，其中服务器名为 “<IP或服务器名>, <端口号>\\<数据库实例名>”

### 分离数据库

在 SSMS 中鼠标右键点击需要分离的数据库，选择 “任务-分离”，勾选 “删除连接”，单击确定即可

> 分离之后再取 mdf 和 ldf 文件

### `select @@IDENTITY`

可以得到上一次插入记录时自动产生的最大 ID

### SqlServer 数据库备份和恢复的

```sql
# 备份
backup database <databaseName> to disk='<backPath>'
# 恢复
Alter Database <databaseName> Set Offline with Rollback immediate;
use master;
restore database <databaseName> from disk='<restorePath>' With Replace;
Alter Database <databaseName> Set OnLine With rollback Immediate;
```
