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

### Step-1.4: Check the owner of the Pod (replicaSet it belongs to)

- Verify the owner reference of the pod.
- Verify under "name" tag under "ownerReferences". We will find the replicaset name to which this pod belongs to.

```

```

## 02. Exposing the replicaSet on Web using Service (nodePort)

### Step-2.1: Exposing the replicaSet on the web

### Step-2.2: Accessing the application

## 03. High-availability (HA) with replicaSets

### Step-3.1: Create a repicaSet

### Step-3.2: Delete a Pod present in a replicaSet manually (to mimic pod failure)

### Step-3.3: Check the replicaSet's desired state (no. of Pods)

## 04. Scaling application with replicaSet

## 05. Deleting a replicaSet
