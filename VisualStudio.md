## Visual Studio 相关

### 将解决方案的 `xxx.sln` 文件设置在包含的多个项目外面的方法

1. 新建一个空白解决方案（在 其他项目类型）

2. 编写应用程序的主入口点（在本解决方案中新建一个类项目）

    ```c#
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.ThreadException += new System.Threading.ThreadExceptionEventHandler(AppThreadException);	// 添加异常处理函数
        Application.Run(new frmLogin());   // 重点
    }
    
    /// <summary>
    /// 异常处理函数
    /// </summary>
    /// <param name="source"></param>
    /// <param name="e"></param>
    private static void AppThreadException(object source, System.Threading.ThreadExceptionEventArgs e)
    {
        string errorMsg = string.Format("未处理异常: \n{0}\n", e.Exception.Message + "         详细：\n" + e.Exception);
        MyLog.WriteLog(e.Exception);
    
        MessageBox.Show("\n  发生系统异常!\t\t\n", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }
    ```

3. 复制其他项目到该解决方案下或者在该解决方案中新建项目 

> 单个解决方案中包含多个项目时，项目之间的依赖很重要，出于这个原因，调试时最好将所有项目重新生成一遍已解决依赖的问题，如果使用了反射技术动态创建类型还需要将后期绑定的程序集(即 xxx.dll )添加到启动项目下，或者直接在启动项目中引用后期绑定的类库

### 在 VS 中修改数据库的表定义时建议设置

选项 => 数据库工具 => 表设计器和数据库设计器 => 防止保存需要重新创建表的更改(取消勾选)

### 类的调用关系可视化

操作： “体系结构-生成依赖关系图-按类” 

使用：按住 “ctrl” 键鼠标会变成抓手，可移动显示界面，此时使用鼠标滚轮还可以放大缩小显示界面

优点：可以跨项目查看调用和定义（跨项目时，使用鼠标右键的“转到定义”只能查看到元数据里面的函数声明，“查看所有引用”查看不到跨项目的引用）

### 批量修改窗体控件 Tab 顺序

选择菜单栏“视图-Tab键顺序”，再按照需要的顺序逐一点击即可

### 从 DLL 中添加控件

在“工具箱”中点击鼠标右键，选择“选择项”

在“浏览”中找到 DLL 文件的位置，点击“确定”

### 页码控制器的使用

1. 在窗体界面中添加页码控制器控件，在工具箱鼠标右键“选择项-浏览-找到WinFormPager.dll ”

2. 在窗体的 Load 事件中为页码控制器添加事件处理函数

    ```C#
    private void frmGoodsIn_Load(object sender, EventArgs e)
    {
        // 其他代码
    	pagerControl1.OnPageChanged += new EventHandler(pagerControl1_OnPageChanged);
    }
    
    void pagerControl1_OnPageChanged(object sender, EventArgs e)
    {
    	// 刷新 dataGridView 的函数，一般为 BindDGV() 函数
    }
    ```

2. 使用控件的属性查询数据库（一般在 dataGridView 的数据获取/更新函数中）

    `pagerControl1.PageSize` 每页数量

    `pagerControl1.PageIndex` 当前页

3. 设置控件的总记录条数

    `pagerControl1.DrawControl(count_数据条数);`
