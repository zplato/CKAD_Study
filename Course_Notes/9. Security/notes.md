# 9. Security 
Focusing on Kubernetes based security 
`kube-api` server is important to protect as it gives API access to interface with kubernetes.
Need to answer two questions:
* Who can access the server?
  * Authentication Mechanisms 
    * Static Files that contain
      * User ID's and Passwords
      * User ID's and Tokens
      * Certificates 
      * External Authentication Providers like LDAP
      * For Machines - Service Accounts 
* What can they do?
  * Role Based Access Controls (RBAC)
  * Attribute Based Access Controls (ABAC)
  * Node Authorization 
  * Webhook Mode

## Authentication 
THere are 4 general groups of users: Admins, Developers, Application End Users and Bots 
Kubernetes does not manage user accounts natively. You cannot create users or view the lists of users (other than serviceaccounts)
All user access is managed through the api-server. The api-server authenticates the request. 
THe kube-apiserver has different authentication methods:
* static password file
  * Contains a list of Passwords, Username and UserID. Example file: `user-details.csv`
* static token file - same as static password file but except a password its a token 
* certificates
* 3rd party certification protocols (LDAP for example)

## Kubeconfig 
The `kubeconfig` file contains authentication details for you to authenticate with the kube-apiserver.
The details include: server, client-key, client-certificate, and certificate-authority 

By Default, the kubectl tool looks for a file under $HOME/.kube/config 

Config file has 3 sections:
* Clusters
  * Different Clusters you have access to (Prod, Test, Dev)
* Contexts
  * Contexts define which user account will access which cluster 
* Users 
  * User accounts defined that you have access to (existing users) 

```yaml
apiVersion: v1
kind: Config

current-context: dev-user@google # user@cluster 

clusters:
  - name: my-kube-playground
    cluster:
      certificate-authority: ca.crt 
      server: https://my-kube-playground:6443
contexts:
  - name: my-kube-admin@my-kube-playground
    context:
      cluster: my-kube-playground
      user: my-kube-admin 
users: 
  - name: my-kube-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
```

Userful commands:
* View Config - `kubectl config view`
* Change Curr Context - `k config use-context <user>@<cluster>`

## API Groups
You can access the kube-master directly from the url: https://kube-master:6443 and pass a `/version` to see a json of the version.
Additionally, you interface with the api through `/api` and to get the list of pods in json you would use `/api/v1/pods`
There are other groups too:
* /metrics
* /healthz
* /apis - named apis. It has groups under it for apps and extensions. 
* /logs 

/api is the core group where all fundamental api endpoints are, such as nodes, pods, services, etc 

Under apis/apps/v1/deployments, where deployments is a resource, there is a list of actions you can invoke through the api on that resource
such as: list, get, create, delete, update, watch - these are known as verbs 

## Authorization 
Authorization defines "What can they do?" after they gain access.
We want to give the restrict access to the minimum required authorization to those who need it.

There are 6 general authorization methods:
* Node
  * Kubelet reports to the API server with information about the node, such as its status. 
  * These requests for kubernetes are handled by a special authorizer known as the node authorizer.
* Attribute Based (ABAC)
  * Where you associate a user or a group of users with specific access 
  * Is defined in JSON Format and its passed into the api server 
* Role Based (RBAC)
  * Instead of associated users or groups of users, we define a role with a set of permissions, and we associate the right people to that role 
  * Going forward, whenever a change needs to be made then we modify the role, and it reflects on all developers immediately 
* Webhook - Utilized for 3rd party tools that handle permissions, such as "Open Policy Agent"
* AlwaysAllow - Allow all requests (default) 
* AlwaysDeny - Deny all requests 

The mode is set using the --authorization-mode="" option on the kube-apiserver 
You can have multiple modes at the same time. The order specified is the order that the request is authenticated in.

## Role Based Access Control (RBAC)
This is the most common way to manage Authorization in K8s. To use RBAC, we create a role definition file and assign the appropriate authorizations rules.
Rules have 3 sections:
* apiGroups - # For core group, we leave the apiGroups blank, else we can specify which specific group we are giving permissions to 
* resources - K8s resources, such as pods, ConfigMaps, StatefulSets, etc 
* verbs - What actions can they take on the resource. 

```yaml
# developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["create"]
```

To link the user to the role, we create another object called RoleBinding

```yaml
#devuser-developer-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:                                 # User Details
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io 
```
**Useful Commands**: 
* To create the role - `k create -f developer-role.yaml`
* To view roles - `k get roles`
* To view rolebindings - `k get rolebindings`
* To describe the role/rolebindings - `k describe role/rolebindings <name>`
* To check your own auth - `k auth can-i <do something>` for example, `k auth can-i create deployments`