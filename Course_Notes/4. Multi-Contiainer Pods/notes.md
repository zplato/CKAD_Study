# Multi-container Pods
Pods can have multiple containers in each of them. 

What do they share within the same pod?
* Lifecycle - created and destroyed together
* Network - each container can refer to each other as localhost 
* Storage - Access to the same storage volumes 

Can add multiple containers to the pod-definition.yaml under the `containers` section.

## Design Patterns

1. Sidecar - Applying a logging agent to collect logs and forward them to a central log server
2. Adapter - Processes logs before sending to a central server (unifies formatting etc.)
3. Ambassador - Utilized as means to provide a single endpoint of communication for multiple environments (dev, test and prod)

### Init Containers 
An Init container is run when the pod starts up and a process is required to load configuration data or wait for another service to be up prior to starting the application container
An Init container is configured ina pod like all other containers, except that it is specified inside a `initContainers` section.
When a Pod is created, the initContainer is run and the process in the initContianer must run to completion before the real container hosting the application will come up.
Its possible to configure multiple init containers in the same pod for various purposes. In the caes of multiple, they are processed in sequential order as defined in the yaml. 

```yaml
spec:
  containers:
    - name: my-container
      <more container stuff...>
  initContainers:
    - name: my-init-container
      image: nginx
      command: ["sh", "-c", ...]
```
