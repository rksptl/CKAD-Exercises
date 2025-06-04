# Debugging Applications in Kubernetes

This section covers techniques for debugging applications running in Kubernetes.

## Key Resources

- [Debug Running Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/)
- [Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug/debug-application/)

## Introduction to Debugging

Debugging applications in Kubernetes requires understanding how to inspect running containers, analyze logs, and troubleshoot common issues.

## Key Concepts

- **Container Inspection**: Examining container state, logs, and environment
- **Pod Troubleshooting**: Diagnosing issues with Pod scheduling and execution
- **Service Debugging**: Resolving networking and service discovery problems
- **Ephemeral Debug Containers**: Running temporary containers in a Pod's namespace for debugging

## Debugging

### Exercise 1: Inspecting Pod Logs

**Goal**: Learn how to view and analyze container logs for troubleshooting.

**Task**: Deploy an application and examine its logs to identify issues.

**Why this matters**: Container logs are often the first place to look when troubleshooting application issues in Kubernetes. Understanding how to effectively retrieve and analyze logs is essential for identifying the root cause of problems and ensuring application reliability.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a problematic application**

Create a file named `problematic-app.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: problematic-app
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true; do
        echo "$(date) - Application encountered an error: Connection refused";
        echo "$(date) - Retrying in 5 seconds...";
        sleep 5;
      done
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f problematic-app.yaml
```

**Step 3: View the Pod logs**

```bash
kubectl logs problematic-app
```

**Step 4: Follow the logs in real-time**

```bash
kubectl logs -f problematic-app
```

**Step 5: View logs with timestamps**

```bash
kubectl logs --timestamps=true problematic-app
```

**Step 6: View logs from a specific time period**

```bash
kubectl logs --since=10m problematic-app
```

**Step 7: View only the most recent logs**

```bash
kubectl logs --tail=20 problematic-app
```

**What this does**:

- Creates a Pod that generates error messages in its logs
- Demonstrates different ways to view and filter logs:
  - Basic log retrieval
  - Following logs in real-time
  - Viewing logs with timestamps
  - Filtering logs by time
  - Viewing only the most recent logs
- This helps identify patterns and issues in application behavior

</p>
</details>

### Exercise 2: Debugging Pod States

**Goal**: Learn how to diagnose issues with Pods that are not running correctly.

**Task**: Troubleshoot Pods in different problematic states.

**Why this matters**: Understanding Pod states and how to diagnose issues when Pods are not running as expected is fundamental to Kubernetes troubleshooting. This skill allows you to quickly identify whether problems are related to image pulling, resource constraints, configuration errors, or other common issues.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with an invalid image**

Create a file named `invalid-image-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: invalid-image-pod
spec:
  containers:
  - name: app
    image: non-existent-image:latest
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f invalid-image-pod.yaml
```

**Step 3: Check the Pod status**

```bash
kubectl get pod invalid-image-pod
```

You should see the Pod in an `ImagePullBackOff` or `ErrImagePull` state.

**Step 4: Get detailed information about the Pod**

```bash
kubectl describe pod invalid-image-pod
```

Look for the events section, which should show errors related to pulling the image.

**Step 5: Create a Pod with insufficient resources**

Create a file named `resource-constrained-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-constrained-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    resources:
      requests:
        memory: "10Gi"
        cpu: "5"
```

**Step 6: Apply the Pod configuration**

```bash
kubectl apply -f resource-constrained-pod.yaml
```

**Step 7: Check the Pod status**

```bash
kubectl get pod resource-constrained-pod
```

You should see the Pod in a `Pending` state.

**Step 8: Get detailed information about the Pod**

```bash
kubectl describe pod resource-constrained-pod
```

Look for the events section, which should show errors related to insufficient resources.

**What this does**:

- Creates Pods with common issues:
  - Invalid image name
  - Resource requests that exceed available resources
- Demonstrates how to diagnose these issues:
  - Using `kubectl get pod` to check the Pod status
  - Using `kubectl describe pod` to get detailed information
  - Interpreting the events section to identify the root cause
- This helps troubleshoot Pods that are not running correctly

</p>
</details>

### Exercise 3: Using Debug Containers

**Goal**: Learn how to use ephemeral debug containers to troubleshoot running Pods.

**Task**: Attach a debug container to a running Pod to diagnose issues.

**Why this matters**: Debug containers provide a powerful way to troubleshoot issues in running Pods without modifying the original Pod specification. This is particularly useful for diagnosing issues in production environments where you can't restart or modify the Pod but need to investigate its behavior.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod to debug**

Create a file named `target-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: target-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f target-pod.yaml
```

