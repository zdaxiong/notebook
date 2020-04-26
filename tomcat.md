# tomcat 简介

## tomcat 架构

1，http服务不会直接调用业务类，而是把请求交个容器来处理，容器通过调用servlet 接口调用业务类。

![image-20200423164007611](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423164007611.png)



用户发起请求，http服务器会将servlet request 对象会把客户端的请求信息进行封装，然后 调用servlet 容器的service方法。servlet容器拿到请求之后，会根据请求的Url 和servlet的映射关系。找到对应的service，如果servlet还没有加载 会用反射机制找到这个servlet ，并调用servlet的init方法来初始化这，接着调用servlet的service来处理请求，请求处理完成之后将请求返回个http服务器。http服务器会把响应返回给客户端。

什么是servlet，

java

### 核心组件

tomcat的核心功能：

连接器负责对外的交流，容器复制内部的处理

1，contector (连接器)：处理Socket 连接，负责网络字节流与reques和response对象的转化

2，container(servlet 容器)： 加载和管理servlet ，以及处理request请求

![image-20200423165631750](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423165631750.png)

#### connector(连接器)

coyote 是tomcat 的连接器框架名称，是tomcat服务器提供给客户端访问的外部接口，客户端通过coyote与服务器建立连接，发送请求并接受响应。

coyote 封装了底层的网络通信，为catalina 容器提供了统一的结构，是catalina容器与具体的请求协议以及IOd操作万元解耦，

coyote 作为独立的模块，只负责具体的协议与IO相关的操作，与servlet规范实现没有直接的管理，

![image-20200423170504984](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423170504984.png)

一个容器可以对接多个连接器。

##### connector组件

![image-20200423171355035](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423171355035.png)

连接器中的各个组件的作用：

| 组件      | 作用                                             |
| --------- | ------------------------------------------------ |
| endpoint  | 接收请求，处理tcp/ip 协议                        |
| processor | 处理http协议                                     |
| adapter   | 将requests请求转化为servlet requests交给容器处理 |
|           |                                                  |





#### catalina  （容器）

tomct 分层结构图：

![image-20200423172258687](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423172258687.png)



tomcat 本质是一款servlet 容器，因此catalina 才是tomcat的核心，其他的都是catalina 的提供支撑的，比如 jasper模块，提供jasp引擎，naming 提供JNDI服务，juli 提供日志服务。

##### catalina 架构

![image-20200423172912942](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423172912942.png)



| 组件      | 职责                                                         |
| --------- | ------------------------------------------------------------ |
| catalina  | 解析tomcat的配置文件，以此创建server 的组件，bin根据命令对其进行管理 |
| server    | 复制启动servlet引擎tomcat连接器，server通过利是封cycle接口，提供一种优雅的启动和关闭整个系统的方式。 |
| service   | 一个服务器下有多个服务。                                     |
| connector | 连接客户端                                                   |
| container | 处理connector 发送的请求。                                   |

##### container

​	![image-20200423173928244](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423173928244.png)

组件的介绍：

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| engine  | servlet 引擎，管理多个虚拟站点。                             |
| Host    | 代表多个的虚拟主机。                                         |
| context | 表示一个web应用程序员，一个web应用程序可以包含多个wapper     |
| wapper  | 表示一个servlet ，wapper 作为容器中的最底层，不能包含子容器。 |

tomcat 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  
  <Service name="Catalina">
  	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
   <Engine name="Catalina" defaultHost="localhost">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
	<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>   
