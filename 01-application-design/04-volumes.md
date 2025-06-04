# Persistent and Ephemeral Volumes

This section covers how to use volumes in Kubernetes to manage data persistence, which is an important part of the CKAD exam.

## Key Resources

- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Introduction to Kubernetes Storage

Kubernetes provides several ways to manage data persistence in containers. Since containers are ephemeral by nature (data is lost when a container restarts), Kubernetes offers volume types to handle various data persistence needs.

## Key Concepts

- **Volumes**: Storage abstraction that allows data to persist beyond the lifecycle of a container
- **Ephemeral Volumes**: Temporary storage that exists only for the lifetime of a Pod
- **PersistentVolumes (PV)**: Cluster-wide storage resources provisioned by administrators
- **PersistentVolumeClaims (PVC)**: Requests for storage by users that can be fulfilled by PersistentVolumes
- **StorageClasses**: Provide a way to describe different "classes" of storage

## PersistentVolumes and PersistentVolumeClaims

PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) provide a way to use storage in Kubernetes that's independent of the Pod lifecycle. This abstraction allows developers to request storage resources without knowing the details of the underlying infrastructure.

- **PersistentVolume (PV)**: A piece of storage in the cluster provisioned by an administrator or dynamically using Storage Classes
- **PersistentVolumeClaim (PVC)**: A request for storage by a user that can be fulfilled by a PV
- **Storage Classes**: Allow dynamic provisioning of PersistentVolumes when PVCs request them

This abstraction allows for a clean separation between how storage is used (PVCs) and how it is implemented (PVs).

### Exercise 1: Using emptyDir Volumes

**Goal**: Learn how to share data between containers in the same Pod using an emptyDir volume.

**Task**: Create a Pod with two containers that share data using an emptyDir volume.

