# Horizontal Pod Autoscaler (HPA)

## What is HPA?
The Horizontal Pod Autoscaler automatically updates a workload resource (such as a Deployment or StatefulSet), with the aim of automatically scaling the workload to match demand. In simple terms, it increases or decreases the number of Pod replicas based on observed metrics like CPU utilization or custom metrics.

## Why do we use it and what problem does it solve?
1. **Handling Variable Traffic**: Without HPA, you have to manually guess how many pods you need. If traffic spikes, your app might crash. If traffic is low, you are wasting resources and money. HPA solves this by dynamically adjusting pods based on real-time load.
2. **Cost Optimization**: By scaling down during off-peak hours, you free up cluster resources and save infrastructure costs.
3. **High Availability**: Automatically adds more instances of your application when the current ones are struggling to keep up with the demand, ensuring your application remains responsive.

## Prerequisites
For HPA to work, your cluster **must** have a **Metrics Server** installed. The Metrics Server collects resource metrics from the Kubelets and exposes them to the Kubernetes API server for the HPA to read.
*(Minikube users can enable it with: `minikube addons enable metrics-server`)*

**Important:** HPA will NOT work if your pods do not have `requests` defined for the resources (like CPU) you are trying to scale on. HPA calculates the utilization percentage based on the requested resource.

## Syntax Example
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment  # The deployment to scale
  minReplicas: 1             # Minimum number of pods to maintain
  maxReplicas: 10            # Maximum number of pods allowed during high load
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50 # Scale up when average CPU utilization across all pods exceeds 50% of the requested CPU
```

## How to Test HPA (Visualizing Scaling)

I've provided three YAML files in this directory to test HPA:

1. **Deploy the App**: 
   `kubectl apply -f 1-app-deployment.yml`
   *(This deploys a sample PHP-Apache app that is designed to consume CPU when requested, and a Service to expose it).*
   
2. **Deploy the HPA**:
   `kubectl apply -f 2-hpa.yml`
   *(This configures HPA to scale the deployment if CPU goes over 50%).*
   
3. **Watch the HPA and Pods**:
   Open a new terminal and run this command to watch HPA continuously:
   `kubectl get hpa -w`
   
   Open another terminal to watch the pods:
   `kubectl get pods -w`

4. **Generate Load**:
   Now, apply the load generator pod which will spam the PHP app with requests:
   `kubectl apply -f 3-load-generator.yml`
   
5. **Observe**:
   Look at your `watch` terminals. Within a minute or two, you will see the CPU utilization shoot up on the `hpa` output (e.g., `250%/50%`). 
   Shortly after, Kubernetes will automatically create new pods to handle the load.

6. **Scale Down**:
   Delete the load generator:
   `kubectl delete -f 3-load-generator.yml`
   Wait a few minutes (default is 5 minutes for scale-down stabilization), and you will see the HPA scale the replicas back down to 1.
