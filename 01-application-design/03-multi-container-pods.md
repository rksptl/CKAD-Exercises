# Multi-Container Pod Design Patterns

This section covers multi-container Pod design patterns, which are essential for building complex applications in Kubernetes.

## Key Resources

- [Pod Design Patterns](https://kubernetes.io/docs/concepts/workloads/pods/#pod-design-patterns)
- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Communication Between Containers in the Same Pod](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

## Introduction to Multi-Container Pods

In Kubernetes, a Pod is the smallest deployable unit that can contain one or more containers. While many Pods contain only a single container, there are several design patterns where multiple containers within a single Pod provide significant benefits for application architecture.

## Imperative vs. Declarative Approaches for Multi-Container Pods

**Important Note for CKAD Exam:** Multi-container pods can only be created using declarative YAML manifests. There is no direct imperative command to create pods with multiple containers. However, you can use a hybrid approach:

1. **Generate a basic Pod YAML** using imperative commands with `--dry-run=client -o yaml`
2. **Edit the generated YAML** to add additional containers and configurations
3. **Apply the modified YAML** to create the multi-container Pod

This hybrid approach is often useful in the CKAD exam when you need to create multi-container pods quickly.

## Key Concepts

- **Pod**: The smallest deployable unit in Kubernetes, containing one or more containers that share network namespace and can optionally share volumes
- **Sidecar Pattern**: Enhances the main container with additional functionality
- **Ambassador Pattern**: Proxies network traffic to/from the main container
- **Adapter Pattern**: Transforms output from the main container
- **Init Containers**: Run before app containers and can be used for setup tasks

## Multi-Container Pod Patterns

### Exercise 1: Sidecar Pattern

**Goal**: Learn how to implement the sidecar pattern with a main application container and a logging sidecar.

**Task**: Create a Pod with a web server container and a sidecar container that forwards logs to a central logging service.

**Why this matters**: The sidecar pattern is widely used in microservice architectures to extend and enhance the functionality of the main container without modifying it. Common examples include logging agents, monitoring agents, and configuration synchronizers. This pattern enables separation of concerns and makes applications more maintainable.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

Create a file named `sidecar-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
  labels:
    app: web-server
spec:
  containers:
    - name: nginx
      image: nginx:1.21
      ports:
        - containerPort: 80
      volumeMounts:
        - name: logs-volume
          mountPath: /var/log/nginx
    - name: log-sidecar
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          "while true; do cat /var/log/nginx/access.log; sleep 10; done",
        ]
      volumeMounts:
        - name: logs-volume
          mountPath: /var/log/nginx
  volumes:
    - name: logs-volume
      emptyDir: {}
```

**Step 2: Create the Pod**

```bash
kubectl apply -f sidecar-pod.yaml
```

**Step 3: Generate some logs**

```bash
kubectl port-forward sidecar-pod 8080:80 &
curl http://localhost:8080
```

**Step 4: Check the logs from the sidecar container**

```bash
kubectl logs sidecar-pod -c log-sidecar
```

**What this does**:

- Creates a Pod with two containers: an nginx web server and a log sidecar
- Both containers share a volume where nginx writes its logs
- The sidecar container reads the logs and outputs them (in a real scenario, it would forward them to a logging service)
- This pattern allows you to add logging functionality without modifying the main application

</p>
</details>

### Exercise 2: Ambassador Pattern

**Goal**: Learn how to implement the ambassador pattern to proxy connections from the main container.

**Task**: Create a Pod with a main application container and an ambassador container that handles outbound connections.

**Why this matters**: The ambassador pattern is valuable when you need to simplify how your application connects to other services. The ambassador container can handle service discovery, connection pooling, or retry logic, allowing the main application to connect to a simple local endpoint. This pattern is particularly useful for legacy applications or when you want to abstract complex networking from your application code.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

Create a file named `ambassador-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
  labels:
    app: ambassador-example
spec:
  containers:
    - name: app
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          "while true; do wget -q -O- http://localhost:9000; sleep 5; done",
        ]
    - name: ambassador
      image: nginx:1.21
      ports:
        - containerPort: 9000
      volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
  volumes:
    - name: nginx-config
      configMap:
        name: ambassador-config
```

**Step 2: Create a ConfigMap for the ambassador's configuration**

Create a file named `ambassador-config.yaml` with the following content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ambassador-config
data:
  default.conf: |
    server {
      listen 9000;
      location / {
        proxy_pass http://kubernetes.default.svc;
        proxy_set_header Host kubernetes.default.svc;
        proxy_ssl_verify off;
      }
    }
```

**Step 3: Apply the ConfigMap and Pod**

```bash
kubectl apply -f ambassador-config.yaml
kubectl apply -f ambassador-pod.yaml
```

> Note: The ambassador pattern requires a manifest file as it involves multiple containers with specific configurations.

**Step 4: Check the logs from the ambassador container**

```bash
kubectl logs ambassador-pod -c ambassador
```

**What this does**:

- Creates a Pod with two containers: an application and an ambassador
- The ambassador container runs nginx configured as a proxy to the Kubernetes API server
- The application container makes requests to the ambassador on localhost:9000
- The ambassador forwards these requests to the actual service
- This pattern simplifies how the application connects to external services

</p>
</details>

### Exercise 3: Adapter Pattern

**Goal**: Learn how to implement the adapter pattern to transform output from the main container.

**Task**: Create a Pod with a main application container and an adapter container that transforms the application's output format.

**Why this matters**: The adapter pattern is essential when you need to standardize or transform the output of an application to match what downstream systems expect. This allows legacy applications or third-party components to integrate with your system without modifying their code. Common use cases include log format standardization, metric transformations, and protocol conversions.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

Create a file named `adapter-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
  labels:
    app: adapter-example
spec:
  containers:
    - name: app
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          'while true; do echo ''{"timestamp": "''$(date +%s)''", "message": "Sample log entry"}'' >> /var/log/app.log; sleep 5; done',
        ]
      volumeMounts:
        - name: log-volume
          mountPath: /var/log
    - name: adapter
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          'tail -f /var/log/app.log | while read line; do echo "$(date -Iseconds) - Transformed: $line"; done',
        ]
      volumeMounts:
        - name: log-volume
          mountPath: /var/log
  volumes:
    - name: log-volume
      emptyDir: {}
