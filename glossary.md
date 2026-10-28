# Glossary

**Agent** — the worker half of K3s, and
a node that runs only it: receives
workloads from a server and runs them
(chapters 2 and 10).

**API server** — the cluster's front
door: every command and report passes
through it, on port 6443 (chapter 2).

**Auto-deploy folder** —
`/var/lib/rancher/k3s/server/manifests/`:
manifests placed there are applied by
the server itself (chapter 7).

**Claim** — see PersistentVolumeClaim.

**Cluster** — the whole system: its
server(s), its agents, and the truth
they share (chapter 1).

**Config file** — `/etc/rancher/k3s/config.yaml`,
the standing settings a server or agent
reads at startup (chapter 11).

**containerd** — the container runtime
bundled in K3s; it starts and stops the
containers (chapter 2).

**Control plane** — the deciding and
remembering parts — API server,
scheduler, controllers, datastore —
carried by server nodes (chapter 2).

**CoreDNS** — the cluster's internal
name service, bundled (chapter 2).

**crictl** — `k3s crictl`: the tool that
views one node's containers as its
runtime sees them (chapter 5).

**ctr** — `k3s ctr`: containerd's own
low-level command tool (chapter 5).

**Data directory** —
`/var/lib/rancher/k3s` by default: the
database, certificates, images and
unpacked helpers (chapter 2).

**Datastore** — where the cluster's
truth lives: SQLite by default, etcd
when servers multiply (chapters 2
and 10).

**Deployment** — a description that
keeps a stated number of copies of a
container running (chapter 6).

**etcd** — the distributed database
stock Kubernetes uses; in K3s, an
embedded option for multi-server
clusters, and the subject of
`k3s etcd-snapshot` (chapters 10
and 12).

**Flannel** — the bundled network layer
that gives pods addresses and tunnels
between nodes (chapter 2).

**Ingress** — routing rules that hand
web requests arriving at the cluster to
services, by host name and path
(chapter 8).

**Kine** — Rancher's translator that
lets the control plane store its truth
in SQLite (or another SQL database)
while believing it speaks to etcd
(chapter 2).

**Kubeconfig** — the file
(`/etc/rancher/k3s/k3s.yaml` on the
server) holding a cluster's address and
an administrator's credentials: a
master key (chapter 4).

**kubectl** — the Kubernetes command
tool, bundled as `k3s kubectl`
(chapter 5).

**Kubelet** — the foreman on each node:
takes orders from the control plane and
reports back (chapter 2).

**LoadBalancer** — a service type
asking for a proper outside address;
at home, answered by ServiceLB
(chapter 8).

**local-path-provisioner** — the
bundled storage provider: volumes are
directories on one node (chapter 9).

**Manifest** — a YAML file describing
resources the cluster should hold true
(chapter 7).

**Node** — a machine (or VM) that has
joined the cluster, as server or agent
(chapter 2).

**PersistentVolumeClaim** — a
workload's request for storage that
outlives its pods (chapter 9).

**Pod** — the smallest thing the
cluster runs: one or more containers
sharing an address and a fate. Most
workloads run one container per pod
(chapter 6).

**Server** — a node running the control
plane; also an agent for its own
workloads, so one server is a complete
cluster (chapter 2).

**Service** — a stable address and name
in front of a set of pods (chapter 6).

**ServiceLB** — K3s's built-in answer
to LoadBalancer services: it claims an
address of a node itself (the programs
are called Klipper) (chapter 8).

**StorageClass** — the menu rule by
which a claim becomes a volume;
`local-path` is the default class
(chapter 9).

**Token** — the shared secret a node
presents to join the cluster; the
server's lives in `server/node-token`
(chapter 10).

**Traefik** — the bundled ingress
controller: it listens on the web ports
and routes by name (chapter 8).

**VXLAN** — the tunnelling flannel uses
by default to carry pod traffic between
nodes, on UDP port 8472 (chapters 2
and 10).
