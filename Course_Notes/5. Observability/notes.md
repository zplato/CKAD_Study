# Section 5. Observability Notes 

---
## Pod Status and Conditions 
**Pod State**s - Describe the current state of the pod high lvl summary, e.g., Pending, Running, Stopped
**Valid States** - Pending, ContainerCreating, Running, etc.

**Pod Conditions** - Gives additional descriptions about the current status of the pod 
**Valid Conditions** - PodScheduled, Initialized, ContainersReady, Ready

### Readiness Probes 
`Ready` condition states that the application in the pod is up and running and ready to accept traffic
Kubernetes by default assumes that as soon as the container is created, it is ready to serve user traffic and sets the value of the `Ready` condition to `true`
To more efficiently determine if an applicaiton is `Ready`, a `Readiness Probe` is used. 

The Readiness Probe can be configured to listen to various sources, such as an endpoint, TCP Socket, or even run a command/script that exits when the application is ready.
To set a readiness probe in a container, under spec->containers->container, you can configure:

```yaml
readinessProbe:  # http 
  httpGet:
    path: api/ready
    port: 8080
    
  initialDelaySeconds: 10 seconds # wait 10 seconds after container is up before checking 
  periodSeconds: 5                # wait 5 seconds between checking
  failureThreshold: 8             # try 8 times, default is 3 

readinessProbe:   # tcp
  tcpSocket: 
    port: 3306    # Database port for example 

readinessProbe: 
  exec:
    command:
      - cat
      - /app/is_ready
```

### Liveness Probe 
A `liveness probe` can be configured to periodically test whether the application in the container is actually healthy. 
If the test fails, the container gets destroyed and recreated. As a developer, we get to define what is considered alive (live) and healthy.
Valid Liveness Probes - http endpoint, tcp socket, exec command

Can be configured the exact same way as above, except replace `readinessProbe` with `livenessProbe`

## Container Logging
`k logs -f <pod-name>` - to stream logs live 
These logs are specific to the pod.

If there are multiple containers in the pod, then you have to pass the name of the container you want to see
For multi-container pod - `k logs -f <pod-name> <container-name>`

## Monitor and Debug Applications 
Monitor and Measure node level metrics, pod level metrics, services, memory, disk i/o and more
Lots of Monitoring Tools available - Prometheus, Elastic Stack, DataDog, Dynatrace

There is a builtlin `metrics-server` per kubernetes cluster. The metrics server is an in-memory metric solution. 
Cannot see historical data (limited). Kubernetes runs an agent on each node, known as the `kubelet`, which is responsible 
for receiving instructions from the K8 API master server and running pods on the nodes.
cAdvisor - is a subcomponent of a kubelet which is responsible for retrieving metrics from the pods and aggregating them up to the metrics server

To enable the metrics-server, use:
* minikube - `minikube addons enable metrics-server`
* All other K8s deployments - `git clone https://github.com/kubernetes-incubator/metrics-serve` and deploy `k create -f deploy/1.8+/`

Viewing Metrics:
* Node level - `kubectl top node`
* Pod Level - `kubectl top pod` 

