# Network Policies

This section covers Kubernetes Network Policies, which provide security controls for Pod network traffic.

## Key Resources

- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

## Introduction to Network Policies

Network Policies in Kubernetes allow you to control the traffic flow between Pods and between Pods and external endpoints. They act as a firewall for Pods, specifying which traffic is allowed and which is denied.

## Imperative vs. Declarative Approaches for Network Policies

**Important Note for CKAD Exam:** Network Policies can **only** be created using declarative YAML manifests. There are no imperative commands available for creating Network Policies. This is a key distinction for the CKAD exam.

**Declarative Approach (Required):**

- Network Policies must be defined in YAML manifests
- Apply the manifests using `kubectl apply -f policy.yaml`
- This is the only approach available for Network Policies

**Supporting Resources (Can use Imperative Commands):**

- While Network Policies themselves require YAML, the resources they protect can be created imperatively:
  - Create namespaces: `kubectl create namespace`
  - Create pods: `kubectl run`
  - Create deployments: `kubectl create deployment`
  - Create services: `kubectl expose`

**Hybrid Workflow for CKAD Exam:**

1. Create supporting resources (pods, deployments, services) using imperative commands
2. Create Network Policies using YAML manifests
3. Test the policies using imperative commands like `kubectl exec`

## Key Concepts

- **Network Policy**: A specification of how groups of Pods are allowed to communicate with each other and other network endpoints
- **Ingress**: Rules controlling incoming traffic to Pods
- **Egress**: Rules controlling outgoing traffic from Pods
- **Selectors**: Used to identify the Pods to which a policy applies and the Pods that are allowed to communicate
- **Default Deny**: By default, Pods are non-isolated and accept traffic from any source

## Network Policies

### Exercise 1: Creating a Default Deny Policy

**Goal**: Learn how to create a default deny Network Policy that blocks all ingress traffic to Pods in a namespace.

**Task**: Create a Network Policy that denies all ingress traffic to Pods in a namespace.

**Why this matters**: A default deny policy is a security best practice that establishes a zero-trust network model. This ensures that only explicitly allowed traffic can reach your Pods, reducing the attack surface of your applications. Starting with a default deny policy and then adding specific allow rules is a fundamental approach to network security.

<details><summary>show solution</summary>
<p>

**Step 1: Create a new namespace for testing**

```bash
kubectl create namespace policy-test
```

**Step 2: Create a Pod in the namespace**

```bash
# Create pod with label using imperative command
kubectl run web --image=nginx --namespace=policy-test --labels=app=web --port=80

# Create service for the pod using imperative command
kubectl expose pod web --namespace=policy-test --port=80
```

> **CKAD Exam Tip:** While Network Policies require YAML manifests, you can create all the supporting resources (pods, services) using imperative commands to save time. The imperative commands shown here are much faster than writing YAML for these basic resources.
>
> Alternative approaches:
>
> - `kubectl run web --image=nginx --namespace=policy-test --labels=app=web --port=80 --expose` (creates both pod and service)
> - Generate a service YAML: `kubectl expose pod web --namespace=policy-test --port=80 --dry-run=client -o yaml > service.yaml`

**Step 3: Verify that the Pod is accessible**

```bash
kubectl run test-pod --namespace=policy-test --image=busybox:1.36 --rm -it -- wget -qO- web
```

You should see the HTML output from the nginx server.

**Step 4: Create a default deny Network Policy**

> **Important for CKAD Exam:** Network Policies must be created using YAML manifests as there are no imperative commands available for creating them. Unlike many other Kubernetes resources, there is no `kubectl create networkpolicy` command. You must write the YAML manifest and apply it with `kubectl apply -f`.

Create a file named `default-deny.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: policy-test
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

**Step 5: Apply the Network Policy**

```bash
kubectl apply -f default-deny.yaml
```

> **CKAD Exam Tip:** There is no imperative alternative to this step. Network Policies must always be created using YAML manifests and applied with `kubectl apply -f`. Remember this distinction when planning your approach to network security questions in the exam.

**Step 6: Verify that the Pod is no longer accessible**

```bash
kubectl run test-pod --namespace=policy-test --image=busybox:1.36 --rm -it -- wget -qO- --timeout=5 web
```

This command should time out, indicating that the traffic is blocked.

**What this does**:

- Creates a Network Policy that selects all Pods in the namespace (`podSelector: {}`)
- Specifies that the policy applies to ingress traffic (`policyTypes: ["Ingress"]`)
- Since no ingress rules are specified, all ingress traffic is denied
- The Pod is no longer accessible from other Pods in the namespace
- This establishes a zero-trust network model where traffic must be explicitly allowed

</p>
</details>

### Exercise 2: Allowing Specific Ingress Traffic

**Goal**: Learn how to create a Network Policy that allows specific ingress traffic to Pods.

**Task**: Create a Network Policy that allows ingress traffic to a web server Pod only from specific Pods.

**Why this matters**: Allowing specific ingress traffic is essential for implementing the principle of least privilege in your network security. This ensures that Pods can only receive traffic from authorized sources, reducing the risk of unauthorized access and potential attacks. This pattern is commonly used to secure microservices communication.

<details><summary>show solution</summary>
<p>

**Step 1: Create a web server Deployment and Service**

```bash
# Create namespace using imperative command
kubectl create namespace web-policy

