# Troubleshooting

Symptom first, cause second, in the
order the documentation and this
guide's own lab suggest you check them.
The universal first step costs nothing:
read the log. On a service install that
is `journalctl -u k3s` (or `-u k3s-agent`);
in the foreground, it is the terminal
you started it in.

## The server will not start

*The service fails, or the foreground
command exits, in the first seconds.*

1. **The machine cannot host it.** Run
   `k3s check-config` (chapter 3). If it
   counts failures — missing kernel
   modules, control groups it cannot
   use — believe it and fix them at the
   machine's level, not K3s's.
2. **The wall this guide's lab hit.**
   In the sandbox's captured start
   (transcript
   `k3s-lab/transcripts/t05-server-attempt.txt`)
   the server built its database, minted
   its certificates, and then stopped:

   ```text
   level=fatal msg="Error: failed to find cpu cgroup (v2)"
   ```

   That sentence means the kernel's
   control-group hierarchy offers the
   process no CPU group to manage —
   the signature of a restricted
   container or a host whose cgroup v2
   controllers are not delegated. It is
   a property of the room, not the
   guest: ordinary desktops, servers,
   virtual machines and Pis do not
   present it. If you meet it, you are
   almost certainly trying to run K3s
   *inside* a container that was not
   built to host it; move to the host or
   a proper VM.
3. **The port is taken.** Something
   already owns 6443 (a previous
   cluster? another Kubernetes?) — the
   log names the bind failure. Stop the
   other tenant or, deliberately, choose
   another `--https-listen-port` and
   accept that every address you publish
   changes with it.
4. **A misspelt config key did it —
   quietly.** Chapter 11's warning cuts
   the other way too: a setting you
   *needed* (a name, a network) can be
   the one that was skipped as unknown.
   Read the first lines of the log for
   `Unknown flag` before deeper surgery.

## The kubeconfig does not work

*`kubectl` refuses, cannot connect, or
asks questions about certificates.*

1. **Permissions.** The file is
   root-only by design (chapter 4). The
   error is the file's mode, not a
   cluster fault: use sudo, copy the
   file and own the copy, or set
   `write-kubeconfig-mode` knowingly.
2. **Wrong address.** The file says
   `127.0.0.1` because it was written
   for the server itself. From your
   desk, the address must be the
   server's — and the name you use must
   be on the server's certificate,
   which is chapter 11's `tls-san`,
   ideally set at birth. A certificate
   error naming the host is this, nine
   times in ten.
3. **Expired certificates.** A cluster
   stopped longer than its certificates
   live wakes up to exactly this
   (chapter 12): `k3s certificate` and
   the project's renewal walk are the
   fix, and a restart *before* expiry
   is the prevention.

## A node never becomes Ready

*`get nodes` shows the new machine,
stuck at `NotReady`.*

1. **The container pieces underneath.**
   `kubectl describe node <name>` lists
   the reasons in its events — read them
   before guessing. Runtime and network
   complaints there are the node's own,
   and chapter 5's `crictl` on that node
   shows the runtime's side of the
   story.
2. **The tunnel cannot form.** Pods
   fine on one node, unreachable across
   nodes, VXLAN errors in the log:
   UDP 8472 is being filtered between
   the machines (chapter 10's table).
   Home networks with "helpful"
   isolation between wired and wireless
   segments do this silently.
3. **The Pi favour.** On small boards,
   the memory control group was never
   enabled (chapter 13): `check-config`
   on the node says so in writing.
4. **Clocks.** Certificate systems are
   clock systems. A node whose time is
   far from the server's fails in
   ways that look like network trouble;
   check the time on both before
   anything exotic.

## Pods will not start

*Deployments created, pods stuck at
`Pending`, `ImagePullBackOff`, or
`CrashLoopBackOff`.*

1. **`kubectl describe pod <name>`** —
   its events section is the cluster
   telling you which of these three it
   is. This is the first command, not
   the third.
2. **Pending** is usually arithmetic or
   geography: not enough room anywhere,
   or a volume claim that can only be
   satisfied on a node the pod cannot
   use (chapter 9's local catch).
3. **ImagePullBackOff** is the node
   failing to fetch the image: a
   misspelt name, a private registry
   with no credentials handed over, or
   — on chapter 13's offline clusters —
   an image that was never in the
   luggage.
4. **CrashLoopBackOff** is the program
   itself starting and dying: the
   cluster is healthy and the workload
   is not. `kubectl logs <name>
   --previous` reads the last words of
   the previous copy.

## A LoadBalancer service sits at Pending

*A chapter 8 service whose `EXTERNAL-IP`
never arrives.*

ServiceLB grants addresses from the
nodes' own. Pending means no grant was
possible: most often the port is
already spoken for on every node that
could claim it (two services, one
port 80 — use an Ingress name instead,
chapter 8), or the service asked for a
specific address the cluster does not
own. `kubectl describe service <name>`
shows the refusal and usually its
reason.

## The cluster works, but slowly, on a small machine

The speedometer is built in:
`kubectl top nodes` and `kubectl top
pods` (metrics-server, chapter 2). On
small hardware the usual truth is
memory pressure — read chapter 15's
datastore ceiling alongside: a server
that is also running every workload,
on a board with a gigabyte to spare,
is doing arithmetic, not failing. The
fix is a bigger node, an agent to share
the load (chapter 10), or fewer
workloads — in that order of honesty.
