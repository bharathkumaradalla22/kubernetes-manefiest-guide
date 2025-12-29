# Kubernetes Probes - Implementation Guide

## Understanding Kubernetes Health Probes

Kubernetes provides **three types of health probes** to monitor container health:

### 1. **Liveness Probe** 🔴
**Purpose:** Determines if a container is running properly  
**Action:** If it fails, Kubernetes **restarts the container**  
**Use Case:** Detect deadlocks, infinite loops, or unresponsive applications  

**Example Scenario:**
- Your app crashes or hangs
- The container is running but the app is frozen
- Liveness probe fails → Kubernetes restarts the container

**When to Use:**
- Applications that can become unresponsive
- Long-running processes that might hang
- Apps that need automatic recovery

---

### 2. **Readiness Probe** 🟡
**Purpose:** Determines if a container is ready to receive traffic  
**Action:** If it fails, Kubernetes **removes the Pod from Service endpoints** (no restart)  
**Use Case:** Wait for initialization, database connections, or temporary unavailability  

**Example Scenario:**
- Your app is starting up and loading data
- Database connection is temporarily lost
- Readiness probe fails → No traffic sent to this Pod (other Pods handle requests)
- Once probe succeeds → Pod receives traffic again

**When to Use:**
- Apps with slow startup times
- Apps that need external dependencies (database, cache)
- Apps that experience temporary unavailability

---

### 3. **Startup Probe** 🟢
**Purpose:** Determines if the application has successfully started  
**Action:** If it fails, Kubernetes **restarts the container**  
**Use Case:** For slow-starting applications, prevents premature liveness/readiness checks  

**Example Scenario:**
- Your legacy app takes 2 minutes to start
- Without startup probe: Liveness probe might kill it during startup
- With startup probe: Liveness/readiness probes are **disabled until startup succeeds**

**When to Use:**
- Legacy applications with long initialization
- Apps that load large datasets on startup
- Containers with unpredictable startup times

---

## Probe Execution Flow

```
Container Start
     ↓
Startup Probe Active (if configured)
     ↓ (keeps checking)
Startup Succeeds
     ↓
Liveness Probe Active ← Continuous monitoring
     ↓
Readiness Probe Active ← Continuous monitoring
     ↓
Container Ready & Receiving Traffic
```

**Key Points:**
- **Startup probe** runs FIRST (if configured)
- **Liveness/Readiness** are DISABLED until startup succeeds
- After startup succeeds, **liveness** and **readiness** run continuously
- All probes can use: HTTP, TCP, or Exec methods

---

## Files That Need Health Probes

This guide shows which YAML files need health probes and exactly what to add.

---

## 1. pod.yaml

**Location:** Add under `spec.containers[].resources`

**All Three Probes (Complete Configuration):**

```yaml
    # Startup Probe - Runs first, protects slow-starting apps
    startupProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 30  # 30 * 10s = 5 minutes max startup time
    
    # Liveness Probe - Detects if container is alive
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    # Readiness Probe - Detects if container can receive traffic
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
      timeoutSeconds: 3
      failureThreshold: 3
```

**Explanation:**
- **startupProbe**: Allows up to 5 minutes for nginx to start (usually starts in seconds)
- **livenessProbe**: After startup, checks every 10s if nginx is responding. Restarts if it fails 3 times (30s)
- **readinessProbe**: Checks every 5s if nginx can serve traffic. Removes from load balancer if it fails

**Insert After:** The `resources.limits` section

---

## 2. deployment.yaml

**Status:** ✅ Already has probes - No changes needed

---

## 3. replicaset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**All Three Probes (Complete Configuration):**

```yaml
        # Startup Probe - Runs first
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30
        
        # Liveness Probe - Monitors health
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Readiness Probe - Controls traffic routing
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Explanation:**
- **startupProbe**: Waits for nginx to start (max 5 minutes)
- **livenessProbe**: Restarts container if nginx stops responding
- **readinessProbe**: Only sends traffic when nginx is ready

**Insert After:** The `resources.limits` section

---

## 4. statefulset.yaml

**Location:** Add under `spec.template.spec.containers[].env`

**All Three Probes (Complete Configuration):**

```yaml
        # Startup Probe - MySQL initialization can take time
        startupProbe:
          exec:
            command:
            - mysqladmin
            - ping
            - -h
            - localhost
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 60  # 60 * 10s = 10 minutes for database initialization
        
        # Liveness Probe - Checks if MySQL daemon is responsive
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
        
        # Readiness Probe - Verifies MySQL can execute queries
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

