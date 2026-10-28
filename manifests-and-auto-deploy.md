# Manifests and Auto-Deploy

Chapter 6 typed commands at the cluster.
This chapter does it the way clusters
are actually run: a **manifest** — a
text file in YAML that describes what
should exist — applied whole, kept in a
folder, changed by editing and
re-applying. By the end you can read the
guide's example manifest line by line,
apply it, and use the folder where
dropping in a file *is* the deployment.

## The example, taken apart

`k3s-lab/manifests/hello.yaml` holds two
documents (the `---` is the seam):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app: hello
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: web
          image: nginx:alpine
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  type: LoadBalancer
  selector:
    app: hello
  ports:
    - port: 8080
      targetPort: 80
```

Read it as two forms:

- **What and who.** Every document opens
  by naming its kind (`Deployment`,
  `Service`), the API version that kind
  belongs to, and its own name in
  `metadata`. Names are how everything
  refers to everything else.
- **Labels and selectors.** The
  deployment stamps its pods with the
  label `app: hello` (in the template),
  declares which pods are *its* by that
  label (the `selector`), and the
  service uses the same label to find
  the pods it fronts. Labels are the
  cluster's only notion of belonging —
  spelling them consistently is most of
  writing manifests.
- **The spec.** The desired state: two
  replicas of one container, image
  `nginx:alpine`, listening on port 80;
  a service answering on 8080 and
  forwarding to 80. Everything else —
  where they run, when they restart —
  is the cluster's business.

The lab built this file's pieces with
chapter 5's client-side generators and
checked what it could: the generators'
output is verbatim real (transcript
t02). What the lab could *not* do
offline is have kubectl bless the file
itself — file-reading commands ask a
live API to interpret the kinds
(transcript t03) — a limitation it
shares with the series' Kubernetes
guide, and the reason this guide leans
on the generators as its local check.

## Applying, and the habit of files

```bash
k3s kubectl apply -f hello.yaml
k3s kubectl get all
```

`apply` compares file to cluster and
closes the gap either way — creating,
updating, and leaving alone what
already matches — so the same command
is install, upgrade and no-op. Re-apply
after every edit. The discipline that
grows from this is the platform's best
habit: **the folder of manifests is the
cluster's description**, kept under
version control, reviewed like code.
Chapter 13's recipes lean on it, and
the *GitOps for Mere Mortals* guide in
this series is where the habit leads
when it grows up.

`get all` is the quick census —
deployments, pods, services in one
list — with `kubectl delete -f hello.yaml`
as the exact undo.

## The auto-deploy folder

K3s adds a second, more automatic way
in. The server watches a folder of its
own:

```text
/var/lib/rancher/k3s/server/manifests/
```

Any manifest file placed there is
applied by the server itself, at start
and whenever the file changes — no
kubectl, no login. It is how the
cluster's own extras arrive (the
bundled components of chapter 2 live
there as manifests and chart files,
which is why `--disable` in chapter 11
works the way it does), and it is
available to you: drop a file in,
restart nothing, and the workload
appears; edit the file and it updates;
remove it and the workload is deleted.

Use it with judgement. It is perfect
for the cluster's own furniture — the
things that should exist on *this*
cluster from the moment it boots, and
for appliances you build for others.
It is a poor place for everyday
application work, because the file acts
silently from a root-owned folder on
one server: no record of who applied
what, and — with several servers —
each server's folder is its own (the
project documents keeping them in step
as your problem). The working rule most
clusters settle on: auto-deploy for the
platform's own pieces, `apply` from a
repository you control for everything
else.

Check: you understand the machinery
when the two routes feel like one idea
at two temperatures — *a manifest is a
description the cluster enforces; apply
is you handing it over, and the folder
is the server picking it up itself.*
