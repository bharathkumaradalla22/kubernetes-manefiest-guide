# Complete YAML Guide for Kubernetes

## Table of Contents
1. [YAML Basics & Syntax](#yaml-basics--syntax)
2. [YAML Structure Fundamentals](#yaml-structure-fundamentals)
3. [Kubernetes YAML Structure](#kubernetes-yaml-structure)
4. [Step-by-Step YAML Explanations](#step-by-step-yaml-explanations)
5. [Combining Multiple Resources](#combining-multiple-resources)
6. [Best Practices](#best-practices)

---

# YAML Basics & Syntax

## What is YAML?

**YAML** = "YAML Ain't Markup Language"
- Human-readable data serialization format
- Uses indentation (spaces, NOT tabs)
- Case-sensitive
- File extension: `.yaml` or `.yml`

## Basic YAML Rules

### 1. Indentation
```yaml
# Use 2 spaces for indentation (consistent throughout)
parent:
  child:
    grandchild: value
```

### 2. Key-Value Pairs
```yaml
# Simple key-value
name: nginx
version: 1.21
port: 80

# String with spaces (quotes optional but recommended for special chars)
description: "This is a web server"
message: 'Single quotes work too'

# Numbers
replicas: 3
cpu: 500
pi: 3.14

# Boolean
enabled: true
disabled: false
```

### 3. Lists/Arrays
```yaml
# Dash notation (most common)
fruits:
- apple
- banana
- orange

# Inline notation
colors: [red, green, blue]

# Nested lists
teams:
- name: frontend
  members:
  - alice
  - bob
- name: backend
  members:
  - charlie
  - david
```

### 4. Objects/Maps
```yaml
# Nested objects
person:
  name: John Doe
  age: 30
  address:
    street: 123 Main St
    city: New York
    zip: 10001

# Inline notation
coordinates: {x: 10, y: 20, z: 30}
```

### 5. Multi-line Strings
```yaml
# Preserve newlines with pipe |
description: |
  This is line 1
  This is line 2
  This is line 3

# Fold newlines with >
summary: >
  This is a very long sentence that
  will be folded into a single line
  without newlines.

# Result: "This is a very long sentence that will be folded into a single line without newlines."
```

### 6. Comments
```yaml
# This is a comment
name: nginx  # Inline comment
# Multi-line comments require
# each line to start with #
```

### 7. Null Values
```yaml
empty_value: null
also_empty: ~
no_value:
```

### 8. Anchors & References (Reusability)
```yaml
# Define anchor with &
default_settings: &defaults
  memory: 256Mi
  cpu: 500m

# Reference anchor with *
container1:
  <<: *defaults
  name: app1

container2:
  <<: *defaults
  name: app2
  memory: 512Mi  # Override specific value
```

---

# YAML Structure Fundamentals

## Common Data Types in YAML

```yaml
# Strings
string1: Hello World
string2: "123"  # This is a string, not a number
string3: 'true'  # This is a string, not a boolean

# Numbers
integer: 42
float: 3.14
negative: -10
exponential: 1.2e+3

# Boolean
isTrue: true
isFalse: false
yesValue: yes
noValue: no

# Null
nothing: null
empty: ~

# Dates (ISO 8601)
date: 2025-12-29
datetime: 2025-12-29T10:30:00Z

# Lists
simple_list:
- item1
- item2
- item3

complex_list:
- name: item1
  value: 100
- name: item2
  value: 200

# Objects/Maps
person:
  name: John
  age: 30
  hobbies:
  - reading
  - coding
```

## YAML Structure Best Practices

### ✅ DO:
```yaml
# Use 2-space indentation consistently
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

### ❌ DON'T:
```yaml
# Don't use tabs
apiVersion: v1
kind: Pod
metadata:
	name: my-pod  # TAB used here - WRONG!

# Don't mix indentation
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
    containers:  # 4 spaces instead of 2 - INCONSISTENT!
  - name: nginx
```

---

# Kubernetes YAML Structure

## Basic Kubernetes Resource Template

Every Kubernetes resource follows this structure:

```yaml
apiVersion: <api-version>    # Which API version to use
kind: <resource-type>        # What type of resource
metadata:                    # Information about the resource
  name: <resource-name>      # Name of the resource
  namespace: <namespace>     # Which namespace (optional)
  labels:                    # Key-value pairs for organization
    key: value
  annotations:               # Non-identifying metadata
    key: value
spec:                        # Desired state specification
  # Resource-specific configuration
status:                      # Current state (managed by Kubernetes)
  # Automatically updated by system
```

## Common API Versions

```yaml
# Core resources
apiVersion: v1
kind: Pod, Service, ConfigMap, Secret, Namespace, PersistentVolume, PersistentVolumeClaim

# Apps resources
apiVersion: apps/v1
kind: Deployment, StatefulSet, DaemonSet, ReplicaSet

# Batch resources
apiVersion: batch/v1
kind: Job, CronJob

# Networking
apiVersion: networking.k8s.io/v1
kind: Ingress, NetworkPolicy, IngressClass

# RBAC
apiVersion: rbac.authorization.k8s.io/v1
kind: Role, RoleBinding, ClusterRole, ClusterRoleBinding

# Storage
apiVersion: storage.k8s.io/v1
kind: StorageClass

# Autoscaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

# Policy
apiVersion: policy/v1
kind: PodDisruptionBudget
```

---

# Step-by-Step YAML Explanations

## 1. Pod - Complete Breakdown

```yaml
apiVersion: v1                    # API version for core resources
kind: Pod                         # Resource type: Pod
metadata:                         # Metadata section
  name: nginx-pod                 # Pod name (must be unique in namespace)
  labels:                         # Labels for organization and selection
    app: nginx                    # Label: app=nginx
    tier: frontend                # Label: tier=frontend
spec:                             # Specification section (desired state)
  containers:                     # List of containers in this pod
  - name: nginx                   # Container name
    image: nginx:1.21             # Docker image to use
    ports:                        # Ports to expose
    - containerPort: 80           # Container listens on port 80
    resources:                    # Resource constraints
      requests:                   # Minimum resources guaranteed
        memory: "64Mi"            # Request 64 megabytes memory
        cpu: "250m"               # Request 250 millicores (0.25 CPU)
      limits:                     # Maximum resources allowed
        memory: "128Mi"           # Limit to 128 megabytes
        cpu: "500m"               # Limit to 500 millicores (0.5 CPU)
    env:                          # Environment variables
    - name: ENVIRONMENT           # Variable name
      value: "production"         # Variable value
    volumeMounts:                 # Mount volumes into container
    - name: config-volume         # Name of volume to mount
      mountPath: /etc/config      # Path inside container
  volumes:                        # Define volumes for the pod
  - name: config-volume           # Volume name
    configMap:                    # Volume source: ConfigMap
      name: app-config            # Name of ConfigMap to use
  restartPolicy: Always           # Restart policy: Always, OnFailure, Never
```

**Key Points:**
- **apiVersion**: Tells Kubernetes which API to use
- **kind**: Type of resource we're creating
- **metadata.name**: Unique identifier within namespace
- **metadata.labels**: Used for selecting and organizing resources
- **spec.containers**: Can have multiple containers in one pod
- **resources.requests**: Kubernetes schedules based on this
- **resources.limits**: Container killed if exceeded
- **volumeMounts**: Connects volumes to container filesystem

---

## 2. Deployment - Complete Breakdown

```yaml
apiVersion: apps/v1               # Apps API for workload controllers
kind: Deployment                  # Resource type: Deployment
metadata:
  name: nginx-deployment          # Deployment name
  labels:
    app: nginx                    # Label for the deployment itself
spec:                             # Deployment specification
  replicas: 3                     # Number of pod replicas to maintain
  selector:                       # How deployment finds pods to manage
    matchLabels:                  # Pods with these labels are managed
      app: nginx                  # Must match template labels below
  template:                       # Pod template (blueprint for pods)
    metadata:
      labels:                     # Labels applied to created pods
        app: nginx                # MUST match selector above
    spec:                         # Pod specification (same as Pod resource)
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:            # Checks if container is alive
          httpGet:                # HTTP GET request
            path: /               # URL path to check
            port: 80              # Port to check
          initialDelaySeconds: 30 # Wait 30s before first check
          periodSeconds: 10       # Check every 10 seconds
        readinessProbe:           # Checks if container is ready to serve
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5  # Wait 5s before first check
          periodSeconds: 5        # Check every 5 seconds
  strategy:                       # Update strategy
    type: RollingUpdate           # Update pods gradually
    rollingUpdate:
      maxSurge: 1                 # Max 1 extra pod during update
      maxUnavailable: 1           # Max 1 pod unavailable during update
```

**Key Points:**
- **replicas**: Desired number of pods
- **selector.matchLabels**: MUST match template.metadata.labels
- **template**: Pod template used to create new pods
- **livenessProbe**: Restarts container if fails
- **readinessProbe**: Removes from service if fails
- **strategy**: How updates are performed
- **maxSurge**: Extra pods allowed during update
- **maxUnavailable**: How many pods can be down during update

---

## 3. Service (ClusterIP) - Complete Breakdown

```yaml
apiVersion: v1                    # Core API version
kind: Service                     # Resource type: Service
metadata:
  name: nginx-service-clusterip   # Service name (DNS name)
  labels:
    app: nginx
spec:
  type: ClusterIP                 # Service type: ClusterIP (internal only)
  selector:                       # Selects pods with these labels
    app: nginx                    # Routes traffic to pods with app=nginx
  ports:                          # Port mappings
  - name: http                    # Port name (optional but recommended)
    protocol: TCP                 # Protocol: TCP or UDP
    port: 80                      # Service port (what clients connect to)
    targetPort: 80                # Pod port (where traffic is sent)
  sessionAffinity: ClientIP       # Stick client to same pod
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800       # Session timeout: 3 hours
```

**Key Points:**
- **type: ClusterIP**: Internal service (default)
- **selector**: Matches pods by labels
- **port**: Port exposed by service
- **targetPort**: Port on the pod (can be name or number)
- **sessionAffinity**: ClientIP = sticky sessions

**How it works:**
1. Service gets a stable ClusterIP (e.g., 10.96.0.1)
2. DNS name: `nginx-service-clusterip.default.svc.cluster.local`
3. Traffic to service IP:80 → pods with app=nginx on port 80
4. Load balanced across all matching pods

---

## 4. Service (NodePort) - Complete Breakdown

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-nodeport
  labels:
    app: nginx
spec:
  type: NodePort                  # Service type: NodePort (external access)
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80                      # ClusterIP port
    targetPort: 80                # Pod port
    nodePort: 30080               # Port on each node (30000-32767)
```

**Key Points:**
- **type: NodePort**: Accessible from outside cluster
- **nodePort**: Port on every node (30000-32767 range)
- Automatically creates ClusterIP too
- Access: `http://<node-ip>:30080`

---

## 5. Service (LoadBalancer) - Complete Breakdown

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-loadbalancer
  labels:
    app: nginx
spec:
  type: LoadBalancer             # Service type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  - name: https                  # Multiple ports
    protocol: TCP
    port: 443
    targetPort: 443
  loadBalancerSourceRanges:      # Restrict source IPs (firewall)
  - "0.0.0.0/0"                  # Allow from anywhere (use specific IPs in production)
```

**Key Points:**
- **type: LoadBalancer**: Creates cloud load balancer
- Requires cloud provider (AWS, GCP, Azure)
- Gets external IP address
- Automatically creates NodePort and ClusterIP

---

## 6. ConfigMap - Complete Breakdown

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:                             # Configuration data
  # Simple key-value pairs
  database_host: "mysql.example.com"
  database_port: "3306"
  log_level: "info"
  
  # File-like keys (entire file content)
  app.properties: |               # Pipe | preserves newlines
    app.name=MyApplication
    app.version=1.0.0
    app.environment=production
    
  nginx.conf: |                   # Another file
    server {
      listen 80;
      server_name localhost;
      location / {
        root /usr/share/nginx/html;
        index index.html;
      }
    }
```

**Usage in Pod:**

```yaml
# Method 1: All keys as environment variables
envFrom:
- configMapRef:
    name: app-config

# Method 2: Specific key as environment variable
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: database_host

# Method 3: Mount as files
volumes:
- name: config
  configMap:
    name: app-config
volumeMounts:
- name: config
  mountPath: /etc/config
  # Creates /etc/config/database_host, /etc/config/app.properties, etc.
```

---

## 7. Secret - Complete Breakdown

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: default
type: Opaque                      # Secret type: generic
data:                             # Base64 encoded values
  username: YWRtaW4=              # "admin" in base64
  password: cGFzc3dvcmQxMjM=      # "password123" in base64
  api-key: bXktc2VjcmV0LWFwaS1rZXk=

---
# TLS Secret
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls          # TLS certificate type
data:
  tls.crt: LS0tLS1CRUdJTi...      # Certificate (base64)
  tls.key: LS0tLS1CRUdJTi...      # Private key (base64)

---
# Docker Registry Secret
apiVersion: v1
kind: Secret
metadata:
  name: docker-registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRo...  # Docker config (base64)
```

**Create Secret:**
```bash
# From literal values
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=password123

# From file
kubectl create secret generic app-secret \
  --from-file=./secret-file.txt

# TLS certificate
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key
```

**Usage in Pod:**
```yaml
# As environment variable
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secret
      key: password

# As volume
volumes:
- name: secret-volume
  secret:
    secretName: app-secret
volumeMounts:
- name: secret-volume
  mountPath: /etc/secrets
  readOnly: true
```

**Encode/Decode:**
```bash
# Encode
echo -n "admin" | base64              # YWRtaW4=

# Decode
echo "YWRtaW4=" | base64 --decode     # admin
```

---

## 8. StatefulSet - Complete Breakdown

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: "mysql"            # Headless service name (required)
  replicas: 3                     # Number of replicas
  selector:
    matchLabels:
      app: mysql
  template:                       # Pod template
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql             # Named port
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: password
        volumeMounts:
        - name: mysql-data        # Must match volumeClaimTemplates name
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
  volumeClaimTemplates:           # Creates PVC for each pod
  - metadata:
      name: mysql-data            # PVC name template
    spec:
      accessModes: 
      - ReadWriteOnce             # Each pod gets its own volume
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi           # Size per pod
```

**Key Points:**
- **serviceName**: Required headless service
- **Stable network IDs**: mysql-statefulset-0, mysql-statefulset-1, mysql-statefulset-2
- **Stable DNS**: mysql-statefulset-0.mysql.default.svc.cluster.local
- **volumeClaimTemplates**: Each pod gets its own PVC
- **Ordered operations**: Pods created/deleted in order (0, 1, 2)

**Headless Service (required):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql                     # Must match serviceName in StatefulSet
spec:
  clusterIP: None                 # Headless service
  selector:
    app: mysql
  ports:
  - port: 3306
    name: mysql
```

---

## 9. DaemonSet - Complete Breakdown

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      tolerations:                # Allow scheduling on master nodes
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            memory: 100Mi
            cpu: 50m
        volumeMounts:             # Access node filesystems
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:                    # HostPath volumes (node filesystem)
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

**Key Points:**
- **One pod per node** automatically
- **tolerations**: Allow on master/control plane nodes
- **hostPath volumes**: Access node filesystem
- Perfect for: logging, monitoring, network plugins

---

## 10. Job - Complete Breakdown

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  completions: 1                  # Number of successful completions needed
  parallelism: 1                  # Number of pods to run in parallel
  backoffLimit: 3                 # Number of retries before marking failed
  activeDeadlineSeconds: 600      # Max time for job (10 minutes)
  template:
    metadata:
      labels:
        app: backup
    spec:
      containers:
      - name: backup
        image: busybox:1.34
        command: 
        - "sh"
        - "-c"
        - "echo Running backup at $(date); sleep 30; echo Backup completed"
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
      restartPolicy: OnFailure    # OnFailure or Never (not Always)
```

**Key Points:**
- **completions**: Total successful pods needed
- **parallelism**: Concurrent pods
- **backoffLimit**: Max failures before giving up
- **activeDeadlineSeconds**: Job timeout
- **restartPolicy**: Must be OnFailure or Never (not Always)

**Examples:**
```yaml
# Run once
completions: 1
parallelism: 1

# Run 5 times sequentially
completions: 5
parallelism: 1

# Run 5 times in parallel
completions: 5
parallelism: 5

# Work queue (run until queue empty)
# Don't set completions
parallelism: 3
```

---

## 11. CronJob - Complete Breakdown

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"           # Cron schedule: Daily at 2 AM
  successfulJobsHistoryLimit: 3   # Keep last 3 successful jobs
  failedJobsHistoryLimit: 1       # Keep last 1 failed job
  concurrencyPolicy: Forbid       # Allow, Forbid, or Replace
  jobTemplate:                    # Job template
    spec:
      template:                   # Pod template
        metadata:
          labels:
            app: backup-cron
        spec:
          containers:
          - name: backup
            image: busybox:1.34
            command:
            - "sh"
            - "-c"
            - "echo Running scheduled backup at $(date); sleep 30; echo Backup completed"
            resources:
              requests:
                memory: "64Mi"
                cpu: "100m"
              limits:
                memory: "128Mi"
                cpu: "200m"
          restartPolicy: OnFailure
```

**Cron Schedule Format:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
* * * * *

Examples:
"0 2 * * *"      Every day at 2:00 AM
"*/5 * * * *"    Every 5 minutes
"0 */4 * * *"    Every 4 hours
"0 0 * * 0"      Every Sunday at midnight
"0 0 1 * *"      1st of every month at midnight
"30 3 * * 1-5"   Weekdays at 3:30 AM
```

**Concurrency Policy:**
- **Allow**: Multiple jobs can run concurrently
- **Forbid**: Skip new run if previous still running
- **Replace**: Cancel current and start new

---

## 12. Ingress - Complete Breakdown

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:                    # Ingress controller specific settings
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx         # Which ingress controller to use
  tls:                            # HTTPS configuration
  - hosts:
    - example.com
    - www.example.com
    secretName: tls-secret        # TLS certificate secret
  rules:                          # Routing rules
  - host: example.com             # Domain name
    http:
      paths:
      - path: /                   # URL path
        pathType: Prefix          # Prefix, Exact, or ImplementationSpecific
        backend:
          service:
            name: nginx-service-clusterip
            port:
              number: 80
  - host: api.example.com         # Different domain
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

**Path Types:**
- **Prefix**: Matches /path and /path/*
- **Exact**: Exact match only
- **ImplementationSpecific**: Depends on ingress controller

**Routing Examples:**
```yaml
# Host-based routing
rules:
- host: app1.example.com
  http:
    paths:
    - path: /
      backend:
        service:
          name: app1-service
          port:
            number: 80
- host: app2.example.com
  http:
    paths:
    - path: /
      backend:
        service:
          name: app2-service
          port:
            number: 80

# Path-based routing
rules:
- host: example.com
  http:
    paths:
    - path: /api
      backend:
        service:
          name: api-service
    - path: /web
      backend:
        service:
          name: web-service
```

---

## 13. PersistentVolume & PersistentVolumeClaim

### PersistentVolume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local
  labels:
    type: local
spec:
  storageClassName: manual        # Storage class name
  capacity:
    storage: 10Gi                 # Total storage capacity
  accessModes:                    # How volume can be mounted
    - ReadWriteOnce               # RWO: Single node read-write
  persistentVolumeReclaimPolicy: Retain  # Retain, Recycle, or Delete
  hostPath:                       # Volume type (testing only)
    path: "/mnt/data"
```

**Access Modes:**
- **ReadWriteOnce (RWO)**: Single node, read-write
- **ReadOnlyMany (ROX)**: Multiple nodes, read-only
- **ReadWriteMany (RWX)**: Multiple nodes, read-write

**Reclaim Policies:**
- **Retain**: Keep data after PVC deletion (manual cleanup)
- **Recycle**: Basic scrub (rm -rf) - deprecated
- **Delete**: Delete storage asset (default for dynamic)

### PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-local
spec:
  storageClassName: manual        # Must match PV storageClassName
  accessModes:
    - ReadWriteOnce               # Must be subset of PV accessModes
  resources:
    requests:
      storage: 5Gi                # Amount requested (≤ PV capacity)
```

**Using PVC in Pod:**
```yaml
spec:
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-local        # Reference PVC name
```

---

## 14. RBAC - Complete Breakdown

### ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-serviceaccount
  namespace: default
automountServiceAccountToken: true  # Auto-mount SA token in pods
```

### Role (Namespace-scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default              # Role is namespace-scoped
rules:                            # List of permissions
- apiGroups: [""]                 # "" = core API group
  resources: ["pods"]             # Resource type
  verbs: ["get", "watch", "list"] # Allowed actions
- apiGroups: [""]
  resources: ["pods/log"]         # Subresource
  verbs: ["get"]
```

**API Groups:**
- `""` (empty): core (pods, services, configmaps, secrets)
- `apps`: deployments, statefulsets, daemonsets
- `batch`: jobs, cronjobs
- `networking.k8s.io`: ingresses, networkpolicies

**Verbs (Actions):**
- `get`: Read single resource
- `list`: List all resources
- `watch`: Watch for changes
- `create`: Create new resource
- `update`: Update existing resource
- `patch`: Partially update resource
- `delete`: Delete resource
- `deletecollection`: Delete multiple resources
- `*`: All verbs

### RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:                         # Who gets permissions
- kind: User                      # User, Group, or ServiceAccount
  name: jane
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: app-serviceaccount
  namespace: default
roleRef:                          # Which role to bind
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole (Cluster-wide)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-read
rules:
- apiGroups: ["*"]                # All API groups
  resources: ["*"]                # All resources
  verbs: ["get", "list", "watch"] # Read-only
```

### ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-all-binding
subjects:
- kind: User
  name: viewer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-read
  apiGroup: rbac.authorization.k8s.io
```

---

## 15. HorizontalPodAutoscaler - Complete Breakdown

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:                 # What to scale
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2                  # Minimum replicas
  maxReplicas: 10                 # Maximum replicas
  metrics:                        # Scaling metrics
  - type: Resource                # Metric type
    resource:
      name: cpu                   # CPU metric
      target:
        type: Utilization         # Utilization or AverageValue
        averageUtilization: 70    # Target 70% CPU
  - type: Resource
    resource:
      name: memory                # Memory metric
      target:
        type: Utilization
        averageUtilization: 80    # Target 80% memory
  behavior:                       # Scaling behavior (optional)
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scale down
      policies:
      - type: Percent
        value: 50                 # Scale down by 50% max
        periodSeconds: 60
      - type: Pods
        value: 2                  # Scale down by 2 pods max
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0    # Scale up immediately
      policies:
      - type: Percent
        value: 100                # Scale up by 100% max
        periodSeconds: 30
      - type: Pods
        value: 4                  # Scale up by 4 pods max
        periodSeconds: 30
```

**Key Points:**
- Requires metrics-server installed
- Scales based on CPU, memory, or custom metrics
- **behavior**: Fine-tune scaling speed and stability

---

## 16. NetworkPolicy - Complete Breakdown

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:                    # Pods this policy applies to
    matchLabels:
      app: backend
  policyTypes:                    # Ingress, Egress, or both
  - Ingress
  ingress:                        # Incoming traffic rules
  - from:                         # Allow from these sources
    - podSelector:                # Pods in same namespace
        matchLabels:
          app: frontend
    ports:                        # On these ports
    - protocol: TCP
      port: 8080
```

**Common Patterns:**

### Deny All Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}                 # All pods in namespace
  policyTypes:
  - Ingress
  # No ingress rules = deny all
```

### Allow from Namespace
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:          # From specific namespace
        matchLabels:
          name: frontend-namespace
```

### Allow Egress to DNS
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

---

## 17. ResourceQuota & LimitRange

### ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:                           # Hard limits
    requests.cpu: "10"            # Total CPU requests
    requests.memory: 20Gi         # Total memory requests
    limits.cpu: "20"              # Total CPU limits
    limits.memory: 40Gi           # Total memory limits
    persistentvolumeclaims: "10"  # Max PVCs
    requests.storage: 100Gi       # Total storage requests
    pods: "50"                    # Max pods
    services: "20"                # Max services
    configmaps: "20"              # Max configmaps
    secrets: "20"                 # Max secrets
```

### LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-memory-limit-range
  namespace: development
spec:
  limits:
  - max:                          # Maximum allowed
      cpu: "2"
      memory: 2Gi
    min:                          # Minimum required
      cpu: 100m
      memory: 64Mi
    default:                      # Default limits (if not specified)
      cpu: 500m
      memory: 512Mi
    defaultRequest:               # Default requests (if not specified)
      cpu: 250m
      memory: 256Mi
    type: Container               # Applies to containers
  - max:
      cpu: "4"
      memory: 4Gi
    min:
      cpu: 200m
      memory: 128Mi
    type: Pod                     # Applies to pods
```

---

# Combining Multiple Resources

## Method 1: Multiple Documents in One File (Using ---)

```yaml
# File: full-stack-app.yaml

# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: myapp

---  # Document separator

# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  database_url: "mysql.myapp.svc.cluster.local"
  log_level: "info"

---

# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: myapp
type: Opaque
data:
  db_password: cGFzc3dvcmQxMjM=

---

# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: db_password

---

# Service
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: myapp
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---

# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: myapp
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Apply all resources:**
```bash
kubectl apply -f full-stack-app.yaml
```

---

## Method 2: Complete Application Stack Example

```yaml
# File: wordpress-stack.yaml

# 1. Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: wordpress

---

# 2. MySQL Secret
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: wordpress
type: Opaque
stringData:  # No need to base64 encode
  root-password: rootpassword123
  database: wordpress
  user: wpuser
  password: wppassword123

---

# 3. MySQL PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: wordpress
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---

# 4. MySQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: user
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc

---

# 5. MySQL Service
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: wordpress
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
  clusterIP: None  # Headless service

---

# 6. WordPress PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
  namespace: wordpress
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---

# 7. WordPress Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql
        - name: WORDPRESS_DB_NAME
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: database
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: user
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        volumeMounts:
        - name: wordpress-storage
          mountPath: /var/www/html
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      volumes:
      - name: wordpress-storage
        persistentVolumeClaim:
          claimName: wordpress-pvc

---

# 8. WordPress Service
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress
spec:
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer

---

# 9. HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
  namespace: wordpress
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Deploy entire stack:**
```bash
# Create everything
kubectl apply -f wordpress-stack.yaml

# Check status
kubectl get all -n wordpress

# Get WordPress URL
kubectl get service wordpress -n wordpress

# Delete entire stack
kubectl delete -f wordpress-stack.yaml
```

---

## Method 3: Kustomize Structure

### Directory Structure
```
my-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── overlays/
│   ├── development/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   └── production/
│       ├── kustomization.yaml
│       ├── patch.yaml
│       └── replica-count.yaml
```

### base/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
- configmap.yaml

commonLabels:
  app: myapp
```

### overlays/production/kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

namespace: production

replicas:
- name: myapp
  count: 5

images:
- name: myapp
  newTag: v1.2.3

patchesStrategicMerge:
- patch.yaml
```

**Apply with Kustomize:**
```bash
# Apply production overlay
kubectl apply -k overlays/production/

# View generated manifests
kubectl kustomize overlays/production/
```

---

# Best Practices

## 1. YAML Writing Best Practices

### ✅ DO:
```yaml
# Use consistent 2-space indentation
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21

# Use quotes for strings with special characters
env:
- name: MESSAGE
  value: "Hello, World!"

# Use meaningful names
metadata:
  name: frontend-web-server
  labels:
    app: frontend
    component: web-server
    environment: production

# Add comments for clarity
spec:
  replicas: 3  # Three instances for high availability
```

### ❌ DON'T:
```yaml
# Don't use tabs
spec:
	containers:  # TAB used

# Don't mix indentation
spec:
  containers:
    - name: nginx  # 4 spaces instead of 2

# Don't use cryptic names
metadata:
  name: app1  # Too generic

# Don't skip quotes when needed
message: Hello, World!  # Comma can cause issues
```

## 2. Resource Organization

### Single File per Resource Type
```
kubernetes/
├── namespaces/
│   └── production.yaml
├── configmaps/
│   ├── app-config.yaml
│   └── nginx-config.yaml
├── secrets/
│   └── app-secrets.yaml
├── deployments/
│   ├── frontend.yaml
│   └── backend.yaml
├── services/
│   ├── frontend-svc.yaml
│   └── backend-svc.yaml
└── ingress/
    └── app-ingress.yaml
```

### Combined by Application
```
kubernetes/
├── frontend/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── ingress.yaml
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── secret.yaml
└── database/
    ├── statefulset.yaml
    ├── service.yaml
    └── pvc.yaml
```

## 3. Labels and Selectors

**Standard labels:**
```yaml
metadata:
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/instance: myapp-prod
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: myapp-stack
    app.kubernetes.io/managed-by: kubectl
    environment: production
    team: frontend-team
```

## 4. Resource Requests and Limits

**Always set them:**
```yaml
resources:
  requests:
    memory: "256Mi"  # Scheduling decision
    cpu: "250m"      # 0.25 CPU
  limits:
    memory: "512Mi"  # Hard limit
    cpu: "500m"      # Hard limit
```

## 5. Health Checks

**Always configure probes:**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## 6. Security

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

## 7. ConfigMaps vs Secrets

**ConfigMap:** Non-sensitive configuration
```yaml
apiVersion: v1
kind: ConfigMap
data:
  api_url: "https://api.example.com"
  timeout: "30"
```

**Secret:** Sensitive data
```yaml
apiVersion: v1
kind: Secret
type: Opaque
stringData:  # Auto-encodes to base64
  password: "secretpassword"
  api_key: "abc123xyz"
```

## 8. Validation

```bash
# Validate syntax
kubectl apply -f deployment.yaml --dry-run=client

# Validate on server (with admission controllers)
kubectl apply -f deployment.yaml --dry-run=server

# View effective configuration
kubectl apply -f deployment.yaml --dry-run=client -o yaml

# Diff before applying
kubectl diff -f deployment.yaml
```

## 9. Version Control

```bash
# .gitignore
*.swp
*~
.DS_Store
secrets/*.yaml  # Never commit secrets!
```

**Use sealed-secrets or external secret managers for secrets in Git**

## 10. Documentation

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    # Documentation annotations
    description: "Frontend web application"
    owner: "frontend-team@example.com"
    version: "1.2.3"
    changelog: "Added new features X, Y, Z"
spec:
  # ...
```

---

# Common Use Cases

## 1. Simple Web Application

```yaml
# Deployment + Service + Ingress
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: myapp:v1
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
```

## 2. Microservices Architecture

```yaml
# Frontend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: frontend:v1
        env:
        - name: BACKEND_URL
          value: "http://backend"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 8080

# Backend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: backend:v1
        env:
        - name: DATABASE_URL
          value: "mysql://database:3306"
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

## 3. Batch Processing

```yaml
# CronJob for regular batch processing
apiVersion: batch/v1
kind: CronJob
metadata:
  name: data-processor
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: processor
            image: data-processor:v1
            env:
            - name: INPUT_BUCKET
              value: "s3://input-data"
            - name: OUTPUT_BUCKET
              value: "s3://processed-data"
          restartPolicy: OnFailure
```

---

# Troubleshooting YAML

## Common YAML Errors

### 1. Indentation Error
```yaml
# ❌ Wrong
spec:
containers:  # Should be indented
- name: nginx

# ✅ Correct
spec:
  containers:
  - name: nginx
```

### 2. Missing Dash for List Items
```yaml
# ❌ Wrong
containers:
  name: nginx  # Missing dash

# ✅ Correct
containers:
- name: nginx
```

### 3. Wrong Data Type
```yaml
# ❌ Wrong
replicas: "3"  # String instead of number

# ✅ Correct
replicas: 3
```

### 4. Invalid YAML
```bash
# Validate YAML syntax
kubectl apply -f deployment.yaml --dry-run=client

# Use YAML linter
yamllint deployment.yaml

# Check Kubernetes schema
kubectl apply -f deployment.yaml --dry-run=server -o yaml
```

---

# Quick Reference Card

## Essential Commands
```bash
# Apply
kubectl apply -f file.yaml
kubectl apply -f directory/
kubectl apply -k kustomize-dir/

# Get
kubectl get pods
kubectl get all
kubectl get pod -o yaml

# Describe
kubectl describe pod pod-name

# Edit
kubectl edit deployment name

# Delete
kubectl delete -f file.yaml
kubectl delete pod pod-name

# Logs
kubectl logs pod-name
kubectl logs -f pod-name

# Execute
kubectl exec -it pod-name -- /bin/bash

# Port Forward
kubectl port-forward pod-name 8080:80
```

## YAML Template
```yaml
apiVersion: <group>/<version>
kind: <Kind>
metadata:
  name: <name>
  namespace: <namespace>
  labels:
    <key>: <value>
  annotations:
    <key>: <value>
spec:
  # Resource-specific configuration
```

---

**Remember**: 
- Always validate YAML before applying
- Use version control for all manifests
- Start simple and add complexity gradually
- Use labels consistently for organization
- Document your configurations
- Test in non-production first

This guide provides the foundation for understanding and writing Kubernetes YAML files. Practice with these examples and gradually build more complex configurations!