**Explanation:**
- **startupProbe**: MySQL initialization (schema creation, recovery) can take up to 10 minutes
- **livenessProbe**: Uses `mysqladmin ping` to check if MySQL daemon is alive. Restarts if unresponsive
- **readinessProbe**: Executes `SELECT 1` query to verify MySQL can process queries. Controls traffic routing

**Insert After:** The `env` section, before `volumeMounts`

---

## 5. daemonset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**All Three Probes (Complete Configuration):**

```yaml
        # Startup Probe - Fluentd initialization
        startupProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 30  # 30 * 5s = 2.5 minutes max startup time
        
        # Liveness Probe - Ensures Fluentd is accepting connections
        livenessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Readiness Probe - Ensures Fluentd can receive logs
        readinessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Explanation:**
- **startupProbe**: Waits for Fluentd to start accepting connections (max 2.5 minutes)
- **livenessProbe**: Checks TCP port 24224. Restarts Fluentd if port is not accepting connections
- **readinessProbe**: Verifies Fluentd is ready to receive log data from applications

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

| File | Startup | Liveness | Readiness | Probe Type | Reason |
|------|---------|----------|-----------|------------|--------|
| pod.yaml | ✅ YES | ✅ YES | ✅ YES | HTTP | Web server |
| deployment.yaml | ⚠️ PARTIAL | ✅ HAS | ✅ HAS | HTTP | Add startup probe |
| replicaset.yaml | ✅ YES | ✅ YES | ✅ YES | HTTP | Web server |
| statefulset.yaml | ✅ YES | ✅ YES | ✅ YES | Exec | Database (slow start) |
| daemonset.yaml | ✅ YES | ✅ YES | ✅ YES | TCP | Log collector |
| job.yaml | ❌ NO | ❌ NO | ❌ NO | - | Run-to-completion |
| cronjob.yaml | ❌ NO | ❌ NO | ❌ NO | - | Run-to-completion |
| complete-app-stack.yaml | ⚠️ PARTIAL | ✅ HAS | ✅ HAS | Mixed | Add startup probes |

---

## Probe Types Explained

### 🟢 Startup Probe
- **Purpose:** One-time check during container startup
- **Disables:** Liveness and readiness probes until it succeeds
- **Failure Action:** Restarts the container
- **Best For:** Slow-starting applications, legacy apps, database initialization

### 🔴 Liveness Probe  
- **Purpose:** Continuous check if container is alive
- **Runs:** After startup probe succeeds (or immediately if no startup probe)
- **Failure Action:** Restarts the container
- **Best For:** Detecting deadlocks, hung processes, crashed applications

### 🟡 Readiness Probe
- **Purpose:** Continuous check if container can handle traffic
- **Runs:** After startup probe succeeds (or immediately if no startup probe)
- **Failure Action:** Removes Pod from Service endpoints (no restart)
- **Best For:** Temporary unavailability, warming up, dependency failures

---

## Probe Mechanisms - Which to Use?

### 1. HTTP GET Probe (httpGet)
**Use For:** Web servers, REST APIs, HTTP services  
**How It Works:** Sends HTTP GET request to specified path and port  
**Success:** HTTP status code 200-399

```yaml
livenessProbe:
  httpGet:
    path: /health        # Health check endpoint
    port: 8080           # Application port
    httpHeaders:         # Optional custom headers
    - name: Custom-Header
      value: HealthCheck
  initialDelaySeconds: 30
  periodSeconds: 10
```

**Best For:**
- nginx, Apache web servers
- REST APIs (Node.js, Python Flask, Go)
- Microservices with health endpoints

---

### 2. TCP Socket Probe (tcpSocket)
**Use For:** Non-HTTP services, network services  
**How It Works:** Attempts to open TCP connection to specified port  
**Success:** Connection established successfully

```yaml
livenessProbe:
  tcpSocket:
    port: 3306          # Database port or service port
  initialDelaySeconds: 15
  periodSeconds: 10
