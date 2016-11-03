在linux里配置maven还是比较轻松的，输入mvn查看是否安装成功。
```
root@sun-virtual-machine:/home/sun# mvn
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.281 s
[INFO] Finished at: 2016-11-01T22:33:07+08:00
[INFO] Final Memory: 5M/31M
[INFO] ------------------------------------------------------------------------
[ERROR] No goals have been specified for this build. You must specify a valid lifecycle phase or a goal in the format <plugin-prefix>:<goal> or <plugin-group-id>:<plugin-artifact-id>[:<plugin-version>]:<goal>. Available lifecycle phases are: validate, initialize, generate-sources, process-sources, generate-resources, process-resources, compile, process-classes, generate-test-sources, process-test-sources, generate-test-resources, process-test-resources, test-compile, process-test-classes, test, prepare-package, package, pre-integration-test, integration-test, post-integration-test, verify, install, deploy, pre-clean, clean, post-clean, pre-site, site, post-site, site-deploy. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/NoGoalSpecifiedException
```
之后可以在命令行里创建maven项目，但是比较麻烦。不如在eclipse里简单。参考链接<http://www.cnblogs.com/mephisto/p/4859564.html>

按照这个参考链接操作，在添加hadoop依赖的时候，配置好pom.xml之后，遇到了很麻烦的缺少jar包的问题。
```
Description	Resource	Path	Location	Type
Archive for required library: '/home/sun/.m2/repository/xml-apis/xml-apis/1.3.04/xml-apis-1.3.04.jar' in project 'second' cannot be read or is not a valid ZIP file	second		Build path	Build Path Problem
```
解决办法是：复制“xml-apis-1.3.04.jar”，在google搜索jar下载，在终端创建提示的文件夹，把jar包添加进去。

在导出jar包的时候，会出现permission denied而无法导出jar包，这是因为文件夹权限的问题。解决的办法是把文件夹的权限改为'777'。

终端执行yarn的时候遇到了问题，发现直接‘apt-get install yarn’不行，在网上的查询如下：
###MapReduce和YARN是什么关系？

答：YARN并不是下一代MapReduce（MRv2），下一代MapReduce与第一代MapReduce（MRv1）在编程接口、数据处理 引擎（MapTask和ReduceTask）是完全一样的， 可认为MRv2重用了MRv1的这些模块，不同的是资源管理和作业管理系统，MRv1中资源管理和作业管理均是由JobTracker实现的，集两个功能 于一身，而在MRv2中，将这两部分分开了， 其中，作业管理由ApplicationMaster实现，而资源管理由新增系统YARN完成，由于YARN具有通用性，因此YARN也可以作为其他计算 框架的资源管理系统，不仅限于MapReduce，也是其他计算框架，比如Spark、Storm等， 通常而言，我们一般将运行在YARN上的计算框架称为“X on YARN”，比如“MapReduce On YARN”, "Spark On YARN"，“Storm On YARN”等。

Hadoop 2.0由三个子系统组成，分别是HDFS、YARN和MapReduce，其中，YARN是一个崭新的资源管理系统，而MapReduce则只是运行在 YARN上的一个应用，如果把YARN看成一个云操作系统，那么MapReduce可认为是运行在这个操作系统上的App。

yarn的配置参考<https://segmentfault.com/a/1190000000626280>

 到这里真是尴尬了，虚拟机里我安装的hadoop是1.1.2，而yarn是hadoop2.0新加的。。。配置好了xml文件，去发现hadoop1.1.2的bin文件里没有start-yarn.sh。现在只能重新配置2.0的hadoop了


