## Tomcat 相关
1. Tomcat 的官方安装说明

    版本说明：Core 的 zip 版本
    
    解压后在 README.md 中可以看到安装教程在 RUNNING.txt 文件中（与安装 JDK1.8 差差不多）

1. Tomcat 启动乱码

    修改 `Tomcat安装目录/conf/logging.properties` 下的  `java.util.logging.ConsoleHandler.encoding = utf-8` 改为 `GBK` 即可
    > 或在 Run->Edit Configurations->Tomcat server->VM Options 加上 `-Dfile.encoding=UTF-8` （未经测试）