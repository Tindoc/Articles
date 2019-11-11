## 其他
1. 绿色安装版软件
    
    使用非 exe 程序安装的软件，多数只要 **解压+设置环境变量** 就可以正常使用，例如：jdk, Tomcat, Maven
    
2. 如何配置环境变量（用命令行实现）？（依然存在问题，建议使用最后的方法设置）

    - Linux: `setenv('JAVA_HOME','C:\Java\jdk<版本>');` + `setenv('PATH', [getenv('PATH') ';目录\bin']);` （当然，目录可以是 环境变量名，例如 `%JAVA_HOME%\bin`）
    - Windows: `setx JAVA_HOME "C:\Java\jdk<版本>" /M` + `setx PATH "%PATH%;目录\bin" /M` 

        > 其中，`/M` 指明为设置系统变量而不是本地(用户)环境变量，加双引号是避免 Windows 目录中空格的影响。[参考](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/setx)。另，设置后在当前终端使用 `echo %JAVA_HOME%` 会显示设置前的值，需要打开一个新的终端才可查看到
        >
        > 可能会出现权限和需要重启系统的问题，如果实在不行请使用 `鼠标右键“我的电脑”->属性->高级系统设置->环境变量` 来添加

4. GitHub 的 SSH Keys 设置

    可参考 [官网](https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)

5. XML 文件的特殊字符 `&`

    `&` 是 XML 中的特殊字符之一，需要使用需用 `&amp;` 代替

### 用户文档/使用文档的编写安排

1. 如何使用本文档（对本文档使用方法的建议）
2. 系统简介（简单介绍系统的功能，与业务对应）
3. 相关术语（介绍本文档中可能难以理解或容易误解的名词）
4. 用户权限（使用用户权限对应系统菜单栏功能介绍）
5. 首次使用（系统的初始化，在所有用户能够正常使用前做的准备工作）
6. 使用场景（日常使用时针对使用场景对应的系统操作，按照 查看/添加/更新/删除 的顺序对有关业务介绍）
7. 注意事项（介绍一些注意事项）