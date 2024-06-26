# Kubernetes Workflow Guide: Lifecycle Management and Pipeline Tasks

## 1. Deployment

### Deploying Applications

#### Create a Deployment
```sh
kubectl apply -f <deployment.yaml>
```
Creates a deployment from a YAML file.

#### Example Deployment YAML
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
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
```

### Exposing Applications

#### Create a Service
```sh
kubectl expose deployment my-app --port=80 --target-port=8080 --type=LoadBalancer
```
Exposes a deployment as a service.

#### Example Service YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### Creating Ingress Resources

#### Create an Ingress
```sh
kubectl apply -f <ingress.yaml>
```
Creates an ingress resource from a YAML file.

#### Example Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

## 2. Scaling

### Scaling Deployments

#### Scale a Deployment
```sh
kubectl scale deployment my-app --replicas=5
```
Scales the specified deployment to the desired number of replicas.

### Auto-Scaling

#### Create a Horizontal Pod Autoscaler
```sh
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```
Automatically scales the deployment based on CPU utilization.

#### Example HPA YAML
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## 3. Updating

### Rolling Updates

#### Update a Deployment
```sh
kubectl set image deployment/my-app my-app-container=my-app-image:v2
```
Updates the image of the deployment to a new version.

### Monitor Rollout Status
```sh
kubectl rollout status deployment/my-app
```
Checks the status of the rollout.

### Rollback a Deployment
```sh
kubectl rollout undo deployment/my-app
```
Rolls back the deployment to the previous version.

## 4. Monitoring

### Viewing Logs

#### View Pod Logs
```sh
kubectl logs <pod-name>
```
Displays the logs for a specific pod.

#### Stream Pod Logs
```sh
kubectl logs -f <pod-name>
```
Streams the logs for a specific pod.

### Describe Resources

#### Describe a Pod
```sh
kubectl describe pod <pod-name>
```
Displays detailed information about a specific pod.

#### Describe a Node
```sh
kubectl describe node <node-name>
```
Displays detailed information about a specific node.

### Resource Usage Metrics

#### Top Pods
```sh
kubectl top pod
```
Displays resource usage for pods.

#### Top Nodes
```sh
kubectl top node
```
Displays resource usage for nodes.

## 5. Storage Management

### Managing Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)

#### Create a Persistent Volume
```sh
kubectl apply -f <pv.yaml>
```
Creates a persistent volume from a YAML file.

#### Create a Persistent Volume Claim
```sh
kubectl apply -f <pvc.yaml>
```
Creates a persistent volume claim from a YAML file.

#### Example PV YAML
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

#### Example PVC YAML
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Configuring Volume Mounts in Pods

#### Example Pod with PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /usr/share/busybox
      name: demo-volume
  volumes:
  - name: demo-volume
    persistentVolumeClaim:
      claimName: pvc-demo
```

### Additional Storage Commands

#### Get Persistent Volume Claim Usage
```sh
kubectl describe pvc <pvc-name> | grep "Used By"
```
Shows which pods are using the specified PVC.

#### Expand a Persistent Volume Claim
If your storage class supports volume expansion, you can resize a PVC:

1. Edit the PVC to request more storage:
   ```sh
   kubectl edit pvc <pvc-name>
   ```
   Update the `spec.resources.requests.storage` field to the new size.

2. Verify the resize operation:
   ```sh
   kubectl get pvc <pvc-name>
   ```

#### Check Storage Class Default
```sh
kubectl get storageclass | grep "(default)"
```
Shows which storage class is set as the default.

### Example Storage Class

Create a `storageclass.yaml` file for a storage class:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Apply the storage class:
```sh
kubectl apply -f storageclass.yaml
```

## 6. Secrets Management

### Managing Secrets

#### Create a Secret from Literal Values
```sh
kubectl create secret generic my-secret --from-literal=username='admin' --from-literal=password='1f2d1e2e67df'
```
Creates a secret from literal values.

#### Create a Secret from a File
```sh
kubectl create secret generic my-secret --from-file=ssh-privatekey=/path/to/private/key --from-file=ssh-publickey=/path/to/public/key
```
Creates a secret from files.

#### Example Secret YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=   # base64 encoded value of 'admin'
  password: MWYyZDFlMmU2N2Rm # base64 encoded value of '1f2d1e2e67df'
