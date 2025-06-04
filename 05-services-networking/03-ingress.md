# Ingress

This section covers Kubernetes Ingress resources, which manage external access to services in a cluster.

## Key Resources

- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

## Introduction to Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

## Key Concepts

- **Ingress**: API object that manages external access to services in a cluster, typically HTTP
- **Ingress Controller**: Implementation that fulfills the Ingress rules (e.g., NGINX, Traefik)
- **Ingress Rules**: Configuration for routing traffic to services based on paths and hosts
- **TLS/SSL**: Configuration for secure HTTPS connections

## Ingress

### Exercise 1: Setting Up an Ingress Controller

**Goal**: Learn how to set up an Ingress Controller in your cluster.

**Task**: Install and configure the NGINX Ingress Controller.

**Why this matters**: An Ingress Controller is required to implement Ingress resources in your cluster. Without an Ingress Controller, Ingress resources have no effect. Understanding how to set up and configure an Ingress Controller is essential for exposing your applications to external traffic in a production environment.

<details><summary>show solution</summary>
<p>

**Step 1: Install the NGINX Ingress Controller**

For Minikube:

```bash
minikube addons enable ingress
```

For standard Kubernetes clusters:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

**Step 2: Verify the installation**

```bash
kubectl get pods -n ingress-nginx
```

Wait until the ingress-nginx-controller pod is running and ready.

**Step 3: Check the Ingress Controller service**

```bash
kubectl get services -n ingress-nginx
```

You should see the ingress-nginx-controller service with a LoadBalancer or NodePort type.

**What this does**:

- Installs the NGINX Ingress Controller in your cluster
- Creates the necessary resources (Deployments, Services, ConfigMaps, etc.)
- Sets up the controller to watch for Ingress resources and implement their rules
- Provides an external entry point for HTTP/HTTPS traffic to your cluster

</p>
</details>

### Exercise 2: Creating a Basic Ingress Resource

**Goal**: Learn how to create a basic Ingress resource to route traffic to a service.

**Task**: Create an Ingress resource that routes traffic to a web application.

**Why this matters**: Ingress resources allow you to define routing rules for external HTTP/HTTPS traffic to your services. This is essential for exposing multiple services through a single external IP address and implementing features like path-based routing, virtual hosts, and TLS termination.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment and Service**

Create a file named `web-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 2
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
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Apply the Deployment and Service**

```bash
kubectl apply -f web-deployment.yaml
```

**Step 3: Create an Ingress resource**

Create a file named `web-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Step 4: Apply the Ingress resource**

```bash
kubectl apply -f web-ingress.yaml
```

**Step 5: Verify the Ingress resource**

```bash
kubectl get ingress
```

**Step 6: Add the hostname to your local hosts file**

```bash
# Get the IP address of the Ingress Controller
kubectl get services -n ingress-nginx

# Add the following line to your /etc/hosts file (replace IP_ADDRESS with the actual IP)
# IP_ADDRESS example.com
```

**Step 7: Access the application**

Open a web browser and navigate to `http://example.com`.

**What this does**:

- Creates a Deployment and Service for a web application
- Creates an Ingress resource that routes traffic for the host `example.com` to the Service
- The Ingress Controller implements the routing rule
- External traffic to `example.com` is routed to the web application
- This demonstrates basic host-based routing with Ingress

</p>
</details>

### Exercise 3: Path-Based Routing

**Goal**: Learn how to create an Ingress resource that routes traffic based on URL paths.

**Task**: Create an Ingress resource that routes traffic to different services based on the URL path.

**Why this matters**: Path-based routing allows you to expose multiple services through a single domain name. This is a common pattern for modern web applications where different parts of the application (API, frontend, admin interface) are implemented as separate services but need to be accessible through a unified domain.

<details><summary>show solution</summary>
<p>

**Step 1: Create multiple Deployments and Services**

