# Probes

This section covers Kubernetes Probes, which are used to monitor container health and control application lifecycle.

## Key Resources

- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

## Introduction to Probes

Kubernetes Probes allow you to customize how Kubernetes determines the health of your containers. They help ensure that traffic is only sent to healthy containers and that unhealthy containers are restarted automatically.

## Imperative vs. Declarative Probe Configuration

Probes in Kubernetes can be configured using both declarative YAML manifests and imperative commands with some limitations:

**Declarative Approach (YAML manifests):**
- Provides full control over all probe parameters
- Required for complex probe configurations
- Better for version control and GitOps workflows
- Recommended for most production scenarios

**Imperative Approach:**
- The `kubectl run` and `kubectl create deployment` commands don't directly support adding probes
- A common workflow is to generate a basic Pod/Deployment YAML using imperative commands with `--dry-run=client -o yaml`, then edit the YAML to add probe configurations
- This hybrid approach is useful for quick prototyping and in exam scenarios

> Note: For the CKAD exam, you should be comfortable with both approaches, but especially with the hybrid approach of generating YAML templates imperatively and then adding probe configurations.

## Key Concepts

- **Liveness Probe**: Determines if a container is running properly; if it fails, the container is restarted
- **Readiness Probe**: Determines if a container is ready to receive traffic; if it fails, the Pod's IP is removed from Service endpoints
- **Startup Probe**: Determines when a container has started successfully; disables liveness and readiness checks until it succeeds
- **Probe Handlers**: HTTP, TCP, and Exec handlers for implementing probes
- **Probe Parameters**: Configuration options like timeout, period, success/failure thresholds

## Probes

### Exercise 1: Configuring Liveness Probes

**Goal**: Learn how to configure liveness probes to automatically restart unhealthy containers.

**Task**: Create a Pod with a liveness probe that checks if a container is healthy.

**Why this matters**: Liveness probes enable Kubernetes to detect when applications are in an unhealthy state that they cannot recover from on their own. By automatically restarting containers that fail liveness checks, Kubernetes helps maintain application availability without manual intervention. This is essential for building self-healing applications that can recover from failures automatically.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with an HTTP liveness probe**

Option 1: Using a manifest file (declarative approach):

Create a file named `liveness-http.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f liveness-http.yaml
```

Option 2: Using kubectl run with probe parameters (imperative approach):

```bash
kubectl run liveness-http --image=k8s.gcr.io/liveness --port=8080 \
  --dry-run=client -o yaml > liveness-http.yaml
```

Then edit the generated YAML to add the liveness probe and apply it:

```bash
# Edit the file to add the livenessProbe section
vim liveness-http.yaml

# Apply the updated configuration
kubectl apply -f liveness-http.yaml
```

> Note: While kubectl run doesn't directly support adding probes in the command line, you can generate the basic Pod YAML and then add the probe configuration. This is a common workflow in the CKAD exam.

**Step 3: Watch the Pod status**

```bash
kubectl get pod liveness-http --watch
```

After about 10 seconds, the container will start failing the liveness probe and Kubernetes will restart it.

**Step 4: Check the Pod events**

```bash
kubectl describe pod liveness-http
```

You should see events indicating that the liveness probe failed and the container was restarted.

**Step 5: Create a Pod with a command liveness probe**

Create a file named `liveness-exec.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox:1.36
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

**Step 6: Apply the Pod configuration**

```bash
kubectl apply -f liveness-exec.yaml
```

**Step 7: Watch the Pod status**

```bash
kubectl get pod liveness-exec --watch
```

After about 30 seconds, the file `/tmp/healthy` will be removed, causing the liveness probe to fail and the container to be restarted.

**What this does**:

- Creates Pods with different types of liveness probes:
  - HTTP probe that checks if an HTTP endpoint returns a success code
  - Exec probe that checks if a command executes successfully
- Configures probe parameters:
  - `initialDelaySeconds`: How long to wait after container startup before running the first probe
  - `periodSeconds`: How often to run the probe
- When the probe fails, Kubernetes automatically restarts the container
- This demonstrates how liveness probes can be used to detect and recover from application failures

</p>
</details>

### Exercise 2: Configuring Readiness Probes

**Goal**: Learn how to configure readiness probes to control when a container is ready to receive traffic.

**Task**: Create a Deployment with a readiness probe that determines when a container is ready to serve traffic.

**Why this matters**: Readiness probes prevent traffic from being sent to Pods that are not yet ready to handle requests. This is crucial during application startup, when handling heavy loads, or after configuration changes. By implementing readiness probes, you ensure that clients only connect to Pods that are fully initialized and capable of processing requests, improving the overall reliability and user experience of your applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with a readiness probe**

Option 1: Using a manifest file (declarative approach):

Create a file named `readiness-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: readiness-test
  template:
    metadata:
      labels:
        app: readiness-test
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

