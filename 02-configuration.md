# CKAD Study Guide - Kubernetes Configuration

## Resource Quota - Namespace restriction on total resource usage

kubernetes.io bookmark: [Create a ResourceQuota](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/#create-a-resourcequota)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota  
spec:
  hard:
    cpu: 500Mi
    memory: 2G
EOF
```

## Limit Range - A policy to constrain resource allocations (to Pods or Containers) in a namespace

kubernetes.io bookmark: [Create a ResourceQuota](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/#create-a-resourcequota)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: my-limit-range  
spec:
  limits:
  - default:
      cpu: 250m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container 
EOF
```

## Configuration Map - Storing environmental variables per namespace

kubernetes.io bookmark: [Define a container environment variable with data from a single ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-a-container-environment-variable-with-data-from-a-single-configmap)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap  
data:
  error-log-level: notice
EOF
```

## Create a configmap named config with values foo=lala,foo2=lolo

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

## Display configmap values

```bash
kubectl get cm config -o yaml
# or
kubectl describe cm config
```

## Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

## Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml
```

## Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml
```

## Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

```bash
kubectl create cm options --from-literal=var5=val5
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    env:
    - name: option # name of the env variable
      valueFrom:
        configMapKeyRef:
          name: options # name of config map
          key: var5 # name of the entity in config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # will show 'option=val5'
```

## Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    envFrom: # different than previous one, that was 'env'
    - configMapRef: # different from the previous one, was 'configMapKeyRef'
        name: anotherone # the name of the config map
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

## Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # your volume mounts are listed here
    - name: myvolume # the name that you specified in pod.spec.volumes.name
      mountPath: /etc/lala # the path inside your container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # will show var8 var9
cat var8 # will show val8
```

## Secret 

## Storing obfuscated environmental variables per namespace

kubernetes.io bookmark: [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

kubernetes.io bookmark: [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io bookmark: [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

## Create a secret called mysecret with the values password=mypass

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

## Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

```bash
kubectl create secret generic mysecret2 --from-file=username
```

## Get the value of mysecret2

```bash
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
```

Alternative using `--jsonpath`:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d  # on MAC it is -D
```

Alternative using `--template`:

```bash
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d  # on MAC it is -D
```

Alternative using `jq`:

```bash
kubectl get secret mysecret2 -o json | jq -r .data.username | base64 -d  # on MAC it is -D
```

## Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # our volume mounts
    - name: foo # name on pod.spec.volumes
      mountPath: /etc/foo #our mount path
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/bash
ls /etc/foo  # shows username
cat /etc/foo/username # shows admin
```
