# Configuration Management

This section covers Kubernetes configuration management resources such as ConfigMaps and Secrets, which are essential for separating configuration from application code.

## Key Resources

- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Environment Variables](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

## Introduction to Configuration Management

Kubernetes provides several ways to separate configuration from application code, following the principles of the [Twelve-Factor App](https://12factor.net/). This separation allows you to deploy the same application code with different configurations across environments.

## Key Concepts

- **ConfigMaps**: Store non-confidential configuration data as key-value pairs
- **Secrets**: Store sensitive information such as passwords, OAuth tokens, and SSH keys
- **Environment Variables**: Pass configuration to containers as environment variables
- **Volume Mounts**: Mount ConfigMaps and Secrets as volumes in containers

## Configuration Management

### Exercise 1: Creating and Using ConfigMaps

**Goal**: Learn how to create ConfigMaps and use them in Pods.

**Task**: Create a ConfigMap with configuration data and consume it in a Pod using environment variables and volume mounts.

**Why this matters**: ConfigMaps are essential for externalizing application configuration from container images. This practice follows the configuration principle of the Twelve-Factor App methodology, allowing you to deploy the same container image across different environments with environment-specific configurations. This improves maintainability and follows security best practices.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap from literal values**

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

**Step 2: Create a ConfigMap from a file**

Create a file named `config.properties` with the following content:

```
database.url=jdbc:mysql://db-service:3306/mydb
database.user=app_user
database.connections=10
```

Create the ConfigMap from this file:

```bash
kubectl create configmap db-config --from-file=config.properties
```

**Step 3: Create a Pod that uses both ConfigMaps**

Create a file named `configmap-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "env && while true; do sleep 3600; done"]
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: APP_DEBUG
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_DEBUG
    - name: APP_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_PORT
    volumeMounts:
    - name: db-config-volume
      mountPath: /etc/config
  volumes:
  - name: db-config-volume
    configMap:
      name: db-config
```

**Step 4: Create the Pod**

```bash
kubectl apply -f configmap-pod.yaml
```

**Step 5: Verify the ConfigMap data in the Pod**

```bash
kubectl logs configmap-pod
kubectl exec configmap-pod -- cat /etc/config/config.properties
```

**What this does**:

- Creates two ConfigMaps:
  - `app-config`: Contains individual key-value pairs
  - `db-config`: Contains configuration from a file
- Creates a Pod that consumes the ConfigMaps in two ways:
  - As environment variables using `valueFrom.configMapKeyRef`
  - As a mounted volume using `volumes` and `volumeMounts`
- The environment variables are accessible directly in the container
- The mounted volume makes the configuration file available at `/etc/config/config.properties`

</p>
</details>

### Exercise 2: Creating and Using Secrets

**Goal**: Learn how to create Secrets and use them in Pods.

**Task**: Create a Secret with sensitive data and consume it in a Pod using environment variables and volume mounts.

**Why this matters**: Secrets provide a way to store and manage sensitive information separately from application code. This separation is crucial for security, as it prevents sensitive data from being exposed in container images or Pod specifications. Understanding how to properly use Secrets is essential for building secure applications in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Secret from literal values**

```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=s3cr3t
```

**Step 2: Create a Secret from a file**

Create a file named `tls.key` with some content (this would normally be your private key):

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7VJTUt9Us8cKj
MzEfYyjiWA4R4/M2bS1GB4t7NXp98C3SC6dVMvDuictGeurT8jNbvJZHtCSuYEvu
NMoSfm76oqFvAp8Gy0iz5sxjZmSnXyCdPEovGhLa0VzMaQ8s+CLOyS56YyCFGeJZ
...
-----END PRIVATE KEY-----
```

Create a file named `tls.crt` with some content (this would normally be your certificate):

```
-----BEGIN CERTIFICATE-----
MIIEWTCCAsGgAwIBAgIJALfRlWsI8YQHMA0GCSqGSIb3DQEBCwUAMEIxCzAJBgNV
BAYTAlVTMQswCQYDVQQIDAJDQTEQMA4GA1UEBwwHT2FrbGFuZDEUMBIGA1UECgwL
...
-----END CERTIFICATE-----
```

Create the Secret from these files:

```bash
kubectl create secret tls tls-secret --key=tls.key --cert=tls.crt
```

**Step 3: Create a Pod that uses both Secrets**

Create a file named `secret-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "env | grep DB_ && ls -la /etc/tls && while true; do sleep 3600; done"]
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
    volumeMounts:
    - name: tls-certs
      mountPath: /etc/tls
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: tls-secret
```

**Step 4: Create the Pod**

```bash
kubectl apply -f secret-pod.yaml
```

**Step 5: Verify the Secret data in the Pod**

```bash
kubectl logs secret-pod
```

**What this does**:

- Creates two Secrets:
  - `db-credentials`: Contains database username and password
  - `tls-secret`: Contains TLS certificate and private key
- Creates a Pod that consumes the Secrets in two ways:
  - As environment variables using `valueFrom.secretKeyRef`
  - As a mounted volume using `volumes` and `volumeMounts`
- The environment variables are accessible directly in the container
- The mounted volume makes the TLS files available at `/etc/tls/tls.crt` and `/etc/tls/tls.key`
- Secrets are base64-encoded in etcd but are automatically decoded when used in Pods

</p>
</details>

### Exercise 3: Using Environment Variables

**Goal**: Learn how to define environment variables in Pods.

**Task**: Create a Pod with environment variables defined directly in the Pod specification.

**Why this matters**: Environment variables are a fundamental way to pass configuration to containers. They provide a simple and standardized interface for applications to access configuration, following the principles of the Twelve-Factor App methodology. Understanding how to properly set and use environment variables is essential for configuring applications in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file with environment variables**

Create a file named `env-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "env && while true; do sleep 3600; done"]
    env:
    - name: APP_NAME
      value: "My Application"
    - name: APP_VERSION
      value: "1.0.0"
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: CPU_LIMIT
      valueFrom:
        resourceFieldRef:
          containerName: app
          resource: limits.cpu
          divisor: "1"
    - name: MEMORY_LIMIT
      valueFrom:
        resourceFieldRef:
          containerName: app
          resource: limits.memory
          divisor: "1Mi"
    resources:
      limits:
        cpu: "500m"
        memory: "128Mi"
      requests:
        cpu: "250m"
        memory: "64Mi"
