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
安装成功
##用maven搭建hadoop环境
1、用Maven创建一个标准化的Java项目

2、导入项目到eclipse

3、增加hadoop依赖，修改pom.xml

4、下载依赖

5、从Hadoop集群环境下载hadoop配置文件

6、配置本地host
###用Maven创建一个标准化的Java项目
这里和在eclipse里创建maven项目是一样的。要有group id，artifact id，version和package
```
root@sun-virtual-machine:/usr/local/workspace/hadoopTest# mvn archetype:generate -DgroupId=cn.luxh.app -DartifactId=myapp -DpackageName=cn.luxh.app.myapp -Dversion=1.0-SNAPSHOT 

[INFO] Using following parameters for creating project from Archetype: schemacrawler-archetype-maven-project:14.10.05
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: cn.luxh.app
[INFO] Parameter: artifactId, Value: myapp
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: cn.luxh.app
[INFO] Parameter: packageInPathFormat, Value: cn/luxh/app
[INFO] Parameter: package, Value: cn.luxh.app
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: groupId, Value: cn.luxh.app
[INFO] Parameter: artifactId, Value: myapp
[INFO] project created from Archetype in dir: /usr/local/workspace/hadoopTest/myapp
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:47 min
[INFO] Finished at: 2016-11-01T23:50:19+08:00
[INFO] Final Memory: 13M/48M
[INFO] ------------------------------------------------------------------------
```
进入项目，执行mvn命令
```
root@sun-virtual-machine:/usr/local/workspace/hadoopTest# cd myapp
root@sun-virtual-machine:/usr/local/workspace/hadoopTest/myapp# mvn clean install

[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myapp ---
[INFO] Building jar: /usr/local/workspace/hadoopTest/myapp/target/myapp-1.0-SNAPSHOT.jar
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ myapp ---
[INFO] Installing /usr/local/workspace/hadoopTest/myapp/target/myapp-1.0-SNAPSHOT.jar to /root/.m2/repository/cn/luxh/app/myapp/1.0-SNAPSHOT/myapp-1.0-SNAPSHOT.jar
[INFO] Installing /usr/local/workspace/hadoopTest/myapp/pom.xml to /root/.m2/repository/cn/luxh/app/myapp/1.0-SNAPSHOT/myapp-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.079 s
[INFO] Finished at: 2016-11-01T23:59:40+08:00
[INFO] Final Memory: 15M/37M
[INFO] ------------------------------------------------------------------------
```
导入eclipse，参考<http://blog.csdn.net/lushuaiyin/article/details/6951196>
