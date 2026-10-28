# K3s for Mere Mortals

**Kubernetes** is the system that runs
containers across a group of machines as
if they were one: you describe the
programs you want running, and it keeps
them running, restarts them when they
fall over, and spreads them over the
machines you give it. It is also famously
heavy — many separate programs to
install, configure and keep in step.
**K3s** is Kubernetes rebuilt by Rancher
(now part of SUSE) to arrive as **one
file**: a single binary, small enough to
carry on a USB stick, that contains the
whole control plane, the container
runtime, the networking, and the command
line tools, and that swaps the heavy
parts for lighter ones where most small
clusters will never notice.

This guide explains what was lightened
and what it cost, how to install that
one binary and grow a cluster from one
machine to several, how to run a real
workload on it and reach it from your
network, where your data lives, and how
to configure, upgrade, back up and
remove the whole thing. The limits are
stated plainly, including the honest
one: a small cluster is still a cluster,
and it still needs looking after.

The rule that governs everything here is
the series rule, stated in chapter 1:
**your clusters, your machines, or
clusters you have explicit permission to
administer.** A cluster tool makes it
easy to run programs on machines; the
rule is about whose machines they are.
The lab obeyed it strictly — everything
it ran, it ran inside its own sandbox.

The lab behind this guide ran the real
binary: **K3s v1.37.1+k3s1**, the
project's own download for 64-bit Intel
and AMD machines, checked against the
release's published checksum before it
was run at all. Against it the lab read
the whole command surface, ran the
built-in configuration check the project
ships for exactly this purpose, built
Kubernetes manifests with the bundled
kubectl's client-side tools, minted a
join token with no cluster in sight,
proved that the configuration file is
read at startup (and that a misspelt key
in it earns a warning, not a failure),
and captured the server's startup log
line by line as it built its database
and minted its certificates.

What the lab could not do, it did not
fake. Its sandbox is a restricted
container: the kernel gives it no
control over CPU or memory groups and
several network pieces are missing
entirely — the project's own check
counts thirteen failures — so the K3s
server, after a healthy start, stopped
at the wall this guide's final chapters
describe, with the error captured word
for word. Starting the server for real,
watching a node report Ready, and
everything that happens on a running
cluster are therefore taught from the
project's documentation and labelled as
such wherever they appear.
`criteria.md` keeps the full ledger of
what was run and what was only read.

## What this guide covers

- [**What K3s is**](what-k3s-is.md) —
  the idea: all of Kubernetes in one
  binary, what was swapped to make it
  small, and the rule about whose
  machines a cluster may touch
- [**The moving parts**](the-moving-parts.md) —
  server and agent nodes, the datastore
  that remembers everything, containerd,
  flannel, and the useful extras in the
  box
- [**Installing K3s**](installing-k3s.md) —
  the install script, the manual binary
  route this guide's lab verified,
  requirements, and the check that tells
  you whether your machine is ready
- [**Your first cluster**](your-first-cluster.md) —
  starting the server, the kubeconfig
  that proves you own it, and reading
  your first node
- [**The bundled tools**](the-bundled-tools.md) —
  kubectl, crictl, ctr, check-config and
  token: the whole toolkit is the one
  binary wearing different hats
- [**Running a workload**](running-a-workload.md) —
  a deployment and a service, created,
  watched, scaled and removed
- [**Manifests and auto-deploy**](manifests-and-auto-deploy.md) —
  writing YAML that describes a workload,
  and the folder where dropping a file
  is the whole deployment
- [**Ingress and load balancers**](ingress-and-services.md) —
  Traefik for routes in, and ServiceLB,
  the trick that gives load-balancer
  services an address on your own
  machines
- [**Storage**](storage.md) — the
  built-in local storage, what a volume
  claim gets you, and the catch: the
  data lives on one node
- [**Adding nodes**](adding-nodes.md) —
  the token that joins an agent, the
  ports that must be open, and growing
  to more than one server
- [**Configuration**](configuration.md) —
  the config file, the flags that matter
  most, and turning bundled pieces off
- [**Upgrades, backups and removal**](upgrades-backups-and-removal.md) —
  moving to a new version, backing up
  the datastore, encrypting secrets, and
  the uninstall scripts that take it all
  away
- [**Recipes**](recipes.md) — a cluster
  on a spare machine, a Raspberry Pi as
  an agent, an offline install prepared
  in advance, and a service the whole
  house can reach
- [**The neighbours**](the-neighbours.md) —
  full-size Kubernetes, kind, minikube,
  MicroK8s and k0s, and when each is the
  better tool
- [**Limits**](limits.md) — the small
  datastore's ceiling, the single point
  of failure, bundled opinions, and what
  light weight cannot do
- [**Troubleshooting**](troubleshooting.md) —
  the server that will not start, the
  node that never becomes Ready, the
  kubeconfig you cannot read, and the
  sandbox wall this guide's own lab hit
- [**Glossary**](glossary.md) — the
  terms, collected in one place

## What it doesn't cover

Running clusters on machines you do not
own or have no right to administer —
that is not a harder chapter of this
book, and chapter 1 explains the line.
Administering someone else's production
platform, very large clusters (hundreds
of nodes have their own literature, and
chapter 15 explains where this guide's
tool stops being the obvious choice),
Windows worker nodes beyond a mention,
and writing the programs that run *in*
the cluster — this guide operates the
platform; your workloads bring
themselves.

## How to use this guide

Read chapter 1 for the idea and the
rules, and chapter 2 before installing
anything — knowing which part holds what
turns later confusion into a named
problem. Chapter 3 ends with the check
that tells you whether your machine can
host a cluster at all; run it before you
troubleshoot anything else. Chapters 4
to 7 take you from an empty machine to a
workload you can reach, in order. The
chapters after that are each
self-contained: grow the cluster
(chapter 10) when one machine stops
being enough, and read chapter 12 before
the cluster holds anything you would
miss. The lab's evidence lives in
`k3s-lab/transcripts/`; its example
manifest is `k3s-lab/manifests/hello.yaml`
and its command list is
`examples/commands.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Kubernetes for Mere Mortals](https://paulsullivan33-dev.github.io/kubernetes-for-mere-mortals/)
- [Raspberry Pi for Mere Mortals](https://paulsullivan33-dev.github.io/raspberry-pi-for-mere-mortals/)
- [Ansible for Mere Mortals](https://paulsullivan33-dev.github.io/ansible-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/k3s-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/k3s-for-mere-mortals/blob/main/LICENSE-CODE).
