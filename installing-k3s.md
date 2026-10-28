# Installing K3s

By the end of this chapter K3s is on the
machine — or you know, from the
project's own check and in its own
words, exactly why this machine cannot
host it yet. Two routes lead to the same
binary: the project's install script,
which most readers should use, and the
manual route — download the file, check
it, put it in place — which is the route
this guide's lab verified step by step,
because a lab with no system manager to
install a service into can still do
everything else honestly.

## What the machine needs

K3s is not fussy, but it is not magic
either:

- **A Linux machine** (or virtual
  machine): a server, a spare PC, a
  Raspberry Pi 4 or 5. Builds exist for
  64-bit Intel/AMD, 64-bit ARM, and
  32-bit ARM, all from the same releases
  page. The server wants, as a rule of
  thumb, a couple of CPU cores and a
  gigabyte or two of memory for a small
  cluster; agents manage with less, and
  the project's documentation has sizing
  notes as clusters grow.
- **A modern kernel with the usual
  container pieces** — control groups,
  namespaces, the bridge and filter
  modules. Ordinary desktops, servers
  and Raspberry Pi OS have them; some
  minimal and containerised environments
  do not, which is exactly what the
  check below detects.
- **Root (sudo) access** for the normal
  install: K3s manages the machine's
  networking, so it runs as a system
  service. (A rootless mode exists; it
  is marked experimental by the project
  and this guide does not teach it.)
- **Ports it can own** — 6443 for the
  API, and between nodes 8472/UDP and
  10250, which chapter 10 lists in full.

## Route one: the install script

The documented route is one command,
run as root or through sudo:

```bash
curl -sfL https://get.k3s.io | sh -
```

The script reads your system, downloads
the right binary for your processor,
puts it at `/usr/local/bin/k3s`, writes
and starts a system service named `k3s`,
and leaves two more gifts in
`/usr/local/bin`: `k3s-uninstall.sh`
and, for agents, `k3s-agent-uninstall.sh`
— chapter 12 uses them. Useful knobs are
environment variables set before the
pipe: `INSTALL_K3S_VERSION` pins an
exact release instead of the latest
(`INSTALL_K3S_VERSION=v1.37.1+k3s1`),
and `INSTALL_K3S_EXEC` adds server flags
at install time. This route is described
from the project's documentation: the
lab's sandbox has no running service
manager to install into, so the script
itself is the one step in this chapter
the lab did not execute. Everything the
script *delivers* — the binary and its
behaviour — the lab did.

## Route two: the binary by hand

The manual route is also the air-gap
and audit-friendly route, and it is
short:

1. From the project's releases page
   (github.com/k3s-io/k3s, under
   Releases), download the asset named
   `k3s` for 64-bit Intel/AMD machines
   (`k3s-arm64` for 64-bit ARM), and the
   `sha256sum-amd64.txt` file beside it.
2. Check the download against the
   published sum before running anything.
3. Make it executable and, as root, move
   it onto your path.

Check: the lab did exactly this with
release v1.37.1+k3s1, and the sum
matched (transcript
`k3s-lab/transcripts/t00-version.txt`):

```text
$ sha256sum k3s
92b94582eb7b34a8cf532e6006842b9ed215a6783cf56a3dd6271fd35243b2c0  k3s
$ k3s --version
k3s version v1.37.1+k3s1 (356c0254)
```

The habit is worth keeping beyond this
guide: a cluster tool runs as root on
every machine you own, so "I downloaded
a binary from the internet" deserves the
thirty seconds a checksum costs.

Run by hand this way, `k3s server` is a
foreground program; the script route's
service is what makes it survive logouts
and reboots on a real machine. On your
own server, prefer the script (or write
your own service unit around the same
command); use the manual route where you
need control, or where there is no
internet to script against (chapter 13's
offline recipe).

## The check: is this machine able?

Before the first cluster, run the
binary's own physical examination:

```bash
k3s check-config
```

It verifies the bundled helper programs
(it unpacks them first — chapter 2's
data directory — and checks their sums),
then reads the kernel you are actually
running: control groups, namespaces,
network modules, storage drivers. It
prints a line per item — `ok`,
`enabled`, `missing`, `bad (fail)` — and
ends with a verdict.

Check: on a healthy ordinary machine
the verdict is a count of zero failures
and the guide can proceed. The lab's
sandbox is the instructive opposite:
its verdict was **`STATUS: 13 (fail)`**
(transcript
`k3s-lab/transcripts/t04-check-config.txt`),
including:

```text
- cgroup hierarchy: cgroups V2 mounted, cpu|cpuset|memory controllers status: bad (fail)
- CONFIG_BRIDGE: missing (fail)
- CONFIG_BRIDGE_NETFILTER: missing (fail)
- CONFIG_IP_NF_FILTER: missing (fail)
```

A restricted container, as suspected:
the kernel gives it no control over CPU
and memory groups, and the modules
container networking needs are absent.
No setting inside the sandbox can fix
that, and chapter 16 names it as a
troubleshooting symptom in its own
right. Two honest consequences follow.
First, `check-config` itself works
anywhere — it is reading and reporting,
and the lab ran it for real. Second,
the *server* needs what the check checks;
on a machine that fails it, the rest of
this guide is still worth reading, but
the doing waits for a machine that
passes — a spare PC, a Pi, a virtual
machine with the kernel pieces present.

## After the install

Whether you arrived by script or by
hand, the same things now exist:

| Place | What it is |
|---|---|
| `/usr/local/bin/k3s` | The binary (script route) |
| `/var/lib/rancher/k3s/` | The data directory: database, certificates, images, bundled helpers |
| `/etc/rancher/k3s/` | Configuration file and, after first start, the kubeconfig |
| Service `k3s` | The running server (script route), managed like any service: `systemctl status k3s` |

One caution that belongs to this
chapter: the ports and powers above mean
a K3s server is a privileged thing on
its network. Install it on machines that
are yours (chapter 1's rule), and do not
expose port 6443 to the open internet —
the cluster's front door should face
your own network only.
