## .NET Framework

### 代码规范

1. 函数命名：
    - `GenXXXX()`     生成XXXX
    - `GetXXXX()`     获取XXXX
    - `BindXXXX()`    XXXX数据获取/更新
2. 控件的命名前加控件类型：
    - `cbx_`   ComboBox
    - `txt_`    TextBox
    - `rb_`     RadioButton
    - `btn_`    Button
    - `lab_`    Label
    - `frm_`    Form
    - `tsmi_`  ToolStripMenuItem
    - `sldlg_`  SaveFileDialog
3. 类内私有数据成员全部使用 `_xxx` 的命名方式

### `#region` 和 `#endregion` 

`#region 区域名` 和 `#endregion` 是用来划分代码块的，与注释差不多

### A 项目引用 B 项目内的类的方法

在项目A的 “引用” 中添加需要引用的项目(项目B)

在项目A中 “using 命名空间名” 来指定引用项目(项目B)的命名空间

### 项目引用动态链接库(xxx.dll)的方法（编译时加载动态运行库）

在项目中设置 "DLL" 的文件夹，将动态链接库放入其中

在项目的 “引用” 中添加动态链接库的引用

### 项目运行时加载动态链接库的方法（运行时动态链接）

```C#
using System.Runtime.InteropServices;

[DllImport("TSCLIB.dll", EntryPoint = "about")]	// 指明DLL文件名以及方法名
public static extern int about();	// 将动态链接库的函数名映射到类内函数名
```

### 窗体间跳转的方法（x2）

1. 以打开新窗口的方式跳转：

``` C#
frmA a = new frmA();
this.Hide();
a.ShowDialog();
```

> 问题：这样做会不会一直占用内存，导致资源紧张？毕竟只是 Hide()

2. 在本窗体的 Panel 中跳转：

    1. 设置本窗体的 `IsMdiContainer` 属性为 `True`，添加控件 `panel1`
    2. 调用自定义函数

    ``` C#
    public void ShowFrom(Form fm)
    {
    	panel1.Controls.Clear();
    
    	fm.MdiParent = this;
    	fm.Parent = panel1;
    	fm.Dock = DockStyle.Fill;	// 使fm窗体填充在panel中
    	fm.FormBorderStyle = FormBorderStyle.None;	// 设置fm窗体的边框为无
    	fm.Show();
    }
    ```

3. 回到跳转的窗体（即从A窗体跳到B窗体，现在需要回到A窗体）

    作用：可以用来实现系统锁屏类似的锁定本程序

    ``` C#
    // frmA.cs
    frmA a = new frmA();
    a.Owner = this; // 注意！
    this.Hide();
    a.ShowDialog();
    
    // frmB.cs
    Owner.Show();
    this.Close();
    ```

4. 注意的地方

    1. frm.Show() 与 frm.ShowDialog() 的区别

        frm.Show() 为非模态显示，可以操作其他窗口

        frm.ShowDialog()为模态显示，不允许操作其他窗口

    2. this.Hide() 与 this.Close() 的区别 `(@ 未验证)`

        this.Hide() 只是将本窗体隐藏起来，所有的申请的资源不释放

        this.Close() 会释放本窗体使用的所有资源

    3. **可以从关闭了的窗体传递数据给其他窗体**（原因：对象在函数作用域内不被销毁，只是不显示了）

        frmB 中有一个 public int c，在 frmA 的某个函数中创建了一个 frmB 的对象且使用 `b.ShowDialog()` 显示，关闭 frmB 之后仍可以在 frmA 的该函数中使用 `b.c` 获取这个数据，函数结束后则不行

### 判断字符串不为 null 或 空

`string.IsNullOrEmpty(s);`

### `int?` 的意义

`int?` 可以被赋值为 null，`int` 不行，所以使用前需要判断

```C#
int? a;
int b;
if (a.HasValue)
    b = a.Value;
```

> 参考：https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable?view=netframework-4.8
>
> 个人理解：基本类型转变为对象

### 本执行文件（exe文件）所在的路径

可以通过 `Application.StartupPath` 获得本执行文件(exe文件)所在的路径

### 使用正则表达式

```C#
using System.Text.RegularExpressions;

Regex rg = new Regex(@"正则表达式");
Match mc = rg.Match(验证的字符串);
return (mc.Success);
```

