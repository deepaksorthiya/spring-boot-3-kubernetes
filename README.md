[![Java Maven Build Test](https://github.com/deepaksorthiya/spring-boot-3-kubernetes/actions/workflows/maven-build.yml/badge.svg)](https://github.com/deepaksorthiya/spring-boot-3-kubernetes/actions/workflows/maven-build.yml)

# Getting Started

## Requirements:

```
Git: 2.49.0
Spring Boot: 3.5.5
Maven: 3.9+
Java: 21
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

docker image save -o image.tar deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT
minikube image load image.tar
minikube image ls --format table

minikube image load deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT
minikube image load deepaksorthiya/spring-boot-3-security-with-permission-evaluator:0.0.1-SNAPSHOT

#run all resource file inside k8s dir
kubectl apply -f k8s

#check ingress
kubectl get ingress

#test ingress
curl --resolve "localhost:80:$( minikube ip )" -i http://localhost/api/server-info

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

curl http://spring-boot-3-kubernetes-service.default.svc.cluster.local:8080/api/server-info
curl http://spring-boot-3-security-with-permission-evaluator-service.default.svc.cluster.local:8080/login


curl http://spring-boot-3-kubernetes-service:8080/api/server-info
curl http://spring-boot-3-security-with-permission-evaluator-service:8080/login
```

### (Optional)Deploy Using Helm Chart

```sh
cd helm
helm install spring-boot-3-kubernetes --values=values.yaml spring-boot-3-kubernetes
helm install spring-boot-3-kubernetes spring-boot-3-kubernetes
helm uninstall spring-boot-3-kubernetes
```

helm chart create

```sh
cd helm
helm create spring-boot-3-kubernetes
helm lint spring-boot-3-kubernetes
```

### Debug Running Pod

Check Docs [Pod Debug Using Curl](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)

```bash
kubectl run -it --rm --restart=Never busybox --image=busybox sh
kubectl run mycurlpod --image=curlimages/curl -i --tty -- /bin/sh --rm
kubectl run -it --rm --image=curlimages/curl curly -- /bin/sh
```

OR

```bash
kubectl run mycurlpod --image=curlimages/curl -i --tty -- //bin//sh --rm
```

### Rolling Update - Scale Application

Running existing load with 5 or more replicas to see changes. Change replica
in [spring-boot-3-kubernetes-app.yml](k8s/spring-boot-3-kubernetes-app.yml) or directly run

```shell
kubectl scale deployments/spring-boot-3-kubernetes --replicas=5
```

Change image version - Run

```bash
docker image tag deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT deepaksorthiya/spring-boot-3-kubernetes:v1
```

Load new image in Minikube Cluster - Run

```bash
docker image save -o image.tar deepaksorthiya/spring-boot-3-kubernetes:v1
minikube image load image.tar
minikube image ls --format table
```

Update Image to new version and see k8s dashboard where old pods are deleting one by one and new one creating one by one

```shell
kubectl set image deployments/spring-boot-3-kubernetes spring-boot-3-kubernetes=deepaksorthiya/spring-boot-3-kubernetes:v1
```

You can move to previous image using

```shell
kubectl set image deployments/spring-boot-3-kubernetes spring-boot-3-kubernetes=deepaksorthiya/spring-boot-3-kubernetes:0.0.1-SNAPSHOT
kubectl rollout history deployments/spring-boot-3-kubernetes
kubectl rollout undo deployment/spring-boot-3-kubernetes
```

### Users for Testing

```
USER1 ==> Username: user Password: password
USER2 ==> Username: admin Password : admin
```

### Rest APIs

http://localhost <br>
http://localhost/login <br>
http://localhost/actuator <br>
http://localhost/admin <br>
http://localhost/server-info
http://localhost/api/server-info

### Reference Documentation

For further reference, please consider the following sections:

* [Official Apache Maven documentation](https://maven.apache.org/guides/index.html)
* [Spring Boot Maven Plugin Reference Guide](https://docs.spring.io/spring-boot/maven-plugin)
* [Create an OCI image](https://docs.spring.io/spring-boot/maven-plugin/build-image.html)
* [Spring Boot Actuator](https://docs.spring.io/spring-boot/reference/actuator/index.html)
* [Spring Web](https://docs.spring.io/spring-boot/reference/web/servlet.html)
* [Spring Data JPA](https://docs.spring.io/spring-boot/reference/data/sql.html#data.sql.jpa-and-spring-data)
* [Validation](https://docs.spring.io/spring-boot//io/validation.html)
* [Flyway Migration](https://docs.spring.io/spring-boot/how-to/data-initialization.html#howto.data-initialization.migration-tool.flyway)

### Guides

The following guides illustrate how to use some features concretely:

* [Building a RESTful Web Service with Spring Boot Actuator](https://spring.io/guides/gs/actuator-service/)
* [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
* [Building REST services with Spring](https://spring.io/guides/tutorials/rest/)
* [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
* [Validation](https://spring.io/guides/gs/validating-form-input/)
* [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
