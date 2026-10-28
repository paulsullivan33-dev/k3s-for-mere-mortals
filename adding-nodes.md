# Adding Nodes

One machine is a cluster; two is a
*system*. This chapter grows the cluster
outward — an agent joining by token — and
then upward, to more than one server, so
the control plane itself can survive a
machine's loss. The procedures are the
project's documented ones (the lab's
sandbox was a single small room, not a
network), with the parts the lab could
verify — the token's shape, the flags and
their environment variables — verified in
chapter 5 and quoted there.

## How joining works

A cluster needs a way to tell a friend
from a stranger at the door. K3s uses a
shared **token**: the server generates
its token at first start and keeps it in

```text
/var/lib/rancher/k3s/server/node-token
```

A new node presents that token as it
registers. Possession of the token is
the whole credential — it joins machines
to your cluster and, as a bootstrap
secret, belongs in the same mental
drawer as the kubeconfig: read it as
root on the server, carry it to the new
machine by a channel you trust, never
post it in a ticket or a chat log.
(Shorter-lived bootstrap tokens exist
for enrolling machines at scale —
chapter 5's `k3s token` manages them.)

## Joining an agent

On the new machine, install K3s (chapter
3) and start it as an agent instead of
a server. By the script:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<server>:6443 K3S_TOKEN=<the token> sh -
```

or by hand, the same two facts as
ordinary flags (the agent's own help,
checked by the lab, names them:
`--server` [$K3S_URL] and `--token`
[$K3S_TOKEN]):

```bash
sudo k3s agent --server https://<server>:6443 --token <the token>
```

where `<server>` is the first machine's
name or address, reachable from the new
node. Moments later, on the server:

```bash
k3s kubectl get nodes
```

lists both machines, the new one with
`ROLES` reading `<none>` — a pure worker.
From that moment the scheduler counts its
capacity as the cluster's, and chapter
6's deployments may land there. Two
details worth knowing:

- **The name matters.** A node joins
  under its hostname; two machines with
  the same hostname are a classic
  chapter 16 morning. Give machines
  distinct names before they join.
- **Labels are introductions.**
  `--node-label` (in the agent's help,
  and on servers too) attaches facts at
  joining — `disk=ssd`, `room=attic` —
  that scheduling can later ask for.

## The doors between machines

Nodes talk constantly. For joining and
ordinary life, the project's documented
requirements are:

| Port | Between | For |
|---|---|---|
| 6443/TCP | agents (and you) → servers | The API and registration |
| 8472/UDP | every node ↔ every node | Flannel's VXLAN tunnels |
| 10250/TCP | servers → agents | The kubelet's reports and metrics |
| 2379–2380/TCP | servers ↔ servers | Embedded etcd, when there are several servers |
| 80, 443/TCP | your network → any node | Traefik's web doors (chapter 8) |

Firewalls *between* nodes are the usual
growth pain: a node that joins but never
turns Ready, or pods that cannot reach
each other across machines, is very often
8472/UDP quietly dropped (chapter 16).
The cluster's own internal networks —
10.42.0.0/16 for pods, 10.43.0.0/16 for
services, the defaults in the server's
help — must not collide with your real
network's addresses; if they do, chapter
11's `--cluster-cidr` and `--service-cidr`
are set at birth, on the first server,
before anything joins.

## More than one server

An agent's death costs its workloads a
restart elsewhere. A **server's** death
costs the cluster its brain: the API
stops, nothing new can be scheduled,
though running workloads run on. To
remove that single point of failure,
servers come in threes (or fives — the
count of voters must be odd for the
datastore's elections to be decisive):

1. Start the first server with
   `--cluster-init`, which births the
   cluster on **embedded etcd** instead
   of lone SQLite (the flag is in the
   server's help; the datastore then
   lives spread across the servers).
2. Start each further server with the
   same `--server` address and token an
   agent would use — a joining server is
   recognised by asking for server duty.

The cluster now tolerates one server's
loss with the brain intact, at the price
of etcd's care: snapshots (chapter 12)
become a real discipline, and the
servers want to be alike and near each
other. It is the point where chapter
15's limits start charging rent, and it
is genuinely optional: a great many
honest clusters are one server and
several agents, with backups standing in
for high availability (chapter 12).

Removing a node is the reverse
courtesy: `kubectl drain` it (the
cluster moves its workloads off),
`kubectl delete node` its record, then
run its uninstall script on the machine
itself. Chapter 12 covers the mechanics.

Check: growth is understood when the
three sentences separate cleanly — *an
agent joins with an address and a token;
servers multiply by election, so they
come in odd numbers; and every port in
the table is a conversation the cluster
cannot live without.*