**Step 3: Wait for the Pod to be running**

```bash
kubectl get pod target-pod --watch
```

**Step 4: Add a debug container to the Pod**

```bash
kubectl debug -it target-pod --image=busybox:1.36 --target=app
```

This will create an ephemeral container in the Pod and give you a shell.

**Step 5: Explore the container environment**

Inside the debug container, run the following commands:

```bash
# Check the processes running in the container
ps aux

# Check network connectivity
wget -O- localhost:80

# Check the file system
ls -la /

# Check environment variables
env

# Check DNS resolution
nslookup kubernetes.default.svc.cluster.local
```

**Step 6: Exit the debug container**

```bash
exit
```

**What this does**:

- Creates a Pod running an nginx container
- Attaches a debug container to the running Pod
- Uses the debug container to:
  - Inspect processes
  - Test network connectivity
  - Examine the file system
  - View environment variables
  - Test DNS resolution
- This helps diagnose issues in the running Pod without modifying it

</p>
</details>

### Exercise 4: Debugging Services and Networking

**Goal**: Learn how to troubleshoot Service and networking issues in Kubernetes.

**Task**: Diagnose and resolve common Service connectivity problems.

**Why this matters**: Networking issues are among the most common and challenging problems in Kubernetes. Understanding how to systematically debug Service connectivity ensures that your applications can communicate properly within the cluster, which is essential for microservices architectures.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment and Service**

Create a file named `service-debug.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app-wrong  # Intentionally wrong selector
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Apply the configuration**

```bash
kubectl apply -f service-debug.yaml
```

**Step 3: Check the Service**

```bash
kubectl get service web-service
```

**Step 4: Check the endpoints**

```bash
kubectl get endpoints web-service
```

You should see no endpoints because the selector doesn't match any Pods.

**Step 5: Diagnose the issue**

```bash
kubectl describe service web-service
```

Note the selector used by the Service.

**Step 6: Check the Pod labels**

```bash
kubectl get pods --show-labels
```

Note that the Pod labels don't match the Service selector.

**Step 7: Fix the Service**

Create a file named `service-fix.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app  # Corrected selector
  ports:
  - port: 80
    targetPort: 80
```

**Step 8: Apply the fix**

```bash
kubectl apply -f service-fix.yaml
```

**Step 9: Verify the endpoints**

```bash
kubectl get endpoints web-service
```

You should now see endpoints for the Service.

**Step 10: Test the Service**

```bash
kubectl run test-pod --image=busybox:1.36 --rm -it -- wget -qO- web-service
```

**What this does**:

- Creates a Deployment and Service with an intentional mismatch in selectors
- Demonstrates how to diagnose Service issues:
  - Checking if the Service exists
  - Checking if the Service has endpoints
  - Examining the Service selector
  - Verifying Pod labels
- Shows how to fix the issue by correcting the Service selector
- Tests the Service to ensure it's working properly
- This helps troubleshoot common Service connectivity issues

</p>
</details>

### Exercise 5: Debugging ConfigMaps and Secrets

**Goal**: Learn how to troubleshoot issues related to ConfigMaps and Secrets.

**Task**: Diagnose and resolve problems with ConfigMap and Secret mounting and usage.

**Why this matters**: ConfigMaps and Secrets are essential for managing application configuration and sensitive data in Kubernetes. Understanding how to debug issues with these resources ensures that your applications can access the configuration and credentials they need to function properly.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap and Secret**

```bash
kubectl create configmap app-config --from-literal=APP_ENV=production
kubectl create secret generic app-secret --from-literal=APP_PASSWORD=supersecret
```

**Step 2: Create a Pod that uses the ConfigMap and Secret**

Create a file named `config-debug.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-debug-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo Config: $APP_ENV, Secret: $APP_PASSWORD; sleep 3600"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config-wrong  # Intentionally wrong name
          key: APP_ENV
    - name: APP_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: APP_PASSWORD
```

**Step 3: Apply the Pod configuration**

```bash
kubectl apply -f config-debug.yaml
```

**Step 4: Check the Pod status**

```bash
kubectl get pod config-debug-pod
```

You should see the Pod in an error state.

**Step 5: Diagnose the issue**

```bash
kubectl describe pod config-debug-pod
```

Look for the events section, which should show errors related to the ConfigMap.

**Step 6: Fix the Pod configuration**

Create a file named `config-fix.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-debug-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "echo Config: $APP_ENV, Secret: $APP_PASSWORD; sleep 3600"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config  # Corrected name
          key: APP_ENV
    - name: APP_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: APP_PASSWORD
