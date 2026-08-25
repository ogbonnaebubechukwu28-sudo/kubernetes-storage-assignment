# Tasks 3–6 — PersistentVolume, PersistentVolumeClaim, and Data Persistence

## What I did
Created a PersistentVolume (`bookstore-pv`, 1Gi, RWO, Retain, backed by hostPath) and a PersistentVolumeClaim (`bookstore-pvc`, requesting 500Mi, RWO). Bound them statically by setting `storageClassName: ""` on both, so the PVC would bind directly to the PV instead of triggering dynamic provisioning. Mounted the PVC into a Pod (`bookstore`) at `/data`, wrote a file, then deleted and recreated the Pod to prove the data survives.

## Findings

**PV and PVC binding**
Both `bookstore-pv` and `bookstore-pvc` showed status `Bound`, with the PVC's `VOLUME` field pointing to `bookstore-pv` and the PV's `CLAIM` field pointing to `default/bookstore-pvc`.

**Why did the file survive after the Pod was deleted?**
Unlike `emptyDir`, a PersistentVolume exists independently of any Pod's lifecycle. The `bookstore-pvc` claim stays bound to `bookstore-pv` regardless of which Pod (or whether any Pod) is using it. When `bookstore` was deleted, only the Pod object was removed — the PVC and its underlying PV (and the data written to the hostPath directory backing it) were untouched. When the Pod was recreated referencing the same PVC, it was reattached to the exact same underlying storage, so `storage-test.txt` was still there.
