### 使用前须知
* 请参考 [Jindo DistCp 介绍](jindo_distcp_overview.md) 文章内容进行环境适配和工具包下载
* 如您在使用过程中遇到问题可参考 [Jindo DistCp 问题排查指南](jindo_distcp_QA_pre.md) 进行解决，也可 [新建 ISSUE](https://github.com/aliyun/alibabacloud-jindodata/issues/new) 向我们反馈

### 1、拷贝数据到 JindoFS Block 模式上
您可以使用如下命令将 HDFS 上的目录拷贝到 JindoFS Block 模式上
```shell
hadoop jar jindo-distcp-3.7.2.jar --src /data --dest jfs://ns/dir/ --ossKey yourkey --ossSecret yoursecret --ossEndPoint oss-cn-xxx.aliyuncs.com --parallelism 10
```
 *(ECS 环境推荐使用内网 ossEndPoint，即 oss-cn-xxx-internal.aliyuncs.com)*

* --src：HDFS 的源路径
* --dest：block 模式下后端 oss 的目标路径
* --ossKey：block 模式下后端 AccessKey
* --ossSecret：block 模式下后端 AccessSecret
* --ossEndPoint：block 模式下后端 endpoint 信息，可以公网或者内网的 endpoint
* --parallelism：任务并发大小，根据集群资源可调整

### 2、增量拷贝文件
如果 Distcp 任务因为各种原因中间失败了，而此时您想进行断点续传，只Copy剩下未Copy成功的文件。或者源端文件新增了部分文件，此时需要您在进行上一次 Distcp 任务完成后进行如下操作：
##### 使用 --update 命令，获得增量的文件列表
```shell
hadoop jar jindo-distcp-3.7.2.jar --src /data --dest jfs://ns/dir/ --ossKey yourkey --ossSecret yoursecret --ossEndPoint oss-cn-xxx.aliyuncs.com --update --parallelism 20
```
 *(ECS 环境推荐使用内网 ossEndPoint，即 oss-cn-xxx-internal.aliyuncs.com)*

如果所有文件都传输完成，则会提示如下信息。

```
INFO distcp.JindoDistCp: Jindo DistCp job exit with 0.
```
### 3、YARN 队列及带宽选择
如您需要对 DistCp 作业使用的 YARN 队列和带宽进行限定，可用如下命令
```shell
hadoop jar jindo-distcp-3.7.2.jar --src /data --dest jfs://ns/dir/ --ossKey yourkey --ossSecret yoursecret --ossEndPoint oss-cn-xxx.aliyuncs.com --queue yarnQueue --bandwidth 100 --parallelism 10
```
 *(ECS 环境推荐使用内网 ossEndPoint，即 oss-cn-xxx-internal.aliyuncs.com)*

* --queue：指定 YARN 队列的名称
* --bandwidth：指定单机限流带宽的大小，单位 MB

### 4、免密及密钥固定存储
通常您需要将 OSS AccessKey/AccessSecret/EndPoint 信息写在参数里，但是Jindo DistCp可以将 其预先写在 Hadoop 的`core-site.xml`文件里 ，以避免使用时多次填写的问题。
如果您需要保存 OSS 的相关信息，您需要将以下信息保存在core-site.xml中，可省去在命令行里填写 ossKey/ossSecret/ossEndPoint 等信息。
```xml
<configuration>
    <property>
        <name>fs.oss.accessKeyId</name>
        <value>xxx</value>
    </property>

    <property>
        <name>fs.oss.accessKeySecret</name>
        <value>xxx</value>
    </property>

    <property>
        <name>fs.oss.endpoint</name>
        <!-- 阿里云 ECS 环境下推荐使用内网 OSS Endpoint，即 oss-cn-xxx-internal.aliyuncs.com -->
        <value>oss-cn-xxx.aliyuncs.com</value>
    </property>
</configuration>
```
### 5、其他功能
如您需要其他使用其他功能，请参考
* [Jindo DistCp 进行数据迁移的使用说明](jindo_distcp_how_to.md)
* [Jindo DistCp 场景化使用指南](jindo_distcp_scenario_guidance.md)

