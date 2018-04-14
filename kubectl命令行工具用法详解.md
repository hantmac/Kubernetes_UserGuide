### kubectl 命令行用法详解

***
##### kubectl作为客户端CLI工具可以让用户通过命令的方式对kubernetes集群进行操作。本章节对kubectl子命令和用法详细说明。
#### kubeclt 用法概述
* kubectl命令轮廓
* kubectl [command] [TYPE] [NAME] [flags]
其中command、TYPE、NAME、flags的含义是：
> 1.command:用于操作集群资源对象的命令，例如create、delete、describe、get、apply等。

> 2.TYPE:资源对象类型，区分大小写，能以单数形式、复数形式或者简写形式表示。例如以下三种TYPE是等价的。
----
> kubectl get pod pod1

> kubectl get pods pod1

> kubeclt get po pod1
----
> 3. NAME: 资源对象的名称，区分大小写、如果不指定名称将返回属于TYPE的所有对象列表，例如kubectl get pods将返回所有pod列表。

> 4. flags:kubectl子命令的可选参数。
* 以下是kubectl可操作的资源对象

| 资源对象名称 | 缩写|
| ------------|------|
| componentstatuses|cs|
| daemonsets | ds |
| deployments | |
| events | ev |
| horizontalpodscalers | hpa |
| ingresses | ing |
| jobs | |
| limitranges | limits |
| nodes | no |
| namespaces | ns |
| pods | po |
| persistentvolumes | pv |
| persistentvolumesclaims | pvc |
| resourcequotas | quota |
| replicationcontrollers | rc |
| secrets | |
| serviceaccounts | |
| services | svc |

* 多种资源和对象的组合
> * 获取多个pod的信息：kubectl get pods pod1 pod2

> * 获取多种对象的信息：kubectl get pod/pod1 rc/rc1

> * 同时应用多个ymal文件：kubectl get pod -f pod1.ymal -f pod2.ymal 或者是：kubectl create -f pod1.yaml -f rc1.yaml -f service1.ymal

#### kubectl 子命令详解

##### kubectl的子命令非常丰富，包括了对资源对象的创建、删除、查看、修改、配置、运行等。
![image](https://ws4.sinaimg.cn/large/006tNc79gy1fqcdygcy98j30ik0i4dta.jpg)
![image](https://ws4.sinaimg.cn/large/006tNc79gy1fqcdzipzvfj30io0mqnim.jpg)

#### kubectl 一些操作示例

> * 创建资源对象:kubectl create -f my-service.yaml -f my-rc.ymal

> * 查看资源对象：1.查看所有pod列表：kubectl get pods 2.查看rc和service列表：kubectl get rc，service

> * 描述资源对象：1.描述node详细信息：kubectl describe nodes <node-name>  2.显示pod的详细信息：kubectl describe pods/<pod-name> 

> * 显示由RC管理的pod信息：kubectl describe pods <rc-name>

> * 删除资源对象： 1.基于ymal的名称删除pod:kubectl delete -f pod.ymal 2. 删除所有包含某个label的pod和serveice： kubectl delete pods,services -l name=<label-name> 3.删除所有pod: kubectl delete pods --all

> 执行容器命令：kubectl exec <pod-name> date 默认执行的是第一个容器 也可以指定容器： kubectl exec <pod-name> -c <container-name> date
> * 登录某个容器： Kubectl exec -ti <pod-name> -c <container-name> /bin/bash
 > * 查看容器的日志：kubectl logs <pod-name>