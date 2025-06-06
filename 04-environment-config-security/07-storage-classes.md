# Storage Classes

This section covers Storage Classes in Kubernetes, which are used for dynamic provisioning of Persistent Volumes.

## Key Resources

- [Storage Classes Documentation](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Dynamic Provisioning Documentation](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)
- [Volume Expansion Documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-expansion)

## Introduction to Storage Classes

Storage Classes provide a way to describe different "classes" of storage offered in a cluster. They can be used to dynamically provision Persistent Volumes when a Persistent Volume Claim is created.

## Creating and Using Storage Classes

### Exercise 1: Creating a Storage Class

**Goal**: Learn how to create a Storage Class.

**Task**: Create a Storage Class with specific provisioner and parameters.

**Why this matters**: Storage Classes are the foundation of dynamic provisioning in Kubernetes. Understanding how to create them is essential for managing storage in a cluster.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Storage Class**

Create a file named `storage-class.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: none
```

Apply the configuration:

```bash
kubectl apply -f storage-class.yaml
```

**Step 2: Verify that the Storage Class was created**

```bash
kubectl get storageclass
```

You should see your Storage Class listed.

**What this does**:

- Creates a Storage Class named `fast`
- Uses the GCE PD provisioner (for Google Cloud)
- Specifies parameters for the provisioner, such as the type of disk to create
- This Storage Class can now be used to dynamically provision Persistent Volumes

Note: The provisioner and parameters will vary depending on your cloud provider or storage system. This example uses GCE PD, but you would use a different provisioner for AWS, Azure, or on-premises storage.

</p>
</details>

### Exercise 2: Using a Storage Class for Dynamic Provisioning

**Goal**: Learn how to use a Storage Class to dynamically provision a Persistent Volume.

**Task**: Create a Persistent Volume Claim that uses a Storage Class.

**Why this matters**: Dynamic provisioning allows you to create Persistent Volumes on-demand, without having to manually create them. This is essential for managing storage at scale.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Persistent Volume Claim that uses a Storage Class**

Create a file named `pvc-with-sc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-fast
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
kubectl apply -f pvc-with-sc.yaml
```

**Step 2: Verify that the Persistent Volume Claim was created and bound**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claim with status `Bound` or `Pending` (if the provisioner is still creating the volume).

```bash
kubectl get pv
```

You should see a Persistent Volume that was dynamically provisioned for your PVC.

**Step 3: Create a Pod that uses the Persistent Volume Claim**

Create a file named `pod-with-sc-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sc-pvc
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
      claimName: pvc-fast
```

Apply the configuration:

```bash
kubectl apply -f pod-with-sc-pvc.yaml
```

**Step 4: Verify that the Pod is running and using the Persistent Volume Claim**

```bash
kubectl get pod pod-with-sc-pvc
```

You should see the Pod running.

**What this does**:

- Creates a Persistent Volume Claim that uses the `fast` Storage Class
- Kubernetes dynamically provisions a Persistent Volume for the PVC
- Creates a Pod that uses the Persistent Volume Claim
- The Pod can now use the dynamically provisioned storage

This demonstrates how to use Storage Classes for dynamic provisioning of Persistent Volumes.

</p>
</details>

### Exercise 3: Setting a Default Storage Class

**Goal**: Learn how to set a default Storage Class.

**Task**: Create a Storage Class and set it as the default.

**Why this matters**: A default Storage Class is used when a Persistent Volume Claim does not specify a Storage Class. This simplifies storage management for users who don't need to specify a Storage Class for every PVC.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Storage Class and set it as the default**

Create a file named `default-sc.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```

Apply the configuration:

```bash
kubectl apply -f default-sc.yaml
```

**Step 2: Verify that the Storage Class was created and set as the default**

```bash
kubectl get storageclass
```

You should see your Storage Class listed with `(default)` next to its name.

**Step 3: Create a Persistent Volume Claim without specifying a Storage Class**

