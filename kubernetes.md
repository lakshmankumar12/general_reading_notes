# links

youtube link: https://www.youtube.com/watch?v=d6WC5n9G_sM&t=2s&ab_channel=freeCodeCamp.org
        Current at 9:22

https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
https://kubernetes.io/docs/concepts/overview/components/
https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/concepts/architecture/nodes/

Wiki has the info pretty laid out - https://en.wikipedia.org/wiki/Kubernetes

Good read on other tools in the kubernetes ecosystem: https://www.densify.com/kubernetes-tools

# Main componenets of Kubernetes

* pod: minimum deployment unit in kubernetes.
    * may have one or more containers. Usually one is common.
    * a pod has to be on the same server.
    * ephemeral, disposable
    * not restarted by scheduler by itself
    * all continers in the pod share the shared-volumes and network-space.
    * has its own ip address and reacheable from rest of the system
        * outside world access isn't always turned on unless configured.
        * within the pod, localhost can used by containers to communicate with each other
        * this allows other instances of the app to bind to the same port on different pods.
* Node:
    * collection of pods.
    * (one machine - physical or virtual or even a umbrella container (eg: minikube))
* Cluster: Collection of nodes.
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
            * the main-brain inside the node
            * agent that runs on each node communicating with api-server
                * takes the pod-spec and ensures containers are running accordingly
            * executes pod-containers via container engine
            * mounts and run pod-volumes and secrets
            * executers health status of pods/volumes and reports back to api-server
        * container runtime
            * docker or whatever
        * kube-proxy
            * manages the networking and connections
            * reflects services as defined on each node
            * network stream or round-robin forwarding across a set of backend containers
            * 3 modes
                * User-space mode
                * iptables mode
                * ipvs mode
        * Actual pods
* spec, state
    * spec tells the desired state of the system

## controller

* Replication Controller
    * Older. Now use replicaset
* ReplicaSet
    * Ensures specified number of pods are running at all times
* Deployments
    * mm.. deployments manages replica-sets which in turn manages pods
    * pause(updates) and resume use-cases.
* daemonset controller
    * ensures all nodes run a specific pod
* job controller
    * like cron-job. Run one process to completion
* Services
    * Allow connectivity between one set of deployments with another
      in a seamless way by anchoring the IP to use.
        ```
                                            +---> Backend Pod 1
                                            |
        FrontEnd Pod ---> Backend Service --+---> Backend Pod 2
                                            |
                                            +---> Backend Pod 3
        ```
    * Can be internal or external.
        * External exposes a node-ip:node-port
        * Load balancer: exposes a application to internet

## Pod States

* Pending - accepted by kubernetes system.
* ContainerCreating
* Running
* Succeeded
* Failed
* CrashLoopBackOff

## PodSpec

* Yaml file that describes a pod

### Sample file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: first-pod
  labels:
    project: qsk-course
spec:
  containers:
    - name: web
      image: educative1/qsk-course:1.0
      ports:
        - containerPort: 8080
```


## Deployment / ReplicaSet

* Resource object that defines how pods should be started
* Helps in the self-healing goal of kubernetes
* A deployment is a collection of replica-sets
* When you update a image, the current replica-sets are replaced with new ones.

### Sample deployment file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anything-say-ngnix-deployment
spec:
  replicas: 2                # tells deployment to run 2 pods matching the template
  selector:
    matchLabels:
      app: ngnix-app
  template:                  # create pods using pod definition in this template
                             # kind of taking a pod.yml here.
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

## service

* Logical abstraction for a collection of pods that function exactly alike
* Pods are ephemeral - so service offers a frontend to the pods to other functions
* This is how the service to pod mapping works:
  ```
  service.yml                                      pod.yml
  spec:                                            metadata:
    selector:             -- matched with -->        label:
      project: name-to-match                           project: name-to-match
  ```
  * Note that that is one-to-many mapping. The service forwards the traffic to all
    pods with the matching label.
  * Kubernets keeps track of which pods are available and which pods are gone.

### Service types

* ClusterIP
    * This is the default
    * (to understand better)Exposes a service which is only accessible from within the cluster.
    * you can use forward-port to expose localhost to the cluster
* NodePort
    * Exposes a service via a static port on every node’s IP.
    * you can use forward-port to expose localhost to the nodeport
* LoadBalancer
    * Exposes the service via the cloud provider’s internet-facing load balancer.
        * If your cluster runs on AWS, this Service will automatically provision
          an AWS Network Load Balancer (NLB) or Classic Load Balancer (CLB).
* ExternalName
    * Maps a service to a predefined externalName field by returning a value for the CNAME record.

### Sample service file

```yaml
apiVerson: apps/v1
kind: service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### Explanation of get output


