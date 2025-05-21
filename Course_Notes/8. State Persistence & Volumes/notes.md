# 8. State Persistence and Volumes 

## Volumes
When a container is created and destroyed, a the data along with the container is destroyed.
To persist data, a Volume is attached to the container. The volume exists outside the container 
Lots of Volume types supported by Kubernetes

```yaml
# In a pod, under spec: and inline with :containers
spec:
  containers:
    - name: alpine
      <container stuff>
      volumeMounts:
        - mountPath: /opt     # Volume mounts to /opt inside the container 
          name: data-volume
  
  volumes:
    - name: data-volume
      hostPath: 
        path: /data           # Host Directory 
        type: Directory 

    - name: mypvc             # how to mount a pvc to a pod 
      persistentVolumeClaims:
      claimName: myclaim
```

---

## Persistent Volumes 
A persistent volume is a cluster-wide pool of storage configured by an administrator to be used by users deploying applications on a cluster
The users can then claim a piece of that storage using Persistent Volume Claims `PVCs`

```yaml
# pv-definition.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce     # Can also be ReadOnlyMany, or ReadWriteMany 
  capacity:
    storage: 1Gi
  
  hostPath:             # Can be replaced with the many options for storage supported in kubernetes (aws-ebs, nfs, etc)
    path: /tmp/data 
```

Once a PVC Binds to a PV, and then later gets removed. By default the storage is `Retained`, but has other options such as Delete or Recycle 
```persistentVolumeReclaimPolicy: Delete``` 

---

## Persistent Volume Claims (PVCs)
A PVC is how you make the storage available to a node. a PVC claims a portion of a PV.
Kubernetes binds the PVCs to the Volumes. Each PVC is bound to a single PV. 
Kubernetes tries to bind the PVC to any PV which matches the config of the PVC. Such as Capacity, Access Modes, Volume Modes, Storage Class, etc. 
If there are multiple matches to a claim, we can use labels and selectors to bind to a specific volume 
There is a 1 to 1 relationship between claims and volumes, kubernetes does not slice the volume or do partial bindings, it binds the whole volume. 
For example: 
* you create a PV that has 100Mi available
* you create a PVC that matches the PV with a request of 50Mi
* The scheduler finds the PV with 100Mi available that satisfies the requirements of the PVC and binds the entire thing to it, making 100Mi available even though 50Mi was requested

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

## Storage Classes 
**Static Provisioning Volumes** - The Disk/Storage type gets created first (EBS) and then the PersistentVolume gets created after. All Done Manually
**Dynamic Provisioning of Volumes** 
* Storage Classes allow you to define a provisioner which automatically provisions storage 
* Create a StorageClass Object, specify a name and use a cloud provisioner 

```yaml
# StorageClass Definition (sc-definition.yaml)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

For the PVC to use the storage class we define, we specify in pvc-definiton.yaml:
`spec.storageClassName: google-storage`
 This is how the PVC knows which storage class to use. 
This still creates a PV, but you don't have to manually create it. Kubernetes through the use of a Storage Class does it for you.

You can create different classes of storage with 'tiered' replication-type. Silver, Gold and Platinum for example. Which has None, or regional replication for the persistent volume

