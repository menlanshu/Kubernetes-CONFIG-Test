# how to get namespace
- kubectl get namespace

# namespace per default
4 namespaces
1. kubernetes-dashboard only with minikube
2. kube-system
> Do not create or modify in kube-system
> system processes
> master kubectl processes
3. kube-public
> publicely accessibale data
> A config map, which contains cluster info
4. kube-node-lease
> heartbeats of node
5. default namespace
> resources you create are located here

# how to create a namespace
kubectl create namespace my-namespace
> you also can create via a yaml file

# why we need namespace?
1. everything in one namespace
> database namespace
> monitoring namespace
> elastic stack namespace
> nginx-ingress namespace
> logging namespace

2. Conflicts: many teams, same application
> same name, different configuration

3. Resource Sharing: staging and development
> staging
> development
> nginx-ingress controller
> elastic stack

4. Blue/Green deployment
two different version for production
> Production blue
> Production Green
-> same way for pilot run, with a diffrent port or configuraiton

5. Access and resoruce limits on namespaces
> project A & B namespace
> only can do things in your own namespace
Limit: CPU, RAM, Storage per NS

# Characteristic of namespace
1. You can't access most resourecs from another namespaceo> you can't access configmap of another namespace
> service can be referenced mysql-service.database(ns name)
2. some components can't not be created with in a namespace
> volume can't be included to one namespace
> kubectl api-resources --namespaced=false

# Create components in namespace
> add namespace in metda data of yaml, you can add it
 namespace: my-namespace
> kubectl apply -f file --namespace=my-namespace

kubens => a tool need to install for ns, you can switch active namespace
kubens my-namespace => change default namespace to my-namespace
