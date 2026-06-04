# The Loop Is Coming Apart

*jj, Crabbox, Depot, Blacksmith — a quiet contest is underway over the inner dev loop. Every player is making the same promise: zero distance between changing your code and knowing if it works, on compute that isn't yours and isn't precious. The diff is becoming the unit of work, and everyone is racing to give it a warm machine to land on.*

---

For about fifty years the inner dev loop has been one thing happening in one place. You edit a file. You save it. You build and run it. All on the machine under your hands, against a checkout that lives on its disk. Git made the *history* distributed in 2005, but the loop itself never left the laptop.

A cluster of recent projects is prying it apart — a version control system, a remote-execution control plane, two CI acceleration companies, and a field guide that refuses to pick a winner. None of them set out to do the same thing. Put them side by side and they are all making the same promise.

## The promise

Strip the marketing off any of these tools and the same sentence is underneath: **the speed of your feedback should not depend on the machine under your hands.**

Every diff should get a warm, isolated, fast machine on demand. The gap between "I changed something" and "I know if it works" should collapse toward zero — and it should collapse on compute that is disposable, so a bad run costs nothing and a parallel run costs nothing more. That's the promise. It is genuinely new. And the moment you take it seriously, the loop has to come apart, because no single laptop can be simultaneously your editor, a fleet of fresh runners, and a warm cache the size of your whole dependency graph.

The interesting part is *where* the promise is being fought over, and *what keeps breaking* when you try to keep it.

## The venue: GitHub Actions, and the cache nobody could see

The contest is happening in CI/CD, because that's where the loop already half-left the laptop. [GitHub Actions](https://github.com/features/actions) is the incumbent venue; [GitLab CI/CD](https://docs.gitlab.com/ee/ci/) is the other big one. This is the arena. And the contested ground inside it turns out to be the most boring-sounding thing imaginable: **the cache.**