* The CLUSTER-IP value is an IP address on the internal Kubernetes Pod
  network and is used by other Pods and applications running on the
  cluster. You won’t be connecting to that address.
* The Kubernetes service is used internally by Kubernetes for Service discovery.

```sh
NAME                                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                   AGE
bootstrapper-orc8r-nginx-proxy          LoadBalancer   10.101.99.35     <pending>     80:31638/TCP,443:31479/TCP,8444:32159/TCP 4h15m
db                                      NodePort       10.107.209.211   <none>        5432:31184/TCP,5433:31046/TCP             4h16m
domain-proxy-configuration-controller   ClusterIP      10.108.80.166    <none>        8080/TCP                                  4h15m
kubernetes                              ClusterIP      10.96.0.1        <none>        443/TCP                                   17h
```


## Labels

* Key value pairs
* Eg:
  ```
  "release" : "stable", "release" : "canary"
  ```
* Label selectors
    * Equality
    * Set based - IN, NOTIN, EXISTS

## Namespaces

* All objects are placed in default namespace at start

## probes

* readiness probe
* liveliness probe


# kubectl commands

## environment

```sh
# this env variable give kubectl information on
# where the cluster runs and the auth details
export KUBECONFIG=/usercode/config
```


## General notes

* apply command
    * takes the yaml file and sends to api server
* get and describe
    * state information about the object
    * get give brief info, describe gives detailed info

## Commands cheat-sheet

* Kubectl own cheatsheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

```sh

# config
kubectl config view

# cluster related
kubectl cluster-info

# node related
kubectl get nodes

# pod related
kubectl get all
kubectl get pods --show-labels
kubectl get pods --selector keyname=value
kubectl get pods -l 'keyname in (value1,value2)'  # notin
kubectl get pods -o wide
kubectl describe pod
kubectl describe pod/pod_name
kubectl label pod/pod_name keyname=value --overwrite
kubectl label pod/pod_name keyname-  # deletes the label keyname
##connecting into a pod
kubectl exec -it pod/pod_name -c container_name_in_pod /bin/bash
kubectl delete pod pod_name

# deployment related
kubectl apply -f deployment.yml
kubectl get deployment/helloworld -o yaml | less
kubectl describe deployment/deployment_name
kubectl expose deployment helloworld --type=NodePort
kubectl delete deployment deployment_name
kubectl rollout history deployment/deployment_name
kubectl rollout undo deployment/deployment_name
kubectl rollout restart deploy deploymentname
#other args
# --to-revision=N       => to that revision as reported in history


# service related
kubectl get service
kubectl describe service
kubectl delete svc service_name

# General creation(?)
kubectl create -f <file.yml>
#other args to create
# --record          => keeps a log of changes.

kubectl logs pod/pod_name

# port forward to a pod
kubectl port-forward --address <address> <podname> port1:port2
## --address <>     -- to understand this. I have seen only 0.0.0.0
## podname or svcname
## port1            -- port of the location machine
## port2            -- port of the pod where its offering service
# to a service
kubectl port-forward --address <address> <svcname> port1:port2

## list all port-forwards
kubectl get svc -o json | jq '.items[] | {name:.metadata.name, p:.spec.ports[] } | select( .p.nodePort != null ) | "\(.name): localhost:\(.p.nodePort) -> \(.p.port) -> \(.p.targetPort)"'

```

# AWS and kubectl

* Install aws-cli : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions

```
temp_dir=$(mktemp -d /tmp/install-aws-XXXX)
echo "temp_dir is $temp_dir"
curr_dir=$(pwd)
cd $temp_dir
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

#This should now work
aws --version

cd $curr_dir
rm -rf $temp_dir
```

* aws configure : https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html
* setting up aws to a cloudinstance (step-2): https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html

# Minikube

minikube is a lightweight Kubernetes implementation that creates a VM/docker on your
local machine and deploys a simple cluster containing only one node.


* Link to instal kubernets/minibox within a vm
  https://webme.ie/how-to-run-minikube-on-a-virtualbox-vm/


```sh
minikube start

minikube service list
minikube service servicename

#destroy everthing
minikube delete --all
```

# Cloud-native application

A cloud-native application must :
* Scale on demand
* Self-heal
* Support zero-downtime rolling updates
* Run anywhere that has Kubernetes