**Why this matters**: While emptyDir volumes are ephemeral (they exist only as long as the Pod exists), they are essential for sharing data between containers in the same Pod. This pattern is commonly used for sidecars, adapters, and other multi-container patterns where data needs to be passed between containers.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo $(date) >> /data/output.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox
    command: ["/bin/sh", "-c", "while true; do cat /data/output.txt; sleep 10; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

**Step 2: Create the Pod**

```bash
kubectl apply -f volume-share-pod.yaml
```

**Step 3: Verify that the containers are sharing data**

```bash
kubectl logs volume-share-pod -c reader
```

**What this does**:

- Creates a Pod with two containers: `writer` and `reader`
- The `writer` container writes the current date to a file every 5 seconds
- The `reader` container reads and outputs the file contents every 10 seconds
- Both containers mount the same emptyDir volume at `/data`, allowing them to share files

</p>
</details>

### Exercise 2: Creating a PersistentVolume

**Goal**: Learn how to create a PersistentVolume resource.

**Task**: Create a PersistentVolume with specific capacity and access modes.

**Why this matters**: PersistentVolumes are the foundation of stateful applications in Kubernetes. Understanding how to create and configure them is essential for applications that need to store data persistently, such as databases, file servers, or any application that needs to maintain state across restarts.

<details><summary>show solution</summary>
<p>

**Step 1: Create a PersistentVolume manifest file**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/data
```

**Step 2: Create the PersistentVolume**

```bash
kubectl apply -f task-pv.yaml
```

**Step 3: Verify the PersistentVolume was created**

```bash
kubectl get pv task-pv
```

**What this does**:

- Creates a PersistentVolume named `task-pv` with 1GB of storage capacity
- Sets the access mode to `ReadWriteOnce`, meaning it can be mounted as read-write by a single node
- Uses the `hostPath` volume type, which mounts a file or directory from the host node's filesystem
- Sets the reclaim policy to `Retain`, meaning the volume will not be automatically deleted when released

> **Note**: In a production environment, you would typically use a more robust storage solution like NFS, cloud storage, or a storage provider specific to your environment, rather than `hostPath`.

</p>
</details>

### Exercise 3: Creating a PersistentVolumeClaim

**Goal**: Learn how to create a PersistentVolumeClaim to request storage.

**Task**: Create a PersistentVolumeClaim that requests storage from the PersistentVolume created in Exercise 2.

**Why this matters**: PersistentVolumeClaims are how applications request and use persistent storage in Kubernetes. This abstraction allows developers to consume storage without needing to know the details of the underlying infrastructure, promoting a clean separation of concerns between infrastructure and application teams.

<details><summary>show solution</summary>
<p>

**Step 1: Create a PersistentVolumeClaim manifest file**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

**Step 2: Create the PersistentVolumeClaim**

```bash
kubectl apply -f task-pvc.yaml
```

**Step 3: Verify the PersistentVolumeClaim was created and bound**

```bash
kubectl get pvc task-pvc
```

**What this does**:

- Creates a PersistentVolumeClaim named `task-pvc` that requests 500MB of storage
- Sets the access mode to `ReadWriteOnce`, matching the PersistentVolume
- Kubernetes will automatically bind this claim to an available PersistentVolume that satisfies the requirements

</p>
</details>

### Exercise 4: Using a PersistentVolumeClaim in a Pod

**Goal**: Learn how to use a PersistentVolumeClaim in a Pod.

**Task**: Create a Pod that uses the PersistentVolumeClaim created in Exercise 3.

**Why this matters**: The final step in using persistent storage is mounting it in a Pod. This skill is essential for deploying stateful applications like databases, which need to maintain data across Pod restarts and reschedules. Understanding how to properly configure volume mounts ensures your application can reliably access its persistent data.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod manifest file that uses the PVC**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pod
spec:
  containers:
  - name: task-container
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: task-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: task-volume
    persistentVolumeClaim:
      claimName: task-pvc
```

**Step 2: Create the Pod**

```bash
kubectl apply -f task-pod.yaml
```

**Step 3: Verify the Pod is running**

```bash
kubectl get pod task-pod
```

**What this does**:

- Creates a Pod named `task-pod` with an nginx container
- Mounts the PersistentVolumeClaim `task-pvc` at `/usr/share/nginx/html` in the container
- The nginx container will now store its web content on the persistent volume, allowing it to survive Pod restarts

</p>
</details>

### Exercise 5: Verifying Data Persistence

**Goal**: Verify that data persists across Pod restarts when using a PersistentVolume.

**Task**: Create a file in the persistent volume, delete the Pod, create a new Pod using the same PVC, and verify the file still exists.

**Why this matters**: This exercise demonstrates the core value proposition of persistent storage: data survival beyond the lifecycle of a Pod. This is critical for stateful applications where data loss would be unacceptable. Understanding how to verify persistence helps ensure your storage configuration is working as expected.

<details><summary>show solution</summary>
<p>

**Step 1: Create a file in the persistent volume**

```bash
kubectl exec task-pod -- sh -c "echo 'Hello from PV' > /usr/share/nginx/html/index.html"
```

**Step 2: Verify the file was created**

```bash
kubectl exec task-pod -- cat /usr/share/nginx/html/index.html
```

**Step 3: Delete the Pod**

```bash
kubectl delete pod task-pod
```

**Step 4: Create a new Pod using the same PVC**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-task-pod
spec:
  containers:
  - name: task-container
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: task-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: task-volume
    persistentVolumeClaim:
      claimName: task-pvc
```

```bash
kubectl apply -f new-task-pod.yaml
```

**Step 5: Verify the file still exists**

```bash
kubectl exec new-task-pod -- cat /usr/share/nginx/html/index.html
```

**What this does**:

- Creates a file in the persistent volume mounted in the first Pod
- Deletes the first Pod
- Creates a new Pod that mounts the same PersistentVolumeClaim
- Verifies that the file created in the first Pod is still accessible in the new Pod
- This demonstrates that the data persists independently of the Pod lifecycle

</p>
</details>
