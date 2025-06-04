# Rolling Updates and Rollbacks

This section covers Rolling Updates and Rollbacks in Kubernetes, which are essential for safely updating applications without downtime.

## Key Resources

- [Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Update Strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
- [Rollback Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)

## Introduction to Rolling Updates and Rollbacks

Rolling updates allow you to update your application with zero downtime by incrementally replacing old Pods with new ones. If something goes wrong during the update, Kubernetes allows you to roll back to a previous version.

## Rolling Updates

### Exercise 1: Performing a Rolling Update

**Goal**: Learn how to perform a rolling update of a Deployment.

**Task**: Create a Deployment and then update it using a rolling update strategy.

**Why this matters**: Rolling updates are the default update strategy in Kubernetes for a reason - they allow you to update your application without downtime, which is crucial for production environments.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with version 1 of an application**

Create a file named `deployment-v1.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
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
        version: v1
    spec:
      containers:
      - name: web-app
        image: nginx:1.19
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

Apply the configuration:

```bash
kubectl apply -f deployment-v1.yaml
```

**Step 2: Verify that the Deployment is running**

```bash
kubectl get deployments
kubectl get pods -l app=web-app
```

**Step 3: Update the Deployment to version 2**

You can update the Deployment in several ways:

Option 1: Using `kubectl set image`:

```bash
kubectl set image deployment/web-app web-app=nginx:1.20 --record
```

Option 2: Editing the Deployment:

```bash
kubectl edit deployment web-app
```

Change the image from `nginx:1.19` to `nginx:1.20`.

Option 3: Applying an updated YAML file:

Create a file named `deployment-v2.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
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
        version: v2
    spec:
      containers:
      - name: web-app
        image: nginx:1.20
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

Apply the configuration:

```bash
kubectl apply -f deployment-v2.yaml
```

**Step 4: Watch the rolling update in progress**

```bash
kubectl rollout status deployment/web-app
```

You can also watch the Pods being replaced:

```bash
kubectl get pods -l app=web-app -w
```

**What this does**:

- Creates a Deployment with 3 replicas running nginx:1.19
- Updates the Deployment to use nginx:1.20
- Kubernetes performs a rolling update, replacing the old Pods one by one
- The `maxSurge: 1` setting allows at most 1 extra Pod to be created during the update
- The `maxUnavailable: 1` setting allows at most 1 Pod to be unavailable during the update
- The readiness probe ensures that new Pods are only considered ready when they can serve traffic

This demonstrates how to perform a rolling update, which is the default update strategy for Deployments.

</p>
</details>

### Exercise 2: Customizing the Rolling Update Strategy

**Goal**: Learn how to customize the rolling update strategy.

**Task**: Create a Deployment with a custom rolling update strategy.

**Why this matters**: Different applications have different requirements for updates. Customizing the rolling update strategy allows you to control how many Pods are replaced at once and how many extra Pods can be created during the update.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with a custom rolling update strategy**

Create a file named `custom-rolling-update.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-update
spec:
  replicas: 5
  selector:
    matchLabels:
      app: custom-update
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: custom-update
    spec:
      containers:
      - name: web-app
        image: nginx:1.19
        ports:
        - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f custom-rolling-update.yaml
```

**Step 2: Update the Deployment**

```bash
kubectl set image deployment/custom-update web-app=nginx:1.20 --record
```

**Step 3: Watch the rolling update in progress**

```bash
kubectl rollout status deployment/custom-update
```

You can also watch the Pods being replaced:

```bash
kubectl get pods -l app=custom-update -w
```

**What this does**:

- Creates a Deployment with 5 replicas
- Sets `maxSurge: 2`, which allows at most 2 extra Pods to be created during the update
- Sets `maxUnavailable: 0`, which ensures that all existing Pods remain available during the update
- Updates the Deployment, and Kubernetes performs a rolling update according to the specified strategy

This demonstrates how to customize the rolling update strategy to meet specific requirements. In this case, the strategy ensures that there is no reduction in capacity during the update (zero downtime).

</p>
</details>

### Exercise 3: Pausing and Resuming a Rollout

**Goal**: Learn how to pause and resume a rollout.

**Task**: Start a rolling update, pause it, and then resume it.

**Why this matters**: Sometimes you may want to pause a rollout to verify that the new version is working correctly before continuing. This allows you to perform canary testing or other validation before fully committing to the update.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment**

Create a file named `pausable-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pausable-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: pausable-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: pausable-app
    spec:
      containers:
      - name: web-app
        image: nginx:1.19
        ports:
        - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f pausable-deployment.yaml
```

**Step 2: Start a rolling update**

```bash
kubectl set image deployment/pausable-app web-app=nginx:1.20 --record
```

**Step 3: Pause the rollout**

```bash
kubectl rollout pause deployment/pausable-app
```

**Step 4: Check the status of the rollout**

```bash
kubectl rollout status deployment/pausable-app
```

You should see that the rollout is paused.

You can also check the Pods:

```bash
kubectl get pods -l app=pausable-app
```

You should see a mix of old and new Pods.

**Step 5: Resume the rollout**

```bash
kubectl rollout resume deployment/pausable-app
```

**Step 6: Verify that the rollout completes**

```bash
kubectl rollout status deployment/pausable-app
```

**What this does**:

- Creates a Deployment with 6 replicas
- Starts a rolling update to a new version
- Pauses the rollout in the middle, leaving some Pods with the old version and some with the new version
- Resumes the rollout, allowing it to complete

This demonstrates how to pause and resume a rollout, which can be useful for canary testing or other validation before fully committing to an update.

</p>
</details>

## Rollbacks

### Exercise 1: Rolling Back a Deployment

**Goal**: Learn how to roll back a Deployment to a previous version.

**Task**: Update a Deployment and then roll it back to the previous version.

**Why this matters**: Sometimes updates introduce bugs or other issues that weren't caught in testing. Being able to quickly roll back to a known-good version is crucial for maintaining application availability.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with version 1**

Create a file named `rollback-deployment-v1.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rollback-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rollback-app
  template:
    metadata:
      labels:
        app: rollback-app
        version: v1
    spec:
      containers:
      - name: web-app
        image: nginx:1.19
        ports:
        - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f rollback-deployment-v1.yaml --record
```

**Step 2: Update the Deployment to version 2**

```bash
kubectl set image deployment/rollback-app web-app=nginx:1.20 --record
```

**Step 3: Verify that the update completed**

```bash
kubectl rollout status deployment/rollback-app
```

**Step 4: View the rollout history**

```bash
kubectl rollout history deployment/rollback-app
```

You should see two revisions in the history.

**Step 5: Roll back to the previous version**

```bash
kubectl rollout undo deployment/rollback-app
```

**Step 6: Verify that the rollback completed**

```bash
kubectl rollout status deployment/rollback-app
```

**Step 7: Check the current version**

```bash
kubectl get deployment rollback-app -o jsonpath='{.spec.template.spec.containers[0].image}'
```

You should see that the image is back to nginx:1.19.

**What this does**:

- Creates a Deployment with version 1
- Updates the Deployment to version 2
- Rolls back the Deployment to version 1
- Kubernetes performs a rolling update to revert to the previous version

This demonstrates how to roll back a Deployment to a previous version, which is useful when an update introduces issues.

</p>
</details>

### Exercise 2: Rolling Back to a Specific Revision

**Goal**: Learn how to roll back a Deployment to a specific revision.

**Task**: Create multiple revisions of a Deployment and then roll back to a specific revision.

**Why this matters**: Sometimes you need to roll back to a specific version that is not the immediately previous one. Kubernetes allows you to roll back to any revision in the history.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with version 1**

Create a file named `multi-revision-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-revision-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multi-revision-app
  template:
    metadata:
      labels:
        app: multi-revision-app
        version: v1
    spec:
      containers:
      - name: web-app
        image: nginx:1.18
        ports:
        - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f multi-revision-deployment.yaml --record
```

**Step 2: Update the Deployment to version 2**

```bash
kubectl set image deployment/multi-revision-app web-app=nginx:1.19 --record
```

**Step 3: Update the Deployment to version 3**

```bash
kubectl set image deployment/multi-revision-app web-app=nginx:1.20 --record
```

**Step 4: View the rollout history**

```bash
kubectl rollout history deployment/multi-revision-app
```

You should see three revisions in the history.

**Step 5: View the details of a specific revision**

```bash
kubectl rollout history deployment/multi-revision-app --revision=2
```

This will show the details of revision 2.

**Step 6: Roll back to revision 1**

```bash
kubectl rollout undo deployment/multi-revision-app --to-revision=1
```

**Step 7: Verify that the rollback completed**

```bash
kubectl rollout status deployment/multi-revision-app
```

**Step 8: Check the current version**

```bash
kubectl get deployment multi-revision-app -o jsonpath='{.spec.template.spec.containers[0].image}'
```

You should see that the image is back to nginx:1.18.

**What this does**:

- Creates a Deployment with version 1 (nginx:1.18)
- Updates the Deployment to version 2 (nginx:1.19)
- Updates the Deployment to version 3 (nginx:1.20)
- Views the rollout history to see all revisions
- Rolls back to revision 1 (nginx:1.18)
- Kubernetes performs a rolling update to revert to the specified revision

This demonstrates how to roll back a Deployment to a specific revision, which is useful when you need to go back to a version that is not the immediately previous one.

</p>
</details>

### Exercise 3: Controlling Revision History

**Goal**: Learn how to control the number of revisions kept in the history.

**Task**: Configure a Deployment to keep a specific number of revisions in its history.

**Why this matters**: By default, Kubernetes keeps 10 revisions in the history. You may want to increase this number to have more rollback options or decrease it to save resources.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment with a custom revision history limit**

Create a file named `revision-history-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: history-app
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: history-app
  template:
    metadata:
      labels:
        app: history-app
    spec:
      containers:
      - name: web-app
        image: nginx:1.18
        ports:
        - containerPort: 80
```

Apply the configuration:

```bash
kubectl apply -f revision-history-deployment.yaml --record
```

**Step 2: Make multiple updates to the Deployment**

```bash
kubectl set image deployment/history-app web-app=nginx:1.19 --record
kubectl set image deployment/history-app web-app=nginx:1.20 --record
kubectl set image deployment/history-app web-app=nginx:1.21 --record
kubectl set image deployment/history-app web-app=nginx:1.22 --record
kubectl set image deployment/history-app web-app=nginx:1.23 --record
kubectl set image deployment/history-app web-app=nginx:1.24 --record
```

**Step 3: View the rollout history**

```bash
kubectl rollout history deployment/history-app
```

You should see at most 5 revisions in the history, even though you made 6 updates.

**What this does**:

- Creates a Deployment with `revisionHistoryLimit: 5`
- Makes multiple updates to the Deployment
- Kubernetes keeps only the 5 most recent revisions in the history
- Older revisions are automatically deleted

This demonstrates how to control the number of revisions kept in the history, which can be useful for managing resources or ensuring that you have enough rollback options.

</p>
</details>

## Best Practices

1. **Always use the `--record` flag** when making changes to Deployments to record the command in the revision history.
2. **Set appropriate values for `maxSurge` and `maxUnavailable`** based on your application's requirements.
3. **Include readiness probes** in your Pods to ensure that new versions are only considered ready when they can serve traffic.
4. **Consider using a blue-green or canary deployment strategy** for critical applications.
5. **Test rollbacks regularly** to ensure that your application can be rolled back if needed.
6. **Set an appropriate `revisionHistoryLimit`** based on how many rollback options you need.
7. **Use labels to track versions** in your Pod templates to make it easier to identify which version is running.
8. **Monitor the health of your application during updates** to catch issues early.
