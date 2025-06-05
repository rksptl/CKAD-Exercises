# Workload Resources

This section covers Kubernetes workload resources such as Deployments, DaemonSets, and CronJobs, which are essential for running applications in Kubernetes.

## Key Resources

- [Workloads](https://kubernetes.io/docs/concepts/workloads/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

## Introduction to Kubernetes Workload Resources

Kubernetes provides several resource types for deploying and managing applications. These workload resources abstract the details of how containers are run and provide features like scaling, rolling updates, and scheduled execution.

## Key Concepts

- **Pods**: The basic building block for running containers
- **Deployments**: Manage ReplicaSets and provide declarative updates for Pods
- **ReplicaSets**: Ensure a specified number of Pod replicas are running at any given time
- **DaemonSets**: Ensure all (or some) nodes run a copy of a Pod
- **StatefulSets**: Manage stateful applications with stable network identities
- **Jobs**: Run Pods to completion
- **CronJobs**: Create Jobs on a schedule

## Workload Resources

### Exercise 1: Creating a Deployment

**Goal**: Learn how to create a Deployment to manage a set of replicated Pods.

**Task**: Create a Deployment with multiple replicas of an nginx container.

**Why this matters**: Deployments are the most common way to deploy applications in Kubernetes. They provide declarative updates, scaling, and self-healing capabilities that are essential for running reliable applications in production. Understanding Deployments is fundamental to managing applications in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Deployment manifest file**

Create a file named `nginx-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

**Step 2: Create the Deployment**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f nginx-deployment.yaml
```

Option 2: Using imperative commands:
```bash
kubectl create deployment nginx-deployment --image=nginx:1.21 --replicas=3 --port=80
kubectl set resources deployment nginx-deployment --requests=cpu=100m,memory=64Mi --limits=cpu=200m,memory=128Mi
```

**Step 3: Verify the Deployment**

```bash
kubectl get deployments
kubectl get pods
```

**Step 4: View detailed information about the Deployment**

```bash
kubectl describe deployment nginx-deployment
```

**What this does**:

- Creates a Deployment named `nginx-deployment` with 3 replicas
- Uses the nginx:1.21 image for the Pods
- Sets resource requests and limits for the containers
- The Deployment controller creates a ReplicaSet, which then creates the Pods
- The Deployment ensures that the desired number of Pods are always running

</p>
</details>

### Exercise 2: Scaling a Deployment

**Goal**: Learn how to scale a Deployment up or down.

**Task**: Scale the Deployment created in Exercise 1 to a different number of replicas.

**Why this matters**: Scaling is a fundamental operation in cloud-native applications. The ability to quickly adjust the number of running instances based on demand is essential for efficient resource utilization and handling traffic spikes. Kubernetes makes scaling operations simple and reliable through Deployments.

<details><summary>show solution</summary>
<p>

**Method 1: Scale using kubectl scale command (imperative)**

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

**Method 2: Scale by editing the Deployment manifest**

```bash
kubectl edit deployment nginx-deployment
```

Change the `replicas` field from 3 to 5, then save and exit.

**Method 3: Scale using a patch command**

```bash
kubectl patch deployment nginx-deployment -p '{"spec":{"replicas":5}}'
```

**Verify the scaling operation**

```bash
kubectl get deployment nginx-deployment
kubectl get pods -l app=nginx
```

**What this does**:

- Increases the number of replicas from 3 to 5
- The Deployment controller creates additional Pods to match the desired state
- All three methods achieve the same result, but offer different approaches:
  - `kubectl scale` is simple and direct
  - `kubectl edit` allows you to modify other aspects of the Deployment as well
  - `kubectl patch` is useful for scripting and automation

</p>
</details>

### Exercise 3: Creating a DaemonSet

**Goal**: Learn how to create a DaemonSet to run a Pod on every node in the cluster.

**Task**: Create a DaemonSet for a logging agent that should run on all nodes.

**Why this matters**: DaemonSets are essential for node-level operations like monitoring, logging, and networking. They ensure that specific Pods run on every node (or a subset of nodes), making them perfect for infrastructure components that need to be present throughout the cluster. This pattern is widely used for logging agents, monitoring collectors, and network plugins.

<details><summary>show solution</summary>
<p>

**Step 1: Create a DaemonSet manifest file**

Create a file named `logging-daemonset.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logging-agent
  labels:
    app: logging-agent
spec:
  selector:
    matchLabels:
      name: logging-agent
  template:
    metadata:
      labels:
        name: logging-agent
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

**Step 2: Create the DaemonSet**

```bash
kubectl apply -f logging-daemonset.yaml
```

> Note: DaemonSets must be created using a manifest file as there is no direct imperative command to create them.

**Step 3: Verify the DaemonSet**

```bash
kubectl get daemonsets
kubectl get pods -l name=logging-agent -o wide
```

**What this does**:

- Creates a DaemonSet named `logging-agent`
- The DaemonSet controller creates a Pod on each node in the cluster
- The Pod mounts the host's `/var/log` directory, allowing the logging agent to access the node's logs
- If nodes are added to the cluster, the DaemonSet controller automatically creates Pods on the new nodes
- If nodes are removed, the corresponding Pods are garbage collected

</p>
</details>

### Exercise 4: Creating a CronJob

**Goal**: Learn how to create a CronJob to run tasks on a schedule.

**Task**: Create a CronJob that runs a backup operation every hour.

**Why this matters**: CronJobs are essential for scheduled tasks like backups, report generation, and data processing. They provide a Kubernetes-native way to run periodic jobs without requiring external schedulers. Understanding CronJobs is important for automating routine operations in your applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a CronJob manifest file**

Create a file named `backup-cronjob.yaml` with the following content:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 * * * *"  # Every hour
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: bitnami/postgresql:14
            command:
            - /bin/sh
            - -c
            - echo "Backing up the database at $(date)"
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
          restartPolicy: OnFailure
```

**Step 2: Create a Secret for the database credentials**

```bash
kubectl create secret generic db-credentials --from-literal=password=mysecretpassword
```

**Step 3: Create the CronJob**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f backup-cronjob.yaml
```

Option 2: Using imperative commands:
```bash
# Create a basic CronJob with imperative command
kubectl create cronjob backup-database --image=mysql:8.0 --schedule="0 * * * *" -- bash -c "echo 'Starting backup'; sleep 5; echo 'Backup completed'"

# Note: For complex CronJobs with environment variables from secrets, volume mounts, or other advanced features,
# the declarative approach with a manifest file is recommended
```

Option 2: Using imperative commands:
```bash
kubectl create cronjob database-backup --image=backup-tool:1.2 --schedule="0 * * * *" -- /backup.sh
```

**Step 4: Verify the CronJob**

```bash
kubectl get cronjobs
```

**Step 5: Check the jobs and pods created by the CronJob**

```bash
kubectl get jobs
kubectl get pods
```

**What this does**:

- Creates a CronJob named `database-backup` that runs every hour (at the top of the hour)
- Sets `concurrencyPolicy: Forbid` to prevent concurrent executions of the job
- Limits the history of successful and failed jobs
- The job runs a container that simulates a database backup
- The container uses a secret for the database password
- Sets `restartPolicy: OnFailure` to restart the container if it fails

</p>
</details>

### Exercise 5: Creating a Job

**Goal**: Learn how to create a Job to run a task to completion.

**Task**: Create a Job that performs a computation and completes.

**Why this matters**: Jobs are essential for batch processing, data migrations, and other tasks that need to run to completion rather than continuously. They ensure that Pods are created and run until the task is complete, with retry capabilities for reliability. This pattern is commonly used for ETL processes, database migrations, and computational tasks.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Job manifest file**

Create a file named `computation-job.yaml` with the following content:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculation
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 4
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

**Step 2: Create the Job**

Option 1: Using a manifest file (declarative approach):
```bash
kubectl apply -f computation-job.yaml
```

Option 2: Using imperative commands:
```bash
kubectl create job pi-calculation --image=perl:5.34 -- perl -Mbignum=bpi -wle "print bpi(2000)"
```

**Step 3: Monitor the Job status**

```bash
kubectl get jobs
```

**Step 4: Check the output of the Job**

```bash
kubectl get pods -l job-name=pi-calculation
kubectl logs -l job-name=pi-calculation
```

**What this does**:

- Creates a Job named `pi-calculation`
- The Job creates a Pod that calculates Pi to 2000 decimal places
- `completions: 1` specifies that the Job should be considered complete when one Pod completes successfully
- `parallelism: 1` specifies that only one Pod should run at a time
- `backoffLimit: 4` specifies that the Job should be retried up to 4 times if it fails
- `restartPolicy: Never` specifies that the Pod should not be restarted if it completes or fails
- Once the Pod completes successfully, the Job is marked as completed

</p>
</details>

### Exercise 6: Creating a StatefulSet

**Goal**: Learn how to create a StatefulSet for stateful applications.

**Task**: Create a StatefulSet for a database application that requires stable network identities and persistent storage.

**Why this matters**: StatefulSets are crucial for deploying stateful applications like databases, which require stable network identities, ordered deployment and scaling, and persistent storage. Understanding StatefulSets is essential for running stateful workloads reliably in Kubernetes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a headless Service for the StatefulSet**

Create a file named `database-service.yaml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
  labels:
    app: database
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None
  selector:
    app: database
```

**Step 2: Create a StorageClass for dynamic provisioning**

```bash
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
EOF
```

**Step 3: Create a StatefulSet manifest file**

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
      terminationGracePeriodSeconds: 10
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-credentials
              key: root-password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

**Step 4: Create a Secret for MySQL credentials**

```bash
kubectl create secret generic mysql-credentials --from-literal=root-password=mysecretpassword
```

> Note: This is already using the imperative command syntax.

**Step 5: Create the Service and StatefulSet**

For the Service:
```bash
kubectl create service clusterip database --clusterip="None" --tcp=3306:3306 --selector=app=database
```

For the StatefulSet (must use manifest file):
```bash
kubectl apply -f database-statefulset.yaml
```

> Note: StatefulSets are complex resources that are best created using manifest files.

**Step 6: Verify the StatefulSet**

```bash
kubectl get statefulsets
kubectl get pods -l app=database
kubectl get pvc
```

**What this does**:

- Creates a headless Service named `database` that provides network identity for the StatefulSet Pods
- Creates a StatefulSet named `mysql` with 3 replicas
- Each Pod gets a stable hostname in the form of `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- The StatefulSet creates PersistentVolumeClaims for each Pod using the volumeClaimTemplates
- Pods are created and deleted in order (from 0 to n-1 for creation, n-1 to 0 for deletion)
- Each Pod has a stable identity and persistent storage, which is essential for stateful applications

</p>
</details>
