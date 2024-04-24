# CKAD Study Guide - Kubernetes Configuration

- Resource Quota - Namespace restriction on total resource usage

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

- Limit Range - A policy to constrain resource allocations (to Pods or Containers) in a namespace

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

- Configuration Map - Storing environmental variables per namespace

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

- Secret - Storing obfuscated environmental variables per namespace

kubernetes.io bookmark: [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```yaml
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: my-secret  
data:
  password: MTIzNDU2
  user: Ym9i
kind: Secret
metadata:
  name: my-secret
EOF
```
