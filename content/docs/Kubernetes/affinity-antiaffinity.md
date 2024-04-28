---
title: Affinity & Antiaffinity
weight: 5
---
# Node Affinity and AntiAffinity

There are two type of affinity rules in kubernetes
1. Node Affinity
2. Pod Affinity / AntiAffinity

## Node Affinity
Kubernetes node affinity is a feature that enables administrators to match pods according to the labels on nodes. It is similar to nodeSelector but provides more granular control over the selection process. 

Node affinity enables a conditional approach with logical operators in the matching process, while nodeSelector is limited to looking for exact label key-value pair matches. Node affinity is specified in the PodSpec using the `nodeAffinity` field in the `affinity` section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:  # Check this rule when scheduling new pod. Ignore Existing
        nodeSelectorTerms:   # Schedule this pod on a node that has the label kubernetes.io/os=linux
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```


## Pod Affinity / AntiAffinity
Pod affinity provides administrators with a way to configure the scheduling process using labels configured on the existing pods within a node.

A pod with a label key-value pair matching a condition can be scheduled on a node containing the matching labels.

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:    # Schedule this pod on a node existing in the same zone and contains a pod having label `security=S1`
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:  # Try not to schedule this pod on a node existing in the same zone and contains a pod having label `security=S2`
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```

### Required vs. preferred rules

Both node affinity and pod affinity can set required and preferred rules that act as hard and soft rules. You can configure these rules with the following fields in a pod manifest:

1. requiredDuringSchedulingIgnoredDuringExecution (Hard limit)
2. preferredDuringSchedulingIgnoredDuringExecution (Soft limit)

These rules are valid for both node and pod affinity and can be configured using the respective affinity fields. Below are example rules to demonstrate this configuration method.


### Usecases
1. Two pods must be on the same node. (use podAffinity)
2. Two pods must not be on the same node to ensure high availability (use podAntiAffinity)
3. A Pod must be scheduled on a Node that has a specific label (use nodeAffinity)