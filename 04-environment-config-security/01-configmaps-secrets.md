# ConfigMaps and Secrets

This section covers ConfigMaps and Secrets, which are Kubernetes resources used to manage configuration data and sensitive information.

## Key Resources

- [ConfigMaps Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)

## ConfigMaps

ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable.

### Exercise 1: Creating and Using ConfigMaps

**Goal**: Learn how to create ConfigMaps and use them in Pods.

**Task**: Create a ConfigMap with configuration data and use it in a Pod as environment variables.

**Why this matters**: Separating configuration from application code follows good development practices and makes applications more portable across different environments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap using the imperative approach**

```bash
# Basic ConfigMap with key-value pairs
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_DEBUG=false

# Multiple ways to create ConfigMaps imperatively:

# From multiple literal values
kubectl create configmap app-settings --from-literal=APP_NAME=MyApp --from-literal=APP_ENV=staging --from-literal=LOG_LEVEL=debug

# From a file (entire file becomes a single key-value pair)
kubectl create configmap logging-config --from-file=log-settings.properties

# From a file with custom key name
kubectl create configmap custom-key-config --from-file=custom-key=config.properties

# From multiple files in a directory
kubectl create configmap multi-file-config --from-file=config-dir/

# Combining different sources
kubectl create configmap hybrid-config --from-literal=ENV=production --from-file=settings.properties
```

> **CKAD Exam Tip:** The imperative command syntax is the fastest way to create ConfigMaps during the exam. Remember that you can combine multiple `--from-literal` and `--from-file` options in a single command to save time.

**Step 2: Verify the ConfigMap**

```bash
kubectl get configmap app-config -o yaml
```

You should see output similar to:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
```

**Step 3: Create a Pod that uses the ConfigMap as environment variables**

Option 1: Using a manifest file (declarative approach):

Create a file named `pod-with-configmap-env.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo APP_ENV=$APP_ENV && echo APP_DEBUG=$APP_DEBUG && sleep 3600"]
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
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-configmap-env.yaml
```

Option 2: Using imperative commands (for simple cases):
```bash
# Using jsonpath to extract values from ConfigMap
kubectl run configmap-env-pod --image=busybox --restart=Never \
  --env="APP_ENV=$(kubectl get configmap app-config -o jsonpath='{.data.APP_ENV}')" \
  --env="APP_DEBUG=$(kubectl get configmap app-config -o jsonpath='{.data.APP_DEBUG}')" \
  -- sh -c "echo APP_ENV=$APP_ENV && echo APP_DEBUG=$APP_DEBUG && sleep 3600"

# Alternative approach using environment variables from ConfigMap directly
# This is a hybrid approach - generate YAML template first
kubectl run configmap-env-pod2 --image=busybox --restart=Never \
  --dry-run=client -o yaml > pod-with-env.yaml

# Then edit the YAML to add ConfigMap references and apply
# (This would be done manually in the exam)
```

> **CKAD Exam Tip:** For complex environment variable configurations from ConfigMaps, the declarative approach is often clearer. However, for simple pods with 1-2 environment variables, the imperative command with jsonpath extraction can save time. Remember that nested subcommands like `$(kubectl get...)` can be error-prone under time pressure, so test carefully.

**Step 4: Check the Pod logs to verify the environment variables**

```bash
kubectl logs configmap-env-pod
```

You should see:

```
APP_ENV=production
APP_DEBUG=false
```

</p>
</details>

### Exercise 2: ConfigMaps from Files

**Goal**: Learn how to create ConfigMaps from files and use them as volumes.

**Task**: Create a ConfigMap from a configuration file and mount it as a volume in a Pod.

**Why this matters**: Many applications read configuration from files rather than environment variables. ConfigMaps allow you to manage these configuration files externally and mount them into containers.

<details><summary>show solution</summary>
<p>

**Step 1: Create configuration files**

Create a file named `app.properties`:

```
app.name=MyApp
app.version=1.0.0
log.level=INFO
```

Create a file named `feature-flags.json`:

```json
{
  "enableFeatureA": true,
  "enableFeatureB": false,
  "maxUsers": 100
}
```

**Step 2: Create a ConfigMap from the files**

```bash
kubectl create configmap app-config-files --from-file=app.properties --from-file=feature-flags.json
```

> Note: This is using the imperative command syntax, which is ideal for creating ConfigMaps from files.

**Step 3: Verify the ConfigMap**

```bash
kubectl get configmap app-config-files -o yaml
```

**Step 4: Create a Pod that mounts the ConfigMap as a volume**

Create a file named `pod-with-configmap-volume.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-vol-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /etc/config/app.properties && echo && cat /etc/config/feature-flags.json && sleep 3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config-files
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-configmap-volume.yaml
```

**Step 5: Check the Pod logs to verify the mounted files**

```bash
kubectl logs configmap-vol-pod
```

You should see the contents of both files.

</p>
</details>

## Secrets

Secrets are similar to ConfigMaps but are designed to hold sensitive information such as passwords, OAuth tokens, and SSH keys.

### Exercise 1: Creating and Using Secrets

**Goal**: Learn how to create Secrets and use them in Pods.

**Task**: Create a Secret with sensitive data and use it in a Pod as environment variables.

**Why this matters**: Storing sensitive information separately from application code and container images is a security best practice. Secrets provide a way to manage this sensitive information.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Secret with sensitive data**

```bash
# Basic Secret with key-value pairs
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=supersecret

