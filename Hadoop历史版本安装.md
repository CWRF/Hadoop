要重新安装Hadoop-1.1.2的历史版本，来满足Mahout-0.8版本的依赖要求。
##Hadoop下载页面
<http://www.apache.org/dyn/closer.cgi/hadoop/common/>

但是没有Hadoop-1.1.2版本，这个版本在github上<https://github.com/apache/hadoop-common/releases/tag/release-1.1.2>
##用源代码构建Hadoop环境
```
tar xvf release-1.1.2.tar.gz
```
解压后的文件放在/usr/lib/hadoop-common-release-1.1.2。查看hadoop目录
```
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# ls -l
total 644
drwxrwxr-x  2 root root   4096 3月   6  2013 bin
-rw-rw-r--  1 root root 120025 3月   6  2013 build.xml
-rw-rw-r--  1 root root 467130 3月   6  2013 CHANGES.txt
drwxrwxr-x  2 root root   4096 3月   6  2013 conf
drwxrwxr-x  2 root root   4096 3月   6  2013 ivy
-rw-rw-r--  1 root root  10525 3月   6  2013 ivy.xml
drwxrwxr-x  4 root root   4096 3月   6  2013 lib
-rw-rw-r--  1 root root  13366 3月   6  2013 LICENSE.txt
-rw-rw-r--  1 root root    101 3月   6  2013 NOTICE.txt
-rw-rw-r--  1 root root   1366 3月   6  2013 README.txt
-rw-rw-r--  1 root root   7815 3月   6  2013 sample-conf.tgz
drwxrwxr-x 16 root root   4096 3月   6  2013 src
```
在根目录下面，没有hadoop-*.jar的各种类库，也没有依赖库。

执行Ant进行编译

从Ant官方网站下载Ant, <http://ant.apache.org/bindownload.cgi>

解压在/usr/lib/apache-ant-1.9.7，设置环境变量，检测环境变量配置成功
```
root@sun-virtual-machine:/etc# vi environment
root@sun-virtual-machine:/etc# source environment 
root@sun-virtual-machine:/etc# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/lib/apache-ant-1.9.7/bin
root@sun-virtual-machine:/etc# ant
Buildfile: build.xml does not exist!
Build failed
```
使用ant编译hadoop，等待几分钟
```
# 安装编译用的库
~ sudo apt-get install autoconf
~ sudo apt-get install libtool

# 用Ant编译Hadoop(切换到hadoop的目录)
~ cd /usr/lib/hadoop-common-release-1.1.2
~ ant
```
写到这里我停下了好久。不管怎么用ant命令在hadoop-common-release-1.1.2里执行，就是没有jar包出来。

通过这个链接<http://www.cnblogs.com/yjmyzz/p/compile-hadoop-2_6_0-source-code-in-centos_x64.html>的“五.编译hadoop”，我才知道ant是根据build.xml编译的，打开build.xml，我才看到java version是1.6。。。而我用的是1.8版本的java。果断下载jdk1.6，重新设置环境变量。终于编译出jar包了。

真的是hadoop历史版本啊，还要用历史版本的java编译。当然，这主要告诉我，要看好build.xml的内容，里面已经说了用1.6版本，我用1.8版本自然会出错。

经历2个小时排错后的结果，珍贵的jar包：
```
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# ll build
total 4308
drwxr-xr-x 13 root root    4096 10月 30 23:19 ./
drwxrwxr-x  9 root root    4096 10月 30 20:35 ../
drwxr-xr-x  3 root root    4096 10月 30 23:19 ant/
drwxr-xr-x  2 root root    4096 10月 30 20:35 c++/
drwxr-xr-x  3 root root    4096 10月 30 23:19 classes/
drwxr-xr-x 13 root root    4096 10月 30 23:19 contrib/
drwxr-xr-x  2 root root    4096 10月 30 23:19 empty/
drwxr-xr-x  2 root root    4096 10月 30 20:35 examples/
-rw-r--r--  1 root root     407 10月 30 23:19 hadoop-client-1.1.3-SNAPSHOT.jar
-rw-r--r--  1 root root 4041575 10月 30 23:19 hadoop-core-1.1.3-SNAPSHOT.jar
-rw-r--r--  1 root root     410 10月 30 23:19 hadoop-minicluster-1.1.3-SNAPSHOT.jar
-rw-r--r--  1 root root  306839 10月 30 23:19 hadoop-tools-1.1.3-SNAPSHOT.jar
drwxr-xr-x  4 root root    4096 10月 30 20:35 ivy/
drwxr-xr-x  3 root root    4096 10月 30 20:35 src/
drwxr-xr-x  6 root root    4096 10月 30 20:35 test/
drwxr-xr-x  3 root root    4096 10月 30 23:19 tools/
drwxr-xr-x  9 root root    4096 10月 30 20:35 webapps/
```
发现hadoop-*.jar都是hadoop-*-1.1.3-SNAPSHOT.jar结尾的，合理的解释就是：上一个版本的release就是下一个版本的SNAPSHOT。