```

**Step 2: Create the Pod**

Hybrid approach (imperative + declarative):
```bash
# Generate the basic pod template with one container
kubectl run adapter-pod --image=busybox:1.36 --labels="app=adapter-example" --dry-run=client -o yaml > adapter-pod.yaml

# Edit the generated YAML to add the adapter container and volume configurations
# vim adapter-pod.yaml

# Apply the completed multi-container pod manifest
kubectl apply -f adapter-pod.yaml
```

> Note: This hybrid approach is very useful in the CKAD exam. You can quickly generate a basic pod template using imperative commands, then modify it to add additional containers and configurations.

> Note: The `kubectl run` command can only create single-container pods. For multi-container pods, we must use the hybrid approach:
> 1. Generate a basic pod template with `kubectl run --dry-run=client -o yaml`
> 2. Edit the generated YAML to add additional containers
> 3. Apply the modified YAML with `kubectl apply -f`
> 
> This workflow is essential to understand for the CKAD exam, as it combines the speed of imperative commands with the flexibility of declarative manifests.

**Step 3: Check the logs from both containers**

```bash
kubectl logs adapter-pod -c app
kubectl logs adapter-pod -c adapter
```

**What this does**:

- Creates a Pod with two containers: an application and an adapter
- The application container writes JSON log entries to a shared volume
- The adapter container reads these logs, transforms their format, and outputs them
- In a real scenario, the adapter might forward the transformed logs to a monitoring system
- This pattern allows you to adapt the output format of an application without modifying it

</p>
</details>

### Exercise 4: Init Container Pattern

**Goal**: Learn how to use init containers to perform setup tasks before the main application starts.

**Task**: Create a Pod with an init container that performs a setup task and a main application container that depends on this setup.

**Why this matters**: Init containers solve the problem of sequencing startup tasks in Kubernetes. They run to completion before the main application containers start, making them perfect for setup tasks like database schema creation, dependency downloads, or permission configuration. This pattern ensures your application starts only when its prerequisites are met.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

Create a file named `init-container-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
  labels:
    app: init-example
