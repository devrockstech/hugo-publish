---
title: Kubernetes Data Persistance
weight: 4
---
# Storage

Persistent Volumes (PV), Persistent Volume Claims (PVC), and Storage Classes (SC) are the cornerstones of Kubernetes storage management. In this document, we look at these fundamental components that underpin Kubernetes storage.

![Storage](https://devrockstech.github.io/hugo-publish/images/storage.jpg)

Here’s why we use each of these components:

## Persistent Volumes (PV)
In Kubernetes Persistent Storage a PersistentVolume (PV) is a piece of storage within the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. PV is an abstraction for the physical storage device (such as NFS or iSCSI communication) that you have attached to the cluster. The main feature of a PV is that it has an independent life cycle which is managed by Kubernetes, and it continues to live when the pods accessing it gets deleted.

PVs define access modes (e.g., ReadWriteOnce, ReadOnlyMany, ReadWriteMany) and security settings for how pods can access the storage, ensuring data integrity and security.

```yaml
## Example of PV of size 3Gi
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-volume
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/app/data"
```

## Persistent Volume Claims (PVC)
A PersistentVolumeClaim (PVC) is a request for storage by a user. The claim can include specific storage parameters required by the application. For example, an amount of storage, or a specific type of access (RWO – ReadWriteOnce, ROX – ReadOnlyMany, RWX – ReadWriteMany, etc.).

`ReadWriteOnce` — the volume can be mounted as read-write by a single node

`ReadOnlyMany` — the volume can be mounted read-only by many nodes

`ReadWriteMany` — the volume can be mounted as read-write by many nodes

```yaml
## Example of PVC that claims a PV of size 3Gi
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
   resources:
      requests:
        storage: 3Gi
```

## Storage Classes
Storage Class allows the provision of Kubernetes persistent storage dynamically. With a storage class, administrators need not create a persistent volume separately before claiming it. Administrators can define several StorageClasses that give users multiple options for performance. For example, one can be on a fast SSD drive, it also supports Cloud Storage Services like AWS EBS, AzureDisk, GCEPersistentDisk, etc.

```yaml
## Example of storage class that uses local disk for volumes
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### Dynamic Provisioning
The dynamic way of storage provisioning involves creating PVs automatically, using StorageClass instead of manual provisioning of PersistentVolumes. Kubernetes will dynamically provision a volume specifically for this storage request, with the help of a StorageClass

![Storage](https://devrockstech.github.io/hugo-publish/images/dynamic-prov.png)

### Static Provisioning
The static way of provisioning a storage volume is when PVs are created before PVCs by an Administrator and exist in the Kubernetes API, waiting to be claimed by a user’s storage request, using PVC.

![Storage](https://devrockstech.github.io/hugo-publish/images/static-prov.png)

## Resizing a PVC
In cases where all storage is consumed by application and more storage is needed, PVCs can be resized according to the needs.
The PVC will enlarge the underlying PV and reattaches to the application pod. All you need to do is to adjust the request requirements on the PVC manifest.

## Example of attaching storage to an application and then Resizing it

Create a storageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azuredisk-standard-hdd-lrs
provisioner: disk.csi.azure.com
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
parameters:
  skuName: Standard_LRS
```

Create a PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-azuredisk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: azuredisk-standard-hdd-lrs
```

Create a Pod that mounts the PVC
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-azuredisk
spec:
  nodeSelector:
    kubernetes.io/os: linux
  containers:
    - image: mcr.microsoft.com/oss/nginx/nginx:1.17.3-alpine
      name: nginx-azuredisk
      command:
        - "/bin/sh"
        - "-c"
        - while true; do echo $(date) >> /mnt/azuredisk/outfile; sleep 1; done
      volumeMounts:
        - name: azuredisk01
          mountPath: "/mnt/azuredisk"
          readOnly: false
  volumes:
    - name: azuredisk01
      persistentVolumeClaim:
        claimName: pvc-azuredisk
```

Resize the PVC form 10Gi to 15Gi
```yaml
kubectl patch pvc pvc-azuredisk --type merge --patch '{"spec": {"resources": {"requests": {"storage": "15Gi"}}}}'
```