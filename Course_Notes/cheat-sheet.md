# Cheat Sheet for CKAD Exam covering Core Concepts 

## Pods
A pod is the smallest deployable unit in kubernetes. 
A pod is a collection of one or more containers tightly coupled and share resources, like networking and storage.

A basic pod definition is shown below:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: simple-webapp
      image: my_image:5000
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```

### Resources 
Pods have resource specifications shown above as requests and limits.
* Request: The initial starting point for a pod to run, it requests the specified resources from the node
* Limit: The pod can continue to up its request as required automatically via the demand of workload on the pod, up to the specified limits.  

Default behavior - if neither requests or limits is set, then it can consume all resources available on the node

#### Limit Ranges and Resource Quotas 
You can set limit-ranges at the namespace level for pods, for example see `limit-range-cpu.yaml`:
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 500m 
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: "100m"
      type: Container 
```
Can utilize the same for memory, `limit-range-memory.yaml`. 

A resource quota sets hard limits for CPU and Memory at the namespace level 
See for example `resource-quota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi 
```
This sets the total requested CPU in the current namespace to 4 and memory to 4Gi. 


#### FAQ on Resources 
* What does `cpu: 2` actually mean? An abstraction of CPU which represents a vCPU
* What if a Pod has no limits set? It can consume as many resources on the node as it demands. Specifying a limit remedies this problem
* What happens if a pod tries to exceed CPU limit? The system throttles CPU so it cannot go beyond specified limit
* What happens if a pod tries to exceed Memory Limit? The pod will exceed and terminate via OOM error (typically 137)


## Deployments, Replication Controller and ReplicaSets 

## Namespaces & Cluster Contexts

*Useful Commands*
* View current Namespace - `kubectl config view`
  * Add `--minify` to only get information from the current context (cluster)
* List all namespaces - `kubectl get namespaces`
* Switch namespaces - `kubectl config set-context --current --namespace=<namespace>`
* List available clusters - `kubectl config get-contexts`
* Switch Clusters - `kubectl config use-context <context-name>`

## Imperative Commands
**What is imperative commands?** - THese are commands used to test and generate resources on the fly without needing a declarative file 

Useful Commands and Args:
* `--dry-run=client` This will not create the resource but instead tell if you if the resource can be created and if your command is correct
* `-o yaml` - Outputs the resource definition in yaml format on screen. Follow up with `>` to redirect the output to a file 

*NOTE* - Utilize the above two commands in combination to quickly generate a resource definition file. For example: `kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml`

Additional Examples:
* `kubectl create deployment --image=nginx nginx` - to create a deployment named nginx
  * Can add replicas on e.g., `--replicas=4`
* Save output to a file: `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`
* `kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml` - create a service named redis-service of type ClusterIP to expose pod redis on port 6379

## Docker
Docker containers share the host kernal. They do not include or emulate a kernal. They run in user space (your file system) on top of the hosts kernal.
Benefits: Lightweight, fast startup and smaller resource footprint. 
Docker Virtualizes: 
* **Namespaces** - isolates processes, networks, mount points users, etc
* **Cgroups** - controls resource limits like CPU, memory
* **Union file systems** - enables layering of images 

All containers on a host must be compatible with the host kernel 
Most Linux containers can run on most linux kernals as long as the kernal is new enough to support sys calls and features the containerized app expects

### Commands in Dockerfiles and Containers in K8s
#### Dockerfiles
`ENTRYPOINT:` -  defines the startup command of the container (PID 1). Once this command is finished then the container gracefully stops.
`CMD;` - Additional CLI args can be passed to entrypoint

#### Containers in K8s
`command:` - defined in the `containers` section, overrides the entrypoint of the dockerfile
`args:` - overrides the `CMD:` of the dockerfile or allows additional arguments to be passed

*NOTE* - if you provide only `args:` in kubernetes and the image has no entrypoint, then the container will fail to start

## Config Maps & Secrets 
Config maps are centrally managed environmental configurations in key, value pairs shared amongst resources. 
Config maps are created and then later injected at deployment time. 

**Imperative Command** - `kubectl create configmap <config-name> --from-literal=<key>=<value>`

**Declarative Command**
Utilizing a configmap.yaml file with `kubectl apply -f <filename>`
Four necessary fields: 
* apiVersion - typical 
* kind: ConfigMap  
* metadata: - typical 
* data: - key value data pairs, see `3. Configuration/config-map.yaml`

**Useful Commands and Args**
`kubectl get configmap` - to view active config maps 
`kubectl describe configmap <configmap-name>` - describe configmap names 

### Secrets 
Similar to configmaps, first we create them and then inject them into the pod 
Secrets are used to store sensitive data with encryption 

**Imperative Secret Generation** - `kubectl create secret generic <secret-name> --from-literal=<key>=<value>`

**Declarative Secret Generation** - `kubectl create secret generic <secret-name> --from-file=<path-to-file>`

**Useful Commands and Args**
`kubectl get secerts` - view active secrets 
`kubectl describe secrets <secret-name>` - describe the secret but won't show the values 
`kubectl get secrets <secret-name> -o yaml` - to view the encoded values 

**Base 64 Encoding** 
Secrets MUST be base64 encoded in kubernetes files 
encode: `echo "my-secret" | base64 --encode`
decode: `echo "my-secret" | base64 --decode` 

#### Referencing a Secret
Inject entire secrets file `app-secret`
```
envFrom:
  - secretRef:
      name: app-secret 
```

OR Inject a single environment variable 
```
env:
  - name: DB_Password
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Password
```

OR Inject secrets as files in a volume 
```
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret 
```

**Best Practices with Secrets**
* Secrets are not encrypted, only encoded - so do not check in secret objects into SCM
* Secrets are not encrypted in etc.d 
  * Enable Encryption at rest to mitigate
* Anyone able to create pods/deployments in the same namespace can access the secrets as well
  * Configure Least-privilege access to secrets - RBAC
* Consider third-party secrets store providers

To enable encryption at rest, you create an EncryptionConfiguration file, reference - https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

## Security Contexts
Specifies things such as user and capabilities to add to the pods
Can be added at the pod level or container level. Example:
```commandline
containers:
    - name: ubuntu
      image: ubuntu
      securityContext:
        runAsUser:1000
        capabilities:
            add: ["MAC_ADMIN"]  
```