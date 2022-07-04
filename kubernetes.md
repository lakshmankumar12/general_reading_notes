# links

youtube link: https://www.youtube.com/watch?v=d6WC5n9G_sM&t=2s&ab_channel=freeCodeCamp.org
        Current at 9:22

https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
https://kubernetes.io/docs/concepts/overview/components/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/concepts/architecture/nodes/

# terminology

* pod: minimum deployment unit in kubernetes.
    * may have one or more containers. Usually one is common.
    * a pod has to be on the same server.
    * all continers in the pod share the shared-volumes and network-space.
    * has its own ip address and reacheable from rest of the system
        * outside world access isn't always turned on unless configured.
        * within the pod, localhost can used by containers to communicate with each other
* Node:
    * collection of pods.
    * (one machine - physical or virtual)
* Cluster: Collectionof nodes.
    * Typically all in same data-center
        * but technically the nodes can be anywhere though.
    * has one master node and other worker nodes
    * master node:
        * api-server (front-ends all kubectl commands)
        * controller
        * scheduler
            * assigns pods to nodes 
        * cluster store(kv)
            * runs etcd
    * worker node. The first 3 are mandatory in all nodes.
        * kubelet
            * takes the pod-spec and ensures containers are running accordinly
            * the main-brain inside the node
        * container runtime
            * docker or whatever
        * kube-proxy
            * manages the networking and connections
        * Actual pods
* spec, state
    * spec tells the desired state of the system

# minikube

Minikube is a lightweight Kubernetes implementation that creates a VM on your
local machine and deploys a simple cluster containing only one node.

```
minikube start

minikube service servicename

```


# Sample deployment file

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anything-say-ngnix-deployment
spec:
  selector:
    matchLabels:
      app: ngnix-app
  replicas: 2
  template:
    metadata:
      labels:
        app: ngnix-app
    spec:
      containers:
      - name: nginx
        image: ngnix:1.14.2
        ports:
        - containerPort: 80
```

```yml
apiVerson: apps/v1
kind: service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

# kubectl commands


```
#deploy
kubectl apply -f deployment.yml
kubectl delete deployment deploymentname
kubectl rollout restart deploy deploymentname

kubectl get pods
kubectl get pods -o wide

kubectl describe pod
kubectl describe service

kubectl get service

```

