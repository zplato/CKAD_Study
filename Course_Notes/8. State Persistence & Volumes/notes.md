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

---

## Stateful Sets 

What is a Stateful Set? 
Similar to deployments for replicasets - scale up, scale down, perform rolling updates and rollbacks. 
Pods are created in a sequential order (unlike replicasets) - to help ensure Master Pod is deployed first and then slaves
Stateful Sets assign a number to each pod, starting at 0 and incrementing ++1 for each pod. 
Names assigned are not random. Naming starts with: `<stateful-set-name>-0` and increments ++1 for each pod in the set. Master pod will always be 0. 


Why Stateful Sets ? 
Typically used for things like a Database to maintain state of the database master and slave pods. 
Also, Deployment order matters, where Master -> Slave 1 -> Slave 2. etc
Need to differentiate These pods from Master to Slave. Need a way to identify the Master, with a static hostname. 

To create a stateful set, its created very similar to a Deployment, except you change the kind, to `Stateful Set`

```yaml
apiVersion: v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    app: mysql

spec:
  template:
    metadata:
      labels:
        app: mysql    # Pod name that matches the stateful set name 
    spec:
      containers:
        - name: mysql
          image: mysql
      replicas: 3
      selector:
        matchLabels: 
          app: mysql          # Links the Pod to the Stateful Set 

      serviceName: mysql-h    # headless service is required for a stateful set which is a stable, unique network identifier
      podManagementPolicy: Parallel # This overrides the default to bring the pods up in parallel
```
---

## Headless Service

In Master/Slave Topology:
* reads - can be processed by master or slave
* writes - must be processed by master 

A headless service, is created like a normal service, but it does not have an IP of its own, and it does not perform any load balancing
All it does is create a DNS Record in the form of: podname.headless-servicename.namespace.svc.cluster-domain

So for example to point to the master node in the mysql stateful-set: `mysql-0.mysql-h.default.svc.cluster.local`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
    - port: 3306
      selector:
        app: mysql
  ClusterIP: None   # This is what makes it a headless service
```

Then under spec, for a StatefulSet:
```yaml
spec:
  serviceName: mysql-h # Must specify the service name explicitly 
  containers:
    <container config>
```

### Storage in Stateful Sets 
If you were to add volumes, to the spec section and specify the PVC, then all pods in the stateful set would have access to that volume. 

However, if you want each instance to have its own storage, for example:
If you had a distributed database, where each instance of the SS had its own database with replicated data. 
Each Pod then needs a PVC for itself, and each PVC is bound to a PV. So a PV->PVC would need to be provisioned for each pod in the SS. 
This can be done with a **Volume Claim Template**. 

Within the spec section of the SS Define like so:

```yaml
spec: # this is the top level SS spec
  volumeClaimTemplates:
  - metadata:
      name: data-volume
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: google-storage # Provisioned through a Storage Class 
      resources: 
        requests:
          storage: 500Mi
```

If a Pod were to crash and get redeployed, then it will bind back to the same PVC via how the controller handles a stateful set
