# deployment as a trigger
deployment has a field named: replicas
if pods count > 2: some pod will be deleted
if pods count < 2: some pod will be created

Who manage all of those actions? **kubectl-controller-manager**


# where is kubectl-controller-manager
- in kubernetes/pkg/controller folder
> deployment is one kind of them for orchestration
> all controllers in this folder follow a command orchestration pattern: Control loop
describe this Control loop:
```go
for {
  actualState := getActualState of object X in cluster
  expectedState := getExpectedState of object X in cluster
  if(actualState == expectedState)
  {do nothing;}
  else
  {
    do orchestration action, change actualState to expectedState
  }
}
```
## where do we get all of this state?
### actual state
- from kubernetes cluster
- use heartbeat to report container and node state
- monitor data saved in monitor systems
- controller collect information that it's interested in
### expected state
- from user's yaml file
- ex: Replicas field value in deployment object

Action actual of controller mostly happen in the third phase of control loop
1. we call it Reconcile loop or sync loop

## template concept
the template field in deployment is PodTemplate
deployment yaml file consisted with two part
1. controller definition, include expected state
2. the object template that need to be control
