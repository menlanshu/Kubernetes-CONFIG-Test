# stateful applciations
databases
applications that stores data

# stateless applciation
don't keep record to state
each request is completedly new

# implementation
stateless application
- deployment

stateful applicatino
- StatefulSet

# differnence
mysql => handles request from a java my-app
scaling mysql pods => how we should to this?
 sticky identity for each pod
- created from same specification, but not interchangebale- why is the identity necessary
mysql-0 => master => /data/vol/pv-0
mysql-1 => worker => /data/vol/pv-1
mysql-2 => worker => /data/vol/pv-2
- data inconsistency
- the worker must know about each chagne to be up-to-date
 > continous sychronous
- but data will lost when all pods die
 > data need survive, even when all pods die
 > persistent volume lifecycle
- remote pv is good


Deployment: random hash
Statefuleset: fixed ordered numbers

1. predictable pod name
2. fixed individual DNS name
