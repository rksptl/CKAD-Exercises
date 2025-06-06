# Persistent Volumes and Persistent Volume Claims

This section covers Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) in Kubernetes, which are essential for managing storage in a cluster.

## Key Resources

- [Persistent Volumes Documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Persistent Volume Claims Documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [Storage Classes Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/)

## Introduction to Persistent Volumes and Persistent Volume Claims

Persistent Volumes (PVs) are cluster resources that provide storage which can outlive the lifecycle of a Pod. Persistent Volume Claims (PVCs) are requests for storage by users. This abstraction allows for a separation of concerns between storage provisioning and consumption.

## Persistent Volumes

### Exercise 1: Creating a Persistent Volume

**Goal**: Learn how to create a Persistent Volume.

**Task**: Create a Persistent Volume with a specific capacity and access mode.

**Why this matters**: Persistent Volumes are the foundation of stateful applications in Kubernetes. Understanding how to create them is essential for managing storage in a cluster.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Persistent Volume**

Create a file named `pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

Apply the configuration:

```bash
kubectl apply -f pv.yaml
```

**Step 2: Verify that the Persistent Volume was created**

```bash
kubectl get pv
```

You should see the Persistent Volume with status `Available`.

**What this does**:

- Creates a Persistent Volume with 1Gi capacity
- Sets the access mode to ReadWriteOnce, which means it can be mounted as read-write by a single node
- Sets the reclaim policy to Retain, which means the volume will not be deleted when the PVC is deleted
- Uses the hostPath volume type, which uses a directory on the node
- Sets the storage class to manual, which means it will not be dynamically provisioned

This demonstrates how to create a basic Persistent Volume. In a real cluster, you would typically use a more robust storage solution like NFS, iSCSI, or a cloud provider's storage service.

</p>
</details>

### Exercise 2: Understanding Access Modes

**Goal**: Learn about the different access modes for Persistent Volumes.

**Task**: Create Persistent Volumes with different access modes.

**Why this matters**: Different applications have different storage requirements. Understanding access modes helps you choose the right type of storage for your application.

<details><summary>show solution</summary>
<p>

**Step 1: Create Persistent Volumes with different access modes**

Create a file named `pv-access-modes.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rwx
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-rwx
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rwo
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-rwo
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-rox
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-rox
```

Apply the configuration:

```bash
kubectl apply -f pv-access-modes.yaml
```

**Step 2: Verify that the Persistent Volumes were created**

```bash
kubectl get pv
```

You should see three Persistent Volumes with different access modes.

**What this does**:

- Creates three Persistent Volumes with different access modes:
  - ReadWriteMany (RWX): Can be mounted as read-write by many nodes
  - ReadWriteOnce (RWO): Can be mounted as read-write by a single node
  - ReadOnlyMany (ROX): Can be mounted as read-only by many nodes

This demonstrates the different access modes available for Persistent Volumes. Note that not all volume types support all access modes. For example, hostPath volumes do not actually support ReadWriteMany, but we're using them here for demonstration purposes.

</p>
</details>

### Exercise 3: Understanding Reclaim Policies

**Goal**: Learn about the different reclaim policies for Persistent Volumes.

**Task**: Create Persistent Volumes with different reclaim policies.

**Why this matters**: The reclaim policy determines what happens to the storage when the PVC is deleted. Understanding reclaim policies helps you manage the lifecycle of your storage.

<details><summary>show solution</summary>
<p>

**Step 1: Create Persistent Volumes with different reclaim policies**

Create a file named `pv-reclaim-policies.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-retain
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-delete
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: manual
  hostPath:
    path: /mnt/data-delete
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-recycle
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: manual
  hostPath:
    path: /mnt/data-recycle
```

Apply the configuration:

```bash
kubectl apply -f pv-reclaim-policies.yaml
```

**Step 2: Verify that the Persistent Volumes were created**

```bash
kubectl get pv
```

You should see three Persistent Volumes with different reclaim policies.

**What this does**:

- Creates three Persistent Volumes with different reclaim policies:
  - Retain: When the PVC is deleted, the PV remains but is not available for reuse
  - Delete: When the PVC is deleted, the PV and its associated storage are deleted
  - Recycle: When the PVC is deleted, the PV is scrubbed and made available for reuse (deprecated)

This demonstrates the different reclaim policies available for Persistent Volumes. Note that the Recycle policy is deprecated and will be removed in a future Kubernetes version.

</p>
</details>

## Persistent Volume Claims

### Exercise 1: Creating a Persistent Volume Claim

**Goal**: Learn how to create a Persistent Volume Claim.

**Task**: Create a Persistent Volume Claim and verify that it binds to a Persistent Volume.

**Why this matters**: Persistent Volume Claims are how applications request storage in Kubernetes. Understanding how to create them is essential for using storage in your applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Persistent Volume**

Create a file named `pv-for-claim.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-for-claim
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-for-claim
```

Apply the configuration:

```bash
kubectl apply -f pv-for-claim.yaml
```

**Step 2: Create a Persistent Volume Claim**

Create a file named `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Mi
  storageClassName: manual
```

Apply the configuration:

```bash
kubectl apply -f pvc.yaml
```

**Step 3: Verify that the Persistent Volume Claim was created and bound**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claim with status `Bound`.

```bash
kubectl get pv
```

You should see the Persistent Volume with status `Bound` and the claim reference pointing to your PVC.

**What this does**:

- Creates a Persistent Volume with 1Gi capacity
- Creates a Persistent Volume Claim requesting 500Mi of storage
- Kubernetes binds the PVC to the PV because they have compatible access modes and storage class
- The PVC is now ready to be used by a Pod

This demonstrates how to create a basic Persistent Volume Claim and how it binds to a Persistent Volume.

</p>
</details>

### Exercise 2: Using a Persistent Volume Claim in a Pod

**Goal**: Learn how to use a Persistent Volume Claim in a Pod.

**Task**: Create a Pod that uses a Persistent Volume Claim.

**Why this matters**: The ultimate goal of Persistent Volumes and Persistent Volume Claims is to provide storage to Pods. Understanding how to use PVCs in Pods is essential for running stateful applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Persistent Volume and Persistent Volume Claim**

Create a file named `pv-pvc-pod.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-pod
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data-pod
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-pod
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 500Mi
  storageClassName: manual
```

Apply the configuration:

```bash
kubectl apply -f pv-pvc-pod.yaml
```

**Step 2: Create a Pod that uses the Persistent Volume Claim**

Create a file named `pod-with-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-pod
```

Apply the configuration:

```bash
kubectl apply -f pod-with-pvc.yaml
```

**Step 3: Verify that the Pod is running and using the Persistent Volume Claim**

```bash
kubectl get pod pod-with-pvc
```

You should see the Pod running.

**Step 4: Write data to the volume**

```bash
kubectl exec pod-with-pvc -- sh -c "echo 'Hello from Kubernetes storage' > /usr/share/nginx/html/index.html"
```

**Step 5: Verify that the data was written**

```bash
kubectl exec pod-with-pvc -- cat /usr/share/nginx/html/index.html
```

You should see the message you wrote.

**What this does**:

- Creates a Persistent Volume and Persistent Volume Claim
- Creates a Pod that uses the Persistent Volume Claim
- Mounts the volume at `/usr/share/nginx/html` in the container
- Writes data to the volume
- Verifies that the data was written

This demonstrates how to use a Persistent Volume Claim in a Pod and how to write data to the volume.

</p>
</details>

### Exercise 3: Dynamic Provisioning with Storage Classes

**Goal**: Learn how to use Storage Classes for dynamic provisioning of Persistent Volumes.

**Task**: Create a Storage Class and use it to dynamically provision a Persistent Volume.

**Why this matters**: Dynamic provisioning allows you to create Persistent Volumes on-demand, without having to manually create them. This is essential for managing storage at scale.

<details><summary>show solution</summary>
<p>

**Note**: This exercise requires a cluster with a dynamic provisioner. If you're using Minikube, you can enable the default StorageClass:

```bash
minikube addons enable default-storageclass
```

**Step 1: Create a Storage Class**

Create a file named `storage-class.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: k8s.io/minikube-hostpath
parameters:
  type: pd-ssd
```

Apply the configuration:

```bash
kubectl apply -f storage-class.yaml
```

**Step 2: Create a Persistent Volume Claim that uses the Storage Class**

Create a file named `pvc-dynamic.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast
```

Apply the configuration:

```bash
kubectl apply -f pvc-dynamic.yaml
```

**Step 3: Verify that the Persistent Volume Claim was created and bound**

```bash
kubectl get pvc pvc-dynamic
```

You should see the Persistent Volume Claim with status `Bound`.

```bash
kubectl get pv
```

You should see a Persistent Volume that was dynamically provisioned for your PVC.

**Step 4: Create a Pod that uses the Persistent Volume Claim**

Create a file named `pod-with-dynamic-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-dynamic-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-dynamic
```

Apply the configuration:

```bash
kubectl apply -f pod-with-dynamic-pvc.yaml
```

**Step 5: Verify that the Pod is running and using the Persistent Volume Claim**

```bash
kubectl get pod pod-with-dynamic-pvc
```

You should see the Pod running.

**What this does**:

- Creates a Storage Class that uses the minikube-hostpath provisioner
- Creates a Persistent Volume Claim that uses the Storage Class
- Kubernetes dynamically provisions a Persistent Volume for the PVC
- Creates a Pod that uses the Persistent Volume Claim
- The Pod can now use the dynamically provisioned storage

This demonstrates how to use Storage Classes for dynamic provisioning of Persistent Volumes. In a real cluster, you would typically use a storage provisioner specific to your environment, such as AWS EBS, GCE PD, or a CSI driver.

</p>
</details>

## Best Practices

1. **Use Storage Classes for dynamic provisioning** whenever possible to avoid manual management of Persistent Volumes.
2. **Choose the appropriate access mode** for your application's needs.
3. **Set appropriate resource requests** in your Persistent Volume Claims to avoid over-provisioning.
4. **Use labels and selectors** to organize and manage your Persistent Volumes and Persistent Volume Claims.
5. **Consider using a volume snapshot feature** for backup and restore operations.
6. **Monitor storage usage** to avoid running out of space.
7. **Use the appropriate reclaim policy** based on your data retention requirements.
8. **Consider using a CSI driver** for advanced storage features like snapshots, cloning, and resizing.
9. **Test storage failover scenarios** to ensure your application can handle storage failures.
10. **Document your storage architecture** to help others understand your storage design decisions.