spec:
  initContainers:
    - name: init-service
      image: busybox:1.36
      command:
        [
          "sh",
          "-c",
          "until nslookup kubernetes.default; do echo waiting for kubernetes service; sleep 2; done;",
        ]
    - name: init-config
      image: busybox:1.36
      command: ["sh", "-c", 'echo "Configuration data" > /config/config.txt']
      volumeMounts:
        - name: config-volume
          mountPath: /config
  containers:
    - name: app
      image: busybox:1.36
      command:
        [
          "sh",
          "-c",
          'cat /config/config.txt; echo "Application running..."; sleep 3600',
        ]
      volumeMounts:
        - name: config-volume
          mountPath: /config
  volumes:
    - name: config-volume
      emptyDir: {}
```

**Step 2: Create the Pod**

```bash
kubectl apply -f init-container-pod.yaml
```

> Note: Pods with init containers must be created using manifest files as there is no direct imperative command for this pattern. You can use the hybrid approach by generating a basic pod template with `kubectl run --dry-run=client -o yaml` and then adding the init containers section to the generated YAML.

**Step 3: Watch the Pod status**

```bash
kubectl get pod init-container-pod -w
```

**Step 4: Check the logs from the main container**

```bash
kubectl logs init-container-pod -c app
```

**What this does**:

- Creates a Pod with two init containers and one main container
- The first init container waits for the Kubernetes service to be available
- The second init container writes configuration data to a shared volume
- The main container starts only after both init containers complete successfully
- The main container reads the configuration data prepared by the init container
- This pattern ensures proper sequencing of startup tasks

</p>
</details>

### Exercise 5: Shared Resources in Multi-Container Pods

**Goal**: Learn how to share resources between containers in a Pod.

**Task**: Create a Pod with multiple containers that share network namespace and storage volumes.

**Why this matters**: Understanding how containers within a Pod share resources is fundamental to designing effective multi-container applications. This knowledge allows you to leverage inter-container communication via localhost and shared storage, which are key advantages of the Pod abstraction in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

Create a file named `shared-resources-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-resources-pod
  labels:
    app: shared-example
spec:
  containers:
    - name: web-server
      image: nginx:1.21
      ports:
        - containerPort: 80
      volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
    - name: content-creator
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          "while true; do echo $(date) > /html/index.html; sleep 10; done",
        ]
      volumeMounts:
        - name: html-volume
          mountPath: /html
    - name: request-maker
      image: busybox:1.36
      command:
        [
          "/bin/sh",
          "-c",
          "while true; do wget -q -O- http://localhost:80; sleep 5; done",
        ]
  volumes:
    - name: html-volume
      emptyDir: {}
```

**Step 2: Create the Pod**

```bash
kubectl apply -f shared-resources-pod.yaml
```

> Note: Pods with shared resources across multiple containers require manifest files as there is no imperative command to create this complex structure. For the CKAD exam, remember that all multi-container pod patterns (sidecar, ambassador, adapter, init containers) require the declarative approach or the hybrid approach of generating a template imperatively and then modifying it.

**Step 3: Check the logs from the request-maker container**

```bash
kubectl logs shared-resources-pod -c request-maker
```

**What this does**:

- Creates a Pod with three containers that demonstrate resource sharing:
  - `web-server`: An nginx container serving web content
  - `content-creator`: A container that updates the web content every 10 seconds
  - `request-maker`: A container that makes requests to the web server using localhost
- The `web-server` and `content-creator` containers share a volume for the web content
- The `request-maker` container communicates with the `web-server` container via localhost (shared network namespace)
- This demonstrates both volume sharing and network namespace sharing within a Pod

</p>
</details>
