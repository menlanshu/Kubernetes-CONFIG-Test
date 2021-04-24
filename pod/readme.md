# initContiner
1. init container will start before spec.containers
2. init container will start follow order on by one
3. only when they started and exit, user container will start

java_war_tomcat.yaml
step 1: war container start, cp /sample.war /app
step 2: war container exit
step 3: tomcat mount the same volume to webapps folder
step 4: when tomcat container start, sample.war file will be in webapps folder already

The coupling of war and tomcat was solved by this combination action

This pattern in container designis called: sidecar
Init Container is the charactor of sidebar

Another example of two container in one pod is log collection

[Design Pattern for containerbased distributed systems](https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns)
