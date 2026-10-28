# Your First Cluster

The outcome of this chapter: a server
started, a kubeconfig in your hand, and
your first node reporting itself Ready —
the smallest possible cluster, complete.
A note on evidence before the commands:
this chapter's procedures are written
from the project's documentation. The
guide's lab sandbox could not start the
server (its kernel lacks the pieces
chapter 3's check counts), so this
chapter quotes no transcript of a Ready
node and pretends to none. What the lab
did verify is everything up to the
start: the install, the check, the
config file's behaviour, and the
server's first second, captured in
transcript t05.

## Starting the server

On a machine installed by the script,
the cluster is already running — the
installer started the service. To start
one by hand, or to see what the service
does for you:

```bash
sudo k3s server
```

The first start is busy for a few
seconds. The server prepares its data
directory, creates its database, and
mints a full set of certificates — the
cluster's own certificate authorities
and the keys for every part — before it
opens the door on port 6443. The lab's
captured start (transcript
`k3s-lab/transcripts/t05-server-attempt.txt`)
shows the healthy beginning, including
the datastore being born:

```text
level=info msg="Starting k3s v1.37.1+k3s1 (356c0254)"
level=info msg="generated self-signed CA certificate CN=k3s-server-ca…"
level=info msg="Datastore connection validated successfully, proceeding with bootstrap data generation"
```

Left running, the log settles and the
server announces itself ready. Started
by the service instead, the same story
is in the service log:

```bash
sudo systemctl status k3s
sudo journalctl -u k3s -f
```

## The kubeconfig: proof you own it

Whoever can read the cluster's
**kubeconfig** can command the cluster:
it holds the server's address and an
administrator's certificate. The server
writes it to:

```text
/etc/rancher/k3s/k3s.yaml
```

root-owned and readable only by root,
which is correct and, the first time,
mildly annoying. Three civilised ways
to work with it:

```bash
# Use it in place, command by command:
sudo k3s kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml get nodes

# Or tell your shell where it lives:
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Or copy it to your own account and own the copy:
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

Note the shape: **`k3s kubectl`** — the
binary wearing its kubectl hat — so a
k3s machine never needs a separate
kubectl installed. (Setting the install
to also write the file readable is
chapter 11's `--write-kubeconfig-mode 644`
— convenient, and a small security
trade to make knowingly, since the file
is a master key: treat whichever copy
you keep like a password.)

Inside the file, one detail surprises
people: the server's address is
`https://127.0.0.1:6443`. The kubeconfig
is written for use *on the server
itself*. To command the cluster from
your desk, copy the file there and
change that one address to the server's
name or network address — chapter 11's
`--tls-san` flag is how you make the
server's certificate valid for that
name in advance.

## Reading the cluster

With the kubeconfig in hand:

```bash
k3s kubectl get nodes
```

The documented healthy answer, a minute
after first start, is one line: your
machine's name, `STATUS` reading
**Ready**, `ROLES` reading
`control-plane,master` (the server is
also a worker — chapter 2), and an age
counting up. Ready means the kubelet
has checked in, the runtime is up, and
the node accepts work. Two more views
complete the first look:

```bash
k3s kubectl get pods -A
k3s kubectl cluster-info
```

`get pods -A` shows every pod in every
corner of the cluster, including the
system's own: CoreDNS, Traefik, the
local-path helper and metrics-server
from chapter 2, each `Running` once the
cluster settles. That list is the extras
in the box, visible at last.
`cluster-info` prints the addresses of
the API and of CoreDNS — the cluster
naming its own front doors.

Check: you have a first cluster when
`get nodes` says Ready and the system's
pods are Running. Everything after this
chapter is using it.

## Stopping and starting

Service installs behave like services:
`sudo systemctl stop k3s` and `start`.
A foreground server stops with Ctrl-C
and shuts down gracefully — the log
says so, and even the lab's doomed start
managed a tidy exit:

```text
level=info msg="Shutdown request received"
level=info msg="Closing database connections…"
```

Nothing is lost by stopping: the truth
is in the data directory, not in the
running process, and the next start
reads it back. What a stop does not do
is remove anything — removal is chapter
12's uninstall story.
