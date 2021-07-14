# Spark客户端源码阅读

此Spark是基于XMPP协议的实现的客户端，与openfire、smack使用。而不是为大规模数据处理而设计的快速通用的计算引擎Apache Spark。
项目地址： https://github.com/igniterealtime/Spark

## 简述几个项目构建工具
- make工具：用于Linux下构建C/C++代码，在makefile文件中指定源文件如何编译以及连接最后生成可执行文件。（Windows下为nmake）。
- ant工具：基于Java的构建工具，类似于make，但是修复了make工具的一些缺陷，例如：可以跨平台。
- maven工具：在ant的基础上改进了一些，使用maven工具后，我们不需要自己再下载项目中所需要的jar包，不需要配置管理，构建的时候，maven会自动下载所需要的jar包。

## 简述pom.xml
- maven构建项目需要pom.xml文件，常常只需要几句命令就可以构建简单的项目。和在make工具中使用makefile差不多。
- pom是项目对象模型，通过xml来管理项目，使用pom.xml实现。
- 功能：管理源代码、配置文件、开发者的信息和角色、问题追踪系统、组织信息、项目授权、项目的url、项目的依赖关系等等。

## pom.xml属性
- pom包括了所有的项目信息，有关pom.xml详细内容请参照博客： https://blog.csdn.net/zhuxinhua/article/details/5788546
 https://blog.csdn.net/qq_33363618/article/details/79438044
 
## 导入项目
- 在Spark/core/src/documentation路径下有一个静态文件ide-eclipse-setup.html或者是ide-intellij-setup.html，使用浏览器打开，按照步骤导入工程。
- 如果导入项目上有红叉的话，而项目没有错，可以运行；则右击项目->maven->Update Project，更新下就好啦。
- 导入的项目出现的那些apple、assembly-descriptor等等这些是插件，多模块maven工程。

## 阅读源码常用快捷键（eclipse）
- Ctrl+Alt+H：查看函数调用栈，层级显示
- Ctrl+Alt+G：在工程中所有出现该函数的地方

## 代码分析
- pom.xml：（聚合）
    - spark/pom.xml关键部分：
<modules\> </modules\> 聚合各个模块，一次构建各个模块，core、emoticons、plugins、distribution。
    - spark-core/pom.xml（子pom.xml）：
<filtering\> </filtering\> 过滤属性，例如：当系统中user.name=xx，则在编译的时候将&{user.name}替换成xx
<dependency\> </dependency\> 管理jar包，编译时自动下载jar包
    - 还有其他的子pom.xml：emoticons/pom.xml、各个插件下的/pom.xml、distribution/pom.xml等等。
- spark-core/src/main/java/org/jivesoftware下：
    - AccountCreationWizard.java：创建账户功能，建立XMPPConnection连接（getConnection()函数），设置端口，SSL等等。
    - LoginDialog.java：登录对话框，即登录页面设计，设置端口、登录判断、登录处理等等。
    - MainWindow.java：主页面，一些按钮事件的处理，如：退出、断线等处理。
    - MainWindowListener.java：接口类，监听事件：最小化主窗口、激活主窗口以及关闭。
    - Restarter.java：重启Spark客户端，根据不同系统选择不同方法。
    - Spark.java：Spark安装结构，运行系统、语言选择、设置外观、传入参数等的。
    - SparkCompatibility.java：兼容性设置，新版本与旧版本。
    - SparkStartupListener.java：使用窗口注册表执行XMPP映射
    
- spark-core/src/main/java/org/jivesoftware/launcher下：
    - Installer.java--使用java安装程序创建器Install4j，创建注册表；JiveClassLoader.java--类加载器；**Startup.java：程序从这里启动**（启动流程：加载类、创建Spark对象等等）
    
- spark-core/src/main/java/org/jivesoftware/resource下：
    - 图片、声音等资源以及国际化。
    
- spark-core/src/main/java/org/jivesoftware/spark下：
    - 声音、按钮、聊天页面的管理。
    
- spark-core/src/main/java/org/jivesoftware/sparkcomponent下：
       1、borders文件夹：边缘设计。
       2、browser文件夹：网页版本的Spark设计。
       3、focus文件夹：顺序焦点遍历策略？
       4、panes文件夹：折叠窗格实现。
       5、renderer文件夹：渲染，扩展联系人项目、列表渲染。
       6、tabedPane文件夹：选项渲染。
- Spark/core/src/documentation文件夹：图片资源以及一些html文件（一些工程源码安装指导）。

- spark-core/src/test/java/org/jivesoftware/spark/plugin下：
    - 插件依赖测试：设置部分编译版本
        