**Step 2: Apply the Deployment**

```bash
kubectl apply -f readiness-deployment.yaml
```

Option 2: Using kubectl create deployment with probe parameters (imperative approach):

```bash
# Generate the basic deployment YAML
kubectl create deployment readiness-test --image=nginx:1.21 --port=80 \
  --replicas=3 --dry-run=client -o yaml > readiness-deployment.yaml

# Edit the file to add the readinessProbe section
vim readiness-deployment.yaml

# Apply the updated configuration
kubectl apply -f readiness-deployment.yaml
```

> Note: For Deployments with probes, the imperative approach typically involves generating the YAML template first, then adding the probe configuration before applying.

**Step 3: Create a Service for the Deployment**

Option 1: Using a manifest file (declarative approach):

Create a file named `readiness-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: readiness-service
spec:
  selector:
    app: readiness-test
  ports:
  - port: 80
    targetPort: 80
```

**Step 4: Apply the Service**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f readiness-service.yaml
```

Option 2: Using imperative commands:
```bash
kubectl expose deployment readiness-test --name=readiness-service --port=80 --target-port=80
```

Option 2: Using kubectl expose command (imperative approach):

```bash
kubectl expose deployment readiness-test --name=readiness-service --port=80 --target-port=80
```

> Note: The imperative `kubectl expose` command is a quick way to create a Service for an existing Deployment, which is useful in the CKAD exam.

**Step 5: Watch the Pod status**

```bash
kubectl get pods -l app=readiness-test --watch
```

You should see the Pods transition to the Ready state after the readiness probe succeeds.

**Step 6: Check the endpoints of the Service**

```bash
kubectl get endpoints readiness-service
```

You should see that the Service has endpoints for all the ready Pods.

**Step 7: Create a Pod that will fail the readiness probe**

Create a file named `failing-readiness.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: failing-readiness
  labels:
    app: readiness-test
spec:
  containers:
  - name: web
    image: nginx:1.21
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /non-existent-path
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

**Step 8: Apply the Pod configuration**

```bash
kubectl apply -f failing-readiness.yaml
```

**Step 9: Check the Pod status**

```bash
kubectl get pod failing-readiness
```

You should see that the Pod is running but not ready.

**Step 10: Check the endpoints of the Service again**

```bash
kubectl get endpoints readiness-service
```

You should see that the failing Pod is not included in the Service endpoints.

**What this does**:

- Creates a Deployment with Pods that have a readiness probe
- The readiness probe checks if the nginx server is responding on port 80
- Creates a Service that selects the Pods
- Only Pods that pass the readiness probe are included in the Service endpoints
- Creates a Pod that will fail the readiness probe
- The failing Pod is not included in the Service endpoints
- This demonstrates how readiness probes control which Pods receive traffic

</p>
</details>

### Exercise 3: Configuring Startup Probes

**Goal**: Learn how to configure startup probes to handle applications with slow startup times.

**Task**: Create a Pod with a startup probe that gives an application time to initialize before liveness and readiness probes take effect.

**Why this matters**: Startup probes are essential for applications that require additional time for initialization before they're ready to serve traffic or be considered healthy. By implementing startup probes, you can prevent premature restarts of containers with slow startup times while still maintaining robust health checking once the application is running. This is particularly important for legacy applications or applications with large initialization processes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a startup probe**

Option 1: Using a manifest file (declarative approach):

Create a file named `startup-probe.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe
spec:
  containers:
  - name: app
    image: nginx:1.21
    ports:
    - containerPort: 80
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 10
      timeoutSeconds: 1
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
      timeoutSeconds: 1
      successThreshold: 2
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f startup-probe.yaml
```

Option 2: Using kubectl run with probe parameters (imperative + declarative approach):

```bash
# Generate the basic pod YAML
kubectl run startup-probe --image=nginx:1.21 --port=80 \
  --dry-run=client -o yaml > startup-probe.yaml

# Edit the file to add all three probe types
vim startup-probe.yaml

# Apply the updated configuration
kubectl apply -f startup-probe.yaml
```

> Note: For pods with multiple probe types (startup, liveness, readiness), you'll need to edit the generated YAML to add all the probe configurations. This combined approach is efficient for the CKAD exam.

