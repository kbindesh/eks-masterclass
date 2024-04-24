# Kubernetes ReplicaSets

## 01. Working with ReplicaSets

### Step-1.0: Syntax - replicaSet manifest

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: <name_of_replicaset>
  labels:
	  key: value
spec:
  replicas: <no_of_replicas>
  selector:
	  matchLabels:
  	  key: value
  template:
	  metadata:
  	  labels:
    	  key:value
	spec:
    containers:
      - name: <container_name>
        image: <container_img>:<tag>
```

### Step-1.1: Create a ReplicaSet

- Create a replicaSet manifest, say _replicaset.yaml_

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

- Now execute the above manifest to create a replicaset

```
kubectl apply -f <name_of_manifest>.yaml
```

### Step-1.2: Get a List of ReplicaSets

```
kubectl get replicasets

OR

kubectl get rs
```

### Step-1.3: Describe a ReplicaSet

```
kubectl describe replicaset <name_of_replicaset>

# Example
kubectl describe replicaset frontend
OR
kubectl describe rs/frontend
```

## 02. Exposing the replicaSet on Web using Service (nodePort)

### Step-2.1: Exposing the replicaSet on the web

```
kubectl expose rs/frontend --type=NodePort --port=80 --name=nodeport-svc

# Get the list of all the services | You'll find a nodePort service -> nodeport-svc
kubectl get services
OR
kubectl get svc

# Copy the node port no. from the above command result
```

### Step-2.2: Accessing the application

```
# Accessing the application
http://<worker_node_public_ip>:<node_port_from_step_2.1>

```

## 03. High-availability (HA) with replicaSets

### Step-3.1: Create a replicaSet

- Create a replicaSet manifest, say _replicaset.yaml_

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

- Now execute the above manifest to create a replicaset

```
kubectl apply -f <name_of_manifest>.yaml
```

### Step-3.2: Delete a Pod present in a replicaSet manually (to mimic pod failure)

```
# List all the pods belongs to a replicaSet
kubectl get pods

# Delete one of the pod from the listing | Automatically a new Pod will be created
kubectl delete pod <pod_name>
```

### Step-3.3: Check the replicaSet's desired state (no. of Pods)

```
# Now, again check the list of pods belongs to a replicaset
kubectl get pods

# Notice the Pod name & Age column values
```

## 04. Scaling application with replicaSet

### 4.1 Scale replicaSet (imperative way)

```
# Syntax
kubectl scale rs/<replicaset_name> --replicas=<updated_replica_count>

# Example
kubectl scale rs/frontend --replicas=5
```

### 4.2 Scale replicaSet (declarative way)

- Simply update the `replicas` field value in replicaSet manifest from **3** to any other value, say **5**.

```
...
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
...

```

## 05. Deleting a replicaSet

```
# Delete a ReplicaSet
kubectl delete rs <replicaset_name>

# Example-01
kubectl delete rs/frontend

# Example-02
kubectl delete rs frontend

# Example-03
kubectl delete replicaset <rs_name>

# Verify if ReplicaSet is deleted | here "rs" is replicaset alias
kubectl get rs
```