# Create deployment using imperative command
kubectl create deployment web --image=nginx --namespace=web-policy --labels=app=web

# Create service using imperative command
kubectl expose deployment web --namespace=web-policy --port=80
```

> Note: These are all using the imperative command syntax, which is recommended for the CKAD exam for quickly creating resources.

**Step 2: Create client Deployments with different labels**

```bash
# Create client deployments with different labels using imperative commands
kubectl create deployment allowed-client --image=busybox:1.36 --namespace=web-policy --labels=app=client,access=allowed -- sleep 3600
kubectl create deployment blocked-client --image=busybox:1.36 --namespace=web-policy --labels=app=client,access=blocked -- sleep 3600
```

> Note: The kubectl create deployment command allows you to set multiple labels using comma-separated key=value pairs, which is very useful for the CKAD exam.

**Step 3: Verify that both clients can access the web server**

```bash
kubectl exec -n web-policy deployment/allowed-client -- wget -qO- --timeout=5 web
kubectl exec -n web-policy deployment/blocked-client -- wget -qO- --timeout=5 web
```

Both commands should return the HTML output from the nginx server.

**Step 4: Create a Network Policy that allows traffic only from allowed clients**

> Note: For Network Policies, you must use YAML manifests. There is no imperative command equivalent in kubectl for creating Network Policies. Understanding how to write these manifests is essential for the CKAD exam.

Create a file named `web-allow-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-allow-from-allowed-clients
  namespace: web-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
```

**Step 5: Apply the Network Policy**

```bash
kubectl apply -f web-allow-policy.yaml
```

**Step 6: Verify that only the allowed client can access the web server**

```bash
kubectl exec -n web-policy deployment/allowed-client -- wget -qO- --timeout=5 web
kubectl exec -n web-policy deployment/blocked-client -- wget -qO- --timeout=5 web
```

The first command should succeed, but the second command should time out.

**What this does**:

- Creates a Network Policy that selects Pods with the label `app: web`
- Allows ingress traffic only from Pods with the label `access: allowed`
- Allows traffic only on TCP port 80
- The allowed client can still access the web server
- The blocked client can no longer access the web server
- This implements the principle of least privilege for network access

</p>
</details>

### Exercise 3: Allowing Ingress Traffic from a Namespace

**Goal**: Learn how to create a Network Policy that allows ingress traffic from Pods in a specific namespace.

**Task**: Create a Network Policy that allows ingress traffic to a database Pod only from Pods in a specific namespace.

**Why this matters**: In multi-tenant Kubernetes clusters, controlling traffic between namespaces is crucial for maintaining proper isolation and security. This pattern allows you to implement namespace-based network segmentation, ensuring that services in one namespace can only be accessed by authorized services from specific namespaces.

<details><summary>show solution</summary>
<p>

**Step 1: Create namespaces for the database and clients**

```bash
kubectl create namespace db-namespace
kubectl create namespace allowed-namespace
kubectl create namespace blocked-namespace
```

**Step 2: Create a database Pod and Service**

```bash
kubectl run db --image=mysql:8.0 --namespace=db-namespace --labels=app=db --env="MYSQL_ROOT_PASSWORD=password"
kubectl expose pod db --namespace=db-namespace --port=3306
```

**Step 3: Create client Pods in different namespaces**

```bash
kubectl run allowed-client --image=busybox:1.36 --namespace=allowed-namespace -- sleep 3600
kubectl run blocked-client --image=busybox:1.36 --namespace=blocked-namespace -- sleep 3600
```

**Step 4: Create a Network Policy that allows traffic only from the allowed namespace**

Create a file named `db-namespace-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-from-allowed-namespace
  namespace: db-namespace
