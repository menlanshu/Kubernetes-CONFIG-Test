# Create a master node
kubeadm init

# Add a node to current cluster
kubeadm join < master node's IP and port >
kubelet do not run as a container

kubadm => kubelet run in the server directly and use container to deploy other components of kubernetes

# install kubeadm from ali mirror
# 配置源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 安装
yum install -y kubelet kubeadm kubectl

# what kubeadm init do
## step 1: pre check
1. linux keneral higher than 3.10
2. hostname is standard? All name stored into Etcd need to use standard DNS name
3. kubeadm and kublet version matched?
4. Already has kubernetes binary file?
5. kubernetes manage port, 10250/10251/10252 occupied or not?
6. ip,mount command exist?
7. Docker is installed or not?

## step 2: crt
kubeadm will generate crt for kubernetes
=> file put in /etc/kubernetes/pki
=> certificate file: ca.crt  private key: ca.key

kubectl use streaming to get container log
=> use kube-apiserver to send command to kubelet
=> kubeamd will generate apiserver-kubelet-client.crt and apiserver-kubelet-client.key

Other certification, such as aggregate API server
You can copy already exist cetification to /etc/kubernetes/pki/ca.{crt,key}

## step 3: Next step is generate configuration file to access kube-apiserver
path: /etc/kubernetes/xxx.conf
admin.conf controller-manage

## step 4: generate pods file for all components
path: /etc/kubernetes/manifest
kube-apiserver.yaml

$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

**started ported with a special way names "Static Pod"**
kubelet auto detect yaml file and auto create pods for master components

Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来。

## step 5: create a bootstrap token for kubeadm
 If you have token, you can use kubeadm join to join cluster in each node installed with kubelet and kubeadm

# what kubeadm join do
kubeadm => master node kube-apiserver => configmap: cluster-info
bootstrap token take the role for security verification

# config parameters you need for kubeadm
> kubeadm init --config kubeadm.yaml
----------------------------------------
> an example for kubeadme.yaml => any configuration you put in this file will cover configuration in manifest folder
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
api:
  advertiseAddress: 192.168.0.102
  bindPort: 6443
  ...
etcd:
  local:
    dataDir: /var/lib/etcd
    image: ""
imageRepository: k8s.gcr.io
kubeProxy:
  config:
    bindAddress: 0.0.0.0
    ...
kubeletConfiguration:
  baseConfig:
    address: 0.0.0.0
    ...
networking:
  dnsDomain: cluster.local
  podSubnet: ""
  serviceSubnet: 10.96.0.0/12
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  ...

# where is the source code of kubeadm?
> path: kubernetes/cmd/kubeadm
> it's a part of kubernetes project
> and code in app/phases is the steps that uphere

# the latest official document
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
