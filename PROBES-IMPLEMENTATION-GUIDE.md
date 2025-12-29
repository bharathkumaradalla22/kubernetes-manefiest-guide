# Kubernetes Probes - Implementation Guide

## Understanding Kubernetes Health Probes - Execution Order

Kubernetes provides **three types of health probes** that execute in a specific order:

### **Probe Execution Order: 1️⃣ → 2️⃣ → 3️⃣**

```
1. Startup Probe (FIRST) 🟢
        ↓
2. Liveness Probe (SECOND) 🔴
        ↓
3. Readiness Probe (THIRD) 🟡
```

---

## 1️⃣ Startup Probe 🟢 (Executes FIRST)

**Order:** Runs **FIRST** when container starts  
**Purpose:** Determines if the application has successfully started  
**Action:** If it fails, Kubernetes **restarts the container**  
**Blocks:** Liveness and Readiness probes are **DISABLED** until startup succeeds  

**Example Scenario:**
- Your legacy app takes 2 minutes to start
- Without startup probe: Liveness probe might kill it during startup
- With startup probe: Liveness/readiness probes wait until startup succeeds

**When to Use:**
- Legacy applications with long initialization (1-10 minutes)
- Apps that load large datasets on startup
- Containers with unpredictable startup times
- Database initialization (schema creation, recovery)

**Configuration:**
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  initialDelaySeconds: 0        # Start checking immediately
  periodSeconds: 10             # Check every 10 seconds
  timeoutSeconds: 5             # Wait 5s for response
  failureThreshold: 30          # 30 × 10s = 5 minutes max startup
```

---

## 2️⃣ Liveness Probe 🔴 (Executes SECOND)

**Order:** Runs **AFTER** startup probe succeeds (or immediately if no startup probe)  
**Purpose:** Determines if a container is running properly  
**Action:** If it fails, Kubernetes **restarts the container**  
**Continuous:** Keeps checking throughout container lifetime  

**Example Scenario:**
- Your app crashes or hangs
- The container is running but the app is frozen
- Liveness probe fails → Kubernetes restarts the container

**When to Use:**
- Applications that can become unresponsive
- Long-running processes that might hang or deadlock
- Apps that need automatic recovery from crashes

**Configuration:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30       # Wait 30s after container starts
  periodSeconds: 10             # Check every 10 seconds
  timeoutSeconds: 5             # Wait 5s for response
  failureThreshold: 3           # Restart after 3 failures (30s)
```

---

## 3️⃣ Readiness Probe 🟡 (Executes THIRD)

**Order:** Runs **AFTER** startup probe succeeds (or immediately if no startup probe)  
**Purpose:** Determines if a container is ready to receive traffic  
**Action:** If it fails, Kubernetes **removes Pod from Service endpoints** (no restart)  
**Continuous:** Keeps checking throughout container lifetime  

**Example Scenario:**
- Your app is starting up and loading cache
- Database connection is temporarily lost
- Readiness probe fails → No traffic sent to this Pod (other Pods handle requests)
- Once probe succeeds → Pod receives traffic again

**When to Use:**
- Apps with slow startup times or warming up period
- Apps that need external dependencies (database, cache, APIs)
- Apps that experience temporary unavailability

**Configuration:**
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5        # Start checking after 5s
  periodSeconds: 5              # Check every 5 seconds
  timeoutSeconds: 3             # Wait 3s for response
  failureThreshold: 3           # Remove from service after 3 failures (15s)
```

---

## Complete Probe Execution Flow

```
Container Start
     ↓
┌────────────────────────────────┐
│ 1️⃣ Startup Probe (FIRST)      │
│    Checks: Is app initialized? │
│    Blocks: Liveness & Readiness│
└────────────────────────────────┘
     ↓ (periodSeconds: 10s)
     ↓ (keeps checking)
     ↓
✅ Startup Succeeds!
     ↓
     ├─────────────────────────────────┐
     ↓                                 ↓
