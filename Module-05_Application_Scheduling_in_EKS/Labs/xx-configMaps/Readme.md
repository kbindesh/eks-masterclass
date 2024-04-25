# Kubernetes ConfigMaps

- A **ConfigMap** is simply a Kubernetes object that stores configuration data in key-value pairs.
- This configuration data can then be used to:
  - Configure the software running in a container by configuring a pod to consume ConfigMaps using environment variables
  - command-line arguments
  - mounting a volume with configuration files

## How a ConfigMap works?

### Step-01: Create a configMap

- Create a new k8s manifest for ConfigMap, say _bin-configmap.yaml_

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-configmap
data:
  username: "DB_ADMIN"
  HomeDir: "/usr/dir"
```

- Execute the above created ConfigMap manifest:

```
kubectl apply -f bin-configmap.yaml
```

### Step-02: List all the ConfigMaps

```
kubectl get configmap

# ConfigMap alias is "cm"
kubectl get cm
```

### Step-03: Describe a ConfigMap

```
kubectl describe configmap <configmap_name>
```

### Step-03: Different ways of using a ConfigMap

**3.1 Inside a container command and args**

**3.2 Environment variables for a container**

**3.3 Add a file in read-only volume, for the application to read**

**3.4 Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap**

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        - name: PLAYER_INITIAL_LIVES
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: player_initial_lives # The key to fetch.
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
  - name: config
    configMap:
      name: game-demo
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"

```