Here is the detail that gives the whole game away. Two different companies — [Depot](https://depot.dev) and [Blacksmith](https://www.blacksmith.sh) — independently **reverse-engineered the GitHub Actions cache protocol**, and each wrote a blog post about it with nearly the same title. Depot's: *"We reverse-engineered the GitHub Actions cache so you don't have to."* Blacksmith's: *"Reverse engineering GitHub Actions cache to make it fast."*

Why would two startups burn engineering on the same undocumented protocol? Because GitHub's hosted runners get roughly **1 Gbps of network throughput — about 125 MB/s** — and on a fresh runner, restoring the cache is the long pole of the entire build. The throttle *is* the bottleneck. When your cache restore dominates wall-clock time, **warmth becomes the product.** So an entire cottage industry formed to sell it back to you, faster, by routing around the cache that GitHub never meant you to see.

## Depot and Blacksmith: warmth, sold as a service

The two reverse-engineering crews built different answers to the same question.

**Depot** ([depot.dev](https://depot.dev)) launches ephemeral, single-tenant EC2 runners via webhook, backs them with a distributed S3 cache, and colocates your BuildKit container builders *in the same private network* as the runners. Cache moves at up to 1000 MiB/s over 12.5 Gbps — roughly **10x** GitHub's throughput — and the headline claim is *up to 55x faster builds at half the cost.*

**Blacksmith** ([blacksmith.sh](https://www.blacksmith.sh)) — built by three engineers out of CockroachDB and Faire — runs Actions on **bare-metal gaming CPUs** with about **2x the single-thread performance** of GitHub's servers, colocates a warm cache for ~4x faster reads and writes (up to 10x for some), and persists Docker layers on NVMe for **2x–40x** faster image builds. The integration is a one-line change: swap `runs-on: ubuntu-latest` for `runs-on: blacksmith-2vcpu-ubuntu-2404`.

Above both of them sits the honest meta-truth, and it's exactly what the [GitHub Actions cache field guide](https://github.com/zozo123/gha-cache-field-guide) was written to say: **CI caching is not one cache.** Native dependency caches win on warm workspaces. Build caches like Incredibuild win when runners are fresh, ephemeral, or churned by timestamps — strongest for C/C++ and Rust. BuildKit is for Docker; Go and JavaScript usually want their tool-specific cache first. There is no single knob. The field guide's refusal to crown a winner is the correct response to a venue where warmth is contested ground and the right move depends entirely on whether your runner is hot or cold.

## jj: the diff stops being a verb

So compute and warmth have left the laptop. What about the code itself?

[Jujutsu](https://github.com/jj-vcs/jj) (`jj`) speaks Git's storage format but discards its mental model. The headline feature sounds like a footnote: **there is no staging area, and your working copy is itself a commit.** `jj` snapshots the working copy on every command, so the moment you save a file, that change *is* a commit — it has an ID, it's in the graph.

In Git, your dirty checkout is a *verb*: a thing in motion that you must freeze — `add`, `stash`, `commit` — before you can do anything with it. In jj, the dirty checkout is a *noun*: a first-class, addressable, syncable object that exists whether or not you've decided you're done. That distinction is the hinge of everything that follows. To ship a diff to a remote machine, the diff first has to *be a thing.*

## Crabbox: the loop, made local-first again

[Crabbox](https://crabbox.sh) — by [Peter Steinberger](https://x.com/steipete) (`@steipete`), creator of OpenClaw — is where the pieces snap together. Its tagline: **"A short-lived box for every run."** Its promise, verbatim: *"Crabbox gives maintainers and agents a fast local loop on shared cloud capacity: lease, sync, run, release."*

```bash
crabbox run -- pnpm test          # lease a box, sync your dirty tree, run, tear down
crabbox warmup                    # provision and keep it warm
crabbox run --id blue-lobster --  # reuse the warm box
```

You keep your editor and your git workflow; Crabbox **rsyncs your dirty checkout** to a leased remote box — it is "local-first and does not require a clean checkout." It seeds remote Git from your base ref, overlays dirty files, skips no-op syncs by fingerprint, and guards against suspicious mass deletions. It does *not* ask you to commit first. That's jj's addressable diff, made portable.

But here's the move that makes Crabbox more than another runner: **it doesn't compete with Depot and Blacksmith — it targets them.** Crabbox is a control plane with a provider list, and that list reads like a census of this whole essay: **Blacksmith Testbox, E2B, Sprites, Modal, Daytona, Tensorlake, AWS, Hetzner — and [islo.dev](https://islo.dev).** Crabbox 0.3.0 literally shipped a "Blacksmith Testbox wrap." It is the layer that makes all of that disposable cloud compute *feel local again.*

And the convergence runs both ways. Blacksmith's own **Testbox** — Linux microVMs that `warmup` then `run`, syncing with `rsync --delete --checksum`, subsequent runs in 1–3 seconds, used to reproduce flaky tests across dozens of boxes in parallel — is the exact same move from the runner vendor's side. Two ends of the market, independently, arrived at: *warm a box, sync the diff, run the suite.*

## The loop, unbundled

Stack it all up and the inner dev loop has come apart into concerns that no longer have to live on the same machine:

| Concern | Used to be | Becoming | Who's building it |
|---|---|---|---|
| **Edit** | local file | local file (still) | you |
| **Version** | freeze the diff first | the diff is already an object | jj |
| **Transport** | n/a — it's right here | sync the diff to compute | Crabbox |
| **Compute** | your laptop | leased ephemeral runner | Depot, Blacksmith |
| **Warmth** | whatever your disk had | an engineered cache strategy | Depot, Blacksmith, the field guide |
| **Isolation** | none — same OS | a fresh sandbox per run | islo.dev, Testbox, E2B |

The accelerant under all of it is agents. An AI coding agent produces dirty diffs at machine speed and has no laptop to run them on. It wants precisely this stack: a diff that's already an object, a way to ship it to real compute, ephemeral isolation so a bad run can't touch anything, and **verifiable evidence** at the end instead of terminal output that scrolls away. Crabbox's pitch — "proof for every run," bundling screenshots, video, JUnit summaries, logs, and lease metadata — is not a coincidence. It's the loop being rebuilt for a participant that was never sitting at the keyboard.

## Where this lands

I work on sandboxed runtimes at [islo.dev](https://islo.dev), and we're already one of the providers in that Crabbox list — so I'll say the thing the rest of the field only implies: once the loop is unbundled, **the sandbox is the substrate everything else sits on.** The diff is mobile, the compute is disposable, the cache is engineered — and the only question left that matters is whether the box your code lands in is fast to start, safe by construction, and warm enough to be useful.

That's also where the promise gets honest. "Zero distance between change and knowing" is only true if the box is *already warm* and *already isolated* when your diff arrives. Cold boxes break the promise. Shared boxes break it differently. The reason two companies reverse-engineered the same opaque cache is that warmth is the hard part, and the reason crabbox and Testbox both reinvented `warmup → sync → run` is that isolation has to be cheap enough to throw away.

jj made the diff addressable. Crabbox made it portable. Depot and Blacksmith made it fast. The field guide made warmth honest. The remaining job — give every mobile diff a clean, fast, isolated machine to land on — is the one worth getting right.

The loop isn't dying. It's just stopped fitting on one desk.