# Multiple ways to create Secrets imperatively:

# From multiple literal values
kubectl create secret generic api-credentials \
  --from-literal=API_KEY=abcdef123456 \
  --from-literal=API_SECRET=xyzpdq987654

# From files (useful for certificates, keys, etc.)
kubectl create secret generic tls-certs \
  --from-file=tls.key \
  --from-file=tls.crt

# Creating TLS secret type (special format for TLS certificates)
kubectl create secret tls ingress-tls \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# Creating Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email@example.com
```

> **CKAD Exam Tip:** Always use the imperative commands for creating Secrets in the exam - they're faster and avoid the need to manually encode values in base64. Remember that Kubernetes has different secret types (`generic`, `tls`, and `docker-registry`) with specific command formats for each.

**Step 2: Verify the Secret**

```bash
kubectl get secret db-credentials -o yaml
```

Note that the values are Base64 encoded.

**Step 3: Create a Pod that uses the Secret as environment variables**

Create a file named `pod-with-secret-env.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo Database User: $DB_USERNAME && echo Database Password: $DB_PASSWORD && sleep 3600"]
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
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-secret-env.yaml
```

**Step 4: Check the Pod logs to verify the environment variables**

```bash
kubectl logs secret-env-pod
```

You should see:

```
Database User: admin
Database Password: supersecret
```

</p>
</details>

### Exercise 2: Secrets from Files and Mounting as Volumes

**Goal**: Learn how to create Secrets from files and mount them as volumes.

**Task**: Create a Secret from files containing sensitive data and mount it as a volume in a Pod.

**Why this matters**: Many applications require sensitive files such as TLS certificates or configuration files with embedded secrets. Kubernetes Secrets allow you to manage these files securely.

<details><summary>show solution</summary>
<p>

**Step 1: Create files with sensitive data**

Create a file named `tls.key`:

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7VJTUt9Us8cKj
MzEfYyjiWA4R4/M2bS1GB4t7NXp98C3SC6dVMvDuictGeurT8jNbvJZHtCSuYEvu
NMoSfm76oqFvAp8Gy0iz5sxjZmSnXyCdPEovGhLa0VzMaQ8s+CLOyS56YyCFGeJZ
...
-----END PRIVATE KEY-----
```

Create a file named `tls.crt`:

```
-----BEGIN CERTIFICATE-----
MIIDETCCAfmgAwIBAgIJALMb7ecMIk3MMA0GCSqGSIb3DQEBCwUAMB4xHDAaBgNV
BAMME3d3dy5leGFtcGxlY29tLmNvbTAeFw0yMDEyMDcxMjU0MjdaFw0yMTEyMDcx
MjU0MjdaMB4xHDAaBgNVBAMME3d3dy5leGFtcGxlY29tLmNvbTCCASIwDQYJKoZI
...
-----END CERTIFICATE-----
```

**Step 2: Create a Secret from the files**

```bash
kubectl create secret tls example-tls --key=tls.key --cert=tls.crt
```

> Note: This is using the imperative command syntax for creating TLS secrets. Other types of secrets can be created with:
> - `kubectl create secret generic` - for generic secrets from files or literal values
> - `kubectl create secret docker-registry` - for Docker registry credentials

**Step 3: Verify the Secret**

```bash
kubectl get secret example-tls -o yaml
```

**Step 4: Create a Pod that mounts the Secret as a volume**

Create a file named `pod-with-secret-volume.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: tls-certs
      mountPath: "/etc/nginx/ssl"
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: example-tls
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-secret-volume.yaml
```

**Step 5: Verify that the files are mounted**

```bash
kubectl exec secret-vol-pod -- ls -la /etc/nginx/ssl
```

You should see the certificate and key files.

</p>
</details>

### Exercise 3: Using envFrom to Load All ConfigMap or Secret Values

**Goal**: Learn how to load all values from a ConfigMap or Secret as environment variables.

**Task**: Create a ConfigMap and a Secret, then load all their values as environment variables in a Pod.

**Why this matters**: Sometimes you need to load multiple configuration values or secrets into your application. Using `envFrom` simplifies this process by loading all values from a ConfigMap or Secret at once.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap with multiple values**

```bash
kubectl create configmap app-settings --from-literal=APP_NAME=MyApp --from-literal=APP_ENV=staging --from-literal=LOG_LEVEL=debug
```

**Step 2: Create a Secret with multiple values**

```bash
kubectl create secret generic app-secrets --from-literal=API_KEY=abcdef123456 --from-literal=AUTH_TOKEN=xyz789 --from-literal=ENCRYPTION_KEY=secretkey
```

**Step 3: Create a Pod that loads all values using envFrom**

Create a file named `pod-with-envfrom.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env | sort && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-settings
    - secretRef:
        name: app-secrets
  restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pod-with-envfrom.yaml
```