```

## tomcat的启动流程

启动时序图：

![image-20200423180527992](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423180527992.png)

tomcat解析：

lifecycle :

​	由于所有的组件存在初始化，启动，停止，等生命周期的方法，拥有生命周期的管理特性，所以tomcat 在设计的受，基于生命周期的抽象称一个接口lifecycle ,  而组件server，service container，executor，connector 组件，都实现了一个生命周期的接口。从而具有了一下生命周期的当中的核心方法。

init(): 初始化组件

start():启动组件

stop():停止组件

destroy():销毁组件

### 请求处理流程

tomcat用mappe组件来完成由那个wrapper容器来出路servlet。

Mapper组件的功能就是将用户的请求的URL定位到一个servlet。

其原理：

​	mapper组件当中保存了web应用的配置信息，其实就是同期组件与访问路径的映射关系。比如：host容器当中得配置的域名信息，context容器当中的web应用路径，以及wrapper容器当中的servlet映射路径，

​	当一个请求发送来的时候，mapper 组件通过解析URl请求的域名和路径，再到map当中去查找，就能定位到一个servlet，（一个请求最后定位到的是wrapper当中，也就是最底层的一个servlet）

![image-20200423203635553](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423203635553.png)

源码的处理流程：

![image-20200423204439921](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423204439921.png)





源码处理请求处理流程：

![image-20200423235249974](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200423235249974.png)





### jsper

对于jsp的web应用，最终输出给用户的是html文件（不包含js，cass），

jsper模块是tomcat的核心引擎，解析jsp语法，生成servlet bin生成class字节模块，用户在进行访问jsp时，会访问servlet，最终将访问的结果响应在浏览器端。另外jsper还会坚持jsp文件是否修改，如果修改，会重新编译jsp文件。

## tomcat服务的配置

### server.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
# 8005 端口 shutdown：关闭服务的指令字符串
<Server port="8005" shutdown="SHUTDOWN">
# 用于以日志形式输出服务器，操作系统，jvm的版本信息
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
# 用于加载和销毁APR，如果找不到ARP库，则会输出日志，并不影响tomcat的启动
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
# 用于避免JRE 内存泄漏问题
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
#用户加载和销毁全局命名服务
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    #用户在context停止时重建executor池的线程，以避免threadLocal的相关内存泄漏。
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
    #全局配置
  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  #service --》多个connector-->一个engine--》多个host
  <Service name="Catalina">
  	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
   <Engine name="Catalina" defaultHost="localhost">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
	<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>   
```

#### listener

listener用于监听servelet当中的事件，例如contex,request,session对象的修改，删除，并触发响应事件，listener是观察者模式的实现，在servlet当中主要的堆，context，request，session对象的生命周期进行监控。启动时，serveletContextListener的执行顺序与web.xml中的配置顺序一致，停止时与执行顺序相反

#### Executor（线程池）

在service当中的配置的线程池为共享的线程池，在tomcat当中允许存在多个connector，当没有配置executor时，各个connector都使用的各自的executor。当配置了，各个connector可以使用共享线程池。connector就不会自己在创建线程池。

```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
maxThreads="150" 
minSpareThreads="4"/>
```

属性说明：

| 属性                    | 含义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| name                    | 线程池的名称                                                 |
| namePrefix              | 创建每个线程的名前缀，单个线程名称为：namePrefix+threadNumber |
| maxthreads              | 线程池当中最大的线程数量                                     |
| minSpareTHreads         | 活跃得线程数量，也是核心池线程数，这些线程不会被销毁，会一直存在 |
| maxIdleTime             | 线程空闲的时间，单位是毫秒，超时会被销毁                     |
| maxQueueSize            | 在被执行前最大的线程书面，默认是Int数据类型的最大值，也就是没有上限，除非特殊情况，这个值不需要进行更改，否则会有请求会被处理的情况发生 |
| prestartminSpareThreads | 启动线程池时是够启动minSpareThreads部分线程，默认是false，不启动 |
| threadPriority          | 线程池当中的线程优先级，默认是5，值：1-10                    |
| classname               | 线程池类，没有指定的情况下是：org.apache.catalina.core.StandardThreadExecutor . |



#### connector(连接器)

一个service下可以有多个connector

```xml
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

 <Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
            	#-1 表示没有超时时间
               connectionTimeout="20000"
               redirectPort="8443" />
#redirectPort：当亲的connector不支持ssl会话，收到一个请，并且符合security-constranint约束，需要ssl传输时，cata自动将这个请求重定向到指定的8443端口
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"maxThreads="150" SSLEnabled="true">
     <SSLHostConfig>
         <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"
                         type="RSA" />
    </SSLHostConfig>
    </Connector>

    <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"  maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>

    <Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />
   
```

htt协议：

​	

```
orgd.apache.coyota.http11.http11NioProtocol #非阻塞java NIO 连接器
orgd.apache.coyota.http11.http11Nio2Protocol #非阻塞java NIO2 连接器
orgd.apache.coyota.http11.http11AprProtocol #非阻塞java APR 连接器
```

AJP 协议：

```
orgd.apache.coyota.ajp.AjpNioProtocol #非阻塞java NIO 连接器
orgd.apache.coyota.ajp.AjpNio2Protocol #非阻塞java NIO2 连接器
orgd.apache.coyota.ajp.AjpAprProtocol #非阻塞java APR 连接器
```

