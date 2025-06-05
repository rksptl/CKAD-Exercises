# Services

This section covers Kubernetes Services, which provide networking and service discovery for Pods.

## Key Resources

- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

## Introduction to Services

Services in Kubernetes provide a way to expose applications running on a set of Pods as a network service. They abstract the Pod IP addresses and provide a stable endpoint for clients to connect to, regardless of Pod lifecycle changes.

## Imperative vs. Declarative Service Creation

Kubernetes Services can be created using both imperative commands and declarative manifests:

**Imperative Commands (using `kubectl expose` or `kubectl create service`):**

- Quick and convenient for simple services
- Good for testing, prototyping, and learning
- Limited options compared to declarative approach
- Not suitable for complex service configurations

**Declarative Manifests (using YAML files):**

- Provides full control over all service parameters
- Required for complex configurations (multi-port, specific nodePort, etc.)
- Better for version control and GitOps workflows
- More verbose but more powerful

## Key Concepts

- **Service**: An abstraction that defines a logical set of Pods and a policy to access them
- **ClusterIP**: Exposes the Service on an internal IP in the cluster
- **NodePort**: Exposes the Service on each Node's IP at a static port
- **LoadBalancer**: Exposes the Service externally using a cloud provider's load balancer
- **ExternalName**: Maps the Service to a DNS name
- **Endpoints**: The actual Pod IPs that a Service routes traffic to
- **Selectors**: Labels used to identify the Pods that a Service targets

## Services

### Exercise 1: Creating a ClusterIP Service

**Goal**: Learn how to create a ClusterIP Service to expose an application within the cluster.

**Task**: Create a Deployment and expose it using a ClusterIP Service.

**Why this matters**: ClusterIP Services are the foundation of service discovery in Kubernetes. They provide a stable internal endpoint for applications to communicate with each other within the cluster. This is essential for building microservices architectures where services need to find and communicate with each other reliably.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment**

Create a file named `web-deployment.yaml` with the following content:

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
```

**Step 2: Apply the Deployment**

```bash
kubectl apply -f web-deployment.yaml
```

**Step 3: Create a ClusterIP Service**

Create a file named `web-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

**Step 4: Create the Service**

Option 1: Using a manifest file (declarative approach):

```bash
kubectl apply -f web-service.yaml
```

Option 2: Using imperative commands:

```bash
kubectl expose deployment web-app --name=web-service --port=80 --target-port=80 --type=ClusterIP
```

**Step 5: Verify the Service**

```bash
kubectl get services
kubectl describe service web-service
```

**Step 6: Test the Service from another Pod**

```bash
kubectl run test-pod --image=busybox:1.36 --rm -it -- wget -qO- web-service
```

**What this does**:

- Creates a Deployment with 3 replicas of an nginx container
- Creates a ClusterIP Service that selects Pods with the label `app=web-app`
- The Service routes traffic on port 80 to port 80 of the selected Pods
- The Service is assigned a stable IP address within the cluster
- Other Pods in the cluster can access the Service using its name (`web-service`) or its IP address
- The Service load balances traffic across all matching Pods

</p>
</details>

### Exercise 2: Creating a NodePort Service

**Goal**: Learn how to create a NodePort Service to expose an application outside the cluster.

**Task**: Create a Deployment and expose it using a NodePort Service.

**Why this matters**: NodePort Services provide a way to expose applications to external clients without requiring a load balancer. This is useful for development environments, on-premises clusters, or when you need to expose services on specific ports. Understanding NodePort Services is important for making your applications accessible outside the cluster.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment (if not already created)**

```bash
kubectl apply -f web-deployment.yaml
```

**Step 2: Create a NodePort Service**

Create a file named `web-nodeport-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

**Step 3: Create the Service**

Option 1: Using a manifest file (declarative approach):

```bash
kubectl apply -f web-nodeport-service.yaml
```

Option 2: Using imperative commands:

```bash
# Create a NodePort service (note: you cannot specify the nodePort with imperative commands)
kubectl expose deployment web-app --name=web-nodeport --port=80 --target-port=80 --type=NodePort

