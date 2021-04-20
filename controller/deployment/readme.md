# orchestration capability
- horizontal scaling out/in
- rolling update
Owner reference of pod is replicaset

deployments->replicaset->pods
So the deployment only allow container's restartPolicy=Always

# how to scale out?
$ kubectl scale deployment nginx-deployment --replicas=4

# record action 
> record each action corresponding command
$ kubectl create -f nginx-deployment.yaml --record
> state when kubectl get deploy
- Ready current/desired
- UP-TO-DATE New version pods' number that up to date
- AVAILABLE Running and latest version and ready

> check rollout status of current deployment
$ kubectl rollout status deployment/nginx-deployment

# how to change deployment configuration
$ kubectl edit deployment/nginx-deployment
=> open nginx-deployment api object in etcd
=> example of rolling update
  Normal  ScalingReplicaSet  10m    deployment-controller  Scaled up replica set nginx-deployment-5d59d67564 to 2
  Normal  ScalingReplicaSet  2m42s  deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 1
  Normal  ScalingReplicaSet  2m18s  deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 1
  Normal  ScalingReplicaSet  2m18s  deployment-controller  Scaled up replica set nginx-deployment-69c44dfb78 to 2
  Normal  ScalingReplicaSet  2m17s  deployment-controller  Scaled down replica set nginx-deployment-5d59d67564 to 0

- there are percentage for old and new
  > default is 25% of DESIRED value
Update strategy in deployment object
  > type: RollingUpdate
  > rollingUpdate:
  > maxSurge
  > maxUnavailable

# how to change and rollout deployment
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
$ kubectl edit deployment 
$ kubectl rollout undo
$ kubectl rollout resume

$ kubectl rollout history deployment/nginx-deployment --reversion=2
> if we think each update save a replicaset is wasting storage
We can run below command before update
$ kubectl rollout pause deployment/nginx-deployment
$ then we can do kubectl edit/kubectl set image
No rolling update will create new replicaset
After all action down, you can use below command
- kubectl rollout resume, only one rolling update will be triggered

# kubectl deployment replicaset history version count
spec.revisionHistoryLimit
the history version count of Kubernetes's deployment

deployment=>replicaset=>pods
