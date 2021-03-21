# External service vs ingress
ingress => redirect to internal service

ingress configuration file
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - backend:
          serviceName: myapp-internal-service
          servicePort: 8080

# how to configure ingress in your cluster?
You need a implementation for ingress
Which is ingress controller
- evaluated and processes ingress rules

# what is ingress controller
- evaluated all the rules
- manages redirections
- entrypoint to the cluster
- many third-party implementations
- k8s nginx ingress controller

# environment on which your cluster runs
- proxy server
 > separate server
 > public IP address and open ports
 > entrypoint to cluster
 > redirect to ingress controller

# try to demo
- minikube addons enable ingress
> you use kubectl -n kube-system get pod to see ingress
- create a ingress in the same namespaec with service and pod you want to link 
> use kubernetes-dashboard to do a test
kubectl apply -f dashboard-ingress.yaml

- change /etc/hosts to dns
ingressipaddress ingresshostname

kubectl describe ingress myapp-ingress
- you can see a default backend with name: default-http-backend
- config default backend for ingress to redirect error

# multiple paths for same host

tls configuration for https://
- secret file notes
> tls.crt, tls.key
> same namespace with ingress components
