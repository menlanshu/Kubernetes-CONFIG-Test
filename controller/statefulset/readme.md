# statefulset
## Topology State
Master Node and Slave node
Master node must be created before slave node
And network label must be the same too, or issue will happen
## Storage State
Database storage instance

# Headless service

## Service
1. VIP Virtual IP
10.0.23.1 proxy for many pods
2. Service DNS
my-svc.my-namespace.svc.cluster.local
Two way to realize Service DNS
- Normal Service, you can get VIP of service through my-svc.xxx domain name
- Headless Service, when you call my-svc.xxx. You can get the ip of one pod directly. You can see, headless Service do not need a VIP, DNS parse out Pod's ip
> You can check headlessService.yaml
> DNS bind name <pod-name>.<svc-name>.<namespace>.svc.cluster.local
The only resolvable identity

## statefulset 
before web-0 become Running state, web-1 will keep in Pending status
!! Pod must set livenessProbe and readinessProbe

$ kubectl run -i --tty --image busybox:1.28.4 dns-test --restart=Never --rm /bin/sh
$ nslookup web-o.nginx
$ nslookup web-1.nginx

## delete pods test
$ kubectl delete pod -l app=nginx
$ kubectl get pod -w -l app=nginx
> StatefulSet keep the stability of pod's network label

# Stateful set 2

## Persistent Volume Claim
- Someone should maintain volume mount and other action for developer
> You can see ceph_rbd.yaml for example
> two issue: 1. some fileds is not easy to understand 2. You can't expose ip to developers of the whole company

- So kubernetes import concept of PVC(Persistent Volume Claim) and PV(Persistent Volume), it makes use of persistent volume easier

- example
See PersistentVolumeClaim.yaml
> no detail info of volume
> only info that are useful:
> - size of volume: 1GiB
> - accessModes: ReadWriteOnce => read and write, and can only be mounte in one node instead of share between multi nodes
[Detail info of AccessMode](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)
Use PVC in pod, see example_use_of_pvc.yaml

## Pod get thos PVC from which place?
The definition of PV/PVC like the use of interface
Kubenetes will bind PVC with PV for us
Interface: PVC
Maintainer: PV
- StatefulSet add an extended field: volumeClaimTemplates
- PVC's name will assign a same id with this pod

example of statefulset_withpvc.yaml
$ kubectl create -f statefulset_withpvc.yaml
$ kubectl get pvc -l app=nginx
$ kubectl get pvc
$ kubectl get pv
$ for i in 0 1; do kubectl exec web-$i -- sh -c 'echo hello $(hostname) > /usr/share/nginx/html/index.html'; done
$ kubectl get pods
$ kubectl exec -it web-0 -- sh
~~ after delete, the data saved in volume still exists
$ kubectl delete pod -l app=nginx
$ kubectl get pods
$ kubectl exec -it web-0 -- sh
$ kubectl exec -it web-1 -- sh

## method of statefulset
1. statefulset controller manage pod directly
- because It's not same as deployment, ReplicaSet has exactly the same pod. Some difference between them
2. Kubernetes can use headless service, create numeric DNS record for those numeric pod
3. StatefulSet create same numeric PVC for each pod
