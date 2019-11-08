## IDEA相关

> 主要参考
> 
> [参考1](https://github.com/judasn/IntelliJ-IDEA-Tutorial)
>
> [参考2](https://www.cnblogs.com/jajian/category/1280011.html)

2. IDEA 安装插件的实际意义 **不是** 单纯安装对应的软件，常用的 Tomcat 和 Git 没有，但 Maven 却有

    由于旗舰版的 IDEA 安装时默认勾选了 Maven，所以在安装的时候就已经把 Maven 装上了，
    想要在 Windows 的终端中执行 mvn 命令的话只需要设置 PATH 即可，无需手动安装 Maven，
    一般来说，IDEA 的 Maven 安装在 `C:/Program Files/JetBrains/IntelliJ IDEA 2019.2.3/plugins/` 文件夹内
    （取决于安装时的选项，还可以在 IDEA 的 `Settings->Build, Execution, Deployment->Maven->Maven home directory` 中查看到具体的目录），
    只需要在该文件夹里找到 Maven 的 `/bin` 目录，添加到系统变量之后重启电脑使之生效即可（可以注意到这里装了 maven2 和 maven3 两个版本）
4. IDEA 的两个重要目录
    - 安装目录中 `C:\Program Files\JetBrains\IntelliJ IDEA <版本>\bin` 有两个重要的文件
        - `idea<64/无>.exe.vmoptions` : 可执行文件的 VM 配置文件（大概是配置 VM 怎么执行可执行文件，即设置执行时的一些参数）
        - `idea.properties` : IDEA 的属性配置文件
        > 建议使用 Help->Edit Custom VM Options 和 Help->Edit Custom Properties 来修改，具体修改参考 [这里](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/installation-directory-introduce.md)
    - **用户设置** 一般在当前用户的 Home 目录（Winodws 为 `%userprofile%`，Mac/Linux 为 `~`）的 `.IntelliJIdea<版本>` 文件夹里，如果配置到不可理解的阶段，可以删除掉整个文件夹，IDEA 会重新创建一个，恢复到默认状态
        - `config` : IDEA 个性化化配置目录，或者说是整个 IDE 设置目录。主要记录了：IDE 主要配置功能、自定义的代码模板、自定义的文件模板、自定义的快捷键、Project 的 tasks 记录等等个性化的设置（安装不同版本时可导入这些设置）
        - `system` : IDEA 系统文件目录，里面主要有：缓存、索引、容器文件输出等等
5. 修改编码

    File->Settings->Editor->File Encodings 建议全部设置为 UTF-8
    
    另，勾选 Properties Files->Transparent native-to-ascii conversion 把 properties 文件的中文显示出来
6. 重要的快捷键
    - `Ctrl + D` 复制光标所在行 或 复制选择内容，并把复制内容插入光标位置下面
    - `Ctrl + Y` 删除光标所在行 或 删除选中的行
    - `Ctrl + +` 展开代码
    - `Ctrl + -` 折叠代码
    - `Ctrl + /` 注释光标所在行代码，会根据当前不同文件类型使用不同的注释符号
    - `Ctrl + End` 跳到文件尾
    - `Ctrl + Home` 跳到文件头
    - `Ctrl + Space` 基础代码补全，默认在 Windows 系统上被输入法占用，需要进行修改，建议修改为 Ctrl + 逗号
    - **`Alt + Enter` IntelliJ IDEA 根据光标所在问题，提供快速修复选择，光标放在的位置不同提示的结果也不同**
    - `Alt + Insert` 代码自动生成，如生成对象的 set/get 方法，构造函数，toString() 等
    - `Shift + F6` 对文件/文件夹重命名（Refactor->Rename）
    - `Ctrl + Alt + L` 格式化代码，可以对当前文件和整个包目录使用
    - `Ctrl + Shift + F` 根据输入内容查找整个项目 或 指定目录内文件
    - `Ctrl + Shift + R` 根据输入内容替换对应内容，范围为整个项目 或 指定目录内文件
    Debug 用
    - `F7` 进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则不会进入该内嵌的方法中
    - `F8` 进入下一步，如果当前行断点是一个方法，则不进入当前方法体内
    - `F9` 恢复程序运行，但是如果该断点下面代码还有断点则停在下一个断点上
    - `Alt + F8` 选中对象，弹出可输入计算表达式调试框，查看该输入内容的调试结果
7. IDEA 里两种代码模板功能实现
    - Live Templates
    - Postfix Completion
8. 其他 [自定义设置](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/settings-recommend-introduce.md)
    - 鼠标滚轮放大/缩小编辑器字体： 勾选 `菜单栏->Settings->Editor->General->Mouse->Change Font Size(Zoom) with Ctrl+Mouse Wheel`
    - 编辑器软换行：在编辑器中点击鼠标右键选择 `Soft-Wrap`
    - 打开 IDEA 时可选择打开的项目而不是打开最近的项目：取消勾选`菜单栏->Settings->Appearance&Behavior->System Settings->Startup/Shutdowm` 里面的项目
    - 生成 serialVersionUID：`菜单栏->Settings->Editor->Inspections->勾选 Serializable class without serialVersionUID` （作用：在实现 Serializable 接口的类没有添加 serialVersionUID 时警告，这时可以利用 Alt+Enter 智能填充来添加 serialVersionUID ）
        > 思路：设置没有包含 serialVersionUID 的为警告，再使用智能填充完成
    - 快速定位当前文件所在项目结构中的位置：`Alt + F1 + 1`
9. 优秀插件推荐
    [参考](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/plugins-settings.md)
10. Java 热部署
    
    可使用 JRebel 插件（付费），具体参考 [这里](https://github.com/judasn/IntelliJ-IDEA-Tutorial/blob/master/jrebel-setup.md)
11. 多个 Project 共享数据库连接
    
    使用 Database->Data Source Properties->Project Data Sources->选定一个数据库连接再鼠标右键选择 Make Global）
12. 使用 IDEA 的 Database 工具生成了新的 Schemas 但刷新不出来
    
    `鼠标右键点击数据库连接名->Database Tools->Manage Shown Schemas` 查看是否有新的 Schemas