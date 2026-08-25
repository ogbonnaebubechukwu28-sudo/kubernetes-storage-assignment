# Kubernetes Storage Practical Assignment

### Student Information

Name: Ebubechukwu Ogbonna

## 2. Objective

This project demonstrates my understanding of Kubernetes storage by deploying a simple bookstore application and working through the different storage mechanisms Kubernetes offers: emptyDir, hostPath, PersistentVolumes, PersistentVolumeClaims, access modes, reclaim policies, StorageClasses, and dynamic provisioning. Each task builds on the last, moving from temporary, Pod-scoped storage to fully persistent, dynamically-provisioned storage that survives Pod deletion and recreation.

## 3. Environment

Kubernetes: Minikube v1.38.1 (Kubernetes v1.31.0)
Container Runtime: Docker (driver: docker)
kubectl: v1.31.14
OS: Ubuntu 24.04 (WSL2)

## 4. Concepts Covered

- emptyDir
- hostPath
- PersistentVolume
- PersistentVolumeClaim
- Access Modes
- Reclaim Policies
- StorageClass
- Dynamic Provisioning

## 5. Architecture Diagram

Kubernetes Cluster
    ↓
Pod
    ↓
PVC (PersistentVolumeClaim)
    ↓
PV (PersistentVolume)
    ↓
Storage Backend (hostPath on the Minikube node)

For dynamically-provisioned storage, the flow is:

PVC
    ↓
StorageClass (standard)
    ↓
Provisioner (k8s.io/minikube-hostpath)
    ↓
PersistentVolume (auto-created)
    ↓
Storage Backend

## 6. Commands Used

```bash
minikube start --driver=docker
kubectl get nodes

kubectl apply -f 01-emptydir/pod.yaml
kubectl exec emptydir-demo -- sh -c "echo Hello Kubernetes > /data/test.txt"
kubectl exec emptydir-demo -- cat /data/test.txt
kubectl delete pod emptydir-demo
kubectl apply -f 01-emptydir/pod.yaml
kubectl exec emptydir-demo -- cat /data/test.txt

kubectl apply -f 02-hostpath/pod.yaml
kubectl exec hostpath-demo -- sh -c "echo Bookstore Data > /data/book.txt"
kubectl exec hostpath-demo -- cat /data/book.txt
minikube ssh
ls /data/bookstore
cat /data/bookstore/book.txt

kubectl apply -f 03-persistent-volume/pv.yaml
kubectl apply -f 03-persistent-volume/pvc.yaml
kubectl apply -f 03-persistent-volume/pod.yaml
kubectl get pv
kubectl get pvc
kubectl exec bookstore -- sh -c "echo Kubernetes Storage > /data/storage-test.txt"
kubectl delete pod bookstore
kubectl apply -f 03-persistent-volume/pod.yaml
kubectl exec bookstore -- cat /data/storage-test.txt

kubectl apply -f 04-access-modes/pvc-rwo.yaml
kubectl get pvc

kubectl apply -f 05-reclaim-policy/pv-retain.yaml
kubectl apply -f 05-reclaim-policy/pvc.yaml
kubectl delete pvc reclaim-demo-pvc
kubectl get pv

kubectl get storageclass
kubectl describe storageclass standard
kubectl apply -f 06-storageclass/pvc.yaml
kubectl get pvc
kubectl get pv

kubectl apply -f final-project/pv.yaml
kubectl apply -f final-project/pvc.yaml
kubectl apply -f final-project/pod.yaml
kubectl exec final-bookstore -- sh -c "echo Final Challenge Data > /data/final-test.txt"
kubectl delete pod final-bookstore
kubectl apply -f final-project/pod.yaml
kubectl exec final-bookstore -- cat /data/final-test.txt
```

## 7. Screenshots

- `minikube.png` - Minikube running, node Ready
- `emptydir.png` - emptyDir file lost after Pod recreation
- `hostpath.png` - hostPath file confirmed on the Minikube node via minikube ssh
- `pv.png` - bookstore-pv created, status Available
- `pvc-bound.png` - bookstore-pvc bound to bookstore-pv
- `persistent-data.png` - data surviving Pod delete/recreate
- `storageclass.png` - kubectl describe storageclass standard
- `dynamic-provisioning.png` - PVC triggering automatic PV creation

## 8. Required Questions

**Question 1 - What is the difference between emptyDir and hostPath?**
emptyDir creates a brand-new, empty directory that is tied to a Pod's lifecycle - it's deleted along with the Pod. hostPath instead mounts a specific, pre-existing path on the node's own filesystem into the Pod, so the data lives independently of any one Pod and persists even after that Pod is deleted, as long as the Pod is rescheduled to the same node.

**Question 2 - What happens to emptyDir data when a Pod is deleted?**
It is permanently lost. I confirmed this directly: after writing test.txt to an emptyDir volume, deleting the Pod, and recreating it from the same manifest, the file was gone - the new Pod got a completely fresh, empty volume with no link to the old one.

