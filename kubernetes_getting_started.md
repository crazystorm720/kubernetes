# Complete Guide: Getting Started with Kubernetes using Minikube

## 1. Introduction
Kubernetes (k8s) is a powerful container orchestration platform. Minikube allows you to run a single-node Kubernetes cluster on your local machine, making it perfect for learning and development.

## 2. Prerequisites
- A computer with virtualization support
- Administrative access to install software

## 3. Installation
1. **Install Docker**: Follow the [Docker Installation Guide](https://docs.docker.com/get-docker/)
2. **Install Minikube**: Follow the [Minikube Installation Guide](https://minikube.sigs.k8s.io/docs/start/)
3. **Install kubectl**: Follow the [kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/)

## 4. Starting Minikube
Open a terminal and run:
```
minikube start
```

Verify the cluster is running:
```
kubectl cluster-info
```

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