Create a file named `multiple-services.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 2
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
    app: web-app
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-app
  labels:
    app: api-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-app
  template:
    metadata:
      labels:
        app: api-app
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
  name: api-service
spec:
  selector:
    app: api-app
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Apply the Deployments and Services**

```bash
kubectl apply -f multiple-services.yaml
```

**Step 3: Create an Ingress resource with path-based routing**

Create a file named `path-based-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /web(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**Step 4: Apply the Ingress resource**

```bash
kubectl apply -f path-based-ingress.yaml
```

**Step 5: Verify the Ingress resource**

```bash
kubectl get ingress
kubectl describe ingress path-based-ingress
```

**Step 6: Access the applications**

```bash
# Access the web application
curl -H "Host: example.com" http://INGRESS_IP/web

# Access the API
curl -H "Host: example.com" http://INGRESS_IP/api
```

**What this does**:

- Creates two Deployments and Services: one for a web application and one for an API
- Creates an Ingress resource that routes traffic based on the URL path:
  - Requests to `/web/*` are routed to the web-service
  - Requests to `/api/*` are routed to the api-service
- The `rewrite-target` annotation rewrites the URL path to remove the prefix before forwarding to the service
- This demonstrates path-based routing with Ingress, allowing multiple services to be exposed through a single domain

</p>
</details>

### Exercise 4: Name-Based Virtual Hosting

**Goal**: Learn how to create an Ingress resource that routes traffic based on hostnames.

**Task**: Create an Ingress resource that routes traffic to different services based on the hostname.

**Why this matters**: Name-based virtual hosting allows you to expose multiple services through different domain names using a single IP address. This is essential for hosting multiple applications or websites on the same cluster while maintaining separate domains for each.

<details><summary>show solution</summary>
<p>

**Step 1: Create multiple Deployments and Services (if not already created)**

```bash
kubectl apply -f multiple-services.yaml
```

**Step 2: Create an Ingress resource with name-based virtual hosting**

Create a file named `name-based-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: name-based-ingress
spec:
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**Step 3: Apply the Ingress resource**

```bash
kubectl apply -f name-based-ingress.yaml
```

**Step 4: Verify the Ingress resource**

```bash
kubectl get ingress
kubectl describe ingress name-based-ingress
```

**Step 5: Add the hostnames to your local hosts file**

```bash
# Get the IP address of the Ingress Controller
kubectl get services -n ingress-nginx

# Add the following lines to your /etc/hosts file (replace IP_ADDRESS with the actual IP)
# IP_ADDRESS web.example.com
# IP_ADDRESS api.example.com
```

**Step 6: Access the applications**

```bash
# Access the web application
curl -H "Host: web.example.com" http://INGRESS_IP

# Access the API
curl -H "Host: api.example.com" http://INGRESS_IP
```

**What this does**:

- Creates an Ingress resource that routes traffic based on the hostname:
  - Requests to `web.example.com` are routed to the web-service
  - Requests to `api.example.com` are routed to the api-service
- The Ingress Controller uses the Host header to determine which service to route traffic to
- This demonstrates name-based virtual hosting with Ingress, allowing multiple services to be exposed through different domains

</p>
</details>

### Exercise 5: TLS/SSL Configuration

**Goal**: Learn how to configure TLS/SSL for an Ingress resource.

**Task**: Create an Ingress resource with TLS/SSL configuration.

**Why this matters**: Securing your applications with TLS/SSL is essential for protecting data in transit and building trust with users. Ingress resources provide a centralized way to manage TLS certificates for multiple services, simplifying the process of implementing HTTPS for your applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a TLS certificate and key**

```bash
# Generate a self-signed certificate and key
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=example.com"
```

**Step 2: Create a Secret with the certificate and key**

```bash
kubectl create secret tls example-tls --key tls.key --cert tls.crt
```

**Step 3: Create an Ingress resource with TLS configuration**

Create a file named `tls-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Step 4: Apply the Ingress resource**

```bash
kubectl apply -f tls-ingress.yaml
```

**Step 5: Verify the Ingress resource**

```bash
kubectl get ingress
kubectl describe ingress tls-ingress
```

**Step 6: Access the application securely**

```bash
# Add the hostname to your local hosts file
# IP_ADDRESS example.com

# Access the application securely
curl -k https://example.com
```

**What this does**:

- Creates a self-signed TLS certificate and key
- Creates a Kubernetes Secret containing the certificate and key
- Creates an Ingress resource with TLS configuration that references the Secret
- The Ingress Controller uses the certificate and key to terminate TLS connections
- External traffic to `https://example.com` is decrypted by the Ingress Controller and forwarded to the web service
- This demonstrates TLS termination with Ingress, allowing secure HTTPS access to your applications

</p>
</details>

### Exercise 6: Ingress Annotations

**Goal**: Learn how to use Ingress annotations to customize the behavior of the Ingress Controller.

**Task**: Create an Ingress resource with annotations for customizing the behavior of the NGINX Ingress Controller.

**Why this matters**: Ingress annotations provide a way to customize the behavior of the Ingress Controller beyond the standard Ingress specification. Different controllers support different annotations, allowing you to leverage their unique features. Understanding how to use annotations is essential for fine-tuning your Ingress configuration for production use cases.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment and Service (if not already created)**

```bash
kubectl apply -f web-deployment.yaml
```

**Step 2: Create an Ingress resource with annotations**

Create a file named `annotated-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: annotated-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "30"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "30"
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Step 3: Apply the Ingress resource**

```bash
kubectl apply -f annotated-ingress.yaml
```

**Step 4: Verify the Ingress resource**

```bash
kubectl get ingress
kubectl describe ingress annotated-ingress
```

**What this does**:

- Creates an Ingress resource with several annotations that customize the behavior of the NGINX Ingress Controller:
  - `rewrite-target`: Rewrites the URL path before forwarding to the service
  - `ssl-redirect`: Disables automatic redirection from HTTP to HTTPS
  - `use-regex`: Enables regular expressions in path matching
  - `proxy-body-size`: Sets the maximum allowed size of the client request body
  - `proxy-connect-timeout`, `proxy-send-timeout`, `proxy-read-timeout`: Set various timeout values
- These annotations are specific to the NGINX Ingress Controller and may not work with other controllers
- This demonstrates how to customize the behavior of the Ingress Controller using annotations

</p>
</details>

### Exercise 7: Canary Deployments with Ingress

**Goal**: Learn how to implement canary deployments using Ingress.

**Task**: Create an Ingress resource that routes a percentage of traffic to a canary version of an application.

**Why this matters**: Canary deployments are a powerful technique for safely rolling out new versions of applications by directing a small percentage of traffic to the new version. Implementing canary deployments with Ingress allows you to control traffic splitting at the ingress level, making it easier to manage and monitor the rollout process.

<details><summary>show solution</summary>
<p>

**Step 1: Create two versions of an application**

Create a file named `canary-deployments.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v1
  labels:
    app: web
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      version: v1
  template:
    metadata:
      labels:
        app: web
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        configMap:
          name: v1-html
---
apiVersion: v1
kind: Service
metadata:
  name: web-v1
spec:
  selector:
    app: web
    version: v1
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-v2
  labels:
    app: web
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      version: v2
  template:
    metadata:
      labels:
        app: web
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        configMap:
          name: v2-html
---
apiVersion: v1
kind: Service
metadata:
  name: web-v2
spec:
  selector:
    app: web
    version: v2
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Create ConfigMaps for the HTML content**

```bash
kubectl create configmap v1-html --from-literal=index.html="<html><body><h1>Version 1</h1></body></html>"
kubectl create configmap v2-html --from-literal=index.html="<html><body><h1>Version 2</h1></body></html>"
```

**Step 3: Apply the Deployments and Services**

```bash
kubectl apply -f canary-deployments.yaml
```

**Step 4: Create an Ingress resource for the main version**

Create a file named `canary-ingress-v1.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-v1
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-v1
            port:
              number: 80
```

**Step 5: Apply the Ingress resource for the main version**

```bash
kubectl apply -f canary-ingress-v1.yaml
```

**Step 6: Create an Ingress resource for the canary version**

Create a file named `canary-ingress-v2.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress-v2
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-v2
            port:
              number: 80
```

**Step 7: Apply the Ingress resource for the canary version**

```bash
kubectl apply -f canary-ingress-v2.yaml
```

**Step 8: Verify the Ingress resources**

```bash
kubectl get ingress
```

**Step 9: Test the canary deployment**

```bash
# Run this multiple times to see the traffic split
curl -H "Host: example.com" http://INGRESS_IP
```

**What this does**:

- Creates two versions of an application: v1 (main) and v2 (canary)
- Creates separate Services for each version
- Creates an Ingress resource for the main version that routes all traffic to v1
- Creates a canary Ingress resource with annotations that:
  - Marks it as a canary (`canary: "true"`)
  - Sets the weight to 20% (`canary-weight: "20"`)
- The NGINX Ingress Controller routes 80% of traffic to v1 and 20% to v2
- This demonstrates how to implement canary deployments using Ingress, allowing you to gradually roll out new versions of your applications

</p>
</details>
