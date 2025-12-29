# Combining Multiple YAML Files in Kubernetes

## Overview

In Kubernetes, you can define multiple resources in a single YAML file by separating them with `---` (three dashes). This is useful for:
- Deploying complete application stacks
- Managing related resources together
- Simplifying deployment commands
- Version control and GitOps workflows

---

## Basic Syntax

Use `---` (three dashes) to separate different resources in a single file:

```yaml
# Resource 1
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx

---

# Resource 2
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
```

---

## Complete Application Example

Here's a full-stack application with all resources in one file:

### File: `complete-app.yaml`

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: my-application
  labels:
    app: my-application

---

# ConfigMap for application configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: my-application
data:
  database_host: "postgres-service"
  database_port: "5432"
  database_name: "myapp"
  log_level: "info"
  app.properties: |
    app.name=MyApplication
    app.version=1.0.0
    app.environment=production

---

# Secret for sensitive data
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: my-application
type: Opaque
data:
  database_username: cG9zdGdyZXM=  # postgres
  database_password: cGFzc3dvcmQxMjM=  # password123
  api_key: bXktc2VjcmV0LWFwaS1rZXk=  # my-secret-api-key

---

# PersistentVolumeClaim for database storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: my-application
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---

# Database Deployment (PostgreSQL)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: my-application
  labels:
    app: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database_username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database_password
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_name
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc

---

# Database Service
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: my-application
  labels:
    app: postgres
spec:
  type: ClusterIP
  selector:
    app: postgres
  ports:
  - name: postgres
    protocol: TCP
    port: 5432
    targetPort: 5432

---

# Application Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-application
  namespace: my-application
  labels:
    app: my-application
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-application
  template:
    metadata:
      labels:
        app: my-application
    spec:
      containers:
      - name: app
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database_username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: database_password
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: api_key
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"

---

# Application Service
apiVersion: v1
kind: Service
metadata:
  name: my-application-service
  namespace: my-application
  labels:
    app: my-application
spec:
  type: ClusterIP
  selector:
    app: my-application
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080

---

# Ingress for external access
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-application-ingress
  namespace: my-application
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-application-service
            port:
              number: 80

---

# HorizontalPodAutoscaler for auto-scaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-application-hpa
  namespace: my-application
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-application
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

---

## Deployment Order Matters

Kubernetes applies resources in the order they appear in the file. Best practice order:

1. **Namespace** - Create the namespace first
2. **ConfigMaps and Secrets** - Configuration needed by pods
3. **PersistentVolumeClaims** - Storage resources
4. **Deployments/StatefulSets** - Workloads
5. **Services** - Service discovery
6. **Ingress** - External access
7. **HPA/VPA** - Auto-scaling

---

## Deploying the Combined YAML

```bash
# Apply all resources at once
kubectl apply -f complete-app.yaml

# View all created resources
kubectl get all -n my-application

# Check status
kubectl get pods -n my-application
kubectl get services -n my-application
kubectl get ingress -n my-application

# Delete all resources
kubectl delete -f complete-app.yaml
```

---

## Organizing by Environment

### Method 1: Separate Files per Environment

**base/app.yaml** (common resources)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: myapp:latest
```

**dev/complete-dev.yaml**
```yaml
# Import base resources
# Add dev-specific configs
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev
data:
  environment: "development"
  log_level: "debug"

---
# Include base app.yaml resources here with dev namespace
```

**prod/complete-prod.yaml**
```yaml
# Import base resources
# Add prod-specific configs
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  environment: "production"
  log_level: "warn"

---
# Include base app.yaml resources here with production namespace
```

### Method 2: Using Kustomize

**base/kustomization.yaml**
```yaml
resources:
- namespace.yaml
- deployment.yaml
- service.yaml
- configmap.yaml
```

**overlays/dev/kustomization.yaml**
```yaml
bases:
- ../../base
namespace: dev
replicas:
- name: my-app
  count: 1
```

**overlays/prod/kustomization.yaml**
```yaml
bases:
- ../../base
namespace: production
replicas:
- name: my-app
  count: 5
```

Deploy:
```bash
# Dev
kubectl apply -k overlays/dev/

