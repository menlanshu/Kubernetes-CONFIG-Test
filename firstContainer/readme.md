# basic rule of kubernetes
kubernetes do not recommend you use command to start a container
you should use configuration file!
Then use below command to start a container: kubectl create -f configurationfile
## the benefit of this pattern
- you know actually what happened
an example of deployment yaml file

apiVersion: apps/v1
-- define kind of component, the type of this API object
kind: Deployment
metadata:
-- a identification for API object
  name: nginx-deployment
spec:
  selector:
-- the rule for labels match
-- we call it: Label Selector
    matchLabels:
      app: nginx
-- replica count is 2
  replicas: 2
  template:
    metadata:
-- key-value tag
-- controller like deployment can use labels to filter out objects that it cared about
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

# create deployment
kubectl create -f nginx-deployment.yaml
-- query all pod with label equal as app: nginx
-- in command line, equal use = not :
kubectl get pods -l app=nginx

# check status of pod
kubectl describe pod podname
> very useful part of describe info is Events
> any abnormal happen, you should check the events immediately

# change nginx version in yaml file
you can use kubectl replace -f nginx-deployment.yaml
And you can use kubectl apply -f nginx-deployment.yaml
it's the advantage of declaration API, you do not need care about the action: create or update

# volume test
emptyDir or hostPath to config volume path
kubectl exec -it podname -- /bin/sh

# create a pod with two container
kubectl apply -f nginxyamlfile
kubectl attach -it nginx-podname -c shell

# namespace share
If container in pod want to share host's namespace, it must be pod level
hostNetwork: true
hostIPC: true
hostPID: true
> init container will start before containers, and follow order strictly

# key concepts
image
command
workingDIr
Ports
volumeMounts
--Special field need to focus on
ImagePullPolicy => define the policy for image pull
  > default is always, each time when you create pod, fetch image again
  > Never or IfNotPresent, pod will never pull image, only pull when no image in host
Lifecycle => COntainer lifecycle hooks => when container status change trigger a serials of hook
----------------------------------------
an example for lifecycle configuration

apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
------execute after container start, but the order is not ensured strictly
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
------execute before container died, block current container kill process until hook finish this action
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]


# Status of Pod API
except metadata and spec, there is another important field named pod.status.phase
1. Pending => pod yaml already submitted to kubernetes, API object already created and saved in Etcd. But some containers can not be created smoothly for some reason.
2. Running => In this status, pod already scheduled successed, and bind with a specific node
3. Succeeded => all container in pod already run normally and quit successfuly
4. Failed => At least one container stay in abnormal status. You need check log or events of pod
5. Unknown => An abnormal status which means status of pod can not be reported by kubelet to kube-apiserver. There maybe some issue in communication
And more detail info in status, can be called conditions: PodScheduled, Ready, Initialized, Unschedulable



# projected volume
- not for data exchange
- not for data storage
- only for projected into container
1. Secret
> help you to secret data that your pod want to access
> save to Etcd
> then you can use volume way to access to Secret info
--Create secret use command or use yaml file
kubectl create secret generic user --from-file=username.txt
kubectl create secret generic pass --from-file=password.txt
--Create secret from file secret.yaml
kubectl exec -it test-projected-volume -- /bin/sh
> after you enter the container, you can use ls /projected-volume to see file list
> username.txt password.txt
--And mount to contiainer in this way, once data in Etcd was changed
> The file data in volume will be updated too. Kubelet maintain those volume timely

2. ConfigMap
> difference with secret => configmap save configuration data not need to be encrpted
> but the usage for configmap basically the same with secret
> you can use kubectl create configmap to create configmap from file/folder
- example of configmap in configmap folder
$ kubectl create configmap ui-config --from-file=ui.properties
3. Download API
> Function: Let all containers in this pod can access POD API object info
4. ServiceAccountToken
A build-in service account in kubernetes
It can assign privilge according object
Example:
Service Account A: only allow to do get opr for Kubernetes API
Service Account B: allow to do all oprs

explainations: service account is a special type of secret
**Kubenertes provide a default service account**
So in a running pod of Kubernetes, you can use this service account directly, and no need to mount it explicitly

The position of serice account in kubernertes container is fixed:
path: /var/run/secrects/kubernetes.io/serviceaccount
files: ca.crt namespace token

# container health check and recover mechanism
1. you can define a probe instead of default interface of docker to detect status of a container
an example of kubernetes document:
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: test-liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /temp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
if you create a pod instead of a deployment
**The pod will not leave current node, even if the node died!**

2. you can use restartPoloty to change recover policy of pod
- Always: restart container once it's not in running status
- OnFailure: On when container is abnormal, restart container
- Never: Never restart container

3. many other ways to use probe to monitor status of pod
- livenessProbe:
    httpGet:
      path: /healthz
      port: 8080
      httpHeaders:
      - name: X-Custom-Header
        value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
- livenessProbe:
    tcpSocket:
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 20
> in this way, you can expose a url for health monitoring or monitor a listening port directly

3. readinessProbe field in k8s pod
readinessProbe check successful or not decide whether this pod can be accessed by the way of Service. Do not impact the lifetime of pod

# PodPreset function
> you can use PodPreset to reduce the configuration difficulty
You can create a PodPreset firstly, then create pod
An example for PodPreset:
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6387"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
> selector is very important => select all pods with label role:frontend
> if you define multi PodPreset for one pod, then k8s will try to *merge* them. But if any conflict happen, those fields will not be changed
