---
title: "使用vagrant搭建k8s集群"
date: 2018-10-22T10:50:46+08:00
lastmod: 2018-10-22T10:50:46+08:00
draft: false
keywords: [vagrant,k8s,kubernets集群]
description: "使用vagrant搭建k8s集群"
tags: [vagrant,k8s,kubernets集群]
categories: [运维人生]
author: "beyondkmp"

---

# vagrant配置文件

vagrant目前只是简单配置，启动三台机器，系统为ubuntu16.04.

1. **k8s-01**: 目前作为master,内存为2GB, ip为192.168.1.4
2. **k8s-02**: 目前作为worker,内存为1GB, ip为192.168.1.5
3. **k8s-03**: 目前作为worker,内存为2GB, ip为192.168.1.6

<!--more-->

## vagrant的配置文件

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "k8s-01" do |edge|
    edge.vm.box =  "bento/ubuntu-16.04"
    edge.vm.network "private_network", ip: "192.168.1.4"
    edge.ssh.insert_key = false
    edge.vm.hostname = "k8s-01"
    edge.memory = 2048
  end

  config.vm.define "k8s-02" do |parent|
    parent.vm.box =  "bento/ubuntu-16.04"
    parent.vm.network "private_network", ip: "192.168.1.5"
    parent.ssh.insert_key = false
    parent.vm.hostname = "k8s-02"
    parent.memory = 1024
  end

  config.vm.define "k8s-03" do |tertium|
    tertium.vm.box =  "bento/ubuntu-16.04"
    tertium.vm.network "private_network", ip: "192.168.1.6"
    tertium.ssh.insert_key = false
    tertium.vm.hostname = "k8s-03"
    tertium.memory = 1024
  end

end
```

# kubernets集群搭建

## master搭建

### 安装软件

本教程主要使用kubeadm来搭建。先使用下面命令安装kubeadm,docker(前提是可以科学上网，要不能下面的命令可能会超时)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y docker.io kubeadm
```

### 初始化

使用下面的yml直接搭建起master,目前主要参考极客时间上kubernets专栏来搭建的。下面的v1alpaa1已经过时，如果直接运行会报下面的错误: `Please use kubeadm v1.11 instead and run 'kubeadm config migrate --old-config old.yaml --new-config new.yaml', which will write the new, similar spec using a newer API version`。


**原始的yml**

```bash
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
apiServerExtraArgs:
  runtime-config: "api/all=true"
kubernetesVersion: "stable-1.11"
```

**修改后的yml**

```bash
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
  advertiseAddress: 192.168.1.4
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
apiServerExtraArgs:
  runtime-config: "api/all=true"
kubernetesVersion: "v1.12.1"
```

上面的配置还添加了对应的ip，这样就不会使用默认的eth0的ip。修改完yml后面，直接运行下面命令.这样master的初始化就完成了。

```bash
 kubeadm init --config kubeadm.yaml
```

初始化完成后，会有一些提示,如下

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.1.4:6443 --token g61cb8.19evngqy28x7wk4d --discovery-token-ca-cert-hash sha256:0315ab4d1a602ab5b27dcd1259af25bb9449b02554189870ec17c088643f2ba2
```

要将kubernets的配置放到这个用户的目录，才不会每次使用都报授权错误

```
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

### 配置网络插件

目前插件都容器化了，基本上也就是一条命令安装下就可以了。没有安装之前，需要网络的插件是pending状态,get node状态也是NotReady状态

```
root@k8s-01:~# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-8s8nh         0/1     Pending   0          3m20s
coredns-576cbf47c7-pwctq         0/1     Pending   0          3m20s
etcd-k8s-01                      1/1     Running   1          2m36s
kube-apiserver-k8s-01            1/1     Running   1          2m22s
kube-controller-manager-k8s-01   1/1     Running   1          2m36s
kube-proxy-7fm5v                 1/1     Running   1          3m20s
kube-scheduler-k8s-01            1/1     Running   1          2m35s

root@k8s-01:~# kubectl get nodes
NAME     STATUS     ROLES    AGE     VERSION
k8s-01   NotReady   master   5h47m   v1.12.1
```

运行下面的命令安装网络插件

```
root@k8s-01:~#  kubectl apply -f https://git.io/weave-kube-1.6
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.extensions/weave-net created
```

再查看相应的状态

```
root@k8s-01:~# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-576cbf47c7-8s8nh         1/1     Running   0          5h47m
coredns-576cbf47c7-pwctq         1/1     Running   0          5h47m
etcd-k8s-01                      1/1     Running   1          5h46m
kube-apiserver-k8s-01            1/1     Running   1          5h46m
kube-controller-manager-k8s-01   1/1     Running   1          5h46m
kube-proxy-7fm5v                 1/1     Running   1          5h47m
kube-scheduler-k8s-01            1/1     Running   1          5h46m
weave-net-2b8q8                  2/2     Running   0          21s

root@k8s-01:~# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
k8s-01   Ready    master   5h47m   v1.12.1
```

到目前为止，master基本就配置完成了。

# kubernets worker配置

kubernets的worker配置就简单多，主要是下面三步

1. 在所有的worker节点执行上面master软件安装的步骤。目前master和woker的软件安装都是一样，主要区别是master上面还会启动kube-apiserver、kube-scheduler、kube-controller-manager这三个系统pod。
2. 执行部署Master节点时最后的join命令

```
kubeadm join 192.168.1.4:6443 --token g61cb8.19evngqy28x7wk4d --discovery-token-ca-cert-hash sha256:0315ab4d1a602ab5b27dcd1259af25bb9449b02554189870ec17c088643f2ba2
```

3. 在专栏里面上运行上面两个就可以通了，但是就是通不了。有一个网络插件weave一直CrashLoopBackOff，后面谷歌下，发现是路由找不到。

在master运行下面的命令

```
root@k8s-01:~# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6h37m
```

然后在worker机器运行下面命令添加路由

```
route add 100.96.0.1 gw <your real master IP>
```

最后查看下worker的状态，没有问题就全部完成了。

```
root@k8s-01:~# kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
k8s-01   Ready    master   6h39m   v1.12.1
k8s-03   Ready    <none>   15m     v1.12.1
```