```

**Best For:**
- Databases (MySQL, PostgreSQL, Redis)
- Message queues (RabbitMQ, Kafka)
- Log collectors (Fluentd, Logstash)
- Any TCP-based service

---

### 3. Exec Command Probe (exec)
**Use For:** Custom checks, complex validation  
**How It Works:** Executes command inside container  
**Success:** Command exits with status code 0

```yaml
livenessProbe:
  exec:
    command:
    - sh
    - -c
    - "pg_isready -U postgres"  # PostgreSQL check
  initialDelaySeconds: 30
  periodSeconds: 10
```

**Best For:**
- Database-specific checks (mysqladmin, pg_isready)
- File existence checks
- Custom application scripts
- Complex validation logic

---

## Probe Configuration Parameters

| Parameter | Description | Default | Recommended |
|-----------|-------------|---------|-------------|
| **initialDelaySeconds** | Wait time before first probe | 0 | Startup: 0-10<br>Liveness: 30-60<br>Readiness: 5-10 |
| **periodSeconds** | How often to probe | 10 | Startup: 5-10<br>Liveness: 10<br>Readiness: 5 |
| **timeoutSeconds** | Probe timeout | 1 | 3-5 seconds |
| **successThreshold** | Min consecutive successes | 1 | 1 (except readiness: 1-2) |
| **failureThreshold** | Max consecutive failures | 3 | Startup: 30-60<br>Liveness: 3<br>Readiness: 3 |

### Calculation Examples:

**Startup Probe:**
```
Max startup time = failureThreshold × periodSeconds
Example: 60 × 10s = 10 minutes max
```

**Liveness Probe:**
```
Time until restart = failureThreshold × periodSeconds
Example: 3 × 10s = 30 seconds of failures before restart
```

**Readiness Probe:**
```
Time until traffic removed = failureThreshold × periodSeconds  
Example: 3 × 5s = 15 seconds before removing from load balancer
```

---

## Probe Selection Guide (Old - See Above for Detailed Version)

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

## Real-World Scenarios

### Scenario 1: Web Application with Database
**Problem:** App crashes occasionally, needs 30s to warm up  
**Solution:**
```yaml
startupProbe:       # Wait for app initialization
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 30  # 5 min max

livenessProbe:      # Detect crashes
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:     # Wait for DB connection
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 3
```

### Scenario 2: Legacy Database (Slow Startup)
**Problem:** MySQL takes 5 minutes to initialize schema  
**Solution:**
```yaml
startupProbe:       # Allow long initialization
  exec:
    command: ["mysqladmin", "ping"]
  periodSeconds: 10
  failureThreshold: 60  # 10 min max

livenessProbe:      # Detect hung processes
  exec:
    command: ["mysqladmin", "ping"]
  periodSeconds: 30

readinessProbe:     # Verify query capability
  exec:
    command: ["mysql", "-e", "SELECT 1"]
  periodSeconds: 10
