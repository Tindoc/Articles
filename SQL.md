## SQL

### 特殊的 SQL

```sql
/* 获取按 Batch 列降序排列的行的 Batch 列数据的第一个（TOP() 还可以用来实现分页） */
SELECT TOP(1) Batch FROM InW ORDER BY Batch DESC;   
/* 获取 Batch 列前11位是‘xxx’字符串对应的行的 Batch 列的前3位字符串（可能只能为字符串数据类型才可以） */
SELECT RIGHT(Batch,3) FROM InW WHERE LEFT(Batch,11)='xxx';  
/* 获取服务器时间 */
SELECT GETDATE();  
```

### 实现“分页” 的 SQL

```C#
/// <summary>
/// 分页获取数据列表
/// </summary>
/// <param name="PageSize">每页记录条数</param>
/// <param name="PageIndex">当前页</param>
/// <param name="strWhere">查询条件</param>
/// <param name="count">总共记录条数</param>
/// <returns></returns>
public DataSet GetPageList(int PageSize, int PageIndex, string strWhere, out int count)
{
    string strSql = "";
    if (PageIndex == 1) {
    	strSql += "select TOP " + PageSize + " * FROM [InW] A";
    } else {
        int sumSize = PageSize * (PageIndex - 1);	// 把需要的记录的前面记录剔除掉，例如：每页10条，需要第三页，则把前10*(3-1)=20条去掉（NOT IN）
        strSql += "select TOP " + PageSize + " * FROM [InW] A WHERE Batch NOT IN(select TOP " + sumSize + " Batch FROM [InW] ORDER BY ID DESC)";
	}
    
    string strCnt = "SELECT count(id) FROM InW ";	// 总记录条数
    if (strWhere.Trim() != "") {
        strSql += " where " + strWhere;
        strCnt += " where " + strWhere;
    }
    strSql += " ORDER BY A.ID DESC";

    count = (int)DbHelperSQL.GetSingle(strCnt);	// 执行获取总记录条数的函数
    return DbHelperSQL.Query(strSql);	// 执行返回结果为 dataSet 的 SQL 语句的函数
}
```

### 多个条件随机组合查询时的 SQL 

每一个条件判断一次，每个条件语句的结尾都加上 “AND”，最后将末尾的 “AND” 去掉，这样可以不用考虑条件的顺序

```C#
if (!string.IsNullOrEmpty(_id)) {
	sqlWhere += " SupplyID LIKE'%" + _id + "%' AND";
}
if (!string.IsNullOrEmpty(_agent)) {
	sqlWhere += " AgentName='" + _agent + "' AND";
}
if (!string.IsNullOrEmpty(_barcode)) {
	sqlWhere += " SupplyID IN(SELECT DISTINCT SupplyID FROM SupplyDetail WHERE Barcode LIKE'%" + _barcode + "%') AND";
}
if (sqlWhere.Contains("AND")) {
	sqlWhere = sqlWhere.Substring(0, sqlWhere.Length - 3);	// 将末尾的 And 字符串去掉
}
```