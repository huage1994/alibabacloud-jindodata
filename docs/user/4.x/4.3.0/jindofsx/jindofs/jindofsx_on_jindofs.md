# 阿里云 OSS-HDFS 服务（JindoFS 服务）透明缓存加速

本文主要介绍 JindoFSx 支持阿里云 OSS-HDFS 服务（JindoFS 服务）透明缓存加速的使用方式。

## 前提条件：
* 已部署 JindoFSx 存储加速系统

关于如何部署 JindoFSx 存储加速系统，请参考 [部署 JindoFSx 存储加速系统](/docs/user/4.x/4.3.0/jindofsx/deploy/deploy_jindofsx.md)

* 已部署 JindoSDK

关于如何部署 JindoSDK，请参考 [部署 JindoSDK](/docs/user/4.x/4.3.0/jindofsx/deploy/deploy_jindosdk.md)

## 服务端配置 OSS-HDFS 服务 AccessKey
在 `JINDOFSX_CONF_DIR` 文件夹下修改配置 jindofsx.cfg 文件, 配置缓存加速的 OSS-HDFS 服务 bucket 对应的`Access Key ID`、`Access Key Secret`、`Endpoint`，
并更新到所需要节点上（Namespace Service 和 Storage Service 所在节点）。

```
[jindofsx-common]
jindofsx.oss.bucket.XXX.accessKeyId = xxx
jindofsx.oss.bucket.XXX.accessKeySecret = xxx
jindofsx.oss.bucket.XXX.endpoint = cn-xxx.oss-dls.aliyuncs.com

jindofsx.oss.bucket.YYY.accessKeyId = xxx
jindofsx.oss.bucket.YYY.accessKeySecret = xxx
jindofsx.oss.bucket.YYY.endpoint = cn-xxx.oss-dls.aliyuncs.com

jindofsx.oss.bucket.YYY.user = xxx #Storage Service 访问 OSS-HDFS 服务使用的用户名
```

说明: XXX 和 YYY 为 OSS-HDFS 服务 bucket 名称。

## 重启 JindoFSx 服务
重启 JindoFSx 服务，使得配置的 OSS-HDFS 服务 bucket 的`Access Key ID`、`Access Key Secret`、`Endpoint`生效。在 master 节点执行以下脚本。
```
cd jindofsx-x.x.x
sh sbin/stop-service.sh
sh sbin/start-service.sh
```

## 配置 JindoSDK

* 配置实现类

将 OSS-HDFS 服务实现类配置到 Hadoop 的`core-site.xml`中。

```xml
<configuration>
    <property>
        <name>fs.AbstractFileSystem.oss.impl</name>
        <value>com.aliyun.jindodata.oss.OSS</value>
    </property>

    <property>
        <name>fs.oss.impl</name>
        <value>com.aliyun.jindodata.oss.JindoOssFileSystem</value>
    </property>

    <property>
        <name>fs.xengine</name>
        <value>jindofsx</value>
    </property>
</configuration>
```

* 配置 AccessKey

将 OSS-HDFS 服务 的 Bucket 对应的`Access Key ID`、`Access Key Secret`等预先配置在 Hadoop 的`core-site.xml`中。
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
</configuration>
```
JindoSDK 还支持更多的 AccessKey 的配置方式，详情参考 [JindoSDK OSS-HDFS 服务 Credential Provider 配置](/docs/user/4.x/4.3.0/jindofs/security/jindosdk_credential_provider_dls.md)。

* 配置 OSS-HDFS 服务 Endpoint

访问 OSS Bucket 上 OSS-HDFS 服务需要配置 Endpoint（`cn-xxx.oss-dls.aliyuncs.com`），与 OSS 对象接口的 Endpoint（`oss-cn-xxx-internal.aliyuncs.com`）不同。JindoSDK 会根据配置的 Endpoint 访问 OSS-HDFS 服务或 OSS 对象接口。

使用 OSS-HDFS 服务时，推荐访问路径格式为：`oss://<Bucket>.<Endpoint>/<Object>`

