# The Moving Parts

By the end of this chapter you can name
every piece inside the one binary, say
which machine each piece lives on, and
point at the folder where the cluster
keeps its memory. When something later
misbehaves, that map is what turns "the
cluster is broken" into a named problem.

## Two kinds of machine

A K3s cluster is made of **nodes** —
machines (or virtual machines) that have
joined it. Nodes come in two kinds:

- A **server** node runs the **control
  plane**: the parts that decide what
  should run where and remember what is
  true. A cluster has one server, or —
  for high availability (chapter 10) —
  three or five.
- An **agent** node runs the workloads:
  the containers themselves. Every
  server also has an agent inside it, so
  a single machine is a complete cluster
  of one, control plane and workloads
  together. That is why one PC is enough
  for most of this guide.

The words come from the binary itself:
`k3s server` and `k3s agent` are the
first two commands in the help
(transcript t01).

## Inside the server

Stock Kubernetes spreads the control
plane across several programs. In K3s
they run as parts of the one process:

- The **API server** is the front door.
  Every command you give and every
  report the nodes send goes through it,
  on port 6443. Nothing talks to the
  cluster except through it, which is
  why the kubeconfig of chapter 4 — the
  key to that door — matters.
- The **scheduler** decides which node
  each new workload lands on.
- The **controllers** watch the world
  and nudge it toward what you
  described: a container died, so start
  another; you asked for three copies,
  so there shall be three copies.
- The **datastore** is where the truth
  lives: every description, every
  secret, every state. Lose it and the
  cluster forgets everything (hence
  chapter 12's backups). Alone, a K3s
  server keeps the truth in **SQLite**,
  spoken to through **Kine**, a
  translator that lets the control plane
  believe it is talking to etcd. The
  lab's startup log shows both in its
  first lines (transcript
  `k3s-lab/transcripts/t05-server-attempt.txt`):

```text
level=info msg="Configuring sqlite3 database connection pooling…"
level=info msg="Kine available at unix://kine.sock"
level=info msg="Datastore connection validated successfully, proceeding with bootstrap data generation"
```

  With several servers, K3s can instead
  run a real **embedded etcd** spread
  across them (`--cluster-init`, chapter
  10), or use an outside database. The
  choice is made when the cluster is
  born and shapes chapter 15's limits.

## Inside every node

Server and agent alike carry:

- **containerd**, the runtime that
  actually starts and stops containers.
  K3s bundles it; you will rarely speak
  to it directly, but chapter 5's `ctr`
  and `crictl` are its two windows.
- The **kubelet**, the node's foreman:
  it receives orders from the control
  plane ("run this, with these settings")
  and reports back ("running; restarted
  twice; out of memory").
- **Flannel**, the network layer: it
  gives every container (**pod**) its
  own address on a cluster-wide private
  network (by default 10.42.0.0/16, from
  the server's own help text) and
  tunnels traffic between nodes, by
  default inside VXLAN packets on UDP
  port 8472.
- A small proxy (kube-proxy, folded into
  the same process in K3s) that makes
  **services** — stable addresses for
  shifting containers — work.

## The extras in the box

A bare Kubernetes can run containers but
cannot yet be reached from outside, give
them DNS names, or store files for them.
K3s starts these helpers for you, as
ordinary workloads in a corner of the
cluster reserved for the system
(`kube-system`):

- **CoreDNS** — name lookup inside the
  cluster, so programs find services by
  name.
- **Traefik** — the **ingress
  controller**: it takes web requests
  arriving at the machine and routes
  them by name and path to the right
  service (chapter 8).
- **ServiceLB** (its programs are called
  Klipper) — the trick that lets a
  service of type LoadBalancer work
  without a cloud: it claims an address
  of the node itself (chapter 8).
- **local-path-provisioner** — the
  default storage: a folder on a node,
  handed out when a workload claims a
  volume (chapter 9).
- **metrics-server** — the numbers
  behind `kubectl top`, the cluster's
  speedometer.

Each is a default, not a fixture:
chapter 11 shows how to switch any of
them off (`--disable traefik` and
friends) when you would rather bring
your own.

## Where it all lives

Nearly everything K3s owns sits in one
**data directory** — the help text names
it, and the lab confirmed it is where
the binary unpacks and works:

```text
--data-dir value, -d value  (data) Folder to hold state default /var/lib/rancher/k3s or ${HOME}/.rancher/k3s if not root [$K3S_DATA_DIR]
```

Inside it: the server's database and
certificates (`server/`), the agent's
state and container images (`agent/`),
and a `data/` folder holding the helper
programs the project bundles, unpacked
on first run — the configuration check
in chapter 3 catches it in the act:

```text
Verifying binaries in /var/lib/rancher/k3s/data/18fd43…/bin:
- sha256sum: good
- links: good
```

Two other addresses complete the map:
**`/etc/rancher/k3s/`**, where the
configuration file and the kubeconfig
live, and **`/var/lib/rancher/k3s/server/manifests/`**,
the auto-deploy folder of chapter 7 —
anything written there is applied to
the cluster by the server itself.

Check: with the chapter done, these
sentences should feel plain — *the
server decides and remembers, the agent
runs, containerd does the running,
flannel does the connecting, and the
data directory holds the memory.* The
rest of the guide is detail hung on
that frame.
