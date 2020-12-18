# Tomcat架构剖析

## Tomcat系统架构

软件以及源码下载地址：

https://tomcat.apache.org/download-80.cgi



#### 浏览器访问服务器的流程

- 用户发起请求
- 浏览器发起TCP连接请求
- 服务器接收请求并建立连接
- 浏览器生成HTTP格式的数据包
- 发送请求数据包
- 解析HTTP格式数据包
- 执行请求
- 生成HTTP格式的数据包
- 服务器发送响应数据包给浏览器
- 浏览器解析HTTP格式的数据包
- 浏览器呈现静态数据给用户

注意浏览器访问服务器使用的是HTTP协议，HTTP是应用层协议，用于定义数据通信的格式，具体的数据传输使用的是TCP/IP协议

#### Tomcat请求处理流程

Tomcat是一个HTTP服务器（能够接收并处理HTTP请求，因此是一个HTTP服务器）



Tomcat请求处理流程：HTTP服务器接收到请求之后，把请求交给Servlet容器来处理，Servlet容器通过Servlet接口调用业务处理类。



Servlet容器和Servlet接口共同组成了Servlet规范。



Tomcat既按照Servlet规范要求去实现了Servlet容器，同时也具有HTTP服务器的功能，因此Tomcat有两个身份：

- HTTP服务器
- Servlet容器



#### Tomcat Servlet容器处理流程

![image-20201216203812085](C:\Users\hgvgh\AppData\Roaming\Typora\typora-user-images\image-20201216203812085.png)

当请求某个URL资源时：

- HTTP服务器会把请求信息封装为ServletRequest对象；
- 根据请求的URI资源映射定位对应的处理Servlet；
- 如果Servlet还未被加载， 则使用反射机制创建该Servlet，并调用Servlet的init方法来完成初始化；
- 调用该Servlet的service方法来处理请求，并将请求结果封装为ServletResponse对象；
- 把  ServletResponse对象返回给HTTP服务器，HTTP服务器将数据返回客户端。

#### Tomcat整体架构

两个核心组件：

- Connector连接器组件：负责和客户端的Socket通信，网络字节流和request、response的转化，对应角色：HTTP服务器

- Container容器组件：加载和管理Servlet，并处理请求逻辑，：对应角色：Servlet容器，



#### Tomcat连接器组件Coyote

Coyote是Tomcat连接器组件的名称。客户端通过Coyote与服务器建立连接，发送请求并返回响应。

- Coyote封装了底层的网络通信(Socket请求以及请求响应）
- Coyote使Catalina容器（容器组件）与具体的请求协议以及IO操作方式完全解耦；
- Coyote将Socket输入转换为Request对象，进一步封装后交由Catalina容器进行处理，处理完成后，Catalina通过Coyote提供的Response对象将结果写入流中；
- Coyote负责具体协议（应用层）和IO（传输层）相关内容

![image-20201216205250667](C:\Users\hgvgh\AppData\Roaming\Typora\typora-user-images\image-20201216205250667.png)

Tomcat支持的应用层协议：

- HTTP/1.1
- AJP
- HTTP/2

Tomcat支持的IO模型：

- NIO
- NIO2
- APR



Coyote的组件：

- EndPoint：Coyote通信端点，即通信监听的接口，是具体Socket接收和发送处理器，是对传输层的抽象，因此EndPoint是用来实现TCP/IP协议的；
- Processor：Coyote协议处理接口，Processor接收来自EndPoint的Socket读取字节流解析成Tomcat的Request对象，并通过Adaptor将其交由容器处理，Processor是对应用层协议的抽象；
- ProtocolHandler：Coyote接口，包括EndPoint和Processor两部分
- Adaptor：负责将Tomcat Request对象转换为ServletRequest对象



#### Tomcat Servlet容器Catalina

Tomcat是由一系列可配置的组件构成的Web容器，而Catalina是Tomcat的Servlet容器。

可以认为整个Tomcat就是一个Catalina实例，Tomcat启动的时候就会初始化这个实例，Catalina实例通过加载server.xml完成其他实例的创建，创建并管理一个Server,Server创建并管理多个服务，每个服务又可以有多个Connector和一个Container。

- Catalina：负责解析Tomcat的配置文件，以此来创建服务器Server组件并进行管理；
- Server：
- Service：是Server内部的组件，一个Server包含多个Service,它将若干个Connector组件绑定到一个Container；
- Container:容器，负责处理用户的Servlet请求，并返回对象给web用户的模块



Container组件包含：

- Engine：表示整个Catalina的Servlet引擎，用来管理多个虚拟站点，一个Service最多只能有一个Engine,但是一个引擎包含多个Host；
- Host：代表一个虚拟主机，一个HOST可以包含多个Context；
- Context：表示一个web应用程序，一个web应用可以包含多个Wrapper；
- Wrapper：表示一个Servletr，Wrapper作为容器中的最底层，不能包含子容器。



## Tomcat服务器核心配置

配置文件：conf/server.xml

核心配置组件结构：

```
<server>
	<listener>
	</listener>
	<GlobalNamingResources>
	</GlobalNamingResources>
	<service>
		<Executor />
		<Connector/>
		<Engine>
			<Host>
			</Host>
		</Engine>
	</service>
</server
```





<Executor />定义的是共享线程池，给Connector使用。

配置示例：

```

<Executor name="commonThreadPool"
namePrefix="thread-exec-"
maxThreads="200"
minSpareThreads="100"
maxIdleTime="60000"
maxQueueSize="Integer.MAX_VALUE"
prestartminSpareThreads="false"
threadPriority="5"
className="org.apache.catalina.core.StandardThreadExecutor"/>
```

相关参数介绍：

- name：线程池名，方便connector组件使用；

- namePrefix：所创建的每个线程的名称前缀，一个单独的线程名称为namePrefix+threadNumber ；

- maxThreads：线程池中最大线程数；

- minSpareThreads：活跃线程数，也就是核心线程数，这些线程不会被销毁，会一直存在；

- maxIdleTime：线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000，单位毫秒，也就是一分钟；

- maxQueueSize：暂不需要改，表示在被执行前最大线程排队数目，默认为int的最大值；

  

使用：

```
<Connector port="8080" 
protocol="HTTP/1.1"
executor="commonThreadPool" 
maxThreads="1000"
minSpareThreads="100"
acceptCount="1000"
maxConnections="1000"
connectionTimeout="20000"
compression="on"
compressionMinSize="2048"
disableUploadTimeout="true"
redirectPort="8443"
URIEncoding="UTF-8" /> 
```





host标签：

- name：名称
- appBase：可执行程序存放目录（可以是相对路径，也可以是绝对路径）
- unpackWARS：是否解压war包，默认为true
- autoDeploy：是否自动部署



<context>标签可以将访问路径映射到磁盘上：

```
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
			
		<Context docBase="C:\Users\hgvgh\Desktop\lagou\ROOT" path="/web3"></Context> 

        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
```



访问：http://localhost:8080/web3/时，就会去加载C:\Users\hgvgh\Desktop\lagou\ROOT下面的内容。注意的是这里的文件路径不要有中文。

