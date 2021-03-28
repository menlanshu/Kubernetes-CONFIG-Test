# import repo for kubernetes to install kubeadm
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# make cache
yum makecache

# install kubeadm
yum install -y kubeadm

# create a yaml file for master node create by kubeadm
> kubeadm.yaml
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
apiServerExtraArgs:
  runtime-config: "api/all=true"
kubernetesVersion: "stable-1.20"

# init master
kubeadm init --config kubeadm.yaml
> generate a command for kubeadm join
kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711

# authuration
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
> Kubernetes 集群默认需要加密方式访问。所以，这几条命令，就是将刚刚部署生成的 Kubernetes 集群的安全配置文件，保存到当前用户的.kube 目录下，kubectl 默认会使用这个目录下的授权信息访问 Kubernetes 集群

# check node not ready error
kubectl describe node master => network issue
$ kubectl get pods -n kube-system
> Core DNS and kube-controler-manager is pending
NAME               READY   STATUS   RESTARTS  AGE
coredns-78fcdf6894-j9s52     0/1    Pending  0     1h
coredns-78fcdf6894-jm4wf     0/1    Pending  0     1h
etcd-master           1/1    Running  0     2s
kube-apiserver-master      1/1    Running  0     1s
kube-controller-manager-master  0/1    Pending  0     1s
kube-proxy-xbd47         1/1    NodeLost  0     1h
kube-scheduler-master      1/1    Running  0     1s

# deploy network plugin
$ kubectl apply -f https://git.io/weave-kube-1.6

# deploy workernodes for kubernetes
step 1: install docker and kubernetes => make sure docker version is supported by kubernetes
step 2: execute command when you install master node
$ kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711

# how to make master nodes run pods
- default condition master node is not allowed to run pods
- kubernetes do this via a policy named : "Taint/Toleration"
- if one node is taged as "Taint", then all pods can not run in this node
- but some pods can declare that they can have "Toleration", then they can run on this node
## how to tag a node as  Taint 
$ kubectl taint nodes node1 foo=bar:NoSchedule
> taint will only has impact no new pods, already exists pod in current node will not be impacted
## how to tag pod Toleration
apiVersion: v1
kind: Pod
...
spec:
  tolerations:
  - key: "foo"
    operator: "Equal"
    value: "bar"
    effect: "NoSchedule"

this configuration means: this pod can tolerate all Taint with key-value as : foo=bar

## check default toleration when you use kubeadm to install master nodes
$ kubectl describe node master
Name:               master
Roles:              master
Taints:             node-role.kubernetes.io/master:NoSchedule
> a Taint names: node-rol.kubernetes.io/master:NoSchedule but no value

You can use below definition to allow pods to run on this node
Pods can tolerate all Taint use foo as key, no matter the value
apiVersion: v1
kind: Pod
...
spec:
  tolerations:
  - key: "foo"
    operator: "Exists"
    effect: "NoSchedule"

### of cause, if you want to run all pods in master node
You can delete the Taint
$ kubectl taint nodes --all node-role.kubernetes.io/master-
the minus "-" means remove all Taints use "node-role.kuberntes.io/master" as key

# deploy a dashboard plugin
$ kubectl apply -f 
$ $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc6/aio/deploy/recommended.yaml


# persistent data for kubernetes
mount a remote data volume, have no relationship with current node, then persistent data is realized
Several ways to persist data:
1. Ceph
2. ClusterFS
3. NFS
A kubernetes persistent project names: Rook

With below several commands, ROok can be deployed

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/common.yaml

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/crds.yaml

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml

-- actually, you can see cluster depend on uphere three components in the yaml file itself
$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