**Step 4: Check the Pod logs to verify the environment variables**

```bash
kubectl logs envfrom-pod
```

You should see all the environment variables from both the ConfigMap and Secret.

</p>
</details>

### Exercise 4: Updating ConfigMaps and Secrets

**Goal**: Learn how ConfigMap and Secret updates affect running Pods.

**Task**: Update a ConfigMap and observe how it affects a Pod that uses it.

**Why this matters**: Understanding how configuration updates propagate to applications is crucial for managing application configuration in production environments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap**

```bash
kubectl create configmap dynamic-config --from-literal=INTERVAL=10 --from-literal=MODE=standard
```

**Step 2: Create a Pod that uses the ConfigMap as a volume**

Create a file named `pod-with-dynamic-config.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "while true; do cat /etc/config/INTERVAL /etc/config/MODE; sleep 10; done"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: dynamic-config
```

Apply the configuration:

```bash
kubectl apply -f pod-with-dynamic-config.yaml
```

**Step 3: Check the Pod logs**

```bash
kubectl logs dynamic-config-pod -f
```

You should see the values `10` and `standard` being printed every 10 seconds.

**Step 4: Update the ConfigMap**

Option 1: Using the edit command (interactive):
```bash
kubectl edit configmap dynamic-config
```

Change the values to:
```yaml
data:
  INTERVAL: "30"
  MODE: "advanced"
```

Option 2: Using imperative commands (non-interactive):
```bash
kubectl patch configmap dynamic-config -p '{"data":{"INTERVAL":"30","MODE":"advanced"}}'
```

Option 3: Using delete and recreate approach:
```bash
kubectl delete configmap dynamic-config
kubectl create configmap dynamic-config --from-literal=INTERVAL=30 --from-literal=MODE=advanced
```

> Note: The patch command is particularly useful for automation and scripting scenarios.

**Step 5: Wait a moment and observe the Pod logs**

After a short delay (usually less than a minute), you should see the values change to `30` and `advanced`.

**Note**: ConfigMap updates are eventually reflected in volume mounts, but not in environment variables. Pods need to be restarted to pick up environment variable changes.

</p>
</details>

## Best Practices

1. **Use ConfigMaps for non-sensitive configuration data** and **Secrets for sensitive data**.
2. **Keep ConfigMaps and Secrets small** to avoid performance issues.
3. **Use volume mounts for configuration files** and **environment variables for simple values**.
4. **Consider using Helm or Kustomize** for managing ConfigMaps and Secrets across environments.
5. **Use immutable ConfigMaps and Secrets** for critical configuration that should not change.
6. **Be aware that Secret values are only Base64 encoded**, not encrypted. Use additional encryption mechanisms for highly sensitive data.
7. **Remember that updating a ConfigMap or Secret does not automatically update Pods** that use them as environment variables. You need to restart the Pods.
8. **Use resource quotas** to limit the number and size of ConfigMaps and Secrets in a namespace.

## CKAD Exam Tips for ConfigMaps and Secrets

### Imperative Command Quick Reference

**ConfigMaps:**
```bash
# Basic ConfigMap
kubectl create configmap <name> --from-literal=<key>=<value>

# From file
kubectl create configmap <name> --from-file=<path-to-file>

# From directory
kubectl create configmap <name> --from-file=<directory-path>
```

**Secrets:**
```bash
# Generic Secret
kubectl create secret generic <name> --from-literal=<key>=<value>

# TLS Secret
kubectl create secret tls <name> --cert=<path-to-cert> --key=<path-to-key>

# Docker Registry Secret
kubectl create secret docker-registry <name> --docker-server=<server> --docker-username=<username> --docker-password=<password>
```

### When to Use Each Approach

**Use Imperative Commands When:**
- Creating simple ConfigMaps and Secrets with a few key-value pairs
- Creating resources from existing files
- Time is limited and you need to create resources quickly

**Use Declarative YAML When:**
- You need complex configurations with many values
- You're creating multiple related resources together
- You need to use advanced features like immutable ConfigMaps

**Use the Hybrid Approach When:**
- You need a starting point for a complex configuration
- You want to avoid syntax errors in your YAML

```bash
# Generate a ConfigMap YAML template
kubectl create configmap <name> --from-literal=<key>=<value> --dry-run=client -o yaml > configmap.yaml

# Generate a Secret YAML template
kubectl create secret generic <name> --from-literal=<key>=<value> --dry-run=client -o yaml > secret.yaml
```

### Exam Time-Saving Tips

1. **Use imperative commands for simple cases** - they're faster than writing YAML
2. **Use the hybrid approach for complex cases** - generate a template and modify it
3. **Remember that Secret values must be base64-encoded in YAML** - use imperative commands to avoid manual encoding
4. **For environment variables from ConfigMaps/Secrets:**
   - Use `envFrom` when you need all values
   - Use `valueFrom` with `configMapKeyRef`/`secretKeyRef` for specific values
5. **For volume mounts:**
   - Mount specific keys using the `items` field
   - Use `subPath` to avoid overwriting directories
