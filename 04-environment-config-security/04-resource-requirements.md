# Resource Requirements

This section covers Resource Requirements, Limits, and Quotas in Kubernetes, which are essential for efficient resource allocation and utilization.

## Key Resources

- [Resource Management Documentation](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Resource Quotas Documentation](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Limit Ranges Documentation](https://kubernetes.io/docs/concepts/policy/limit-range/)

## Resource Requirements

### Exercise 1: Setting Resource Requests and Limits

**Goal**: Learn how to set resource requests and limits for containers.

**Task**: Create a Pod with specific CPU and memory requests and limits.

**Why this matters**: Setting appropriate resource requests and limits ensures that your applications have the resources they need while preventing any single application from consuming too many resources and affecting other workloads on the cluster.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with resource requests and limits**

Option 1: Using imperative command:

```bash
# Create a pod with resource requests and limits using imperative command
kubectl run resource-demo --image=nginx \
  --requests=cpu=250m,memory=64Mi \
  --limits=cpu=500m,memory=128Mi
```

Option 2: Using a manifest file:

Create a file named `pod-resources.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: resource-demo-ctr
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

Apply the configuration:

```bash
kubectl apply -f pod-resources.yaml
```

**Step 2: Verify the resource settings**

```bash
kubectl describe pod resource-demo
```

Look for the "Containers" section, which should show the resource requests and limits.

**What this does**:

- `requests.memory: "64Mi"`: The container needs at least 64 MiB of memory to run
- `requests.cpu: "250m"`: The container needs at least 0.25 CPU cores to run
- `limits.memory: "128Mi"`: The container is not allowed to use more than 128 MiB of memory
- `limits.cpu: "500m"`: The container is not allowed to use more than 0.5 CPU cores

The scheduler uses the requests to find a node with enough resources, while the limits prevent the container from using more than the specified amount of resources.

</p>
</details>

### Exercise 2: Understanding Resource Units

**Goal**: Learn about the units used for CPU and memory resources.

**Task**: Create Pods with different resource unit specifications.

**Why this matters**: Understanding how Kubernetes measures and allocates resources is crucial for setting appropriate resource requests and limits. Different units can be used to express resource quantities, and knowing how to use them correctly ensures that your applications get the resources they need.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with CPU resources specified in different units**

> Note: For multi-container pods with different resource specifications, a manifest file is required as imperative commands cannot create multiple containers in a single pod.

For a single container with CPU resources:

```bash
# Create a pod with CPU resources specified in millicores
kubectl run cpu-millicores-demo --image=busybox --command -- sh -c "sleep 3600" \
  --requests=cpu=250m --limits=cpu=500m

# Create another pod with CPU resources specified in cores
kubectl run cpu-cores-demo --image=busybox --command -- sh -c "sleep 3600" \
  --requests=cpu=0.25 --limits=cpu=0.5
```

For multiple containers in a single pod (requires manifest):

Create a file named `cpu-units.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-units-demo
spec:
  containers:
  - name: cpu-millicores
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: "250m"  # 250 millicores = 0.25 CPU cores
      limits:
        cpu: "500m"  # 500 millicores = 0.5 CPU cores
  - name: cpu-cores
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: "0.25"  # 0.25 CPU cores = 250 millicores
      limits:
        cpu: "0.5"   # 0.5 CPU cores = 500 millicores
```

Apply the configuration:

```bash
kubectl apply -f cpu-units.yaml
```

**Step 2: Create a Pod with memory resources specified in different units**

Create a file named `memory-units.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-units-demo
spec:
  containers:
  - name: memory-bytes
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        memory: "67108864"  # 64 MiB in bytes
      limits:
        memory: "134217728"  # 128 MiB in bytes
  - name: memory-mebibytes
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        memory: "64Mi"  # 64 MiB
      limits:
        memory: "128Mi"  # 128 MiB
  - name: memory-gigabytes
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        memory: "0.064Gi"  # 0.064 GiB = 64 MiB
      limits:
        memory: "0.125Gi"  # 0.125 GiB = 128 MiB
```

Apply the configuration:

```bash
kubectl apply -f memory-units.yaml
```

**Step 3: Verify the resource settings**

```bash
kubectl describe pod cpu-units-demo
kubectl describe pod memory-units-demo
```

**What this does**:

- CPU units:
  - `250m` = 250 millicores = 0.25 CPU cores
  - `0.25` = 0.25 CPU cores = 250 millicores
- Memory units:
  - `67108864` = 64 MiB in bytes
  - `64Mi` = 64 MiB
  - `0.064Gi` = 0.064 GiB = 64 MiB

This demonstrates the different ways to specify CPU and memory resources in Kubernetes.

</p>
</details>

### Exercise 3: Resource Quotas

**Goal**: Learn how to set resource quotas for a namespace.

**Task**: Create a resource quota for a namespace and observe how it affects Pod creation.

**Why this matters**: Resource quotas help you manage and limit the total resource consumption in a namespace. This is important for multi-tenant clusters where you need to ensure fair resource allocation and prevent resource exhaustion.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace**

```bash
# This is an imperative command to create a namespace
kubectl create namespace quota-demo
```

**Step 2: Create a resource quota for the namespace**

Create a file named `resource-quota.yaml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: demo-quota
  namespace: quota-demo
spec:
  hard:
    pods: "5"
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

Apply the configuration:

```bash
kubectl apply -f resource-quota.yaml
```

**Step 3: Verify the resource quota**

```bash
kubectl describe resourcequota demo-quota -n quota-demo
```

**Step 4: Create a Pod that fits within the quota**

Create a file named `pod-within-quota.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: within-quota
  namespace: quota-demo
spec:
  containers:
  - name: main
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "400m"
```

Apply the configuration:

```bash
kubectl apply -f pod-within-quota.yaml
```

**Step 5: Create a Pod that exceeds the quota**

Create a file named `pod-exceeds-quota.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exceeds-quota
  namespace: quota-demo
spec:
  containers:
  - name: main
    image: nginx
    resources:
      requests:
        memory: "2Gi"
        cpu: "2"
      limits:
        memory: "4Gi"
        cpu: "4"
```

Apply the configuration:

```bash
kubectl apply -f pod-exceeds-quota.yaml
```

This should fail because it exceeds the quota.

**What this does**:

- Creates a namespace with a resource quota that limits:
  - The number of Pods to 5
  - Total CPU requests to 1 core
  - Total memory requests to 1 GiB
  - Total CPU limits to 2 cores
  - Total memory limits to 2 GiB
- Creates a Pod that fits within the quota
- Attempts to create a Pod that exceeds the quota, which fails

This demonstrates how resource quotas can be used to control resource usage in a namespace.

</p>
</details>

### Exercise 4: Limit Ranges

**Goal**: Learn how to set default resource limits and enforce minimum and maximum resource usage.

**Task**: Create a limit range for a namespace and observe how it affects Pod creation.

**Why this matters**: Limit ranges help you enforce resource constraints and set default values for containers that don't specify their own resource requests and limits. This ensures that all containers in a namespace have appropriate resource settings.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace**

```bash
kubectl create namespace limitrange-demo
```

**Step 2: Create a limit range for the namespace**

Create a file named `limit-range.yaml`:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: demo-limits
  namespace: limitrange-demo
spec:
  limits:
  - type: Container
    default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 128Mi
    min:
      cpu: 100m
      memory: 64Mi
    max:
      cpu: 1
      memory: 512Mi
```

Apply the configuration:

```bash
kubectl apply -f limit-range.yaml
```

**Step 3: Verify the limit range**

```bash
kubectl describe limitrange demo-limits -n limitrange-demo
```

**Step 4: Create a Pod without specifying resource requirements**

Create a file named `pod-no-resources.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-resources
  namespace: limitrange-demo
spec:
  containers:
  - name: main
    image: nginx
```

Apply the configuration:

```bash
kubectl apply -f pod-no-resources.yaml
```

**Step 5: Verify that the default resource requirements were applied**

```bash
kubectl describe pod no-resources -n limitrange-demo
```

Look for the "Containers" section, which should show the default resource requests and limits.

**Step 6: Create a Pod with resource requirements below the minimum**

Create a file named `pod-below-min.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: below-min
  namespace: limitrange-demo
spec:
  containers:
  - name: main
    image: nginx
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
```

Apply the configuration:

```bash
kubectl apply -f pod-below-min.yaml
```

This should fail because the resource requests are below the minimum.

**What this does**:

- Creates a namespace with a limit range that:
  - Sets default resource limits (500m CPU, 256Mi memory)
  - Sets default resource requests (200m CPU, 128Mi memory)
  - Sets minimum resource requirements (100m CPU, 64Mi memory)
  - Sets maximum resource limits (1 CPU, 512Mi memory)
- Creates a Pod without specifying resource requirements, which gets the default values
- Attempts to create a Pod with resource requirements below the minimum, which fails

This demonstrates how limit ranges can be used to enforce resource constraints and set default values.

</p>
</details>

### Exercise 5: Quality of Service (QoS) Classes

**Goal**: Learn about the different QoS classes and how they affect Pod eviction.

**Task**: Create Pods with different QoS classes and understand their implications.

**Why this matters**: Kubernetes assigns different QoS classes to Pods based on their resource specifications. These classes determine the order in which Pods are evicted when the node runs out of resources. Understanding QoS classes helps you design more reliable applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with Guaranteed QoS class**

Option 1: Using imperative command:

```bash
# Create a pod with Guaranteed QoS class (requests equal to limits)
kubectl run guaranteed-qos --image=nginx \
  --requests=memory=256Mi,cpu=500m \
  --limits=memory=256Mi,cpu=500m
```

Option 2: Using a manifest file:

Create a file named `guaranteed-qos.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-qos
spec:
  containers:
  - name: main
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

Apply the configuration:

```bash
kubectl apply -f guaranteed-qos.yaml
```

**Step 2: Create a Pod with Burstable QoS class**

Option 1: Using imperative command:

```bash
# Create a pod with Burstable QoS class (requests less than limits)
kubectl run burstable-qos --image=nginx \
  --requests=memory=128Mi,cpu=250m \
  --limits=memory=256Mi,cpu=500m
```

Option 2: Using a manifest file:

Create a file named `burstable-qos.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable-qos
spec:
  containers:
  - name: main
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

Apply the configuration:

```bash
kubectl apply -f burstable-qos.yaml
```

**Step 3: Create a Pod with BestEffort QoS class**

Option 1: Using imperative command (simplest approach):

```bash
# Create a pod with BestEffort QoS class (no resource requests or limits)
kubectl run besteffort-qos --image=nginx
```

Option 2: Using a manifest file:

Create a file named `besteffort-qos.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: besteffort-qos
spec:
  containers:
  - name: main
    image: nginx
```

Apply the configuration:

```bash
kubectl apply -f besteffort-qos.yaml
```

**Step 4: Verify the QoS classes**

```bash
kubectl get pods guaranteed-qos burstable-qos besteffort-qos -o custom-columns=NAME:.metadata.name,QOS:.status.qosClass
```

You should see:
- `guaranteed-qos` with QoS class `Guaranteed`
- `burstable-qos` with QoS class `Burstable`
- `besteffort-qos` with QoS class `BestEffort`

**What this does**:

- Creates a Pod with Guaranteed QoS class:
  - Memory requests equal to memory limits
  - CPU requests equal to CPU limits
- Creates a Pod with Burstable QoS class:
  - Memory requests less than memory limits
  - CPU requests less than CPU limits
- Creates a Pod with BestEffort QoS class:
  - No resource requests or limits specified

The QoS class affects the order in which Pods are evicted when the node runs out of resources:
1. BestEffort Pods are evicted first
2. Burstable Pods are evicted next
3. Guaranteed Pods are evicted last

This helps you understand how to design your applications for different levels of reliability.

</p>
</details>

## Best Practices

1. **Always specify resource requests and limits** for your containers to ensure they get the resources they need and don't consume too many resources.
2. **Set realistic resource requests** based on actual application needs to avoid over-provisioning or under-provisioning.
3. **Use resource quotas** to prevent a single namespace from consuming all the resources in the cluster.
4. **Use limit ranges** to enforce minimum and maximum resource usage and set default values.
5. **Consider the QoS class** when designing your applications, especially for critical workloads that should not be evicted.
6. **Monitor actual resource usage** and adjust requests and limits accordingly.
7. **Be aware of the units** used for CPU and memory resources to avoid mistakes.
8. **Use horizontal pod autoscaling** to automatically adjust the number of Pods based on resource usage.

## CKAD Exam Tips for Resource Requirements

### Quick Reference for Resource Requirements

```yaml
# Pod with resource requests and limits
spec:
  containers:
  - name: app
    resources:
      requests:              # Minimum resources needed
        cpu: 250m            # 250 millicores = 0.25 cores
        memory: 64Mi         # 64 Mebibytes
      limits:                # Maximum resources allowed
        cpu: 500m            # 500 millicores = 0.5 cores
        memory: 128Mi        # 128 Mebibytes
```

```yaml
# ResourceQuota for namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
spec:
  hard:
    pods: "10"               # Max 10 pods
    requests.cpu: "4"        # Max 4 CPU cores total requests
    requests.memory: 8Gi     # Max 8 GiB memory total requests
    limits.cpu: "8"          # Max 8 CPU cores total limits
    limits.memory: 16Gi      # Max 16 GiB memory total limits
```

```yaml
# LimitRange for default values
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - type: Container
    default:                 # Default limits if not specified
      cpu: 500m
      memory: 256Mi
    defaultRequest:          # Default requests if not specified
      cpu: 200m
      memory: 128Mi
    min:                     # Minimum allowed values
      cpu: 100m
      memory: 64Mi
    max:                     # Maximum allowed values
      cpu: 1
      memory: 512Mi
```

### Imperative Command Examples

**Creating Pods with Resource Requirements:**

```bash
# Basic pod with requests and limits
kubectl run nginx --image=nginx --restart=Never \
  --requests=cpu=250m,memory=64Mi \
  --limits=cpu=500m,memory=128Mi

# Pod with only requests (Burstable QoS)
kubectl run nginx-burstable --image=nginx --restart=Never \
  --requests=cpu=250m,memory=64Mi

# Pod with equal requests and limits (Guaranteed QoS)
kubectl run nginx-guaranteed --image=nginx --restart=Never \
  --requests=cpu=500m,memory=128Mi \
  --limits=cpu=500m,memory=128Mi

# Pod with no resource specifications (BestEffort QoS)
kubectl run nginx-besteffort --image=nginx --restart=Never
```

**Creating ResourceQuotas:**

```bash
# Create a namespace with a resource quota
kubectl create namespace team-a
kubectl create quota team-a-quota --namespace=team-a \
  --hard=pods=10,requests.cpu=4,requests.memory=8Gi,limits.cpu=8,limits.memory=16Gi
```

**Creating LimitRanges:**

```bash
# LimitRanges must be created using YAML manifests
# There's no direct imperative command for complex LimitRange creation
# Use the hybrid approach instead:

# Generate a basic LimitRange template
kubectl create -f - <<EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: team-a
spec:
  limits:
  - type: Container
    default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 128Mi
EOF
```

### Common Resource Patterns for CKAD

**1. Web Application Pattern:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: 250m
        memory: 64Mi
      limits:
        cpu: 500m
        memory: 128Mi
```

**2. Database Pattern (higher resources):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: database
spec:
  containers:
  - name: postgres
    image: postgres
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1
        memory: 1Gi
```

**3. Multi-Container Pod with Different Resource Needs:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: 250m
        memory: 64Mi
      limits:
        cpu: 500m
        memory: 128Mi
  - name: sidecar
    image: fluentd
    resources:
      requests:
        cpu: 100m
        memory: 32Mi
      limits:
        cpu: 200m
        memory: 64Mi
```

### CKAD Exam Strategy for Resource Requirements

1. **Know your resource units**:
   - CPU: `m` for millicores (1000m = 1 CPU core)
   - Memory: `Ki`, `Mi`, `Gi` for binary units (1024-based)
   - Memory: `K`, `M`, `G` for decimal units (1000-based)

2. **Understand QoS classes**:
   - **Guaranteed**: requests = limits for both CPU and memory
   - **Burstable**: requests < limits for either CPU or memory
   - **BestEffort**: no requests or limits specified

3. **Verify resource settings**:
   ```bash
   # Check resource settings for a pod
   kubectl describe pod pod-name | grep -A 8 Resources
   
   # Check QoS class
   kubectl get pod pod-name -o jsonpath='{.status.qosClass}'
   
   # Check resource quota usage
   kubectl describe quota -n namespace-name
   ```

4. **Troubleshoot resource issues**:
   ```bash
   # Check if pod is pending due to insufficient resources
   kubectl describe pod pod-name
   
   # Check if pod is being throttled due to CPU limits
   kubectl top pod pod-name
   
   # Check if pod is being OOM killed due to memory limits
   kubectl logs pod-name --previous
   ```

### Time-Saving Tips for the CKAD Exam

1. **Use the imperative approach** for creating pods with resource requirements:
   ```bash
   kubectl run nginx --image=nginx --restart=Never \
     --requests=cpu=250m,memory=64Mi \
     --limits=cpu=500m,memory=128Mi
   ```

2. **Use the hybrid approach** for more complex resource configurations:
   ```bash
   # Generate YAML template
   kubectl run nginx --image=nginx --restart=Never \
     --requests=cpu=250m,memory=64Mi \
     --dry-run=client -o yaml > pod.yaml
   
   # Edit the YAML to add more complex resource settings
   # Apply the configuration
   kubectl apply -f pod.yaml
   ```

3. **Remember common resource values** that might be needed in the exam:
   - Small container: 100m CPU, 64Mi memory
   - Medium container: 250m CPU, 128Mi memory
   - Large container: 500m CPU, 256Mi memory

4. **Know the relationship between ResourceQuotas and resource requirements**:
   - Pods without resource requirements will be rejected in namespaces with ResourceQuotas
   - ResourceQuotas enforce aggregate resource usage across all pods in a namespace

5. **Remember that LimitRanges apply to new resources only**:
   - Existing pods are not affected when you create or modify a LimitRange
   - LimitRanges can provide default values, which simplifies pod creation

6. **Use meaningful names** for your resource-related objects:
   - `small-nginx-pod`
   - `team-a-quota`
   - `default-limits`

7. **Be aware of resource validation timing**:
   - Resource requests and limits are validated at pod creation time
   - ResourceQuotas are enforced at creation time
   - LimitRanges are applied at creation time
