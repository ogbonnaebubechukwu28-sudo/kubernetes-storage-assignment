# Task 8 — Reclaim Policy

## What I did
Created a PV (`reclaim-demo-pv`, Retain policy) and a PVC (`reclaim-demo-pvc`) that bound to it, then deleted the PVC and observed what happened to the PV.

## Findings

**What happened when the PVC was deleted?**
The PV was not deleted. It transitioned to status `Released`, and its `CLAIM` field still referenced the deleted PVC (`default/reclaim-demo-pvc`). The underlying storage and any data in it remained untouched — the PV just isn't bound to a claim anymore, and Kubernetes does not automatically make it available for a new PVC to bind to.

## Retain vs Delete vs Recycle

- **Retain** — the PV and its data are kept when the PVC is deleted. It moves to `Released` status, and needs manual intervention (deleting/reclaiming the PV) before reuse.
- **Delete** — the PV and its underlying storage are automatically deleted along with the PVC. This is the default for most dynamically-provisioned volumes.
- **Recycle** — deprecated; it used to wipe the volume's contents and make the PV available again, but was replaced by dynamic provisioning since it was unsafe and inflexible.

## Why is Retain potentially useful for important data?
It acts as a safety net against accidental data loss. If a PVC is deleted unintentionally (bad command, failed deployment rollback, etc.), the underlying data isn't destroyed — it stays intact until an administrator deliberately decides what to do with it. This matters a lot for critical persistent data like databases.
