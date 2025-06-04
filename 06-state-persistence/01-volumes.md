# Volumes

This section covers Kubernetes Volumes, which provide storage for Pods that persists beyond the lifecycle of containers.

## Key Resources

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Introduction to Volumes

Kubernetes Volumes provide a way to store and share data between containers and across Pod restarts. They address the ephemeral nature of container storage by providing persistent storage options.

## Key Concepts

- **Volume**: Storage that is accessible to containers in a Pod
- **Volume Types**: emptyDir, hostPath, configMap, secret, persistentVolumeClaim, etc.
- **Persistent Volume (PV)**: Cluster-level storage resource
- **Persistent Volume Claim (PVC)**: Request for storage by a user
- **Storage Class**: Describes the "class" of storage provided by a storage backend

## Volumes

### Exercise 1: Using emptyDir Volumes

**Goal**: Learn how to use emptyDir volumes to share data between containers in a Pod.

**Task**: Create a Pod with multiple containers that share data using an emptyDir volume.

**Why this matters**: emptyDir volumes provide temporary storage that exists for the lifetime of a Pod. They are useful for sharing data between containers in the same Pod, such as for sharing files between a main application container and a sidecar container. Understanding how to use emptyDir volumes is essential for implementing common container patterns like sidecars, adapters, and ambassadors.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with an emptyDir volume**

Create a file named `emptydir-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: writer
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "$(date) - New data" >> /data/output.txt;
        sleep 5;
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        if [ -f /data/output.txt ]; then
          echo "Reader: $(tail -1 /data/output.txt)";
        fi;
        sleep 10;
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f emptydir-pod.yaml
```

**Step 3: Check the Pod status**

```bash
kubectl get pod emptydir-pod
```

**Step 4: View the logs from the reader container**

```bash
kubectl logs emptydir-pod -c reader
```

You should see the reader container reading the data written by the writer container.

**Step 5: Verify the shared data**

```bash
kubectl exec emptydir-pod -c writer -- cat /data/output.txt
kubectl exec emptydir-pod -c reader -- cat /data/output.txt
```

Both commands should show the same content, demonstrating that the data is shared between containers.

**What this does**:

- Creates a Pod with two containers: a writer and a reader
- The writer container writes data to a file in the shared volume
- The reader container reads data from the same file
- Both containers mount the same emptyDir volume at `/data`
- The emptyDir volume is created when the Pod is assigned to a node and exists for the lifetime of the Pod
- This demonstrates how containers in the same Pod can share data using an emptyDir volume

</p>
</details>

### Exercise 2: Using hostPath Volumes

**Goal**: Learn how to use hostPath volumes to access files from the node's filesystem.

**Task**: Create a Pod that accesses node's filesystem using a hostPath volume.

**Why this matters**: hostPath volumes allow Pods to access files from the node's filesystem. This is useful for accessing node-level logs, configuration files, or other data that exists on the node. However, hostPath volumes should be used with caution as they can introduce security risks and node dependencies.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Pod with a hostPath volume**

Create a file named `hostpath-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: test-container
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "Node hostname: $(cat /node-data/hostname)";
        echo "Node kernel version: $(cat /node-data/kernel-version)";
        sleep 30;
      done
    volumeMounts:
    - name: node-data
      mountPath: /node-data
  volumes:
  - name: node-data
    hostPath:
      path: /etc
      type: Directory
```

**Step 2: Apply the Pod configuration**

```bash
kubectl apply -f hostpath-pod.yaml
```

**Step 3: Check the Pod status**

```bash
kubectl get pod hostpath-pod
```

**Step 4: View the Pod logs**

```bash
kubectl logs hostpath-pod
```

You should see the container reading data from the node's filesystem.

**Step 5: Verify the hostPath volume**

```bash
kubectl exec hostpath-pod -- ls -la /node-data
```

This should show the contents of the `/etc` directory on the node.

**What this does**:

- Creates a Pod with a container that accesses data from the node's filesystem
- The container mounts a hostPath volume at `/node-data`
- The hostPath volume points to the `/etc` directory on the node
- The container reads files from the node's filesystem
- This demonstrates how Pods can access node-level data using hostPath volumes
- Note: hostPath volumes should be used with caution as they can introduce security risks and node dependencies

</p>
</details>

### Exercise 3: Using ConfigMap and Secret Volumes

**Goal**: Learn how to mount ConfigMaps and Secrets as volumes in Pods.

**Task**: Create ConfigMaps and Secrets and mount them as volumes in a Pod.

**Why this matters**: ConfigMap and Secret volumes provide a way to inject configuration data and sensitive information into containers. This is a key part of the Kubernetes configuration management strategy, allowing you to separate configuration from application code and manage sensitive data securely.

<details><summary>show solution</summary>
<p>

**Step 1: Create a ConfigMap**

```bash
kubectl create configmap app-config --from-literal=app.properties="server.port=8080
log.level=INFO
app.mode=production"
```

**Step 2: Create a Secret**

```bash
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=supersecret
```

**Step 3: Create a Pod that mounts the ConfigMap and Secret as volumes**

Create a file named `config-volume-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "Config file content:";
        cat /etc/config/app.properties;
        echo "Secret username:";
        cat /etc/secret/username;
        echo "Secret password:";
        cat /etc/secret/password;
        sleep 30;
      done
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
  - name: secret-volume
    secret:
      secretName: app-secret
```

**Step 4: Apply the Pod configuration**

```bash
kubectl apply -f config-volume-pod.yaml
```

**Step 5: Check the Pod status**

```bash
kubectl get pod config-volume-pod
```

**Step 6: View the Pod logs**

```bash
kubectl logs config-volume-pod
```