**Step 3: Watch the Pod status**

```bash
kubectl get pod startup-probe --watch
```

**Step 4: Check the Pod details**

```bash
kubectl describe pod startup-probe
```

**What this does**:

- Creates a Pod with startup, liveness, and readiness probes
- The startup probe checks if the nginx server is responding on port 80
- The startup probe has a `failureThreshold` of 30 and a `periodSeconds` of 10, giving the container up to 300 seconds (30 * 10) to start up
- The liveness and readiness probes are disabled until the startup probe succeeds
- Once the startup probe succeeds, the liveness and readiness probes take over
- This demonstrates how startup probes can be used to handle applications with slow startup times

</p>
</details>

### Exercise 4: Combining Different Types of Probes

**Goal**: Learn how to use different types of probes together to create a robust health checking strategy.

**Task**: Create a Deployment with liveness, readiness, and startup probes configured for different aspects of application health.

**Why this matters**: A comprehensive health checking strategy often requires combining different types of probes to address various aspects of application health and availability. By using liveness, readiness, and startup probes together, you can create a robust system that handles initialization, detects failures, and manages traffic appropriately throughout the application lifecycle.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with multiple probe types**

Create a file named `combined-probes.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: combined-probes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: combined-probes
  template:
    metadata:
      labels:
        app: combined-probes
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 80
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
          timeoutSeconds: 1
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
          timeoutSeconds: 1
          successThreshold: 2
```

**Step 2: Apply the Deployment**

```bash
kubectl apply -f combined-probes.yaml
```

**Step 3: Watch the Pod status**

```bash
kubectl get pods -l app=combined-probes --watch
```

**Step 4: Create a Service for the Deployment**

Create a file named `combined-probes-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: combined-probes-service
spec:
  selector:
    app: combined-probes
  ports:
  - port: 80
    targetPort: 80
```

**Step 5: Apply the Service**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f combined-probes-service.yaml
```

Option 2: Using imperative commands:
```bash
kubectl expose deployment combined-probes --name=combined-probes-service --port=80 --target-port=80
```

> Note: While the service can be created imperatively, the probes in the deployment must be configured using a manifest file as imperative commands don't support probe configuration.

**What this does**:

- Creates a Deployment with Pods that have startup, liveness, and readiness probes
- The startup probe gives the application up to 300 seconds to initialize
- The liveness probe checks if the application is running properly and restarts it if it's not
- The readiness probe checks if the application is ready to receive traffic
- The probes have different parameters:
  - `periodSeconds`: How often to run the probe
  - `timeoutSeconds`: How long to wait for a response
  - `failureThreshold`: How many consecutive failures are needed to consider the probe failed
  - `successThreshold`: How many consecutive successes are needed to consider the probe successful
- This demonstrates how to combine different types of probes to create a robust health checking strategy

</p>
</details>

### Exercise 5: Using TCP Socket Probes

**Goal**: Learn how to configure TCP socket probes to check if a port is open and accepting connections.

**Task**: Create a Pod with a TCP socket probe that checks if a container is listening on a specific port.

**Why this matters**: TCP socket probes are useful for checking if a service is listening on a specific port without making an HTTP request. This is particularly useful for databases, caches, and other services that don't expose an HTTP endpoint but still need to be monitored for availability. Understanding how to use TCP socket probes expands your toolkit for implementing comprehensive health checking strategies.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a TCP socket probe**

Option 1: Using a manifest file (declarative approach):

Create a file named `tcp-probe.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-probe
spec:
  containers:
  - name: redis
    image: redis:6.2
    ports:
    - containerPort: 6379
    livenessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 15
      periodSeconds: 10
    readinessProbe:
      tcpSocket:
        port: 6379
      initialDelaySeconds: 5
      periodSeconds: 10
```

**Step 2: Apply the Pod configuration**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f tcp-probe.yaml
```

Option 2: Using imperative + declarative approach (hybrid):
```bash
# Generate the basic pod YAML
kubectl run tcp-probe --image=redis:6 --port=6379 --dry-run=client -o yaml > tcp-probe.yaml

# Edit the file to add the TCP socket probe
# Add the following under the container spec:
#   livenessProbe:
#     tcpSocket:
#       port: 6379
#     initialDelaySeconds: 15
#     periodSeconds: 10

# Apply the updated configuration
kubectl apply -f tcp-probe.yaml
```

> Note: This hybrid approach of generating a template with imperative commands and then adding probe configurations is very useful for the CKAD exam.

Option 2: Using kubectl run with probe parameters (imperative + declarative approach):