#### engine

engine 作为servlet引擎的顶级元素，内部可以嵌入，：cluster ，listener ，realm， valve 和host

```xml
<engine name="catalina" defaultHost="localhost">
	.......
<\engine>
```

| 属性        | 解释                           |
| ----------- | ------------------------------ |
| name        | engine的名称                   |
| defaultHost | 主机的名称，这里指定的当前主机 |



#### host

host 元素用于配置一个虚拟主机，他支持嵌入元素：Alias，Cluster，listener,valve,realm,context 如果在engine 下配置realm那么此配置将在当前engine下的所有Host共享，同样在Host中配置Realm，则在当前的目录下的context中共享，优先级：context当中的Realm>Host当中的Realm>engine当中的Realm。

```xml
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
```

| 属性       | 解释                                                         |
| ---------- | ------------------------------------------------------------ |
| name       | 当前host的通用的网络名称，必须与DNS服务器上的主从信息一致。engine中包的Host必须必须存在一个defaultHost设置一致，可以是域名，主机ip |
| appBase    | 当前host的基础目录，当前host上部署的web应用聚在该目录下（可以是绝对目录，也可以是相对的路径），默认是webapps |
| unpackWARS | 设置为true，host在启动时会将appBase目录下的war解压为目录，设置为false，host将直接从war文件启动 |
| autoDeploy | 控制tomcat是否在运行是定期检测并自动部署新增或变更的web应用。 |

##### host添加别名	

```xml
<host name="localhost" appBase="webapps" unpackWARS="true" autoDeploy="true" >
    <alias> www.baidu.com</alias>
</host>
```

#### context

用于配置一个web应用

内嵌元素有：cookieprocessor，loger，manager，realm，resources，watchedresources，jarScanner，valve等

```xml
<context docBase="webapps",path="/myapp">
    .....
</context>
```

| 元素    | 解释                                                         |
| ------- | ------------------------------------------------------------ |
| docBase | web应用的目录或者的是war包的部署路径，可以是相对路径，也可以是绝对路径，可以是相对于host appbase的相对路径 |
| path    | web应用的context路径，如果我们的host是localhost，则web应用访问的根路径是http://localhost/webapps/ |

### tomcat-user.xml

用于配置tomcat管理员以及用户的信息

```xml

```

### context.xml



## tomcat web应用配置

### web.xml

#### context-parm

​	配置context初始化的时候的一些参数信息

```xml
<context-param>
    <param-name>project_param_01</param-name>
    <param-value>itcast</param-value>
</context-param>
```

#### session

服务器与客户端建立连接时，http为无状态请求，服务器无法判断本次会话与上一次是否一致。通过会话保持用户访问服务器的信息。

![image-20200424134817113](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424134817113.png)

jsessionid:其实就是服务返回给客户端的cookie，客户端再次访问时，就会使用cookie去访问服务器。

```xml
<session-config>
    	<session-timeout>30</session-timeout>
    	<cookie-config>
            #cookie 名称
            <name>jsessionid6666</name>
            #域名
            <domain>localhost</domain>
            #访问路径
            <path>/</path>
            #注释信息
            <comment>session cookie</comment>
            #这个cookie只能通过http携带访问服务器
            <http-only>true</http-only>
            #true只有https才能携带cookie访问服务器
            <secure>false</secure>
            #cookie有效期限，单位是秒
            <max-age>3600</max-age>
    	</cookie-config>
    	#session的会话跟踪方式：COOKIE，URL
    	<tracking-mode>COOKIE</tracking-mode>
</session-config>
```

#### servlet

servlet 主要有时两部分，servlet和servlet-mapping

```xml
 <servlet>
     #servlet的名称，具有唯一性
        <servlet-name>default</servlet-name>
    #用于指定servlet的类   
     <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
     #指定servlet的初始化参数，
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
    #用于控制web应用启动时，servlet的加载顺序，小于0，web应用启动时，不加载改servlet.
        <load-on-startup>1</load-on-startup>
    </servlet>

#访问的url要映射给那个servlet
<servlet-mapping>
    <servlet-name>myservlet</servlet-name>
    <url-pattern>*.do</url-pattern>
    <url-pattern>/myservet/*</url-pattern>
</servlet-mapping>
```



## tomcat的管理配置

tomcat 的在webapp目录下提供了两个的tomcat管理的GUI:host