```

### Scenario 3: Microservice with Dependencies
**Problem:** Service depends on external API, can become temporarily unavailable  
**Solution:**
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 5
  failureThreshold: 20

livenessProbe:      # Only check service itself
  httpGet:
    path: /health   # Should NOT check external deps
    port: 8080
  periodSeconds: 10

readinessProbe:     # Check external dependencies
  httpGet:
    path: /ready    # SHOULD check if deps available
    port: 8080
  periodSeconds: 5
  failureThreshold: 1  # Quick removal from service
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

### Issue 1: Pod keeps restarting (CrashLoopBackOff)
**Cause:** Liveness probe failing repeatedly  
**Symptoms:**
```bash
kubectl get pods
NAME           READY   STATUS             RESTARTS   AGE
myapp-xxx      0/1     CrashLoopBackOff   5          3m
```

**Solutions:**
1. **Check if startup probe is needed:**
   ```yaml
   startupProbe:  # Add this if app starts slowly
     httpGet:
       path: /health
       port: 8080
     failureThreshold: 30
   ```

2. **Increase liveness initialDelaySeconds:**
   ```yaml
   livenessProbe:
     initialDelaySeconds: 60  # Give more time
   ```

3. **View container logs:**
   ```bash
   kubectl logs <pod-name> --previous  # See why it crashed
   ```

---

### Issue 2: Pod never becomes READY
**Cause:** Readiness probe always failing  
**Symptoms:**
```bash
kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
myapp-xxx      0/1     Running   0          5m
```

**Solutions:**
1. **Check readiness endpoint:**
   ```bash
   kubectl port-forward pod/myapp-xxx 8080:8080
   curl http://localhost:8080/health
   ```

2. **View detailed events:**
   ```bash
   kubectl describe pod myapp-xxx
   # Look for "Readiness probe failed: ..."
   ```

3. **Verify dependencies are available:**
   - Database connection
   - External API access
   - ConfigMaps/Secrets mounted

---

### Issue 3: Probe timeout errors
**Cause:** Application responds too slowly  
**Symptoms:**
```
Readiness probe failed: Get "http://10.1.2.3:8080/health": context deadline exceeded
```

**Solutions:**
1. **Increase timeout:**
   ```yaml
   readinessProbe:
     timeoutSeconds: 10  # From default 1s
   ```

2. **Optimize health endpoint:**
   - Don't query database in liveness checks
   - Cache health status
   - Use lightweight checks

3. **Example: Good vs Bad health endpoints:**
   ```python
   # ❌ BAD - Too slow
   @app.route('/health')
   def health():
       db.execute("SELECT COUNT(*) FROM large_table")  # Slow!
       return "OK"
   
   # ✅ GOOD - Fast
   @app.route('/health')
   def health():
       return "OK", 200  # Just check if app responds
   ```

---

### Issue 4: Startup probe kills slow container
**Cause:** failureThreshold too low for startup time  
**Symptoms:**
```
Container failed startup probe, will be restarted
```

**Solution:**
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 10
  failureThreshold: 60  # Increase: 60 × 10s = 10 min max
```

---

### Issue 5: Service not receiving traffic
**Cause:** Readiness probe not passing  
**Symptoms:**
```bash
kubectl get endpoints myapp-service
NAME            ENDPOINTS   AGE
myapp-service   <none>      5m  # Should show Pod IPs
```

**Solutions:**
1. **Check readiness probe status:**
   ```bash
   kubectl describe pod myapp-xxx | grep -A5 "Readiness"
   ```

2. **Verify probe configuration matches service port:**
   ```yaml
   # Service
   ports:
   - port: 80
     targetPort: 8080
   
   # Probe should check targetPort
   readinessProbe:
     httpGet:
       port: 8080  # Must match targetPort
   ```

---

## Best Practices Summary

### ✅ DO:

1. **Always use startup probes for slow-starting apps**
   ```yaml
   startupProbe:  # Protects from premature liveness checks
     failureThreshold: 30
   ```

2. **Keep liveness checks simple and fast**
   ```yaml
   livenessProbe:
     httpGet:
       path: /health  # Just "is app alive?"
   ```

3. **Use readiness for dependency checks**
   ```yaml
   readinessProbe:
     httpGet:
       path: /ready  # "Can I handle traffic?"
   ```

4. **Set appropriate timeouts**
   - Startup: High failureThreshold (30-60)
   - Liveness: Medium delay (30-60s), low failure (3)
   - Readiness: Low delay (5-10s), low failure (3)

5. **Use different endpoints for each probe**
   ```
   /startup - One-time initialization check
   /health  - Simple alive check
   /ready   - Full dependency check
   ```

### ❌ DON'T:

1. **Don't check external dependencies in liveness probe**
   ```yaml
   # ❌ BAD - Will restart if DB is down
   livenessProbe:
     httpGet:
       path: /check-database
   ```

2. **Don't use exec probes unnecessarily**
   - Slower than HTTP/TCP
   - Uses container resources
   - Only use when HTTP/TCP won't work

3. **Don't set failureThreshold too low**
   ```yaml
   # ❌ BAD - Too aggressive
   livenessProbe:
     failureThreshold: 1  # One failure = restart
   ```

4. **Don't forget to test probes before production**
   ```bash
   # Simulate probe failure
   kubectl exec -it myapp-xxx -- rm /health
   # Watch what happens
   kubectl get pods -w
   ```

---

## Common Issues and Solutions (Old - See Above for Detailed Version)

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
