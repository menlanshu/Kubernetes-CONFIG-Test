# service types
1. clusterip services
2. headless services
3. nodeport services
4. loadbalancer services


# why we need services?
Pods dies => dynamic ip address
Services
1. stable IP address
2. loadbalancing
3. losse coupling
3. within or outside cluster


# how service connect to pods
selctor app: micrservice-one
> pods are
> key value paires
> labels of pods
> random label names
kubernetes will create a component named Endpoints
> you can get them via kubectl get endpoints
> keeps track of which pods are the members/endpoints of the service

# clusterip
1. default type
- example
 > micro-service container
 > side-car container

# headless services
- client want to communicate with 1 specific pod directly
- pods want to talk directly with specific pod
- so , not randomly selected
- use case: stateful applicatinos ,like databases
  > my sql , only master node allow to write
CLients need to figure out IP addresses of each pod
> option 1: api call to k8s api server
> option 2: DNS lookup
> Set clusterIP to "None" - returns Pod IP address instead

# nodeport services
Cluster ip only accessibel within cluster
External traffic has access to fixed port on each worker node
- nodeport services: not secure

# loadbalancer services
- nodeport and clusterip service created automatically
- cloud providers loadbalancer
