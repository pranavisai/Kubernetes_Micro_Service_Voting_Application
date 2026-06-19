# Kubernetes Voting App - Quick Notes

## 1. Verify Kubernetes Cluster

Check current context:

```bash
kubectl config current-context
```

Check nodes:

```bash
kubectl get nodes
```

---

## 2. Create Resources

Create Pods, Services, or Deployments:

```bash
kubectl apply -f <file-name>.yaml
```

Examples:

```bash
kubectl apply -f voting-app-pod.yaml
kubectl apply -f voting-service.yaml
```

---

## 3. View Cluster Resources

View all pods:

```bash
kubectl get pods
```

View services:

```bash
kubectl get svc
```

View deployments:

```bash
kubectl get deployments
```

View everything:

```bash
kubectl get all
```

---

## 4. Debugging Services

Check service details:

```bash
kubectl describe service voting-service
```

Check service endpoints:

```bash
kubectl get endpoints voting-service
```

Example:

```text
10.244.0.21:80
10.244.0.27:80
10.244.0.28:80
```

This confirms the service is connected to pods.

---

## 5. Verify Pod Labels

Show pod labels:

```bash
kubectl get pod voting-app-pod --show-labels
```

Important:

Service selectors must match pod labels.

Example:

```yaml
labels:
  name: voting-app-pod
  app: demo-voting-app
```

```yaml
selector:
  name: voting-app-pod
  app: demo-voting-app
```

---

## 6. View Logs

Check application logs:

```bash
kubectl logs worker-app-pod
```

Check previous crashed container logs:

```bash
kubectl logs worker-app-pod --previous
```

Used to identify:

* DNS issues
* Database connection issues
* Authentication failures

---

## 7. Describe Pods

Detailed pod information:

```bash
kubectl describe pod worker-app-pod
```

Useful for:

* Environment variables
* Restart count
* Events
* CrashLoopBackOff debugging

---

## 8. Access Applications

### Port Forwarding

Voting App:

```bash
kubectl port-forward service/voting-service 8080:80
```

Result App:

```bash
kubectl port-forward service/result-service 8081:80
```

Access:

```text
http://localhost:8080
http://localhost:8081
```

---

## 9. Scale Deployments

Increase replicas:

```bash
kubectl scale deployment voting-app-deploy --replicas=3
```

Verify:

```bash
kubectl get pods
```

Result:

```text
voting-app-deploy-xxxxx
voting-app-deploy-yyyyy
voting-app-deploy-zzzzz
```

---

## 10. Common Issues We Fixed

### Service Not Finding Pods

Problem:

```text
ENDPOINTS <none>
```

Cause:

Service selector did not match pod labels.

Fix:

Ensure labels and selectors match exactly.

---

### Worker Could Not Find Database

Error:

```text
No such device or address
```

Cause:

Worker expected service name:

```text
db
```

But service was named:

```text
postgres
```

Fix:

```yaml
metadata:
  name: db
```

---

### PostgreSQL Authentication Error

Error:

```text
Authentication method not supported
```

Cause:

Using latest PostgreSQL image.

```yaml
image: postgres
```

Fix:

Use older version compatible with worker:

```yaml
image: postgres:9.4
```

Recreate pod afterwards.

---

## 11. Deployment Benefits

Deployments provide:

* Self-healing
* Scaling
* Rolling updates
* Replica management

Example:

Delete a pod:

```bash
kubectl delete pod voting-app-deploy-xxxxx
```

Deployment automatically creates a replacement pod.

---

## Key Learning

Kubernetes debugging usually follows:

1. Check pods
2. Check services
3. Check endpoints
4. Check labels/selectors
5. Check logs
6. Check deployments
7. Verify DNS/service names

Most issues can be found using:

```bash
kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```
