# Kubernetes Core Concepts to Master

## 1. Pods

- **Definition**: Smallest deployable units in Kubernetes
- **Key Characteristics**:
  - Can contain one or more containers
  - Share network namespace and storage
  - Ephemeral (not designed to be long-lived)
- **Use Cases**:
  - Running a single container
  - Tightly coupled application components
- **Important Considerations**:
  - Resource requests and limits
  - Readiness and liveness probes
  - Init containers
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: nginx:latest
      ports:
      - containerPort: 80
  ```

## 2. Deployments

- **Definition**: Manages a replicated application on your cluster
- **Key Characteristics**:
  - Ensures a specified number of pod replicas are running
  - Supports rolling updates and rollbacks
  - Provides declarative updates for Pods and ReplicaSets
- **Use Cases**:
  - Deploying stateless applications
  - Ensuring high availability of applications
- **Important Considerations**:
  - Update strategy (RollingUpdate vs Recreate)
  - Revision history limit
  - Scaling (manual and auto)
- **Example YAML**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:latest
          ports:
          - containerPort: 80
  ```

## 3. Services

- **Definition**: An abstract way to expose an application running on a set of Pods
- **Key Characteristics**:
  - Provides stable network endpoint
  - Load balances traffic to Pods
  - Supports different types: ClusterIP, NodePort, LoadBalancer
- **Use Cases**:
  - Exposing applications within or outside the cluster
  - Service discovery within the cluster
- **Important Considerations**:
  - Service types and their use cases
  - Service discovery and DNS
  - Headless services for direct Pod addressing
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    selector:
      app: nginx
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    type: ClusterIP
  ```

## 4. ConfigMaps and Secrets

- **Definition**: 
  - ConfigMaps: Store non-confidential configuration data
  - Secrets: Store sensitive information
- **Key Characteristics**:
  - Decouple configuration from image content
  - Can be used as environment variables, command-line arguments, or config files
- **Use Cases**:
  - Application configuration
  - Storing API keys, passwords, certificates
- **Important Considerations**:
  - Updating ConfigMaps and Secrets
  - Secret encryption at rest
  - Mounting as volumes vs. environment variables
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: blue
    APP_MODE: prod

  ---
  apiVersion: v1
  kind: Secret
  metadata:
    name: app-secret
  type: Opaque
  data:
    DB_PASSWORD: cGFzc3dvcmQ=  # base64 encoded
  ```

## 5. Persistent Volumes and Claims

- **Definition**:
  - Persistent Volumes (PV): A piece of storage in the cluster
  - Persistent Volume Claims (PVC): A request for storage by a user
- **Key Characteristics**:
  - Abstracts details of how storage is provided from how it is consumed
  - Supports different storage classes
- **Use Cases**:
  - Databases, file storage, shared application data
- **Important Considerations**:
  - Storage classes and provisioners
  - Access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
  - Reclaim policies
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-volume
  spec:
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: manual
    nfs:
      server: nfs-server.default.svc.cluster.local
      path: "/path/to/data"

  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pv-claim
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
    storageClassName: manual
  ```

## 6. Namespaces

- **Definition**: Virtual clusters backed by the same physical cluster
- **Key Characteristics**:
  - Provides a scope for names
  - Allows resource isolation and quota management
- **Use Cases**:
  - Multi-tenant environments
  - Separating different stages (e.g., development, staging, production)
- **Important Considerations**:
  - Resource quotas
  - Network policies between namespaces
  - Default vs. custom namespaces
- **Example YAML**:
  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: development
  ```

## 7. Ingress

- **Definition**: API object that manages external access to services in a cluster
- **Key Characteristics**:
  - Provides HTTP/HTTPS routing
  - Can handle SSL termination
  - Supports name-based virtual hosting
- **Use Cases**:
  - Exposing multiple services under a single IP address
  - SSL/TLS termination
  - URL-based routing
- **Important Considerations**:
  - Ingress controller selection (e.g., Nginx, Traefik)
  - SSL/TLS certificate management
  - Path-based and host-based routing rules
