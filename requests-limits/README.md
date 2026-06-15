# Kubernetes Requests and Limits

## Why do we use them?
In Kubernetes, **Requests** and **Limits** are used to manage and control the compute resources (like CPU and memory) allocated to containers within a Pod. 

* **Requests**: The guaranteed *minimum* amount of resources a container needs to run. The Kubernetes scheduler uses this to decide which worker node has enough available capacity to host the Pod.
* **Limits**: The *maximum* amount of resources a container is allowed to consume. This acts as a hard boundary.

## What problem do they solve?
1. **The "Noisy Neighbor" Problem**: Limits prevent a single runaway container (e.g., due to a bug or traffic spike) from consuming all the resources on a node and starving other critical applications.
2. **Predictable Scheduling**: Requests ensure that pods are only placed on nodes that actually have the capacity to support them, preventing node overloading right from the start.
3. **Cluster Stability**: By enforcing limits, Kubernetes can safely throttle CPU usage or terminate (OOMKill) containers that exceed their memory limits, protecting the overall stability of the host node.
4. **Efficient Resource Utilization**: It helps Kubernetes intelligently pack pods onto nodes (bin-packing), maximizing hardware usage without compromising performance.

## Syntax Example
Resource requests and limits are defined inside the `resources` block of a container's specification in the YAML file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: my-app-container
    image: nginx
    resources:
      requests:
        memory: "64Mi" # Guaranteed minimum memory
        cpu: "250m"    # Guaranteed minimum CPU (250 millicores or 0.25 CPU)
      limits:
        memory: "128Mi" # Hard maximum memory limit (container is killed if exceeded)
        cpu: "500m"    # Hard maximum CPU limit (container is throttled if exceeded)
```

### Note on Units:
* **CPU**: Measured in cores. `1` means one whole CPU core. `500m` means 500 millicores (half a core).
* **Memory**: Measured in bytes. Usually expressed with suffixes like `Mi` (Mebibytes) or `Gi` (Gibibytes).