spec:
  podSelector:
    matchLabels:
      app: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: allowed-namespace
      ports:
        - protocol: TCP
          port: 3306
```

**Step 5: Apply the Network Policy**

```bash
kubectl apply -f db-namespace-policy.yaml
```

**Step 6: Label the allowed namespace**

```bash
kubectl label namespace allowed-namespace kubernetes.io/metadata.name=allowed-namespace
```

**Step 7: Verify that only the allowed client can access the database**

```bash
kubectl exec -n allowed-namespace allowed-client -- nc -zv db.db-namespace.svc.cluster.local 3306
kubectl exec -n blocked-namespace blocked-client -- nc -zv db.db-namespace.svc.cluster.local 3306
```

The first command should succeed, but the second command should fail.

**What this does**:

- Creates a Network Policy that selects Pods with the label `app: db`
- Allows ingress traffic only from Pods in the namespace with the label `kubernetes.io/metadata.name: allowed-namespace`
- Allows traffic only on TCP port 3306
- The client in the allowed namespace can access the database
- The client in the blocked namespace cannot access the database
- This implements namespace-based network segmentation

</p>
</details>

### Exercise 4: Controlling Egress Traffic

**Goal**: Learn how to create a Network Policy that controls outgoing traffic from Pods.

**Task**: Create a Network Policy that allows egress traffic from a Pod only to specific destinations.

**Why this matters**: Controlling egress traffic is essential for preventing data exfiltration and limiting the impact of compromised applications. By restricting where your Pods can send traffic, you can enforce security policies that prevent unauthorized communication with external services or other parts of your infrastructure.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace for testing**

```bash
kubectl create namespace egress-test
```

**Step 2: Create a client Pod**

```bash
kubectl run client --image=busybox:1.36 --namespace=egress-test -- sleep 3600
```

**Step 3: Create destination Pods**

```bash
kubectl run allowed-dest --image=nginx --namespace=egress-test --labels=access=allowed --expose --port=80
kubectl run blocked-dest --image=nginx --namespace=egress-test --labels=access=blocked --expose --port=80
```

**Step 4: Verify that the client can access both destinations**

```bash
kubectl exec -n egress-test client -- wget -qO- --timeout=5 allowed-dest
kubectl exec -n egress-test client -- wget -qO- --timeout=5 blocked-dest
```

Both commands should return the HTML output from the nginx server.

**Step 5: Create a Network Policy that allows egress traffic only to allowed destinations**

Create a file named `egress-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: client-egress-policy
  namespace: egress-test
spec:
  podSelector:
    matchLabels:
      run: client
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53
```

**Step 6: Apply the Network Policy**

```bash
kubectl apply -f egress-policy.yaml
```

**Step 7: Verify that the client can only access the allowed destination**

```bash
kubectl exec -n egress-test client -- wget -qO- --timeout=5 allowed-dest
kubectl exec -n egress-test client -- wget -qO- --timeout=5 blocked-dest
```

The first command should succeed, but the second command should time out.

**What this does**:

- Creates a Network Policy that selects the client Pod
- Allows egress traffic only to Pods with the label `access: allowed`
- Allows egress traffic to the DNS service in the kube-system namespace (necessary for DNS resolution)
- The client can access the allowed destination
- The client cannot access the blocked destination
- This implements the principle of least privilege for outgoing network traffic

</p>
</details>

### Exercise 5: Combining Ingress and Egress Policies

**Goal**: Learn how to create a Network Policy that controls both incoming and outgoing traffic for Pods.

**Task**: Create a Network Policy that allows specific ingress and egress traffic for a Pod.

**Why this matters**: Comprehensive network security requires controlling both incoming and outgoing traffic. By combining ingress and egress policies, you can create a complete security boundary around your applications, ensuring that they can only communicate with authorized services in both directions. This is essential for implementing defense-in-depth security strategies.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace for testing**

```bash
kubectl create namespace combined-policy
```

**Step 2: Create an application Pod**

```bash
kubectl run app --image=nginx --namespace=combined-policy --labels=app=web --expose --port=80
```

**Step 3: Create client Pods with different labels**

```bash
kubectl run allowed-client --image=busybox:1.36 --namespace=combined-policy --labels=app=client,access=allowed -- sleep 3600
kubectl run blocked-client --image=busybox:1.36 --namespace=combined-policy --labels=app=client,access=blocked -- sleep 3600
```

**Step 4: Create destination Pods with different labels**

```bash
kubectl run allowed-dest --image=nginx --namespace=combined-policy --labels=app=dest,access=allowed --expose --port=80
kubectl run blocked-dest --image=nginx --namespace=combined-policy --labels=app=dest,access=blocked --expose --port=80
```

**Step 5: Create a Network Policy that controls both ingress and egress traffic**

Create a file named `combined-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
  namespace: combined-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
  egress:
    - to:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53
