---
title: "k8s部署"
date: 2025-03-08
author: liwener
---



## Kubectl

### 安装

windows

```powershell
winget install -e --id Kubernetes.kubectl
```



## Helm

### 安装

windows

```sh
winget install Helm.Helm
```

### 更换镜像源

```sh
helm repo add bitnami "https://helm-charts.itboon.top/bitnami" --force-update
helm repo add grafana "https://helm-charts.itboon.top/grafana" --force-update
helm repo add prometheus-community "https://helm-charts.itboon.top/prometheus-community" --force-update
helm repo add ingress-nginx "https://helm-charts.itboon.top/ingress-nginx" --force-update
helm repo update
```



## Kind

### 安装

https://github.com/kubernetes-sigs/kind/releases

### 创建集群

```sh
kind create cluster --config kind-cluster.yaml 
```

```yaml
# kind-cluster.yaml 添加了镜像源

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["https://registry.dockermirror.com"]
nodes:
- role: control-plane
- role: worker
```



## Dubbo

```sh
kubectl create ns dubbo-demo
```

### zookeeper

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install zookeeper bitnami/zookeeper --set persistence.enabled=false -n dubbo-demo
kubectl get pods -n dubbo-demo
```

### dubbo-admin

```sh
git clone https://github.com/apache/dubbo-admin.git && cd /dubbo-admin/kubernetes
kubectl apply -f ./ -n dubbo-demo
kubectl --namespace dubbo-demo port-forward service/dubbo-admin 38080:38080
```



