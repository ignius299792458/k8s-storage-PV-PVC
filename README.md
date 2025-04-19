# Kubernetes Persistent Volumes and Persistent Volume Claims
Read to understand overall flow of pv, pvc, pod, node and kind-container : https://github.com/ignius299792458/k8s-storage-PV-PVC/blob/main/overall-flow-pv-and-pvc.md 
## Persistent Volume (PV)

A Persistent Volume is a piece of storage in your Kubernetes cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

**Key characteristics:**
- Exists independently of any pod that uses it
- Has a lifecycle independent of any pod
- Can be provisioned statically or dynamically
- Defined by administrators with specific capacity, access modes, and storage class

## Persistent Volume Claim (PVC)

A Persistent Volume Claim is a request for storage by a user (pod).

**Key characteristics:**
- Acts as a request for specific size, access mode, and storage class
- Binds to an available PV that meets its requirements
- Used by pods to mount storage
- Protects pods from the details of how storage is implemented

## How They Work Together

1. **Administrator creates PVs** (or sets up dynamic provisioning via StorageClasses)
2. **User creates PVC** to request storage
3. **Kubernetes binds PVC to a suitable PV**
4. **Pod uses the PVC** to mount storage

## Example Flow

```
Administrator:
  ↓ creates
PV (10GB, ReadWriteOnce)
  ↑ binds to
PVC (requests 8GB, ReadWriteOnce)
  ↑ referenced by
Pod (mounts at /data)
```

## Access Modes

- **ReadWriteOnce (RWO)**: Volume can be mounted as read-write by a single node
- **ReadOnlyMany (ROX)**: Volume can be mounted read-only by many nodes
- **ReadWriteMany (RWX)**: Volume can be mounted as read-write by many nodes

## Storage Classes

Define different "classes" of storage with different performance characteristics or policies.

## Reclaim Policies

- **Retain**: Manual reclamation
- **Delete**: Automatically delete PV when PVC is deleted
- **Recycle**: Basic scrub (deprecated)

This separation of concerns allows users to consume storage resources without knowing the underlying details of the storage infrastructure.
