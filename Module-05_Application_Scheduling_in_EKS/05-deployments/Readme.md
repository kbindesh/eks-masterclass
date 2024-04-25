# Kubernetes Deployments

## 01. Create, List and Describe `Deployments`

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

## 04. Perform Rolling updates (updating deployment)

- Rolling updates provide a way to update a Deployment to a newer version more effectively and efficiently.
- This way, you can update Kubernetes objects such as replicas and pods gradually with nearly zero downtime.

### Rolling updates (imperative way)

```
# Create a new kubernetes deployment
kubectl create deployment <deployment_name> --image=<image_name>:<tag>

kubectl create deployment app-deployment --image=nginx:latest

# Update the container image
kubectl set image deployment/app-deployment nginx=nginx:1.25.5 --record

# --record flag records information about the updates so that it can be rolled back later.

# You can either use --record flag or --record=true flag.

# Describe deployment to check the image update
kubectl describe deployment app-deployment
```

### Rolling updates (declarative way)

- Create a deployment manifest, say _nginx-deployment.yaml_ (refer [manifest folder](./manifests/nginx-deployment.yaml) for sample deployment.yaml file)

- Execute the manifest by running following command

```
kubectl apply -f nginx-deployment.yaml
```

- Now, update the container image (from nginx:latest to nginx:1.25.5) by editing the manifest.
- Deploy the updated deployment manifest:

```
kubectl apply -f nginx-deployment.yaml
```

- Now, verify the deployment image update

```
# Describe deployment to check the image update
kubectl describe deployment nginx-deployment
```

## 05. Rollback a Deployment

- Rollback allows us to revert our application to a previous state (deployment) seamlessly.

### Performing quick rollback to previous revision

```
# To quickly recover if you need to perform a rollback
kubectl rollout undo deployments <deployment_name>

kubectl rollout undo deployments nginx-deployment

# To verify the rolled back image (prev)
kubectl describe deployment <deployment_name>
```

### Keeping track of Deployment history

```
kubectl rollout history deployment <deployment_name>
```

### Rollback to a specific version (using `--to-revision` flag)

```
# Syntax
kubectl rollout undo deployment <deployment_name> --to-revision=<revision_number>

# Example
kubectl rollout undo deployment nginx-deployment --to-revision=2

# To verify the rollback, simply check the image by describing the deployment
kubectl describe deployment <deployment_name>
```

## 06. Scale a Deployment

- When our application becomes popular, in order to handle increasingly on-demand requests, we need to spin up multiple instances of applications (Pods) to satisfy the workload requirements.

- When you have a Deployment, scaling is achieved by changing the number of replicas.

```
# Syntax
kubectl scale deployment <deployment_name> --replicas=<new_replica_count>

# Example
kubectl scale deployment nginx-deployment --replicas=10
```

- Apart from manually scaling the Deployments with the **kubectl scale** command, we also have another way of scaling a Deployment and its ReplicaSets, which is **HorizontalPodAutoscaler (HPA)**.
