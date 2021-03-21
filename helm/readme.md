# main concept of helm

# helm changes alot between versions
 understand the basic common principles and use cases

# what is helm?
1. pacakge manager for kubernetes
- to package yaml files
- distribute them in public and private

# what is heml chart?
- bundle of yaml files
- create your own helm chats with helm
- push them to helm repository
- download an use existing ones
> data base apps, monioring apps, premetuis
> reuse configuration

# how to use them?
- sharing helm charts
- Need some deployment?
heml serach <keyword>
heml hub: https://hub.helm.sh/
helm charts github project: https://github.com/helm/charts
install helm

# when to use them?
1. define a common blueprint
2. dynamic values are replaced
> you can have a template file, but instead of values
> .Values.container.name
> values.yaml
name: my-app
container:
  iamge: my-app-image
  port: 9001

## practical for CI/CD => use template and value.yaml

## Another use case
- same applications across different environments
helm chart structure
mychart/
  chart.yaml => meta info about charts
  values.yaml => values for template values
  charts/ => dependencies
  templates/ => actual template files
> helm install <chartname>
> optionally other files like readme and license file
> helm install --values=my-values.yaml <chartname>
 - values injection into template files
> helm install --set version=x.fdsaf

## third feature - release management
helm version 2 comes in two parts
client(helm CLI)  =====>   server(Tiller)
Create or change deployment 
> keeping track of all chart executions
helm install/upgrade/rollback
