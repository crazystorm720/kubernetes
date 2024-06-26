The `kubectl config` commands are essential for managing Kubernetes configuration files and switching contexts, clusters, and users. Here are some of the most commonly used `kubectl config` commands:

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
