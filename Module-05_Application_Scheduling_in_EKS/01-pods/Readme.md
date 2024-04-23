# Kubernetes Pods

## 01. Working with K8s Pods

### Step-01: Create a Pod

**Create a Pod using imperative approach**

```
# Syntax to create a Pod
kubectl run <pod-name> --image=<image-name:image-tag>

# Example-01: Create a Pod with nginx:alpine image
kubectl run bindesh-nginx-pod --image=nginx:alpine

# You will see the output is returned as created

# Example-02: Create a Pod with busybox image
kubectl run busybox --rm -it --image=busybox /bin/sh

# You can also deploy a nginx image and then export the YAML definition
kubectl run nginx --image=nginx --dry-run -o yaml > pod-sample.yaml
```

**Create a Pod using declarative approach**

- `Create a YAML file with the Pod specification`, as follows:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

- `Deploy a Kubernetes Pod manifest`

```
kubectl apply -f <k8s_manifest_filename>.yaml
```

### Step-02: List Pods

```
# List existing Pods
kubectl get pods

# Alias name for pods is "po"
kubectl get po

# List Pods with more details
kubectl get pods -o wide

# List Pods present in all the namespaces
kubectl get pods -A
```

### Step-03: Describe Pods

```
# Syntax: To describe a Pod
kubectl describe pod <pod_name>

# Example: To describe a Pod
kubectl describe pod bindesh-app-pod
```

### Step-04: Run a command in a Pod

```
# Syntax: Connect to a container in a Pod
kubectl exec -it <pod_name> -- <command_to_execute>

# Example
kubectl exec -it bindesh-app-pod -- /bin/bash

# Example
kubectl exec my-pod -- ls /
```

### Step-05: Dump the `Pod logs`

```
# Syntax: To dump Pod logs
kubectl logs <pod_name>

# Example: To dump Pod logs
kubectl logs bindesh-app-pod

# Example: To dump the pod log files in the namespace if you have multiple containers
kubectl logs my-pod -c <container_name>

# Stream pod logs with -f option and access app to see the logs
kubectl logs -f <pod_name>
```

### Step-06: Delete a Pod

```
# Syntax: To delete a Pod
kubectl delete pod <pod_name>

# Example: To delete a Pod
kubectl delete pod bindesh-app-pod

```

## 02. Expose a Pod (application) with a `Service`

- Expose a Pod with a _service_ (NodePort service) to access the application from the web.

### Step-01: Create a Pod

```
# Create a Pod with nginx:alpine image
kubectl run bindesh-nginx-pod --image=nginx:alpine

# Get the list of Pods
kubectl get pods
```

### Step-02: Expose Pod as a Service

```
kubectl expose pod my-first-pod  --type=NodePort --port=80 --name=my-first-service
```

### Step-03: Get Service Info

### Step-04: Accessing the application (Pod)

```
# Get public IP of the worker nodes
kubectl get nodes -o wide

# Accessing the application
http://<node1_public_ip>:<node_port>

```
