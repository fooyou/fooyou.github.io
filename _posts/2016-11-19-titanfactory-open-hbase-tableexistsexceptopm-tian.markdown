---
layout: post
title: TitanFactory open titan-hbase-es.properties TableExistsExcptions 的解决方法 
category: Document
tags: bigdata
date: 2016-11-19 17:11:37
published: true
summary: 用titan + hbase 伪分布式搭建好了使用 GraphOfTheGodsFactory.load(graph) 也成功了，可以在创建其他图时手动删除了 hbase 数据库目录，但再次 TitanFactory.open('conf/titan-hbase-es.properties') 时竟然弹出了一大串的异常...
image: pirates.svg
comment: true
---

上篇说到了使用 titan + hbase 搭建为分布式图数据库引擎，并且使用 GraphOfTheGodsFactory 成功加载到了 hbase 里。现在需要把 GraphOfTheGodsFactory 的这个图删掉，然后用其他测试数据创建图。

## 问题

一开始我找到 /tmp/ 目录下把 /hbase 目录删除了，然后在 gremlin 中重新 open 新图，却发生了如下异常：

```
gremlin> graph = TitanFactory.open('conf/titan-hbase-es.properties')
16:54:34 WARN  com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager  - Unexpected exception during getDeployment()
java.lang.RuntimeException: com.thinkaurelius.titan.diskstorage.TemporaryBackendException: Temporary failure in storage backend
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getDeployment(HBaseStoreManager.java:351)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getFeatures(HBaseStoreManager.java:389)
	at com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration.<init>(GraphDatabaseConfiguration.java:1321)
	at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:94)
	at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:62)
	at com.thinkaurelius.titan.core.TitanFactory$open.call(Unknown Source)
	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:45)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:110)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:122)
	at groovysh_evaluate.run(groovysh_evaluate:3)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Interpreter.evaluate(Interpreter.groovy:69)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Groovysh.execute(Groovysh.groovy:185)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Shell.leftShift(Shell.groovy:119)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.ShellRunner.work(ShellRunner.groovy:94)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.super$2$work(InteractiveShellRunner.groovy)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:90)
	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:324)
	at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1207)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuperN(ScriptBytecodeAdapter.java:130)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuper0(ScriptBytecodeAdapter.java:150)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.work(InteractiveShellRunner.groovy:123)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.ShellRunner.run(ShellRunner.groovy:58)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.super$2$run(InteractiveShellRunner.groovy)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:90)
	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:324)
	at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1207)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuperN(ScriptBytecodeAdapter.java:130)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuper0(ScriptBytecodeAdapter.java:150)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.run(InteractiveShellRunner.groovy:82)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.apache.tinkerpop.gremlin.console.Console.<init>(Console.groovy:144)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.apache.tinkerpop.gremlin.console.Console.main(Console.groovy:303)
Caused by: com.thinkaurelius.titan.diskstorage.TemporaryBackendException: Temporary failure in storage backend
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.ensureTableExists(HBaseStoreManager.java:759)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getLocalKeyPartition(HBaseStoreManager.java:556)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getDeployment(HBaseStoreManager.java:347)
	... 45 more
Caused by: org.apache.hadoop.hbase.TableExistsException: org.apache.hadoop.hbase.TableExistsException: titan
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at org.apache.hadoop.ipc.RemoteException.instantiateException(RemoteException.java:95)
	at org.apache.hadoop.ipc.RemoteException.unwrapRemoteException(RemoteException.java:79)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.translateException(RpcRetryingCaller.java:207)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.translateException(RpcRetryingCaller.java:221)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:121)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:90)
	at org.apache.hadoop.hbase.client.HBaseAdmin.executeCallable(HBaseAdmin.java:3327)
	at org.apache.hadoop.hbase.client.HBaseAdmin.createTableAsync(HBaseAdmin.java:603)
	at org.apache.hadoop.hbase.client.HBaseAdmin.createTable(HBaseAdmin.java:494)
	at org.apache.hadoop.hbase.client.HBaseAdmin.createTable(HBaseAdmin.java:428)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseAdmin0_98.createTable(HBaseAdmin0_98.java:99)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.createTable(HBaseStoreManager.java:792)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.ensureTableExists(HBaseStoreManager.java:756)
	... 47 more
Caused by: org.apache.hadoop.hbase.ipc.RemoteWithExtrasException: org.apache.hadoop.hbase.TableExistsException: titan
	at org.apache.hadoop.hbase.master.handler.CreateTableHandler.prepare(CreateTableHandler.java:137)
	at org.apache.hadoop.hbase.master.HMaster.createTable(HMaster.java:1868)
	at org.apache.hadoop.hbase.master.HMaster.createTable(HMaster.java:2110)
	at org.apache.hadoop.hbase.protobuf.generated.MasterProtos$MasterService$2.callBlockingMethod(MasterProtos.java:44117)
	at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:2195)
	at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:104)
	at org.apache.hadoop.hbase.ipc.FifoRpcScheduler$1.run(FifoRpcScheduler.java:74)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

	at org.apache.hadoop.hbase.ipc.RpcClient.call(RpcClient.java:1450)
	at org.apache.hadoop.hbase.ipc.RpcClient.callBlockingMethod(RpcClient.java:1654)
	at org.apache.hadoop.hbase.ipc.RpcClient$BlockingRpcChannelImplementation.callBlockingMethod(RpcClient.java:1712)
	at org.apache.hadoop.hbase.protobuf.generated.MasterProtos$MasterService$BlockingStub.createTable(MasterProtos.java:42717)
	at org.apache.hadoop.hbase.client.HConnectionManager$HConnectionImplementation$5.createTable(HConnectionManager.java:1970)
	at org.apache.hadoop.hbase.client.HBaseAdmin$2.call(HBaseAdmin.java:607)
	at org.apache.hadoop.hbase.client.HBaseAdmin$2.call(HBaseAdmin.java:603)
	at org.apache.hadoop.hbase.client.RpcRetryingCaller.callWithRetries(RpcRetryingCaller.java:114)
	... 55 more
16:54:35 WARN  com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager  - Unexpected exception during getDeployment()
java.lang.RuntimeException: com.thinkaurelius.titan.diskstorage.TemporaryBackendException: Temporary failure in storage backend
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getDeployment(HBaseStoreManager.java:351)
	at com.thinkaurelius.titan.diskstorage.hbase.HBaseStoreManager.getFeatures(HBaseStoreManager.java:389)
	at com.thinkaurelius.titan.diskstorage.Backend.getStandaloneGlobalConfiguration(Backend.java:438)
	at com.thinkaurelius.titan.graphdb.configuration.GraphDatabaseConfiguration.<init>(GraphDatabaseConfiguration.java:1322)
	at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:94)
	at com.thinkaurelius.titan.core.TitanFactory.open(TitanFactory.java:62)
	at com.thinkaurelius.titan.core.TitanFactory$open.call(Unknown Source)
	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:45)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:110)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:122)
	at groovysh_evaluate.run(groovysh_evaluate:3)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Interpreter.evaluate(Interpreter.groovy:69)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Groovysh.execute(Groovysh.groovy:185)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.Shell.leftShift(Shell.groovy:119)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.ShellRunner.work(ShellRunner.groovy:94)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.super$2$work(InteractiveShellRunner.groovy)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:90)
	at groovy.lang.MetaMethod.doMethodInvoke(MetaMethod.java:324)
	at groovy.lang.MetaClassImpl.invokeMethod(MetaClassImpl.java:1207)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuperN(ScriptBytecodeAdapter.java:130)
	at org.codehaus.groovy.runtime.ScriptBytecodeAdapter.invokeMethodOnSuper0(ScriptBytecodeAdapter.java:150)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.work(InteractiveShellRunner.groovy:123)
	at org.codehaus.groovy.vmplugin.v7.IndyInterface.selectMethod(IndyInterface.java:215)
	at org.codehaus.groovy.tools.shell.ShellRunner.run(ShellRunner.groovy:58)
	at org.codehaus.groovy.tools.shell.InteractiveShellRunner.super$2$run(InteractiveShellRunner.groovy)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.codehaus.groovy.reflection.CachedMethod.invoke(CachedMethod.java:90)
    ...
```

最重要的看到的异常是 `TabelExistsException: titan`。

## 解决方法

原来在删除掉 Hbase 的存储目录后，还需要清空 Zookeeper 里的 /hbase 目录，方法如下：

如果 Zookeeper 是使用的 Hbase 自带的，先确保 Hbase 运行，然后：

```sh
$ ./bin/hbase zkcli
zkcli> rmr /hbase
```

然后重启 hbase 就 OK 了。
