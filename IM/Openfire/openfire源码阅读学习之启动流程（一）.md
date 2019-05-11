# openfire启动流程（ServerStarter类、XMPPServer类）
## 首先从org.jivesoftware.openfire.starter包下的ServerStarter.java文件中启动：

**流程图**：  

![Aron Swartz](https://raw.githubusercontent.com/Xiawen9/blogs/master/pictures/openfire%2BServerStart%E7%B1%BB.png)

   -  final ClassLoader parent = findParentClassLoader();装载启动类加载器（bootstrap contaioner）：
      - 原因：首先程序中的.java文件经过编译成功后生成.class文件，我们需要使用类加载将.class文件加载到JVM虚拟机中，这样程序就可以正常运行了。如果不使用类加载的话，程序也可以正常运行，但是JVM启动的时候不会一次性加载所有.class文件，而是根据程序运行的需要动态的加载基础类，其他类则等到需要的时候再加载，虽然节省了内存的开销，但是可能引起耗时等等。 
     - 扩展：Bootstrap Loader是使用C++语言编写的，在java中不存在实体；该启动类加载器主要加载JVM自身工作需要的基本类，如java.lang.\*、java.util.\*等等；ExtClassLoader加载位于$JAVA_HOME/jre/ib/ext目录下的扩展jar；AppClassLoader是ExtClassLoader子类，也是程序中的默认加载器，加载$CLASSPATH下的目录和jar。ClassLoader使用了双亲委托模式进行加载。
       - 扩展：**双亲委托模式**。
        1、首先，当前类加载器从自己已经加载的类中查询此类是否已经加载，如果已经加载则直接返回已经加载的类。
        2、其次，如果当前类加载器没有找到步骤1中的类，则委托父类加载器去加载，父类加载器执行步骤1，查询是否加载，如果成功则返回已加载的类；否则的话，继续委托父类的父类加载器去加载，直到bootstrp ClassLoader。
        3、如果所有的类加载器都没有加载此类，再由当前的类加载器加载，并放入自己的缓存中（每个类加载器都有自己的缓存，当类被加载后，则把类放进自己的缓存中，这样下次加载的时候就可以直接返回）。  
        4、综上3点，也就是，自己找不到此类的话，就让父类加载器去找，父类加载器找不到的话，再继续向上委托，直到父类加载器找到此类，停止向上委托，返回类；当然，如果到bootstrp ClassLoader还是找不到的话，那就还是自己去加载此类，并把它放入缓存中。
        
   - System.getProperty("openfire.lib.dir");获取目录：当前项目即openfire下的lib路径，自定义存放jar包的位置，如果不存在，则创建一个存放lib的文件夹（在本程序中最开始定义的DEFAULT_LIB_DIR，默认位置）。
   - ClassLoader loader = new JiveClassLoader(parent, libDir);该JiveClassLoader类负责装载openfire安装目录下面的lib目录下的jar或者zip资源。
    - 扩展：ClassLoader是一个抽象类，所有用户自定义的类装载器都实例化自ClassLoader。  
      
   - Thread.currentThread().setContextClassLoader(loader);   
      containerClass.newInstance();  
      将libDir文件的url加入到ClassLoader中，然后将ClassLoader类的对象也就是loader设置为当前线程的Context Class Loader。也就是为线程提供适当的类加载器。
   -  Class containerClass = loader.loadClass("org.jivesoftware.openfire.XMPPServer");
   containerClass.newInstance(); 
                    这是通过java的反射机制来加载org.jivesoftware.openfire包下的XMPPServer类的构造函数。  
                         
   - 在findParentClassLoader()函数中，ClassLoader parent = Thread.currentThread().getContextClassLoader();获取到了当前线程的类加载器。后面，如果不存在的话，就获取当前类即ServerStarter类相对于Class类的对象的加载器。如果还是没有获取到，就获取系统的类加载器。  
## 关于org.jivesoftware.openfire包下面的XMPPServer类
- **简介**：  
XMPPServer类将加载、初始化和启动所有服务器的主XMPP server模块。在JVM中，服务器是唯一的，本类中可以通过使用getInstance()方法获取单例XMPPServer。加载的模块将被初始化，并且可以通过服务器访问其他模块。这意味着一个模块定义另外一个模块的唯一方法是通过服务器。这个服务器也维护已加载模块的列表。在启动所有模块后，服务器将加载任何可用的插件。服务器的配置文件在Openfire/distribution/src/conf路径下的openfire.xml文件。  

- **启动流程**：  

流程图：  

 ![Aron Swartz](https://raw.githubusercontent.com/Xiawen9/blogs/master/pictures/openfire%2BXMPPServer%E7%B1%BB%2Bstart\(\).png)  
 
 本类的构造方法中调用start()函数，**服务器从start()函数启动**。  
 
1、首先调用initalize()方法初始化服务器；  

2、File pluginDir = new File(openfireHome, "plugins");
     pluginManager = new PluginManager(pluginDir);
在项目路径下创建一个文件为plugins用于创建一个插件管理器；  

3、如果服务器已经设置好了即setupMode为false(没设置的时候，默认值为true)，则调用verifyDataSource(); 进行数据库验证；loadModules();加载启动模块；initModules();初始化模块；startModules();启动插件模块；  

4、ServerTrafficCounter.initStatistics();初始化启动信息；  

5、pluginManager.start();装载插件并启动插件监控管理；  

6、然后打印日志，并将变量started设置为true，表示已经启动；  

7、唤醒服务器监听器监听，插件初始化和启动完成后会通知监听器；  

8、如果服务器设置成功，则调用scanForSystemPropertyClasses();扫描系统属性类。  

9、在启动过程中，如果出现错误，则打印log日志，并调用shutdownServer()函数关闭服务器。  


- **部分解释**：
   1、变量：
   logger：获取LoggerFactory中的日志；
   instance：单例XMPPServer；
   initialized：是否初始化；
   started：是否启动；
   nodeID：NodeID对象，表示一些byte[]操作；
   DEFAULT_NODE_ID：获取NodeID的实例；
   EXIT：表示终结；
   XML_ONLY_PROPERTIES：静态Set集合；
   modules：使用Map<>集合存储模块；
   listeners：使用List<>集合存储监听器；
   openfireHome：项目home地址；
   loader：类加载器；
   pluginManager：插件管理器；
   componentManager：组件管理器；
   remoteSessionLocator：远程会话定义器；
   setupMode：设置模式；
   STARTER_CLASSNAME：服务器启动类的相对地址字符串；
   WRAPPER_CLASSNAME：包装管理器；
   shuttingDown：是否关闭服务器；
   xmppServerInfo：xmpp Server信息 。
   
    - 扩展：使用**WrapperManager**原因：使用wrapper可以将Java程序包装成系统服务，可以随着系统的运行而自动运行，此方法修复了传统的做法带来的问题。传统做法是把程序压缩成jar包独立运行，这样当服务器重启或者出现异常的时候，程序往往无法自动修复或重启，也有解决方法，比如编写一段shell脚本随服务器启动而运行，但是这样指标不治本。  
   
    2、静态代码块：存储一些属性properties，使用List集合存储，如：管理控制台网络设置（管理控制台端口、管理控制台安全端口、管理控制台接口、网络接口）、移动信息服务中心设置（区域、全限定域名fqdn、安装程序、群集管理器属性名称、数据库连接驱动URL、数据库用户名、数据库密码、数据库连接测试、数据库连接超时或等待、数据库最小连接、数据库最大连接等等）。然后添加属性集。  
    
    3、部分函数解读：  
    
    ** initialize()：**  
    
    流程图：  
    
    ![Aron Swartz](https://raw.githubusercontent.com/Xiawen9/blogs/master/pictures/openfire%2BXMPPServer%E7%B1%BB%2Binitialize\(\).png)  
    
     1、初始化服务器，调用locateOpenfire()函数定位openfire路径；  
     
     2、setup节点为true时，则表示已经配置完成，将setupMode设置为false；启动应用程序时，需要配置JiveGlobals类，以便可以从配置文件中加载应用程序的初始配置。配置文件保存以XML格式、数据库配置和用户身份验证配置存储的属性；并进入管理员登录界面。  
     
     3、如果程序以独立模式运行，打印log日志，在JVM中增加一个关闭的钩子，当JVM关闭的时候，执行系统中已经设置的所有通过方法addShutdownHook添加的钩子，当系统执行完这些钩子后，JVM才会关闭。这些钩子在JVM关闭的时候进行内存清理、对象销毁等操作。也就是说在JVM关闭前执行一个线程，这个线程执行服务器关闭即shutdownServer()方法。  
     
     4、获取当前线程的类加载器。  
     
     5、创建缓存对象。  
     
     6、更新服务器信息。  
     
     7、如果setupMode和"autosetup.run"均为true，则说明服务器未配置，则进入浏览器打开9090端口进去的界面，未配置的openfire界面。此时调用runAutoSetup()方法进行配置，并保存相关信息；  
     
     8、将fqdn的存储从数据库移到xml配置文件。  
     

     getInstance()：获取单例服务器对象；
     
     XMPPServer()：构造方法，在这里调用start()函数来启动openfire服务器；
     
     getServerInfo()：获取服务器状态信息，XMPPServerInfo类对象；
     
     isLocal( JID jid )：判断给定地址是否是服务器的本地地址，如果是，返回true；
     
     isRemote( JID jid )：判断给定地址与本地服务器主机名是否匹配，不匹配返回true；
     
     getNodeID()：获取集群模式，唯一标识服务器的ID；
     
     setNodeID(NodeID nodeID)：设置集群模式，服务器节点ID；
     
     getDefaultNodeID()：获取集群模式之前此服务器使用的默认节点ID；
     
     matchesComponent( JID jid )：如果给定地址与组件服务JID匹配，则返回true；     
     
     createJID(String username, String resource)：创建本服务器的本地XMPP地址；
     
     createJID(String username, String resource, boolean skipStringprep)：创建本服务器的本地XMPP地址，新的JID的建设可以通过跳过字符串准备操作来优化；
     
     getAdmins()：返回包含服务器管理员的JID的集合；
      
     addServerListener(XMPPServerListener listener)：添加XMPP Server监听器；
     
     removeServerListener(XMPPServerListener listener)：移除XMPP Server监听器；
