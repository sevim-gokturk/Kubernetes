# Kubernetes Volumes

## What is a Volume?
A **Kubernetes Volume** is a storage unit for data inside a pod. It helps store data beyond the container lifecycle.

## Types of Volumes

### 1. **emptyDir**
- Temporary storage.
- Data is lost when the pod stops.
- Example: Cache files.

### 2. **hostPath**
- Uses the host machine's directory.
- Example: Accessing log files.

### 3. **PersistentVolume (PV) & PersistentVolumeClaim (PVC)**
- Permanent storage.
- Used for databases or important files.

### 4. **configMap & secret**
- Stores configuration files and sensitive data.
- Example: API keys, passwords.

### 5. **nfs (Network File System)**
- Shared storage over a network.
- Used when multiple pods need access to the same files.

### 6. **Cloud Storage (AWS, Azure, GCP)**
- **awsElasticBlockStore** → AWS storage.
- **azureDisk** → Azure storage.
- **gcePersistentDisk** → Google Cloud storage.

### 7. **Distributed Storage (cephFS, glusterFS)**
- Used for large-scale applications.

## When to Use Each?
| **Volume Type** | **Use Case** |
|---------------|------------|
| emptyDir | Temporary data |
| hostPath | Log files, host system access |
| PersistentVolume | Databases, important files |
| configMap & secret | App settings, passwords |
| nfs | Shared storage |
| Cloud Storage | Cloud-based applications |
| Distributed Storage | Scalable, high-availability apps |

