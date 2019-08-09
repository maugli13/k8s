## Kubernetes cheatsheet - part I

#### Simple commands examples:

##### - kubectl version

This will display two different versions: the version of the local kubectl tool, as well
as the version of the Kubernetes API server

##### Example:

```shell
[root@node4 ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```

##### - kubectl get componentstatuses

A good way to verify that cluster is generally healthy

##### Example:
```
[root@node4 ~]# kubectl get componentstatuses
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-2               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```
- The** controller-manager** is responsible for running various controllers that regulate behavior in the cluster: for example, ensuring that all of the replicas of a service are available and healthy.
- The **scheduler** is responsible for placing different pods onto different nodes in the cluster.
- The **etcd** server is the storage for the cluster where all of the API objects are stored.

##### - kubectl get nodes [-o wide]

List out all of the nodes in our cluster

##### Example:
```
[root@node4 ~]# kubectl get nodes
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    <none>   31h   v1.15.0
node2   Ready    <none>   31h   v1.15.0
node3   Ready    <none>   31h   v1.15.0
node4   Ready    master   31h   v1.15.0
node5   Ready    master   32h   v1.15.0
```
##### - kubectl describe nodes [node-name]

basic information about the node/nodes with useful details

##### - kubectl get namespaces

Default kubernetes namespace is kube-system

##### Example:
```
[root@node4 ~]# kubectl get namespaces
NAME              STATUS   AGE
default           Active   32h
ingress-nginx     Active   32h
kube-node-lease   Active   32h
kube-public       Active   32h
kube-system       Active   32h
```
##### - kubectl get daemonSet --namespace=kube-system kube-proxy

Kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster

#####Example:
```
[root@node4 ~]# kubectl get daemonSets --namespace=kube-system kube-proxy
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-proxy   5         5         5       5            5           beta.kubernetes.io/os=linux   32h
```
#####- kubectl get deployments --namespace=kube-system coredns

Kubernetes also runs a DNS server, which provides naming and discovery for the services that are defined in the cluster

#####Example:
```
[root@node4 ~]# kubectl get deployments --namespace=kube-system coredns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           32h
```
#####- kubectl get services --namespace=kube-system coredns

Kubernetes service that performs load-balancing for the DNS server. This information will be populated in /etc/resolv.conf file in each container

#####Example:
```
[root@node4 ~]# kubectl get services --namespace=kube-system coredns
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
coredns   ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   32h
```
#####- kubectl get deployments --namespace=kube-system kubernetes-dashboard

Kubernetes UI service

#####Example:
```
[root@node4 ~]# kubectl get deployments --namespace=kube-system kubernetes-dashboard
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-dashboard   1/1     1            1           32h
```
#####- kubectl get services --namespace=kube-system kubernetes-dashboard

Kubernetes UI internal IP address and ports
```
[root@node4 ~]# kubectl get services --namespace=kube-system kubernetes-dashboard
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes-dashboard   ClusterIP   10.233.40.188   <none>        443/TCP   32h
```
#####- kubectl cluster-info [dump]

Display cluster information
Dump - collections of debugs

#####Example:
```
[root@node4 ~]# kubectl cluster-info
Kubernetes master is running at https://192.168.166.84:6443
coredns is running at https://192.168.166.84:6443/api/v1/namespaces/kube-system/services/coredns:dns/proxy
kubernetes-dashboard is running at https://192.168.166.84:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
```
####Advanced command example:

#####- kubectl get ``<resource name>`` ``<object>``

By default this command will provide human readable limited output.
To get detailed output -o json option can be used
For precise output for particular object from json output --template option can be used, which is based on jsonpath query language

##### Example:
```
[root@node4 ~]# kubectl get pods --namespace=kube-system kubernetes-dashboard-7c547b4c64-cw2fn -o jsonpath --template={.status.hostIP} | xargs
192.168.166.83
```
####Debugging commands:

#####- kubectl logs <pod-name> [ -c <container-name> ] [ -f to strem the logs ]

Output logs from pod or container

#####Example:
```
[root@node4 ~]# kubectl logs --namespace=kube-system kubernetes-dashboard-7c547b4c64-cw2fn
```
#####- kubectl exec ``--namespace=<namespace name>`` `` -it <pod-name> -- <command to execute in pod or container>``

Execute some command in pod or container, or bash to interact with container.
In order to exit correctly from container: Ctrl + p -> Ctrl + q

#####Example:
```
[root@node4 ~]# kubectl exec --namespace=kube-system  -it nginx-proxy-node1 -- /bin/bash --version | grep bash.*version
GNU bash, version 4.4.12(1)-release (x86_64-pc-linux-gnu)
```
#####- kubectl cp ``<pod-name>:/path/to/remote/file`` ``/path/to/local/file``

Copy from container to local machine or opposite
