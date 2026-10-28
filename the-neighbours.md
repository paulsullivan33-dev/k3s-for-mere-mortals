# The Neighbours

K3s is one answer to "give me
Kubernetes, small". This chapter walks
the street: what the other answers are,
what each sees that K3s doesn't, and
the honest reasons to live in a
different house. The comparisons are
characterisations, not benchmarks —
every tool named here is a genuine
Kubernetes unless it says otherwise,
and the series' *Kubernetes for Mere
Mortals* guide teaches the shared
concepts underneath all of them.

## Stock Kubernetes, assembled yourself

**What it is:** the upstream pieces —
API server, etcd, kubelet, a runtime, a
network plugin — installed and kept in
step by you, often with the kubeadm
tool doing the heavy lifting.

**What it has that K3s doesn't:** every
knob. Your choice of network layer,
storage, ingress, etcd topology; the
newest features the week they land;
nothing between you and the machinery,
which is also its price. **Choose it
when** the cluster *is* the subject —
you are learning platform engineering,
or your scale and compliance needs have
outgrown defaults. **K3s wins when** the
cluster is the *means*: you wanted your
workloads running some chapters ago.

## kind and minikube: the practice clusters

**What they are:** Kubernetes that lives
inside containers or a small virtual
machine on your own workstation, made
to be created and thrown away. **kind**
(clusters as Docker containers) is the
testing instrument of the Kubernetes
project itself; **minikube** is the
friendlier all-in-one with drivers and
addons for learning.

**What they have that K3s doesn't:**
disposability — a three-node cluster in
a minute, deleted in a second, on the
laptop you already have; and no lasting
state to protect. **Choose them for**
experiments, CI pipelines, and rehearsing
chapter 10's multi-node shapes safely.
**K3s wins when** the cluster should
still be there next month, on its own
hardware, serving something. The two
are complements: many people rehearse
in kind and live in K3s.

## MicroK8s and k0s: the cousins

**MicroK8s** (Canonical) packages
Kubernetes as a snap, with its own
one-word addons and a similar
small-cluster heart; **k0s** is the
other single-binary distribution,
closest in spirit to K3s, with the
control plane and worker as separate
roles of one file.

**The honest difference** is taste and
ecosystem rather than capability: how
each bundles its extras, how upgrades
flow, whose documentation voice you
like. A reader who chose either has not
made an error. K3s's particular claims
are its head start (it has been the
small-cluster default longest), its
presence on everything from Pis to
appliances, and the bundled defaults
this guide walked through — Traefik,
ServiceLB and local-path arriving
together, already agreeing with each
other.

## Managed Kubernetes: someone else's cluster

**What it is:** the cloud providers'
clusters (and their smaller rivals): the
control plane is theirs, the bill is
metered, the load balancers of chapter
8 are real products. **What it has that
K3s doesn't:** the absence of chapter
12 — no servers of yours to patch, back
up or bury; scale that shrugs; and an
engineer on call who is not you.
**Choose it when** the work is public,
the audience real, and your time worth
more than the invoice. **K3s wins**
at home, at the edge, offline, and
wherever the running cost of a small
cluster should be the electricity.

## No cluster at all

The neighbour most often forgotten:
**plain containers**, run by Docker or
Podman on one machine, composed in a
file and restarted by the system. For
one machine and a handful of services
that do not need rescheduling, that is
not a lesser answer — it is the
*correct-sized* one, and this series
has guides to both tools. K3s earns its
keep the day you need a second machine,
self-healing across it, or the
Kubernetes API itself. Choosing small
on purpose is the theme of the whole
street; the houses differ, the virtue
doesn't.
