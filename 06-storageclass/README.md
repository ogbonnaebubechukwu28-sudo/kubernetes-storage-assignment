# Tasks 9-10 - StorageClass & Dynamic Provisioning

## What I did
Inspected Minikube's built-in standard StorageClass, then created a PVC (bookstore-dynamic-pvc) referencing it explicitly without manually creating a PV, to observe dynamic provisioning in action.

## Dynamic Provisioning Flow

PVC
  down to
StorageClass (standard)
  down to
Provisioner (k8s.io/minikube-hostpath)
  down to
PersistentVolume (auto-created)
  down to
hostPath storage on the node

Applying bookstore-dynamic-pvc immediately resulted in a brand-new PV (pvc-03931d9b-...) being created and bound automatically - no pv.yaml was written for it.

## StorageClass Details

**Name:** standard

**Provisioner:** k8s.io/minikube-hostpath - Minikube's built-in provisioner that dynamically creates hostPath-backed volumes on the node.

**Reclaim policy:** Delete - the dynamically-created PV and its underlying storage are deleted automatically when the PVC is deleted.

**Volume binding mode:** Immediate - the PV is provisioned as soon as the PVC is created, rather than waiting for a Pod to consume it.

**Default StorageClass:** Yes - confirmed by IsDefaultClass: Yes. This is why any PVC that doesn't specify a storageClassName automatically uses it.
