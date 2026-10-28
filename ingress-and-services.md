# Ingress and Load Balancers

A workload nobody can reach is a diary.
This chapter covers the two doors K3s
opens by default: **ServiceLB**, which
gives a service an address on your own
machines, and **Traefik**, the bundled
ingress controller, which routes web
traffic by name. Like chapter 4, the
live behaviour is described from the
project's documentation — the doors are
on the running cluster the sandbox could
not host — while the service half of the
guide's example was built and checked
in chapter 5's transcripts.

## Three kinds of door, quickly

A **Service** (chapter 6) is a stable
address in front of shifting pods. Its
`type` says how wide the door opens:

- **ClusterIP** (the default): reachable
  inside the cluster only. Programs
  talking to programs; the name resolves
  through CoreDNS.
- **NodePort**: the cluster also opens
  one high port (from a range around
  30000) on *every* node, forwarding to
  the service. Plain, dependable, and
  the numbers are ugly.
- **LoadBalancer**: "give this a proper
  outside address." In a cloud, the
  provider sells you one. At home, K3s
  answers the request itself.

## ServiceLB: the home answer

When a LoadBalancer service appears,
**ServiceLB** (chapter 2's Klipper)
claims an address belonging to one of
the nodes and runs a small forwarder
for it, so the service's port answers
on that node's address. Chapter 6's
walkthrough ended exactly there: the
service read back an `EXTERNAL-IP`
equal to a node's own address, and the
port answered on the home network.

Its character, honestly:

- **Zero configuration.** Nothing to
  install, nothing to apply for; the
  first LoadBalancer service you create
  simply works. For a home or lab
  cluster this is the difference between
  an afternoon and a project.
- **The address is a node's.** If that
  node goes down, the address goes with
  it — ServiceLB will move the claim to
  another node, but clients knew the old
  address. It is a doorbell, not a
  floating palace.
- **Ports are the node's ports.** Two
  LoadBalancer services cannot both have
  port 80 on the same node address; the
  second waits, `Pending`, until the
  situation makes sense (chapter 16
  lists the symptom). Web traffic has a
  better answer, next.

## Traefik: names instead of ports

**Ingress** is the web-native door: a
set of routing rules — *this host name,
this path, goes to that service* —
enforced by an **ingress controller**.
K3s bundles **Traefik** in that role. It
listens on the node's web ports (80 and
443) and reads Ingress descriptions
from the cluster:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
spec:
  rules:
    - host: hello.example.lan
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello
                port:
                  number: 8080
```

With that applied and the name
`hello.example.lan` resolving to a node
(a line in your router's or hosts file
DNS — your network's arrangement),
requests to the name arrive at Traefik
and land on the service, whatever port
the service itself uses. Many names can
share the one pair of web ports, which
is the whole advantage: services stop
competing for 80 and 443, and each gets
a human address instead of a number.

Two honest notes. Traefik also
understands certificates: hand it one
(or let it request public ones on a
public-facing cluster) and the same
rules answer on 443 with HTTPS — the
project documents the arrangements, and
on a home network a certificate you
make yourself is the usual story.
And Traefik is a default, not a
destiny: chapter 11's `--disable traefik`
removes it, and other controllers can be
installed in its place by manifest, as
in any Kubernetes.

## Which door when

- Programs inside the cluster finding
  each other: ClusterIP and a name —
  nothing in this chapter needed.
- One service, quickly, on the home
  network: LoadBalancer and ServiceLB.
- Web things with names, several of
  them, possibly with HTTPS: an Ingress
  through Traefik.
- Something that is not web traffic and
  needs its own port on a known address:
  LoadBalancer if the port is free,
  NodePort if it just needs to work.

Check: the picture is complete when a
`Pending` LoadBalancer reads to you as
"no door was available", and an Ingress
as "a name, routed" — two different
answers to two different questions,
both of which a cloud would have
answered with a bill.
