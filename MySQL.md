## MySQL 的安装
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