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
There is a 1 to 1 relationship between claims and volumes 

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

