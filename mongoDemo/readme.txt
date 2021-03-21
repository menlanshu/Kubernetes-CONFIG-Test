## mongo db setup demo
1. mongodb pod set up only for internal service
2. setup mongo express
 - connect to db
 - authenticate to db => db.usr/pwd
Setup Configmap
Setup Secret
 - generate password => echo -n "username" | base64
 - start secret => kubectl apply -f mongo-secret.yaml
3. External service for mongo express
 - ip address of Node
 - Port of external service

Whole flow with below
webbrower=>mongo express external service=>mongo express=>mongo internal service(configmap url)=>mongo db


step 1: Create file for secret and deployment
1.1 echo -n "username" | base64
1.2 secretKeyRef in deployment yaml file for reference

step 2: apply configuration file
2.1 kubectl apply -f mongo-secret.yaml
2.2 kubectl apply -f mongo-deployment.yaml
2.3 if there area any issue, use kubectl describe/logs to check any abnormal
#tip if you put --- in yaml file, then you can put multiple documents in 1 file

step 3: Create file for service
3.1 explain service file
- kind: Service
- metadata/name: a random name
- selector: to connect to pod through label
- ports:
  port: Service port
  targetPort: ContainerPort of deployment

step 4: apply configration file for service
- check describe of service
  Endpoints: the ip address of pod and port which is listening
  You can use kubectl get pod -o wide to verify
- You can use kubectl get all | grep mongo to check all component that you already created for mongo

step 5: config mongo express file
- config a yaml file for mongo express
- config a yaml for configmap
5.1 configMapKeyRef in deployment yaml file for reference

step 6: apply mongoexpress configmap and db service

step 7: create and apply a mongoexpress service
- how to make it an external service?
  add a item in file:
  type: LoadBalancer => assign server an external ip address and so accpets external request
  when you choose LoadBalance, if you use minikube service may always in status of pending
  you can try NodePort for test
  nodePort: between 30000~32767 => 30000
mongo express external service=> mongo express pod=>mongo internal sercice=> mongo db pod
