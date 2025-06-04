# Deployment Strategies

This section covers various deployment strategies in Kubernetes, which are essential for updating applications with minimal or zero downtime.

## Key Resources

- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Canary Deployments](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)

## Introduction to Deployment Strategies

Deployment strategies define how updates are rolled out to your application. Kubernetes provides several built-in strategies, and you can implement more advanced patterns as needed. Choosing the right strategy depends on your application's requirements for availability, risk tolerance, and user experience during updates.

## Key Concepts

- **Rolling Updates**: Gradually replace old instances with new ones
- **Recreate**: Terminate all existing instances before creating new ones
- **Blue/Green**: Deploy new version alongside old version, then switch traffic
- **Canary**: Direct a small percentage of traffic to the new version before full rollout
- **A/B Testing**: Route traffic to different versions based on specific criteria

## Deployment Strategies

### Exercise 1: Rolling Updates

**Goal**: Learn how to perform a rolling update of a Deployment.

**Task**: Create a Deployment and update it using the rolling update strategy.

**Why this matters**: Rolling updates are the default and most common deployment strategy in Kubernetes. They allow you to update your application with zero downtime by gradually replacing old instances with new ones. This is crucial for maintaining service availability during updates in production environments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment manifest file**

Create a file named `app-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

**Step 2: Create the Deployment**

```bash
kubectl apply -f app-deployment.yaml
```

**Step 3: Verify the Deployment**

```bash
kubectl get deployments
kubectl get pods -l app=web-app
```

**Step 4: Update the Deployment to use a new image version**

```bash
kubectl set image deployment/web-app nginx=nginx:1.21
```

**Step 5: Watch the rolling update in progress**

```bash
kubectl rollout status deployment/web-app
```

**Step 6: Verify the update**

```bash
kubectl describe deployment web-app
```

**What this does**:

- Creates a Deployment with 4 replicas running nginx:1.20
- Configures a rolling update strategy with:
  - `maxSurge: 1`: At most 1 Pod above the desired number of Pods
  - `maxUnavailable: 1`: At most 1 Pod below the desired number of Pods
- Updates the Deployment to use nginx:1.21
- The Deployment controller gradually replaces old Pods with new ones
- At any point during the update, at least 3 Pods are available (desired - maxUnavailable)

</p>
</details>

### Exercise 2: Rollback a Deployment

**Goal**: Learn how to roll back a Deployment to a previous revision.

**Task**: Update a Deployment and then roll it back when issues are detected.

**Why this matters**: The ability to quickly roll back to a previous known-good state is crucial for maintaining service reliability. Kubernetes' built-in rollback capabilities provide a safety net when deployments introduce unexpected issues, allowing you to restore service quickly while you investigate and fix the problems.

<details><summary>show solution</summary>
<p>

**Step 1: Check the current revision history**

```bash
kubectl rollout history deployment/web-app
```

**Step 2: Update the Deployment to a problematic version**

```bash
kubectl set image deployment/web-app nginx=nginx:nonexistent
```

**Step 3: Check the status of the rollout**

```bash
kubectl rollout status deployment/web-app
```

You'll notice the rollout gets stuck because the image doesn't exist.

**Step 4: Verify that some Pods are failing**

```bash
kubectl get pods -l app=web-app
```

You should see some Pods in `ImagePullBackOff` or `ErrImagePull` state.

**Step 5: Roll back to the previous revision**

```bash
kubectl rollout undo deployment/web-app
```

**Step 6: Verify the rollback**

```bash
kubectl rollout status deployment/web-app
kubectl get pods -l app=web-app
kubectl describe deployment web-app
```

**What this does**:

- Checks the revision history of the Deployment
- Updates the Deployment to use a non-existent image, causing the rollout to fail
- Rolls back the Deployment to the previous revision (nginx:1.21)
- The Deployment controller restores the previous configuration
- All Pods eventually return to a running state with the previous image

</p>
</details>

### Exercise 3: Recreate Deployment Strategy

**Goal**: Learn how to use the Recreate deployment strategy.

**Task**: Configure a Deployment to use the Recreate strategy and observe its behavior during updates.

**Why this matters**: While not ideal for high-availability applications, the Recreate strategy is useful for applications that can't run multiple versions simultaneously or require a clean slate for updates. Understanding this strategy helps you handle applications with version incompatibility issues or those requiring a complete restart during updates.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment manifest file with Recreate strategy**

Create a file named `recreate-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recreate-app
  labels:
    app: recreate-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: recreate-app
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: recreate-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

**Step 2: Create the Deployment**

```bash
kubectl apply -f recreate-deployment.yaml
```

**Step 3: Verify the Deployment**

```bash
kubectl get deployments
kubectl get pods -l app=recreate-app
```

**Step 4: Update the Deployment to use a new image version**

```bash
kubectl set image deployment/recreate-app nginx=nginx:1.21
```

