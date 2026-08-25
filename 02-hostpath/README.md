# Task 2 — hostPath

## What I did
Created a Pod (`hostpath-demo`) using a `hostPath` volume mounted at `/data`, backed by `/data/bookstore` on the Minikube node. Wrote a file (`book.txt`) into `/data` from inside the Pod, then verified it directly from the node itself using `minikube ssh`.

## Findings

**What is hostPath?**
A volume type that mounts a file or directory directly from the host node's filesystem into the Pod.

**Where is the data actually stored?**
On the Minikube node's own filesystem, at `/data/bookstore` — confirmed by running `minikube ssh` and finding `book.txt` sitting there directly, outside any container.

**What happens if the Pod moves to another node?**
The data doesn't move with it. Since hostPath is tied to one specific node's disk, a Pod rescheduled to a different node would find an empty (or nonexistent) directory, not the original data.

**Why can hostPath be problematic in multi-node production clusters?**
Data isn't shared or replicated across nodes, so if a Pod is rescheduled elsewhere (which Kubernetes does routinely for load-balancing, node failures, or upgrades), the application loses access to its previous data. It also creates a hidden dependency between a Pod and a specific physical node, undermining Kubernetes' node-agnostic scheduling model.

**One situation where hostPath might still be useful:**
Node-level monitoring/logging agents that specifically need to read files that exist on that particular node's filesystem (e.g. `/var/log`), or single-node dev/test setups like this Minikube exercise where there's only one node anyway.