我们修改一下build.xml文件31行，把包名改成hadoop-*-1.1.2.jar，再重新ant
```
~ vi build.xml

~ rm -rf build
~ ant
```
```
total 4304
drwxr-xr-x 13 root root    4096 10月 31 08:53 ./
drwxrwxr-x  9 root root    4096 10月 31 08:52 ../
drwxr-xr-x  3 root root    4096 10月 31 08:53 ant/
drwxr-xr-x  2 root root    4096 10月 31 08:52 c++/
drwxr-xr-x  3 root root    4096 10月 31 08:53 classes/
drwxr-xr-x 13 root root    4096 10月 31 08:53 contrib/
drwxr-xr-x  2 root root    4096 10月 31 08:53 empty/
drwxr-xr-x  2 root root    4096 10月 31 08:52 examples/
-rw-r--r--  1 root root     398 10月 31 08:53 hadoop-client-1.1.2.jar
-rw-r--r--  1 root root 4035813 10月 31 08:53 hadoop-core-1.1.2.jar
-rw-r--r--  1 root root     401 10月 31 08:53 hadoop-minicluster-1.1.2.jar
-rw-r--r--  1 root root  306839 10月 31 08:53 hadoop-tools-1.1.2.jar
drwxr-xr-x  4 root root    4096 10月 31 08:52 ivy/
drwxr-xr-x  3 root root    4096 10月 31 08:52 src/
drwxr-xr-x  6 root root    4096 10月 31 08:52 test/
drwxr-xr-x  3 root root    4096 10月 31 08:53 tools/
drwxr-xr-x  9 root root    4096 10月 31 08:52 webapps/
```
##快速配置hadoop环境脚本
1. 配置环境变量
2. Hadoop的3个配置文件
3. 创建Hadoop目录
4. 配置hostname和hosts
5. 生成SSH免登陆(说明链接<http://blog.csdn.net/wh_19910525/article/details/7433164>)
6. 第一次启动格式化HDFS
7. 启动Hadoop的服务

下面的脚本要写在一起
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/lib/apache-ant-1.9.7/bin:/usr/lib/jdk1.8.0_111/bin:/usr/lib/apache-maven-3.3.9/bin:/usr/lib/apache-tomcat-8.5.6/bin"

JAVA_HOME=/usr/lib/jdk1.8.0_111
```
/conf在/hadoop-common-release-1.1.2下
```
~ vi conf/core-site.xml
<configuration>
<property>
<name>fs.default.name</name>
<value>hdfs://localhost:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/usr/lib/hadoop/tmp</value>
</property>
<property>
<name>io.sort.mb</name>
<value>256</value>
</property>
</configuration>


~ vi conf/hdfs-site.xml
<configuration>
<property>
<name>dfs.data.dir</name>
<value>/usr/lib/hadoop/data</value>
</property>
<property>
<name>dfs.replication</name>
<value>1</value>
</property>
<property>
<name>dfs.permissions</name>
<value>false</value>
</property>
</configuration>

~ vi conf/mapred-site.xml
<configuration>
<property>
<name>mapred.job.tracker</name>
<value>hdfs://localhost:9001</value>
</property>
</configuration>

~ mkdir /home/conan/hadoop/data
~ mkdir /home/conan/hadoop/tmp
~ sudo chmod 755 /home/conan/hadoop/data/
~ sudo chmod 755 /home/conan/hadoop/tmp/

~ sudo hostname master
~ sudo vi /etc/hosts
127.0.0.1       localhost
192.168.1.210   sun-virtual-machine

~ ssh-keygen -t rsa
~ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

~ bin/hadoop namenode -format
~ bin/start-all.sh
```
检测hadoop运行状态
```
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# jps
5136 DataNode
5794 NameNode
6051 JobTracker
5268 SecondaryNameNode
5492 TaskTracker
6233 Jps

root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# bin/hadoop dfsadmin -report
Configured Capacity: 0 (0 KB)
Present Capacity: 0 (0 KB)
DFS Remaining: 0 (0 KB)
DFS Used: 0 (0 KB)
DFS Used%: �%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0

-------------------------------------------------
Datanodes available: 0 (0 total, 0 dead)
```
###参考链接
<http://blog.fens.me/hadoop-history-source-install/>

<http://blog.csdn.net/hitwengqi/article/details/8008203>
