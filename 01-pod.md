# CKAD Study Guide - Kubernetes Core Concepts

- Pod - A pod is the smallest execution unit in Kubernetes
- Limits and Requests - CPU and Memory reservation for a Pod
- Liveness and Readiness - Health Checks for a Pod
- ServiceAccount - Functional ID inside the Pod to connect to the API server
- Security Context - Linux privilege and access control settings for a Pod or Container

![https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.png](https://github.com/cirrostech/ckad-study-guide/blob/main/image/pod.jpg)

## What is a Kubernetes Namespace?
* In Kubernetes, a namespace is a way to divide cluster resources between multiple users (via resource quotas). 
* Namespaces are also used to isolate resources within a cluster, so that different teams or projects can use the same names for resources without conflicting with each other. 
* In short, namespaces are a way to organize and divide resources in a Kubernetes cluster, and can be used to isolate resources and set resource quotas.

kubernetes.io bookmark: [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

### Create a namespace with the name: `ns-bootcamp-pod`

```bash
kubectl create namespace ns-bootcamp-pod
kubectl config set-context --current --namespace=ns-bootcamp-pod
```

### Get the YAML for a new namespace called 'myns' without creating it
```bash
kubectl create namespace myns -o yaml --dry-run=client
```

## What is a Kubernetes Pod?
* In Kubernetes, a pod is the smallest deployable unit. 
* It is the basic execution unit of a Kubernetes application–the smallest and simplest unit in the Kubernetes object model that you create or deploy.
* A pod consists of one or more containers, and is used to host the containers that make up an application.

kubernetes.io bookmark: [Using Pods](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods)

### Run a Pod called `my-pod` using the `nginx` image exposing port 80

```bash
# Create the pod via the command line imperatively
kubectl run my-pod --image=nginx:1.7.1 --port=80
```

### Get pod's YAML

```bash
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
```

### Get information about the pod, including details about potential issues

```bash
kubectl describe po nginx
```

### Get pod logs

```bash
kubectl logs nginx
```

### If pod crashed and restarted, get logs about the previous instance

```bash
kubectl logs nginx -p
# or
kubectl logs nginx --previous
```

### Change pod's image to nginx:1.20.0. Observe that the container will be restarted as soon as the image gets pulled

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.20.0
kubectl describe po nginx # you will see an event 'Container will be killed and recreated'
kubectl get po nginx -w # watch it
```

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'
```bash
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

##### Note: you can check pod's image by running
```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

```bash
# Create a  YAML template with this command
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
```

```bash
# cat envpod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

```

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
# or
kubectl run nginx --image nginx --restart=Never --env=var1=val1 -it --rm -- sh -c 'echo $var1'
```

### Start a hazelcast pod and let the container expose port 5701

```bash
kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
```

### Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the container

```bash
kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
```

### Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container

```bash
kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"
```

### Start a busybox pod and keep it in the foreground, don't restart it if it exits

```bash
kubectl run -i -t busybox --image=busybox --restart=Never
```

## What is a Kubernetes Limit and Request?
* In Kubernetes, a limit is the maximum amount of resources (such as CPU or memory) that a container is allowed to use. 
* A request is the minimum amount of resources that a container is guaranteed to be allocated.
* When you create a pod in Kubernetes, you can specify resource limits and requests for the containers in the pod. 
* The resource limits define the maximum amount of resources that a container is allowed to use. 
* If a container exceeds its resource limits, it may be terminated by the Kubernetes system.
* The resource requests define the minimum amount of resources that a container is guaranteed to be allocated. 
* When the Kubernetes scheduler places a pod on a node, it ensures that the node has enough resources available to meet the resource requests of all of the containers in the pod.
* By setting resource limits and requests, you can ensure that your containers have the resources they need to run properly, and that the resources in your cluster are being used efficiently.

kubernetes.io bookmark: [Meaning of Memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory)

### Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

### Create a Pod with Limits and Requests

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:1.20.0
    ports:
    - containerPort: 80
    resources: # CPU & RAM Resources
      requests: # Minimum “Request”
        memory: "64Mi"  # 64Mi = 64 Megabyte
        cpu: "250m" # 250m = 250 milliCPU
      limits:  # Maximum “Limit”
        memory: "128Mi" # 128Mi = 128 Megabyte
        # cpu: "500m" # 500m = 500 milliCPU (½ CPU)
EOF
```



