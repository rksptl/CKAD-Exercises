# Service Accounts

This section covers Kubernetes Service Accounts, which are used to provide an identity for processes that run in a Pod.

## Key Resources

- [Service Accounts Documentation](https://kubernetes.io/docs/concepts/security/service-accounts/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

## Introduction to Service Accounts

Service Accounts provide an identity for processes that run in a Pod, allowing them to interact with the Kubernetes API server and other services. Every Pod that runs in a Kubernetes cluster is associated with a Service Account, which provides the Pod with an identity and credentials to access the Kubernetes API.

## Key Concepts

- **Service Account**: An identity used by Pods to authenticate to the Kubernetes API server
- **Tokens**: Automatically mounted to Pods to authenticate API requests
- **Default Service Account**: Every namespace has a default Service Account automatically created
- **Custom Service Accounts**: Can be created for specific applications with tailored permissions

## Service Accounts

### Exercise 1: Creating and Using Service Accounts

**Goal**: Learn how to create Service Accounts and use them in Pods.

**Task**: Create a Service Account and configure a Pod to use it.

**Why this matters**: Service Accounts are essential for controlling what Pods can do within a Kubernetes cluster. Understanding how to create and use Service Accounts is important for implementing proper security controls.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Service Account**

```bash
kubectl create serviceaccount app-sa
```

**Step 2: Verify the Service Account**

```bash
kubectl get serviceaccount app-sa -o yaml
```

You should see output similar to:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
```

**Step 3: Create a Pod that uses the Service Account**

Create a file named `pod-with-sa.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: bitnami/kubectl:latest
    command: ["sleep", "3600"]
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-sa.yaml
```

**Step 4: Verify the Pod is using the Service Account**

```bash
kubectl get pod sa-pod -o yaml | grep serviceAccount
```

You should see:

```
serviceAccount: app-sa
serviceAccountName: app-sa
```

**Step 5: Test the Service Account token**

```bash
kubectl exec -it sa-pod -- ls -l /var/run/secrets/kubernetes.io/serviceaccount/
kubectl exec -it sa-pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

**What this does**:

- Creates a custom Service Account
- Creates a Pod that uses the custom Service Account
- The Pod automatically gets a token for the Service Account mounted at `/var/run/secrets/kubernetes.io/serviceaccount/`
- This token can be used to authenticate API requests from the Pod

</p>
</details>

### Exercise 2: Service Accounts and RBAC

**Goal**: Learn how to use RBAC to control what a Service Account can do.

**Task**: Create a Service Account with limited permissions and test it.

**Why this matters**: Role-Based Access Control (RBAC) is used to restrict what actions a Service Account can perform, implementing the principle of least privilege.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Service Account**

```bash
kubectl create serviceaccount restricted-sa
```

**Step 2: Create a Role with limited permissions**

Create a file named `pod-reader-role.yaml` with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Apply the configuration:

```bash
kubectl apply -f pod-reader-role.yaml
```

**Step 3: Bind the Role to the Service Account**

Create a file named `pod-reader-rolebinding.yaml` with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: restricted-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the configuration:

```bash
kubectl apply -f pod-reader-rolebinding.yaml
```

**Step 4: Create a Pod that uses the Service Account**

Create a file named `restricted-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  serviceAccountName: restricted-sa
  containers:
  - name: kubectl-container
    image: bitnami/kubectl:latest
    command: ["sleep", "3600"]
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f restricted-pod.yaml
```

**Step 5: Test the permissions**

```bash
# This should work (list pods)
kubectl exec -it restricted-pod -- kubectl get pods

# This should fail (create pods)
kubectl exec -it restricted-pod -- kubectl run test --image=nginx

# This should fail (list deployments)
kubectl exec -it restricted-pod -- kubectl get deployments
```

**What this does**:

- Creates a Service Account with limited permissions
- Creates a Role that only allows reading Pods
- Binds the Role to the Service Account
- Creates a Pod that uses the Service Account
- Tests that the Pod can only perform the allowed actions

</p>
</details>

### Exercise 3: Disabling Service Account Token Automounting

**Goal**: Learn how to disable the automatic mounting of the Service Account token.

**Task**: Create a Pod that does not have a Service Account token mounted.

**Why this matters**: For security reasons, you might want to prevent a Pod from accessing the Kubernetes API by disabling token automounting.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Service Account with automounting disabled**

Create a file named `no-automount-sa.yaml` with the following content:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-automount-sa
automountServiceAccountToken: false
```

Apply the configuration:

```bash
kubectl apply -f no-automount-sa.yaml
```

**Step 2: Create a Pod that uses the Service Account**

Create a file named `no-token-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-token-pod
spec:
  serviceAccountName: no-automount-sa
  containers:
  - name: app
    image: busybox:1.36
    command: ["sleep", "3600"]
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f no-token-pod.yaml
```

**Step 3: Verify that the token is not mounted**

```bash
kubectl exec -it no-token-pod -- ls -l /var/run/secrets/kubernetes.io/serviceaccount/ 2>/dev/null || echo "Token not mounted"
```

You should see "Token not mounted" or an error indicating that the directory doesn't exist.

**Step 4: Create a Pod with automounting disabled at the Pod level**

Create a file named `pod-no-automount.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-automount
spec:
  automountServiceAccountToken: false
  containers:
  - name: app
    image: busybox:1.36
    command: ["sleep", "3600"]
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-no-automount.yaml
```

**Step 5: Verify that the token is not mounted**

```bash
kubectl exec -it pod-no-automount -- ls -l /var/run/secrets/kubernetes.io/serviceaccount/ 2>/dev/null || echo "Token not mounted"
```

**What this does**:

- Creates a Service Account with token automounting disabled
- Creates a Pod that uses the Service Account
- Verifies that the token is not mounted in the Pod
- Shows how to disable token automounting at the Pod level, which takes precedence over the Service Account setting

</p>
</details>

## CKAD Exam Tips for Service Accounts

### Quick Reference for Service Accounts

#### Service Account - Imperative Commands

```bash
# Create a Service Account
kubectl create serviceaccount <name>

# Get Service Account details
kubectl get serviceaccount <name> -o yaml

# Delete a Service Account
kubectl delete serviceaccount <name>
```

#### Pod with Service Account - YAML Patterns

```yaml
# Pod using a specific Service Account
apiVersion: v1
kind: Pod
metadata:
  name: <pod-name>
spec:
  serviceAccountName: <service-account-name>
  containers:
  - name: <container-name>
    image: <image>
```

```yaml
# Pod with Service Account token automounting disabled
apiVersion: v1
kind: Pod
metadata:
  name: <pod-name>
spec:
  automountServiceAccountToken: false
  containers:
  - name: <container-name>
    image: <image>
```

### Common Service Account Patterns for CKAD

**1. Default Service Account:**
```yaml
# Pod implicitly using the default Service Account
apiVersion: v1
kind: Pod
metadata:
  name: default-sa-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
```

**2. Custom Service Account:**
```yaml
# Pod using a custom Service Account
apiVersion: v1
kind: Pod
metadata:
  name: custom-sa-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: nginx:1.21
```

**3. No Service Account Token:**
```yaml
# Pod with Service Account token automounting disabled
apiVersion: v1
kind: Pod
metadata:
  name: no-token-pod
spec:
  automountServiceAccountToken: false
  containers:
  - name: app
    image: nginx:1.21
```

### Imperative vs Declarative Approaches

**Imperative Approach:**
- Creating Service Accounts: `kubectl create serviceaccount <name>`
- Fast and simple for basic Service Account creation
- Limited to basic Service Account creation without additional configuration

**Declarative Approach:**
- Required for Service Accounts with automountServiceAccountToken set to false
- Required for Service Accounts with annotations or labels
- Required for Service Accounts with imagePullSecrets
- More flexible and provides complete control

**Hybrid Approach:**
```bash
# Generate a Service Account YAML template
kubectl create serviceaccount app-sa --dry-run=client -o yaml > app-sa.yaml

# Edit the YAML to add additional configuration
# Apply the configuration
kubectl apply -f app-sa.yaml
```

### CKAD Exam Strategy for Service Accounts

1. **Use imperative commands for basic Service Account creation**:
   ```bash
   kubectl create serviceaccount app-sa
   ```

2. **Use the hybrid approach for Service Accounts with additional configuration**:
   ```bash
   kubectl create serviceaccount app-sa --dry-run=client -o yaml > app-sa.yaml
   # Edit the YAML to add automountServiceAccountToken: false
   kubectl apply -f app-sa.yaml
   ```

3. **Verify Service Account configuration**:
   ```bash
   kubectl get serviceaccount <name> -o yaml
   ```

4. **Check if a Pod is using a specific Service Account**:
   ```bash
   kubectl get pod <pod-name> -o yaml | grep serviceAccount
   ```

5. **Test Service Account token access in a Pod**:
   ```bash
   kubectl exec -it <pod-name> -- ls -l /var/run/secrets/kubernetes.io/serviceaccount/
   ```

### Time-Saving Tips for the CKAD Exam

1. **Use imperative commands** for creating basic Service Accounts:
   ```bash
   kubectl create serviceaccount app-sa
   ```

2. **Remember that every Pod uses the default Service Account** unless specified otherwise.

3. **Use the `--serviceaccount` flag** when creating Pods imperatively:
   ```bash
   kubectl run app-pod --image=nginx --serviceaccount=app-sa
   ```

4. **Know how to disable token automounting** at both the Service Account and Pod levels:
   ```yaml
   # Service Account level
   automountServiceAccountToken: false
   
   # Pod level (takes precedence)
   spec:
     automountServiceAccountToken: false
   ```

5. **Remember that RBAC resources (Role, RoleBinding) cannot be created imperatively** in a useful way. Always use YAML for these resources.

6. **For the CKAD exam, focus on**:
   - Creating Service Accounts
   - Configuring Pods to use specific Service Accounts
   - Disabling token automounting
   - Understanding the basics of RBAC (Roles and RoleBindings)

7. **Common troubleshooting commands**:
   ```bash
   # Check if Service Account exists
   kubectl get serviceaccount
   
   # Check if Pod is using the correct Service Account
   kubectl get pod <pod-name> -o yaml | grep serviceAccount
   
   # Check if token is mounted
   kubectl exec -it <pod-name> -- ls -l /var/run/secrets/kubernetes.io/serviceaccount/
   ```
