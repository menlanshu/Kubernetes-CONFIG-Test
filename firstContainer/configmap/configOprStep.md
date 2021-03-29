kubectl create configmap ui-config --from-file=ui.properties

kubectl get configmaps ui-config -o yaml

Downward API
