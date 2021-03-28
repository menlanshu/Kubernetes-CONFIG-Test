kubeadm config images list

然后使用下边的脚本拉镜像并tag成指定Google的镜像

images=(
    k8s.gcr.io/kube-apiserver-amd64:v1.11.4
    k8s.gcr.io/kube-controller-manager-amd64:v1.11.4
    k8s.gcr.io/kube-scheduler-amd64:v1.11.4
    k8s.gcr.io/kube-proxy-amd64:v1.11.4
    k8s.gcr.io/pause:3.1
    k8s.gcr.io/etcd-amd64:3.2.18
    k8s.gcr.io/coredns:1.1.3
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
done