**Question 3 - What is the difference between a PV and PVC?**
A PersistentVolume (PV) is the actual piece of storage in the cluster - provisioned either manually by an admin or dynamically by a StorageClass. A PersistentVolumeClaim (PVC) is a request for storage made by a user or Pod, specifying how much space and which access mode is needed. A Pod never talks to a PV directly - it always goes through a PVC, which gets matched (bound) to a suitable PV.

**Question 4 - Describe the PV/PVC binding process.**
When a PVC is created, Kubernetes looks for a PV that satisfies its requirements (enough capacity, matching access mode, and matching storageClassName). If a suitable PV already exists (as with my statically-created bookstore-pv), the PVC binds directly to it. If none exists and a StorageClass is specified, the StorageClass's provisioner dynamically creates a new PV that exactly matches the PVC's request, and binds them together. Once bound, the PV and PVC are exclusively linked to each other until the PVC is deleted.

**Question 5 - What does ReadWriteOnce mean?**
The volume can be mounted as read-write by a single node at a time. Multiple Pods can still use it if they're scheduled on that same node, but it cannot be simultaneously mounted read-write across different nodes.

**Question 6 - What does ReadOnlyMany mean?**
The volume can be mounted as read-only by many nodes at the same time. It's useful for distributing the same static data or configuration to many Pods spread across different nodes, none of which need to write to it.

**Question 7 - What does ReadWriteMany mean?**
The volume can be mounted as read-write by many nodes simultaneously. This is needed for shared storage scenarios where multiple Pods across different nodes need to read and write the same data concurrently, such as a shared uploads directory.

**Question 8 - Why might hostPath be unsuitable for a multi-node production application?**
Because hostPath data lives on one specific node's disk only. If a Pod using hostPath is rescheduled to a different node (which happens routinely during scaling, node failures, or upgrades), it will not find its previous data - it will see an empty or nonexistent directory on the new node instead. This makes hostPath unreliable for any application that needs consistent access to its data regardless of which node it lands on.

**Question 9 - What is a StorageClass?**
A StorageClass defines a "class" or category of storage available in the cluster, along with which provisioner should be used to create volumes of that class and what policies (reclaim policy, binding mode) apply. It lets a PVC request storage without needing a matching PV to already exist - the StorageClass handles creating one automatically.

**Question 10 - What is dynamic provisioning?**
Dynamic provisioning is when Kubernetes automatically creates a PersistentVolume on demand, triggered by a PVC that references a StorageClass, instead of an administrator manually pre-creating PVs ahead of time. I demonstrated this directly: applying a PVC referencing the standard StorageClass immediately resulted in a new PV being created and bound, with no pv.yaml written for it.

**Question 11 - What is the difference between Retain and Delete reclaim policies?**
With Retain, when the bound PVC is deleted, the PV is not deleted - it moves to a Released state, keeping its data intact until an administrator manually decides what to do with it. With Delete, the PV and its underlying storage are automatically deleted as soon as the PVC is deleted. I observed both behaviors directly: my Retain-policy PV became Released (not removed) after its PVC was deleted, while my dynamically-provisioned PVs used the Delete policy by default.

**Question 12 - Why is Recycle generally not recommended for modern Kubernetes?**
Recycle is deprecated. It used to perform a basic scrub of the volume's data (essentially an rm -rf) and make the PV available for a new claim, but this approach was unsafe (a simple wipe isn't equivalent to properly reprovisioning storage) and inflexible across different storage backends. Dynamic provisioning with StorageClasses has replaced it as the modern, safer approach.

**Question 13 - What happens to a PV when its PVC is deleted under the Retain policy?**
The PV is not deleted. It transitions to a Released status and keeps a reference to the deleted PVC in its CLAIM field. The underlying data remains fully intact, but the PV is not automatically made available for a new PVC to bind to - it requires manual intervention. I confirmed this exact behavior when I deleted reclaim-demo-pvc and reclaim-demo-pv remained, showing status Released.

**Question 14 - Why is persistent storage important for databases?**
Databases hold state that must survive far beyond the lifetime of any single Pod. Pods are inherently disposable - they get rescheduled, restarted, or replaced regularly. Without persistent storage, a database Pod restarting would lose all its data instantly, which is unacceptable. A PV/PVC ensures the actual data lives independently of the Pod running the database engine, so the Pod can be replaced without any data loss.

**Question 15 - What is the role of a CSI driver?**
A CSI (Container Storage Interface) driver is the component that lets Kubernetes talk to a specific storage backend (like AWS EBS, Azure Disk, NFS, or in this case Minikube's own hostpath provisioner) in a standardized way. It implements the actual work behind provisioning, attaching, mounting, and deleting volumes for whichever storage system it represents, so Kubernetes doesn't need built-in, storage-specific code for every possible backend - any storage vendor can write a CSI driver and plug into Kubernetes using the same interface.
