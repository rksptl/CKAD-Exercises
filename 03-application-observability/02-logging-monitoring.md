# Logging and Monitoring

This section covers Kubernetes logging and monitoring capabilities, which are essential for observing application behavior and troubleshooting issues.

## Key Resources

- [Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [Tools for Monitoring Resources](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/)

## Introduction to Logging and Monitoring

Effective logging and monitoring are critical for understanding application behavior, troubleshooting issues, and ensuring optimal performance in Kubernetes environments.

## Key Concepts

- **Container Logs**: Output streams from containers captured by Kubernetes
- **Structured Logging**: JSON or other formatted logs that are machine-parseable
- **Log Aggregation**: Collecting logs from multiple sources into a central system
- **Metrics**: Numerical data points that represent system or application state
- **Resource Monitoring**: Tracking CPU, memory, and other resource usage
- **Prometheus**: Popular monitoring system and time series database for Kubernetes
- **Grafana**: Visualization tool often used with Prometheus for dashboarding

## Logging and Monitoring

### Exercise 1: Accessing Container Logs

**Goal**: Learn how to access and analyze container logs in Kubernetes.

**Task**: Deploy an application and view its logs using kubectl.

**Why this matters**: Container logs are the primary source of information for troubleshooting application issues in Kubernetes. Understanding how to effectively retrieve and analyze logs is essential for identifying the root cause of problems and ensuring application reliability.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod that generates logs**

Create a file named `logging-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logging-pod
spec:
  containers:
  - name: counter
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      i=0;
      while true;
      do
        echo "$(date) - INFO: Counter: $i";
        i=$((i+1));
        if [ $((i % 10)) -eq 0 ]; then
          echo "$(date) - WARN: Counter reached multiple of 10: $i";
        fi;
        if [ $((i % 50)) -eq 0 ]; then
          echo "$(date) - ERROR: Counter reached multiple of 50: $i";
        fi;
        sleep 1;
      done
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f logging-pod.yaml
```

**Step 3: View the Pod logs**

```bash
kubectl logs logging-pod
```

**Step 4: Follow the logs in real-time**

```bash
kubectl logs -f logging-pod
```

Press Ctrl+C to exit the follow mode.

**Step 5: View logs with timestamps**

```bash
kubectl logs --timestamps=true logging-pod
```

**Step 6: Filter logs by pattern**

```bash
kubectl logs logging-pod | grep ERROR
```

**Step 7: View logs from the last 5 minutes**

```bash
kubectl logs --since=5m logging-pod
```

**Step 8: View only the most recent logs**

```bash
kubectl logs --tail=20 logging-pod
```

**What this does**:

- Creates a Pod that generates different types of log messages
- Demonstrates different ways to view and filter logs:
  - Basic log retrieval
  - Following logs in real-time
  - Viewing logs with timestamps
  - Filtering logs by pattern
  - Filtering logs by time
  - Viewing only the most recent logs
- This helps identify patterns and issues in application behavior

</p>
</details>

### Exercise 2: Multi-Container Pod Logging

**Goal**: Learn how to access logs from specific containers in multi-container Pods.

**Task**: Deploy a Pod with multiple containers and view logs from each container.

**Why this matters**: Multi-container Pods are a common pattern in Kubernetes for implementing sidecars, adapters, and other container design patterns. Understanding how to access logs from specific containers within a Pod is essential for troubleshooting issues in these more complex deployments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with multiple containers**

Create a file named `multi-container-logging.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: main-app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "$(date) - Main application log";
        sleep 5;
      done
  - name: sidecar
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "$(date) - Sidecar container log";
        sleep 3;
      done
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f multi-container-logging.yaml
```

**Step 3: View logs from the main container**

```bash
kubectl logs multi-container-pod -c main-app
```

**Step 4: View logs from the sidecar container**

```bash
kubectl logs multi-container-pod -c sidecar
```

**Step 5: Follow logs from a specific container**

```bash
kubectl logs -f multi-container-pod -c main-app
```

Press Ctrl+C to exit the follow mode.

**Step 6: View logs from all containers**

```bash
kubectl logs multi-container-pod --all-containers=true
```

**Step 7: View logs from previous container instances**

If a container has been restarted:

```bash
kubectl logs multi-container-pod -c main-app --previous
```

**What this does**:

- Creates a Pod with two containers that generate different logs
- Demonstrates how to view logs from specific containers:
  - Using the `-c` flag to specify the container name
  - Using `--all-containers=true` to view logs from all containers
  - Using `--previous` to view logs from previous container instances
- This helps troubleshoot issues in multi-container Pods

</p>
</details>

### Exercise 3: Structured Logging

**Goal**: Learn how to generate and parse structured logs in JSON format.

**Task**: Deploy an application that generates structured logs and extract specific fields.

**Why this matters**: Structured logging makes it easier to parse, filter, and analyze logs programmatically. By outputting logs in a structured format like JSON, you can more effectively search for specific events, extract relevant information, and integrate with log aggregation systems.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod that generates structured logs**

Create a file named `structured-logging.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: structured-logging
spec:
  containers:
  - name: logger
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        timestamp=$(date -Iseconds);
        level="INFO";
        component="user-service";
        message="User login successful";
        user_id=$((RANDOM % 1000));
        
        if [ $((RANDOM % 10)) -eq 0 ]; then
          level="WARN";
          message="Failed login attempt";
        fi;
        
        if [ $((RANDOM % 50)) -eq 0 ]; then
          level="ERROR";
          message="Database connection failed";
          component="database";
        fi;
        
        echo "{\"timestamp\":\"$timestamp\",\"level\":\"$level\",\"component\":\"$component\",\"message\":\"$message\",\"user_id\":$user_id}";
        sleep 1;
      done
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f structured-logging.yaml
```

**Step 3: View the structured logs**

```bash
kubectl logs structured-logging
```

**Step 4: Extract specific fields using jq**

First, install jq if it's not already installed:

```bash
# This is for demonstration purposes - in a real environment, you'd use a tool like jq
kubectl logs structured-logging | grep ERROR
```

**Step 5: Filter logs by log level**

```bash
kubectl logs structured-logging | grep "\"level\":\"ERROR\""
```

**Step 6: Filter logs by component**

```bash
kubectl logs structured-logging | grep "\"component\":\"database\""
```

**What this does**:

- Creates a Pod that generates structured logs in JSON format
- Demonstrates how to view and filter structured logs:
  - Viewing the raw JSON logs
  - Filtering logs by specific fields
  - Extracting information from structured logs
- This helps implement more sophisticated logging strategies

</p>
</details>

### Exercise 4: Monitoring Pod Resource Usage

**Goal**: Learn how to monitor CPU and memory usage of Pods.

**Task**: Deploy resource-intensive applications and monitor their resource usage.

**Why this matters**: Monitoring resource usage is essential for optimizing application performance, planning capacity, and identifying resource bottlenecks. Understanding how to track CPU and memory usage helps ensure that applications have the resources they need without wasting cluster capacity.

<details><summary>show solution</summary>
<p>

**Step 1: Deploy the Metrics Server**

Note: In a real Kubernetes cluster, you would need to deploy the Metrics Server if it's not already installed. For the purpose of this exercise, we'll assume it's already installed.

**Step 2: Create Pods with different resource profiles**

Create a file named `resource-monitoring.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-intensive
  labels:
    app: resource-demo
spec:
  containers:
  - name: cpu-load
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true; do
        for i in $(seq 1 10000); do
          echo "$i" > /dev/null;
        done;
      done
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
---
apiVersion: v1
kind: Pod
metadata:
  name: memory-intensive
  labels:
    app: resource-demo
spec:
  containers:
  - name: memory-load
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true; do
        dd if=/dev/zero of=/tmp/file bs=1M count=50;
        sleep 10;
        rm /tmp/file;
        sleep 5;
      done
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

**Step 3: Apply the Pod configurations**

```bash
kubectl apply -f resource-monitoring.yaml
```

**Step 4: Monitor Pod resource usage**

```bash
kubectl top pods
```

**Step 5: Monitor specific Pods**

```bash
kubectl top pod cpu-intensive
kubectl top pod memory-intensive
```

**Step 6: Monitor all Pods with a specific label**

```bash
kubectl top pods -l app=resource-demo
```

**Step 7: Monitor container-level resource usage**

```bash
kubectl top pods --containers=true
```

**What this does**:

- Creates Pods that consume different types of resources:
  - CPU-intensive Pod
  - Memory-intensive Pod
- Demonstrates how to monitor resource usage:
  - Using `kubectl top pods` to view Pod-level metrics
  - Filtering by Pod name or label
  - Viewing container-level metrics
- This helps identify resource bottlenecks and optimize resource allocation

</p>
</details>

### Exercise 5: Implementing Custom Metrics

**Goal**: Learn how to expose and collect custom application metrics.

**Task**: Deploy an application that exposes custom metrics and collect them.

**Why this matters**: While built-in CPU and memory metrics are important, custom metrics allow you to monitor application-specific indicators that are more directly related to business goals and user experience. Implementing custom metrics enables more comprehensive monitoring and better-informed scaling decisions.

<details><summary>show solution</summary>
<p>

**Step 1: Create a simple application that exposes metrics**

Note: In a real scenario, you would use Prometheus client libraries to expose metrics from your application. For this exercise, we'll simulate this with a simple script.

Create a file named `custom-metrics.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-metrics-demo
  labels:
    app: metrics-demo
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
spec:
  containers:
  - name: metrics-app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true; do
        mkdir -p /tmp/metrics;
        requests=$((RANDOM % 100));
        errors=$((RANDOM % 10));
        latency=$((RANDOM % 1000));
        
        echo "# HELP http_requests_total Total number of HTTP requests" > /tmp/metrics/index.html;
        echo "# TYPE http_requests_total counter" >> /tmp/metrics/index.html;
        echo "http_requests_total{method=\"get\"} $requests" >> /tmp/metrics/index.html;
        
        echo "# HELP http_errors_total Total number of HTTP errors" >> /tmp/metrics/index.html;
        echo "# TYPE http_errors_total counter" >> /tmp/metrics/index.html;
        echo "http_errors_total{code=\"500\"} $errors" >> /tmp/metrics/index.html;
        
        echo "# HELP http_request_duration_milliseconds HTTP request latency" >> /tmp/metrics/index.html;
        echo "# TYPE http_request_duration_milliseconds gauge" >> /tmp/metrics/index.html;
        echo "http_request_duration_milliseconds{path=\"/api\"} $latency" >> /tmp/metrics/index.html;
        
        echo "Metrics updated: requests=$requests, errors=$errors, latency=${latency}ms";
        
        cd /tmp/metrics && busybox httpd -f -p 8080 &
        PID=$!;
        sleep 10;
        kill $PID;
      done
    ports:
    - containerPort: 8080
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f custom-metrics.yaml
```

**Step 3: Port-forward to access the metrics endpoint**

```bash
kubectl port-forward custom-metrics-demo 8080:8080
```

**Step 4: In another terminal, access the metrics**

```bash
curl localhost:8080/metrics
```

**Step 5: Observe the metrics format**

The output should look like Prometheus-formatted metrics:

```
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="get"} 42

# HELP http_errors_total Total number of HTTP errors
# TYPE http_errors_total counter
http_errors_total{code="500"} 3

# HELP http_request_duration_milliseconds HTTP request latency
# TYPE http_request_duration_milliseconds gauge
http_request_duration_milliseconds{path="/api"} 123
```

**What this does**:

- Creates a Pod that simulates exposing Prometheus-formatted metrics
- Demonstrates the format of custom metrics:
  - Help text and type information
  - Metric names and values
  - Labels for additional dimensions
- Shows how to access metrics using port-forwarding
- This helps implement custom application metrics for more comprehensive monitoring

</p>
</details>
