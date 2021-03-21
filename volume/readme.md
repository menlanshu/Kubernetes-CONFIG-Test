# how to persist data in Kubernetes using volumes?
- persistent volume
- persistent volume claim
- storage class


# the need for volumes
1. database persistence
my-app  <==> mysql
a. Storage that does't depend on the pod lifecycle
b. Storage mus be available on all nodes
c. Storage need to survive even if cluster crashed


# Persistent Volume
- a cluster resource
> need actual local disk
> need nfs server maybe
> cloud storage
> PV is not managed by namespace
questions
1. what type of storage do you need?
2. you need to create and manage them by yourself
external plugin to your cluster
- created via yaml file
--- yaml file
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-name
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolueReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.0
  nfs:
    path: /dir/path/on/nfs/server
    server: nfs-server-ip-address
--- yaml file for google cloud
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-volume
  labels:
    failure-domain.beta.kubernetes.io/zone: us-central1-a__us-central1-b
spec:
  capacity:
    storage: 400Gi
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    pdName: my-data-disk
    fsType: ext4
--- yaml file for local storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolueReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
        operator: In
        values:
        - example-node


## persistent volume claim
- pvc must in the same namespace with other components
pvc => pv
in pod configuration
- container can have pvc, each pod may have multi containers
- pod can have pvc

## configmap and scecret
no need create via pc and pvc
prometheus-app

## each container can have multi volumes
pvc, configmap and secret

## 3rd K8S component => storage class
SC provision persistent volumes dynamically
also use and claim by PVC~