```bash
# Generate the basic pod YAML
kubectl run tcp-probe --image=redis:6.2 --port=6379 \
  --dry-run=client -o yaml > tcp-probe.yaml

# Edit the file to add the TCP socket probes
vim tcp-probe.yaml

# Apply the updated configuration
kubectl apply -f tcp-probe.yaml
```

> Note: TCP socket probes are useful for applications where you want to check if a port is open and accepting connections, without making an actual HTTP request.

**Step 3: Watch the Pod status**

```bash
kubectl get pod tcp-probe --watch
```

**Step 4: Check the Pod details**

```bash
kubectl describe pod tcp-probe
```

**What this does**:

- Creates a Pod running a Redis container
- Configures a liveness probe that checks if the container is listening on port 6379
- Configures a readiness probe that also checks if the container is listening on port 6379
- The probes use the `tcpSocket` handler to check if the port is open
- This demonstrates how to use TCP socket probes to check if a service is listening on a specific port
- This is useful for services that don't expose an HTTP endpoint but still need to be monitored

</p>
</details>

### Exercise 6: Using Exec Probes

**Goal**: Learn how to configure exec probes to run commands inside a container to check its health.

**Task**: Create a Pod with an exec probe that runs a command inside the container to check its health.

**Why this matters**: Exec probes provide the most flexibility for health checking by allowing you to run arbitrary commands inside a container. This is particularly useful for applications that don't expose HTTP endpoints or TCP ports, or when you need to perform complex health checks that can't be done with HTTP or TCP probes. Understanding how to use exec probes enables you to implement custom health checking logic for any application.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with an exec probe**

Create a file named `exec-probe.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-probe
spec:
  containers:
  - name: postgres
    image: postgres:13
    ports:
    - containerPort: 5432
    env:
    - name: POSTGRES_PASSWORD
      value: "password"
    livenessProbe:
      exec:
        command:
        - pg_isready
        - -U
        - postgres
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      exec:
        command:
        - pg_isready
        - -U
        - postgres
      initialDelaySeconds: 5
      periodSeconds: 10
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f exec-probe.yaml
```

**Step 3: Watch the Pod status**

```bash
kubectl get pod exec-probe --watch
```

**Step 4: Check the Pod details**

```bash
kubectl describe pod exec-probe
```

**What this does**:

- Creates a Pod running a PostgreSQL container
- Configures a liveness probe that runs the `pg_isready` command to check if PostgreSQL is running
- Configures a readiness probe that also runs the `pg_isready` command
- The probes use the `exec` handler to run commands inside the container
- This demonstrates how to use exec probes to run commands inside a container to check its health
- This is useful for applications that require custom health checking logic

</p>
</details>

### Exercise 7: Advanced Probe Configuration

**Goal**: Learn how to configure advanced probe parameters to fine-tune health checking behavior.

**Task**: Create a Deployment with probes that use advanced configuration parameters.

**Why this matters**: Fine-tuning probe parameters is essential for adapting health checks to the specific characteristics and requirements of your applications. By understanding and configuring advanced probe parameters, you can create more robust and reliable health checking strategies that minimize false positives/negatives and optimize resource usage.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with advanced probe configuration**

Create a file named `advanced-probes.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: advanced-probes
spec:
  replicas: 3
  selector:
    matchLabels:
      app: advanced-probes
  template:
    metadata:
      labels:
        app: advanced-probes
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
            httpHeaders:
            - name: Custom-Header
              value: Awesome
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 2
          failureThreshold: 3
          successThreshold: 1
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 1
          failureThreshold: 2
          successThreshold: 2
```

**Step 2: Apply the Deployment**

```bash
kubectl apply -f advanced-probes.yaml
```

**Step 3: Watch the Pod status**

```bash
kubectl get pods -l app=advanced-probes --watch
```

**Step 4: Check the Pod details**

```bash
kubectl describe pod -l app=advanced-probes
```

**What this does**:

- Creates a Deployment with Pods that have liveness and readiness probes
- Configures advanced probe parameters:
  - `initialDelaySeconds`: How long to wait after container startup before running the first probe
  - `periodSeconds`: How often to run the probe
  - `timeoutSeconds`: How long to wait for a response before considering the probe failed
  - `failureThreshold`: How many consecutive failures are needed to consider the probe failed
  - `successThreshold`: How many consecutive successes are needed to consider the probe successful
- Adds custom HTTP headers to the liveness probe
- This demonstrates how to fine-tune probe parameters for different requirements
- Different values are used for liveness and readiness probes to show that they can be configured independently

</p>
</details>
