# Resource Management

- K8s allows us to specify the resource requirements of a container in the pod specs.
- _kube-scheduler_ uses the resource request information that you specify for a container in a pod to decide on which worker node to schedule the pod.
- Resource request refers to the amount of resources that are necessary to run a container, and what kubernetes does is govern on which worker node the containers will actually be scheduled.

## How Resource management works with Pods

- It is up to _kubelet_ to enforce these resource limits when you specify them for the containers in the pod so that the running container can go beyond a set limit
- _kubelet_ also make sure that it reserves at least the requested resource amount of a system resource for a container to use.
- Resource specs are usually defined by following Pod attributes:
  - **resources.limits.cpu** is the resource limit set on CPU usage.
  - **resources.limits.memory** is the resource limit set on memory usage.
  - resources.requests.cpu is the minimum CPU usage requested to allow your application
  - to be up and running.
    **resources.requests.memory** is the minimum memory usage requested to allow your application to be up and running. In the case that a container exceeds its memory request, the worker node that it runs on becomes short on overall memory at the same time, and the pod that the container belongs to is likely to be evicted too.
  - **resources.limits.ephemeral-storage** is the limit on ephemeral storage resources.

### Step-01: Create a Pod with resource limits

- Create a k8s manifest for a Pod with Container resource requirements (cpu, memory)

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-res-limit
spec:
  containers:
    - name: bin-container
      image: busybox
      command: ["sh", "-c", "echo stay tuned! && sleep 3600"]
      resources:
        requests:
          memory: "64Mi" # 64 Megabytes
          cpu: "250m" # 250 millicpu or millicores
        limits:
          memory: "128Mi"
          cpu: "500m"

```

- Execute the above pod manifest

```
kubectl apply -f pod-with-res-limits.yml
```

## References

- [Resource Management for Pods & Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