```

**Step 7: Apply the fix**

```bash
kubectl delete pod config-debug-pod
kubectl apply -f config-fix.yaml
```

**Step 8: Verify the Pod is running**

```bash
kubectl get pod config-debug-pod
```

**Step 9: Check the Pod logs**

```bash
kubectl logs config-debug-pod
```

You should see the environment variables correctly set.

**What this does**:

- Creates a ConfigMap and Secret
- Creates a Pod that uses them with an intentional error
- Demonstrates how to diagnose ConfigMap and Secret issues:
  - Checking the Pod status
  - Examining the Pod events
  - Identifying the incorrect ConfigMap reference
- Shows how to fix the issue by correcting the ConfigMap name
- Verifies that the Pod can access the ConfigMap and Secret data
- This helps troubleshoot common configuration issues

</p>
</details>

### Exercise 6: Debugging Resource Constraints

**Goal**: Learn how to identify and resolve resource-related issues in Kubernetes.

**Task**: Diagnose and fix problems related to CPU and memory constraints.

**Why this matters**: Resource constraints are a common source of performance issues and application failures in Kubernetes. Understanding how to identify when applications are hitting resource limits and how to adjust those limits appropriately is essential for maintaining application performance and reliability.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with tight resource constraints**

Create a file named `resource-debug.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-debug-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "64Mi"
        cpu: "100m"
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f resource-debug.yaml
```

**Step 3: Generate load on the Pod**

```bash
kubectl exec -it resource-debug-pod -- /bin/bash -c "apt-get update && apt-get install -y stress && stress --cpu 2 --vm 1 --vm-bytes 128M"
```

This will likely cause the container to be OOM killed.

**Step 4: Check the Pod status**

```bash
kubectl get pod resource-debug-pod
```

**Step 5: Check the Pod events**

```bash
kubectl describe pod resource-debug-pod
```

Look for events related to OOM killing or CPU throttling.

**Step 6: Check resource usage**

```bash
kubectl top pod resource-debug-pod
```

**Step 7: Fix the resource constraints**

Create a file named `resource-fix.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-debug-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    resources:
      requests:
        memory: "128Mi"
        cpu: "200m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

**Step 8: Apply the fix**

```bash
kubectl delete pod resource-debug-pod
kubectl apply -f resource-fix.yaml
```

**Step 9: Generate load again**

```bash
kubectl exec -it resource-debug-pod -- /bin/bash -c "apt-get update && apt-get install -y stress && stress --cpu 2 --vm 1 --vm-bytes 128M"
```

The container should now be able to handle the load.

**What this does**:

- Creates a Pod with tight resource constraints
- Generates load that exceeds those constraints
- Demonstrates how to diagnose resource issues:
  - Checking the Pod status
  - Examining the Pod events
  - Using `kubectl top` to monitor resource usage
- Shows how to fix the issue by increasing the resource limits
- Verifies that the Pod can handle the load with the new limits
- This helps troubleshoot common resource constraint issues

</p>
</details>

### Exercise 7: Using kubectl exec for Debugging

**Goal**: Learn how to use `kubectl exec` to run commands in containers for debugging.

**Task**: Use `kubectl exec` to diagnose issues inside a container.

**Why this matters**: The ability to execute commands directly inside a running container is a powerful debugging technique. It allows you to inspect the container environment, test connectivity, examine files, and run diagnostic tools without modifying the container image or restarting the Pod.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod to debug**

Create a file named `exec-debug.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-debug-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f exec-debug.yaml
```

**Step 3: Execute a simple command in the container**

```bash
kubectl exec exec-debug-pod -- ls -la /etc/nginx
```

**Step 4: Get an interactive shell in the container**

```bash
kubectl exec -it exec-debug-pod -- /bin/bash
```

**Step 5: Explore the container environment**

Inside the container, run the following commands:

```bash
# Check the processes
ps aux

# Check the network configuration
ip addr
netstat -tuln

# Check the file system
ls -la /etc/nginx
cat /etc/nginx/nginx.conf

# Check environment variables
env

# Test the web server
curl localhost

# Exit the shell
exit
```

**Step 6: Run a specific diagnostic command**

```bash
kubectl exec exec-debug-pod -- cat /var/log/nginx/error.log
```

**What this does**:

- Creates a Pod running an nginx container
- Demonstrates different ways to use `kubectl exec`:
  - Running a single command
  - Getting an interactive shell
  - Examining specific files
- Shows how to explore the container environment:
  - Checking processes
  - Examining network configuration
  - Viewing configuration files
  - Testing services
- This helps diagnose issues inside the container

</p>
</details>
