---
layout: post
title:  "玩一玩 Kubernetes cluster"
date:   2023-01-19 11:19 +0800
categories: k8s
---

# 创建 cluster

```shell
kind create cluster # 默认的上下文名称是 kind
```

如果一直卡在 `Ensuring node image (kindest/node:v1.25.3)`，则需要更换 docker 国内镜像源，然后运行 `docker pull kindest/node:v1.25.3`。

```shell
vim /etc/docker/daemon.json
```

加入以下内容：

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"  
  ]
}
```

```shell
systemctl daemon-reload
systemctl restart docker
```

# 用 kubectl 查看 cluster 信息

```shell
kubectl version
kubectl get nodes
```

# 创建 deployment

deployment 的功能：
* 寻找一个合适的 node 去运行 app。
* 安排 app 在 node 上运行的时间。
* 当原先运行 app 的 node 出现故障时，寻找另一个可以运行 app 的 node。

```shell
kubectl create deployment [name] --image=[app image location]
kubectl get deployments
```

# 查看 pod 信息

使用 kubectl create deployment 创建 delployment 的同时，会创建一个 pod。

```shell
kubectl get pods # 列出所有pods
kubectl describe pods # 查看pods的详细信息
kubectl logs [pod name] # 查看pod的日志，pod_name在前两个命令的输出里都能找到
kubectl exec [pod name] -- env # 查看pod的环境变量
```

# 进入 pod 内部

1. 执行 `kubectl exec -ti [pod name] --bash` 进入 pod 内部环境。
2. 执行 `exit` 退出。

# cluster 与外部通信

## 通过 proxy

cluster 是一个封闭的网络环境，它内部的 nodes 可以互相通信，但是不能和 cluster 外部的主机通信。

通过运行 `kubectl proxy` 命令，可以在主机和 cluster 中间搭建通信的桥梁。从而直接通过 http 请求和 cluster 通信。

```shell
kubectl proxy
# Starting to serve on 127.0.0.1:8001
```

```shell
# k8s server version
curl http://localhost:8001/version

# pod info
curl http://localhost:8001/api/v1/namespaces/default/pods/[POD_NAME]/
```

## 通过 service

service 有这几种 type，分别代表不同的通信方式：
* ClusterIP (默认) - 在集群的内部 IP 上公开 Service 。这种类型使得 Service 只能从集群内访问。
* NodePort - 使用 NAT 在集群中每个选定 Node 的相同端口上公开 Service 。使用<NodeIP>:<NodePort> 从集群外部访问 Service。是 ClusterIP 的超集。
* LoadBalancer - 在当前云中创建一个外部负载均衡器(如果支持的话)，并为 Service 分配一个固定的外部IP。是 NodePort 的超集。
* ExternalName - 通过返回带有该名称的 CNAME 记录，使用任意名称(由 spec 中的externalName指定)公开 Service。不使用代理。这种类型需要kube-dns的v1.7或更高版本。

```shell
# 列出所有services
kubectl get services
# 默认的service叫kubernetes，它的type是clusterIP，表示nodes只在cluster内部通信
# NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
# kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   27s

# 新建一个service，设置type为NodePort，暴露nodes的8080端口
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

# 现在发现多了一个叫kubernetes-bootcamp的service
kubectl get services
# NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
# kubernetes            ClusterIP   10.96.0.1      <none>        443/TCP          4m1s
# kubernetes-bootcamp   NodePort    10.99.57.124   <none>        8080:30389/TCP   2m52s

# 现在可以通过[node ip]:[node port]来访问node了
curl [node ip]:8080
```