┌────────────────────────┐    ┌─────────────────────────┐
│ 2️⃣ Liveness Probe      │    │ 3️⃣ Readiness Probe      │
│    (Runs continuously)  │    │    (Runs continuously)   │
│ Checks: Is app alive?   │    │ Checks: Can handle load? │
│ Action: Restart on fail │    │ Action: Remove traffic   │
└────────────────────────┘    └─────────────────────────┘
     ↓ (every 10s)                    ↓ (every 5s)
     ↓                                ↓
Container Running & Receiving Traffic
```

**Key Points:**
- **Startup** runs FIRST and only once (until it succeeds)
- **Liveness** and **Readiness** are DISABLED until startup succeeds
- After startup succeeds, **Liveness** and **Readiness** run CONTINUOUSLY
- All three probes can run SIMULTANEOUSLY after startup
- All probes can use: HTTP, TCP, or Exec methods

---

## Files That Need Health Probes

This guide shows which YAML files need health probes and exactly what to add **in the correct order**.

---

## 1. pod.yaml

**Location:** Add under `spec.containers[].resources`

**All Three Probes in Correct Order (1️⃣ 2️⃣ 3️⃣):**

```yaml
    # 1️⃣ FIRST: Startup Probe - Runs first, protects slow-starting apps
    startupProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 0
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 30  # 30 × 10s = 5 minutes max startup time
    
    # 2️⃣ SECOND: Liveness Probe - Detects if container is alive
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    # 3️⃣ THIRD: Readiness Probe - Detects if container can receive traffic
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
- **1️⃣ startupProbe**: Allows up to 5 minutes for nginx to start (usually starts in seconds)
- **2️⃣ livenessProbe**: After startup, checks every 10s if nginx is responding. Restarts if it fails 3 times (30s)
- **3️⃣ readinessProbe**: Checks every 5s if nginx can serve traffic. Removes from load balancer if it fails

**Insert After:** The `resources.limits` section

---

## 2. deployment.yaml

**Status:** ⚠️ Already has liveness/readiness - **Add startup probe**

**Add Startup Probe (1️⃣) at the beginning:**

```yaml
        # 1️⃣ Add this FIRST
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 0
          periodSeconds: 10
          failureThreshold: 30
        
        # 2️⃣ Already exists
        livenessProbe:
          # ... existing config
        
        # 3️⃣ Already exists
        readinessProbe:
          # ... existing config
```

---

## 3. replicaset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**All Three Probes in Correct Order (1️⃣ 2️⃣ 3️⃣):**

```yaml
        # 1️⃣ FIRST: Startup Probe - Runs first
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30
        
        # 2️⃣ SECOND: Liveness Probe - Monitors health
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # 3️⃣ THIRD: Readiness Probe - Controls traffic routing
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
- **1️⃣ startupProbe**: Waits for nginx to start (max 5 minutes)
- **2️⃣ livenessProbe**: Restarts container if nginx stops responding
- **3️⃣ readinessProbe**: Only sends traffic when nginx is ready

**Insert After:** The `resources.limits` section

---

## 4. statefulset.yaml

**Location:** Add under `spec.template.spec.containers[].env`

**All Three Probes in Correct Order (1️⃣ 2️⃣ 3️⃣):**

```yaml
        # 1️⃣ FIRST: Startup Probe - MySQL initialization can take time
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
          failureThreshold: 60  # 60 × 10s = 10 minutes for database initialization
        
        # 2️⃣ SECOND: Liveness Probe - Checks if MySQL daemon is responsive
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
        
        # 3️⃣ THIRD: Readiness Probe - Verifies MySQL can execute queries
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
- **1️⃣ startupProbe**: MySQL initialization (schema creation, recovery) can take up to 10 minutes
- **2️⃣ livenessProbe**: Uses `mysqladmin ping` to check if MySQL daemon is alive. Restarts if unresponsive
- **3️⃣ readinessProbe**: Executes `SELECT 1` query to verify MySQL can process queries. Controls traffic routing

**Insert After:** The `env` section, before `volumeMounts`

---

## 5. daemonset.yaml

**Location:** Add under `spec.template.spec.containers[].resources`

**All Three Probes in Correct Order (1️⃣ 2️⃣ 3️⃣):**

