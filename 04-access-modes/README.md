# Task 7 — Access Modes

## What I did
Created a PVC (`bookstore-pvc-rwo`) requesting 500Mi with access mode `ReadWriteOnce`, and researched what each access mode means.

## Access Modes Table

| Access Mode | Meaning | Example Use Case |
|---|---|---|
| RWO (ReadWriteOnce) | Volume can be mounted as read-write by a single node at a time (multiple Pods on that same node can still use it) | A single-instance database like MySQL or PostgreSQL, or any app that doesn't need to share storage across nodes |
| ROX (ReadOnlyMany) | Volume can be mounted read-only by many nodes simultaneously | Distributing shared, static reference data or configuration files to many read-only replicas, e.g. a shared dataset for multiple analytics Pods |
| RWX (ReadWriteMany) | Volume can be mounted as read-write by many nodes simultaneously | Shared file storage for multiple Pods writing/reading concurrently, e.g. a shared uploads directory for a horizontally-scaled web app |

## Does every storage backend support every access mode?

No. Access mode support depends entirely on the underlying storage backend/CSI driver. For example, `hostPath` (used throughout this assignment) effectively only supports `ReadWriteOnce`, since it's tied to a single node's local disk — there's no mechanism for multiple nodes to access the same local directory. Network-based storage systems like NFS support `ReadWriteMany`, while typical cloud block storage (like AWS EBS or GCE Persistent Disk) generally only supports `ReadWriteOnce`. This is why it's important to check a StorageClass/provisioner's documentation for which access modes it actually supports before assuming a mode will work.
