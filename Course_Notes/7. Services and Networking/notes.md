# 7. Services and Networking

## Services 
K8s services help us connect together applications. Services enable connectivity between groups of pods. 
helps communications between frontend and backend pods. Services enable loose coupling in the application. 

### IP Address Distributions
VM/EC2/Host Computer - Has its own IP Address (example 192.168.1.10)
Kubernetes Node - Has its own IP (192.168.1.2)
Internal Pod Network - IP (10.244.0.0)
Application Pod - 10.244.0.2

Cannot directly ping from Host Computer to Pod as they are in a separate network. 
Need something in the middle that maps and connects the Internal Pod Network to the Host Computer. 
This is where the K8s service comes into play. Its use case is to listen to a port on a node and forward requests to pods. 

**Service Types**:
* NodePort - Port Forwards messages from a port on the node to pods
  * Three Ports involved:
    * Target Port - Port on the pod (target)  
    * Service Port - Port on the actual service object (middleman) 
    * Node Port - Port on the node to access the service. Valid range (30000-32767)
* ClusterIP - Establishes Connectivity between Services by creating a virtual ID inside the cluster to enable communication between services inside the cluster
  * Groups pods together and provides a single interface to access pods in a group (think selector->label)
  * 
* LoadBalancer - Provisions a load balancer in supported cloud providers to distribute load across the servers.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort        # Cluster IP, NodePort, or LoadBalancer 
  ports:    
    - targetPort: 80    # This config is specific to a nodePort 
      port: 80
      nodePort: 30008
  selector:             # Selector with Labels that map to a specific pod, automatically load balances to all pods that match the label
    app: myapp
    type: frontend
    
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
  - targetPort: 80    # Port where the backend is exposed
    port: 80          # Port where the service is exposed
  selector:           # Selector and Labels - Links service to pods  
    app: myapp
    type: backend 
```


** Useful Service Commands** 
`k create -f service-definition.yaml`
`k get services`

---
## Ingress 
Ingress helps users access your application via an externally accessible URL that we can configure to route traffic to different services and 
at the same time can implement security on your cluster with ssl. 

Ingress is configured with two basic building blocks:
* Ingress Controller - A controller is synonymous to a deployed reverse proxy (nginx, istio, gcp)
  * Not deployed by default 
  * Defined as a deployment file with some special configs 
  * Common way to Fetch and Deploy an Ingress controller `helm install ingress-nginx ingress-nginx/ingress-nginx` and `k apply -f <manifest file>`
* Ingress Resources - is the configuration that the controller acts upon. Similar YAML files. 



---
## Networking Fundamentals

Common Networking Fundamentals: 
* /etc/hosts file - DNS Override file on unix/linux systems 
* CNAME - Is a record in DNS which maps (sets an alias) of a domain name to another domain name
* Route53 - a highly available and scalable DNS Service offered by Amazon Web Services
* CoreDNS is the default DNS service in Kubernetes
* Proxy Server (Client Side) - Digital Intermediary that acts as a gateway between a users computer and the internet routing traffic and potentially hiding the users IP Address
* Reverse Proxy (Server Side) - Hides the Server from the client. Does additional Load balancing, SSL Termination and caching. Example (Nginx infront of backend services)
* L4 Load Balancer - OSI model layer 4 (transport layer), handling TCP and UDP Traffic without inspecting applicaiton level content 
  * Works at the network level, checking things like source and destination ip, ports. 
  * Routes traffic via predefined rules / algorithm (round-robin, least-connections, etc)
  * Doesn't understand protocols such as HTTP, gRPC, or WebSocket, it just forwards packets 
* L7 Load Balancer - OSI model layer 7 (Application Layer)
  * Routing Logic - URL Path, Headers, Cookies, Hostname
  * Protocols - HTTP(s), WebSockets, gRPC
  * Content-based routing - can route to endpoints /api vs /admin
  * TLS Termination - Can decrypt and rencrypt https traffic
  * Performance - flexible and heavier than L4 Load balancer
  * Commonly L7 is preferred. 
* 

