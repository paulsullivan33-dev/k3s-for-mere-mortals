# Recipes

Short, complete jobs, each standing on
chapters already read. As ever in this
guide, the cluster behaviours are the
project's documented ones; the recipes
are arrangements of them, not new
claims.

## A cluster on a spare machine

*For: the old PC in the corner, promoted.*

1. Install a plain server Linux (the
   series' own Linux guide if wanted),
   updates on, a fixed address on your
   network.
2. Run `k3s check-config` from the
   downloaded binary before installing
   anything (chapter 3) — a spare
   machine's kernel surprises are free
   to learn now, expensive later.
3. Install by script, with a name you
   will type at your desk already on the
   certificate:

   ```bash
   curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san spare.home" sh -
   ```
4. Copy the kubeconfig to your desk
   (chapter 4), change its server
   address to `spare.home`, and keep the
   copy like a password.
5. Apply chapter 6's `hello.yaml`, open
   the service's address from another
   machine, and only then delete it.
   The cluster has now been proved end
   to end: brain, runtime, network,
   door.

## A Raspberry Pi joins as an agent

*For: the Pi 4 or 5 doing nothing in a
drawer.*

1. Prepare the Pi with its standard
   operating system, a fixed address,
   and one kernel favour the
   documentation asks of small boards:
   enable the memory control group in
   the boot configuration (`cgroup_memory=1 cgroup_enable=memory`
   on the kernel command line), then
   reboot. Chapter 3's check will tell
   you in writing whether the board is
   ready — Pis are exactly the machines
   `check-config` shines on.
2. Download the ARM build (`k3s-arm64`)
   or let the install script choose it:
   the script route detects the board.
3. Join it with the server's address
   and token (chapter 10). Label it as
   it arrives — `--node-label
   board=pi5` — because the day you own
   two kinds of machine, you will want
   to say which workloads suit the
   small, cool one.
4. Mind its storage card: container
   images and chapter 9's volumes write
   often; a Pi cluster node is happier
   booting from a USB disk, which the Pi
   guides in this series describe.

## An install with no internet

*For: the workshop, the boat, the
air-gapped bench.*

Prepare on a connected machine, carry
on a stick:

1. The binary for the target's
   processor (chapter 3), with its
   checksum file.
2. The project's **airgap images**
   archive from the same release — every
   container image the cluster itself
   needs, in one tarball.
3. The install script itself, saved as
   a file (it can run locally with
   `INSTALL_K3S_SKIP_DOWNLOAD=true`).

On the target: place the images
archive in the data directory's
`agent/images/` folder before first
start, install from the saved script,
and the server comes up without a
single call home. Your own workloads'
images are your luggage too — export
them from a connected cluster
(`k3s ctr images export`, chapter 5's
low window) or pull them on a twin
machine and carry the files.

## A service the whole house can reach

*For: the photo archive, the family
dashboard, the thing everyone asks you
"how do I get to" about.*

1. Run the workload with a claim on
   local storage if it keeps data
   (chapter 9), on the node whose disk
   you trust.
2. Give it an Ingress name instead of a
   port (chapter 8): `photos.home`, say,
   via Traefik.
3. Make the name resolve: most home
   routers will add a local DNS entry;
   failing that, the household machines'
   hosts files are the honest small
   answer.
4. Decide its audience with chapter 1's
   rule held up to the light: a name on
   your own network is a household
   thing; port-forwarding it to the
   internet makes it a publication, and
   wants certificates (Traefik can
   request public ones) and a harder
   think. Nothing in this recipe
   requires the second step.

## The two-node starter, honestly

*For: readers tempted to begin with
high availability.*

Resist, gently. Begin with one server
and one agent (chapter 10): the agent
doubles the room, the server stays
simple SQLite with chapter 12's file
backups, and every concept in this
guide is present. Add the second and
third *servers* when the honest reason
arrives — "the brain being down while
I'm away is a real cost" — and not
before; etcd will still be there, and
you will meet it knowing exactly what
it is for.