```yaml
        # 1️⃣ FIRST: Startup Probe - Fluentd initialization
        startupProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 30  # 30 × 5s = 2.5 minutes max startup time
        
        # 2️⃣ SECOND: Liveness Probe - Ensures Fluentd is accepting connections
        livenessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # 3️⃣ THIRD: Readiness Probe - Ensures Fluentd can receive logs
        readinessProbe:
          tcpSocket:
            port: 24224
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

**Explanation:**
- **1️⃣ startupProbe**: Waits for Fluentd to start accepting connections (max 2.5 minutes)
- **2️⃣ livenessProbe**: Checks TCP port 24224. Restarts Fluentd if port is not accepting connections
- **3️⃣ readinessProbe**: Verifies Fluentd is ready to receive log data from applications

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

**Status:** ⚠️ Already has liveness/readiness - **Add startup probes**

**Update all deployments to include startup probe FIRST:**

```yaml
        # 1️⃣ Add startup probe FIRST
        startupProbe:
          httpGet:
            path: /health
            port: 8080
          periodSeconds: 10
          failureThreshold: 30
        
        # 2️⃣ Existing liveness probe
        livenessProbe:
          # ... existing config
        
        # 3️⃣ Existing readiness probe
        readinessProbe:
          # ... existing config
```

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

| File | 1️⃣ Startup | 2️⃣ Liveness | 3️⃣ Readiness | Probe Type | Reason |
|------|---------|----------|-----------|------------|--------|
| pod.yaml | ✅ YES | ✅ YES | ✅ YES | HTTP | Web server |
| deployment.yaml | ⚠️ ADD | ✅ HAS | ✅ HAS | HTTP | Add startup probe first |
| replicaset.yaml | ✅ YES | ✅ YES | ✅ YES | HTTP | Web server |
| statefulset.yaml | ✅ YES | ✅ YES | ✅ YES | Exec | Database (slow start) |
| daemonset.yaml | ✅ YES | ✅ YES | ✅ YES | TCP | Log collector |
| job.yaml | ❌ NO | ❌ NO | ❌ NO | - | Run-to-completion |
| cronjob.yaml | ❌ NO | ❌ NO | ❌ NO | - | Run-to-completion |
| complete-app-stack.yaml | ⚠️ ADD | ✅ HAS | ✅ HAS | Mixed | Add startup probes first |

---

## Probe Execution Order Explained

### Why This Order Matters:

**❌ Wrong Order (Random):**
```yaml
readinessProbe:  # Wrong - should be third
  ...
startupProbe:    # Wrong - should be first
  ...
livenessProbe:   # Wrong - should be second
  ...
```
**Result:** Works, but confusing to read and maintain

**✅ Correct Order (1️⃣ 2️⃣ 3️⃣):**
```yaml
# 1️⃣ FIRST: Startup Probe
startupProbe:
  ...

# 2️⃣ SECOND: Liveness Probe  
livenessProbe:
  ...

# 3️⃣ THIRD: Readiness Probe
readinessProbe:
  ...
```
**Result:** Clear, follows execution order, easy to understand

---

## Probe Mechanisms - Which to Use?

### 1. HTTP GET Probe (httpGet)
**Use For:** Web servers, REST APIs, HTTP services  
**How It Works:** Sends HTTP GET request to specified path and port  
**Success:** HTTP status code 200-399

```yaml
# Order: 1️⃣ 2️⃣ 3️⃣
startupProbe:
  httpGet:
    path: /startup      # Startup check endpoint
    port: 8080
  periodSeconds: 10
  failureThreshold: 30

livenessProbe:
  httpGet:
    path: /health       # Health check endpoint
    port: 8080
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready        # Readiness check endpoint
    port: 8080
  periodSeconds: 5
  failureThreshold: 3
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
# Order: 1️⃣ 2️⃣ 3️⃣
startupProbe:
  tcpSocket:
    port: 3306
  periodSeconds: 10
  failureThreshold: 30

livenessProbe:
  tcpSocket:
    port: 3306          # Database port or service port
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  tcpSocket:
    port: 3306
  periodSeconds: 5
  failureThreshold: 3
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
# Order: 1️⃣ 2️⃣ 3️⃣
startupProbe:
  exec:
    command:
    - mysqladmin
    - ping
  periodSeconds: 10
  failureThreshold: 60

