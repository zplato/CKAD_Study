# Section 6. Pod Design 

---

## Labels and Selectors
What we know already:
* Labels - are a standard way to group things together (classify) and filter them based on that label
  * Labels are properties attached to each item
  * Labels are applied in the metadata->labels: field of a pod-definition.yaml in a key : value format
* Selectors - Help select labels to filter
  * To select pods via a label, utilize `k get pods --selector key=value`

Internally Kubernetes uses labels and selectors to create things such as a replicaset, or a deployment. 
Here is an example replicaset with the usage of selectors and labels:
```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
  name: my-replica-set
  labels:
    app: App1           # Label for the Replica Set itself
    function: front-end # another label for the replicaset 
  annotation:
    buildVersion: 1.34
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1     # The label of the selector which must match the label of the pods 
  template:
      metadata:
        labels:
          app: App1 # Label of the pod, which matches the selector label, This is how the connection is made from pod <-> replicaset
      spec:
        containers:
          - name: simple-webapp
            image: simple-webapp
```
---
## Annotations
Annotations are used to record other details for information purpose. 
Annotations are commonly used for buildversion, or other metadata like fields. See above for an annotation defintion

---

## Rollouts and Versioning 
Updates and rollbacks with a deployment. When you first create a deployment it triggers a rollout with an initial version (revision 1).
Everytime you deploy with a new version of the deployment, a new rollout is triggered and a new revision is created (revision 2). 
This helps us keep track of the changes made to our deployment and helps us rollback if necessary.

* Rollout Command - `k rollout status deployment/<my-deployment>` which will show you the current status of a rollout 
  * add `--record` arg to make `CHANGE-CAUSE` recorded reason appear on rollout history'
* Rollout History - `k rollout history deployment/<my-deployment>` which will show you the revision history of the deployment 
* Rollback to Prev Revision - `k rollout undo deployment/myapp-deployment` which will rollback to the prev. revision

There are 4 types of deployment strategies:
1. **Recreate Strategy (Destroy and Recreate)** - destroy all 5 running instances and deploy 5 new ones. 
   1. Causes downtime
2. **Rolling Update (default)** - take pods down one by one and replace them with an updated version. Creates no downtime
3. **Blue Green** - Old Version (Blue), New Version (Green)
   4. Deploy to an Alt (Prod2 Environment) and switch traffic from old to new once all tests are passed in new
   5. Not Specified in the deployment, but implemented in a different way 
   6. Strategy is best implemented with service mesh such as Istio 
   7. Without Istio, we can swap deployments with Selectors->Labels to implement a blue-green update strategy
      8. Labels are swapped on the service, which will update to point from the blue to green. 
         9. Single Service Swapped from app-blue.yaml (app-blue label) and app-green.yaml (app-green label)
4. **Canary** - Route a small percentage of traffic top the new deployment. Run tests and if everything looks good, then we upgrade the original deployment with a newer version of the upgrade. 
   5. Primary Deployment - Has its own service to route traffic with label set on the pods
   6. Second Deployment - Canary, has a common label between primary and and secondary. 
      7. To make the secondary deployment smaller, then create a smaller amount of pods on the secondary deployment
      8. Have limited amount of control over the amount of traffic to each pod
         9. Istio solves this problem and will allow you define traffic routing percentages 

In the case of a single pod with rolling update:
1. New pod is created 
2. Readiness Probe check on new pod
3. If readiness probe is good, then delete old pod and serve traffic on new pod

If you deploy with a bad deployment, then it will attempt to take down the first pod in the replicaset, Available=Desired=1. 

## Jobs in Kubernetes
A job is a workload meant to live for a short period of time. 
In Kubernetes, a job is a meant to run a set of pods for a period of time to perform a given task to completion. 
A Job can be defined as follows:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '*', '2']
      restartPolicy: Never 
  backoffLimit: 4
```



Useful Commands and Args:
* `k create job <my-job> --image=<my-image>` - can also add `--` and an executable command at the end 
* `k get jobs` - to view the job data 
* `k logs <pod-name>` - to see the pod data
* Under `spec`, you can add `completions: 3` to force the job to get at least 3 completions before its finished
* Only a RestartPolicy equal to `Never` or `OnFailure` is allowed.
* `backoffLimit` is defined on the job to specify how many failures can occur before the job is stopped. 
* `parallelism` determines how many of the jobs to run in parallel

---

## CronJobs in Kubernetes 
Schedule and run jobs periodically with a Cronjob.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:            
    name: reporting-cron-job
spec:               # Spec section for cronjob 
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:           # Spec copied from the job 
      completions: 3
      parallelism: 3
      template:
        spec:       # Spec of the pod 
          containers:
            - name: reporting-tool
              image: reporting-tool
          restartPolicy: Never 
      backoffLimit: 4
```