```

**Step 6: Apply the Network Policy**

```bash
kubectl apply -f combined-policy.yaml
```

**Step 7: Verify ingress traffic control**

```bash
kubectl exec -n combined-policy allowed-client -- wget -qO- --timeout=5 app
kubectl exec -n combined-policy blocked-client -- wget -qO- --timeout=5 app
```

The first command should succeed, but the second command should time out.

**Step 8: Verify egress traffic control**

```bash
kubectl exec -n combined-policy app -- wget -qO- --timeout=5 allowed-dest
kubectl exec -n combined-policy app -- wget -qO- --timeout=5 blocked-dest
```

The first command should succeed, but the second command should time out.

**What this does**:

- Creates a Network Policy that selects Pods with the label `app: web`
- Controls both ingress and egress traffic (`policyTypes: ["Ingress", "Egress"]`)
- Allows ingress traffic only from Pods with the label `access: allowed`
- Allows egress traffic only to Pods with the label `access: allowed`
- Allows egress traffic to the DNS service in the kube-system namespace
- This creates a complete security boundary around the application
- Only authorized clients can access the application, and the application can only access authorized destinations

</p>
</details>

### Exercise 6: Allowing Traffic Based on IP Blocks

**Goal**: Learn how to create a Network Policy that allows traffic based on IP CIDR blocks.

**Task**: Create a Network Policy that allows traffic from specific IP ranges.

**Why this matters**: In many real-world scenarios, you need to allow traffic from external sources that can't be identified by Pod or namespace selectors. IP-based Network Policies allow you to control access from external clients, on-premises networks, or other cloud resources. This is essential for securing applications that need to be accessible from outside the cluster.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace for testing**

```bash
kubectl create namespace ip-policy
```

**Step 2: Create a web server Pod and Service**

```bash
kubectl run web --image=nginx --namespace=ip-policy --labels=app=web
kubectl expose pod web --namespace=ip-policy --port=80 --type=NodePort
```

**Step 3: Create a Network Policy that allows traffic from specific Pods**

Create a file named `web-allow-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-allow-from-pods
  namespace: ip-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: allowed
      ports:
        - protocol: TCP
          port: 80
```

**Step 4: Apply the Network Policy**

```bash
kubectl apply -f web-allow-policy.yaml
```

**Step 5: Create a Network Policy that allows traffic from specific IP ranges**

Create a file named `ip-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-allow-from-ip-ranges
  namespace: ip-policy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.0.0.0/16
            except:
              - 10.0.5.0/24
      ports:
        - protocol: TCP
          port: 80
```

**Step 4: Apply the Network Policy**

```bash
kubectl apply -f ip-policy.yaml
```

**What this does**:

- Creates a Network Policy that selects Pods with the label `app: web`
- Allows ingress traffic only from the IP range `10.0.0.0/16`
- Excludes the IP range `10.0.5.0/24` from the allowed range
- Allows traffic only on TCP port 80
- This allows you to control access based on source IP addresses
- This is useful for allowing traffic from external networks or specific clients

**Note**: Testing this policy requires access to the cluster from the specified IP ranges. In a real environment, you would use IP ranges that correspond to your corporate network, VPN, or other trusted sources.

</p>
</details>

### Exercise 7: Creating a Network Policy for Multiple Port Protocols

**Goal**: Learn how to create a Network Policy that allows traffic on multiple ports and protocols.

**Task**: Create a Network Policy that allows traffic to a Pod on multiple ports and protocols.

**Why this matters**: Many applications expose multiple ports for different purposes, such as HTTP, HTTPS, and admin interfaces. Creating Network Policies that allow traffic on multiple ports and protocols is essential for properly securing these applications while ensuring they remain functional. This pattern is commonly used for applications that provide multiple services.

<details><summary>show solution</summary>
<p>

**Step 1: Create a namespace for testing**

```bash
kubectl create namespace multi-port-policy
```

**Step 2: Create a Pod that listens on multiple ports**

```bash
kubectl run multi-port-app --image=nginx --namespace=multi-port-policy --labels=app=multi-port
kubectl expose pod multi-port-app --namespace=multi-port-policy --port=80,443 --name=multi-port-service
```

**Step 3: Create a Network Policy that allows traffic on multiple ports and protocols**

Create a file named `multi-port-policy.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-port-policy
  namespace: multi-port-policy
