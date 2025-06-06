# Jobs and CronJobs

This section covers Jobs and CronJobs in Kubernetes, which are essential for batch processing and scheduled tasks.

## Key Resources

- [Jobs Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [CronJobs Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

## Introduction to Jobs and CronJobs

Jobs and CronJobs are Kubernetes resources designed for batch processing and scheduled tasks:

- **Jobs** create one or more Pods and ensure they run to completion
- **CronJobs** create Jobs on a time-based schedule

## Imperative vs. Declarative Approaches for Jobs and CronJobs

Both Jobs and CronJobs can be created using imperative commands or declarative manifests, each with their own advantages:

### Imperative Commands

**For Jobs:**
```bash
kubectl create job <job-name> --image=<image-name> [-- <command>] [<args>]
```

**For CronJobs:**
```bash
kubectl create cronjob <cronjob-name> --image=<image-name> --schedule="<cron-schedule>" [-- <command>] [<args>]
```

**Advantages:**
- Quick to create for simple use cases
- Excellent for the CKAD exam to save time
- No need to write YAML for basic configurations

**Limitations:**
- Cannot set advanced parameters like `completions`, `parallelism`, `backoffLimit` directly
- Limited control over job behavior and retry policies
- No support for `concurrencyPolicy` in CronJobs via imperative commands

### Declarative Manifests

**Advantages:**
- Full control over all Job and CronJob parameters
- Required for advanced configurations
- Better for version control and GitOps workflows

**When to use each approach:**
- **Use imperative commands** for simple Jobs/CronJobs during the CKAD exam or quick testing
- **Use declarative YAML** for production workloads or when you need advanced parameters
- **Use hybrid approach** (generate YAML with `--dry-run=client -o yaml`) when you need to start with a template and customize it

## Jobs

### Exercise 1: Creating a Simple Job

**Goal**: Learn how to create a basic Job that runs a task to completion.

**Task**: Create a Job that performs a simple computation.

**Why this matters**: Jobs are essential for batch processing, data migrations, and other tasks that need to run to completion rather than continuously.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Job**

Option 1: Using imperative commands (recommended for CKAD exam):

```bash
# Basic job creation - calculates Pi to 2000 decimal places
kubectl create job pi-calculation --image=perl:5.34 -- perl -Mbignum=bpi -wle "print bpi(2000)"

# Alternative with environment variables
kubectl create job env-job --image=busybox -- sh -c "echo Job executed with env var: $RANDOM_VALUE" --env="RANDOM_VALUE=42"

# Generate YAML template without creating the job (hybrid approach)
kubectl create job pi-calculation --image=perl:5.34 --dry-run=client -o yaml > pi-job.yaml
# Then edit the YAML file to add advanced parameters and apply
```

> **CKAD Exam Tip:** The imperative `kubectl create job` command is much faster than writing YAML from scratch. For simple jobs, this should be your go-to approach during the exam.

Option 2: Using a manifest file (for more control):

Create a file named `pi-job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculation
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f pi-job.yaml
```

**Step 2: Monitor the Job status**

```bash
# Check the job status
kubectl get jobs

# Watch the pods created by the job
kubectl get pods -l job-name=pi-calculation
```

**Step 3: View the Job output**

```bash
# Get the logs from the completed job
kubectl logs -l job-name=pi-calculation
```

**What this does**:

- Creates a Job named `pi-calculation`
- The Job creates a Pod that calculates Pi to 2000 decimal places
- Once the Pod completes successfully, the Job is marked as completed

</p>
</details>

### Exercise 2: Job with Multiple Completions

**Goal**: Learn how to create a Job that runs multiple times.

**Task**: Create a Job that runs a task multiple times in sequence.

**Why this matters**: Some batch workloads require running the same task multiple times, such as processing items from a queue.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Job with multiple completions**

> **Important for CKAD Exam:** Setting `completions` and `parallelism` requires a manifest file as the imperative command doesn't support these parameters directly. This is a key limitation to be aware of during the exam. Use the hybrid approach (generate YAML with `--dry-run=client -o yaml`) when you need these advanced features.

Create a file named `multiple-completions-job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-job
spec:
  completions: 5
  parallelism: 2
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo Processing item $RANDOM; sleep 5"]
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f multiple-completions-job.yaml
```

**Step 2: Monitor the Job progress**

```bash
# Watch the job status
kubectl get jobs multi-completion-job -w

# See the pods being created and completed
kubectl get pods -l job-name=multi-completion-job
```

**Step 3: View the output from each completion**

```bash
# Get logs from all pods created by the job
kubectl logs -l job-name=multi-completion-job --tail=20
```

**What this does**:

- Creates a Job that needs to complete 5 times (`completions: 5`)
- Runs up to 2 Pods in parallel (`parallelism: 2`)
- Each Pod runs a simple task and then completes
- The Job is considered complete only when 5 successful completions have occurred

</p>
</details>

### Exercise 3: Job with Failure Handling

**Goal**: Learn how to handle failures in Jobs.

**Task**: Create a Job that demonstrates failure handling and retries.

**Why this matters**: In real-world scenarios, tasks can fail for various reasons. Understanding how Jobs handle failures is crucial for building reliable batch processes.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Job that sometimes fails**

Create a file named `job-with-failures.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sometimes-fails
spec:
  backoffLimit: 4
  template:
    spec:
      containers:
      - name: random-fail
        image: busybox
        command: ["sh", "-c", "if [ $((RANDOM % 3)) -eq 0 ]; then echo 'Task succeeded'; else echo 'Task failed' && exit 1; fi"]
      restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f job-with-failures.yaml
```

**Step 2: Monitor the Job and observe retries**

```bash
# Watch the job status
kubectl get jobs sometimes-fails -w

# See the pods being created and potentially failing
kubectl get pods -l job-name=sometimes-fails
```

**Step 3: View the logs from the pods**

```bash
# Get logs from all pods created by the job
kubectl logs -l job-name=sometimes-fails
```

**What this does**:

- Creates a Job that has a 2/3 chance of failing
- Sets `backoffLimit: 4` to allow up to 4 retries
- Uses `restartPolicy: Never` to create a new Pod for each retry
- If the Job succeeds within the retry limit, it's marked as completed
- If all retries fail, the Job is marked as failed

> Note: In the CKAD exam, understanding how to configure `backoffLimit` and `restartPolicy` is important for controlling Job behavior.

</p>
</details>

## CronJobs

### Exercise 1: Creating a Simple CronJob

**Goal**: Learn how to create a CronJob that runs on a schedule.

**Task**: Create a CronJob that runs a simple task every minute.

**Why this matters**: CronJobs are essential for scheduled tasks such as backups, reports, and periodic cleanup operations.

<details><summary>show solution</summary>
<p>

**Step 1: Create a CronJob**

Option 1: Using imperative commands (recommended for CKAD exam):

```bash
# Basic cronjob - logs the date every minute
kubectl create cronjob minute-logger --image=busybox --schedule="*/1 * * * *" -- sh -c "date; echo Hello from Kubernetes cronjob"

# CronJob that runs every 5 minutes
kubectl create cronjob five-minute-job --image=busybox --schedule="*/5 * * * *" -- sh -c "echo Job running at $(date)"

# CronJob that runs at specific time (8:30 AM daily)
kubectl create cronjob daily-report --image=busybox --schedule="30 8 * * *" -- sh -c "echo Daily report generated at $(date)"

# Generate YAML template without creating the cronjob (hybrid approach)
kubectl create cronjob minute-logger --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml > cronjob.yaml
# Then edit the YAML file to add advanced parameters and apply
```

> **CKAD Exam Tips:** 
> - The `--schedule` parameter uses standard cron syntax (minute hour day-of-month month day-of-week)
> - Use [crontab.guru](https://crontab.guru/) to verify your cron expressions
> - Remember that Kubernetes uses UTC time for CronJob schedules
> - For simple CronJobs, imperative commands save significant time in the exam

Option 2: Using a manifest file:

Create a file named `minute-cronjob.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: minute-logger
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: logger
            image: busybox
            command: ["sh", "-c", "date; echo 'Hello from CronJob'"]
          restartPolicy: OnFailure
```

Apply the configuration:

```bash
kubectl apply -f minute-cronjob.yaml
```

**Step 2: Monitor the CronJob**

```bash
# Check the cronjob status
kubectl get cronjobs

# After a minute, check for jobs created by the cronjob
kubectl get jobs
```

**Step 3: View the output from a job**

```bash
# Get the logs from the most recent job
kubectl logs $(kubectl get pods -l job-name=$(kubectl get jobs -l cronjob-name=minute-logger -o jsonpath='{.items[0].metadata.name}') -o jsonpath='{.items[0].metadata.name}')
```

**What this does**:

- Creates a CronJob named `minute-logger` that runs every minute
- Each time the schedule triggers, a new Job is created
- The Job creates a Pod that runs the specified command
- The CronJob continues to create Jobs according to the schedule

</p>
</details>

### Exercise 2: CronJob with Concurrency Policy

**Goal**: Learn how to control concurrent execution of CronJobs.

**Task**: Create a CronJob with a specific concurrency policy.

**Why this matters**: Controlling how CronJobs handle concurrent executions is important for preventing resource contention and ensuring proper execution order.

<details><summary>show solution</summary>
<p>

**Step 1: Create a CronJob with concurrency policy**

> Note: Setting concurrency policy requires a manifest file as the imperative command doesn't support this option.

Create a file named `concurrency-cronjob.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: long-running-task
spec:
  schedule: "*/2 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: task
            image: busybox
            command: ["sh", "-c", "echo Starting task at $(date); sleep 180; echo Completed task at $(date)"]
          restartPolicy: OnFailure
```

Apply the configuration:

```bash
kubectl apply -f concurrency-cronjob.yaml
```

**Step 2: Monitor the CronJob behavior**

```bash
# Check the cronjob status
kubectl get cronjobs long-running-task

# After a few minutes, check for jobs created by the cronjob
kubectl get jobs -l cronjob-name=long-running-task
```

**Step 3: View the job details**

```bash
# Get details of the jobs
kubectl describe jobs -l cronjob-name=long-running-task
```

**What this does**:

- Creates a CronJob that runs every 2 minutes
- Each job takes 3 minutes to complete (longer than the schedule interval)
- Sets `concurrencyPolicy: Forbid` to prevent new jobs from starting if the previous job is still running
- This ensures that jobs don't overlap, which is important for tasks that can't run concurrently

> Note: There are three concurrency policies:
> - `Allow` (default): Allows concurrent jobs to run
> - `Forbid`: Prevents new jobs from starting if the previous job is still running
> - `Replace`: Cancels the currently running job and starts a new one

</p>
</details>

### Exercise 3: CronJob with History Limits

**Goal**: Learn how to control the history of completed and failed Jobs.

**Task**: Create a CronJob with specific history limits.

**Why this matters**: Managing the history of completed and failed Jobs is important for troubleshooting and preventing resource leaks.

<details><summary>show solution</summary>
<p>

**Step 1: Create a CronJob with history limits**

Create a file named `history-limits-cronjob.yaml`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-demo
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: task
            image: busybox
            command: ["sh", "-c", "if [ $((RANDOM % 3)) -eq 0 ]; then exit 1; else echo Success; fi"]
          restartPolicy: Never
```

Apply the configuration:

```bash
kubectl apply -f history-limits-cronjob.yaml
```

**Step 2: Monitor the CronJob history**

```bash
# Wait for several minutes to allow multiple jobs to run
# Then check the jobs
kubectl get jobs -l cronjob-name=history-demo
```

**What this does**:

- Creates a CronJob that runs every minute
- The job has a 1/3 chance of failing
- Sets `successfulJobsHistoryLimit: 3` to keep only the 3 most recent successful jobs
- Sets `failedJobsHistoryLimit: 2` to keep only the 2 most recent failed jobs
- Older completed or failed jobs are automatically deleted

> Note: For the CKAD exam, understanding how to manage job history is important for resource management.

</p>
</details>

## Best Practices

1. **Use appropriate `restartPolicy`**:
   - For Jobs, use `Never` or `OnFailure`
   - Never use `Always` for Jobs or CronJobs

2. **Set reasonable `backoffLimit`** to prevent infinite retries for failing Jobs.

3. **Use `activeDeadlineSeconds`** to set a time limit for job execution.

4. **Choose the right `concurrencyPolicy`** for CronJobs based on whether tasks can run concurrently.

5. **Set appropriate history limits** to prevent accumulation of completed and failed Jobs.

6. **Use labels** to organize and identify Jobs created by CronJobs.

7. **Consider time zones** when setting CronJob schedules, as Kubernetes uses UTC.

8. **Test CronJob schedules** using tools like [crontab.guru](https://crontab.guru/) to ensure they run at the expected times.

## CKAD Exam Tips for Jobs and CronJobs

### Imperative Command Quick Reference

**Jobs:**
```bash
# Basic job
kubectl create job <name> --image=<image> -- <command>

# With restart policy
kubectl create job <name> --image=<image> --restart=OnFailure -- <command>
```

**CronJobs:**
```bash
# Basic cronjob
kubectl create cronjob <name> --image=<image> --schedule="<cron-expression>" -- <command>
```

### When to Use Each Approach

**Use Imperative Commands When:**
- Creating simple Jobs or CronJobs with basic configurations
- You don't need advanced parameters like `completions`, `parallelism`, or `concurrencyPolicy`
- Time is limited and you need to create resources quickly

**Use Declarative YAML When:**
- You need advanced Job parameters (`completions`, `parallelism`, `backoffLimit`, etc.)
- You need advanced CronJob parameters (`concurrencyPolicy`, `successfulJobsHistoryLimit`, etc.)
- You're creating complex configurations that need to be version controlled

**Use the Hybrid Approach When:**
- You need a starting point for a complex configuration
- You want to avoid syntax errors in your YAML
- You need to make minor modifications to a standard configuration

```bash
# Generate a Job YAML template
kubectl create job <name> --image=<image> --dry-run=client -o yaml > job.yaml

# Generate a CronJob YAML template
kubectl create cronjob <name> --image=<image> --schedule="<cron-expression>" --dry-run=client -o yaml > cronjob.yaml
```

Remember that for the CKAD exam, using the right approach for each scenario will save you valuable time.
