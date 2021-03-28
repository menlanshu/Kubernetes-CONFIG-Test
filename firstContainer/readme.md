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
