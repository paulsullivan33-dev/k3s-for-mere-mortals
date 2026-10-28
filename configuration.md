# Configuration

A K3s server or agent is steered three
ways: flags on its command line,
environment variables, and a
configuration file. This chapter teaches
the file — the way that survives
restarts and reading — and the handful
of settings real clusters actually
change. One behaviour in it is verified
by the lab for real, and it is the
friendliest possible introduction: the
file is forgiven its spelling mistakes.

## The config file

On a server or agent, K3s reads

```text
/etc/rancher/k3s/config.yaml
```

at startup. Its keys are the long flag
names with the dashes removed from
their job — `write-kubeconfig-mode`
above is the flag `--write-kubeconfig-mode` —
and lists and booleans are written as
YAML:

```yaml
write-kubeconfig-mode: "644"
tls-san:
  - "k3s.home"
  - "192.0.2.10"
disable:
  - traefik
```

The file you write is merged with the
command line, the command line winning
where they disagree. After editing,
restart the service (`systemctl restart
k3s`) and the new settings are the
cluster's.

Check — and the lab ran this one
(transcript
`k3s-lab/transcripts/t06-config-and-token.txt`):
a misspelt or invented key does **not**
stop the start. It earns one warning
line and is skipped:

```text
level=warning msg="Unknown flag --no-such-setting found in config.yaml, skipping\n"
```

Which is kind, and a trap of exactly
the same size: a real setting misspelt
is skipped just as quietly, and the
cluster runs without the thing you
meant. After any config change, check
the start of the log for that warning
before assuming your setting is live.

## The settings that matter most

**Names on the certificate: `tls-san`.**
The server's certificate covers the
names it is born knowing — localhost,
the node's own addresses, the cluster's
internal names. If you will reach the
API by another name (a DNS name for the
server, a load-balancer address in
front of several servers), list it here
*from the start*: the lab's start log
shows the certificate being minted at
first boot, and adding a name later
means the certificate chapter of the
project's docs, not a flag. This is the
single most-forgotten setting in home
clusters; it is chapter 4's desk-side
kubeconfig that fails without it.

**The kubeconfig's permissions:
`write-kubeconfig-mode`.** `644` makes
`/etc/rancher/k3s/k3s.yaml` readable by
everyone on the machine — convenient on
a single-user box, and a master key
left on the table anywhere else. Choose
knowingly.

**Turning the extras off: `disable`.**
Each bundled component of chapter 2 can
be retired by name — `traefik`,
`servicelb`, `coredns`, `local-storage`,
`metrics-server` — for readers who
bring their own ingress, storage or
DNS. Mechanically, K3s marks the
component's manifest in the auto-deploy
folder (chapter 7) as skipped, so the
off-switch and the folder are the same
story told twice. Replace like with
unlike carefully: disabling CoreDNS or
the storage class on a cluster that
already uses them is surgery, not
tidying.

**Where things live: `data-dir`.**
Chapter 2's folder, movable — the lab
ran every one of its starts with the
data directory pointed at a scratch
folder, which is how sandbox work stays
out of the system's way. On real
machines the reason to move it is a
bigger or faster disk.

**The networks, at birth:
`cluster-cidr` and `service-cidr`.** The
defaults (10.42.0.0/16 and
10.43.0.0/16 — the lab read them from
the server's own help) are chosen to be
unusual; if your real network already
uses them, change them on the first
server before anything joins, because
they are woven into certificates,
routes and rules from the first minute.

**Node facts: `node-label` and
`node-taint`.** Labels describe a node
for scheduling (`disk=ssd`); taints
repel all but tolerating workloads —
the standard way to keep ordinary pods
off the server nodes of a large cluster,
or off one peculiar machine. In a small
cluster, labels occasionally, taints
rarely.

## Environment variables, briefly

The installer and the binary both
listen to variables — every flag's help
line names its own, in brackets: the
lab's transcripts show `[$K3S_URL]`,
`[$K3S_TOKEN]`, `[$K3S_DATA_DIR]` and
friends. Their best use is install time
(chapter 3's `INSTALL_K3S_VERSION` and
chapter 10's joining one-liner). For
the standing configuration of a running
cluster, the file is the record; a
setting that exists only in a shell's
history is a setting the next restart
will lose.

Check: configuration is understood when
a change follows the whole loop in your
head — *edit the file, restart, read
the log's first lines for the warning,
then verify the behaviour, not the
belief.*
