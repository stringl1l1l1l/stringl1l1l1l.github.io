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

Windows

```sh
winget install Helm.Helm
```

Linux（Debian）

```sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
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

```sh
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64

# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-arm64

chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

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



## Kubectl

### 安装

Linux

```sh
# x86-64
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```



## Dubbo

创建命名空间

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
git clone https://github.com/apache/dubbo-admin.git && cd ./dubbo-admin/kubernetes
kubectl apply -f ./ -n dubbo-demo
kubectl --namespace dubbo-demo port-forward service/dubbo-admin 38080:38080
```



