# ☸️ Kubernetes

> Frequently used kubectl commands for managing Kubernetes clusters, workloads, and resources.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Cluster & Config](#cluster--config)
- [Namespaces](#namespaces)
- [Pods](#pods)
- [Deployments](#deployments)
- [Services](#services)
- [ConfigMaps & Secrets](#configmaps--secrets)
- [Persistent Volumes](#persistent-volumes)
- [Ingress](#ingress)
- [Nodes](#nodes)
- [RBAC](#rbac)
- [Scaling & Rolling Updates](#scaling--rolling-updates)
- [Troubleshooting](#troubleshooting)
- [Common Manifests](#common-manifests)

---

## Cluster & Config

```bash
# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context my-cluster

# View kubeconfig
kubectl config view

# Set default namespace
kubectl config set-context --current --namespace=myapp

# Check cluster info
kubectl cluster-info

# Check component status
kubectl get componentstatuses
```

---

## Namespaces

```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace myapp
kubectl create ns myapp

# Delete namespace
kubectl delete namespace myapp

# Get resources in a namespace
kubectl get pods -n myapp
kubectl get all -n myapp

# Get resources in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A
```

---

## Pods

```bash
# List pods
kubectl get pods
kubectl get pods -n myapp
kubectl get pods -o wide        # with node info
kubectl get pods --show-labels

# Describe a pod
kubectl describe pod mypod
kubectl describe pod mypod -n myapp

# Get pod logs
kubectl logs mypod
kubectl logs -f mypod                    # follow
kubectl logs mypod -c mycontainer        # specific container
kubectl logs --previous mypod            # previous instance

# Execute into a pod
kubectl exec -it mypod -- bash
kubectl exec -it mypod -c mycontainer -- sh

# Port forward
kubectl port-forward pod/mypod 8080:80

# Delete a pod
kubectl delete pod mypod
kubectl delete pod mypod --grace-period=0 --force

# Create a pod from manifest
kubectl apply -f pod.yaml

# Run a temporary pod
kubectl run tmp --image=busybox --rm -it -- sh
kubectl run nginx --image=nginx --port=80
```

---

## Deployments

```bash
# List deployments
kubectl get deployments
kubectl get deploy -n myapp

# Create a deployment
kubectl create deployment myapp --image=nginx:latest

# Apply from manifest
kubectl apply -f deployment.yaml

# Describe deployment
kubectl describe deployment myapp

# Scale deployment
kubectl scale deployment myapp --replicas=3

# Update image
kubectl set image deployment/myapp container=nginx:1.25

# Rollout status
kubectl rollout status deployment/myapp

# Rollout history
kubectl rollout history deployment/myapp

# Rollback
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=2

# Pause / resume rollout
kubectl rollout pause deployment/myapp
kubectl rollout resume deployment/myapp

# Delete deployment
kubectl delete deployment myapp

# Edit deployment
kubectl edit deployment myapp
```

---

## Services

```bash
# List services
kubectl get services
kubectl get svc

# Create a service
kubectl expose deployment myapp --port=80 --target-port=8080 --type=ClusterIP
kubectl expose deployment myapp --port=80 --type=NodePort
kubectl expose deployment myapp --port=80 --type=LoadBalancer

# Apply from manifest
kubectl apply -f service.yaml

# Describe service
kubectl describe service myapp

# Delete service
kubectl delete service myapp

# Port-forward a service
kubectl port-forward svc/myapp 8080:80
```

**Service types:**

| Type | Description |
|------|-------------|
| `ClusterIP` | Internal cluster access only (default) |
| `NodePort` | Exposes on each node's IP at a static port |
| `LoadBalancer` | Exposes via cloud load balancer |
| `ExternalName` | Maps to an external DNS name |

---

## ConfigMaps & Secrets

```bash
# Create ConfigMap
kubectl create configmap myconfig --from-literal=key=value
kubectl create configmap myconfig --from-file=config.properties

# List ConfigMaps
kubectl get configmaps
kubectl get cm

# Describe ConfigMap
kubectl describe configmap myconfig

# Delete ConfigMap
kubectl delete configmap myconfig

# Create Secret (generic)
kubectl create secret generic mysecret --from-literal=password=s3cr3t
kubectl create secret generic mysecret --from-file=ssh-key=~/.ssh/id_rsa

# Create Secret (docker registry)
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# List Secrets
kubectl get secrets

# Describe Secret
kubectl describe secret mysecret

# Decode a secret value
kubectl get secret mysecret -o jsonpath='{.data.password}' | base64 --decode
```

---

## Persistent Volumes

```bash
# List PersistentVolumes
kubectl get pv

# List PersistentVolumeClaims
kubectl get pvc
kubectl get pvc -n myapp

# Describe PVC
kubectl describe pvc mypvc

# Delete PVC
kubectl delete pvc mypvc

# Apply PV / PVC
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

---

## Ingress

```bash
# List ingresses
kubectl get ingress
kubectl get ing

# Describe ingress
kubectl describe ingress myingress

# Apply ingress
kubectl apply -f ingress.yaml

# Delete ingress
kubectl delete ingress myingress
```

---

## Nodes

```bash
# List nodes
kubectl get nodes
kubectl get nodes -o wide

# Describe node
kubectl describe node node1

# Cordon (stop scheduling new pods)
kubectl cordon node1

# Uncordon
kubectl uncordon node1

# Drain node (evict pods before maintenance)
kubectl drain node1 --ignore-daemonsets --delete-emptydir-data

# Label a node
kubectl label node node1 env=production

# Taint a node
kubectl taint nodes node1 key=value:NoSchedule
kubectl taint nodes node1 key=value:NoSchedule-   # remove taint
```

---

## RBAC

```bash
# Create service account
kubectl create serviceaccount mysa -n myapp

# Create role
kubectl create role myrole --verb=get,list,watch --resource=pods -n myapp

# Create role binding
kubectl create rolebinding myrb \
  --role=myrole \
  --serviceaccount=myapp:mysa \
  -n myapp

# Create cluster role
kubectl create clusterrole myclusterrole --verb=get,list,watch --resource=pods

# Create cluster role binding
kubectl create clusterrolebinding mycrb \
  --clusterrole=myclusterrole \
  --serviceaccount=myapp:mysa

# Check permissions
kubectl auth can-i get pods --as=system:serviceaccount:myapp:mysa -n myapp
```

---

## Scaling & Rolling Updates

```bash
# Manual scale
kubectl scale deployment myapp --replicas=5

# Autoscale (HPA)
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80

# List HPAs
kubectl get hpa

# Update strategy (set in manifest):
# RollingUpdate: gradual replacement (default)
# Recreate: all pods terminated before new ones created

# Set resource limits
kubectl set resources deployment/myapp \
  --limits=cpu=500m,memory=256Mi \
  --requests=cpu=100m,memory=128Mi
```

---

## Troubleshooting

```bash
# Get events
kubectl get events -n myapp --sort-by='.lastTimestamp'

# Check pod status
kubectl get pod mypod -o yaml

# Check node resources
kubectl top nodes

# Check pod resource usage
kubectl top pods
kubectl top pods -n myapp

# Debug networking with a test pod
kubectl run nettest --image=busybox --rm -it -- sh
  # inside pod:
  wget -qO- http://myservice.myapp.svc.cluster.local

# Copy files from pod
kubectl cp mypod:/app/logs/app.log ./app.log

# API resources
kubectl api-resources
kubectl api-versions

# Apply with dry-run
kubectl apply -f manifest.yaml --dry-run=client

# Diff current vs manifest
kubectl diff -f manifest.yaml
```

---

## Common Manifests

**Pod:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: myapp
spec:
  containers:
  - name: app
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

[← Back to Home](../README.md)