```

**Step 2: Create the Pod**

```bash
kubectl apply -f env-pod.yaml
```

**Step 3: Verify the environment variables in the Pod**

```bash
kubectl logs env-pod
```

**What this does**:

- Creates a Pod with various types of environment variables:
  - Static values (`APP_NAME`, `APP_VERSION`)
  - Pod metadata (`POD_NAME`, `POD_NAMESPACE`, `POD_IP`, `NODE_NAME`)
  - Resource limits (`CPU_LIMIT`, `MEMORY_LIMIT`)
- The environment variables are accessible directly in the container
- This demonstrates different ways to define environment variables in Kubernetes:
  - Direct values using `value`
  - Pod fields using `valueFrom.fieldRef`
  - Resource fields using `valueFrom.resourceFieldRef`

</p>
</details>

### Exercise 4: Using ConfigMaps for Configuration Files

**Goal**: Learn how to use ConfigMaps to provide configuration files to applications.

**Task**: Create a ConfigMap with a configuration file and mount it in a Pod.

**Why this matters**: Many applications require configuration files rather than environment variables. ConfigMaps allow you to store these files separately from your container images and mount them into containers at runtime. This approach is essential for applications that expect configuration in specific file formats or locations.

<details><summary>show solution</summary>
<p>

**Step 1: Create a configuration file**

Create a file named `nginx.conf` with the following content:

```
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /api {
        proxy_pass http://backend-service:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

**Step 2: Create a ConfigMap from the configuration file**

```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```

**Step 3: Create a Pod that uses the ConfigMap**

Create a file named `nginx-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
      items:
      - key: nginx.conf
        path: default.conf
```

**Step 4: Create the Pod**

```bash
kubectl apply -f nginx-pod.yaml
```

**Step 5: Verify the configuration file in the Pod**

```bash
kubectl exec nginx-pod -- cat /etc/nginx/conf.d/default.conf
```

**Step 6: Create a Service for the Pod**

```bash
kubectl expose pod nginx-pod --port=80 --name=nginx-service
```

**Step 7: Test the nginx server**

```bash
kubectl port-forward service/nginx-service 8080:80 &
curl http://localhost:8080
```

**What this does**:

- Creates a ConfigMap containing an nginx configuration file
- Creates a Pod that mounts the ConfigMap as a volume
- The configuration file is available at `/etc/nginx/conf.d/default.conf` in the container
- The nginx server uses this configuration file
- This demonstrates how to use ConfigMaps to provide configuration files to applications
- The `items` field allows you to rename the file when mounting it

</p>
</details>

### Exercise 5: Using Secrets for Sensitive Configuration Files

**Goal**: Learn how to use Secrets to provide sensitive configuration files to applications.

**Task**: Create a Secret with a sensitive configuration file and mount it in a Pod.

**Why this matters**: Sensitive configuration files, such as SSL certificates or authentication tokens, should be stored securely. Secrets provide a way to store and manage these files separately from application code and ConfigMaps. Understanding how to use Secrets for configuration files is essential for building secure applications in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a sensitive configuration file**

Create a file named `api-key.json` with the following content:

```json
{
  "api_key": "abcdef123456",
  "api_secret": "xyz789",
  "endpoint": "https://api.example.com",
  "timeout": 30
}
```

**Step 2: Create a Secret from the configuration file**

```bash
kubectl create secret generic api-credentials --from-file=api-key.json
```

**Step 3: Create a Pod that uses the Secret**

Create a file named `api-client-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-client-pod
spec:
  containers:
  - name: api-client
    image: busybox:1.36
    command: ["/bin/sh", "-c", "cat /etc/api/api-key.json && while true; do sleep 3600; done"]
    volumeMounts:
    - name: api-credentials-volume
      mountPath: /etc/api
      readOnly: true
  volumes:
  - name: api-credentials-volume
    secret:
      secretName: api-credentials
```

**Step 4: Create the Pod**

```bash
kubectl apply -f api-client-pod.yaml
```

**Step 5: Verify the configuration file in the Pod**

```bash
kubectl logs api-client-pod
```

**What this does**:

- Creates a Secret containing a sensitive configuration file
- Creates a Pod that mounts the Secret as a volume
- The configuration file is available at `/etc/api/api-key.json` in the container
- The container can read the configuration file
- The `readOnly: true` flag ensures the container cannot modify the Secret
- This demonstrates how to use Secrets to provide sensitive configuration files to applications

</p>
</details>

### Exercise 6: Using Environment Variables from ConfigMaps and Secrets

**Goal**: Learn how to use all environment variables from ConfigMaps and Secrets.

**Task**: Create a Pod that uses all keys from ConfigMaps and Secrets as environment variables.

**Why this matters**: Sometimes you want to expose all configuration values from a ConfigMap or Secret as environment variables without specifying each key individually. This approach is useful when you have many configuration values or when the keys may change over time. Understanding this pattern helps you create more flexible and maintainable configurations.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap with multiple keys**

```bash
kubectl create configmap app-settings \
  --from-literal=DEBUG=true \
  --from-literal=LOG_LEVEL=info \
  --from-literal=API_VERSION=v2 \
  --from-literal=CACHE_TTL=300
```

**Step 2: Create a Secret with multiple keys**

```bash
kubectl create secret generic app-secrets \
  --from-literal=API_KEY=abcdef123456 \
  --from-literal=API_SECRET=xyz789 \
  --from-literal=AUTH_TOKEN=token123
```

**Step 3: Create a Pod that uses all keys from the ConfigMap and Secret**

Create a file named `envfrom-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c", "env | sort && while true; do sleep 3600; done"]
    envFrom:
    - configMapRef:
        name: app-settings
    - secretRef:
        name: app-secrets
```

**Step 4: Create the Pod**

```bash
kubectl apply -f envfrom-pod.yaml
```

**Step 5: Verify the environment variables in the Pod**

```bash
kubectl logs envfrom-pod
```

**What this does**:

- Creates a ConfigMap with multiple configuration values
- Creates a Secret with multiple sensitive values
- Creates a Pod that uses `envFrom` to load all keys from both the ConfigMap and Secret as environment variables
- All keys in the ConfigMap and Secret become environment variables in the container
- This is more concise than specifying each key individually using `env`
- The environment variables from the Secret are automatically decoded

</p>
</details>
