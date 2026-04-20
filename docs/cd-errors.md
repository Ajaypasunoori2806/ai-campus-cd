# 🚨 CD Issues & Fixes (AKS + ArgoCD + Monitoring)

This section documents real-world issues encountered during deployment on AKS and their resolutions.

---

## ❌ 1. ImagePullBackOff

### Cause:

* Image not available in ACR
* Wrong image name/tag

### Fix:

Verified image exists in ACR and is correctly tagged.

```bash
docker images
docker tag ai-campus-app:latest aicampusacr2806.azurecr.io/ai-campus-app:latest
docker push aicampusacr2806.azurecr.io/ai-campus-app:latest
```

---

## ❌ 2. CreateContainerConfigError

### Error:

```
Error: secret "openai-secret" not found
```

### Cause:

* Kubernetes secret missing

### Fix:

Created secret manually:

```bash
kubectl create secret generic openai-secret \
--from-literal=OPENAI_API_KEY=<your-api-key>
```

Verified:

```bash
kubectl get secrets
```

---

## ❌ 3. CrashLoopBackOff

### Cause:

* Missing dependency: `python-multipart`

### Error:

```
Form data requires "python-multipart" to be installed
```

### Fix:

Updated `requirements.txt`:

```text
python-multipart
```

Rebuilt & pushed image:

```bash
docker build -t ai-campus-app .
docker tag ai-campus-app aicampusacr2806.azurecr.io/ai-campus-app:latest
docker push aicampusacr2806.azurecr.io/ai-campus-app:latest
```

Restarted deployment:

```bash
kubectl rollout restart deployment ai-campus
```

---

## ❌ 4. External IP Pending (LoadBalancer)

### Cause:

Azure limit reached:

```
PublicIPCountLimitReached
```

### Fix:

* Deleted LoadBalancer service
* Switched to NodePort

```bash
kubectl delete svc ai-campus-service
```

Updated service.yaml:

```yaml
type: NodePort
```

---

## ❌ 5. Service Not Accessible

### Cause:

* Azure NSG not allowing NodePort

### Fix:

Added inbound rule:

* Port: 30691
* Protocol: TCP
* Action: Allow

---

## ❌ 6. Port Forward Failed

### Error:

```
address already in use
```

### Fix:

Used different port:

```bash
kubectl port-forward pod/<pod-name> 8090:8000
```

---

## ❌ 7. Application Running but Blank Page

### Cause:

* Root endpoint not defined

### Fix:

Added root API in FastAPI:

```python
@app.get("/")
def root():
    return {"message": "Server is running"}
```

---

## ❌ 8. Ingress External IP Not Assigned

### Cause:

* Public IP quota exhausted

### Fix:

Reused existing public IP from Azure:

```bash
az network public-ip list -o table
```

Patched ingress service:

```bash
kubectl patch svc ingress-nginx-controller -n ingress-nginx \
-p '{"spec": {"loadBalancerIP": "20.228.91.240"}}'
```

---

## ❌ 9. Helm Install Failed

### Error:

```
cannot re-use a name that is still in use
```

### Fix:

```bash
helm uninstall monitoring -n monitoring
helm install monitoring prometheus-community/kube-prometheus-stack \
--namespace monitoring --create-namespace
```

---

## ❌ 10. Grafana Login Failed

### Cause:

Wrong password used

### Fix:

Fetched correct password:

```bash
kubectl get secret monitoring-grafana -n monitoring \
-o jsonpath="{.data.admin-password}" | base64 -d
```

---

## ❌ 11. Prometheus Targets Not Showing

### Cause:

No metrics endpoint exposed

### Fix:

Ensure app exposes `/metrics` endpoint (if needed)

---

## 🎯 Summary

Major issues faced:

* Kubernetes Secrets
* Azure limits (Public IP)
* Dependency failures
* Networking & access
* Monitoring setup

All issues were debugged and resolved during deployment.
