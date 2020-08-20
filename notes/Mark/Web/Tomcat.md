## 概述


### Tomcat连接器的四种工作模式
1）BIO(Blocking I/O)
同步阻塞，一个 Acceptor 线程负责创建连接，把建立好连接的 socket 给到 SocketProcessor, 然后把SocketProcessor 丢到线程池里去执行。经过一系列处理后最终到达 CoyoteAdapter. 
线程模型即：
- Acceptor: 单线程
- SocketProcessor：线程池
简述：一个线程处理一个请求，在并发量高时，线程数较多，浪费资源。（Tomcat7以下默认工作模式）

![400](https://fdx321.github.io/images/%E3%80%90Tomcat%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%E3%80%91Connector-1.png)

2）NIO(New I/O)
同步非阻塞，一个 Acceptor 线程负责创建连接，把建立好连接的 socket 交给 一个Poller. 这里有多个Poller，每个Poller一个线程。Poller 负责把 socket 注册到 selector, 负责轮询 selector, 有数据可读的时候，就交给SocketProcessor丢到线程池里去执行。经过一系列处理后最终到达 CoyoteAdapter. 
线程模型即：
- Acceptor: 单线程
- Poller：多个线程
- SocketProcessor: 线程池

3）NIO2
异步阻塞，


适用场景：

BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。

NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。


HTTP
BIO:`org.apache.coyote.http11.Http11Protocol`
NIO:`org.apache.coyote.http11.Http11NioProtocol`
NIO2:`org.apache.coyote.http11.Http11Nio2Protocol`
APR:`org.apache.coyote.http11.Http11AprProtocol`




## 安装

### 安装 OpenJDK
```shell
yum list openjdk
yum install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64 -y

## 添加环境变量 /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/proile
```





## 维护


### tomcat9_JVM 性能优化
#### Server.xml配置修改
Connector是连接器，负责接收客户的请求，以及向客户端回送响应的消息。
配置文件：`tomcat/conf/server.xml`
```shell
默认配置：
<Connector port="8080" protocol="HTTP/1.1"  
              connectionTimeout="20000"  
              redirectPort="8443" /> 
修改为：
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol" # 调整工作模式为 NIO
        enableLookups="false"            # 关闭dns解析，提高响应时间
        maxThreads="10000"               # 最大线程数，默认150。增大值避免队列请求过多，导致响应缓慢。
        minSpareThreads="1000"           # 最小空闲线程数。                
        acceptCount="9000"               # 当处理请求超过此值时，将后来请求放到队列中等待。
        disableUploadTimeout="true"      # 禁用上传超时时间       
        connectionTimeout="20000"        # 连接超时，单位毫秒，0代表不限制       
        URIEncoding="UTF-8"              # URI地址编码使用UTF-8             
        redirectPort="8443" 
        compression="on"                 # 启用压缩功能
        compressionMinSize="1024"        # 最小压缩大小，单位Byte       
        useSendfile="true"
        noCompressionUserAgents="gozilla, traviata"           
 compressibleMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/javascript " #压缩的文件类型  />
```
#### JVM 优化
修改 tomcat/bin/catalina.sh （8G 内存机器参考配置）
```shell
JAVA_OPTS="-Xms4G -Xmx4G -Xmn1024m -XX:MetaspaceSize=1024M -XX:MaxMetaspaceSize=1024M 
-XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+HeapDumpOnOutOfMemoryError 
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps 
-Xloggc:/appl/gc.log -XX:CMSInitiatingOccupancyFraction=75 
-XX:+UseCMSInitiatingOccupancyOnly"

####  参数说明  ####
-Xms4G         初始分配的堆内存
-Xmx4G         最大允许分配的堆内存，这两个配成一样。
-Xmn1024m      最小允许分配的堆内存。
-XX:MetaspaceSize=1024M       初始元空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
-XX:MaxMetaspaceSize=1024M
-XX:+UseConcMarkSweepGC       并发标记清除（CMS）收集器
-XX:+CMSClassUnloadingEnabled  
-XX:+HeapDumpOnOutOfMemoryError  表示当JVM发生OOM时，自动生成DUMP文件。
-XX:HeapDumpPath=${目录}参数表示生成DUMP文件的路径，也可以指定文件名称，例如：-XX:HeapDumpPath=${目录}/java_heapdump.hprof。如果不指定文件名，默认为：java_<pid>_<date>_<time>_heapDump.hprof。
-verbose:gc   输出GC日志 ，  -XX:+PrintGC 与 -verbose:gc 是一样的，可以认为-verbose:gc 是 -XX:+PrintGC的别名.
-XX:+PrintGCDetails  打印GC详细信息
-XX:+PrintGCTimeStamps 打印gc时间戳
-XX:+PrintGCDateStamps  
-Xloggc:/appl/gc.log    定义gc日志目录
-XX:CMSInitiatingOccupancyFraction=75    是指设定CMS在对内存占用率达到75%的时候开始GC(因为CMS会有浮动垃圾,所以一般都较早启动GC);
-XX:+UseCMSInitiatingOccupancyOnly    只是用设定的回收阈值(上面指定的75%),如果不指定,JVM仅在第一次使用设定值,后续则自动调整
```

#### 关闭 AJP 端口
AJP是为 Tomcat 与 HTTP 服务器之间通信而定制的协议，能提供较高的通信速度和效率。如果tomcat前端放的是apache的时候，会使用到AJP这个连接器。如果前端由nginx做的反向代理，则无需此连接器，因此需要注销掉该连接器。
`<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->`

*补充：*默认 Tomcat 是开启了对war包的热部署的。为了防止被植入木马等恶意程序，因此我们要关闭自动部署。
`<Host name="localhost"  appBase=""    unpackWARs="false" autoDeploy="false">`


### 安全优化
1）启动安全模式
为了限制脚本的访问权限，防范 webshell 木马，建议启动时增加安全参数启动。
`Tomcat/bin/startup.sh –security`

2）删除Tomcat默认页面
删除`tomcat/webapps/`目录下的所有文件及目录。

`Tomcat/webapps/docs/`
`Tomcat/webapps/examples/`
`Tomcat/webapps/host-manager/`
`Tomcat/webapps/manager/`
`Tomcat/webapps/ROOT/`

删除 Tomcat 的 admin 控制台软件:删除`{Tomcat安装目录}\webapps`下`admin.xml`文件。
删除 Tomcat 的 Manager 控制台软件:删除`{Tomcat安装目录}\webapps`下`manager.xml`文件。

3）删除jspx文件解析
Tomcat 默认是可以解析 jspx 文件格式的后缀，解析 jspx 给服务器带来了极大的安全风险，若不需要使用 jspx 文件，建议删除对 jspx 的解析。修改 `conf/web.xml` 文件。
`# <url-pattern>*.jspx</url-pattern>	# 注释掉`

4）禁止显示错误信息
Tomcat 在程序执行失败时会有错误信息提示，可能泄漏服务器的敏感信息，需要关闭错误提示信息。可以通过指定错误页面的方式不将错误信息显示给用户，修改`tomcat/conf/web.xml`,增加如下配置项：

```shell
<error-page>
<error-code>500</error-code>
<location>/500.jsp</location>
</error-page>
```

5）禁止列目录
打开web.xml，将`<param-name>listings</param-name>` 改成`</param-name> false</param-name>` ，防止直接访问目录时由于找不到默认页面而列出目录下的文件。

6）禁用tomcat默认帐号
打开conf/tomcat-user.xml文件，将以下用户注释掉：
```shell
<!--
<role rolename="tomcat"/>
<role rolename="role1"/>
<user username="tomcat" password="tomcat" roles="tomcat"/>
<user username="both" password="tomcat" roles="tomcat,role1"/>
<user username="role1" password="tomcat" roles="role1"/>
-->
```

7）禁用不需要的http方法
对应开头，一般禁用delete，put方法，修改web.xml文件，增加如下内容：
```shell
<security-constraint>
<web-resource-collection>
<url-pattern>/*</url-pattern>
<http-method>PUT</http-method>
<http-method>DELETE</http-method>
<http-method>HEAD</http-method>
<http-method>OPTIONS</http-method>
<http-method>TRACE</http-method>
</web-resource-collection>
<auth-constraint>
</auth-constraint>
</security-constraint>
<login-config>
<auth-method>BASIC</auth-method>
</login-config>
```
*注：*web.xml文件的修改，需重启tomcat生效。

8）启用安全cookie
防止xss跨站点攻击，tomcat6开始支持此属性，此处在context.xml中添加启用配置，context.xml配置即调用时生效不需要重启tomcat
```SHELL
# http://tomcat.apache.org/tomcat-6.0-doc/config/context.html
<Context useHttpOnly="true">
```

9）修改tomcat版本信息
进入apache-tomcat目录lib下，找到catalina.jar，使用压缩工具依次找到org\apache\catalina\util下的ServerInfo.properties。打开ServerInfo.properties编辑：（去掉版本信息）如下：
```shell
server.info=Apache Tomcat
server.number=
server.built=
```

10）重定向错误页面
conf/web.xml在倒数第1行之前加如下内容：
```shell
<error-page>
     <error-code>401</error-code>
     <location>/401.htm</location>
</error-page>
<error-page>
     <error-code>404</error-code>
     <location>/404.htm</location>
</error-page>
<error-page>
     <error-code>500</error-code>
     <location>/500.htm</location>
</error-page>
```
然后在webapps\manger目录中创建相应的401.html\404.htm\500.htm文件，错误返回页也可在应用中配置，应用中配置则只在当前应用生效。




## 故障处理
中心思想：
- 尽快恢复服务，消除影响。
- 保留部分现场，再去定位问题、解决问题和复盘。
  （内存dump、线程dump、gc log等等）
![img](http://ww1.sinaimg.cn/large/005XyHIDly1ge88vduwvlj31260emq4t.jpg)


### 常见问题排查思路

#### CPU 利用率高/飙升

线上常见问题排查手册：<https://developer.aliyun.com/article/757655>
Java 应用线上问题排查思路、常用工具小结：<https://ricstudio.top/archives/java-online-question-probe>

常见原因：
- 频繁 GC
- 死循环、线程阻塞、io wait...

1）定位出问题的线程
方法1.传统方法
1. `top`定位CPU最高的进程
2. `top -Hp pid`定位改进程CPU最高的线程
3. `printf '0x%x' tid`线程ID转化为16进制
4. `jstack pid | grep tid -A 30` 找到线程堆栈

方法2.show-busy-java-threads
该脚本来自于github上一个开源项目，项目提供了很多有用的脚本，show-busy-java-threads就是其中的一个。使用这个脚本，可以直接简化方法1中的繁琐步骤。
```shell
wget --no-check-certificate https://raw.github.com/oldratlee/useful-scripts/release-2.x/bin/show-busy-java-threads
chmod +x show-busy-java-threads
./show-busy-java-threads

# 从所有运行的Java进程中找出最消耗CPU的线程（缺省5个），打印出其线程栈
# 缺省会自动从所有的Java进程中找出最消耗CPU的线程，这样用更方便
# 当然你可以手动指定要分析的Java进程Id，以保证只会显示你关心的那个Java进程的信息

show-busy-java-threads -p <指定的Java进程Id>
show-busy-java-threads -c <要显示的线程栈数>
```
方法3.arthas thread
Arthas 是由阿里巴巴开源的Java诊断工具。
https://github.com/alibaba/arthas
教程：https://arthas.aliyun.com/doc

```shell
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
$ thead -n ID
```


#### 频繁 GC
GC 流程图：
![img](http://ww1.sinaimg.cn/large/005XyHIDly1g9v43r4weaj30wa0siq93.jpg)

方法a : 查看gc 日志
方法b : jstat -gcutil 进程号 统计间隔毫秒 统计次数（缺省代表一致统计
方法c : 如果所在公司有对应用进行监控的组件当然更方便（比如Prometheus + Grafana）

开启 GC 日志：
- JDK自带的jinfo工具
- arthas的vmoption
- 启动时指定参数

获取到gc日志之后，可以上传到GC easy<https://gceasy.io/>帮助分析，得到可视化的图表分析结果.

GC 可能原因：
- survivor 区太小，对象过早进入老年代
  查看 SurvivorRatio 参数
- 大对象分配，没有足够的内存
  dump 堆，profiler/MAT 分析对象占用情况
- old 区存在大量对象
  dump 堆，profiler/MAT 分析对象占用情况

可以从full GC 的效果来推断问题，正常情况下，一次full GC应该会回收大量内存，所以 正常的堆内存曲线应该是呈锯齿形。如果你发现full gc 之后堆内存几乎没有下降，那么可以推断： 堆中有大量不能回收的对象且在不停膨胀，使堆的使用占比超过full GC的触发阈值，但又回收不掉，导致full GC一直执行。换句话来说，可能是内存泄露了。

一般GC相关的异常推断都需要涉及到内存分析，使用jmap之类的工具dump出内存快照（或者 Arthas的heapdump）命令，然后使用MAT、JProfiler、JVisualVM等可视化内存分析工具。

#### 线程池异常

**异常说明：**
Java 线程池以有界队列的线程池为例，当新任务提交时，如果运行的线程少于 corePoolSize，则创建新线程来处理请求。如果正在运行的线程数等于 corePoolSize 时，则新任务被添加到队列中，直到队列满。当队列满了后，会继续开辟新线程来处理任务，但不超过 maximumPoolSize。当任务队列满了并且已开辟了最大线程数，此时又来了新任务，ThreadPoolExecutor 会拒绝服务。

**常见问题和原因：**
1. 下游服务 响应时间（RT）过长
2. 数据库慢 sql 或者数据库死锁
3. Java 代码死锁
   `jstack –l pid | grep -i –E 'BLOCKED | deadlock'`



