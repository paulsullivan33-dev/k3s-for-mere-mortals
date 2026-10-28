# Upgrades, Backups and Removal

The cluster is running and holding
things. This chapter is the adult
supervision: moving it to a new version
without drama, keeping a copy of its
memory against the day the machine
dies, encrypting what it stores, and —
when the day comes — taking it off a
machine completely. Procedures here are
the project's documented ones; the
guide's lab, which never had a running
cluster, verified the tools' existence
and shapes (chapter 5) and no more, and
says so.

## Upgrading

A K3s upgrade is an install repeated:
the binary is replaced, the service
restarts, the data directory — database,
certificates, images — is untouched and
read back by the new version. By the
script, naming the target release:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.37.1+k3s1 sh -
```

or manually, by swapping the binary for
the new one (checked against its
checksum, chapter 3's habit) and
restarting the service. Order matters
in a multi-machine cluster: **servers
first, one at a time, then agents** —
the new brain must be able to speak to
the old workers, which it can, across
one version step. The project's own
rule of thumb is to walk release by
release rather than leapfrogging
several, and to read the upgrade notes
for anything your cluster uses. For
readers who grow tired of doing this by
hand on many machines, the project's
system-upgrade-controller automates the
same dance *from inside* the cluster —
a manifest in chapter 7's sense, and a
good first "real" workload to adopt.

Check, after every upgrade: `k3s
--version` and `kubectl get nodes` agree
on the new version everywhere, and the
system pods of chapter 4 are Running
before you call it done.

## Backing up the memory

Everything the cluster knows is in the
datastore, so a backup is a copy of the
datastore — plus the joining token and
the server's configuration, which
together let a rebuilt machine *be*
this cluster again. What "copy" means
depends on the datastore you chose at
birth (chapter 2):

- **Embedded etcd** (several servers):
  the binary's own tool,
  **`k3s etcd-snapshot`**, saves point-in-time
  snapshots — the server takes them on a
  schedule by itself once told, and
  `save`, `list` and `restore` are its
  verbs. A restore is a new cluster
  raised from the snapshot with
  `--cluster-reset`. Snapshots live in
  the data directory (`server/db/snapshots`)
  and a backup that lives only on the
  machine it protects is half a backup:
  copy them elsewhere.
- **SQLite** (the single-server
  default): the datastore is a database
  file under the data directory's
  `server/db/`. The documented route is
  to stop the server and copy that
  folder — or the whole data directory —
  somewhere safe. Restoring is putting
  it back and starting. Simple, because
  the design is simple; the price is the
  stop.

And the sentence that saves somebody a
weekend, every time it is printed:
**the datastore is not the data.**
Chapter 9's volumes — the folders of
local-path storage — are separate
citizens. A backup routine for a real
cluster copies both, or it has backed
up a perfect description of services
whose contents are gone.

## Encrypting the stored secrets

Kubernetes Secrets are, by default,
only encoded, not encrypted, inside the
datastore — anyone who can read the
database can read them. K3s can do
better, with the tool from chapter 5:

```bash
k3s secrets-encrypt enable
k3s secrets-encrypt status
k3s secrets-encrypt rotate
```

Enabling turns on encryption at rest on
a running server; existing secrets are
rewritten as they are touched (the
project documents a `reencrypt` step
that does them all); `rotate` changes
the key on a schedule you choose. On a
home cluster this is defence in depth
for the database file's travels — it
makes your *backups* safe to keep,
which is where it earns its keep. It
does not change who may ask the API for
a secret: that remains the kubeconfig's
and the cluster's permission system to
decide.

## Certificates, briefly

Chapter 4's certificates are minted for
a year at a time, and K3s renews the
ones it manages as they approach
expiry — a server restarted in time
rolls them forward. A cluster left
stopped for a very long season can wake
to expired credentials; **`k3s
certificate`** is the managing tool,
and the project's documentation walks
the renewal (and the deliberate
rotation of the whole certificate
authority, a rarer operation with its
own ceremony). Chapter 16 lists the
symptom.

## Removal

Uninstalling is a script, because the
installer wrote one for exactly this:

```bash
# On a server:
/usr/local/bin/k3s-uninstall.sh

# On an agent:
/usr/local/bin/k3s-agent-uninstall.sh
```

It stops the service, removes the
binary, the service definition, the
data directory and the networking K3s
wove — a genuinely complete goodbye,
which is why it is also destructive:
the datastore and the local-path
volumes go with it. Drain first
(chapter 10) if the machine's workloads
should live on elsewhere; back up first
(chapter 12, this one) if the cluster's
memory should outlive it. A manual
install is undone by the same script if
the script route was ever used, or by
hand in the same order: stop the
process, delete the data directory,
delete the binary, delete
`/etc/rancher/k3s`.

Check: this chapter is understood when
"upgrade" reads as *replace the binary,
keep the memory*, "backup" as *copy the
memory — and the data*, and "remove" as
*a script, run deliberately, after both
of the others have been considered.*
