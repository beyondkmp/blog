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

## vagrant配置文件

vagrant目前只是简单配置，启动三台机器，系统为ubuntu16.04.

1. **k8s-01**: 目前作为master,内存为2GB, ip为192.168.1.4
2. **k8s-02**: 目前作为worker,内存为1GB, ip为192.168.1.5
3. **k8s-03**: 目前作为worker,内存为2GB, ip为192.168.1.6

<!--more-->

### vagrant的配置文件

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

## kubernets集群搭建

### master搭建

#### 安装软件

本教程主要使用kubeadm来搭建。先使用下面命令安装kubeadm,docker(前提是可以科学上网，要不能下面的命令可能会超时)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y docker.io kubeadm
```

#### 初始化

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

####