```

#### Get Secrets
```sh
kubectl get secrets
```
Lists all secrets in the current namespace.

#### Describe a Secret
```sh
kubectl describe secret <secret-name>
```
Displays detailed information about a specific secret.

#### Delete a Secret
```sh
kubectl delete secret <secret-name>
```
Deletes a specific secret.

## 7. Config Maps

### Managing ConfigMaps

#### Create a ConfigMap from Literal Values
```sh
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```
Creates a ConfigMap from literal values.

#### Create a ConfigMap from a File
```sh
kubectl create configmap my-config --from-file=config-file.conf
```
Creates a ConfigMap from a file.

#### Example ConfigMap YAML
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
```

#### Get ConfigMaps
```sh
kubectl get configmaps
```
Lists all ConfigMaps in the current namespace.

#### Describe a ConfigMap
```sh
kubectl describe configmap <configmap-name>
```
Displays detailed information about a specific ConfigMap.

#### Delete a ConfigMap
```sh
kubectl delete configmap <configmap-name>
```
Deletes a specific ConfigMap.

## 8. Troubleshooting

### Debugging Pods

#### Get Pod Events
```sh
kubectl get events --namespace=<namespace> --field-selector involvedObject.name=<pod-name>
```
Gets events for a specific pod.

#### Execute a Command in a Pod
```sh
kubectl exec -it <pod-name> -- <command>
```
Executes a command in a specific pod.

#### Debugging with Ephemeral Containers
```sh
kubectl debug <pod-name> --image=busybox --target=<target-container>
```
Adds an ephemeral container to a

 running pod for debugging.

### Checking Cluster Health

#### Get Node Status
```sh
kubectl get nodes
```
Lists all nodes and their status.

#### Describe Node
```sh
kubectl describe node <node-name>
```
Displays detailed information about a specific node.

#### Check API Server Logs
```sh
kubectl logs -n kube-system <api-server-pod-name>
```
Displays the logs of the API server pod.

## 9. Security Best Practices

### Network Policies

#### Example Network Policy YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-my-app
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: my-app
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: my-app
```
Applies a network policy that allows traffic only to and from pods with the `my-app` label.

### Role-Based Access Control (RBAC)

#### Example RBAC YAML
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
Creates a role that allows reading pods.

#### Create a Role Binding
```sh
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=<username> --namespace=default
```
Binds the role to a user in the default namespace.

## 10. Cleaning Up

### Delete Resources

#### Delete a Deployment
```sh
kubectl delete deployment my-app
```
Deletes a specific deployment.

#### Delete a Service
```sh
kubectl delete service my-app-service
```
Deletes a specific service.

#### Delete an Ingress
```sh
kubectl delete ingress my-app-ingress
```
Deletes a specific ingress.

#### Delete a Persistent Volume
```sh
kubectl delete pv pv-demo
```
Deletes a specific persistent volume.

#### Delete a Persistent Volume Claim
```sh
kubectl delete pvc pvc-demo
```
Deletes a specific persistent volume claim.

## Configuration Management

The `kubectl config` commands are essential for managing Kubernetes configuration files and switching contexts, clusters, and users.

### 1. View the Current Configuration
```sh
kubectl config view
```
Displays the entire kubeconfig file.

### 2. Set a Context
```sh
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
```
Sets a new context or updates an existing one.

### 3. Use a Context
```sh
kubectl config use-context <context-name>
```
Switches to the specified context.

### 4. Set a Cluster
```sh
kubectl config set-cluster <cluster-name> --server=<server-address> --certificate-authority=<path-to-ca.crt>
```
Adds or modifies a cluster entry.

### 5. Set a User
```sh
kubectl config set-credentials <user-name> --client-certificate=<path-to-client.crt> --client-key=<path-to-client.key>
```
Adds or modifies a user entry.

### 6. Get Current Context
```sh
kubectl config current-context
```
Displays the name of the currently active context.

### 7. Delete a Context
```sh
kubectl config delete-context <context-name>
```
Removes a context from the kubeconfig file.

### 8. Delete a Cluster
```sh
kubectl config delete-cluster <cluster-name>
```
Removes a cluster entry from the kubeconfig file.

### 9. Delete a User
```sh
kubectl config delete-user <user-name>
```
Removes a user entry from the kubeconfig file.

### 10. List Contexts
```sh
kubectl config get-contexts
```
Lists all available contexts.

### 11. Rename a Context
```sh
kubectl config rename-context <old-context-name> <new-context-name>
```
Renames an existing context.

### 12. Set the Default Namespace for a Context
```sh
kubectl config set-context --current --namespace=<namespace>
```
Sets the default namespace for the currently active context.

### 13. Merge Multiple Kubeconfig Files
You can merge multiple kubeconfig files by setting the `KUBECONFIG` environment variable:

```sh
export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/config2
kubectl config view --merge --flatten > $HOME/.kube/merged-config
mv $HOME/.kube/merged-config $HOME/.kube/config
```

### Example Workflow

1. **Add a New Cluster**:
   ```sh
   kubectl config set-cluster my-cluster --server=https://my-cluster-server:6443 --certificate-authority=/path/to/ca.crt
   ```

2. **Add a New User**:
   ```sh
   kubectl config set-credentials my-user --client-certificate=/path/to/client.crt --client-key=/path/to/client.key
   ```

3. **Create a New Context**:
   ```sh
   kubectl config set-context my-context --cluster=my-cluster --user=my-user --namespace=my-namespace
   ```

4. **Switch to the New Context**:
   ```sh
   kubectl config use-context my-context
   ```

5. **Verify the Current Context**:
   ```sh
   kubectl config current-context
   ```

These commands allow you to effectively manage multiple Kubernetes clusters, users, and contexts, making it easier to switch between different environments and configurations.

---

# Kubernetes Workflow Guide: Lifecycle Management and Pipeline Tasks

## 1. Deployment

### Deploying Applications

#### Create a Deployment
```sh
kubectl apply -f <deployment.yaml>
```
Creates a deployment from a YAML file.

#### Example Deployment YAML
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
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
```

