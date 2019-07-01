---
title: jdk常用工具总结
date: 2019-02-19 17:21:55
tags:
- jdk
- jvm
categories:
- java
---
因为线上排查问题的需要,另一个也是基本技能,对java常用工具做一个小结.
#### 常用的工具
- jar
- jarsigner		签名工具,可对未签名的apk进行签名
- java  
- javac 	        编译工具
- javadoc         生成doc文档工具
- javafxpackager	打包工具,可以生成.exe文件
- javap         	查看.class文件的字节码
- javapackager 	jdk1.7后自带的一个打包工具，可以生成本地exe安装包
- jcmd          	打印一个 Java 进程的类，线程以及虚拟机信息。适合用在脚本中。使用 jcmd - h 来查看使用方法。
- jconsole 	  	提供 JVM 活动的图形化展示，包括线程使用，类使用以及垃圾回收（GC）信息
- jdb             调试工具,比较牛逼
- jdeps         	包依赖分析工具
- jhat          	帮助分析内存堆存储
- jinfo         	访问 JVM 系统属性，同时可以动态修改这些属性。
- jmap          	提供 JVM 内存使用信息，适用于脚本中。
- jmc             性能监控工具
- jps             显示当前所有java进程pid的命令
- jstack        	提供 Java 进程内的线程堆栈信息。
- jstat         	提供 Java 垃圾回收以及类加载信息。
- jstatd          一个rmi的server应用，用于监控jvm的创建和结束，并且提供接口让监控工具（如visualvm）可以远程连接到本机的jvms
- jvisualvm     	监控 JVM 的可视化工具，剖析运行中的应用程序，分析 JVM 堆存储。

从常用的开始讲起
#### javap
[三目运算符](/2019/02/15/java/三目运算符的坑/)

#### jcmd
待补充

#### jconsole
和jvisualvm功能相似,但jvisualvm功能更强
```
 添加启动参数 -Djava.rmi.server.hostname=192.168.0.83 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9111 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
```
![](http://ww1.sinaimg.cn/mw690/007DdbdYly1g0c06hz75cj30pg106djg.jpg)
![](http://ww1.sinaimg.cn/mw690/007DdbdYly1g0c07fv6r1j31bi0wudog.jpg)

#### jinfo
主要作用是实时查看和调整JVM配置参数
##### 查看jvm参数
- jinfo -flag <name> PID
用法：jinfo -flag <name>  PID
例如：
jinfo -flag MaxMetaspaceSize 18348，得到结果-XX:MaxMetaspaceSize=536870912，即MaxMetaspaceSize为512M
jinfo -flag ThreadStackSize 18348，得到结果-XX:ThreadStackSize=256，即Xss为256K

##### 修改jvm参数
如果是布尔类型的JVM参数: jinfo -flag [+|-]<name>  PID，enable or disable the named VM flag
如果是数字/字符串类型的JVM参数    jinfo  -flag <name>=<value> PID，to set the named VM flag to the given value
##### 实时修改jvm参数
标记为manageable可以修改
java -XX:+PrintFlagsInitial | grep manageable
![](http://ww1.sinaimg.cn/mw690/007DdbdYly1g0c0unt4n4j313w0fudkv.jpg)

#### jmap
堆內存分析工具
- jmap -heap pid:查看堆使用情况
- jmap -histo pid：查看堆中对象数量和大小
- jmap -dump:format=b,file=heapdump pid：将内存使用的详细情况输出到文件
- jmap -dump:file=./dump.mdump pid
jdk8 的堆分析
![](http://ww1.sinaimg.cn/mw690/007DdbdYly1g0ea6y82r9j30i815oai0.jpg)

#### jps
显示当前所有java线程
- 用法:
 >╰─λ jps -help
    usage: jps [-help]
           jps [-q] [-mlvV] [<hostid>]
    Definitions:
    <hostid>:      <hostname>[:<port>]
- jps -q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数
- jps -m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
- **jps -l** 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
- **jps -v** 输出传递给JVM的参数
- jps -V 输出线程id和java命令
>[root@k ~]# jps -V
   21547 Jps
   14799 jar

#### jstack
显示线程堆栈信息
主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待。
>pid转换成16进制 printf "%x\n" pid
jstack -l PID > jstack_pid.log
![](http://ww1.sinaimg.cn/mw690/007DdbdYly1g0eb1l59k7j31dq0hi443.jpg)

#### jstat
` jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]] `
` jstat -gcutil pid 2000 10 `
` interval 毫秒  `
- jstat –class<pid> : 显示加载class的数量，及所占空间等信息
- jstat -compiler <pid>显示VM实时编译的数量等信息
- jstat -gc <pid>: 可以显示gc的信息，查看gc的次数，及时间
- jstat -gccapacity <pid>:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小
- jstat -gcutil <pid>:统计gc信息
- jstat -gcnew <pid>:年轻代对象的信息
- jstat -gcnewcapacity<pid>: 年轻代对象的信息及其占用量
- jstat -gcold <pid>：old代对象的信息
- jstat -gcoldcapacity <pid>: old代对象的信息及其占用量
- jstat -gcpermcapacity<pid>: perm对象的信息及其占用量
- jstat -printcompilation <pid>：当前VM执行的信息

