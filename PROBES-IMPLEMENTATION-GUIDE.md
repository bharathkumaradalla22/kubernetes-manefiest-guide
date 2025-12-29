# Kubernetes Probes - Implementation Guide

## Files That Need Liveness and Readiness Probes

This guide shows which YAML files need health probes and exactly what to add.

---

## 1. pod.yaml

**Location:** Add under `spec.containers[].resources`

**Probes to Add:**

```yaml
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

**Why:**
- Liveness: Ensures nginx is responding
- Readiness: Ensures nginx is ready to serve traffic

**Insert After:** The `resources.limits` section

---

## 2. deployment.yaml

**Status:** ✅ Already has probes - No changes needed

---

## 3. replicaset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**Probes to Add:**

```yaml
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Why:**
- Web server needs health checks to ensure it's serving traffic properly

**Insert After:** The `resources.limits` section

---

## 4. statefulset.yaml

**Location:** Add under `spec.template.spec.containers[].env`

**Probes to Add:**

```yaml
        livenessProbe:
          exec:
            command:
            - mysqladmin
            - ping
            - -h
            - localhost
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          exec:
            command:
            - mysql
            - -h
            - localhost
            - -e
            - "SELECT 1"
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Why:**
- Liveness: Checks if MySQL daemon is responsive
- Readiness: Verifies MySQL can execute queries

**Insert After:** The `env` section, before `volumeMounts`

---

## 5. daemonset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**Probes to Add:**

```yaml
        livenessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Why:**
- Fluentd needs to be accepting connections on its default port
- TCP check is appropriate for log collection service

**Insert After:** The `resources` section, before `volumeMounts`

---

## 6. job.yaml

**Status:** ❌ No probes needed

**Reason:** Jobs are run-to-completion tasks that don't serve ongoing traffic. They exit when done.

---

## 7. cronjob.yaml

**Status:** ❌ No probes needed

**Reason:** CronJobs create Jobs which are run-to-completion tasks.

---

## 8. complete-app-stack.yaml

**Status:** ✅ Already has probes for all deployments - No changes needed

---

## Files That Don't Need Probes

These resource types don't run containers or don't need health checks:

- ❌ **namespace.yaml** - No containers
- ❌ **configmap.yaml** - Configuration only
- ❌ **secret.yaml** - Configuration only
- ❌ **service-clusterip.yaml** - Network resource
- ❌ **service-nodeport.yaml** - Network resource
- ❌ **service-loadbalancer.yaml** - Network resource
- ❌ **ingress.yaml** - Routing resource
- ❌ **persistentvolume.yaml** - Storage resource
- ❌ **persistentvolumeclaim.yaml** - Storage request
- ❌ **storageclass.yaml** - Storage configuration
- ❌ **serviceaccount.yaml** - Identity resource
- ❌ **role.yaml** - RBAC resource
- ❌ **rolebinding.yaml** - RBAC resource
- ❌ **clusterrole.yaml** - RBAC resource
- ❌ **clusterrolebinding.yaml** - RBAC resource
- ❌ **horizontalpodautoscaler.yaml** - Autoscaling config
- ❌ **verticalpodautoscaler.yaml** - Autoscaling config
- ❌ **poddisruptionbudget.yaml** - Disruption policy
- ❌ **networkpolicy.yaml** - Network security
- ❌ **resourcequota.yaml** - Resource limits
- ❌ **limitrange.yaml** - Default limits

---

## Summary Table

| File | Needs Probes? | Probe Type | Reason |
|------|---------------|------------|--------|
| pod.yaml | ✅ YES | HTTP | Web server |
| deployment.yaml | ✅ HAS | HTTP | Already configured |
| replicaset.yaml | ✅ YES | HTTP | Web server |
| statefulset.yaml | ✅ YES | Exec | Database |
| daemonset.yaml | ✅ YES | TCP | Log collector |
| job.yaml | ❌ NO | - | Run-to-completion |
| cronjob.yaml | ❌ NO | - | Run-to-completion |
| complete-app-stack.yaml | ✅ HAS | Mixed | Already configured |

---

## Probe Selection Guide

### HTTP Probe
**Use for:** Web servers, REST APIs, HTTP services
**Files:** pod.yaml, replicaset.yaml, deployment.yaml (done)

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

### TCP Probe
**Use for:** Non-HTTP services, databases without exec access
**Files:** daemonset.yaml

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```

### Exec Probe
**Use for:** Custom checks, database queries
**Files:** statefulset.yaml

```yaml
livenessProbe:
  exec:
    command:
    - mysqladmin
    - ping
```

---

## Implementation Steps

### For pod.yaml:
1. Open `pod.yaml`
2. Find line with `cpu: "500m"` (under limits)
3. Add the HTTP probes after the `resources` section
4. Ensure proper indentation (4 spaces for container properties)

### For replicaset.yaml:
1. Open `replicaset.yaml`
2. Find line with `cpu: "500m"` (under limits)
3. Add the HTTP probes after the `resources` section
4. Ensure proper indentation (8 spaces for pod template container properties)

### For statefulset.yaml:
1. Open `statefulset.yaml`
2. Find the `env` section (after MYSQL_ROOT_PASSWORD)
3. Add the Exec probes after the `env` section
4. Ensure proper indentation (8 spaces for pod template container properties)

### For daemonset.yaml:
1. Open `daemonset.yaml`
2. Find line with `cpu: 50m` (under requests)
3. Add the TCP probes after the `resources` section
4. Ensure proper indentation (8 spaces for pod template container properties)

---

## Testing Your Probes

After adding probes, test them:

```bash
# Apply the updated YAML
kubectl apply -f <filename>.yaml

# Check pod status
kubectl get pods

# Describe pod to see probe results
kubectl describe pod <pod-name>

# Look for these events:
# - "Liveness probe succeeded"
# - "Readiness probe succeeded"
# - "Liveness probe failed" (if something is wrong)

# Check if pod is ready
kubectl get pods -o wide

# The READY column should show 1/1 when readiness probe passes
```

---

## Common Issues and Solutions

### Issue: Pod keeps restarting
**Cause:** Liveness probe failing
**Solution:** 
- Increase `initialDelaySeconds`
- Check if the path/port is correct
- View logs: `kubectl logs <pod-name> --previous`

### Issue: Pod never becomes ready
**Cause:** Readiness probe failing
**Solution:**
- Check application logs
- Verify the endpoint works: `kubectl port-forward pod/<name> 8080:80`
- Test endpoint: `curl localhost:8080/health`

### Issue: Probe timeout
**Cause:** Application too slow to respond
**Solution:**
- Increase `timeoutSeconds`
- Optimize the health check endpoint

---

## Quick Reference

### Timing Recommendations

**Web Applications:**
```yaml
initialDelaySeconds: 30
periodSeconds: 10
timeoutSeconds: 5
failureThreshold: 3
```

**Databases:**
```yaml
initialDelaySeconds: 30
periodSeconds: 10
timeoutSeconds: 5
failureThreshold: 3
```

**Log Collectors/DaemonSets:**
```yaml
initialDelaySeconds: 30
periodSeconds: 10
timeoutSeconds: 5
failureThreshold: 3
```

---

## Next Steps

1. ✅ Review which files need probes (see Summary Table above)
2. ✅ Copy the appropriate probe configuration
3. ✅ Add to your YAML files in the specified locations
4. ✅ Test the configuration
5. ✅ Deploy to cluster
6. ✅ Monitor probe results

---

**For detailed examples and best practices, see:**
- [LIVENESS-READINESS-PROBES-GUIDE.md](LIVENESS-READINESS-PROBES-GUIDE.md)