### Exposing Applications

#### Create a Service
```sh
kubectl expose deployment my-app --port=80 --target-port=8080 --type=LoadBalancer
```
Exposes a deployment as a service.

#### Example Service YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### Creating Ingress Resources

#### Create an Ingress
```sh
kubectl apply -f <ingress.yaml>
```
Creates an ingress resource from a YAML file.

#### Example Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

## 2. Scaling

### Scaling Deployments

#### Scale a Deployment
```sh
kubectl scale deployment my-app --replicas=5
```
Scales the specified deployment to the desired number of replicas.

### Auto-Scaling

#### Create a Horizontal Pod Autoscaler
```sh
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10
```
Automatically scales the deployment based on CPU utilization.

#### Example HPA YAML
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

## 3. Updating

### Rolling Updates

#### Update a Deployment
```sh
kubectl set image deployment/my-app my-app-container=my-app-image:v2
```
Updates the image of the deployment to a new version.

### Monitor Rollout Status
```sh
kubectl rollout status deployment/my-app
```
Checks the status of the rollout.

### Rollback a Deployment
```sh
kubectl rollout undo deployment/my-app
```
Rolls back the deployment to the previous version.

## 4. Monitoring

### Viewing Logs

#### View Pod Logs
```sh
kubectl logs <pod-name>
```
Displays the logs for a specific pod.

#### Stream Pod Logs
```sh
kubectl logs -f <pod-name>
```
Streams the logs for a specific pod.

### Describe Resources

#### Describe a Pod
```sh
kubectl describe pod <pod-name>
```
Displays detailed information about a specific pod.

#### Describe a Node
```sh
kubectl describe node <node-name>
```
Displays detailed information about a specific node.

### Resource Usage Metrics

#### Top Pods
```sh
kubectl top pod
```
Displays resource usage for pods.

#### Top Nodes
```sh
kubectl top node
```
Displays resource usage for nodes.

## 5. Storage Management

### Managing Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)

#### Create a Persistent Volume
```sh
kubectl apply -f <pv.yaml>
```
Creates a persistent volume from a YAML file.

#### Create a Persistent Volume Claim
```sh
kubectl apply -f <pvc.yaml>
```
Creates a persistent volume claim from a YAML file.

#### Example PV YAML
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

#### Example PVC YAML
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Configuring Volume Mounts in Pods

#### Example Pod with PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /usr/share/busybox
      name: demo-volume
  volumes:
  - name: demo-volume
    persistentVolumeClaim:
      claimName: pvc-demo