spec:
  podSelector:
    matchLabels:
      app: multi-port
  policyTypes:
    - Ingress
  ingress:
    - ports:
        - protocol: TCP
          port: 80
        - protocol: TCP
          port: 443
        - protocol: UDP
          port: 53
    - from:
        - podSelector:
            matchLabels:
              role: monitoring
      ports:
        - protocol: TCP
          port: 9090
```

**Step 4: Apply the Network Policy**

```bash
kubectl apply -f multi-port-policy.yaml
```

**What this does**:

- Creates a Network Policy that selects Pods with the label `app: multi-port`
- Allows ingress traffic on TCP ports 80 and 443, and UDP port 53 from any source
- Allows ingress traffic on TCP port 9090 only from Pods with the label `role: monitoring`
- This demonstrates how to create more complex Network Policies that allow traffic on multiple ports and protocols
- The policy also shows how to combine port rules with source selectors for fine-grained control

</p>
</details>

## CKAD Exam Tips for Network Policies

### Quick Reference for Network Policy Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: policy-name
  namespace: target-namespace
spec:
  # Which pods does this policy apply to?
  podSelector:
    matchLabels:
      app: my-app

  # What types of traffic does this policy control?
  policyTypes:
    - Ingress # Incoming traffic
    - Egress # Outgoing traffic

  # Rules for incoming traffic
  ingress:
    - from:
        # Pod selector: from pods with specific labels
        - podSelector:
            matchLabels:
              role: frontend

        # Namespace selector: from pods in specific namespaces
        - namespaceSelector:
            matchLabels:
              project: myproject

        # IP block: from specific IP ranges
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24

      # Ports allowed
      ports:
        - protocol: TCP
          port: 80
        - protocol: TCP
          port: 443

  # Rules for outgoing traffic
  egress:
    - to:
        - podSelector:
            matchLabels:
              role: database
      ports:
        - protocol: TCP
          port: 5432
```

### Common Network Policy Patterns for CKAD

**1. Default Deny All Ingress Traffic:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: your-namespace
spec:
  podSelector: {} # Applies to all pods in the namespace
  policyTypes:
    - Ingress
```

**2. Default Deny All Egress Traffic:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: your-namespace
spec:
  podSelector: {} # Applies to all pods in the namespace
  policyTypes:
    - Egress
```

**3. Allow Traffic from Specific Namespace:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-namespace
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              project: frontend
```

**4. Allow Traffic to Specific External Endpoints:**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-egress
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24 # External service IP range
```

### CKAD Exam Strategy for Network Policies

1. **Remember the hybrid approach**:

   - Create pods, deployments, and services using imperative commands
   - Create Network Policies using YAML manifests

2. **Test your policies** using temporary pods:

   ```bash
   # Test connectivity to a service
   kubectl run test-pod --image=busybox:1.36 --rm -it -- wget -qO- --timeout=5 service-name

   # Test connectivity to a specific pod IP
   kubectl run test-pod --image=busybox:1.36 --rm -it -- wget -qO- --timeout=5 10.244.1.5
   ```

3. **Troubleshoot Network Policies**:

   ```bash
   # List all Network Policies
   kubectl get networkpolicies --all-namespaces

   # Describe a Network Policy
   kubectl describe networkpolicy policy-name -n namespace
   ```

4. **Remember the selector combinations**:
   - `podSelector`: Select pods by labels
   - `namespaceSelector`: Select namespaces by labels
   - `ipBlock`: Select IP ranges
   - Combining selectors:
     - `from: [{podSelector}, {namespaceSelector}]` = AND (both must match)
     - `from: [{podSelector}], [{namespaceSelector}]` = OR (either can match)

### Time-Saving Tips for the CKAD Exam

1. **Create a template** for common Network Policy patterns and modify as needed

2. **Use labels consistently** across your resources to make selector configuration easier

3. **Start with a default deny policy** and then add specific allow rules

4. **Test incrementally** after applying each policy to verify it works as expected

5. **Remember the limitations** of Network Policies:

   - They are namespace-scoped
   - They require a CNI plugin that supports Network Policies
   - They are additive (multiple policies can apply to the same pods)

6. **Know the syntax differences** between ingress and egress rules:

   - Ingress uses `from:` to specify sources
   - Egress uses `to:` to specify destinations

7. **Use meaningful policy names** that describe their function:
   - `web-allow-from-api`
   - `db-deny-external-egress`
   - `default-deny-all`
