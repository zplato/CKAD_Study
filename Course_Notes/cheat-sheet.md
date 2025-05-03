# Cheat Sheet for CKAD Exam covering Core Concepts 

## Pods

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