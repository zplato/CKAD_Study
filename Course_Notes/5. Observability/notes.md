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



