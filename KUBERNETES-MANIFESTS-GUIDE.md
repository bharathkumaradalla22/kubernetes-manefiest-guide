# Kubernetes Manifests Complete Guide

This guide provides comprehensive explanations for all Kubernetes manifest files, including what they are, why we need them, and how to use them.

---

## Table of Contents
1. [Basic Workload Resources](#basic-workload-resources)
2. [Service Resources](#service-resources)
3. [Configuration Resources](#configuration-resources)
4. [Storage Resources](#storage-resources)
5. [Security & Access Control](#security--access-control)
6. [Networking Resources](#networking-resources)
7. [Scheduling & Resource Management](#scheduling--resource-management)
8. [Advanced Resources](#advanced-resources)

---

## Basic Workload Resources

### 1. Pod (`pod.yaml`)

**Definition:**
A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster and can contain one or more containers.

**What it is:**
- A wrapper around one or more containers
- Shares network namespace (all containers share the same IP)
- Shares storage volumes
- Has a unique IP address within the cluster

**Why we need it:**
- Fundamental building block of Kubernetes applications
- Enables containers to work together as a cohesive unit
- Provides resource limits and requests
- Manages container lifecycle

**How to apply:**
```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl delete -f pod.yaml
```

**When to use:**
- Testing and development
- Running one-off tasks
- Usually managed by higher-level controllers (Deployments, StatefulSets)

---

### 2. Deployment (`deployment.yaml`)

**Definition:**
A Deployment provides declarative updates for Pods and ReplicaSets. It manages the desired state of your application.

**What it is:**
- Higher-level controller that manages ReplicaSets
- Ensures specified number of pod replicas are running
- Provides rolling updates and rollbacks
- Self-healing mechanism

**Why we need it:**
- **Zero-downtime deployments**: Rolling updates without service interruption
- **Scaling**: Easy horizontal scaling (increase/decrease replicas)
- **Self-healing**: Automatically replaces failed pods
- **Version control**: Easy rollback to previous versions
- **Declarative management**: Describe desired state, Kubernetes maintains it

**How to apply:**
```bash
# Create deployment
kubectl apply -f deployment.yaml

# Check deployment status
kubectl get deployments
kubectl rollout status deployment/nginx-deployment

# Scale deployment
kubectl scale deployment/nginx-deployment --replicas=5

# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Delete deployment
kubectl delete -f deployment.yaml
```

**Best practices:**
- Always use Deployments instead of bare Pods in production
- Set resource requests and limits
- Configure liveness and readiness probes
- Use rolling update strategy for zero-downtime deployments

---

### 3. ReplicaSet (`replicaset.yaml`)

**Definition:**
A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

**What it is:**
- Maintains a stable set of replica Pods
- Usually created and managed by Deployments
- Ensures pod availability

**Why we need it:**
- **High availability**: Maintains desired number of pods
- **Load distribution**: Multiple replicas distribute workload
- **Fault tolerance**: Automatically replaces failed pods

**How to apply:**
```bash
kubectl apply -f replicaset.yaml
kubectl get replicasets
kubectl describe replicaset nginx-replicaset
kubectl scale replicaset nginx-replicaset --replicas=5
kubectl delete -f replicaset.yaml
```

**Note:** In practice, you should use Deployments instead of directly creating ReplicaSets.

---

### 4. StatefulSet (`statefulset.yaml`)

**Definition:**
StatefulSet is used for stateful applications that require stable network identities and persistent storage.

**What it is:**
- Manages deployment of stateful applications
- Provides stable, unique network identifiers
- Provides stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

**Why we need it:**
- **Stable network identity**: Each pod gets a predictable name (mysql-0, mysql-1, mysql-2)
- **Persistent storage**: Each pod gets its own PersistentVolumeClaim
- **Ordered operations**: Pods are created, updated, and deleted in order
- **Stateful applications**: Databases, caching systems, message queues

**Use cases:**
- Databases (MySQL, PostgreSQL, MongoDB)
- Distributed systems (Kafka, Elasticsearch, Cassandra)
- Applications requiring stable storage and identity

**How to apply:**
```bash
kubectl apply -f statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=mysql
kubectl describe statefulset mysql-statefulset

# Scale statefulset
kubectl scale statefulset mysql-statefulset --replicas=5

# Delete (pods deleted in reverse order)
kubectl delete -f statefulset.yaml
```

---

### 5. DaemonSet (`daemonset.yaml`)

**Definition:**
A DaemonSet ensures that all (or some) nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them.

**What it is:**
- Runs one pod per node
- Automatically adds pods to new nodes
- Removes pods when nodes are removed

**Why we need it:**
- **Node-level services**: Services that need to run on every node
- **Log collection**: Running log collectors on all nodes
- **Monitoring agents**: Node monitoring on every node
- **Storage daemons**: Local storage management
- **Network plugins**: Network configuration on all nodes

**Use cases:**
- Log aggregators (Fluentd, Logstash)
- Monitoring agents (Prometheus Node Exporter, Datadog agent)
- Storage daemons (GlusterFS, Ceph)
- Network proxies (kube-proxy)

**How to apply:**
```bash
kubectl apply -f daemonset.yaml
kubectl get daemonsets
kubectl describe daemonset fluentd-daemonset
kubectl get pods -l app=fluentd -o wide
kubectl delete -f daemonset.yaml
```

---

### 6. Job (`job.yaml`)

**Definition:**
A Job creates one or more Pods and ensures that a specified number of them successfully terminate.

**What it is:**
- Runs pods to completion
- Tracks successful completions
- Can run pods in parallel
- Automatically cleans up completed pods

**Why we need it:**
- **Batch processing**: One-time tasks that must complete
- **Data processing**: ETL jobs, backups, reports
- **Reliability**: Retries on failure
- **Parallelism**: Run multiple pods simultaneously

**Use cases:**
- Database backups
- Data migrations
- Batch processing jobs
- Report generation
- Image processing

**How to apply:**
```bash
kubectl apply -f job.yaml
kubectl get jobs
kubectl describe job backup-job
kubectl logs job/backup-job
kubectl get pods --selector=job-name=backup-job
kubectl delete -f job.yaml
```

---

### 7. CronJob (`cronjob.yaml`)

**Definition:**
A CronJob creates Jobs on a repeating schedule, similar to Unix cron.

**What it is:**
- Scheduled job execution
- Uses cron syntax for scheduling
- Creates Jobs at specified times
- Manages job history

**Why we need it:**
- **Scheduled tasks**: Periodic backups, reports, cleanups
- **Automation**: Regular maintenance tasks
- **Time-based triggers**: Run jobs at specific times

**Use cases:**
- Daily database backups
- Hourly data synchronization
- Weekly report generation
- Monthly cleanup tasks
- Certificate renewal

**Cron schedule format:**
```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
# │ │ │ │ │
# * * * * *

Examples:
0 2 * * *     - Daily at 2 AM
*/5 * * * *   - Every 5 minutes
0 */4 * * *   - Every 4 hours
0 0 * * 0     - Weekly on Sunday midnight
0 0 1 * *     - Monthly on 1st at midnight
```

**How to apply:**
```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl describe cronjob backup-cronjob
kubectl get jobs --watch  # Watch jobs being created
kubectl delete -f cronjob.yaml
```

---

## Service Resources

### 8. Service - ClusterIP (`service-clusterip.yaml`)

**Definition:**
ClusterIP service exposes the application on an internal IP in the cluster. This is the default service type.

**What it is:**
- Internal load balancer
- Only accessible within the cluster
- Provides stable IP and DNS name
- Load balances traffic across pods

**Why we need it:**
- **Internal communication**: Services communicate with each other
- **Stable endpoint**: Pods come and go, service IP remains constant
- **Load balancing**: Distributes traffic to healthy pods
- **Service discovery**: DNS-based discovery (service-name.namespace.svc.cluster.local)

**How to apply:**
```bash
kubectl apply -f service-clusterip.yaml
kubectl get services
kubectl describe service nginx-service-clusterip

# Test from within cluster
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://nginx-service-clusterip

kubectl delete -f service-clusterip.yaml
```

---

### 9. Service - NodePort (`service-nodeport.yaml`)

**Definition:**
NodePort service exposes the application on each Node's IP at a static port (30000-32767).

**What it is:**
- Exposes service on each node's IP
- Allocates a port from range 30000-32767
- External traffic can reach service via NodeIP:NodePort
- Also creates ClusterIP automatically

**Why we need it:**
- **External access**: Access services from outside cluster
- **Development/Testing**: Quick external access without load balancer
- **Simple external exposure**: No cloud provider required

**How to apply:**
```bash
kubectl apply -f service-nodeport.yaml
kubectl get services
kubectl describe service nginx-service-nodeport

# Access from outside cluster
curl http://<node-ip>:30080

kubectl delete -f service-nodeport.yaml
```

**Note:** For production, prefer LoadBalancer or Ingress.

---

### 10. Service - LoadBalancer (`service-loadbalancer.yaml`)

**Definition:**
LoadBalancer service exposes the application externally using a cloud provider's load balancer.

**What it is:**
- Creates external load balancer (AWS ELB, GCP Load Balancer, Azure Load Balancer)
- Automatically creates NodePort and ClusterIP
- Provides external IP address
- Cloud provider dependent

**Why we need it:**
- **Production external access**: Proper load balancing for external traffic
- **High availability**: Cloud load balancers provide HA
- **SSL termination**: Some support SSL/TLS termination
- **Health checks**: Built-in health checking

**How to apply:**
```bash
kubectl apply -f service-loadbalancer.yaml
kubectl get services

# Wait for EXTERNAL-IP to be assigned
kubectl get service nginx-service-loadbalancer --watch

# Access via external IP
curl http://<external-ip>

kubectl delete -f service-loadbalancer.yaml
```

**Note:** Requires cloud provider support. Each LoadBalancer service gets its own IP (can be costly).

---

## Configuration Resources

### 11. ConfigMap (`configmap.yaml`)

**Definition:**
ConfigMap is an API object used to store non-confidential configuration data in key-value pairs.

**What it is:**
- Stores configuration data
- Decouples configuration from container images
- Can store files or key-value pairs
- Can be consumed as environment variables, command-line arguments, or config files

**Why we need it:**
- **Configuration management**: Separate config from code
- **Environment-specific settings**: Different configs for dev/staging/prod
- **No image rebuilds**: Change config without rebuilding images
- **Centralized configuration**: Manage configuration in one place

**Use cases:**
- Application settings (database URLs, API endpoints)
- Configuration files (nginx.conf, app.properties)
- Feature flags
- Environment variables

**How to apply:**
```bash
# Create from file
kubectl apply -f configmap.yaml

# Create from literal values
kubectl create configmap app-config --from-literal=key1=value1

# Create from file
kubectl create configmap app-config --from-file=config.properties

# View configmap
kubectl get configmaps
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Use in pod
kubectl apply -f pod.yaml  # Pod references ConfigMap

kubectl delete -f configmap.yaml
```

**Consumption methods:**
```yaml
# As environment variable
env:
- name: DATABASE_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: database_host

# As volume mount
volumes:
- name: config
  configMap:
    name: app-config
volumeMounts:
- name: config
  mountPath: /etc/config
```

---

### 12. Secret (`secret.yaml`)

**Definition:**
Secret is similar to ConfigMap but specifically designed to hold confidential data like passwords, tokens, and keys.

**What it is:**
- Stores sensitive information
- Base64 encoded (not encrypted by default)
- Can be encrypted at rest with proper cluster configuration
- Limited to 1MB in size

**Types of Secrets:**
- **Opaque**: Generic secret (default)
- **kubernetes.io/tls**: TLS certificate and key
- **kubernetes.io/dockerconfigjson**: Docker registry credentials
- **kubernetes.io/service-account-token**: Service account token
- **kubernetes.io/ssh-auth**: SSH authentication

**Why we need it:**
- **Security**: Keep sensitive data separate from application code
- **Access control**: RBAC can control who can read secrets
- **Reduced risk**: Secrets not exposed in image or config
- **Encryption**: Can be encrypted at rest

**How to apply:**
```bash
# Create from file
kubectl apply -f secret.yaml

# Create from literal
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/cert.key

# Create Docker registry secret
kubectl create secret docker-registry docker-secret \
  --docker-server=docker.io \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# View secrets (values are hidden)
kubectl get secrets
kubectl describe secret app-secret

# Decode secret
kubectl get secret app-secret -o jsonpath='{.data.username}' | base64 --decode

kubectl delete -f secret.yaml
```

**Usage in Pods:**
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
```

**Security best practices:**
- Enable encryption at rest
- Use RBAC to limit access
- Use external secret management (Vault, AWS Secrets Manager)
- Rotate secrets regularly
- Never commit secrets to version control

---

### 13. Namespace (`namespace.yaml`)

**Definition:**
Namespace provides a mechanism for isolating groups of resources within a single cluster.

**What it is:**
- Virtual clusters within a physical cluster
- Logical isolation of resources
- Scope for names (names must be unique within a namespace)
- Resource quotas can be applied per namespace

**Why we need it:**
- **Multi-tenancy**: Separate teams or projects
- **Environment separation**: dev, staging, production in same cluster
- **Resource isolation**: Apply quotas and limits per namespace
- **Access control**: RBAC policies per namespace
- **Organization**: Logical grouping of resources

**Default namespaces:**
- `default`: Default namespace for objects without namespace
- `kube-system`: System components
- `kube-public`: Publicly readable (even unauthenticated)
- `kube-node-lease`: Node heartbeat information

**How to apply:**
```bash
kubectl apply -f namespace.yaml
kubectl get namespaces
kubectl describe namespace development

# Create resources in namespace
kubectl apply -f deployment.yaml -n development

# View resources in namespace
kubectl get all -n development

# Set default namespace for kubectl
kubectl config set-context --current --namespace=development

# Delete namespace (deletes all resources in it)
kubectl delete -f namespace.yaml
```

**Best practices:**
- Use namespaces for environment separation
- Apply ResourceQuotas to prevent resource exhaustion
- Use NetworkPolicies for network isolation
- Name namespaces clearly (team-name, environment-name)

---

## Storage Resources

### 14. PersistentVolume (PV) (`persistentvolume.yaml`)

**Definition:**
A PersistentVolume is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

**What it is:**
- Cluster-level resource (not namespaced)
- Represents physical storage (NFS, iSCSI, cloud disks)
- Independent lifecycle from pods
- Can be statically or dynamically provisioned

**Storage types:**
- HostPath (local storage - testing only)
- NFS (Network File System)
- Cloud disks (AWS EBS, GCP Persistent Disk, Azure Disk)
- Ceph, GlusterFS
- iSCSI

**Reclaim policies:**
- **Retain**: Manual reclamation after release
- **Recycle**: Basic scrub (rm -rf /volume/*)
- **Delete**: Delete underlying storage asset

**Why we need it:**
- **Persistent data**: Data survives pod restarts and rescheduling
- **Storage abstraction**: Abstract storage details from applications
- **Storage management**: Centralized storage provisioning

**How to apply:**
```bash
kubectl apply -f persistentvolume.yaml
kubectl get persistentvolumes
kubectl get pv
kubectl describe pv pv-local

# Check status
kubectl get pv pv-local -o jsonpath='{.status.phase}'

kubectl delete -f persistentvolume.yaml
```

**PV States:**
- **Available**: Free and not yet bound
- **Bound**: Bound to a PVC
- **Released**: PVC deleted but resource not reclaimed
- **Failed**: Automatic reclamation failed

---

### 15. PersistentVolumeClaim (PVC) (`persistentvolumeclaim.yaml`)

**Definition:**
A PersistentVolumeClaim is a request for storage by a user. It's similar to a Pod - Pods consume node resources and PVCs consume PV resources.

**What it is:**
- Namespace-scoped resource
- Request for storage with specific size and access mode
- Binds to a PersistentVolume
- Used by Pods to access storage

**Access Modes:**
- **ReadWriteOnce (RWO)**: Mounted read-write by a single node
- **ReadOnlyMany (ROX)**: Mounted read-only by many nodes
- **ReadWriteMany (RWX)**: Mounted read-write by many nodes

**Why we need it:**
- **Storage abstraction**: Users request storage without knowing details
- **Dynamic provisioning**: Automatically provision storage
- **Pod portability**: Pods reference PVCs, not specific volumes

**How to apply:**
```bash
kubectl apply -f persistentvolumeclaim.yaml
kubectl get persistentvolumeclaims
kubectl get pvc
kubectl describe pvc pvc-local

# Check if bound
kubectl get pvc pvc-local -o jsonpath='{.status.phase}'

# Use in pod
kubectl apply -f pod.yaml  # Pod uses PVC

kubectl delete -f persistentvolumeclaim.yaml
```

**Usage in Pod:**
```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: pvc-local
containers:
- volumeMounts:
  - name: data
    mountPath: /data
```

---

### 16. StorageClass (`storageclass.yaml`)

**Definition:**
StorageClass provides a way to describe different "classes" of storage with different quality-of-service levels, backup policies, or other policies.

**What it is:**
- Defines storage provisioner
- Enables dynamic volume provisioning
- Specifies storage parameters (type, IOPS, etc.)
- Can set reclaim policy and binding mode

**Provisioners:**
- AWS: `kubernetes.io/aws-ebs`
- GCP: `kubernetes.io/gce-pd`
- Azure: `kubernetes.io/azure-disk`
- NFS: Custom provisioners
- Ceph RBD: `kubernetes.io/rbd`

**Why we need it:**
- **Dynamic provisioning**: Automatic PV creation
- **Storage tiers**: Different performance levels (SSD, HDD)
- **Policy management**: Different backup and retention policies
- **On-demand storage**: Storage created when needed

**How to apply:**
```bash
kubectl apply -f storageclass.yaml
kubectl get storageclasses
kubectl get sc
kubectl describe sc fast-ssd

# Mark as default
kubectl patch storageclass fast-ssd \
  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# PVC automatically uses StorageClass
kubectl apply -f persistentvolumeclaim.yaml

kubectl delete -f storageclass.yaml
```

---

### 17. VolumeSnapshot (`volumesnapshot.yaml`)

**Definition:**
VolumeSnapshot provides a standardized way to copy a volume's contents at a particular point in time without creating a new volume.

**What it is:**
- Point-in-time copy of a volume
- Can restore data from snapshots
- Requires CSI driver support
- Similar to PV/PVC model

**Why we need it:**
- **Backup**: Create backups of persistent data
- **Disaster recovery**: Restore from snapshots
- **Testing**: Clone production data for testing
- **Data migration**: Move data between clusters

**How to apply:**
```bash
kubectl apply -f volumesnapshot.yaml
kubectl get volumesnapshots
kubectl describe volumesnapshot mysql-snapshot

# Restore from snapshot
kubectl apply -f restore-from-snapshot.yaml

kubectl delete -f volumesnapshot.yaml
```

---

## Security & Access Control

### 18. ServiceAccount (`serviceaccount.yaml`)

**Definition:**
ServiceAccount provides an identity for processes that run in a Pod to authenticate with the API server.

**What it is:**
- Identity for pods
- Authenticates with API server
- Can be assigned RBAC permissions
- Automatically mounted in pods

**Why we need it:**
- **Pod identity**: Pods need identity to access API server
- **RBAC**: Assign specific permissions to pods
- **Security**: Principle of least privilege
- **Service-to-service auth**: Applications authenticate as service accounts

**How to apply:**
```bash
kubectl apply -f serviceaccount.yaml
kubectl get serviceaccounts
kubectl get sa
kubectl describe sa app-serviceaccount

# Use in pod
kubectl apply -f pod-with-sa.yaml

# Get token
kubectl create token app-serviceaccount

kubectl delete -f serviceaccount.yaml
```

**Usage in Pod:**
```yaml
spec:
  serviceAccountName: app-serviceaccount
  automountServiceAccountToken: true
```

---

### 19. Role (`role.yaml`)

**Definition:**
Role contains rules that represent a set of permissions within a specific namespace. Permissions are purely additive (no deny rules).

**What it is:**
- Namespace-scoped permissions
- Defines what actions can be performed
- Contains list of API groups, resources, and verbs
- Used with RoleBinding

**Verbs (actions):**
- get, list, watch (read operations)
- create, update, patch (write operations)
- delete, deletecollection (delete operations)
- * (all verbs)

**Why we need it:**
- **Least privilege**: Grant only necessary permissions
- **Security**: Control what users/services can do
- **Namespace isolation**: Permissions limited to namespace

**How to apply:**
```bash
kubectl apply -f role.yaml
kubectl get roles
kubectl describe role pod-reader

# View permissions
kubectl auth can-i list pods --as=system:serviceaccount:default:app-serviceaccount

kubectl delete -f role.yaml
```

---

### 20. RoleBinding (`rolebinding.yaml`)

**Definition:**
RoleBinding grants the permissions defined in a Role to a user, group, or ServiceAccount within a specific namespace.

**What it is:**
- Links Role to subjects (users, groups, service accounts)
- Namespace-scoped
- Grants permissions within namespace

**Subjects:**
- User: Human users
- Group: Groups of users
- ServiceAccount: Pod identities

**Why we need it:**
- **Access control**: Assign permissions to entities
- **RBAC implementation**: Connect roles to subjects
- **Security**: Control who can do what

**How to apply:**
```bash
kubectl apply -f rolebinding.yaml
kubectl get rolebindings
kubectl describe rolebinding read-pods-binding

# Test permissions
kubectl auth can-i get pods --as=jane

kubectl delete -f rolebinding.yaml
```

---

### 21. ClusterRole (`clusterrole.yaml`)

**Definition:**
ClusterRole is like Role but cluster-wide. It can grant permissions to cluster-scoped resources or be used across all namespaces.

**What it is:**
- Cluster-wide permissions (not namespace-scoped)
- Can access cluster resources (nodes, persistentvolumes)
- Can be used across all namespaces
- More powerful than Role

**Use cases:**
- Cluster-scoped resources (nodes, namespaces, PVs)
- Non-resource endpoints (/healthz, /metrics)
- Aggregated permissions across namespaces

**Why we need it:**
- **Cluster administration**: Manage cluster resources
- **Cross-namespace access**: Access resources in all namespaces
- **Node management**: Manage cluster nodes
- **Global permissions**: Apply permissions cluster-wide

**How to apply:**
```bash
kubectl apply -f clusterrole.yaml
kubectl get clusterroles
kubectl describe clusterrole cluster-admin-read

# View built-in cluster roles
kubectl get clusterroles

kubectl delete -f clusterrole.yaml
```

---

### 22. ClusterRoleBinding (`clusterrolebinding.yaml`)

**Definition:**
ClusterRoleBinding grants the permissions defined in a ClusterRole to subjects cluster-wide.

**What it is:**
- Links ClusterRole to subjects
- Cluster-scoped (applies everywhere)
- Grants cluster-wide or cross-namespace permissions

**Why we need it:**
- **Cluster administration**: Grant admin permissions
- **Monitoring**: Allow monitoring tools to access all namespaces
- **Service accounts**: Global service account permissions

**How to apply:**
```bash
kubectl apply -f clusterrolebinding.yaml
kubectl get clusterrolebindings
kubectl describe clusterrolebinding read-all-binding

# Test cluster-wide permissions
kubectl auth can-i get nodes --as=viewer

kubectl delete -f clusterrolebinding.yaml
```

---

### 23. PodSecurityPolicy (`podsecuritypolicy.yaml`)

**Definition:**
PodSecurityPolicy (deprecated in 1.25, removed in 1.25+) controls security-sensitive aspects of pod specification.

**What it is:**
- Cluster-level resource
- Controls pod security settings
- Validates pod security configuration
- Enforces security best practices

**Note:** PodSecurityPolicy is deprecated. Use Pod Security Standards (PSS) and Pod Security Admission instead.

**Controls:**
- Privileged containers
- Host namespaces (network, PID, IPC)
- Volume types
- User/group IDs
- Capabilities
- SELinux, AppArmor, Seccomp

**Why we needed it:**
- **Security enforcement**: Prevent insecure pod configurations
- **Compliance**: Meet security standards
- **Multi-tenancy**: Prevent privilege escalation

**How to apply (deprecated):**
```bash
kubectl apply -f podsecuritypolicy.yaml
kubectl get psp
kubectl describe psp restricted-psp

# Modern alternative: Pod Security Standards
kubectl label namespace default pod-security.kubernetes.io/enforce=restricted
```

---

## Networking Resources

### 24. Ingress (`ingress.yaml`)

**Definition:**
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

**What it is:**
- Layer 7 (HTTP/HTTPS) load balancer
- Routes traffic based on host and path
- SSL/TLS termination
- Requires Ingress Controller (nginx, traefik, etc.)

**Why we need it:**
- **Single entry point**: One load balancer for multiple services
- **Path-based routing**: /api → api-service, /web → web-service
- **Host-based routing**: api.example.com → api-service
- **SSL/TLS**: Centralized certificate management
- **Cost-effective**: One load balancer instead of many

**Features:**
- Host-based routing
- Path-based routing
- SSL/TLS termination
- URL rewriting
- Redirects

**How to apply:**
```bash
# Install Ingress Controller first (nginx example)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Apply ingress
kubectl apply -f ingress.yaml
kubectl get ingress
kubectl describe ingress app-ingress

# Get ingress IP/hostname
kubectl get ingress app-ingress -o jsonpath='{.status.loadBalancer.ingress[0]}'

# Test
curl -H "Host: example.com" http://<ingress-ip>/

kubectl delete -f ingress.yaml
```

---

### 25. IngressClass (`ingressclass.yaml`)

**Definition:**
IngressClass represents the class of the Ingress, which specifies which Ingress Controller should handle the Ingress.

**What it is:**
- References Ingress Controller
- Multiple controllers can coexist
- Default IngressClass can be set
- Ingress selects IngressClass

**Why we need it:**
- **Multiple controllers**: Use different controllers for different ingresses
- **Controller selection**: Specify which controller handles ingress
- **Flexibility**: Switch between controllers easily

**How to apply:**
```bash
kubectl apply -f ingressclass.yaml
kubectl get ingressclasses
kubectl get ingressclass nginx

# Mark as default
kubectl annotate ingressclass nginx ingressclass.kubernetes.io/is-default-class="true"

kubectl delete -f ingressclass.yaml
```

---

### 26. NetworkPolicy (`networkpolicy.yaml`)

**Definition:**
NetworkPolicy specifies how groups of pods are allowed to communicate with each other and other network endpoints.

**What it is:**
- Firewall rules for pods
- Controls ingress (incoming) and egress (outgoing) traffic
- Pod-level network segmentation
- Requires CNI plugin support (Calico, Cilium, etc.)

**Why we need it:**
- **Security**: Prevent unauthorized network access
- **Micro-segmentation**: Isolate workloads
- **Compliance**: Meet network security requirements
- **Defense in depth**: Additional security layer

**Policy types:**
- **Ingress**: Control incoming traffic
- **Egress**: Control outgoing traffic

**Selectors:**
- podSelector: Select pods in same namespace
- namespaceSelector: Select namespaces
- ipBlock: CIDR ranges

**How to apply:**
```bash
# Requires CNI plugin with NetworkPolicy support

kubectl apply -f networkpolicy.yaml
kubectl get networkpolicies
kubectl get netpol
kubectl describe networkpolicy deny-all

# Test connectivity
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://backend-service

kubectl delete -f networkpolicy.yaml
```

**Common patterns:**
```yaml
# Deny all ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# Allow from specific namespace
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        name: frontend
```

---

### 27. Endpoint & EndpointSlice (`endpoint.yaml`, `endpointslice.yaml`)

**Definition:**
Endpoints and EndpointSlices track the IP addresses of the pods that back a service.

**What they are:**
- Automatically created by Services
- List of pod IPs backing a service
- EndpointSlice is newer, more scalable version
- Can be manually created for external services

**Why we need them:**
- **Service routing**: Services route to these IPs
- **External services**: Connect to external databases
- **Service mesh**: Integration with service meshes
- **Scalability**: EndpointSlices scale better (1000+ endpoints)

**How to apply:**
```bash
# Usually auto-created, but can create manually for external services
kubectl apply -f endpoint.yaml
kubectl get endpoints
kubectl describe endpoints external-database

# EndpointSlices (newer)
kubectl get endpointslices
kubectl describe endpointslice external-api-slice

kubectl delete -f endpoint.yaml
```

---

## Scheduling & Resource Management

### 28. ResourceQuota (`resourcequota.yaml`)

**Definition:**
ResourceQuota provides constraints that limit aggregate resource consumption per namespace.

**What it is:**
- Namespace-level resource limits
- Limits compute resources (CPU, memory)
- Limits object counts (pods, services, etc.)
- Limits storage requests

**Quota types:**
- **Compute**: requests.cpu, requests.memory, limits.cpu, limits.memory
- **Storage**: requests.storage, persistentvolumeclaims
- **Object count**: pods, services, configmaps, secrets

**Why we need it:**
- **Multi-tenancy**: Prevent resource monopolization
- **Cost control**: Limit resource usage
- **Cluster stability**: Prevent resource exhaustion
- **Fair allocation**: Ensure fair resource distribution

**How to apply:**
```bash
kubectl apply -f resourcequota.yaml
kubectl get resourcequota -n development
kubectl describe resourcequota compute-quota -n development

# View quota usage
kubectl get resourcequota -n development -o yaml

kubectl delete -f resourcequota.yaml
```

**Important:** Pods must specify resource requests/limits when ResourceQuota is active.

---

### 29. LimitRange (`limitrange.yaml`)

**Definition:**
LimitRange sets default resource requests/limits and enforces min/max constraints on resources per pod or container.

**What it is:**
- Namespace-level resource constraints
- Sets default requests and limits
- Enforces minimum and maximum values
- Applies to containers, pods, PVCs

**Why we need it:**
- **Resource management**: Ensure all containers have limits
- **Prevent abuse**: Limit maximum resource usage
- **Defaults**: Auto-assign limits to containers without them
- **Quality of Service**: Ensure minimum resource guarantees

**Applies to:**
- Container
- Pod
- PersistentVolumeClaim

**How to apply:**
```bash
kubectl apply -f limitrange.yaml
kubectl get limitranges -n development
kubectl describe limitrange cpu-memory-limit-range -n development

# Containers without limits will get defaults
kubectl apply -f pod-without-limits.yaml -n development

kubectl delete -f limitrange.yaml
```

---

### 30. PriorityClass (`priorityclass.yaml`)

**Definition:**
PriorityClass defines the relative importance of pods. Higher priority pods can preempt (evict) lower priority pods when resources are scarce.

**What it is:**
- Cluster-level resource
- Numeric priority value (higher = more important)
- Used for pod scheduling
- Enables pod preemption

**Why we need it:**
- **Critical workloads**: Ensure critical pods run first
- **Resource contention**: Prioritize important services
- **Cost optimization**: Lower priority for batch jobs
- **Preemption**: Evict low priority pods for high priority

**Default priorities:**
- system-cluster-critical: 2000000000
- system-node-critical: 2000001000

**How to apply:**
```bash
kubectl apply -f priorityclass.yaml
kubectl get priorityclasses
kubectl get pc
kubectl describe pc high-priority

# Use in pod
kubectl apply -f pod-with-priority.yaml

kubectl delete -f priorityclass.yaml
```

**Pod usage:**
```yaml
spec:
  priorityClassName: high-priority
```

---

### 31. HorizontalPodAutoscaler (HPA) (`horizontalpodautoscaler.yaml`)

**Definition:**
HorizontalPodAutoscaler automatically scales the number of pods based on observed CPU utilization, memory, or custom metrics.

**What it is:**
- Automatic horizontal scaling
- Scales Deployment, ReplicaSet, or StatefulSet
- Based on metrics (CPU, memory, custom)
- Adjusts replica count

**Why we need it:**
- **Auto-scaling**: Automatically handle load changes
- **Cost optimization**: Scale down during low traffic
- **Performance**: Scale up during high traffic
- **Reliability**: Maintain performance SLAs

**Metrics types:**
- **Resource**: CPU, memory utilization
- **Pods**: Custom per-pod metrics
- **Object**: Metrics from other objects
- **External**: Metrics from external systems

**How to apply:**
```bash
# Requires metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl apply -f horizontalpodautoscaler.yaml
kubectl get hpa
kubectl describe hpa nginx-hpa

# Watch autoscaling
kubectl get hpa --watch

# Generate load to test
kubectl run load-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nginx-service; done"

kubectl delete -f horizontalpodautoscaler.yaml
```

---

### 32. VerticalPodAutoscaler (VPA) (`verticalpodautoscaler.yaml`)

**Definition:**
VerticalPodAutoscaler automatically adjusts CPU and memory requests/limits based on usage.

**What it is:**
- Vertical scaling (resource adjustment)
- Analyzes resource usage patterns
- Recommends or auto-applies resource changes
- Requires VPA controller installation

**Update modes:**
- **Off**: Only provide recommendations
- **Initial**: Set resources on pod creation
- **Recreate**: Update running pods (restarts pods)
- **Auto**: Automatically update resources

**Why we need it:**
- **Right-sizing**: Optimize resource allocation
- **Cost savings**: Remove over-provisioning
- **Performance**: Ensure adequate resources
- **Automation**: No manual resource tuning

**How to apply:**
```bash
# Install VPA
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

kubectl apply -f verticalpodautoscaler.yaml
kubectl get vpa
kubectl describe vpa nginx-vpa

# View recommendations
kubectl get vpa nginx-vpa --output yaml

kubectl delete -f verticalpodautoscaler.yaml
```

---

### 33. PodDisruptionBudget (PDB) (`poddisruptionbudget.yaml`)

**Definition:**
PodDisruptionBudget limits the number of pods that can be down simultaneously due to voluntary disruptions.

**What it is:**
- Availability guarantees during disruptions
- Protects against voluntary disruptions (node drain, updates)
- Specifies minimum available or maximum unavailable pods
- Does NOT protect against involuntary disruptions

**Disruption types:**
- **Voluntary**: Node drain, deployment updates, kubectl delete
- **Involuntary**: Hardware failure, kernel panic, node disappears

**Why we need it:**
- **High availability**: Ensure minimum replicas during updates
- **Safe deployments**: Prevent too many pods down at once
- **Cluster operations**: Safe node maintenance
- **Application SLA**: Maintain service availability

**How to apply:**
```bash
kubectl apply -f poddisruptionbudget.yaml
kubectl get poddisruptionbudgets
kubectl get pdb
kubectl describe pdb nginx-pdb

# Check allowed disruptions
kubectl get pdb nginx-pdb -o jsonpath='{.status.disruptionsAllowed}'

# Test during node drain
kubectl drain <node-name> --ignore-daemonsets

kubectl delete -f poddisruptionbudget.yaml
```

**Spec options:**
- `minAvailable: 2` - At least 2 pods must be available
- `maxUnavailable: 1` - At most 1 pod can be unavailable

---

## Advanced Resources

### 34. CustomResourceDefinition (CRD) (`customresourcedefinition.yaml`)

**Definition:**
CustomResourceDefinition allows you to define custom resources and extend the Kubernetes API with your own resource types.

**What it is:**
- Extends Kubernetes API
- Defines new resource types
- Schema validation
- Custom controllers can watch CRDs

**Why we need it:**
- **Custom resources**: Define application-specific resources
- **Operators**: Build Kubernetes operators
- **Platform extension**: Extend Kubernetes functionality
- **Declarative APIs**: Create declarative APIs for apps

**Use cases:**
- Databases operators (MySQL, PostgreSQL)
- Certificate managers (cert-manager)
- Backup solutions
- Application-specific configurations

**How to apply:**
```bash
kubectl apply -f customresourcedefinition.yaml
kubectl get crds
kubectl describe crd applications.example.com

# Create custom resource instance
cat <<EOF | kubectl apply -f -
apiVersion: example.com/v1
kind: Application
metadata:
  name: my-app
spec:
  replicas: 3
  image: nginx:1.21
  port: 80
  environment: production
EOF

# View custom resources
kubectl get applications
kubectl describe application my-app

kubectl delete -f customresourcedefinition.yaml
```

---

### 35. MutatingWebhookConfiguration (`mutatingwebhookconfiguration.yaml`)

**Definition:**
MutatingWebhookConfiguration defines admission webhooks that can mutate (modify) objects before they are persisted.

**What it is:**
- Admission controller webhook
- Intercepts API requests
- Can modify objects before creation/update
- Runs before validation

**Why we need it:**
- **Auto-injection**: Inject sidecars, environment variables
- **Default values**: Add default values to resources
- **Policy enforcement**: Enforce organizational standards
- **Security**: Inject security contexts

**Use cases:**
- Sidecar injection (Istio, linkerd)
- Image pull secret injection
- Resource limit defaults
- Label/annotation injection

**How to apply:**
```bash
# Requires webhook service deployed first
kubectl apply -f mutatingwebhookconfiguration.yaml
kubectl get mutatingwebhookconfigurations
kubectl describe mutatingwebhookconfiguration pod-mutating-webhook

kubectl delete -f mutatingwebhookconfiguration.yaml
```

---

### 36. ValidatingWebhookConfiguration (`validatingwebhookconfiguration.yaml`)

**Definition:**
ValidatingWebhookConfiguration defines admission webhooks that validate (approve/reject) objects before they are persisted.

**What it is:**
- Admission controller webhook
- Validates objects against custom rules
- Can accept or reject requests
- Runs after mutation

**Why we need it:**
- **Policy enforcement**: Enforce organizational policies
- **Security**: Validate security requirements
- **Compliance**: Ensure compliance standards
- **Custom validation**: Business logic validation

**Use cases:**
- Require specific labels
- Enforce naming conventions
- Validate resource limits
- Security policy enforcement
- Image registry whitelist

**How to apply:**
```bash
# Requires webhook service deployed first
kubectl apply -f validatingwebhookconfiguration.yaml
kubectl get validatingwebhookconfigurations
kubectl describe validatingwebhookconfiguration pod-validating-webhook

kubectl delete -f validatingwebhookconfiguration.yaml
```

---

### 37. RuntimeClass (`runtimeclass.yaml`)

**Definition:**
RuntimeClass defines different container runtimes that can be used to run pods.

**What it is:**
- Selects container runtime
- Configures runtime-specific parameters
- Supports multiple runtimes in same cluster
- Scheduling constraints

**Runtimes:**
- **runc**: Default OCI runtime
- **gVisor**: Sandboxed runtime (runsc)
- **Kata Containers**: VM-based runtime
- **Firecracker**: MicroVM runtime

**Why we need it:**
- **Security**: Sandboxed runtimes for untrusted code
- **Isolation**: Stronger isolation with VMs
- **Performance**: Different runtimes for different workloads
- **Multi-tenancy**: Different isolation levels

**How to apply:**
```bash
kubectl apply -f runtimeclass.yaml
kubectl get runtimeclasses
kubectl get rc
kubectl describe runtimeclass gvisor

# Use in pod
kubectl apply -f pod-with-runtime.yaml

kubectl delete -f runtimeclass.yaml
```

**Pod usage:**
```yaml
spec:
  runtimeClassName: gvisor
```

---

### 38. Lease (`lease.yaml`)

**Definition:**
Lease provides a mechanism for distributed locks and leader election.

**What it is:**
- Distributed lock primitive
- Leader election mechanism
- Heartbeat mechanism
- Used internally by Kubernetes

**Why we need it:**
- **Leader election**: Only one instance of controller runs
- **Distributed coordination**: Coordinate distributed systems
- **Node heartbeats**: Track node health
- **High availability**: Ensure single active controller

**How to apply:**
```bash
kubectl apply -f lease.yaml
kubectl get leases
kubectl describe lease my-lease

# View kube-system leases (used internally)
kubectl get leases -n kube-system

kubectl delete -f lease.yaml
```

---

### 39. CertificateSigningRequest (CSR) (`certificatesigningrequest.yaml`)

**Definition:**
CertificateSigningRequest allows requesting signed certificates from the cluster's certificate authority.

**What it is:**
- Certificate request
- Cluster CA signs certificates
- Used for node and user certificates
- TLS certificate management

**Why we need it:**
- **User certificates**: Create certificates for users
- **Node certificates**: Certificate rotation for nodes
- **Service certificates**: TLS for services
- **PKI management**: Centralized certificate management

**How to apply:**
```bash
# Generate key and CSR
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=user/O=developers"

# Create CSR in Kubernetes
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user-csr
spec:
  request: $(cat user.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF

# Approve CSR
kubectl certificate approve user-csr

# Get certificate
kubectl get csr user-csr -o jsonpath='{.status.certificate}' | base64 -d > user.crt

kubectl delete csr user-csr
```

---

### 40. PodPreset (`podpreset.yaml`)

**Definition:**
PodPreset (deprecated) automatically injects configuration into pods at creation time based on label selectors.

**What it is:**
- Automatic pod configuration
- Injects volumes, environment variables
- Based on label selectors
- Applied at pod creation

**Note:** PodPreset is deprecated and removed in Kubernetes 1.20+. Use MutatingWebhooks instead.

**Why it was needed:**
- Auto-configure pods
- DRY principle (Don't Repeat Yourself)
- Consistent configuration

**Modern alternative:** Use MutatingWebhookConfiguration for similar functionality.

---

### 41. PodTemplate (`podtemplate.yaml`)

**Definition:**
PodTemplate describes a template for creating pods. Used internally by controllers.

**What it is:**
- Pod specification template
- Used by Deployment, ReplicaSet, etc.
- Not typically used directly by users
- Defines pod structure

**Why we need it:**
- **Reusable templates**: Define pod templates once
- **Controller integration**: Used by workload controllers
- **Consistency**: Ensure consistent pod configuration

**Note:** Usually embedded in Deployments, Jobs, etc., not created standalone.

---

## Common kubectl Commands

### Basic Operations
```bash
# Apply manifests
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/
kubectl apply -f <url>

# Get resources
kubectl get pods
kubectl get all
kubectl get <resource> -o wide
kubectl get <resource> -o yaml
kubectl get <resource> -o json

# Describe resources
kubectl describe <resource> <name>

# Delete resources
kubectl delete -f <file.yaml>
kubectl delete <resource> <name>

# Edit resources
kubectl edit <resource> <name>

# Logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs -f <pod-name>  # Follow logs
kubectl logs --previous <pod-name>  # Previous instance

# Execute commands in pod
kubectl exec <pod-name> -- <command>
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding
kubectl port-forward <pod-name> <local-port>:<pod-port>
kubectl port-forward service/<service-name> <local-port>:<service-port>

# Copy files
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

### Debugging
```bash
# Events
kubectl get events --sort-by='.lastTimestamp'

# Resource usage
kubectl top nodes
kubectl top pods

# API resources
kubectl api-resources
kubectl explain <resource>
kubectl explain <resource>.spec

# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>

# Contexts and namespaces
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
```

### Manifest Management
```bash
# Dry run
kubectl apply -f <file.yaml> --dry-run=client
kubectl apply -f <file.yaml> --dry-run=server

# Diff changes
kubectl diff -f <file.yaml>

# Validate without applying
kubectl apply -f <file.yaml> --dry-run=client -o yaml

# Generate manifests
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml > service.yaml
```

---

## Best Practices Summary

### 1. **Resource Management**
- Always set resource requests and limits
- Use ResourceQuotas and LimitRanges in namespaces
- Monitor resource usage with metrics

### 2. **High Availability**
- Use Deployments, not bare Pods
- Set multiple replicas
- Configure PodDisruptionBudgets
- Use readiness and liveness probes

### 3. **Security**
- Use namespaces for isolation
- Implement RBAC (Roles, RoleBindings)
- Use NetworkPolicies
- Store secrets in Secrets, not ConfigMaps
- Enable Pod Security Standards
- Run containers as non-root
- Use ServiceAccounts with minimal permissions

### 4. **Configuration**
- Use ConfigMaps for configuration
- Use Secrets for sensitive data
- Don't hardcode configuration in images
- Use environment-specific namespaces

### 5. **Storage**
- Use PersistentVolumes for stateful apps
- Choose appropriate StorageClasses
- Use StatefulSets for stateful workloads
- Regular backups with VolumeSnapshots

### 6. **Networking**
- Use Services for pod communication
- Use Ingress for external HTTP/HTTPS access
- Implement NetworkPolicies for security
- Use ClusterIP for internal services

### 7. **Monitoring & Logging**
- Implement liveness and readiness probes
- Centralize logging (ELK, Fluentd)
- Use metrics-server for resource metrics
- Monitor with Prometheus/Grafana

### 8. **Deployment Strategy**
- Use rolling updates for zero-downtime
- Configure maxSurge and maxUnavailable
- Test in development/staging first
- Use HPA for auto-scaling
- Version your container images

### 9. **Labels & Annotations**
- Use consistent labeling conventions
- Label for organization (app, tier, environment)
- Use labels for selectors
- Use annotations for metadata

### 10. **Version Control**
- Store all manifests in Git
- Never commit secrets
- Use GitOps (ArgoCD, Flux)
- Document changes in commit messages

---

## Troubleshooting Guide

### Pod Issues

**Pod not starting:**
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events
```

Common causes:
- Image pull errors (check ImagePullPolicy)
- Resource constraints (insufficient CPU/memory)
- Volume mount issues
- Security context issues

**Pod crashes/restarts:**
```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

Common causes:
- Application errors (check logs)
- Resource limits exceeded (OOMKilled)
- Liveness probe failures
- Container image issues

### Service Issues

**Service not accessible:**
```bash
kubectl get endpoints <service-name>
kubectl describe service <service-name>
```

Common causes:
- No pods matching selector
- Pods not ready (readiness probe failing)
- Network policy blocking traffic
- Wrong port configuration

### Storage Issues

**PVC not binding:**
```bash
kubectl describe pvc <pvc-name>
kubectl get pv
```

Common causes:
- No available PV matching requirements
- Access mode mismatch
- Storage size too large
- StorageClass not available

### Performance Issues

**High resource usage:**
```bash
kubectl top pods
kubectl top nodes
kubectl describe node <node-name>
```

Solutions:
- Scale horizontally with HPA
- Optimize resource requests/limits
- Use VPA for right-sizing

---

## Conclusion

This guide covered all major Kubernetes manifest types from basic to advanced. Each resource serves a specific purpose in the Kubernetes ecosystem:

- **Workloads**: Run your applications (Pods, Deployments, StatefulSets)
- **Services**: Expose applications (Services, Ingress)
- **Configuration**: Manage configuration (ConfigMaps, Secrets)
- **Storage**: Persist data (PV, PVC, StorageClass)
- **Security**: Control access (RBAC, NetworkPolicy, ServiceAccounts)
- **Scheduling**: Optimize resource usage (HPA, VPA, ResourceQuota)
- **Advanced**: Extend Kubernetes (CRDs, Webhooks)

### Next Steps

1. **Start simple**: Begin with Pods, Deployments, and Services
2. **Add storage**: Learn PersistentVolumes and StatefulSets
3. **Implement security**: Set up RBAC and NetworkPolicies
4. **Optimize**: Use HPA, ResourceQuotas, and LimitRanges
5. **Advanced features**: Explore CRDs, Operators, and Webhooks

### Resources

- Official Kubernetes Documentation: https://kubernetes.io/docs/
- Kubernetes API Reference: https://kubernetes.io/docs/reference/
- kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- Kubernetes Patterns: https://k8spatterns.io/

---

**Remember**: Start with the basics, understand the fundamentals, and gradually move to advanced features as your needs grow. Kubernetes is powerful but complex - take it step by step!
