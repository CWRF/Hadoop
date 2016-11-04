参考链接<http://blog.fens.me/hadoop-mapreduce-recommend/>
##如何运行hadoop程序
按照这篇博客直接运行
```
hadoop fs -cat /user/hdfs/recommend/step1/part-00000
```
终端提示，链接localhost失败。原因在于没有启动hadoop服务。参考链接<http://stackoverflow.com/questions/10918269/unable-to-check-nodes-on-hadoop-connection-refused>

于是切换到hadoop的路径，在终端执行：
```
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# bin/hadoop namenode -format
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# bin/start-all.sh
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# jps
root@sun-virtual-machine:/usr/lib/hadoop-common-release-1.1.2# bin/hadoop dfsadmin -report
Configured Capacity: 19945680896 (18.58 GB)
Present Capacity: 11507994639 (10.72 GB)
DFS Remaining: 11507965952 (10.72 GB)
DFS Used: 28687 (28.01 KB)
DFS Used%: 0%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0

-------------------------------------------------
Datanodes available: 1 (1 total, 0 dead)

Name: 127.0.0.1:50010
Decommission Status : Normal
Configured Capacity: 19945680896 (18.58 GB)
DFS Used: 28687 (28.01 KB)
Non DFS Used: 8437686257 (7.86 GB)
DFS Remaining: 11507965952(10.72 GB)
DFS Used%: 0%
DFS Remaining%: 57.7%
Last contact: Thu Nov 03 21:48:52 CST 2016
```
通过启动hadoop，并检查hadoop运行状态，可知hadoop运行起来了，执行程序命令：
```
root@sun-virtual-machine:/home/sun/workspace/movie# hadoop fs -cat /usr/lib/hdfs/recommend/step1/part-00000
cat: File does not exist: /usr/lib/hdfs/recommend/step1/part-00000
```
存在的文件夹说不存在。参考<http://hadoop.apache.org/docs/r1.0.4/cn/mapred_tutorial.html>可以知道如何在运行hadoop的项目
