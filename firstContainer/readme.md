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
4. ServiceAccountToken

