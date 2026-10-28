# Limits

What K3s cannot do, what it should not
be asked to do, and what its light
weight honestly costs — the chapter the
rest of the guide has been promising.

## The small datastore has a ceiling

A lone server remembers the cluster in
one SQLite file, written through one
process. For a home or small-team
cluster — dozens of workloads, a handful
of nodes — that is not a compromise at
all; it is the reason the whole thing
fits in a file and backs up with a
copy. But SQLite is a single-writer
database, and past some size of cluster
and rate of change it becomes the
bottleneck everything queues behind.
The exits are documented and real —
embedded etcd across three servers, or
an external database — but they are
exits *from the default*, taken
deliberately, usually at birth. A
cluster expected to grow into hundreds
of nodes and constant churn should
start life as one of chapter 14's other
houses.

## One server is one point of failure

The single-node cluster this guide
builds first is exactly as available as
its machine. When the server stops —
hardware, power, a failed upgrade — the
API is gone: nothing can be changed,
nothing new can start, and chapter 9's
stranded volumes are stranded in both
senses. Running workloads usually run
on, which softens the blow and sharpens
the point: the failure you notice is
losing the *brain*. High availability
(chapter 10) is the genuine fix and it
costs etcd's care forever; backups
(chapter 12) are the honest alternative
for clusters whose downtime is an
inconvenience rather than a loss. What
is not available is pretending a single
server is highly available because the
software is good.

## The defaults are opinions

Traefik, flannel, local-path storage,
ServiceLB, CoreDNS: each is a sensible
default, none is the only choice, and
each will eventually meet the reader it
doesn't fit — a network plugin with
policies flannel doesn't express, a
second ingress with different habits, a
workload whose data must float between
nodes (chapter 9). K3s lets you replace
every one of them (chapter 11), at
which point you are assembling more of
the cluster yourself and the
one-binary simplicity is partly spent.
That is not a flaw; it is the honest
exchange the whole design makes, and it
is better entered knowingly.

## Smaller print, plainly

- **Rootless mode is experimental.**
  The normal install runs as root and
  manages the host's networking; readers
  for whom that is unacceptable should
  know the rootless alternative is
  labelled a work in progress by the
  project itself.
- **Windows nodes are a narrower
  road.** Windows machines can join as
  agents for Windows containers; the
  server is Linux, several features do
  not cross, and the documentation's
  caveats are longer there than anywhere
  else in it.
- **Air-gap works, but it is luggage.**
  Chapter 13's offline recipe is real
  and used — and every image, including
  your own workloads' updates, travels
  by hand forever after.
- **Certificates and versions move.**
  The cluster's certificates have
  lifetimes (chapter 12), Kubernetes
  releases arrive quarterly, and a
  cluster left entirely alone for years
  will eventually need a chapter of
  archaeology. Small is not the same as
  maintenance-free.
- **It will not make bad workloads
  good.** Kubernetes restarts what
  crashes; it does not fix why. A
  program that leaks memory will leak it
  on a schedule.

## The standing limit: whose machines

Chapter 1's rule — *your clusters, your
machines, or explicit permission* — has
a cluster-shaped edge worth naming. The
tooling makes it technically trivial to
enrol machines you merely have access
to, to run heavy workloads on shared
hardware, and to leave agents behind on
systems their owners forgot you touched.
Each of those is the same act at
different scales: spending someone
else's property as if it were yours. In
many workplaces and institutions that
is a sackable offence quite apart from
the law of computer misuse, and in some
of its forms it is a crime with a
modern name — cryptojacking is only the
gaudiest. Everything inside the rule
encloses years of interesting work on
hardware that costs less than a night
out. There is no need, and no excuse,
to borrow anyone else's.