# Production
kubectl apply -k overlays/prod/
```

---

## Best Practices

### ✅ DO:

1. **Group Related Resources**
   - Keep application components together
   - Example: app + database + service + ingress in one file

2. **Use Meaningful Names**
   ```yaml
   # Good
   frontend-deployment.yaml
   backend-complete.yaml
   database-stack.yaml
   
   # Bad
   app1.yaml
   file2.yaml
   ```

3. **Add Comments**
   ```yaml
   # Frontend Deployment
   # Handles all user-facing web traffic
   # Connects to backend-service
   apiVersion: apps/v1
   kind: Deployment
   ```

4. **Order Resources Logically**
   - Namespace first
   - ConfigMaps/Secrets before Deployments
   - Services after Deployments
   - Ingress last

5. **Use Labels Consistently**
   ```yaml
   labels:
     app: my-application
     component: frontend
     version: v1.0.0
     environment: production
   ```

### ❌ DON'T:

1. **Mix Unrelated Resources**
   - Don't combine frontend + monitoring + logging in one file
   - Keep logical boundaries

2. **Create Massive Files**
   - If file > 500 lines, consider splitting
   - Balance between convenience and maintainability

3. **Hardcode Environment-Specific Values**
   ```yaml
   # Bad
   image: myapp:production-v1.0.0
   
   # Good - use Kustomize or Helm
   image: myapp:VERSION
   ```

---

## Advanced Patterns

### 1. Multi-Tier Application

**File: `full-stack-app.yaml`**
```yaml
# Database Tier
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  POSTGRES_DB: myapp
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  # ... database config

# Backend Tier
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  DB_HOST: postgres-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  # ... backend config
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  # ... backend service

# Frontend Tier
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config
data:
  API_URL: http://backend-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  # ... frontend config
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  # ... frontend service
```

### 2. Microservices Pattern

**File: `microservices.yaml`**
```yaml
# Namespace for all microservices
apiVersion: v1
kind: Namespace
metadata:
  name: microservices

---
# User Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: microservices
# ...

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices
# ...

---
# Order Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: microservices
# ...

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: microservices
# ...

---
# Payment Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
  namespace: microservices
# ...

---
apiVersion: v1
kind: Service
metadata:
  name: payment-service
  namespace: microservices
# ...
```

---

## Splitting Large Files

If your combined YAML becomes too large, split by:

### By Type:
```
project/
├── 01-namespace.yaml
├── 02-configmaps.yaml
├── 03-secrets.yaml
├── 04-deployments.yaml
├── 05-services.yaml
└── 06-ingress.yaml
```

### By Component:
```
project/
├── database/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── pvc.yaml
├── backend/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
└── frontend/
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

Deploy:
```bash
# Apply entire directory
kubectl apply -f project/

# Or apply in order
kubectl apply -f project/01-namespace.yaml
kubectl apply -f project/02-configmaps.yaml
# ... etc
```

---

## Tools for Managing Combined YAMLs

### 1. Kustomize (built into kubectl)
```bash
kubectl apply -k ./overlay/production/
```

### 2. Helm (package manager)
```bash
helm install my-app ./my-app-chart
```

### 3. Skaffold (development workflow)
```bash
skaffold dev
```

### 4. ArgoCD (GitOps)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    path: k8s/
    repoURL: https://github.com/myorg/myapp
```

---

## Real-World Example: WordPress + MySQL

**File: `wordpress-stack.yaml`**

```yaml
# MySQL Secret
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # password123

---
# MySQL PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
# MySQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: wordpress
    tier: database
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
      tier: database
  template:
    metadata:
      labels:
        app: wordpress
        tier: database
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc

---
# MySQL Service
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: wordpress
    tier: database
spec:
  type: ClusterIP
  selector:
    app: wordpress
    tier: database
  ports:
  - port: 3306
    targetPort: 3306

---
# WordPress PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

---
# WordPress Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - name: wordpress
        image: wordpress:6.4
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-service:3306
        - name: WORDPRESS_DB_NAME
          value: wordpress
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 80
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
# WordPress Service
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
  labels:
    app: wordpress
    tier: frontend
spec:
  type: LoadBalancer
  selector:
    app: wordpress
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
```

**Deploy:**
```bash
kubectl apply -f wordpress-stack.yaml
kubectl get all -l app=wordpress
kubectl get service wordpress-service  # Get LoadBalancer IP
```

---

## Summary

### Key Points:
1. Use `---` to separate resources in single file
2. Order matters: namespace → config → storage → workloads → services → ingress
3. Balance convenience vs maintainability
4. Use tools like Kustomize or Helm for complex scenarios
5. Keep related resources together
6. Add comments for clarity
7. Use consistent labeling

### When to Use Single File:
- ✅ Small to medium applications (< 500 lines)
- ✅ Related components (app + database)
- ✅ Simple deployments
- ✅ Quick testing/development

### When to Split Files:
- ✅ Large applications (> 500 lines)
- ✅ Multiple environments (dev/staging/prod)
- ✅ Many microservices
- ✅ Complex configurations
- ✅ Team collaboration (easier to review changes)

---

**For more information:**
- Kubernetes Documentation: https://kubernetes.io/docs/
- Kustomize: https://kustomize.io/
- Helm: https://helm.sh/
