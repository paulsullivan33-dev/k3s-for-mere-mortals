# The Bundled Tools

One binary arrives; five tools move in.
This chapter is a tour of the hats the
`k3s` command wears — the tour is short
because the guide's lab could exercise
most of it with no cluster at all, and
did. Knowing which hat does what keeps
you from reaching for a download you
already own.

## kubectl: the remote control

`k3s kubectl` is the genuine kubectl,
the standard Kubernetes command tool,
compiled into the same file. Everything
written for kubectl — every tutorial,
every cheat sheet — works after those
two words. Its client half needs no
cluster, and the lab used it to do real
work: asking the binary to *describe* a
workload is a local act, and
`--dry-run=client -o yaml` prints the
exact manifest it would have sent
(transcript
`k3s-lab/transcripts/t02-dry-run.txt`):

```bash
$ k3s kubectl create deployment hello --image=nginx:alpine --replicas=2 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
…
```

Chapter 7 builds the guide's example
manifest with this trick. The same
transcript file's sibling (t03) is the
honest boundary: commands that start
from a manifest *file* ask the cluster
what the file's kinds mean, so with no
cluster they stop at "connection
refused" — kubectl looking for a server
at the default address and finding
none. That failure is information, not
breakage.

Offline, kubectl will also tell you its
own version (`version --client`, t00),
explain its commands (`--help` after any
subcommand), and complete your shell
(the binary's `completion` command
installs that). Everything else it does
is a conversation with a cluster.

## ctr and crictl: the runtime's windows

Two hats look not at the cluster but at
the container runtime on one node:

- **`k3s ctr`** is containerd's own
  command tool: images, containers and
  tasks, raw and low-level. Its help —
  which the lab ran — opens with the
  containerd banner and a long command
  list. You will mostly meet it in
  troubleshooting advice ("list the
  images the node actually holds").
- **`k3s crictl`** speaks the standard
  runtime interface (CRI) that the
  kubelet uses: `crictl ps` shows the
  containers on this node as the kubelet
  sees them, `crictl images` their
  images, `crictl logs` a container's
  output when a pod is too broken for
  kubectl to describe. The lab's copy
  reported itself as `v1.37.0-k3s1`.

Both need the node's runtime socket, so
they are run as root, on the node, while
it runs — on a live cluster they are
chapter 16's flashlights. A habit worth
forming early: reach for kubectl first
(the cluster's view), and for these only
when the question is about one machine.

## check-config: the physical

Chapter 3 introduced `k3s check-config`,
the kernel-and-hardware examination,
and the lab's transcript t04 is its full
report on an unfit machine. Two more
things are worth knowing. It runs
anywhere, cluster or no cluster — the
lab ran it in a sandbox that could never
host the server it was checking for. And
it is the cheapest first step in half of
chapter 16: a server that will not start
on a strange machine is very often a
machine that would have failed this
check, in writing, in advance.

## token: the joining secret

`k3s token` manages the **tokens** that
prove a new node may join (chapter 10).
Most of its subcommands — `create`,
`list`, `delete`, `rotate` — operate on
a running server. One works with no
server at all, and the lab ran it
(transcript t06):

```bash
$ k3s token generate
7cx9eb.u71f7j8cojx5pwtx
```

`generate` prints a fresh bootstrap
token without recording it anywhere —
useful when a joining script wants a
token minted on the spot. The cluster's
own long-lived joining secret is a
different thing: the server generates it
at first start and keeps it in the data
directory (`server/node-token`), and
chapter 10 treats it with the respect a
master key deserves.

## The remaining two

- **`k3s etcd-snapshot`** manages
  backups of the datastore on clusters
  that use embedded etcd: save, list,
  restore, prune. Chapter 12 is its
  home; it matters exactly when the
  cluster's memory does.
- **`k3s secrets-encrypt`** turns on and
  rotates encryption of the cluster's
  stored secrets, and **`k3s certificate`**
  manages the cluster's certificates —
  both chapter 12 material, both working
  against a running server.

Check: the tour is complete when the
shape feels plain — *kubectl talks to
the cluster, ctr and crictl talk to one
node's runtime, check-config talks about
the machine, and token, etcd-snapshot
and secrets-encrypt look after the
cluster's keys and memory.* One binary;
you already have all of it.