manager和manager。我们可以通过这两个GUI对其进行tomcat的配置。

默认情况下者两个GUI界面需要用户进行登录认证和只能在本地进行访问，如果需要使用除主机之外的ip进行访问，需要修改其文件目录下的/META-INFO/context.xml,这里我们将源有的访问的IP修改为允许192.168.0.0/16网段的主机可以访问。

/host-manager/META-INF/context.xml

```xml
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="192\.168\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

/manager/META-INFO/context.xml

```xml
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="192\.168\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

用户配置，这个时候需要在tomcat-user.xml添加用户信息 用户：tomcat，密码：123456

```xml
<tomcat-users>
  <role rolename="admin-gui"/>
  <role rolenam="admin-script"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="tomcat" password="123456" roles="admin-gui,admin-script,manager-gui,manager-script" />
</tomcat-users>
```



host-manager界面：

![image-20200424232203461](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424232203461.png)



manager界面：

![image-20200424232307424](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424232307424.png)





## JVM配置

常见的JVM配置在大多数的情况下主要是内存的分配，一般情况下JVM分配的内存不能满足我们手动配置的tomcat启动时的内存，这个时候需要对JVM内存进行重新的分配。

### JVM内存模型

内存模型图

![image-20200424232754626](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424232754626.png)

主要配置的空间是heap内存空间。

![image-20200424235946201](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200424235946201.png)

在linux关于heap内存的空间配置文件时catalina.sh

```xml
JAVA_OPTS="-server-Xms2048m -Xmx2048 -XX:MaxMetaspaceSize=256m -XX:MaxMetaspaceSize=256m -XX:survivorRatio=8"
```

| 参数                 | 含义                                                         |                         |
| -------------------- | ------------------------------------------------------------ | ----------------------- |
| -server              | 以服务器端模式运行                                           | 建议开启                |
| -Xms                 | 堆内存初始化值                                               | 建议与-xmx相同          |
| -Xmx                 | 堆内存最大值                                                 | 建议设置为可用内存的80% |
| -Xmn                 | 新生代的内存大小，建议是整个堆内存的3/8                      |                         |
| -XX:MetaspaceSize    | 元空间初始化大小                                             |                         |
| -XX:MaxMetaspaceSize | 元空间最大值                                                 | 默认是不限大小          |
| -XX:survivorRatio    | 指定eden区域与幸存区（from和to的内存大小）的大小，8表示eden 占4/5 | 不建议修改              |
| -XX:newRatio         | 新生代与老年代的比例                                         | 不建议修改              |

## tomcat 集群搭建

![image-20200425000933717](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200425000933717.png)

修改server.xml的端口,两台服务器使用不同的端口进行访问，将下面的默认配置进行修改

```xml
#原始端口配置
port="8005"
port="8080"
port="8009"
#tomcat01修改配置为
port="8015"
port="8888"
port="8019"
#tomcat02修改配置
port="8025"
port="9999"
port="8029"
```



nginx配置，一个虚拟主机

```
upstream serverpool{
	server localhost:8888;
	server localhost:9999;
}
server{
	listen 8081;
	server_name localhost;
	
	location / {
	  proxy_pass http://serverpool/;
	}
}
```



### 集群session 会话保持解决

#### session 共享方案

##### nginx的负载均衡实现

ip_hash: 用户发起请求只会到之前的tomcat上

##### session 复制

tomcat01 与tomcat 02 实现同步复制，不推荐使用，会降低tomcat的性能

##### sso单点登录

目前使用最多的企业业务的解决方案。

![image-20200425003623693](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200425003623693.png)

## tomcat的安全

### 配置安全

1）删除webapps下的所有文件，禁止tomcat管理界面

2) 注释和删除掉tomcat-user.xml 文件内的所有用户权限

3）更改关闭tomcat的指令或禁用

​		tomcat 的server,.xml 文件当中的定义了直接关闭的tomcat的管理端口（8005）

```
<servlet port="8829" shutdown="HELLO">
<servlet port="-1" shutdown="SHUTDWON">
```

### tomcat支持https

配置server.xml：

```xml
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="conf/localhost-rsa-key.pem"
                         certificateFile="conf/localhost-rsa-cert.pem"
                         certificateChainFile="conf/localhost-rsa-chain.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
```

## tomcat的性能优化

### 压力测试

#### apacheBench  (ab)

