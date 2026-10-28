# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's traffic is the lab talking to itself.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 15 names what the tool
   cannot do: grow past its single small database without
   changing it, make one server highly available, keep data
   on one node and have it follow a workload elsewhere, or
   treat its bundled defaults as the only possible choices —
   and these notes state the guide's own verification ceiling
   plainly: the lab's sandbox could not run the server, so the
   cluster's live behaviour is documented-only, as the series'
   Kubernetes guide was with its client-side ceiling.
8. **The ethics rule is in the text, early.** Chapter 1 and the
   README state the rule plainly — *your clusters, your
   machines, or explicit permission* — before any procedure
   depends on it, and later chapters point back to it where
   the tooling would make crossing the line easy: chapter 10,
   where joining a machine takes an address and a token, and
   chapter 15, where the line is drawn out in full.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a 2-CPU, 8 GB x86_64 sandbox — a
restricted container, a fact that decides this guide's ceiling.
The official **k3s v1.37.1+k3s1** binary (81,543,330 bytes) was
downloaded from the project's GitHub releases into a scratch
folder, verified against the release's published
`sha256sum-amd64.txt` before it was run, and exercised from
that folder only; every server start used a data directory
inside the scratch space. All transcripts live in
`k3s-lab/transcripts/`; the example manifest is
`k3s-lab/manifests/hello.yaml`.

Run for real, with transcripts as evidence:

- The download's checksum matching the published sum, the
  binary's version and the bundled kubectl's client version
  (t00), and the full top-level command surface — server,
  agent, kubectl, crictl, ctr, check-config, token,
  etcd-snapshot, secrets-encrypt, certificate (t01).
- Client-side manifest work: `kubectl create deployment` and
  `kubectl create service loadbalancer` with
  `--dry-run=client -o yaml`, generating the manifests the
  guide's example is built from (t02).
- The exact boundary of that client-side work: file-based
  `kubectl create -f` and `apply -f` refuse without a cluster,
  because they ask the API to interpret the file's kinds —
  the errors are quoted verbatim (t03).
- `k3s check-config`, the project's own readiness check, run
  in full: it unpacked and verified its bundled binaries,
  then reported the sandbox unfit — cgroup v2 CPU, cpuset and
  memory controllers unavailable, and the bridge, netfilter
  and VXLAN kernel pieces missing; final verdict
  `STATUS: 13 (fail)` (t04).
- A genuine single-node `k3s server` attempt, with the data
  directory, kubeconfig output and TLS names pointed into
  scratch space. The start was healthy as far as it went:
  the SQLite datastore was created through Kine and
  validated, the full set of cluster certificates was
  generated, and the supervisor began listening — then the
  kernel wall arrived and the server shut itself down
  gracefully on the fatal error `failed to find cpu cgroup
  (v2)`, about a second in (t05). Nothing was left running;
  all state was in the scratch folder and has been removed.
- Configuration handling: a config file carrying one valid
  key and one invented key was read at startup, the invented
  key producing exactly the documented warning and being
  skipped while startup continued to the same cgroup wall;
  and `k3s token generate` minted a bootstrap token with no
  server present (t06).
- The server's and agent's real help text, from which the
  guide's defaults were read: ports 6443, pod and service
  networks 10.42.0.0/16 and 10.43.0.0/16, the VXLAN flannel
  default, the `--cluster-init` embedded-etcd switch, and
  the agents' `--server`/`--token` (K3S_URL/K3S_TOKEN) join
  flags (t01, and the checks behind chapters 2, 10 and 11).

Documented only — described from the project's documentation,
not run, and the text does not claim otherwise:

- Everything on a running cluster: a node reporting Ready,
  pods starting, the bundled CoreDNS, Traefik, ServiceLB,
  local-path-provisioner and metrics-server at work,
  scaling, rollouts and self-healing. The sandbox's kernel
  (no delegated cgroup v2 controllers; bridge, netfilter and
  VXLAN pieces missing, per the check-config transcript) is
  the whole reason, and these are limits of the sandbox, not
  of K3s — none of them apply to an ordinary laptop, spare
  machine, virtual machine or Raspberry Pi.
- The install script's run (the sandbox has no service
  manager to install into); the binary it delivers and
  everything after delivery were verified by the manual
  route instead.
- Joining agents, multi-server clusters with embedded etcd,
  upgrades, snapshots and restores, secrets encryption on a
  live server, and removal by the uninstall scripts.
- The comparison tools in chapter 14 are characterised, not
  benchmarked.

Fixtures are this guide's own and deliberately generic: one
small nginx deployment and service used throughout, a token
minted by the lab binary for the occasion and never used to
join anything, and a sandbox that talked only to itself. No
example is modelled on any real person, network, service or
organisation.