livenessProbe:
  exec:
    command:
    - mysqladmin
    - ping
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  exec:
    command:
    - mysql
    - -e
    - "SELECT 1"        # Verify can execute queries
  periodSeconds: 5
  failureThreshold: 3
```

**Best For:**
- Database-specific checks (mysqladmin, pg_isready)
- File existence checks
- Custom application scripts
- Complex validation logic

---

## Probe Configuration Parameters

| Parameter | Description | Default | Recommended by Probe |
|-----------|-------------|---------|---------------------|
| **initialDelaySeconds** | Wait time before first probe | 0 | **Startup:** 0-10<br>**Liveness:** 30-60<br>**Readiness:** 5-10 |
| **periodSeconds** | How often to probe | 10 | **Startup:** 5-10<br>**Liveness:** 10<br>**Readiness:** 5 |
| **timeoutSeconds** | Probe timeout | 1 | **All:** 3-5 seconds |
| **successThreshold** | Min consecutive successes | 1 | **All:** 1 (except readiness: 1-2) |
| **failureThreshold** | Max consecutive failures | 3 | **Startup:** 30-60<br>**Liveness:** 3<br>**Readiness:** 3 |

### Calculation Examples:

**1️⃣ Startup Probe:**
```
Max startup time = failureThreshold × periodSeconds
Example: 60 × 10s = 10 minutes max
```

**2️⃣ Liveness Probe:**
```
Time until restart = failureThreshold × periodSeconds
Example: 3 × 10s = 30 seconds of failures before restart
```

**3️⃣ Readiness Probe:**
```
Time until traffic removed = failureThreshold × periodSeconds  
Example: 3 × 5s = 15 seconds before removing from load balancer
```

---

## Real-World Scenarios (Correct Order: 1️⃣ 2️⃣ 3️⃣)

### Scenario 1: Web Application with Database
**Problem:** App crashes occasionally, needs 30s to warm up  
**Solution:**
```yaml
# 1️⃣ FIRST: Wait for app initialization
startupProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 30  # 5 min max

# 2️⃣ SECOND: Detect crashes
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
  failureThreshold: 3

# 3️⃣ THIRD: Wait for DB connection
readinessProbe:
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
# 1️⃣ FIRST: Allow long initialization
startupProbe:
  exec:
    command: ["mysqladmin", "ping"]
  periodSeconds: 10
  failureThreshold: 60  # 10 min max

# 2️⃣ SECOND: Detect hung processes
livenessProbe:
  exec:
    command: ["mysqladmin", "ping"]
  periodSeconds: 30
  failureThreshold: 3

# 3️⃣ THIRD: Verify query capability
readinessProbe:
  exec:
    command: ["mysql", "-e", "SELECT 1"]
  periodSeconds: 10
  failureThreshold: 3
```

### Scenario 3: Microservice with Dependencies
**Problem:** Service depends on external API, can become temporarily unavailable  
**Solution:**
```yaml
# 1️⃣ FIRST: Service initialization
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  periodSeconds: 5
  failureThreshold: 20

# 2️⃣ SECOND: Only check service itself
livenessProbe:
  httpGet:
    path: /health   # Should NOT check external deps
    port: 8080
  periodSeconds: 10
  failureThreshold: 3

# 3️⃣ THIRD: Check external dependencies
readinessProbe:
  httpGet:
    path: /ready    # SHOULD check if deps available
    port: 8080
  periodSeconds: 5
  failureThreshold: 1  # Quick removal from service