- **Example YAML**:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example-ingress
  spec:
    rules:
    - host: example.com
      http:
        paths:
        - path: /app
          pathType: Prefix
          backend:
            service:
              name: app-service
              port: 
                number: 80
  ```

## 8. RBAC (Role-Based Access Control)

- **Definition**: Regulates access to resources based on the roles of individual users
- **Key Characteristics**:
  - Defines Roles (namespace-specific) and ClusterRoles (cluster-wide)
  - Uses RoleBindings and ClusterRoleBindings to assign roles to users
- **Use Cases**:
  - Implementing principle of least privilege
  - Managing access for different teams or users
- **Important Considerations**:
  - Granularity of permissions
  - Service account roles
  - Regular audits of RBAC policies
- **Example YAML**:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: default
    name: pod-reader
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: read-pods
    namespace: default
  subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
  ```

Each of these concepts is crucial for effectively working with Kubernetes. Understanding their definitions, characteristics, use cases, and important considerations will provide a solid foundation for managing containerized applications in a Kubernetes environment.

---

# Kubernetes (K8s) Strategy Document

## 1. Understanding Our Needs

### Key Questions to Consider:
- What are our primary goals for adopting Kubernetes?
- What types of applications will we be running on Kubernetes?
- What is our current infrastructure setup?
- What is our team's current knowledge level with Kubernetes?

### Potential Use Cases:
- Microservices architecture
- CI/CD pipeline improvement
- Multi-cloud strategy
- Scaling and high availability requirements

## 2. Core Concepts to Master

1. Pods
2. Deployments
3. Services
4. ConfigMaps and Secrets
5. Persistent Volumes and Claims
6. Namespaces
7. Ingress
8. RBAC (Role-Based Access Control)

## 3. Infrastructure Considerations

### Cluster Setup:
- On-premises vs. Cloud-managed Kubernetes services
- Single cluster vs. Multi-cluster strategy
- Development, staging, and production environments

### Networking:
- Network policies
- Service mesh considerations (e.g., Istio, Linkerd)
- Ingress controllers (e.g., Nginx, Traefik)

### Storage:
- Storage classes
- Persistent volume provisioners

## 4. Security Strategy

- Image scanning and signing
- Pod security policies
- Network policies
- Secrets management (e.g., Vault integration)
- RBAC implementation

## 5. Monitoring and Logging

- Prometheus for metrics
- Grafana for visualization
- ELK stack or alternatives for logging
- Tracing with Jaeger or Zipkin

## 6. CI/CD Integration

- GitOps workflow
- Helm for package management
- Continuous deployment strategies (e.g., Blue/Green, Canary)

## 7. Developer Experience

- Local development environments (e.g., Minikube, Kind)
- Standardized Dockerfile and Kubernetes manifests
- Development guidelines and best practices

## 8. Operations and Maintenance

- Upgrade strategies
- Backup and disaster recovery
- Capacity planning and auto-scaling
- Troubleshooting procedures

## 9. Cost Optimization

- Resource requests and limits
- Cluster autoscaler
- Spot instances (if cloud-based)
- Cost monitoring and allocation

## 10. Training and Documentation

- Internal knowledge base
- Training programs for developers and ops teams
- Runbooks for common scenarios

## 11. Compliance and Governance

- Audit logging
- Policy enforcement (e.g., OPA, Kyverno)
- Multi-tenancy considerations

## 12. Advanced Topics to Explore

- Custom Resource Definitions (CRDs) and Operators
- Serverless on Kubernetes (e.g., Knative)
- AI/ML workloads on Kubernetes

## Next Steps

1. Conduct a team workshop to discuss these points
2. Prioritize areas of focus based on immediate needs
3. Create a phased adoption plan
4. Set up a proof of concept for critical use cases
5. Develop initial policies and guidelines
6. Plan for gradual rollout and continuous improvement

Remember, Kubernetes adoption is a journey. Start small, learn continuously, and iterate on your approach as you gain more experience and insights.

---

# Complete Guide: Getting Started with Kubernetes using Minikube

## 1. Introduction
Kubernetes (k8s) is a powerful container orchestration platform. Minikube allows you to run a single-node Kubernetes cluster on your local machine, making it perfect for learning and development.

## 5. Deployment Methods

### 5.1 kubectl CLI
This is the most straightforward method for deploying applications.

Example:
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

Access the deployed application:
```
minikube service hello-minikube
```

### 5.2 YAML Files
Create a file named `deployment.yaml`:

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
      - name: my-app
        image: nginx:latest
        ports:
        - containerPort: 80
