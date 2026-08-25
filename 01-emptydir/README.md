# Task 1 — emptyDir

## What I did
Created a Pod (`emptydir-demo`) using an `emptyDir` volume mounted at `/data`. Wrote a file (`test.txt`) into `/data`, verified it, then deleted and recreated the Pod to test whether the file survived.

## Findings

**What is emptyDir?**
A volume that's created empty when a Pod is scheduled to a node, and exists only for the lifetime of that Pod. It's backed by the node's local disk (or memory, if configured) and lives alongside the Pod, not independently of it.

**When is it useful?**
For temporary scratch space a container needs during its lifetime — caching, temp files, sharing data between containers in the same Pod, or working space for a batch job. It doesn't need to survive Pod restarts.

**When is it inappropriate?**
For anything that must survive a Pod being deleted or rescheduled — application data, database files, user uploads, logs that need to be kept. It offers no durability at all.

**What happened to the file after the Pod was deleted?**
It was permanently lost. When the Pod was deleted, Kubernetes wiped the emptyDir volume's contents from the node's disk along with it. When the new Pod was created — even from the same manifest and with the same name — it got a brand-new, empty emptyDir volume with no link to the old one. This was confirmed when `cat /data/test.txt` on the recreated Pod returned "No such file or directory".