```

## 6. Cleaning Up

### Delete Resources

#### Delete a Deployment
```sh
kubectl delete deployment my-app
```
Deletes a specific deployment.

#### Delete a Service
```sh
kubectl delete service my-app-service
```
Deletes a specific service.

#### Delete an Ingress
```sh
kubectl delete ingress my-app-ingress
```
Deletes a specific ingress.

#### Delete a Persistent Volume
```sh
kubectl delete pv pv-demo
```
Deletes a specific persistent volume.

#### Delete a Persistent Volume Claim
```sh
kubectl delete pvc pvc-demo
```
Deletes a specific persistent volume claim.

---

# Kubernetes `kubectl` Command Guide

## Storage Management

`kubectl` commands related to storage typically involve managing persistent volumes (PVs), persistent volume claims (PVCs), storage classes, and configuring volume mounts in pods.

### 1. Managing Persistent Volumes (PVs)

#### Create a Persistent Volume
```sh
kubectl apply -f <pv.yaml>
```
Creates a persistent volume from a YAML file.

#### Get Persistent Volumes
```sh
kubectl get pv
```
Lists all persistent volumes in the cluster.

#### Describe a Persistent Volume
```sh
kubectl describe pv <pv-name>
```
Displays detailed information about a specific persistent volume.

#### Delete a Persistent Volume
```sh
kubectl delete pv <pv-name>
```
Deletes a specific persistent volume.

### 2. Managing Persistent Volume Claims (PVCs)

#### Create a Persistent Volume Claim
```sh
kubectl apply -f <pvc.yaml>
```
Creates a persistent volume claim from a YAML file.

#### Get Persistent Volume Claims
```sh
kubectl get pvc
```
Lists all persistent volume claims in the current namespace.

#### Describe a Persistent Volume Claim
```sh
kubectl describe pvc <pvc-name>
```
Displays detailed information about a specific persistent volume claim.

#### Delete a Persistent Volume Claim
```sh
kubectl delete pvc <pvc-name>
```
Deletes a specific persistent volume claim.

### 3. Managing Storage Classes

#### Create a Storage Class
```sh
kubectl apply -f <storageclass.yaml>
```
Creates a storage class from a YAML file.

#### Get Storage Classes
```sh
kubectl get storageclass
```
Lists all storage classes in the cluster.

#### Describe a Storage Class
```sh
kubectl describe storageclass <storageclass-name>
```
Displays detailed information about a specific storage class.

#### Delete a Storage Class
```sh
kubectl delete storageclass <storageclass-name>
```
Deletes a specific storage class.

### 4. Configuring Volume Mounts in Pods

Volumes can be configured in the pod specification within deployments, statefulsets, or pod manifests.

#### Example: Configuring a Volume Mount in a Pod

1. Create a `pv.yaml` file for the persistent volume:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv-demo
   spec:
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: /mnt/data
   ```

2. Create a `pvc.yaml` file for the persistent volume claim:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: pvc-demo
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

3. Create a `pod.yaml` file for the pod using the PVC:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: pod-demo
   spec:
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
       volumeMounts:
       - mountPath: /usr/share/busybox
         name: demo-volume
     volumes:
     - name: demo-volume
       persistentVolumeClaim:
         claimName: pvc-demo
   ```

4. Apply the persistent volume:
   ```sh
   kubectl apply -f pv.yaml
   ```

5. Apply the persistent volume claim:
   ```sh
   kubectl apply -f pvc.yaml
   ```

6. Apply the pod:
   ```sh
   kubectl apply -f pod.yaml
   ```

### Additional Storage Commands

#### Get Persistent Volume Claim Usage
```sh
kubectl describe pvc <pvc-name> | grep "Used By"
```
Shows which pods are using the specified PVC.

#### Expand a Persistent Volume Claim
If your storage class supports volume expansion, you can resize a PVC:

1. Edit the PVC to request more storage:
   ```sh
   kubectl edit pvc <pvc-name>
   ```
   Update the `spec.resources.requests.storage` field to the new size.

2. Verify the resize operation:
   ```sh
   kubectl get pvc <pvc-name>
   ```

#### Check Storage Class Default
```sh
kubectl get storageclass | grep "(default)"
```
Shows which storage class is set as the default.

### Example Storage Class

Create a `storageclass.yaml` file for a storage class:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Apply the storage class:
```sh
kubectl apply -f storageclass.yaml
```

---

## Networking Management

`kubectl` commands related to networking typically involve managing services, ingresses, and network policies.

### 1. Managing Services

#### Create a Service
```sh
kubectl expose deployment <deployment-name> --port=<port> --target-port=<target-port> --type=<service-type>
```
Exposes a deployment as a service.

#### Get Services
```sh
kubectl get services
```
Lists all services in the current namespace.

#### Describe a Service
```sh
kubectl describe service <service-name>
```
Displays detailed information about a specific service.

#### Delete a Service
```sh
kubectl delete service <service-name>
```
Deletes a specific service.

### 2. Managing Ingress

#### Create an Ingress
```sh
kubectl apply -f <ingress.yaml>
```
Creates an ingress resource from a YAML file.

#### Get Ingresses
```sh
kubectl get ingress
```
Lists all ingresses in the current namespace.

#### Describe an Ingress
```sh
kubectl describe ingress <ingress-name>
```
Displays detailed information about a specific ingress.

#### Delete an Ingress
```sh
kubectl delete ingress <ingress-name>
```
Deletes a specific ingress.

### 3. Managing Network Policies

#### Create a Network Policy
```sh
kubectl apply -f <network-policy.yaml>
```
Creates a network policy from a YAML file.

#### Get Network Policies
```sh
kubectl get networkpolicy
```
Lists all network policies in the current namespace.

#### Describe a Network Policy
```sh
kubectl describe networkpolicy <policy-name>
```
Displays detailed information about a specific network policy.

#### Delete a Network Policy
```sh
kubectl delete networkpolicy <policy-name>
```
Deletes a specific network policy.

### 4. Port Forwarding

#### Forward Ports
```sh
kubectl port-forward <pod-name> <local-port>:<pod-port>
```
Forwards one or more local ports to a pod.

### 5. Accessing Services

#### Get Service Endpoints
```sh
kubectl get endpoints
```
Lists all endpoints in the current namespace.

#### Describe an Endpoint
```sh
kubectl describe endpoints <endpoint-name>
```
Displays detailed information about a specific endpoint.

### Example Use Cases

#### Expose a Deployment as a Service
```sh
kubectl expose deployment my-deployment --port=80 --target-port=8080 --type=NodePort
```

#### Create an Ingress Resource
Create an `ingress.yaml` file:
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
            name: my-service
            port:
              number: 80
```
Apply the ingress resource:
```sh
kubectl apply -f ingress.yaml
```