```

Apply the deployment:
```
kubectl apply -f deployment.yaml
```

### 5.3 Helm Charts
1. **Install Helm**: Follow the [Helm Installation Guide](https://helm.sh/docs/intro/install/)
2. **Add a Helm repository**:
   ```
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
3. **Install a chart**:
   ```
   helm install my-release bitnami/wordpress
   ```

### 5.4 Minikube Addons
List available addons:
```
minikube addons list
```

Enable an addon:
```
minikube addons enable ingress
```

### 5.5 Docker Builds
Use Minikube's Docker daemon:
```
eval $(minikube docker-env)
docker build -t my-image:tag .
```

Then use the image in a deployment:
```
kubectl create deployment my-app --image=my-image:tag
```

## 6. Basic Kubernetes Concepts

### Pods
- Smallest deployable units in Kubernetes
- Can contain one or more containers
- Share the same network namespace and storage

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Deployments
- Manage the deployment and scaling of a set of Pods
- Ensure a specified number of pod replicas are running

Example:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Services
- Expose applications running on a set of Pods
- Provide a stable network endpoint

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Namespaces
- Virtual clusters for resource isolation
- Useful for multi-tenant environments

Example:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

## 7. Useful Commands

### Viewing Resources
- List all pods:
  ```
  kubectl get pods
  ```
  Example output:
  ```
  NAME                                READY   STATUS    RESTARTS   AGE
  nginx-deployment-6b474476c4-2z9xk   1/1     Running   0          2m
  nginx-deployment-6b474476c4-9x8jd   1/1     Running   0          2m
  nginx-deployment-6b474476c4-tqllm   1/1     Running   0          2m
  ```

