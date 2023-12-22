# 问题描述
首次对 broker 进行启停时，未发现异常。再次启动 broker 时却启动失败，追踪到`~/logs/rocketmq~/store.log`日志报错，示例如下：
``` log
java.lang.IllegalStateException: java.lang.reflect.InaccessibleObjectException: Unable to make public void jdk.internal.ref.Cleaner.clean() accessible: module java.base does not "exports jdk.internal.ref" to unnamed module @3590fc5b
 at org.apache.rocketmq.store.MappedFile$1.run(MappedFile.java:105) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at java.base/java.security.AccessController.doPrivileged(Native Method) ~[na:na]
 at org.apache.rocketmq.store.MappedFile.invoke(MappedFile.java:98) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.MappedFile.clean(MappedFile.java:94) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.MappedFile.cleanup(MappedFile.java:434) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.ReferenceResource.release(ReferenceResource.java:63) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.ReferenceResource.shutdown(ReferenceResource.java:47) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.MappedFile.destroy(MappedFile.java:442) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.index.IndexFile.destroy(IndexFile.java:89) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.index.IndexService.load(IndexService.java:71) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.store.DefaultMessageStore.load(DefaultMessageStore.java:195) ~[rocketmq-store-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.broker.BrokerController.initialize(BrokerController.java:256) ~[rocketmq-broker-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.broker.BrokerStartup.createBrokerController(BrokerStartup.java:218) ~[rocketmq-broker-4.5.0.jar:4.5.0]
 at org.apache.rocketmq.broker.BrokerStartup.main(BrokerStartup.java:58) ~[rocketmq-broker-4.5.0.jar:4.5.0]
Caused by: java.lang.reflect.InaccessibleObjectException: Unable to make public void jdk.internal.ref.Cleaner.clean() accessible: module java.base does not "exports jdk.internal.ref" to unnamed module @3590fc5b
 at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:340) ~[na:na]
 at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:280) ~[na:na]
 at java.base/java.lang.reflect.Method.checkCanSetAccessible(Method.java:198) ~[na:na]
 at java.base/java.lang.reflect.Method.setAccessible(Method.java:192) ~[na:na]
 at org.apache.rocketmq.store.MappedFile$1.run(MappedFile.java:102) ~[rocketmq-store-4.5.0.jar:4.5.0]
 ... 13 common frames omitted

```
# 原因分析
在Java9之后引入了模块化的概念，是将类型和资源封装在模块中，并仅导出其他模块要访问其公共类型的软件包。如果模块中的软件包未导出或打开，则表示模块的设计人员无意在模块外部使用这些软件包。一般这种报错包含以下三种：

| operation | jvm启动参数 |
|--|--|
|exports|--add-exports|
|opens|--add-opens|
|requires|--add-reads|


# 解决方法
修改`runbroker.sh`，对启动参数增加一项：
``` bash
--add-exports java.base/jdk.internal.ref=ALL-UNNAMED
```