# If you need to specify the nodePort, you must use a manifest file or patch the service
```

> Note: The imperative command doesn't allow specifying the nodePort. Kubernetes will assign a random port in the NodePort range (30000-32767). If you need a specific nodePort, use a manifest file or patch the service after creation.

**Step 4: Verify the Service**

```bash
kubectl get services
kubectl describe service web-nodeport
```

**Step 5: Get the Node IP**

```bash
kubectl get nodes -o wide
```

**Step 6: Access the Service**

```bash
# Replace NODE_IP with the IP address of any node in your cluster
curl http://NODE_IP:30080
```

**What this does**:

- Creates a NodePort Service that selects Pods with the label `app=web-app`
- The Service routes traffic on port 80 to port 80 of the selected Pods
- The Service is exposed on port 30080 on all nodes in the cluster
- External clients can access the Service using the IP address of any node in the cluster and the NodePort
- The valid range for NodePort is 30000-32767 by default
- If you don't specify a nodePort, Kubernetes will assign one automatically

</p>
</details>

### Exercise 3: Creating a LoadBalancer Service

**Goal**: Learn how to create a LoadBalancer Service to expose an application using a cloud provider's load balancer.

**Task**: Create a Deployment and expose it using a LoadBalancer Service.

**Why this matters**: LoadBalancer Services are the standard way to expose applications to the internet in cloud environments. They provision an external load balancer that routes traffic to your service, providing a single stable IP address for external clients. This is essential for production applications that need to be accessible from the internet.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment (if not already created)**

```bash
kubectl apply -f web-deployment.yaml
```

**Step 2: Create a LoadBalancer Service**

Create a file named `web-loadbalancer-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer
```

**Step 3: Create the Service**

Option 1: Using a manifest file (declarative approach):

```bash
kubectl apply -f web-loadbalancer-service.yaml
```

Option 2: Using imperative commands:

```bash
kubectl expose deployment web-app --name=web-loadbalancer --port=80 --target-port=80 --type=LoadBalancer
```

**Step 4: Verify the Service**

```bash
kubectl get services
kubectl describe service web-loadbalancer
```

**Step 5: Access the Service**

```bash
# Get the external IP address
kubectl get service web-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Access the service
curl http://EXTERNAL_IP
```

**Note**: This will only work in cloud environments that support LoadBalancer Services. In local environments like Minikube or Kind, you may need to use `minikube tunnel` or similar commands to simulate a LoadBalancer.

**What this does**:

- Creates a LoadBalancer Service that selects Pods with the label `app=web-app`
- The Service routes traffic on port 80 to port 80 of the selected Pods
- The cloud provider provisions an external load balancer that forwards traffic to the Service
- External clients can access the Service using the external IP address assigned by the cloud provider
- The Service load balances traffic across all matching Pods

</p>
</details>

### Exercise 4: Creating an ExternalName Service

**Goal**: Learn how to create an ExternalName Service to map a Service to an external DNS name.

**Task**: Create an ExternalName Service that maps to an external service.

**Why this matters**: ExternalName Services provide a way to create service abstractions for external services. This allows applications in your cluster to use Kubernetes service discovery to access external services as if they were internal, simplifying configuration and enabling easier migration of external services into the cluster in the future.

<details><summary>show solution</summary>
<p>

**Step 1: Create an ExternalName Service**

Create a file named `external-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: database.example.com
```

**Step 2: Create the Service**

Option 1: Using a manifest file (declarative approach):

```bash
kubectl apply -f external-service.yaml
```

Option 2: Using imperative commands:

```bash
kubectl create service externalname external-database --external-name=database.example.com
```

> Note: ExternalName services are useful for mapping external services into your Kubernetes namespace, allowing applications to use consistent naming regardless of environment.

**Step 3: Verify the Service**

```bash
kubectl get services
kubectl describe service external-database
```

**Step 4: Test the Service from a Pod**

```bash
kubectl run test-pod --image=busybox:1.36 --rm -it -- nslookup external-database
```

**What this does**:

- Creates an ExternalName Service named `external-database`
- The Service maps to the external DNS name `database.example.com`
- Pods in the cluster can access the external service using the name `external-database`
- When Pods resolve the DNS name `external-database`, they get a CNAME record pointing to `database.example.com`
- No proxying or load balancing is performed; this is purely a DNS mapping
- This is useful for accessing external services using internal service names

</p>
</details>

### Exercise 5: Creating a Headless Service

**Goal**: Learn how to create a Headless Service for direct Pod-to-Pod communication.

**Task**: Create a StatefulSet and expose it using a Headless Service.

**Why this matters**: Headless Services are essential for stateful applications where clients need to connect to specific Pods rather than being load balanced across all Pods. They provide DNS entries for each Pod, enabling direct Pod-to-Pod communication. This pattern is commonly used with StatefulSets for databases and other stateful applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Headless Service**

Create a file named `headless-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  clusterIP: None
  selector:
    app: database
  ports:
    - port: 3306
      targetPort: 3306
```

**Step 2: Create a StatefulSet**

Create a file named `database-statefulset.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: database
  serviceName: "database"
  replicas: 3
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
              name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "password"
            - name: MYSQL_DATABASE
              value: "testdb"
```

**Step 3: Apply the Service and StatefulSet**

```bash
kubectl apply -f headless-service.yaml
kubectl apply -f database-statefulset.yaml
```

**Step 4: Verify the Service and StatefulSet**

```bash
kubectl get services
kubectl get statefulsets
kubectl get pods -l app=database
```

**Step 5: Test DNS resolution for individual Pods**

```bash
kubectl run test-pod --image=busybox:1.36 --rm -it -- nslookup mysql-0.database
kubectl run test-pod --image=busybox:1.36 --rm -it -- nslookup mysql-1.database
kubectl run test-pod --image=busybox:1.36 --rm -it -- nslookup mysql-2.database
```

**What this does**:

- Creates a Headless Service (setting `clusterIP: None`) named `database`
- Creates a StatefulSet that uses the Headless Service
- Each Pod in the StatefulSet gets a stable hostname in the form of `<pod-name>.<service-name>`
- DNS lookups for the service return the IP addresses of all Pods
- DNS lookups for `<pod-name>.<service-name>` return the IP address of the specific Pod
- This enables direct Pod-to-Pod communication without load balancing
- This is useful for stateful applications where clients need to connect to specific Pods

</p>
</details>

### Exercise 6: Service Discovery and DNS

**Goal**: Learn how to use Kubernetes DNS for service discovery.

**Task**: Create multiple Services and use DNS to discover and communicate between them.

**Why this matters**: DNS-based service discovery is a core feature of Kubernetes that enables microservices to find and communicate with each other. Understanding how Kubernetes DNS works is essential for building distributed applications that can dynamically discover and connect to services without hardcoded IPs or manual configuration.

<details><summary>show solution</summary>
<p>

**Step 1: Create a backend Deployment and Service**

Create a file named `backend-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: nginx:1.21
          ports:
            - containerPort: 80
