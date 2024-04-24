# Kubernetes Deployments

## 01. Create, List and Describe `Deployments`

# Scale Up the Deployment

kubectl scale --replicas=20 deployment/<Deployment-Name>
kubectl scale --replicas=20 deployment/my-first-deployment

### Step-00: Deployment manifest Syntax

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <name_of_deployment>
spec:
  replicas: <no_of_replicas>
  selector:
    matchLabels:
      <key>: <value>
  template:
    metadata:
      <name>: <value>
      labels:
        <key>: <value>
    spec:
      containers:
        - name: <container_name>
          image: <image_name>
          ports:
            - containerPort: <port_no>
```

### Step-01: Create a Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.5
        ports:
        - containerPort: 80
```

- Execute the above manifest to create a Deployment

```
kubectl apply -f nginx-deployment.yaml
```

### Step-02: Get the list of all the Deployments

```
kubectl get deployments

# Deployments alias is "deploy"
kubectl get deploy
```

- Notice the desired replicas and actual replica count + Age

### Step-03: Check the replicaSet created by the Deployment

```
kubectl get rs

OR

kubectl get replicasets
```

- **Notice that the name of the ReplicaSet is always formatted as [DEPLOYMENT-NAME]-[HASH]**.
- This name will become the basis for the Pods which are created.

### Step-04: To see the labels automatically generated for each Pod

```
kubectl get pods --show-labels
```

### Step-05: Describe a Deployment

```
# Syntax
kubectl describe deployment <deployment-name>

# Example
kubectl describe deployment nginx-deployment
```

## 02. `Scaling` a Deployment

### Step-2.1: Scale a deployment

```
# Syntax
kubectl scale --replicas=<replica_count> deployment/<deployment_name>

# Example (scale-up)
kubectl scale --replicas=10 deployment/nginx-deployment

# Example (scale-down)
kubectl scale --replicas=2 deployment/nginx-deployment
```

### Step-2.2: Verify the deployment after scaling

```
# List the Deployments
kubectl get deployments

# List the replicasets
kubectl get replicasets

# List the Pods
kubectl get pods
```

## 03. Expose a Deployment as a Service (nodePort)

```
# Syntax
kubectl expose deployment <deployment_name> --type=NodePort --port=<port> --target-port=<port> --name=<service_to_be_created>

# Example
kubectl expose deployment nginx-deployment --type=NodePort --port=80 --target-port=80 --name=webapp-np-service

# Get Service details
kubectl get svc

# Kindly make a note of port which starts with 3 (Example: 80:3xxxx/TCP). We'll need it in our next step.

# Get Public IP of Worker Nodes
kubectl get nodes -o wide

# Kindly make a note of "EXTERNAL-IP" of a worker node
```

- Accessing the application hosted in Deployment

```
http://<any_worker_node_public_ip>:<node_port>
```

## 04. Update a Deployment

## 05. Rollback a Deployment
