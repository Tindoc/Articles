## 其他
1. 绿色安装版软件
    
    使用非 exe 程序安装的软件，多数只要 **解压+设置环境变量** 就可以正常使用，例如：jdk, Tomcat, Maven
    
2. 如何配置环境变量（用命令行实现）？（依然存在问题，建议使用最后的方法设置）

    - Linux: `setenv('JAVA_HOME','C:\Java\jdk<版本>');` + `setenv('PATH', [getenv('PATH') ';目录\bin']);` （当然，目录可以是 环境变量名，例如 `%JAVA_HOME%\bin`）
    - Windows: `setx JAVA_HOME "C:\Java\jdk<版本>" /M` + `setx PATH "%PATH%;目录\bin" /M` 

        > 其中，`/M` 指明为设置系统变量而不是本地(用户)环境变量，加双引号是避免 Windows 目录中空格的影响。[参考](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/setx)。另，设置后在当前终端使用 `echo %JAVA_HOME%` 会显示设置前的值，需要打开一个新的终端才可查看到
        >
        > 可能会出现权限和需要重启系统的问题，如果实在不行请使用 `鼠标右键“我的电脑”->属性->高级系统设置->环境变量` 来添加
3. JDK 的安装介绍

    如果安装的版本为 1.8 的话，可以只设置 JAVA_HOME 和 PATH 这两个环境变量，具体的参考 [官网](https://docs.oracle.com/javase/8/docs/technotes/guides/install/windows_jdk_install.html#BABGDJFH)
    
    > 简单解释：JAVA_HOME 就可以理解为一个变量，可用于给其他变量引用使用，例如：`PATH=%PATH%;<new path>` 这里面的 `%PATH%` 就是引用了 PATH 这个变量；PATH 也是一个变量，系统根据这个变量系统寻找可执行文件
4. GitHub 的 SSH Keys 设置

    可参考 [官网](https://help.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)
5. XML 文件的特殊字符 `&`

    `&` 是 XML 中的特殊字符之一，需要使用需用 `&amp;` 代替