**Step 5: Watch the update in progress**

```bash
kubectl get pods -l app=recreate-app -w
```

**What this does**:

- Creates a Deployment with 3 replicas running nginx:1.20
- Configures the Recreate strategy
- Updates the Deployment to use nginx:1.21
- The Deployment controller:
  1. Terminates all existing Pods
  2. Creates new Pods with the updated image
- During the update, there is a period of downtime where no Pods are available
- This strategy is suitable for applications that:
  - Cannot run multiple versions simultaneously
  - Require a clean slate for updates
  - Can tolerate downtime during updates

</p>
</details>

### Exercise 4: Canary Deployment

**Goal**: Learn how to implement a canary deployment pattern.

**Task**: Deploy a new version of an application to a small subset of users before rolling it out to everyone.

**Why this matters**: Canary deployments are a powerful risk-mitigation strategy that allows you to test new versions with a small percentage of real traffic. This helps detect issues early with minimal impact, making it essential for safely deploying changes to production systems with high reliability requirements.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Service to route traffic to the application**

Create a file named `canary-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: canary-service
spec:
  selector:
    app: canary-app
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Create the stable Deployment**

Create a file named `stable-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stable-app
  labels:
    app: stable-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: canary-app
      version: stable
  template:
    metadata:
      labels:
        app: canary-app
        version: stable
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

**Step 3: Create the canary Deployment**

Create a file named `canary-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: canary-app
  labels:
    app: canary-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: canary-app
      version: canary
  template:
    metadata:
      labels:
        app: canary-app
        version: canary
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**Step 4: Apply the Service and Deployments**

```bash
kubectl apply -f canary-service.yaml
kubectl apply -f stable-deployment.yaml
kubectl apply -f canary-deployment.yaml
```

**Step 5: Verify the Deployments and Service**

```bash
kubectl get deployments
kubectl get pods -l app=canary-app --show-labels
kubectl get service canary-service
```

**Step 6: Gradually increase the canary Deployment**

```bash
kubectl scale deployment canary-app --replicas=2
```

**Step 7: Complete the canary deployment by scaling down the stable version**

```bash
kubectl scale deployment stable-app --replicas=0
```

**What this does**:

- Creates a Service that selects Pods with the label `app=canary-app`
- Creates two Deployments:
  - `stable-app`: 4 replicas of nginx:1.20 with labels `app=canary-app` and `version=stable`
  - `canary-app`: 1 replica of nginx:1.21 with labels `app=canary-app` and `version=canary`
- The Service routes traffic to both Deployments, with 80% going to the stable version and 20% to the canary version
- Gradually increases the canary version and decreases the stable version
- This allows you to test the new version with a small percentage of traffic before full rollout

</p>
</details>

### Exercise 5: Blue/Green Deployment

**Goal**: Learn how to implement a blue/green deployment pattern.

**Task**: Deploy a new version of an application alongside the existing version, then switch traffic all at once.

**Why this matters**: Blue/Green deployments provide a way to deploy new versions with zero downtime and immediate rollback capability. This strategy is valuable for critical applications where you need to thoroughly test the new version in production before directing traffic to it, and where you need the ability to instantly revert if issues arise.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Service to route traffic to the blue Deployment**

Create a file named `blue-green-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: my-app
    version: blue
  ports:
  - port: 80
    targetPort: 80
```

**Step 2: Create the blue Deployment**

Create a file named `blue-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-app
  labels:
    app: my-app
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

**Step 3: Apply the Service and blue Deployment**

```bash
kubectl apply -f blue-green-service.yaml
kubectl apply -f blue-deployment.yaml
```

**Step 4: Verify the Deployment and Service**

```bash
kubectl get deployments
kubectl get pods -l app=my-app --show-labels
kubectl get service app-service
```

**Step 5: Create the green Deployment**

Create a file named `green-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-app
  labels:
    app: my-app
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**Step 6: Apply the green Deployment**

```bash
kubectl apply -f green-deployment.yaml
```

**Step 7: Verify that both Deployments are running**

```bash
kubectl get deployments
kubectl get pods -l app=my-app --show-labels
```

**Step 8: Switch traffic from blue to green by updating the Service**

```bash
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'
```

**Step 9: Verify the Service is now pointing to the green Deployment**

```bash
kubectl describe service app-service
```

**Step 10: Once the green Deployment is confirmed working, delete the blue Deployment**

```bash
kubectl delete deployment blue-app
```

**What this does**:

- Creates a Service that initially routes traffic to the blue Deployment
- Deploys the blue version (nginx:1.20) and verifies it's working
- Deploys the green version (nginx:1.21) alongside the blue version
- The green version can be tested independently before switching traffic
- Updates the Service to route all traffic to the green version at once
- Deletes the blue version after confirming the green version is working
- This provides a clean cutover with zero downtime and easy rollback (by switching the Service back to blue)

</p>
</details>
