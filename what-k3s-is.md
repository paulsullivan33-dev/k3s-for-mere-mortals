# What K3s Is

A program that must never stop — a
website, a home automation brain, a file
service — is usually run as a
**container**: the program and
everything it needs, packed into one
box that any suitable machine can run.
One container on one machine is easy.
Fifty containers on five machines, kept
alive, restarted when they crash, moved
when a machine goes down for repairs —
that is a job in itself, and it is the
job **Kubernetes** does. You describe
what should be running; Kubernetes
makes it so, and keeps making it so.

Stock Kubernetes is not one program. It
is a control plane of several programs
(an API server, a scheduler, controllers,
a datastore called etcd), a worker
program on every machine, a container
runtime underneath, a network layer, and
a command-line tool — each installed and
kept in step. For a platform team that
is normal. For one person with three
small machines it is a weekend of
plumbing before the first useful thing
runs.

**K3s is the whole of that, repackaged
by Rancher (now part of SUSE) into a
single binary file.** One download —
this guide's lab measured the current
one at about 78 megabytes — contains
the server, the agent, the runtime, the
networking and the tools. It is a real,
certified Kubernetes distribution: the
same API, the same `kubectl` commands,
the same manifest files. Nothing about
what you learn here is a dialect. What
changed is the packaging and a handful
of part swaps:

- **The datastore.** Stock Kubernetes
  remembers everything in etcd, a
  separate distributed database. K3s,
  serving alone, remembers it in a
  **SQLite** file — the same small
  database that lives inside countless
  apps — reached through a translator
  the project calls Kine. Chapter 2 has
  the details, and chapter 15 the
  ceiling this puts on growth.
- **The runtime.** K3s bundles
  **containerd**, the industry-standard
  container runtime. There is no Docker
  to install and nothing to keep in
  step.
- **The extras.** A working cluster
  needs more than Kubernetes proper:
  DNS inside the cluster, a way in for
  web traffic, somewhere for data to
  live. K3s includes working defaults
  for all of them (CoreDNS, the Traefik
  ingress controller, a simple storage
  provider and more) so a fresh cluster
  is useful in minutes.
- **The leftovers.** Code for cloud
  platforms and hardware that a small
  cluster will never meet is left out.
  That absence is most of the weight
  saving.

A first taste, after downloading the
binary (chapter 3 shows the checked
route): the one file is the server, the
worker and the toolbox, depending on the
word you give it first — the lab's
transcript `k3s-lab/transcripts/t01-help.txt`
is the whole menu:

```text
$ k3s --help
COMMANDS:
   server           Run management server
   agent            Run node agent
   kubectl          Run kubectl
   crictl           Run crictl
   ctr              Run ctr
   check-config     Run config check
   token            Manage tokens
   etcd-snapshot    Manage etcd snapshots
   secrets-encrypt  Control secrets encryption and keys rotation
   certificate      Manage K3s certificates
   completion       Install shell completion script
```

Check: the same binary reports its
version, and the Kubernetes inside it,
as transcript `k3s-lab/transcripts/t00-version.txt`
shows:

```text
$ k3s --version
k3s version v1.37.1+k3s1 (356c0254)

$ k3s kubectl version --client
Client Version: v1.37.1+k3s1
```

Read the version as *Kubernetes 1.37.1,
K3s build 1*: K3s versions track the
Kubernetes they contain, so everything
written for Kubernetes 1.37 applies.

## The rule, stated before everything else

A cluster is machinery for running
programs on machines — including, if you
point it that way, other people's
machines and other people's programs. So
this guide has one rule, and it is not
fine print:

> **Build and administer only clusters
> that are yours — on machines you own,
> or where every party involved has given
> you explicit permission.**

Your spare PC, your Raspberry Pis, a
server you rent and control: yes.
Joining a machine you merely have an
account on to a cluster you run, running
workloads on an employer's or a school's
hardware because you can, poking at a
cluster someone else operates because
its address was written down somewhere:
no. The lab in this guide went further
than most readers will need to: it ran
everything inside one restricted
sandbox, on no network but its own. If a
future idea you have for this tool does
not fit inside the rule, the idea is the
problem. Chapter 15 returns to the edge.

## Why anyone would do this

- **A home lab that behaves like the
  real thing.** Learn and break genuine
  Kubernetes — not a simulator — on
  hardware that sips power.
- **Small servers that must look after
  themselves.** A handful of services on
  one modest machine, restarted
  automatically when they fall over,
  updated by describing the new version.
- **Edge and workshop machines.** Shops,
  boats, labs and field kits run K3s
  because the whole platform is one file
  that works the same on a server and on
  a Raspberry Pi — including, with
  preparation, entirely offline
  (chapter 13).
- **A stepping stone.** Everything here
  — manifests, services, ingress — is
  portable to a full-size or cloud
  cluster unchanged. Small is a starting
  size, not a different subject.

## Who it is for

The person with a spare machine and a
curiosity about how the big platforms
work. The home-lab keeper whose services
have outgrown a pile of hand-started
containers. The maker who wants a
Raspberry Pi cluster that is a real
cluster. The common thread is modest
hardware, one administrator, and wanting
the genuine article at a size one person
can hold.

## What it is not

K3s is not a container tool on its own:
if you want to run one program on one
machine and think no further, a plain
container runtime is simpler, and
chapter 14 says so. It is not a managed
service: somebody still patches,
upgrades and backs up the cluster, and
that somebody is you (chapters 12
and 15). It is not a miniature in the
sense of missing pieces: the API is
complete, but the *defaults* are chosen
for small clusters, and at large scale
those choices — one small database, one
server — are the limits chapter 15
names. And it is not five nines in a
trench coat: a single K3s server on a
single PC is exactly as reliable as
that PC, which for learning and for
home services is usually the right
trade, honestly made.