- List all deployments:
  ```
  kubectl get deployments
  ```
  Example output:
  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3/3     3            3           5m
  ```

- List all services:
  ```
  kubectl get services
  ```
  Example output:
  ```
  NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP        1h
  nginx-service   LoadBalancer   10.110.126.65   <pending>     80:31234/TCP   3m
  ```

### Detailed Information
- View logs for a specific pod:
  ```
  kubectl logs nginx-deployment-6b474476c4-2z9xk
  ```
  Example output:
  ```
  10.244.0.1 - - [25/Jun/2023:12:00:00 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 ..." "-"
  ```

- Get detailed information about a pod:
  ```
  kubectl describe pod nginx-deployment-6b474476c4-2z9xk
  ```
  Example output (truncated):
  ```
  Name:         nginx-deployment-6b474476c4-2z9xk
  Namespace:    default
  Priority:     0
  Node:         minikube/192.168.49.2
  Start Time:   Sun, 25 Jun 2023 12:00:00 +0000
  Labels:       app=nginx
                pod-template-hash=6b474476c4
  Status:       Running
  IP:           172.17.0.4
  ...
  ```

### Scaling
- Scale a deployment:
  ```
  kubectl scale deployment nginx-deployment --replicas=5
  ```
  Example output:
  ```
  deployment.apps/nginx-deployment scaled
  ```

### Updating
- Apply changes from a YAML file:
  ```
  kubectl apply -f updated-deployment.yaml
  ```
  Example output:
  ```
  deployment.apps/nginx-deployment configured
  ```

### Deleting Resources
- Delete a deployment:
  ```
  kubectl delete deployment nginx-deployment
  ```
  Example output:
  ```
  deployment.apps "nginx-deployment" deleted
  ```

## 8. Cleaning Up
Delete a deployment:
```
kubectl delete deployment <deployment-name>
```

Stop Minikube:
```
minikube stop
```

Delete Minikube cluster:
```
minikube delete
```

## 9. Next Steps
- Explore Kubernetes documentation: [kubernetes.io](https://kubernetes.io/docs/home/)
- Try deploying more complex applications
- Learn about Kubernetes networking and storage concepts

## 10. Standard Kubernetes Namespace Schema

### Core Namespaces

1. **kube-system**: Reserved for Kubernetes system components.
2. **kube-public**: Publicly readable, used for cluster-wide resources.
3. **kube-node-lease**: Used for node heartbeat data.

### Environment Namespaces

4. **development**: For development workloads.
5. **staging**: For pre-production testing.
6. **production**: For production workloads.

### Application Namespaces

7. **app-{name}**: For each major application or service (e.g., `app-frontend`, `app-backend`).

### Infrastructure Namespaces

8. **monitoring**: For monitoring tools (e.g., Prometheus, Grafana).
9. **logging**: For logging infrastructure (e.g., ELK stack).
10. **ingress**: For ingress controllers and related configs.

### Team Namespaces

11. **team-{name}**: For team-specific resources (e.g., `team-data-science`, `team-devops`).

### Utility Namespaces

12. **tools**: For shared development tools (e.g., CI/CD pipelines).
13. **security**: For security-related tools and policies.

### Tenant Namespaces (for multi-tenant clusters)

14. **tenant-{name}**: For isolating resources of different tenants.

### Best Practices

- **Consistent Naming Conventions**: Use lowercase letters and hyph

ens for spaces.
- **Resource Quotas and Limits**: Apply resource quotas and limits to each namespace to manage capacity.
- **RBAC Policies**: Implement role-based access control (RBAC) policies for fine-grained access control.
- **Network Policies**: Use network policies to control communication between namespaces.
- **Consistent Labeling**: Label all resources within namespaces consistently to facilitate management and automation.

## 11. Why Namespaces are Important

### Resource Isolation
Namespaces provide a scope for names, allowing you to organize cluster resources into non-overlapping groups.

### Access Control
You can use namespaces to implement role-based access control (RBAC), limiting who can do what in specific parts of your cluster.

### Resource Quotas
Namespaces allow you to set resource quotas at a group level, helping manage cluster capacity and prevent resource hogging.

### Environment Separation
You can use namespaces to separate different environments (e.g., development, staging, production) within the same cluster.

### Multi-tenancy
In shared clusters, namespaces can be used to separate different teams, projects, or customers.

## 12. How to Utilize Namespaces Correctly

1. **Create Meaningful Namespaces**:
   Use descriptive names that reflect the purpose of the namespace. For example:
   ```
   kubectl create namespace production-env
   kubectl create namespace dev-team-a
   ```

2. **Use Namespaces in Resource Definitions**:
   Always specify the namespace when creating resources. In YAML files:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
     namespace: dev-team-a
   ```

3. **Set a Default Namespace for Your Context**:
   To avoid always specifying the namespace, set a default:
   ```
   kubectl config set-context --current --namespace=dev-team-a
   ```

4. **Use Namespace-Based Resource Quotas**:
   Implement resource quotas to limit resource usage per namespace:
   ```yaml
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: compute-resources
     namespace: dev-team-a
   spec:
     hard:
       requests.cpu: "1"
       requests.memory: 1Gi
       limits.cpu: "2"
       limits.memory: 2Gi
   ```

5. **Implement Network Policies**:
   Use network policies to control traffic flow between namespaces:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-from-other-namespaces
     namespace: dev-team-a
   spec:
     podSelector:
       matchLabels:
     ingress:
     - from:
       - podSelector: {}
   ```

6. **Use RBAC with Namespaces**:
   Implement role-based access control scoped to namespaces:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: dev-team-a
     name: pod-reader
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "watch", "list"]
   ```

7. **Namespace Naming Conventions**:
   Establish a clear naming convention for your namespaces. For example:
   - `<environment>-<team>-<project>`
   - `prod-frontend`, `staging-backend`, `dev-data-processing`

8. **Avoid Overuse**:
   While namespaces are useful, don't create too many. A good rule of thumb is to create namespaces around team boundaries or major subsystems.

9. **Be Aware of Namespace Limitations**:
   Some Kubernetes resources (like Nodes and PersistentVolumes) are cluster-scoped and don't belong to any namespace.

10. **Use Namespace Selectors in Ingress**:
    If you're using Ingress resources, you can use namespace selectors to route traffic to services in specific namespaces.

By following these best practices, you'll be able to effectively utilize Kubernetes namespaces to improve the organization, security, and resource management of your cluster.

---

## Core Kubernetes Concepts and Components

### 1. Pods

**Description**: 
- The smallest deployable units in Kubernetes, which can contain one or more containers.
- Containers in a Pod share the same network namespace and storage volumes.

**Example YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 2. Deployments

**Description**: 
- Used to manage the deployment and scaling of a set of Pods.
- Ensures a specified number of replicas are running.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 3. Services

**Description**: 
- Provides a stable network endpoint to expose applications running on a set of Pods.
- Can be of different types like ClusterIP, NodePort, LoadBalancer, etc.

**Example YAML**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### 4. ConfigMaps

**Description**: 
- Used to store configuration data in key-value pairs.
- Decouples configuration artifacts from image content to keep containerized applications portable.

**Example YAML**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key: value
  another-key: another-value
```

### 5. Secrets

**Description**: 
- Similar to ConfigMaps but used to store sensitive data such as passwords, OAuth tokens, and SSH keys.
- Data is encoded in base64.

**Example YAML**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

### 6. Persistent Volumes (PV) and Persistent Volume Claims (PVC)

**Description**: 
- PV: A piece of storage in the cluster provisioned by an administrator or dynamically provisioned.
- PVC: A request for storage by a user.

**Example YAML**:

*Persistent Volume*:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

*Persistent Volume Claim*:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 7. Ingress

**Description**: 
- Manages external access to services, typically HTTP.
- Provides load balancing, SSL termination, and name-based virtual hosting.

**Example YAML**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

### 8. RBAC (Role-Based Access Control)

**Description**: 
- Mechanism for regulating access to resources based on the roles of individual users within the cluster.

**Example YAML**:

*Role*:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

*RoleBinding*:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 9. Network Policies

**Description**: 
- Specifies how groups of pods are allowed to communicate with each other and other network endpoints.

**Example YAML**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
```

### 10. Horizontal Pod Autoscaler (HPA)

**Description**: 
- Automatically scales the number of pods in a deployment or replica set based on

 observed CPU utilization or other select metrics.

**Example YAML**:
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

### 11. DaemonSets

**Description**: 
- Ensures that all (or some) nodes run a copy of a Pod.
- Used for running agents on each node, such as log collectors or monitoring agents.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      name: example
  template:
    metadata:
      labels:
        name: example
    spec:
      containers:
      - name: example-container
        image: example/image:latest
```

### 12. StatefulSets

**Description**: 
- Manages the deployment and scaling of a set of Pods with unique, persistent identities and stable network identities.
- Used for stateful applications, such as databases.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: example/image:latest
        ports:
        - containerPort: 80
```

## Next Steps
- Document advanced concepts such as Kubernetes Operators, Custom Resource Definitions (CRDs), and Service Mesh.
- Explore Kubernetes monitoring and logging tools (e.g., Prometheus, Grafana, ELK stack).
- Learn about Kubernetes security best practices and tools (e.g., OPA, Falco).

By documenting these core concepts and components, you will provide a comprehensive guide that helps users understand and effectively use Kubernetes for their container orchestration needs.

---

# Complete Guide: Getting Started with Kubernetes using Minikube

## 1. Introduction
Kubernetes (k8s) is a powerful container orchestration platform. Minikube allows you to run a single-node Kubernetes cluster on your local machine, making it perfect for learning and development.

## 5. Deployment Methods

### 5.1 kubectl CLI
This is the most straightforward method for deploying applications.

Example:
```
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

Access the deployed application:
```
minikube service hello-minikube
```

### 5.2 YAML Files
Create a file named `deployment.yaml`:

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
      - name: my-app
        image: nginx:latest
        ports:
        - containerPort: 80
```

Apply the deployment:
```
kubectl apply -f deployment.yaml
```

### 5.3 Helm Charts
1. **Install Helm**: Follow the [Helm Installation Guide](https://helm.sh/docs/intro/install/)
2. **Add a Helm repository**:
   ```
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
3. **Install a chart**:
   ```
   helm install my-release bitnami/wordpress
   ```

### 5.4 Minikube Addons
List available addons:
```
minikube addons list
```

Enable an addon:
```
minikube addons enable ingress
```

### 5.5 Docker Builds
Use Minikube's Docker daemon:
```
eval $(minikube docker-env)
docker build -t my-image:tag .
```

Then use the image in a deployment:
```
kubectl create deployment my-app --image=my-image:tag
```

## 6. Basic Kubernetes Concepts

### Pods
- Smallest deployable units in Kubernetes
- Can contain one or more containers
- Share the same network namespace and storage

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Deployments
- Manage the deployment and scaling of a set of Pods
- Ensure a specified number of pod replicas are running

Example:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Services
- Expose applications running on a set of Pods
- Provide a stable network endpoint

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### Namespaces
- Virtual clusters for resource isolation
- Useful for multi-tenant environments

Example:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

## 7. Useful Commands

### Viewing Resources
- List all pods:
  ```
  kubectl get pods
  ```
  Example output:
  ```
  NAME                                READY   STATUS    RESTARTS   AGE
  nginx-deployment-6b474476c4-2z9xk   1/1     Running   0          2m
  nginx-deployment-6b474476c4-9x8jd   1/1     Running   0          2m
  nginx-deployment-6b474476c4-tqllm   1/1     Running   0          2m
  ```

- List all deployments:
  ```
  kubectl get deployments
  ```
  Example output:
  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3/3     3            3           5m
  ```

- List all services:
  ```
  kubectl get services
  ```
  Example output:
  ```
  NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP        1h
  nginx-service   LoadBalancer   10.110.126.65   <pending>     80:31234/TCP   3m
  ```

### Detailed Information
- View logs for a specific pod:
  ```
  kubectl logs nginx-deployment-6b474476c4-2z9xk
  ```
  Example output:
  ```
  10.244.0.1 - - [25/Jun/2023:12:00:00 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 ..." "-"
  ```

- Get detailed information about a pod:
  ```
  kubectl describe pod nginx-deployment-6b474476c4-2z9xk
  ```
  Example output (truncated):
  ```
  Name:         nginx-deployment-6b474476c4-2z9xk
  Namespace:    default
  Priority:     0
  Node:         minikube/192.168.49.2
  Start Time:   Sun, 25 Jun 2023 12:00:00 +0000
  Labels:       app=nginx
                pod-template-hash=6b474476c4
  Status:       Running
  IP:           172.17.0.4
  ...
  ```

### Scaling
- Scale a deployment:
  ```
  kubectl scale deployment nginx-deployment --replicas=5
  ```
  Example output:
  ```
  deployment.apps/nginx-deployment scaled
  ```

### Updating
- Apply changes from a YAML file:
  ```
  kubectl apply -f updated-deployment.yaml
  ```
  Example output:
  ```
  deployment.apps/nginx-deployment configured
  ```

### Deleting Resources
- Delete a deployment:
  ```
  kubectl delete deployment nginx-deployment
  ```
  Example output:
  ```
  deployment.apps "nginx-deployment" deleted
  ```

## 8. Cleaning Up
Delete a deployment:
```
kubectl delete deployment <deployment-name>
```

Stop Minikube:
```
minikube stop
```

Delete Minikube cluster:
```
minikube delete
```

## 9. Next Steps
- Explore Kubernetes documentation: [kubernetes.io](https://kubernetes.io/docs/home/)
- Try deploying more complex applications
- Learn about Kubernetes networking and storage concepts

## 10. Why Namespaces are Important

### Resource Isolation
Namespaces provide a scope for names, allowing you to organize cluster resources into non-overlapping groups.

### Access Control
You can use namespaces to implement role-based access control (RBAC), limiting who can do what in specific parts of your cluster.

### Resource Quotas
Namespaces allow you to set resource quotas at a group level, helping manage cluster capacity and prevent resource hogging.

### Environment Separation
You can use namespaces to separate different environments (e.g., development, staging, production) within the same cluster.

### Multi-tenancy
In shared clusters, namespaces can be used to separate different teams, projects, or customers.

## 11. How to Utilize Namespaces Correctly

1. **Create meaningful namespaces**:
   Use descriptive names that reflect the purpose of the namespace. For example:
   ```
   kubectl create namespace production-env
   kubectl create namespace dev-team-a
   ```

2. **Use namespaces in resource definitions**:
   Always specify the namespace when creating resources. In YAML files:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
     namespace: dev-team-a
   ```

3. **Set a default namespace for your context**:
   To avoid always specifying



 the namespace, set a default:
   ```
   kubectl config set-context --current --namespace=dev-team-a
   ```

4. **Use namespace-based resource quotas**:
   Implement resource quotas to limit resource usage per namespace:
   ```yaml
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: compute-resources
     namespace: dev-team-a
   spec:
     hard:
       requests.cpu: "1"
       requests.memory: 1Gi
       limits.cpu: "2"
       limits.memory: 2Gi
   ```

5. **Implement network policies**:
   Use network policies to control traffic flow between namespaces:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-from-other-namespaces
     namespace: dev-team-a
   spec:
     podSelector:
       matchLabels:
     ingress:
     - from:
       - podSelector: {}
   ```

6. **Use RBAC with namespaces**:
   Implement role-based access control scoped to namespaces:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: dev-team-a
     name: pod-reader
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "watch", "list"]
   ```

7. **Namespace naming conventions**:
   Establish a clear naming convention for your namespaces. For example:
   - `<environment>-<team>-<project>`
   - `prod-frontend`, `staging-backend`, `dev-data-processing`

8. **Don't overuse namespaces**:
   While namespaces are useful, don't create too many. A good rule of thumb is to create namespaces around team boundaries or major subsystems.

9. **Be aware of namespace limitations**:
   Some Kubernetes resources (like Nodes and PersistentVolumes) are cluster-scoped and don't belong to any namespace.

10. **Use namespace selectors in your Ingress**:
    If you're using Ingress resources, you can use namespace selectors to route traffic to services in specific namespaces.

By following these practices, you'll be able to effectively utilize Kubernetes namespaces to improve your cluster's organization, security, and resource management.

---

# Standard Kubernetes Namespace Schema

## Core Namespaces

1. **kube-system**: Reserved for Kubernetes system components.
2. **kube-public**: Publicly readable, used for cluster-wide resources.
3. **kube-node-lease**: Used for node heartbeat data.

## Environment Namespaces

4. **development**: For development workloads.
5. **staging**: For pre-production testing.
6. **production**: For production workloads.

## Application Namespaces

7. **app-{name}**: For each major application or service (e.g., `app-frontend`, `app-backend`).

## Infrastructure Namespaces

8. **monitoring**: For monitoring tools (e.g., Prometheus, Grafana).
9. **logging**: For logging infrastructure (e.g., ELK stack).
10. **ingress**: For ingress controllers and related configs.

## Team Namespaces

11. **team-{name}**: For team-specific resources (e.g., `team-data-science`, `team-devops`).

## Utility Namespaces

12. **tools**: For shared development tools (e.g., CI/CD pipelines).
13. **security**: For security-related tools and policies.

## Tenant Namespaces (for multi-tenant clusters)

14. **tenant-{name}**: For isolating resources of different tenants.

## Best Practices

- **Consistent Naming Conventions**: Use lowercase letters and hyphens for spaces.
- **Resource Quotas and Limits**: Apply resource quotas and limits to each namespace to manage capacity.
- **RBAC Policies**: Implement role-based access control (RBAC) policies for fine-grained access control.
- **Network Policies**: Use network policies to control communication between namespaces.
- **Consistent Labeling**: Label all resources within namespaces consistently to facilitate management and automation.

## Why Namespaces are Important

1. **Resource Isolation**: Namespaces provide a scope for names, allowing you to organize cluster resources into non-overlapping groups.
2. **Access Control**: Implementing RBAC within namespaces limits who can perform actions in specific parts of your cluster.
3. **Resource Quotas**: Enforcing resource quotas at the namespace level helps manage cluster capacity and prevent resource hogging.
4. **Environment Separation**: Use namespaces to separate different environments (e.g., development, staging, production) within the same cluster.
5. **Multi-tenancy**: Namespaces enable resource isolation for different teams, projects, or customers in shared clusters.

## How to Utilize Namespaces Correctly

1. **Create Meaningful Namespaces**:
   - Use descriptive names reflecting the purpose (e.g., `production-env`, `dev-team-a`).

2. **Specify Namespace in Resource Definitions**:
   - Always include the namespace in your YAML files.
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
     namespace: dev-team-a
   ```

3. **Set Default Namespace for Context**:
   - To avoid specifying the namespace each time, set a default:
   ```
   kubectl config set-context --current --namespace=dev-team-a
   ```

4. **Namespace-Based Resource Quotas**:
   - Implement quotas to limit resource usage per namespace:
   ```yaml
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: compute-resources
     namespace: dev-team-a
   spec:
     hard:
       requests.cpu: "1"
       requests.memory: 1Gi
       limits.cpu: "2"
       limits.memory: 2Gi
   ```

5. **Network Policies**:
   - Control traffic flow between namespaces:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: deny-from-other-namespaces
     namespace: dev-team-a
   spec:
     podSelector:
       matchLabels:
     ingress:
     - from:
       - podSelector: {}
   ```

6. **RBAC with Namespaces**:
   - Apply role-based access control scoped to namespaces:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: dev-team-a
     name: pod-reader
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "watch", "list"]
   ```

7. **Naming Conventions**:
   - Establish a clear naming convention for your namespaces (e.g., `<environment>-<team>-<project>`, such as `prod-frontend`).

8. **Avoid Overuse**:
   - Avoid creating too many namespaces. Group resources logically around team boundaries or major subsystems.

9. **Be Aware of Namespace Limitations**:
   - Some resources (e.g., Nodes, PersistentVolumes) are cluster-scoped and do not belong to any namespace.

10. **Namespace Selectors in Ingress**:
    - Use namespace selectors to route traffic to services in specific namespaces when using Ingress resources.

By following these best practices, you can effectively utilize Kubernetes namespaces to improve the organization, security, and resource management of your cluster.

---

After documenting namespaces, it's essential to cover other core Kubernetes concepts and components to provide a comprehensive guide. Here are the next key topics you should clearly document:

## Core Kubernetes Concepts and Components

### 1. Pods

**Description**: 
- The smallest deployable units in Kubernetes, which can contain one or more containers.
- Containers in a Pod share the same network namespace and storage volumes.

**Example YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 2. Deployments

**Description**: 
- Used to manage the deployment and scaling of a set of Pods.
- Ensures a specified number of replicas are running.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 3. Services

**Description**: 
- Provides a stable network endpoint to expose applications running on a set of Pods.
- Can be of different types like ClusterIP, NodePort, LoadBalancer, etc.

**Example YAML**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### 4. ConfigMaps

**Description**: 
- Used to store configuration data in key-value pairs.
- Decouples configuration artifacts from image content to keep containerized applications portable.

**Example YAML**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key: value
  another-key: another-value
```

### 5. Secrets

**Description**: 
- Similar to ConfigMaps but used to store sensitive data such as passwords, OAuth tokens, and SSH keys.
- Data is encoded in base64.

**Example YAML**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

### 6. Persistent Volumes (PV) and Persistent Volume Claims (PVC)

**Description**: 
- PV: A piece of storage in the cluster provisioned by an administrator or dynamically provisioned.
- PVC: A request for storage by a user.

**Example YAML**:

*Persistent Volume*:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

*Persistent Volume Claim*:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 7. Ingress

**Description**: 
- Manages external access to services, typically HTTP.
- Provides load balancing, SSL termination, and name-based virtual hosting.

**Example YAML**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

### 8. RBAC (Role-Based Access Control)

**Description**: 
- Mechanism for regulating access to resources based on the roles of individual users within the cluster.

**Example YAML**:

*Role*:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

*RoleBinding*:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 9. Network Policies

**Description**: 
- Specifies how groups of pods are allowed to communicate with each other and other network endpoints.

**Example YAML**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: backend
```

### 10. Horizontal Pod Autoscaler (HPA)

**Description**: 
- Automatically scales the number of pods in a deployment or replica set based on observed CPU utilization or other select metrics.

**Example YAML**:
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-example
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

### 11. DaemonSets

**Description**: 
- Ensures that all (or some) nodes run a copy of a Pod.
- Used for running agents on each node, such as log collectors or monitoring agents.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      name: example
  template:
    metadata:
      labels:
        name: example
    spec:
      containers:
      - name: example-container
        image: example/image:latest
```

### 12. StatefulSets

**Description**: 
- Manages the deployment and scaling of a set of Pods with unique, persistent identities and stable network identities.
- Used for stateful applications, such as databases.

**Example YAML**:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: example/image:latest
        ports:
        - containerPort: 80
```

## Next Steps
- Document advanced concepts such as Kubernetes Operators, Custom Resource Definitions (CRDs), and Service Mesh.
- Explore Kubernetes monitoring and logging tools (e.g., Prometheus, Grafana, ELK stack).
- Learn about Kubernetes security best practices and tools (e.g., OPA, Falco).

By documenting these core concepts and components, you will provide a comprehensive guide that helps users understand and effectively use Kubernetes for their container orchestration needs.