#### Create a Network Policy
Create a `network-policy.yaml` file:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: my-app
    ports:
    - protocol: TCP
      port: 80
```
Apply the network policy:
```sh
kubectl apply -f network-policy.yaml
```

---

## Configuration Management

The `kubectl config` commands are essential for managing Kubernetes configuration files and switching contexts, clusters, and users.

### 1. View the Current Configuration
```sh
kubectl config view
```
Displays the entire kubeconfig file.

### 2. Set a Context
```sh
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
```
Sets a new context or updates an existing one.

### 3. Use a Context
```sh
kubectl config use-context <context-name>
```
Switches to the specified context.

### 4. Set a Cluster
```sh
kubectl config set-cluster <cluster-name> --server=<server-address> --certificate-authority=<path-to-ca.crt>
```
Adds or modifies a cluster entry.

### 5. Set a User
```sh
kubectl config set-credentials <user-name> --client-certificate=<path-to-client.crt> --client-key=<path-to-client.key>
```
Adds or modifies a user entry.



### 6. Get Current Context
```sh
kubectl config current-context
```
Displays the name of the currently active context.

### 7. Delete a Context
```sh
kubectl config delete-context <context-name>
```
Removes a context from the kubeconfig file.

### 8. Delete a Cluster
```sh
kubectl config delete-cluster <cluster-name>
```
Removes a cluster entry from the kubeconfig file.

### 9. Delete a User
```sh
kubectl config delete-user <user-name>
```
Removes a user entry from the kubeconfig file.

### 10. List Contexts
```sh
kubectl config get-contexts
```
Lists all available contexts.

### 11. Rename a Context
```sh
kubectl config rename-context <old-context-name> <new-context-name>
```
Renames an existing context.

### 12. Set the Default Namespace for a Context
```sh
kubectl config set-context --current --namespace=<namespace>
```
Sets the default namespace for the currently active context.

### 13. Merge Multiple Kubeconfig Files
You can merge multiple kubeconfig files by setting the `KUBECONFIG` environment variable:

```sh
export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/config2
kubectl config view --merge --flatten > $HOME/.kube/merged-config
mv $HOME/.kube/merged-config $HOME/.kube/config
```

### Example Workflow

1. **Add a New Cluster**:
   ```sh
   kubectl config set-cluster my-cluster --server=https://my-cluster-server:6443 --certificate-authority=/path/to/ca.crt
   ```

2. **Add a New User**:
   ```sh
   kubectl config set-credentials my-user --client-certificate=/path/to/client.crt --client-key=/path/to/client.key
   ```

3. **Create a New Context**:
   ```sh
   kubectl config set-context my-context --cluster=my-cluster --user=my-user --namespace=my-namespace
   ```

4. **Switch to the New Context**:
   ```sh
   kubectl config use-context my-context
   ```

5. **Verify the Current Context**:
   ```sh
   kubectl config current-context
   ```

These commands allow you to effectively manage multiple Kubernetes clusters, users, and contexts, making switching between different environments and configurations easier.