apacheBench  是一apache 基准的测试工具，对 apache server 的服务能力（每秒的请求数），不仅可以对apache进行测试，还可以用于tomcat，lighthttp，nginx，IIS等进行压测。

安装

```sh
yum install httpd-tools
```

测试

```sh
#使用ab命令进行测试
```

常见参数：

| 参数 | 含义                                         |
| ---- | -------------------------------------------- |
| -n   | 请求的总次数                                 |
| -s   | 请求超时时间                                 |
| -c   | 一次请求个数（每次请求需要发送多少次数据包） |
| -p   | 指定post请求的数据文件，json文件             |
| -t   | 请求的超时的时间                             |
| -T   | 指定post数据的context-type的头信息           |

#### JVM 优化

##### 内存优化

参考JVM配置

查看tomcat内存使用

``` sh
jmap -heap PID
```

##### 垃圾回收

jvm 垃圾回收新能有两个主要的指标：

​	吞吐量：工作时间（排除GC时间）占总时间的百分比，工作使劲并不是程序运行，还包含内存分配的时间

​	暂停时间：测试时间段内，有垃圾回收导致的应用程序停止响应次数/时间

在sun公司推出的hostSpotJVM 中，包含以下几种类型的垃圾收集器：

| 垃圾收集器 | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| 串行收集器 | 采用单线程执行所有的垃圾回收工作，适用于单核的cpu            |
| 并行收集器 | 以并行的方式执行年轻代的垃圾回收，该方式可以显著减低垃圾回收的开销（指的是多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态），适用于多处理器或者多线程硬件上运行数量较大的应用 |
| 并发收集器 | 以并发的方式执行大部分的垃圾回收工作，以缩短垃圾回收的的暂停时间。适用于那些响应时间有限与吞吐时间的应用。 |
| CMS收集器  | 并发标记清除收集器，适用于更愿意缩短来及回收的暂停时间并且负担的起与垃圾回收共享处理器资源的应用 |
| G1收集器   | 适用于大容量内存的多核服务器，可以在满足垃圾回收暂停时间的同时，以最大的可能实现吞吐量 |

​	不同的应用，对应垃圾回收的策略不同。

![image-20200425021535271](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200425021535271.png)

#### tomcat配置优化

​	主要修改的是连接器的配置。修改server.xml当中的<connector>标签当中的数据

```xml
 <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxConnections="2000"
               maxThreads= "100"
           		acceptCount="100"/>
```

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| maxConnections | 最大连接数，达到该值后，服务器不会再处理更多的请求，额外的请求会将会阻塞，直到连接数低于maxConnections，可以通过ulimit -a 查看服务器的限制，对于cpu要求高（计算类型），建议设置较低，对于cpu要求不高的可以设置在2000左右 |
| maxThreads     | 最大的线程数，要根据服务器的硬件情况，进行合理的配置         |
| acceptCount    | 最大排队的等待数，当服务器的请求连接数大于maxConnections时，此时的tomcat会将后面的请求存放在任务队列当中进行排序，accptCount值的就是在排队当中的等待的请求数量，一台tomcat服务器能够处理的最大的请求数时：maxConnections+acceptCount |

## websocket

### 简介

websocket 是html新增的协议，其目的是在浏览器与服务器之间建立一个不受双向同喜的通道，例如：服务器可以随时发消息给客户端。

为什么传统的http协议不能做到websocket的功能了。

http协议支持一个请求-响应的协议，请求必须先通过浏览器发送给服务器。服务器才能响应这个请求，再把数据发送给浏览器。这个请求过程是只是一个单向的请求。

这样，要在浏览器当中实现实时聊天，或者在线的多人游戏的话就没法实现，只能借助于flash插件。

传统的实时会话交流是通过轮询或comet，设置定时器，定时向服务器发送请求，在固定的间隔内去请求服务器。

html5推出了websocket标准，让浏览器与服务器之间建立无限制的全双工通信，任何一方都可以发送消息给对方，websocket不是全新的协议，而是建立在http协议连接之上。

![image-20200425131955296](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200425131955296.png)

websocket连接第一次连接必须通过浏览器建立。

![image-20200425132720610](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200425132720610.png)

### tomcat当中的jwebsocket

tomcat7.0.5 版本开始支持websocket ，并且实现了java websocket规范

java websocket 应用由一系列的websocketEndpoint组成，Endpoint是java对象，代表websocket连接的一端，对于服务器端，我们可以视作处理websocket消息的接口，就像servlet与http请求一样。

