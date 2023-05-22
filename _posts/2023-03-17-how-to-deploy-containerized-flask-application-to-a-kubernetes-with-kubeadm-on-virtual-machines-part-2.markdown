---
layout: post
title: "How to deploy containerized Flask application to a Kubernetes with Kubeadm on Virtual Machines - Part 2"
date: 2023-05-22 00:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
This is the second part of a series of hands-on tutorials for deploying containerized Flask application to a
Kubernetes with Kubeadm on Virtual Machines.
<!-- excerpt -->

# Series
1. [Part 1]({% post_url 2023-03-17-how-to-deploy-containerized-flask-application-to-a-kubernetes-with-kubeadm-on-virtual-machines-part-1%})
2. [Part 2]({% post_url 2023-03-17-how-to-deploy-containerized-flask-application-to-a-kubernetes-with-kubeadm-on-virtual-machines-part-2%})


# Overview
In this tutorial, we are going to 
1. Install a container runtime
2. Setup network forwarding
3. Setup a Kubernetes cluster
4. Verify our Kubernetes cluster's setup  



# 1. Install a container runtime (Both Master and Workers)
We are going to use `containerd` for our setup. The `containerd` is an industry-standard container runtime and it was adopted by many including `Kubernetes`, `Amazon` and `Docker`. You can use `ssh` to our `master` virtual machine or you can run the command directly from the `master` virtual machine.

## 1.1 Install `containerd` runtime
You can Install `containerd` using 

```
sudo apt install -y containerd
```

## 1.2 Configuring the `systemd` cgroup driver

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo nano /etc/containerd/config.toml
```

Under
      ```[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      ...```

Change the value for `SystemCgroup` from `false` to `true` and save the file.

After this change, make sure to restart `containerd` by typing

```
sudo systemctl restart containerd
```


# 2. Setup network forwarding (Both Master and Workers)

## 2.1 Forwarding IPv4 and letting iptables see bridged traffic

You need to run the following lines to set up the network.
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

## 2.2 Apply sysctl parameters without rebooting

```
sudo sysctl --system
```


# 3. Install kubeadm (Both Master and Workers)
Using `kubeadm`, you can create a minimum viable Kubernetes cluster that conforms to the Kubernetes Conformance tests.

## 3.1 Update the apt package index and install packages needed to use the Kubernetes apt repository

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

## 3.2 Download the Google Cloud public signing key

```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```
## 3.3 Add the Kubernetes `apt` repository:
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

## 3.4 Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```



## 3.5 Disable swap
You need to disable the swap otherwise you might have an error in the next step
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
```

# 4. Setup Kubernetes cluster

## 4.1 Initialize the Kubernetes cluster on Master (Master only)
We can now start initializing the cluster.
 ```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
> `sudo kubeadm reset` if you want to reset the cluster

Once you run it successfully, some instructions will be displayed on the terminal. You can just follow the instructions by

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Take not of the `kubeadm join` instruction on the screen. We need this for the next section.

## 4.2 Joining the cluster (Workers only)
You have to follow the instruction displayed on the terminal after the previous section 4.1. It should be different from mine. 

```
kubeadm join 10.0.2.15:6443 --token 88jjbi.bi0onhiorwb0ifk5 \
    --discovery-token-ca-cert-hash sha256:038d7af07f1f68b91e1220eb4437cfbbc9068a78d4f944d464de97f1ce1803ec
```
Once you joined to the cluster successfully, the `This node has joined the cluster` message will be displayed.

## 4.3 Checking Nodes (Master only)
You can check the nodes by running 


```kubectl get nodes```

The following will be displayed on the terminal. So master is not ready yet.

```
NAME          STATUS     ROLES           AGE     VERSION
master-vbox   NotReady   control-plane   17m     v1.27.2
worker-vbox   NotReady   <none>          4m30s   v1.27.2

```

## 4.4 Install Flannel (Master only)

We need to install `Flannel`. Flannel is an overlay network provider that can be used with Kubernetes. 
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
And after we check the pods using the following command.

```
kubectl get pods -A
```
You should see the following pods are creating/running. 
``` text
NAMESPACE      NAME                                  READY   STATUS              RESTARTS   AGE
kube-flannel   kube-flannel-ds-c4wfg                 1/1     Running             0          28s
kube-flannel   kube-flannel-ds-p775s                 1/1     Running             0          28s
kube-system    coredns-5d78c9869d-f5dgw              0/1     ContainerCreating   0          20m
kube-system    coredns-5d78c9869d-f6wmn              0/1     ContainerCreating   0          20m
kube-system    etcd-master-vbox                      1/1     Running             0          20m
kube-system    kube-apiserver-master-vbox            1/1     Running             0          20m
kube-system    kube-controller-manager-master-vbox   1/1     Running             0          20m
kube-system    kube-proxy-gzcgl                      1/1     Running             0          20m
kube-system    kube-proxy-v5lzg                      1/1     Running             0          7m34s
kube-system    kube-scheduler-master-vbox            1/1     Running             0          20m
```

## 4.5 Checking Nodes again (Master only)
You can check the nodes by running 


```kubectl get nodes```

Now you can see that both the master and worker are ready.

```
NAME          STATUS   ROLES           AGE   VERSION
master-vbox   Ready    control-plane   25m   v1.27.2
worker-vbox   Ready    <none>          12m   v1.27.2
```

# Reference
1. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl
2. https://www.nocentino.com/posts/2021-12-27-installing-and-configuring-containerd-as-a-kubernetes-container-runtime