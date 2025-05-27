# CKAD Cheat Sheet

A concise reference for the Certified Kubernetes Application Developer (CKAD) exam. Focused on high-frequency concepts and kubectl fluency.

---

## 1. Core Pod & Deployment

### Create a Pod

```bash
kubectl run nginx --image=nginx
```

### Create a Deployment

```bash
kubectl create deployment nginx --image=nginx --replicas=2
```

### Scale a Deployment

```bash
kubectl scale deployment nginx --replicas=3
```

### Edit Resources

```bash
kubectl edit deployment nginx
```

### View YAML of a resource

```bash
kubectl get deployment nginx -o yaml
```

---

## 2. ConfigMaps & Secrets

### Create from literal

```bash
kubectl create configmap app-config --from-literal=ENV=prod
kubectl create secret generic db-secret --from-literal=PASSWORD=secret
```

### Mount into Pod (example pod spec)

```yaml
env:
- name: ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: ENV
```

---

## 3. Jobs & CronJobs

### Job Example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "hello"]
      restartPolicy: Never
```

### CronJob Shortcut

```bash
kubectl create cronjob hello --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello'
```

---

## 4. Services & Networking

### Expose a Deployment

```bash
kubectl expose deployment redis --port=6379 --target-port=6379 --name=redis-service
```

### Port Forward

```bash
kubectl port-forward pod/mypod 8080:80
```

---

## 5. Volumes & PVCs

### Pod with PVC

```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: mypvc
volumeMounts:
- mountPath: /data
  name: data
```
---

## 6. Security Contexts

### Add Capabilities and Run as Root

```yaml
securityContext:
  runAsUser: 0 # root 
  capabilities:
    add: ["SYS_TIME"]
```

---

## 7. Probes

### Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## 8. Namespaces

### Create & Use

```bash
kubectl create namespace dev
kubectl config set-context --current --namespace=dev
```

---

## 9. Debugging

```bash
kubectl logs podname
kubectl describe pod podname
kubectl exec -it podname -- sh
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## 10. Imperative YAML Generation

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deploy.yaml
```

---

## Bonus: Handy Shortcuts

```bash
k get all
k get po -A
k delete pod mypod
k explain deployment.spec.template.spec.containers
alias kn="kubectl config set-context --current --namespace "
```

---

> Study Tip: Use `kubectl explain`, tab-complete aggressively, and practice under time pressure with tools like [Killer.sh](https://killer.sh)
