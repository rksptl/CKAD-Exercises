# Security Contexts

This section covers Security Contexts in Kubernetes, which define privilege and access control settings for Pods and containers.

## Key Resources

- [Security Context Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

## Introduction to Security Contexts

Security Contexts allow you to define privilege and access control settings for Pods and containers. They include settings for:

- User and group IDs
- Linux capabilities
- SELinux context
- AppArmor and seccomp profiles
- Privilege escalation controls
- Read-only root filesystem

## Security Contexts

### Exercise 1: Setting User and Group IDs

**Goal**: Learn how to run containers as non-root users.

**Task**: Create a Pod that runs with a specific user ID and group ID.

**Why this matters**: Running containers as non-root users is a security best practice that reduces the risk of container breakout attacks. If a container is compromised, the attacker will have limited privileges on the host system.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a security context that specifies user and group IDs**

> Note: Security contexts with multiple settings like user IDs, group IDs, and volume mounts are best defined using manifest files. While basic security settings can be set imperatively, complex security configurations require the declarative approach.

Option 1: Using imperative command for a simple security context (limited functionality):

```bash
# Create a pod with a specific user ID (limited security context options)
kubectl run security-context-demo --image=busybox --restart=Never \
  --overrides='{"spec":{"securityContext":{"runAsUser":1000}}}' \
  -- sh -c "sleep 3600"
```

Option 2: Using a manifest file (recommended for complete security context):

Create a file named `pod-user-context.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /data/demo
  volumes:
  - name: data-volume
    emptyDir: {}
```

Apply the configuration:

```bash
kubectl apply -f pod-user-context.yaml
```

**Step 2: Verify the user and group IDs**

```bash
kubectl exec security-context-demo -- id
```

You should see output similar to:

```
uid=1000 gid=3000 groups=2000,3000
```

**Step 3: Check the ownership of the mounted volume**

```bash
kubectl exec security-context-demo -- ls -la /data
```

You should see that the `/data/demo` directory is owned by user 1000 and group 2000.

**What this does**:

- `runAsUser: 1000`: Runs the container processes as user ID 1000
- `runAsGroup: 3000`: Runs the container processes with primary group ID 3000
- `fsGroup: 2000`: Any volumes mounted will be owned by group ID 2000

This ensures that the container runs with non-root privileges and that any files created in the volume have the correct ownership.

</p>
</details>

### Exercise 2: Container-Level Security Context

**Goal**: Learn how to set security contexts at the container level.

**Task**: Create a Pod with different security contexts for each container.

**Why this matters**: Different containers within the same Pod may have different security requirements. Container-level security contexts allow you to apply specific security settings to individual containers.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with different security contexts for each container**

> Note: Multi-container pods with different security contexts must be created using YAML manifests as there are no imperative commands that can set different security contexts for multiple containers in a single pod.

Create a file named `pod-container-context.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-security
spec:
  containers:
  - name: first
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
  - name: second
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```

Apply the configuration:

```bash
kubectl apply -f pod-container-context.yaml
```

**Step 2: Verify the user ID for the first container**

```bash
kubectl exec multi-container-security -c first -- id
```

You should see output showing user ID 1000.

**Step 3: Verify the user ID for the second container**

```bash
kubectl exec multi-container-security -c second -- id
```

You should see output showing user ID 2000.

**Step 4: Check capabilities for the first container**

```bash
kubectl exec multi-container-security -c first -- grep Cap /proc/1/status
```

You should see that the container has the `NET_ADMIN` and `SYS_TIME` capabilities.

**What this does**:

- First container:
  - Runs as user ID 1000
  - Has additional Linux capabilities `NET_ADMIN` and `SYS_TIME`
- Second container:
  - Runs as user ID 2000
  - Prevents privilege escalation

This demonstrates how to apply different security settings to containers within the same Pod.

</p>
</details>

### Exercise 3: Preventing Privilege Escalation

**Goal**: Learn how to prevent privilege escalation in containers.

**Task**: Create a Pod that prevents privilege escalation.

**Why this matters**: Privilege escalation allows a process to gain more privileges than its parent process. Preventing privilege escalation is a security best practice that limits the potential damage if a container is compromised.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod that prevents privilege escalation**

Create a file named `pod-no-privilege-escalation.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-privilege-escalation
spec:
  containers:
  - name: main
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
```

Apply the configuration:

```bash
kubectl apply -f pod-no-privilege-escalation.yaml
```

**Step 2: Verify that privilege escalation is prevented**

```bash
kubectl exec no-privilege-escalation -- grep NoNewPrivs /proc/1/status
```

You should see `NoNewPrivs: 1`, indicating that privilege escalation is prevented.

**What this does**:

- `allowPrivilegeEscalation: false`: Ensures that no child process of the container can gain more privileges than its parent

This is an important security setting that helps prevent certain types of container breakout attacks.

</p>
</details>

### Exercise 4: Read-Only Root Filesystem

**Goal**: Learn how to make the root filesystem read-only.

**Task**: Create a Pod with a read-only root filesystem.

**Why this matters**: A read-only root filesystem prevents attackers from modifying the container's filesystem, which can help prevent certain types of attacks and ensure container immutability.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a read-only root filesystem**

Create a file named `pod-readonly-fs.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-fs
spec:
  containers:
  - name: main
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: tmp-volume
      mountPath: /tmp
    - name: var-run-volume
      mountPath: /var/run
    - name: var-cache-nginx
      mountPath: /var/cache/nginx
  volumes:
  - name: tmp-volume
    emptyDir: {}
  - name: var-run-volume
    emptyDir: {}
  - name: var-cache-nginx
    emptyDir: {}
```

Apply the configuration:

```bash
kubectl apply -f pod-readonly-fs.yaml
```

**Step 2: Verify that the root filesystem is read-only**

```bash
kubectl exec readonly-fs -- touch /test-file
```

This should fail with a "read-only file system" error.

**Step 3: Verify that the mounted volumes are writable**

```bash
kubectl exec readonly-fs -- touch /tmp/test-file
```

This should succeed because the `/tmp` directory is mounted as a writable volume.

**What this does**:

- `readOnlyRootFilesystem: true`: Makes the container's root filesystem read-only
- Mounts writable volumes for directories that need to be writable (`/tmp`, `/var/run`, `/var/cache/nginx`)

This ensures that the container's filesystem cannot be modified, which improves security and enforces immutability.

</p>
</details>

### Exercise 5: Linux Capabilities

**Goal**: Learn how to add or drop Linux capabilities.

**Task**: Create a Pod with specific Linux capabilities.

**Why this matters**: Linux capabilities provide fine-grained control over privileged operations. By adding only the specific capabilities that a container needs, you can reduce the security risk compared to running the container with full root privileges.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with specific Linux capabilities**

Create a file named `pod-capabilities.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capabilities-demo
spec:
  containers:
  - name: main
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
        drop: ["CHOWN", "MKNOD"]
```

Apply the configuration:

```bash
kubectl apply -f pod-capabilities.yaml
```

**Step 2: Verify the capabilities**

```bash
kubectl exec capabilities-demo -- grep Cap /proc/1/status
```

You should see that the container has the `NET_ADMIN` and `SYS_TIME` capabilities, but not `CHOWN` and `MKNOD`.

**Step 3: Test the NET_ADMIN capability**

```bash
kubectl exec capabilities-demo -- ip link set lo down
```

This should succeed because the container has the `NET_ADMIN` capability.

**Step 4: Test the dropped CHOWN capability**

```bash
kubectl exec capabilities-demo -- chown 1000:1000 /etc/passwd
```

This should fail because the container does not have the `CHOWN` capability.

**What this does**:

- `capabilities.add`: Adds specific Linux capabilities to the container
- `capabilities.drop`: Removes specific Linux capabilities from the container

This allows you to fine-tune the privileges that a container has, following the principle of least privilege.

</p>
</details>

### Exercise 6: SELinux Context

**Goal**: Learn how to set SELinux context for a Pod.

**Task**: Create a Pod with a specific SELinux context.

**Why this matters**: SELinux provides mandatory access control for processes, which can help prevent unauthorized access to resources. Setting an SELinux context for a Pod ensures that it runs with the appropriate security constraints.

<details><summary>show solution</summary>
<p>

**Note**: This exercise requires a Kubernetes cluster with SELinux enabled.

**Step 1: Create a Pod with an SELinux context**

Create a file named `pod-selinux.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selinux-demo
spec:
  securityContext:
    seLinuxOptions:
      level: "s0:c123,c456"
  containers:
  - name: main
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

Apply the configuration:

```bash
kubectl apply -f pod-selinux.yaml
```

**Step 2: Verify the SELinux context**

```bash
kubectl exec selinux-demo -- cat /proc/self/attr/current
```

If SELinux is enabled, you should see the SELinux context.

**What this does**:

- `seLinuxOptions.level: "s0:c123,c456"`: Sets the SELinux level for the Pod

This ensures that the Pod runs with the specified SELinux context, which can help enforce security policies.

</p>
</details>

### Exercise 7: Combining Security Contexts

**Goal**: Learn how to combine multiple security context settings.

**Task**: Create a Pod with a comprehensive security context.

**Why this matters**: Real-world applications often require multiple security settings to be applied together. Understanding how to combine these settings is important for securing your applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a comprehensive security context**

> Note: Complex security configurations with multiple settings at both pod and container levels must be created using YAML manifests. The imperative approach is not suitable for this level of complexity.

Create a file named `pod-comprehensive-security.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: main
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
    volumeMounts:
    - name: tmp-volume
      mountPath: /tmp
    - name: var-run-volume
      mountPath: /var/run
    - name: var-cache-nginx
      mountPath: /var/cache/nginx
  volumes:
  - name: tmp-volume
    emptyDir: {}
  - name: var-run-volume
    emptyDir: {}
  - name: var-cache-nginx
    emptyDir: {}
```

Apply the configuration:

```bash
kubectl apply -f pod-comprehensive-security.yaml
```

> Note: For the CKAD exam, it's important to understand that security contexts are typically defined in YAML manifests. While basic pods can be created imperatively, security settings generally require the declarative approach.

**Step 2: Verify the user and group IDs**

```bash
kubectl exec secure-pod -- id
```

**Step 3: Verify that privilege escalation is prevented**

```bash
kubectl exec secure-pod -- grep NoNewPrivs /proc/1/status
```

**Step 4: Verify that the root filesystem is read-only**

```bash
kubectl exec secure-pod -- touch /test-file
```

**Step 5: Verify the capabilities**

```bash
kubectl exec secure-pod -- grep Cap /proc/1/status
```

**What this does**:

- Pod-level security context:
  - Runs as user ID 1000
  - Runs with primary group ID 3000
  - Sets fsGroup to 2000
- Container-level security context:
  - Prevents privilege escalation
  - Makes the root filesystem read-only
  - Drops all capabilities except `NET_BIND_SERVICE`
- Mounts writable volumes for directories that need to be writable

This creates a Pod with strong security settings that follows security best practices.

</p>
</details>

## Best Practices

1. **Run containers as non-root users** whenever possible.
2. **Use a read-only root filesystem** to prevent modifications to the container's filesystem.
3. **Prevent privilege escalation** by setting `allowPrivilegeEscalation: false`.
4. **Drop unnecessary Linux capabilities** and only add those that are required.
5. **Use Pod Security Standards** to enforce security best practices across your cluster.
6. **Apply the principle of least privilege** by giving containers only the permissions they need.
7. **Use security contexts in combination with other security features** like NetworkPolicies, RBAC, and PodSecurityPolicies.
8. **Regularly audit and review security contexts** to ensure they align with your security requirements.

## CKAD Exam Tips for Security Contexts

### Quick Reference for Security Context Settings

```yaml
# Pod-level security context
spec:
  securityContext:
    runAsUser: 1000                # User ID to run all containers
    runAsGroup: 3000               # Primary group ID
    fsGroup: 2000                  # Group ID for volume ownership
    supplementalGroups: [5555,6666] # Additional group memberships
    sysctls:                       # Kernel parameters
    - name: net.ipv4.ip_forward
      value: "1"

# Container-level security context
spec:
  containers:
  - name: app
    securityContext:
      runAsUser: 1000              # Override pod-level setting
      runAsNonRoot: true           # Prevent running as root
      allowPrivilegeEscalation: false # Prevent privilege escalation
      privileged: false            # Don't run with elevated privileges
      readOnlyRootFilesystem: true # Make root filesystem read-only
      capabilities:
        add: ["NET_BIND_SERVICE"]  # Add specific capabilities
        drop: ["ALL"]             # Drop capabilities
      seLinuxOptions:              # SELinux context
        level: "s0:c123,c456"
```

### Imperative vs. Declarative Approaches for Security Contexts

**Important Note for CKAD Exam:** While basic pods can be created imperatively, security contexts with multiple settings are best defined using declarative YAML manifests.

**Limited Imperative Options:**

```bash
# Create a pod with a basic user ID security context
kubectl run secure-pod --image=nginx --restart=Never \
  --overrides='{"spec":{"securityContext":{"runAsUser":1000}}}' \
  -- sleep 3600

# Create a pod with container-level security context
kubectl run secure-pod --image=nginx --restart=Never \
  --overrides='{"spec":{"containers":[{"name":"secure-pod","image":"nginx","securityContext":{"capabilities":{"add":["NET_ADMIN"]}}}]}}' \
  -- sleep 3600
```

**Hybrid Approach (Recommended for CKAD):**

```bash
# Generate a basic pod YAML template
kubectl run secure-pod --image=nginx --restart=Never --dry-run=client -o yaml > secure-pod.yaml

# Then edit the YAML to add security context settings
# This is the most practical approach for the exam
```

### Common Security Context Patterns for CKAD

**1. Non-Root User Pattern:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
```

**2. Read-Only Root Filesystem with Writable Volumes:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

**3. Dropping All Capabilities and Adding Specific Ones:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limited-capabilities-pod
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      capabilities:
        drop: ["ALL"]
        add: ["NET_BIND_SERVICE"]
```

### CKAD Exam Strategy for Security Contexts

1. **Remember the hierarchy**:
   - Container-level security context settings override Pod-level settings
   - Not all settings can be specified at both levels

2. **Test your security settings**:
   ```bash
   # Check user/group IDs
   kubectl exec pod-name -- id
   
   # Check if filesystem is read-only
   kubectl exec pod-name -- touch /test-file
   
   # Check Linux capabilities
   kubectl exec pod-name -- grep Cap /proc/1/status
   ```

3. **Troubleshoot security issues**:
   ```bash
   # Check pod status for security-related errors
   kubectl describe pod pod-name
   
   # Check container logs
   kubectl logs pod-name
   ```

### Time-Saving Tips for the CKAD Exam

1. **Use the hybrid approach** for creating pods with security contexts:
   ```bash
   # Generate YAML template
   kubectl run secure-pod --image=nginx --dry-run=client -o yaml > pod.yaml
   
   # Edit the YAML to add security context
   # Apply the configuration
   kubectl apply -f pod.yaml
   ```

2. **Remember common capability names** that might be needed in the exam:
   - `NET_ADMIN`: Network administration
   - `NET_BIND_SERVICE`: Bind to privileged ports (< 1024)
   - `SYS_TIME`: Modify system time
   - `CHOWN`: Change file ownership
   - `DAC_OVERRIDE`: Bypass file permission checks

3. **Know which directories need to be writable** when using a read-only root filesystem:
   - `/tmp`: Temporary files
   - `/var/run`: Runtime files
   - `/var/log`: Log files
   - Application-specific directories (e.g., `/var/cache/nginx` for NGINX)

4. **Use meaningful names** for your security-related resources:
   - `readonly-nginx-pod`
   - `nonroot-web-pod`
   - `secure-app-pod`

5. **Remember that security contexts are validated at runtime**:
   - A pod might be created successfully but fail to start if security settings are incompatible with the container
   - Always check pod status and logs if a pod with security context doesn't start
