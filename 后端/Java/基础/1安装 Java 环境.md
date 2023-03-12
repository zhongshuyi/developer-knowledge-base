---
order: 1

author: 钟舒艺
---
# 安装 Java 环境

安装环境总是第一步，下面就介绍一些安装 Java 环境的方式

**什么是 JDK?**

SUN 公司提供了一套 Java 开发环境，简称 JDK(JavaDevelopmentKit)，它是整个 Java 的核心，其中包括 Java 编译器、Java 运行工具、Java 文档生成工具、Java 打包工具等。

SUN 公司除了提供 JDK，还提供了一种 JRE(JavaRuntimeEnvironment) 工具，它是 Java 运行环境，是提供给普通用户使用的。由于用户只需要运行事先编写好的程序，不需要自己动手编写程序，因此 JRE 工具中只包含 Java 运行工具，不包含 Java 编译工具。值得一提的是，为了方便使用，SUN 公司在其 JDK 工具中自带了一个 JRE 工具，也就是说开发环境中包含运行环境，这样一来，开发人员只需要在计算机上安装 JDK 即可，不需要专门安装 JRE 工具了。

JDK 下载地址：[https://www.oracle.com/java/technologies/java-se-glance.html](https://www.oracle.com/java/technologies/java-se-glance.html)

## Windows

### Chocolatey 安装

在网络好的情况下，推荐使用这种方式

Chocolatey 是一款专为 Windows 系统开发的、基于 NuGet 的包管理器工具，类似于 Node.js 的 npm，MacOS 的 brew，Ubuntu 的 apt-get，它简称为 choco。Chocolatey 的设计目标是成为一个去中心化的框架，便于开发者按需快速安装应用程序和工具。

Chocolatey 的官网： [https://chocolatey.org/](https://chocolatey.org/)

安装：[安装文档的链接](https://chocolatey.org/install)

安装只需要以管理员身份打开 cmd 或者 power shell，运行以下命令

```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

等一会，就安装好了 Chocolatey 了，然后就可以安装 Java 了

```bash
choco install jdk8
```

需要网!!!!    输入几个 yes，等一会就安装好了，环境变量也帮你配置好了

MySQL 同理，不用配置这么多东西直接安装好了

### msi 安装包安装

这个方式最常见，但很多人找不到 JDK 的下载地址，Oracle 官网下载的话还要登录，不方便

我们可以使用国内的清华大学镜像源的 **Adoptium** 源

地址：[Adoptium OpenJDK 清华源](https://mirrors.tuna.tsinghua.edu.cn/Adoptium/)

选择你要安装的 JDK 版本，然后选择 jkd -> x64 -> windows

windows 建议.msi 的安装包，直接下载安装就好，它会自动配置环境变量

例如：jdk 8 的下载地址是 [https://mirrors.tuna.tsinghua.edu.cn/Adoptium/8/jdk/x64/windows/OpenJDK8U-jdk_x64_windows_hotspot_8u352b08.msi](https://mirrors.tuna.tsinghua.edu.cn/Adoptium/8/jdk/x64/windows/OpenJDK8U-jdk_x64_windows_hotspot_8u352b08.msi)

下载后双击，一直下一步就可以了

## Ubuntu

先更新源：

`apt-get update`

然后直接安装 java

`sudo apt install openjdk-11-jdk`

修改 `~/.bashrc`文件，增加环境变量

```shell
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

然后输入 `java -version`查看 java 版本信息，如果出现了版本信息就是配置好了
