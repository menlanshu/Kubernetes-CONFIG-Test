# what's pod?
Pod is atomic unit of kubernetes

# why we need pods?
namespace for isolated
cgroups for limitation
rootfs for filesystem
Why we need pods?
The essence of container is process

Three container(process) should be run at the same time and in the same place
So several containers combined as one pod
- direct file exchagne
- use localhost/socket for local communication
- frequently remote invoke
- need share linux namespace, a container need add to another network namespace

Pod: container design pattern

## the implementation principle of pdo
### pod is only a logic concept
Pod is some containers shared some resource
It's still operation for namespace and cgroups

### containers in pod share a network namespace and declare a volume
A, B container share network and volume
$ docker run --net=B --volumes-from=B --name=A image=A
but it's not a equal relationship
B container need start before A

So in kubernetes, Pod need a middle container named Infra
So in pod, Infra container is always the first container that be created
Other container will use Join Network Namespace to connect with Infra
- can use localhost to communicate
- see network equipment as infra
- a pod only has a ip address
- all other network resource is separated by pod
- The life time for pod only same with Infra container, has no relationship with A and B

----------------------------------------
two container both use shared-data volumes, the folder mounted to two containers at the same time
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    hostPath:      
      path: /data
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]

# example: war package and web server
java web war pacakge must be put in webapps folder in Tomcat

----------------------------------------
web is a init container instead of normal container
in a pod, all ini container will start before normal containers
All initial containers will start step by step and shutdown after all of them started
Then normal containers started
apiVersion: v1
kind: Pod
metadata:
  name: javaweb-2
spec:
  initContainers:
  - image: geektime/sample:v2
    name: war
    command: ["cp", "/sample.war", "/app"]
    volumeMounts:
    - mountPath: /app
      name: app-volume
  containers:
  - image: geektime/tomcat:7.0
    name: tomcat
    command: ["sh","-c","/root/apache-tomcat-7.0.42-v2/bin/start.sh"]
    volumeMounts:
    - mountPath: /root/apache-tomcat-7.0.42-v2/webapps
      name: app-volume
    ports:
    - containerPort: 8080
      hostPort: 8001 
  volumes:
  - name: app-volume
    emptyDir: {}

This combination is a very usual pattern: sidebar
sidebar: start a assistant container to finish tasks for main container

# example: log collection for container
- an application need to log log file to /var/log
- we can mount a volume of pod to /var/log
And we create a sidecar container, declare the same volume mounted to its own /var/log

- sidebar container only do one job
- keep reading log file from /var/log of its own container
- redirect to MongoDB or Elasticsearch

# classical example: Istio project, use sidecar to finish micro service administration
(design pattern)[https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns]

Process Group concept is very very important!

# nodeselector
choose the node you want to deploy your pod

# nodename
set by scheduler
if the value is settled, then k8s think it's scheduled already
but user can use nodename tobcheat k8s

# hostalias
spec:
  hostAliases:
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
it actually change the host config in pod