如: `oss://mydlsbucket.cn-shanghai.oss-dls.aliyuncs.com/Test`。

这种方式在访问路径中包含 OSS-HDFS 服务的 Endpoint，JindoSDK 会根据路径中的 Endpoint 访问对应的 OSS-HDFS 服务接口。 JindoSDK 还支持更多的 Endpoint 配置方式，详情参考 [OSS-HDFS 服务 Endpoint 配置](/docs/user/4.x/4.3.0/jindofs/configuration/jindosdk_endpoint_configuration.md)。


* 配置 JindoFSx Namespace 服务地址

配置在 Hadoop 的`core-site.xml`中。
```xml
<configuration>
    <property>
        <!-- Namespace service 地址 -->
        <name>fs.jindofsx.namespace.rpc.address</name>
        <value>headerhost:8101</value>
    </property>
</configuration>
```
若使用高可用 Namespace, 请参考 [高可用 JindoFSx Namespace 配置和使用](/docs/user/4.x/4.3.0/jindofsx/deploy/deploy_raft_ns.md)

* 启用缓存加速功能

启用缓存会利用本地磁盘对访问的热数据块进行缓存，默认状态为禁用，即所有OSS读取都直接访问OSS上的数据。

可根据情况将以下配置添加到 Hadoop 的`core-site.xml`中。
```xml
<configuration>

    <property>
        <!-- 数据缓存开关 -->
        <name>fs.jindofsx.data.cache.enable</name>
        <value>true</value>
    </property>

    <property>
        <!-- 元数据缓存开关 -->
        <name>fs.jindofsx.meta.cache.enable</name>
        <value>false</value>
    </property>

    <property>
        <!-- 小文件缓存优化开关 -->
        <name>fs.jindofsx.slice.cache.enable</name>
        <value>false</value>
    </property>

    <property>
        <!-- 内存缓存开关 -->
        <name>fs.jindofsx.ram.cache.enable</name>
        <value>false</value>
    </property>

    <property>
        <!-- 短路读开关 -->
        <name>fs.jindofsx.short.circuit.enable</name>
        <value>true</value>
    </property>

</configuration>
```

完成以上配置后，作业读取 JindoFS 上的数据后，会自动缓存到 JindoFSx 存储加速系统中，作业访问 JindoFS 的方式无需做任何修改, 后续访问相同的数据就能够命中缓存。

注意：此配置为客户端配置，不需要重启 JindoFSx 服务。

## 磁盘空间水位控制
缓存启用后，JindoFSx 服务会自动管理本地缓存备份，通过水位清理本地缓存，请您根据需求配置一定的比例用于缓存。JindoFSx 后端基于 OSS，可以提供海量的存储，但是本地盘的容量是有限的，因此 JindoFSx 会自动淘汰本地较冷的数据备份
我们提供了`storage.watermark.high.ratio`和`storage.watermark.low.ratio`两个参数来调节本地存储的使用容量，值均为0～1的小数，表示使用磁盘空间的比例。
修改 `JINDOFSX_CONF_DIR` 文件夹下修改配置 jindofsx.cfg 文件, 并更新至所有服务节点。
```
[jindofsx-storage]# Storage service 配置
storage.watermark.high.ratio=0.8 # 表示磁盘使用量的上水位比例，每块数据盘的 JindoFSx 数据目录占用的磁盘空间到达上水位即会触发清理。默认值：0.8。
storage.watermark.low.ratio=0.6 # 表示使用量的下水位比例，触发清理后会自动清理冷数据，将JindoFS数据目录占用空间清理到下水位。默认值：0.6。
```

说明: 您可以通过设置上水位比例调节期望分给 JindoFSx 的磁盘空间，下水位必须小于上水位，设置合理的值即可。
注意: 配置完成后需重启 JindoFSx 服务。

## 参数调优
JindoSDK 包含一些高级调优参数，配置方式以及配置项参考文档 [JindoSDK 配置项列表](../configuration/jindosdk_configuration_list.md)