### ConfigurationManagement 类的使用

即使 `using System.Configuration;` 依然无法使用 ConfigurationManager 类，还需要在解决方案资源管理器中本项目添加引用 System.configuration

### 改变鼠标指针样式

`this.Cursor = Cursors.xxxxx`

### 使用 VS 内置的 “关于”窗体

在 “项目-添加新项” 中选择 “’关于‘框”

其中的产品名称、版本、版权、公司名称等信息均保存在 `Properties/AssemblyInfo.cs` 中，在该文件中修改即可在运行时显示在关于窗体上

> 另：`Properties/AssemblyInfo.cs` 中的 AssemblyTitle 和 AssemblyDescription 属性可以在编译的 xxx.exe 文件的属性中查看到

### Assembly 程序集

1. 概念

    编译出来的可执行文件(.exe 文件)或者动态链接库文件(.dll 文件) 是程序集的表现形式

2. 分类

    - Shared Assemblies 共享程序集：可以在不同 .NET 应用程序中使用的程序集。.NET 框架程序集就是共享程序集。

        因为可以供不同应用程序使用所以需要使用强命名，必须完整定义版本信息。

        一般会被安装在 Global Assembly Cache,GAC 全局程序集缓存中

    - 私有程序集：指某个应用程序、组件和组成部分。

        一般部署在与应用程序安装目录相同的文件夹中。

3. 结构

    - 程序集元数据：包含程序集标识信息（名称、版本等），文件列表（程序集由哪些文件组成）和引用程序集列表等
    - 类型元数据：列举程序集中包含的类型信息，说明包含的类，每个类的属性和方法（即类的定义，无实现）
    - MSIL 代码：微软中间语言，实现程序集的功能
    - 资源：可能包含的图像、图标、声音等资源

4. 如何查看？

    在 "Visual Studio Tools - Visual Studio <版本号> 命令提示" 中输入命令 `ildasm` ，选择程序集文件即可查看到程序集结构

5.  作用

    查看并分析 IL 指令和程序集的元数据，有助于深入理解、掌握 .NET Framework 技术内幕

### .NET 程序部署方式

#### XCOPY 方式

> 优点：方便
>
> 缺点：需要用户计算机上安装了对应的 .NET Framework 及数据库，程序才可以正常运行

步骤：

1. 以 Release 方式编译程序，将项目 bin/Release 下的 xxx.exe 和 xxx.exe.config 复制到用户计算机
2. 在用户计算机完成数据库的附加工作（如果数据库需配置在本地的情况下）
3. 修改 xxx.exe.config 的数据库连接字符串
4. 运行

#### 制作安装程序

步骤：

1. 以 Release 方式编译程序

2. 打开 VS 新建一个项目，项目类型为 “其他项目类型-安装和部署-安装项目” ，输入项目名称

3. 在右边的解决方案管理器中鼠标右键项目，修改属性中的 Manufacturer 和 ProductName 

    > 这里会决定安装文件夹，默认为 `[ProgramFilesFolder][Manufacturer]\[ProductName]` ，不过可以在 “目标计算机上的文件系统中-应用程序文件夹” 的 DefaultLocation 属性中修改默认格式

4. 在左边的 “目标计算机上的文件系统中” 的 “应用程序文件夹” 鼠标右键选择 “添加-文件” ，选择原项目 Release 文件夹下的 xxx.exe 和 xxx.exe.config 

5. 在 “目标计算机上的文件系统中” 的 “应用程序文件夹” 添加图标文件

6. 在 “目标计算机上的文件系统中” 的 “用户的的“程序”菜单” 新建一个文件夹，命名为 ProductName

7. 在 “目标计算机上的文件系统中” 的 “用户的的“程序”菜单-ProductName文件夹” 和 “用户桌面” 中粘贴在 “应用程序文件夹” 中由 xxx.exe 生成的快捷方式（需要事先设置快捷方式的 Icon 属性为图标文件）

8. 选择 VS 的菜单的 “生成-生成解决方案”

9. 将生成的 xxx.msi 和 setup.exe 复制到用户主机安装

10. 安装完成后到安装目录修改连接数据库字符串，即可开始使用

解释：

“目标计算机上的文件系统中” 即待安装本程序的电脑结构，即模拟出来的电脑文件系统，这里添加了什么文件，到时候安装的电脑上对应的文件夹上就会有什么文件

