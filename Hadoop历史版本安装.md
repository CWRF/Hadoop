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
