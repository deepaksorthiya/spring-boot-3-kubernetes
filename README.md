# Getting Started

## Requirements:

```
Spring Boot: 3.4.0
Maven: 3.9+
Java: 17
Docker Engine: 25.0.2
Minikube: v1.34.0
```

### Clone this repository:

```bash
git clone https://github.com/deepaksorthiya/spring-boot-3-kubernetes.git
```

```bash
cd spring-boot-3-kubernetes
```

### Build Docker Image(docker should be running):

```bash
./mvnw clean spring-boot:build-image -DskipTests
```

### Start minikube cluster and enable addons(first time only)

```bash
#to check available docker context
docker context list

#use default
docker context use default

#start minikube with docker
minikube start --container-runtime=docker --driver=docker

#enable nginx ingress
minikube addons enable ingress

#enable metrics for dashboard
minikube addons enable metrics-server

#check services via kubectl
kubectl get service --namespace kube-system
```

### Minikube Testing && Useful Commands

```bash
#kubernetes dashboard ui
minikube dashboard

#route minikube-host traffic
minikube tunnel

#load local docker image into minikube cluster if not work try below registry option(take some time to load)
minikube image load deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT
minikube image load deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT

#run all resource file inside k8s dir
kubectl apply -f k8s

#check ingress
kubectl get ingress

#remove resource files inside k8s dir
kubectl delete -f k8s

#docker inside minikube for linux/mac OS
eval $(minikube docker-env)

#docker inside minikube for windows OS
minikube -p minikube docker-env --shell powershell | Invoke-Expression

#remove image by ID
docker rmi deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT
docker rmi deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT

# start alpine for ingress tunnel to access localhost(optional)
docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TCP:host.docker.internal:5000"
```

### This is another way to load image in minikube using local registry

```bash
#enable registry
minikube addons enable registry

#for pushing host docker images to minikube registry
kubectl port-forward --namespace kube-system service/registry 5000:80

#tag docker image for minikube
docker tag deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT localhost:5000/deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT

#push to minikube docker
docker push localhost:5000/deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT

#tag docker image for minikube
docker tag deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT localhost:5000/deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT

#push to minikube docker
docker push localhost:5000/deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT
```

### Testing inside POD

```bash
#get all pods
kubectl get pods

#get all containers in pod
kubectl get pods spring-boot-3-kubernetes-85cf8f4bdf-ptx8r -o jsonpath='{.spec.containers[*].name}'

#logs in pod
kubectl logs spring-boot-3-kubernetes-85cf8f4bdf-ptx8r

#logs in pod with container
kubectl logs spring-boot-3-kubernetes-85cf8f4bdf-ptx8r -c spring-boot-3-kubernetes

#terminal in pod
kubectl exec --stdin --tty spring-boot-3-kubernetes-85cf8f4bdf-ptx8r -- /bin/sh

#terminal in pod with container
kubectl exec --stdin --tty spring-boot-3-kubernetes-85cf8f4bdf-ptx8r -c spring-boot-3-kubernetes -- /bin/sh

##git bash windows
kubectl exec --stdin --tty spring-boot-3-kubernetes-85cf8f4bdf-ptx8r -- //bin//sh


docker exec -it -u root 6912e8c5a4f5 //bin//sh

#for alpline
apk add update && apk add curl
#for linux
apt-get update && apt-get install curl && apt install dnsutils

nslookup spring-boot-3-kubernetes-service

nslookup spring-boot-3-security-with-permission-evaluator-service

curl http://spring-boot-3-kubernetes-service.default.svc.cluster.local:8080/api/headers
curl http://spring-boot-3-security-with-permission-evaluator-service.default.svc.cluster.local:8080/login


curl http://spring-boot-3-kubernetes-service:8080/api/headers
curl http://spring-boot-3-security-with-permission-evaluator-service:8080/login
```

### Users for Testing

```
USER1 ==> Username: user Password: password
USER2 ==> Username: admin Password : admin
```

### Rest APIs

http://localhost/api/headers <br>
http://localhost/login <br>
http://localhost/actuator <br>
http://localhost/admin <br>
http://localhost/server-info

### Reference Documentation

For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/docs/3.2.4/maven-plugin/reference/html/)
* [Create an OCI image](https://docs.spring.io/spring-boot/docs/3.2.4/maven-plugin/reference/html/#build-image)
* [Spring Web](https://docs.spring.io/spring-boot/docs/3.2.4/reference/htmlsingle/index.html#web)

### Guides

The following guides illustrate how to use some features concretely:

* [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
* [Building REST services with Spring](https://spring.io/guides/tutorials/rest/)

