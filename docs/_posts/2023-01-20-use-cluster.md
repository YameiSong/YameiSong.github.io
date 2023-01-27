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

# cluster 与外部通信

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