```

Create a file named `backend-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
```

**Step 2: Create a frontend Deployment and Service**

Create a file named `frontend-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: busybox:1.36
          command:
            [
              "/bin/sh",
              "-c",
              "while true; do wget -q -O- http://backend-service; sleep 5; done",
            ]
```

Create a file named `frontend-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
```

**Step 3: Create the Services**

Option 1: Using manifest files (declarative approach):

```bash
# Create backend service
kubectl apply -f backend-service.yaml

# Create frontend service
kubectl apply -f frontend-service.yaml
```

Option 2: Using imperative commands:

```bash
# Create backend service
kubectl expose deployment backend --name=backend-service --port=80 --target-port=80

# Create frontend service
kubectl expose deployment frontend --name=frontend-service --port=80 --target-port=80
```

**Step 4: Verify the Deployments and Services**

```bash
kubectl get deployments
kubectl get services
kubectl get pods
```

**Step 5: Check the logs of the frontend Pods**

```bash
kubectl logs -l app=frontend
```

**Step 6: Create a test Pod to explore DNS**

```bash
kubectl run dns-test --image=busybox:1.36 --rm -it -- sh
```

Inside the Pod, run the following commands:

```bash
# Look up the backend service
nslookup backend-service

# Look up the frontend service
nslookup frontend-service

# Look up the backend service with the full domain name
nslookup backend-service.default.svc.cluster.local

# Look up the Kubernetes API service
nslookup kubernetes.default.svc.cluster.local

# Try accessing the backend service
wget -q -O- http://backend-service
```

**What this does**:

- Creates two Deployments and Services: `backend` and `frontend`
- The frontend Pods communicate with the backend service using its DNS name
- Demonstrates how Kubernetes DNS works for service discovery:
  - Services are accessible by their name within the same namespace
  - The full DNS name format is `<service-name>.<namespace>.svc.cluster.local`
  - DNS resolution automatically returns the ClusterIP of the service
- This enables microservices to discover and communicate with each other without hardcoded IPs

</p>
</details>

### Exercise 7: Multi-Port Services

**Goal**: Learn how to create a Service that exposes multiple ports.

**Task**: Create a Deployment with a container that listens on multiple ports and expose it using a Service.

**Why this matters**: Many applications expose multiple ports for different purposes, such as HTTP and HTTPS, or application and admin interfaces. Understanding how to configure Services to expose multiple ports is essential for properly exposing these applications in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with a container that listens on multiple ports**

Create a file named `multi-port-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-port-app
  labels:
    app: multi-port-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: multi-port-app
  template:
    metadata:
      labels:
        app: multi-port-app
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
          ports:
            - containerPort: 80
              name: http
            - containerPort: 443
              name: https
```

**Step 2: Create a Service that exposes multiple ports**

Create a file named `multi-port-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: multi-port-app
  ports:
    - name: http
      port: 80
      targetPort: http
    - name: https
      port: 443
      targetPort: https
```

**Step 3: Create the Deployment and Service**

```bash
# Create deployment
kubectl apply -f multi-port-deployment.yaml

# Create service (must use manifest file for multi-port services)
kubectl apply -f multi-port-service.yaml
```

> Note: While simple services can be created imperatively, multi-port services require a manifest file as the `kubectl expose` command doesn't support defining multiple ports.

**Step 4: Verify the Deployment and Service**

```bash
kubectl get deployments
kubectl get services
kubectl describe service multi-port-service
```

**Step 5: Test accessing the Service on different ports**

```bash
kubectl run test-pod --image=busybox:1.36 --rm -it -- sh
```

Inside the Pod, run the following commands:

```bash
# Access the HTTP port
wget -q -O- http://multi-port-service

# Access the HTTPS port (this will fail without proper TLS setup, but demonstrates the concept)
wget -q --no-check-certificate -O- https://multi-port-service
```

**What this does**:

- Creates a Deployment with containers that expose ports 80 (HTTP) and 443 (HTTPS)
- Creates a Service that exposes both ports
- Names the ports in both the Pod and Service definitions
- Uses the port names to map Service ports to container ports
- This allows clients to access the application on either port
- Naming ports makes the configuration more maintainable and self-documenting

</p>
</details>
