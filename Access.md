## Access 数据库

### Access 的 OleDb 类型对照表

[链接](https://blog.csdn.net/hbxtlhx/article/details/12848303) 

### Access 的保留字

[参考](https://support.office.com/zh-cn/article/Access-%E4%BF%9D%E7%95%99%E5%AD%97%E5%92%8C%E7%AC%A6%E5%8F%B7-ae9d9ada-3255-4b12-91a9-f855bdd9c5a2#__toc262648755) ，如果在表中声明了与保留字同样的列名，在编写 SQL 时需要为列名添上 `[columnName]` ，否则引发错误 "IErrorInfo.GetDescription 因 E_FAIL(0x80004005) 而失败"

### Access 一次只能执行一条 SQL 语句，且不支持存储过程 

### Access 对参数顺序有严格要求，使用 parameters 添加参数时要注意

### Access 实现 SQL Server 的 isnull()

- SQL Server `isnull(columnName, 0)`

- Access `iif(isnull(columnName), 0, columnName) `

### Access 文档所在

[Office VBA](https://docs.microsoft.com/zh-cn/office/vba/api/overview/) 这里的不是很完整

[VB Script](https://www.w3school.com.cn/vbscript/vbscript_ref_functions.asp) 这里也有部分

> 暂时还不是很清楚

[教程](http://www.accessoft.com/rdp/tutorial.html) 这个网站还提供很多资源

### Access 中参数的日期需要用 `#` 括起来

## Access 与 SQL Server 数据库的区别

|                                                              | Access                                                       | SQL Server                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 联结不同（SQL Server 联结后的 on 条件字段自动合并且 tableA.column=tableB.column 不影响实际输出的列名，而 Access 会影响到列名） | INNER JOIN                                                   | JOIN                                                         |
| 表别名不同（Access 必须使用 AS）                             | tableName AS otherName                                       | tableName otherName                                          |
| isnull() 的不同                                              | 只能判断是不是 null，格式 isnull(columnName)                 | 除了判断还可以在 true 的情况下赋值，格式 isnull(columnName, 0) |
| 转换函数不同                                                 | 使用 Str(), CDbl() 等，具体见 [链接](https://www.w3school.com.cn/vbscript/vbscript_ref_functions.asp) | Cast 基本解决一切                                            |
| 日期的比较                                                   | 用 `#` 括起来参数的日期                                      | 用 `'` 括起来参数的日期                                      |
|                                                              |                                                              |                                                              |
|                                                              |                                                              |                                                              |
|                                                              |                                                              |                                                              |
|                                                              |                                                              |                                                              |