You should see the container reading data from the ConfigMap and Secret volumes.

**Step 7: Verify the mounted volumes**

```bash
kubectl exec config-volume-pod -- ls -la /etc/config
kubectl exec config-volume-pod -- ls -la /etc/secret
```

**What this does**:

- Creates a ConfigMap and a Secret with configuration data
- Creates a Pod that mounts the ConfigMap and Secret as volumes
- The ConfigMap is mounted at `/etc/config`
- The Secret is mounted at `/etc/secret` with read-only access
- Each key in the ConfigMap and Secret becomes a file in the mounted directory
- This demonstrates how to inject configuration and sensitive data into containers using volumes

</p>
</details>

### Exercise 4: Using Persistent Volumes and Claims

**Goal**: Learn how to create and use Persistent Volumes and Persistent Volume Claims.

**Task**: Create a Persistent Volume, claim it with a Persistent Volume Claim, and use it in a Pod.

**Why this matters**: Persistent Volumes and Claims provide a way to decouple storage provisioning from storage consumption in Kubernetes. This abstraction allows administrators to provision storage resources and developers to consume them without needing to know the details of the underlying storage infrastructure.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Persistent Volume**

Create a file named `persistent-volume.yaml` with the following content:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /tmp/data
```

**Step 2: Apply the Persistent Volume configuration**

```bash
kubectl apply -f persistent-volume.yaml
```

**Step 3: Create a Persistent Volume Claim**

Create a file named `persistent-volume-claim.yaml` with the following content:

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
  storageClassName: manual
```

**Step 4: Apply the Persistent Volume Claim configuration**

```bash
kubectl apply -f persistent-volume-claim.yaml
```

**Step 5: Check the status of the PV and PVC**

```bash
kubectl get pv
kubectl get pvc
```

The PV should be in the "Bound" state, and the PVC should be bound to the PV.

**Step 6: Create a Pod that uses the PVC**

Create a file named `pvc-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "$(date) - Writing to persistent storage" >> /data/output.txt;
        echo "Current content of output.txt:";
        cat /data/output.txt;
        sleep 10;
      done
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: task-pvc
```

**Step 7: Apply the Pod configuration**

```bash
kubectl apply -f pvc-pod.yaml
```

**Step 8: Check the Pod status**

```bash
kubectl get pod pvc-pod
```

**Step 9: View the Pod logs**

```bash
kubectl logs pvc-pod
```

You should see the container writing to and reading from the persistent storage.

**Step 10: Delete the Pod and create a new one**

```bash
kubectl delete pod pvc-pod
kubectl apply -f pvc-pod.yaml
```

**Step 11: View the logs of the new Pod**

```bash
kubectl logs pvc-pod
```

You should see that the data persisted even after the Pod was deleted and recreated.

**What this does**:

- Creates a Persistent Volume (PV) using hostPath (for demonstration purposes)
- Creates a Persistent Volume Claim (PVC) that requests storage from the PV
- Creates a Pod that uses the PVC for storage
- Demonstrates that data persists even when the Pod is deleted and recreated
- This shows how to use persistent storage in Kubernetes to store data that needs to survive Pod restarts

</p>
</details>

### Exercise 5: Using Storage Classes for Dynamic Provisioning

**Goal**: Learn how to use Storage Classes for dynamic provisioning of Persistent Volumes.

**Task**: Create a Storage Class and use it to dynamically provision a Persistent Volume.

**Why this matters**: Storage Classes enable dynamic provisioning of Persistent Volumes, eliminating the need for cluster administrators to pre-provision storage. This automation simplifies storage management and allows applications to request storage on-demand, which is essential for cloud-native applications.

<details><summary>show solution</summary>
<p>

**Step 1: Create a Storage Class**

Note: The exact configuration of the Storage Class depends on the cloud provider or storage system you're using. This example uses a generic configuration.

Create a file named `storage-class.yaml` with the following content:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: k8s.io/minikube-hostpath  # Use the appropriate provisioner for your environment
parameters:
  type: ssd
```

**Step 2: Apply the Storage Class configuration**

```bash
kubectl apply -f storage-class.yaml
```

**Step 3: Create a Persistent Volume Claim that uses the Storage Class**

Create a file named `dynamic-pvc.yaml` with the following content:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast
```

**Step 4: Apply the Persistent Volume Claim configuration**

```bash
kubectl apply -f dynamic-pvc.yaml
```

**Step 5: Check the status of the PVC**

```bash
kubectl get pvc dynamic-pvc
```

The PVC should be in the "Bound" state, and a PV should have been dynamically provisioned.

**Step 6: Check the dynamically provisioned PV**

```bash
kubectl get pv
```

You should see a PV that was automatically created to satisfy the PVC.

**Step 7: Create a Pod that uses the PVC**

Create a file named `dynamic-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["/bin/sh", "-c"]
    args:
    - >
      while true;
      do
        echo "$(date) - Writing to dynamically provisioned storage" >> /data/output.txt;
        echo "Current content of output.txt:";
        cat /data/output.txt;
        sleep 10;
      done
    volumeMounts:
    - name: data-volume
      mountPath: /data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

**Step 8: Apply the Pod configuration**

```bash
kubectl apply -f dynamic-pod.yaml
```

**Step 9: Check the Pod status**

```bash
kubectl get pod dynamic-pod
```

**Step 10: View the Pod logs**

```bash
kubectl logs dynamic-pod
```

**What this does**:

- Creates a Storage Class that defines the type of storage to provision
- Creates a PVC that references the Storage Class
- The system dynamically provisions a PV to satisfy the PVC
- Creates a Pod that uses the dynamically provisioned storage
- This demonstrates how to use Storage Classes for dynamic provisioning of storage

</p>
</details>