```

---

## Best Practices Summary

### ✅ DO:

1. **Always define probes in correct order: 1️⃣ 2️⃣ 3️⃣**
   ```yaml
   startupProbe:   # 1️⃣ FIRST
     ...
   livenessProbe:  # 2️⃣ SECOND
     ...
   readinessProbe: # 3️⃣ THIRD
     ...
   ```

2. **Always use startup probes for slow-starting apps**
   ```yaml
   startupProbe:  # Protects from premature liveness checks
     failureThreshold: 30
   ```

3. **Keep liveness checks simple and fast**
   ```yaml
   livenessProbe:
     httpGet:
       path: /health  # Just "is app alive?"
   ```

4. **Use readiness for dependency checks**
   ```yaml
   readinessProbe:
     httpGet:
       path: /ready  # "Can I handle traffic?"
   ```

5. **Use different endpoints for each probe**
   ```
   /startup - One-time initialization check
   /health  - Simple alive check
   /ready   - Full dependency check
   ```

### ❌ DON'T:

1. **Don't write probes in wrong order**
   ```yaml
   # ❌ BAD - Confusing order
   readinessProbe:
   startupProbe:
   livenessProbe:
   
   # ✅ GOOD - Correct order
   startupProbe:   # 1️⃣
   livenessProbe:  # 2️⃣
   readinessProbe: # 3️⃣
   ```

2. **Don't check external dependencies in liveness probe**
   ```yaml
   # ❌ BAD - Will restart if DB is down
   livenessProbe:
     httpGet:
       path: /check-database  # WRONG!
   
   # ✅ GOOD - Check only the app
   livenessProbe:
     httpGet:
       path: /health  # Just app health
   
   # ✅ GOOD - Check dependencies in readiness
   readinessProbe:
     httpGet:
       path: /check-database  # OK here!
   ```

3. **Don't forget startup probe for slow apps**
   ```yaml
   # ❌ BAD - No startup probe
   livenessProbe:
     initialDelaySeconds: 300  # Hacky workaround
   
   # ✅ GOOD - Use startup probe
   startupProbe:
     failureThreshold: 30  # Proper solution
   livenessProbe:
     initialDelaySeconds: 30  # Can be shorter now
   ```

---

## Quick Reference Card

### Probe Order Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│         Kubernetes Probe Execution Order                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1️⃣ STARTUP PROBE (First)                                   │
│     - Runs: ONCE during container start                    │
│     - Blocks: Liveness and Readiness                       │
│     - Fails: Container restarts                            │
│     - Config: High failureThreshold (30-60)                │
│                                                             │
│  2️⃣ LIVENESS PROBE (Second)                                 │
│     - Runs: CONTINUOUSLY after startup                     │
│     - Checks: Is app alive?                                │
│     - Fails: Container restarts                            │
│     - Config: Medium delay, low failures (3)               │
│                                                             │
│  3️⃣ READINESS PROBE (Third)                                 │
│     - Runs: CONTINUOUSLY after startup                     │
│     - Checks: Can handle traffic?                          │
│     - Fails: Removes from service (no restart)             │
│     - Config: Low delay, low failures (3)                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### YAML Template (Correct Order)

```yaml
containers:
- name: myapp
  image: myapp:latest
  ports:
  - containerPort: 8080
  
  # 1️⃣ FIRST: Startup Probe
  startupProbe:
    httpGet:
      path: /startup
      port: 8080
    initialDelaySeconds: 0
    periodSeconds: 10
    timeoutSeconds: 5
    failureThreshold: 30      # 5 min max
  
  # 2️⃣ SECOND: Liveness Probe
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 5
    failureThreshold: 3       # 30s before restart
  
  # 3️⃣ THIRD: Readiness Probe
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
    timeoutSeconds: 3
    failureThreshold: 3       # 15s before traffic removal
```

---

## Next Steps

1. ✅ **Understand probe execution order** (1️⃣ Startup → 2️⃣ Liveness → 3️⃣ Readiness)
2. ✅ **Review which files need probes** (see Summary Table above)
3. ✅ **Copy the appropriate probe configuration IN ORDER**
4. ✅ **Add to your YAML files in the specified locations**
5. ✅ **Test the configuration** (watch for events in correct order)
6. ✅ **Deploy to cluster**
7. ✅ **Monitor probe results**

---

**For more detailed examples and best practices, see:**
- [LIVENESS-READINESS-PROBES-GUIDE.md](LIVENESS-READINESS-PROBES-GUIDE.md)

---

**Remember: The order matters for readability and understanding the flow!**  
**Always write: 1️⃣ Startup → 2️⃣ Liveness → 3️⃣ Readiness** 🎯