其中，

- 应用程序文件夹：是程序的安装目录，放置程序的 xxx.exe 等文件，目录地址由属性 DefaultLocation 决定

- 用户的的“程序”菜单：即在电脑的菜单栏上是否需要显示本程序，一般为快捷方式

- 用户桌面：即在电脑桌面上是否需要显示本程序，一般为快捷方式

### C# 使用反射动态创建类型的实例

1. 基础

    一个解决方案中多个项目，其中类库项目会生成 .dll 文件，窗体项目会生成 .exe 文件，这两个文件都是 “程序集”。

    如果窗体项目 A 需要引用类库项目 B，生成时会自动将 B 类库生成的 B.dll 文件复制到 A 项目的 bin/debug/ 文件夹下，如果B类库项目还依赖于类库项目 C ，则会自动将 C.dll 复制到 A 项目的 bin/debug/ 文件夹下

2. 使用

    ```c#
    using System.Reflection;
    
    Assembly.Load(“程序集名称”).CreateInstance("命名空间.类名称");
    ```

3. 解释

    使用反射动态创建类型的实例即后期绑定，属于动态编程，[官方介绍](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/)

### ADO.NET 对数据库的操作

ADO.NET 对数据库的操作可以分为四类：

- 对数据库进行非连接式查询操作，返回多条记录。可以通过 SqlDataAdapter 对象的 Fill 方法来完成
- 对数据库进行连接式查询操作，返回多条查询记录。可以通过 SqlCommand 对象的 ExecuteReader 方法来完成
- 从数据库中检索单个值。可以通过 SqlCommand 对象的 ExecuteScalar 来完成
- 对数据库执行增、删、改操作。可以通过 SqlCommand 对象的 ExecuteNonQuery 来完成

### MVC

![三层架构演变](assets/三层架构演变.png)

#### 解释

表示层：只提供软件系统与用户交互的接口

业务逻辑层：负责数据处理的传递

数据访问层：数据的存取工作

业务实体：封装实体类数据结构，一般用于映射数据库的数据表或视图（多少个表多少个实体），用以描述业务中的对象，在各层中传递

通用类库：包含通用的辅助工具类

#### 编写

Model：根据表编写，几个表几个类

BLL：根据功能编写，如果同属于一个表的，编在一个类中，类内函数调用 DAL 实现数据库访问

DAL：数据库访问代码

> 还可以衍生出多个 DAL 实现接口 IDAL，BLL 只调用接口，再使用反射实现的动态生成对象的 DALFactory 类，可以实现工厂模式下的 DAL 访问

表示层、Common

## DataGridView 控件

> 官方：https://docs.microsoft.com/zh-cn/dotnet/framework/winforms/controls/datagridview-control-windows-forms

### 如何给行中没有内容的列添加内容（未绑定列）

> 官方称为“未绑定列”，官方有另外一种实现方法，参考：https://docs.microsoft.com/zh-cn/dotnet/framework/winforms/controls/unbound-column-to-a-data-bound-datagridview

在 DataGridView1 的 **`CellFormatting`** 事件中添加如下代码

``` C#
if (e.ColumnIndex == dataGridView1.Columns["ColumnName"].Index)
{
    e.Value = "需要显示的内容";
}
```

用处：可以用在直接在列上添加了一些操作功能（例如：手动实现删除本列的功能(dataGridView本身包含删除的事件)）时在行中对应位置显示“删除”这两个字，**但只建议用来操作显示的内容，不用来操作计算的内容（交给数据更新 BindDGV() 函数实现）**

如果要实现“删除”功能的话则需要在 DataGridView1 的 **`CellContentClick`** 事件中添加如下代码

``` C#
if (e.ColumnIndex == dataGridView1.Columns["ColumnName_del"].Index)	// 判断点击的是哪列
{
    // 获取某个 ID 号，根据该号执行删除功能（如果ID号不需要显示在 dataGridView 中，可以添加该列但不显示出来）
}
```

### 如何添加不是数据查询的行

> 类似“未绑定列”，就叫“未绑定行”好了

在获取/更新数据源的时候为 DataGridView 的数据来源对象(例如，dataTable)添加一个 DataRow 对象，用来设置特定的格式即可

