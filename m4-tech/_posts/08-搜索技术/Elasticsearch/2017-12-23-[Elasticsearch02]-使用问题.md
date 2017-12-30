---
layout: blog
title: Elasticsearch01-使用问题
tags: [java, Elasticsearch,]
desc: Elasticsearch常见问题汇总
---

#### 1.启动es报错
```
[root@bobyang apps]# cd elasticsearch-6.0.1/bin/
[root@bobyang bin]# ./elasticsearch
Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 986513408 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /root/apps/elasticsearch-6.0.1/bin/hs_err_pid801.log
```
##### 原因：
由于elasticsearch6.0默认分配jvm空间大小为1g，jvm参数过大，服务器内存不够
##### 解决办法：
临时解决办法，启动时指定jvm参数
```
ES_JAVA_OPTS="-Xms512m -Xmx512m" ./bin/elasticsearch
```
或
修改config下jvm.options文件

```
-Xms512m
-Xmx512m
```

##### 参考链接：
http://blog.csdn.net/u013368491/article/details/69110163?utm_source=itdadao&utm_medium=referral


#### 2.使用root账号启动es后

```
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:134) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:121) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:69) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:134) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:85) ~[elasticsearch-6.0.1.jar:6.0.1]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:104) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:171) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:322) ~[elasticsearch-6.0.1.jar:6.0.1]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:130) ~[elasticsearch-6.0.1.jar:6.0.1]
        ... 6 more
```
##### 原因：
这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考虑， 
建议创建一个单独的用户用来运行ElasticSearch

##### 解决办法：
创建elsearch用户组及elsearch用户
```
groupadd elsearch
useradd elsearch -g elsearch -p elasticsearch
```
更改elasticsearch文件夹及内部文件的所属用户及组为elsearch:elsearch
```
cd /opt
chown -R elsearch:elsearch  elasticsearch-6.0.1
```
切换到elsearch用户再启动
```
su elsearch 
cd elasticsearch/bin
./elasticsearch
```

##### 参考链接：
https://my.oschina.net/topeagle/blog/591451?fromerr=mzOr2qzZ


#### 3.切换至elsearch用户执行启动命令
```
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch 
```
##### 原因：
该账号不是root权限，无法cd到指定目录，多半是你把程序解压在了/root/目录下，随便移动到其他位置即可
##### 解决办法：
将安装包移至/opt/即可
##### 参考链接：
http://blog.csdn.net/yao970953039/article/details/62047755


#### 4.max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
##### 具体现象：

```
ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
```

##### 解决办法：
```
解决：修改切换到root用户修改配置limits.conf 添加下面两行

命令:vi /etc/security/limits.conf

*        hard    nofile           65536
*        soft    nofile           65536
# 退出重新登录即可，不行就重启服务器。
```
##### 参考链接：
https://www.cnblogs.com/sloveling/p/elasticsearch.html

#### 5.max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
##### 具体错误：
```
ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
[2017-12-08T17:12:34,931][INFO ][o.e.n.Node               ] [SUagXLp] stopping ...
[2017-12-08T17:12:34,980][INFO ][o.e.n.Node               ] [SUagXLp] stopped
[2017-12-08T17:12:34,980][INFO ][o.e.n.Node               ] [SUagXLp] closing ...
[2017-12-08T17:12:35,011][INFO ][o.e.n.Node               ] [SUagXLp] closed
```

##### 解决办法：

```
#解决：切换到root用户修改配置sysctl.conf
vi /etc/sysctl.conf 
#添加下面配置：
v.max_map_count=655360
#或
sudo sysctl -w vm.max_map_count=262144
#并执行命令：
sysctl -p
```
##### 参考链接：
https://www.cnblogs.com/sloveling/p/elasticsearch.html
#### 6.system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
##### 原因：
因为Centos6不支持SecComp，而ES5.2.1默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动。详见 ：https://github.com/elastic/elasticsearch/issues/22899
##### 解决办法：
在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:
```
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```
##### 参考链接：
http://blog.csdn.net/liangzhao_jay/article/details/56840941






