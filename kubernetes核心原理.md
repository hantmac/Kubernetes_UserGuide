### kubernete API Server原理分析
---
* 核心功能：提供k8s各类资源对象（pod,RC,serfvice等）的增、删、改、查以及Watch等httpRest接口，是集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。
* 集群管理的API入口
* 资源配额控制入口
* 集群安全机制
#### kubernete API server 概述
* 通过kube-apiserver进程提供服务，运行在master节点上。在master节点上可以直接使用curl命令，得到JSON方式返回的kubernetes API的版本信息。可使用kubectl proxy 启动内部代理来拒绝某个资源的外部访问
* 也可通过编程方式调用kubernete API server，具体场景可分两种：
* 1. pod里的用户进程调用kubernete API ：kubernetes API service 本身就是一个service，名字是""kubernetes",其cluster ip是cluster IP地址池的第一个，所服务的端口是HTTPS的443端口
* 2. 开发基于kubernetes的管理平台。调用kubernetes API 来实现pos,service,RC等资源的图形化管理界面。
#### 独特的kubercetes proxy API接口
* kubernetes proxy API接口是一种特殊的rest接口，其作用是代理rest请求，即kubernetes API server 把收到的rest请求转发到Node上的Kubelet守护进行的rest端口上，由该进程相应。
* pod的proxy接口作用与意义：集群之外访问pod容器的服务（http服务），可以使用proxy API。

### 集群功能模块之间的通信
集群内部API server有三种交互场景
* 1. kubelet与API server交互。每个node上的kubelet每隔一段时间就会调用一次API server的REST接口报告自身状态。
* 2. kube-controller-manager进程与API server的交互。controller中的node controller模块通过APIserver的watch接口实时监控node的信息，并处理。
* 3. kube-scheduller与API server的交互。scheduller通过API server的watch接口监听新建pod副本信息，执行pod的调度逻辑，最终将pod绑定到目标节点。
* 缓存机制用来缓解各模块对API server的访问加压力

### Controller-manager原理分析
Controller manager是集群内部的管理控制中心，负责集群内部资源管理，某个node宕机时，其会及示范县并执行自动化修复流程。
* 顾名思义，Controller manager是各种Controller的管理者。
* 1. ReplicationController：Reolication Controller的核心作用的确保在任何时候集群中一个RC所关联的POD副本数量保持预设值。RC中的pod模板就像一个模具，pod可以通过修改label实现脱离RC的管控。删除一个RC不会影响其所创建的pod。要快速删除RC上的pod，可以将RC副本数量设置为0。总结RC使用场景为：(1).重新调度（2）弹性伸缩（3）滚动更新
* 2. nodeController:node controller通过API server实时获取node的相关信息。
* 3. ResourceQuatoController：指定资源对象在任何时间都不会超量占用系统物理资源。目前kubernetes支持如下三个层次的资源配额管理：
        * （1）容器级别，可以对CPU和memory进行限制
        * （2）Pod级别，可以对一个pod内所有容器的可用资源进行限制。
        * （3）namespace级别，为namespace（多租户）资源限制：pod数量；RC数量；service数量；secret数量；可持有的PVC

* 4. NamespaceController: 用户可以通过API server创建新的Namespace并保存到etcd中，Namespace controller 定时通过API server读取这些Namespaces信息。
* 5. service Controller与Endpoint COntroller：EndPoints表示了一个service对应的所有Pod的访问地址，而Endpoints controller 就是负责生成和维护所有Endpoints对象的Controller。每个node上的kube-proxy进行获取每个service的#Endpoint，实现了service的负载均衡功能。service Controller是内部集群与外部平台之间的接口控制器。service Controller监听service的变化，如果是一个LB类型的service，Controller确保外部云平台上该service对应的LB实例被相应的创建，删除以及更新路由转发表。
### scheduler原理分析
kubernetes schelduler 在整个系统中承上启下：承上是指负责接收Controllermanager创建的新pod安排node；启下是指安置工作完成后目标node上的kubelet服务进程接管后继工作，负责pod生命周期的下半阶段。