Create a file named `pvc-default-sc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply the configuration:

```bash
kubectl apply -f pvc-default-sc.yaml
```

**Step 4: Verify that the Persistent Volume Claim was created and bound**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claim with status `Bound` or `Pending` (if the provisioner is still creating the volume).

```bash
kubectl describe pvc pvc-default
```

You should see that the Storage Class is set to `standard`.

**What this does**:

- Creates a Storage Class named `standard`
- Sets it as the default Storage Class using the annotation `storageclass.kubernetes.io/is-default-class: "true"`
- Creates a Persistent Volume Claim without specifying a Storage Class
- Kubernetes uses the default Storage Class to dynamically provision a Persistent Volume for the PVC

This demonstrates how to set a default Storage Class and how it is used when a Persistent Volume Claim does not specify a Storage Class.

</p>
</details>

## Advanced Storage Class Features

### Exercise 1: Volume Expansion

**Goal**: Learn how to enable and use volume expansion.

**Task**: Create a Storage Class that allows volume expansion and expand a Persistent Volume Claim.

**Why this matters**: Volume expansion allows you to increase the size of a Persistent Volume Claim without having to recreate it. This is useful when your application needs more storage than initially allocated.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Storage Class that allows volume expansion**

Create a file named `expandable-sc.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expandable
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
allowVolumeExpansion: true
```

Apply the configuration:

```bash
kubectl apply -f expandable-sc.yaml
```

**Step 2: Create a Persistent Volume Claim that uses the Storage Class**

Create a file named `expandable-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expandable-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: expandable
```

Apply the configuration:

```bash
kubectl apply -f expandable-pvc.yaml
```

**Step 3: Verify that the Persistent Volume Claim was created and bound**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claim with status `Bound` or `Pending` (if the provisioner is still creating the volume).

**Step 4: Expand the Persistent Volume Claim**

Edit the PVC to increase the storage request:

```bash
kubectl edit pvc expandable-pvc
```

Change the `storage` value from `1Gi` to `2Gi`.

Alternatively, you can create a new file with the updated configuration and apply it:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: expandable-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  storageClassName: expandable
```

Apply the configuration:

```bash
kubectl apply -f expandable-pvc-expanded.yaml
```

**Step 5: Verify that the Persistent Volume Claim was expanded**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claim with the new size.

**What this does**:

- Creates a Storage Class that allows volume expansion
- Creates a Persistent Volume Claim that uses the Storage Class
- Expands the Persistent Volume Claim from 1Gi to 2Gi
- Kubernetes expands the underlying Persistent Volume

This demonstrates how to enable and use volume expansion. Note that volume expansion is only supported by certain storage providers and may require the Pod to be restarted to see the new size.

</p>
</details>

### Exercise 2: Volume Binding Modes

**Goal**: Learn about the different volume binding modes.

**Task**: Create Storage Classes with different volume binding modes.

**Why this matters**: Volume binding modes determine when volume binding and dynamic provisioning occur. This can be important for scheduling Pods that require storage.

<details><summary>show solution</summary>
<p>

**Step 1: Create Storage Classes with different volume binding modes**

Create a file named `binding-modes-sc.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: immediate-binding
provisioner: kubernetes.io/gce-pd
volumeBindingMode: Immediate
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: waitforfirstconsumer-binding
provisioner: kubernetes.io/gce-pd
volumeBindingMode: WaitForFirstConsumer
```

Apply the configuration:

```bash
kubectl apply -f binding-modes-sc.yaml
```

**Step 2: Create Persistent Volume Claims that use the Storage Classes**

Create a file named `binding-modes-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: immediate-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: immediate-binding
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: waitforfirstconsumer-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: waitforfirstconsumer-binding
```

Apply the configuration:

```bash
kubectl apply -f binding-modes-pvc.yaml
```

**Step 3: Verify the status of the Persistent Volume Claims**

```bash
kubectl get pvc
```

You should see that the `immediate-pvc` is `Bound` or `Pending` (if the provisioner is still creating the volume), while the `waitforfirstconsumer-pvc` is `Pending` until a Pod uses it.

**Step 4: Create a Pod that uses the WaitForFirstConsumer PVC**

Create a file named `waitforfirstconsumer-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: waitforfirstconsumer-pod
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
      claimName: waitforfirstconsumer-pvc
```

