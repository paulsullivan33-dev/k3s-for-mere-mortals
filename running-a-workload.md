# Running a Workload

The outcome: a small web server running
on your cluster — two copies of it —
reachable through an address the cluster
hands out, then scaled, then removed.
Everything here happens on a live
cluster, so like chapter 4 this chapter
is written from the project's
documentation; the manifest it deploys,
though, is real work from the guide's
lab, built and checked with the tools of
chapter 5.

## The two descriptions

Kubernetes runs **workloads**: programs
described, not started by hand. The two
descriptions this chapter uses are the
two a beginner uses daily:

- A **Deployment** says: run this
  container image, this many copies,
  and keep that true. Kill a copy and
  the deployment replaces it; that
  stubbornness is the point of the
  platform.
- A **Service** says: give those copies
  one stable address and name inside
  the cluster, however often the copies
  themselves are replaced.

The program is nginx, the small web
server, in its Alpine flavour — chosen
because it answers on port 80 with a
page that proves it lives. The
lab-built manifest for both descriptions
is `k3s-lab/manifests/hello.yaml`, and
chapter 7 takes it apart line by line.
Its deployment half began life as a
chapter 5 dry run:

```bash
k3s kubectl create deployment hello --image=nginx:alpine --replicas=2 --dry-run=client -o yaml
```

## Starting it, and watching

The imperative way — fine for a first
look, and how most people meet the
platform:

```bash
k3s kubectl create deployment hello --image=nginx:alpine --replicas=2
k3s kubectl get pods --watch
```

The documented story: the deployment is
created, the scheduler picks nodes (on
a one-machine cluster, yours), the node
pulls the image from its registry, and
two pods — one container each — come up
`Running`, named `hello-…` with
generated suffixes, because copies are
interchangeable and named like it. The
first pull is the slow part; later
starts reuse the node's image store.
`get pods` without `--watch` is the
same snapshot on demand, and
`k3s kubectl describe deployment hello`
narrates the whole event history when a
pod does *not* come up — the first
command of chapter 16.

Two copies deserve a sentence: asking
for two and watching one get replaced
after you delete it

```bash
k3s kubectl delete pod <one of the two names>
k3s kubectl get pods
```

is the moment the platform's promise
stops being a slogan. The deployment
noticed the world disagreed with its
description and corrected it. Nobody
restarted anything.

## Giving it an address

Pods have addresses, but they are
ephemeral — a replaced pod is a new
address. The service is the stable
front:

```bash
k3s kubectl create service loadbalancer hello --tcp=8080:80
```

That is the exact command whose dry run
the lab captured (transcript t02), with
the dry-run flags removed. `LoadBalancer`
is the interesting word: on a cloud
cluster it would order a real load
balancer from the provider. On K3s,
ServiceLB (chapter 2) answers instead:
the service's external address becomes
an address of one of your nodes, and
the port — 8080 here — answers there.
Read it back:

```bash
k3s kubectl get service hello
```

The `EXTERNAL-IP` column names the node
address it claimed, and from any machine
on your network,
`http://<that address>:8080` returns
nginx's welcome page. Inside the
cluster, the same service answers to
the plain name `hello` — CoreDNS at
work — which is how your own programs
will find each other.

(The other service types earn a line
each: `ClusterIP`, the default, is
inside-the-cluster only; `NodePort`
opens a high numbered port on *every*
node. LoadBalancer-via-ServiceLB is
K3s's home answer and the friendliest
default at this scale; chapter 8 gives
the whole picture, including Traefik
for proper web names.)

## Scaling and updating

```bash
k3s kubectl scale deployment hello --replicas=4
k3s kubectl set image deployment hello nginx=nginx:1.27-alpine
k3s kubectl rollout status deployment hello
```

Scaling is a number change the
controllers enforce; updating is a
description change they roll out copy
by copy, old ones retired as new ones
prove healthy — `rollout status` is the
command that waits and tells you. Both
are the same idea as the resurrected
pod: the description is the truth, and
the cluster closes the gap.

## Taking it away

```bash
k3s kubectl delete service hello
k3s kubectl delete deployment hello
```

and the cluster closes that gap too:
pods end, the address is released.
Deleting is as declarative as creating,
which is the discipline chapter 7's
manifests formalise — everything made
from a file is removed by deleting the
file's contents, nothing lingers by
hand. Check: the workload chapter is
done when you can say what a deployment
promises, what a service promises, and
why the pair — not either alone — is a
program the cluster can look after.
