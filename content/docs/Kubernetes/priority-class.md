---
title: Kubernetes Priority Class
weight: 6
---
# PriorityClasses

Priority indicates the importance of Pod relatives to other pods. When Pod priority is enabled, the scheduler orders pending Pods by their priority and a pending Pod is placed ahead of other pending Pods with lower priority in the scheduling queue. As a result, the higher priority Pod may be scheduled sooner than Pods with lower priority if its scheduling requirements are met.

Kubernetes already ships with two PriorityClasses: `system-cluster-critical` and `system-node-critical`. These are common classes and are used to ensure that critical components are always scheduled first.

Custom PriorityClasses cannot start with `system-` prefix.

value range of PriorityClass object is from `-2147483648` to `1000000000` inclusive

**The higher the value, the higher the priority**.

## PriorityClass Example

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
preemptionPolicy: Never  # Do not evict lower priority pods inorder to schedule pods using this class
globalDefault: false     # Do not evaluate existing pods. Only for new pods
description: "This priority class should be used for XYZ service pods only."
```

Here is an example of a pod using the above priority class
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```