Apply the configuration:

```bash
kubectl apply -f waitforfirstconsumer-pod.yaml
```

**Step 5: Verify that the Persistent Volume Claim is now bound**

```bash
kubectl get pvc
```

You should see that the `waitforfirstconsumer-pvc` is now `Bound`.

**What this does**:

- Creates two Storage Classes with different volume binding modes:
  - `Immediate`: Volume binding and dynamic provisioning occur as soon as the PVC is created
  - `WaitForFirstConsumer`: Volume binding and dynamic provisioning are delayed until a Pod using the PVC is created
- Creates two Persistent Volume Claims, one for each Storage Class
- Creates a Pod that uses the `waitforfirstconsumer-pvc`
- Kubernetes binds and provisions the volume for the `waitforfirstconsumer-pvc` when the Pod is created

This demonstrates the different volume binding modes. `WaitForFirstConsumer` is useful for topology-aware provisioning, where the volume should be provisioned in the same zone as the Pod that will use it.

</p>
</details>

### Exercise 3: Reclaim Policies

**Goal**: Learn how to set reclaim policies for Storage Classes.

**Task**: Create Storage Classes with different reclaim policies.

**Why this matters**: The reclaim policy determines what happens to the Persistent Volume when the Persistent Volume Claim is deleted. This is important for data retention and cleanup.

<details><summary>show solution</summary>
<p>

**Step 1: Create Storage Classes with different reclaim policies**

Create a file named `reclaim-policies-sc.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: retain-sc
provisioner: kubernetes.io/gce-pd
reclaimPolicy: Retain
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delete-sc
provisioner: kubernetes.io/gce-pd
reclaimPolicy: Delete
```

Apply the configuration:

```bash
kubectl apply -f reclaim-policies-sc.yaml
```

**Step 2: Create Persistent Volume Claims that use the Storage Classes**

Create a file named `reclaim-policies-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: retain-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: retain-sc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: delete-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: delete-sc
```

Apply the configuration:

```bash
kubectl apply -f reclaim-policies-pvc.yaml
```

**Step 3: Verify that the Persistent Volume Claims were created and bound**

```bash
kubectl get pvc
```

You should see the Persistent Volume Claims with status `Bound` or `Pending` (if the provisioner is still creating the volumes).

```bash
kubectl get pv
```

You should see the Persistent Volumes that were dynamically provisioned for your PVCs.

**Step 4: Delete the Persistent Volume Claims**

```bash
kubectl delete pvc retain-pvc
kubectl delete pvc delete-pvc
```

**Step 5: Verify what happened to the Persistent Volumes**

```bash
kubectl get pv
```

You should see that the Persistent Volume for the `retain-pvc` is still there with status `Released`, while the Persistent Volume for the `delete-pvc` has been deleted.

**What this does**:

- Creates two Storage Classes with different reclaim policies:
  - `Retain`: When the PVC is deleted, the PV remains but is not available for reuse
  - `Delete`: When the PVC is deleted, the PV and its associated storage are deleted
- Creates two Persistent Volume Claims, one for each Storage Class
- Deletes the Persistent Volume Claims
- Kubernetes handles the Persistent Volumes according to their reclaim policies

This demonstrates how to set reclaim policies for Storage Classes and how they affect the lifecycle of Persistent Volumes.

</p>
</details>

## Best Practices

1. **Use appropriate Storage Classes for different workloads** based on performance, availability, and cost requirements.
2. **Set a default Storage Class** to simplify storage management for users.
3. **Enable volume expansion** for Storage Classes that support it to allow for growth without data migration.
4. **Use the WaitForFirstConsumer binding mode** for topology-aware provisioning.
5. **Set appropriate reclaim policies** based on your data retention requirements.
6. **Document your Storage Classes** to help users choose the right one for their workloads.
7. **Monitor storage usage** to avoid running out of space.
8. **Consider using a CSI driver** for advanced storage features like snapshots, cloning, and resizing.
9. **Test storage failover scenarios** to ensure your application can handle storage failures.
10. **Regularly review and update your Storage Classes** as your storage requirements evolve.
