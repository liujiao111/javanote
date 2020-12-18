# Nginx核心知识

## Nginx基础知识

#### Nginx定义

Nginx是一个高性能的HTTP和反向代理服务器。

#### Nginx作用

- HTTP服务器（WEB服务器）
- 反向代理服务器
- 负载均衡服务器
- 动静分离

#### Nginx特点

- 跨平台
- 占用内存少
- 并发能力强（能支持5W个并发）
- 上手容易，配置简单
- 稳定性强，宕机概率低

#### Nginx安装

- Nginx下载地址：http://nginx.org 

- 安装Nginx依赖：

```
  yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel 
```

- 解压Nginx压缩包：`tar -xvf nginx-1.17.8.tar `

- 进入软件执行命令：

```
  cd nginx-1.17.8 
  ./conﬁgure 
  make 
  make install
```

- 执行完毕后会在/usr/local/目录下生成nginx目录

- 进入sbin目录中，执行启动命令

```
  cd nginx/sbin ./nginx
```

- 访问服务器80端口（Nginx默认监听80端口）

Nginx其他命令：

- ./nginx启动nginx
- ./nginx -s stop终止nginx进程（当然也可以找到nginx进程号，用kill -9 pid来结束进程）
- ./nginx -s reload（修改了配置文件后使用此命令来重新加载nginx.conf配置文件）

## Nginx核心配置文件

`conf/nginx.conf`

主要包括三块：

- 全局块
  - 运行用户
  - worker进程数量，通常设置为cpu数量相同
  - 日志信息
  - PID文件位置
- events块：影响nginx服务器和用户的网络连接
  - worker_connections：单个worker进程的最大并发连接数
- http块：（自行配置的核心）：
  - 虚拟主机的配置
  - 监听端口的配置

## Niginx应用场景之反向代理

- 正向代理

  在访问目标网站的时候，代理服务器将请求转发到目标网站，获取目标网站的响应并返回给客户端。

- 反向代理

  客户端发送请求到反向代理服务器时，由反向代理服务器选择目标服务器提供服务并获取响应最终将数据返回给客户端。



Nginx反向代理配置实践一：将访问9003端口的请求转发到百度首页。

- 修改Nginx配置文件，并重新加载

```
   server {
          listen       9003;
          server_name  localhost;
  
          location / {
                   proxy_pass http://www.baidu.com;
             # root   html;
              #index  index.html index.htm;
          }
```

  

  

  Nginx反向代理配置实践而：将访问9003端口的/abc请求转发到百度首页；将访问9003端口的/def请求转发到taobao首页。

- 配置：

```
  location /abc {
         proxy_pass http://www.baidu.com;
  }
  location /def {
           proxy_pass http://www.taobao.com/;
  }
```


## Nginx应用场景之负载均衡

当一个请求发送后，Nginx反向代理服务器根据请求找到对应的一个服务器来处理请求，这叫做反向代理，如果目标服务器有多台，找哪台目标服务器来处理当前请求，这一过程就是负载均衡。

负载均衡就是为了解决高负载的问题。



同一个url请求，对应多个服务，Nginx处理的过程就是负载均衡。



Nginx提供的负载均衡策略：

- 轮询：

```
upstream lagouServer{
    server 111.229.248.243:8080;
    server 111.229.248.243:8082; 
}
location /abc {
	proxy_pass http://lagouServer/;
}

```

- 权重：权重越高，被调用的几率更高。（用于服务器性能不均匀的场景）

```
  upstream lagouServer{
      server 111.229.248.243:8080 weight=1;
      server 111.229.248.243:8082 weight=2;
  }
  location /abc {
  	proxy_pass http://lagouServer/;
  }
```

- ip_hash（每个客户端的请求会被分配到同一个目标服务器处理，能解决session共享问题）

```
  upstream lagouServer{  
      ip_hash;  
      server 111.229.248.243:8080;
      server 111.229.248.243:8082;
  }
  location /abc {
  	proxy_pass http://lagouServer/;
  }
```

  其中`lagouServer`是集群的名字。

  

## Nginx应用场景之动静分离

将静态资源和动态资源交由不同的服务器来处理，充分发挥各自服务器的优势。

擅长处理静态资源的服务器软件：Apache、Nginx。



```
location /static/ {
	root staticData;
}
```



## Nginx底层进程机制剖析

Nginx启动后以daemon多进程方式在后台运行，包括一个Master和多个Worker进程

- Master进程：管理worker进程：
  - 接收外界信号向各worker进程发送信号（./nginx -s reload）
  - 监控worker进程运行状态，当worker进程异常退出后，Master进程会自动重新启动新的worker进程。
- worker进程：负责具体处理网络请求，各个进程之间互相独立，一个请求只能被一个worker处理。worker进程数是可以设置的，一般设置与机器cpu核数相同。



./nginx -s reload热加载后的worker进程的PID会发生改变，旧worker进程会关闭，建立新的worker，因此PID会发生变化



Nginx进程请求：

- master进程创建之后，会建立好需要监听的socket，然后再从master进程fork出多个worker进程；
- Nginx使用互斥锁来保证只有一个woker进程能够处理请求，拿到互斥锁的那个进程注册listened读事件，在读事件里调用accept接收该请求，然后进行处理



Nginx多进程模型好处：

- 每个worker进程模型都是独立的，不需要枷锁，节省开销；
- 每个worker进程都是独立 的，互不影响，一个异常结束，其他的也能继续提供服务；
- 多进程模型为reload热部署机制提供了支撑。



附录：

启动tomcat cd /home/tomcat/bin

sh startup.sh

Cannot find bin/catalina.sh 
The file is absent or does not have execute permission 

This file is needed to run this program 

原因： 是因为没有权限

解决 ： chmod 777 *.sh 