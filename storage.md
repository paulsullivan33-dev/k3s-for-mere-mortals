# Storage

Containers are amnesiacs: a pod's own
filesystem vanishes when the pod does.
Programs that keep anything — a
database, uploaded photos, a game's
world — need **volumes**: storage that
outlives the pod and is handed to
whichever copy replaces it. This
chapter covers the storage K3s brings
in the box, what it honestly is (a
folder on one node), and the moment to
know you've outgrown it.

## Claims and volumes

Kubernetes separates asking from
providing:

- A **PersistentVolumeClaim** (a claim)
  is a workload saying: *I need a volume,
  about this big, that I can write to.*
- A **PersistentVolume** is the storage
  itself, provided by something that
  knows how.
- A **StorageClass** is the menu the
  provider offers — the rule by which a
  claim becomes a volume automatically.

A pod names a claim in its manifest the
way chapter 7's named labels, the claim
binds to a volume, and the program sees
an ordinary directory that happens to
survive it. Delete the pod, the
directory persists; delete the claim,
though, and the class's reclaim rule
decides the folder's fate — the default
`local-path` class deletes it with the
claim, so treat deleting a claim as
deleting the data unless you defined a
class of your own that retains it.

## What K3s provides: local-path

Out of the box, the provider is the
**local-path-provisioner** and the
default class is named `local-path`.
When a claim arrives, it makes a real
directory on a node — under the data
directory's `storage/` folder — and
that directory is the volume. A pod
with a claim therefore looks like:

```yaml
    spec:
      containers:
        - name: app
          image: your-image
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: app-data
```

with the claim itself a small manifest
of its own (`kind: PersistentVolumeClaim`,
naming the class `local-path` and a
size). Nothing to install, nothing to
format: chapter 4's cluster stores data
from its first day.

## The honest catch

The name is the warning: **local**. The
volume is a folder on *one* node, and
the cluster knows it — a pod using it
can only ever run on that node, because
that is where its data is. On a
one-machine cluster this costs nothing
and buys simplicity and speed (it is
just a directory; backups are file
copies, chapter 12). On a growing
cluster it quietly decides your
architecture for you:

- A node dies, and every workload whose
  volume lived there is down until the
  node returns — the data is not lost,
  it is *stranded*.
- Scaling a workload to a second node
  does not give the second copy the
  data; there is one folder, on one
  machine.

That is the right trade for a great
deal of real work — a home cluster's
services usually live happily on the
node they were born on — and the wrong
one the day a workload must float
freely. Chapter 15 counts it among the
limits. The exits, when that day comes,
are network storage (an NFS share is
the homeliest — the series' *Samba and
NFS for Mere Mortals* guide covers
providing one) and distributed storage
systems that run *on* the cluster as
workloads, at the price of their own
care and feeding.

## Manners that apply everywhere

- **Size is a request, not a wall** with
  local-path: the folder grows as the
  program writes, whatever the claim
  said. Watch the node's disk, not the
  claim.
- **Back up the folder, not the idea.**
  Chapter 12's backup story covers the
  datastore; your volumes are separate
  citizens, and a cluster backup that
  forgets them remembers everything
  except the data.
- **Names in manifests, not in code.**
  Programs should be told *a path*
  (`/data`), never which node or folder
  stands behind it — the day you change
  providers, only the claim changes.

Check: storage here is understood when
"the pod moved but its files didn't" and
"the pod can't move because its files
can't" both read as the same sentence —
the volume is a place, and places are
on machines.