### 使用 `RowPostPaint` 事件设置单元格的样式

总结：DataGridView 的`RowPostPaint`用来设置整体统一的显示格式(颜色、文字居中等)，`CellFormatting`用来对特定行、列、单元格设置显示格式(颜色、文字内容等)，`CellContentClick`用来对特定行、列、单元格点击时执行的动作

### 列的 Name 和 DataPropertyName

操作 DataGridView 的数据源（即 DataTable）的列时使用的是 DataPropertyName

操作 DataGridView 自身的列时使用的是 Name （**建议命名以 c 开头，以指明为 Cells，便于区分**）

``` C#
// DataGridView 的列的 Name=cID，DataPropertyName=ID
DataTable dt = new DataTable();
dt.Rows[0]["ID"] = 1;	// DataProperty

dataGridView1.Rows[0].Cells["cID"].Value.ToString();	// Name
```

### 问题“无法添加该列，原因是它被冻结并被置于未冻结的列之后”解决：

1. 在窗体的 Load 事件中添加 dataGridView1.AutoGenerateColumns = false;
2. 如果窗体设计是复制而来的，很有可能是 dataGridView 的列对象生成在 dataGridView 的对象生成之后了，查看 XXXX.Designer.cs 文件修改


## ReportViewer 控件

> 官方：https://docs.microsoft.com/zh-cn/sql/reporting-services/application-integration/using-the-winforms-reportviewer-control?view=sql-server-2017

### 使用

1. 创建窗体，在窗体上添加 ReportViewer 控件

2. 在本项目中新建一个 ”报表“(xxx.rdlc) 项，设计报表的界面

    > 注意事项：
    >
    > 1. 可以在工具箱中拖动 ”报表项“ 到报表界面。显示中文需要指定字体为中文字体；添加没有数据关联的表时，先将原 “数据行” 删除再添加行，否则易出重复行；
    > 2. 可以在 “报表数据” 栏中添加数据集和参数，两者的名字很重要，在为报表填充数据时需要指定该名字；
    > 3. 报表可以完全不使用数据集（使用场景：需要显示的关联数据行较多或数据列较多），**如果需要添加数据集的话，需要先在项目中添加该数据集依赖的数据源，可以是在项目中添加的数据项(xxx.xsd)，可以是通过 ”数据-添加新数据源“ 添加的数据源(./Properties/xxx)**。如果在数据源中看不到项目中已添加的数据源，需要成功生成项目后再添加

3. 在 ReportViewer 控件中指定报表为第2步中新建的 “报表”(xxx.rdlc)

    > 另：可以在 ReportViewer 的 ”选择数据源“ 中指定添加到报表的数据集的实例，默认情况下会自动生成一个数据绑定控件和一个数据集控件，系统自动设置 “选择数据源” 为数据绑定控件，数据绑定控件的数据源为数据集控件（可以重新选择数据源，数据集来源与第3步一致；**即时设置了数据集的实例，但是可以完全不使用**）

4. 编写代码

    ```C#
    using Microsoft.Reporting.WinForms;
    
    private void frmReport1_Load(object sender, EventArgs e)
    {
        // 填充参数
        ReportParameter pam = new ReportParameter("pam_name", "参数值");	// 注意 pam_name 必须和报表中新建的参数名相同
        reportViewer1.LocalReport.SetParameters(pam);
    
        // 填充数据集
        ReportDataSource rds = new ReportDataSource("DataSet1", dt);	// 注意这里的 DataSet1 必须和报表中新建的数据集同名，类似于为报表的这个属性赋值
        reportViewer1.LocalReport.DataSources.Clear();
        reportViewer1.LocalReport.DataSources.Add(rds);	// 这种使用方法是直接跳过了 ReportViewer 自动生成的数据绑定控件和数据集控件，可以操作这两个控件来达到同样的填充数据集功能
    	
        // 刷新报表
        this.reportViewer1.RefreshReport();
    }
    ```

### 其他

1. 如果出现“System.CannotUnloadAppDomainException”类型的未经处理的异常出现在 mscorlib.dll 中。其他信息: 卸载 Appdomain 时出错。 (异常来自 HRESULT:0x80131015)”

    解决：在 Form 的 Closing 事件中添加如下代码 `reportViewer1.LocalReport.ReleaseSandboxAppDomain();`