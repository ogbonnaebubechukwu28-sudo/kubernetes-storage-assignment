# Final Challenge - Complete Storage Demo

## What I did
Combined everything from the earlier tasks into one self-contained demo: created a PersistentVolume (final-pv, 1Gi, RWO, Retain, hostPath-backed), a PersistentVolumeClaim (final-pvc) statically bound to it, and a Pod (final-bookstore) mounting the PVC at /data. Wrote a file, deleted the Pod, recreated it, and confirmed the file was still present.

## Architecture

Kubernetes Cluster
  down to
Pod (final-bookstore)
  down to
PVC (final-pvc)
  down to
PV (final-pv)
  down to
Storage Backend (hostPath at /data/final-project on the node)

## Result
After writing final-test.txt to /data, deleting the Pod, and recreating it from the same pod.yaml, running cat /data/final-test.txt on the new Pod still returned "Final Challenge Data". This confirms the PVC/PV pairing is fully decoupled from the Pod's lifecycle - the Pod is disposable, but the storage and its contents persist independently, exactly as a PersistentVolume is designed to behave.
