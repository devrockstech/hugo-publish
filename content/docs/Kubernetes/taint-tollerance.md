---
title: Kubernetes Taints & Tollerance
weight: 6
---
# Taints and Tolerations
In Kubernetes, taints and tolerations are used to control the scheduling of pods onto nodes in a cluster. They help to ensure that only certain pods are placed on specific nodes, based on specific conditions and requirements. Let's explore each concept with a proper explanation and an example:

## Taints
Taints are applied to nodes in a Kubernetes cluster to repel pods from being scheduled on those nodes, except for pods that explicitly tolerate the taint. Taints are typically used to mark nodes with specific attributes or limitations, such as reserving certain nodes for particular workloads or preventing pods from being placed on nodes with specific hardware characteristics.

Each taint consists of three parts:
**Key:**: The key represents the name of the taint and is used to uniquely identify it.
**Value:**: An optional value associated with the taint key.
**Effect:** The effect determines how the taint affects pod scheduling.


There are three possible values for the effect:
**NoSchedule:** Prevents new pods from being scheduled on the tainted node.
**PreferNoSchedule:** Similar to NoSchedule, but the scheduler tries to avoid placing new pods on the tainted node unless necessary.
**NoExecute:** Evicts existing pods that do not tolerate the taint.


### Example of a taint:

Let's say you have a node with a specific GPU and you want to reserve it for running only GPU-intensive workloads. You can taint the node as follows:

```bash
kubectl taint nodes <node-name> gpu=true:NoSchedule
```

## Tolerations:
Tolerations are applied to pods and indicate that the pod can be scheduled on nodes with specific taints. A pod with toleration will only be scheduled on nodes that have a matching taint. By setting tolerations, you can make sure that certain pods are placed on nodes with specific attributes or restrictions, even if those nodes are tainted.

### Example of toleration:

Let's consider the previous example of the node with the GPU taint. To allow a pod to be scheduled on the GPU node, the pod's YAML definition would include toleration like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
  - name: my-app
    image: my-app-image
  tolerations:
  - key: "gpu"
    operator: "Exists"
    effect: "NoSchedule"
```

In this example, the pod has a toleration for the "gpu" taint with "NoSchedule" effect. This means the pod can be scheduled on nodes with the taint "gpu=true:NoSchedule," allowing it to utilize the GPU resources.