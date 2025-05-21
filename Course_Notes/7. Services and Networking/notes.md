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
Ingress helps users access your application via a single externally accessible URL that we can configure to route traffic to different services and 
at the same time can implement security on your cluster with ssl. Think of ingress as a layer 7 load balancer builtin to the kubernetes cluster configured using native k8s primitives

Ingress is configured with two basic building blocks:
* **Ingress Controller** - A controller is synonymous to a deployed reverse proxy (nginx, istio, gcp)
  * Not deployed by default 
  * Defined as a deployment file with some special configs 
  * Common way to Fetch and Deploy an Ingress controller `helm install ingress-nginx ingress-nginx/ingress-nginx` and `k apply -f <manifest file>`
  * To deploy an ingress controller, you need a Deployment file (nginx), a service, a configmap and a service account for authorization
* **Ingress Resources** - is the configuration (rules) that the controller acts upon. Similar YAML files. 
  * Example Rules - Route traffic to X and Y applications based on the URL (endpoint or domain name)
  * Within each rule we can determine where to route the traffic via URL
* **Ingress-service** (either nodePort or load balancer) - one time configuration to expose the ingress to the world

See example Ingress Resource 
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
    - http:
        paths:
          - path: /wear
            backend:
              serviceName: wear-service
              servicePort: 80
          - path: /watch
            backend:
              service:
                name: wear-service
                port:
                  number: 80
                

# To split by Hostname utilize 2 rules
    rules:
      - host: x.com
        http:
          paths: # rest of above
      - host: y.com
        http:
          paths: # rest of above 
```

**Imperative Cmd to create ingress resource**
`kubectl create ingress <ingress-name> --rule="host/path=service:port"`
Example - kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
---

## Network Policies 
Each Node has an IP Address, as well as each pod and service. 
Each Pod should be able to communicate with eachother without having to configure any additional settings, such as routes
All Pods are in a virtual private network that spans across the nodes in a cluster. 
Kubernetes is by default configured to "all allow" rule, which allows all traffic between pods

A **Network Policy** is another object in the k8s namespace that gets linked to one or more pods, which defines policies that match specified rules.
We utilize Labels and Selectors to link the network policy to a Pod. 
Two policy types:
* Ingress - Allow Traffic into the Pod  
  * from:
    * podSelector - Allows ingress traffic from Pod 
    * namespaceSelector - Filters on namespace to allow traffic from a specific namespace 
    * ipBlock - Filters on IP to allow traffic from a specific IP
* Egress -  Allow Traffic from the Pod 
  * to:
    * podSelector - Allows egress traffic from my pod to this pod  
    * namespaceSelector - Filters on namespace to allow traffic from a specific namespace 
    * ipBlock - Filters on IP to allow traffic from my pod to a specific IP 

A single NetworkPolicy can have both Ingress or Egress Types

Whatever is specified in the NetworkPolicy applied to the pod will be allowed, all else won't be.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db      # Label specified on the db pod - so that it applies this policy to that pod
  policyTypes:
  - Ingress       # Only Ingress Traffic is Isolated and Egress Traffic is unaffected
  - Egress
  ingress: 
  - from: 
    - podSelector: 
        matchLabels:
          name: api-pod  # Allow traffic from pods that have label api-pod
      namespaceSelector:
        matchLabels:
          name: prod 
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306         # Traffic allowed on Port 3306 
  egress:
    - to:
        - ipBlock: 
            cidr: 192.168.5.10/32
      ports:
        - protocol: TCP
          port: 80
```

---
## Networking Fundamentals

Common Networking Fundamentals: 
* /etc/hosts file - DNS Override file on unix/linux systems 
* CNAME - Is a record in DNS which maps (sets an alias) of a domain name to another domain name
* Route53 - a highly available and scalable DNS Service offered by Amazon Web Services
* CoreDNS is the default DNS service in Kubernetes
* Proxy Server (Client Side) - Digital Intermediary that acts as a gateway between a users computer and the internet routing traffic and potentially hiding the users IP Address
* Reverse Proxy (Server Side) - Hides the Server from the client. Does additional Load balancing, SSL Termination and caching. Example (Nginx in front of backend services)
* L4 Load Balancer - OSI model layer 4 (transport layer), handling TCP and UDP Traffic without inspecting application level content 
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

