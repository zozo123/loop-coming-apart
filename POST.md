# The Loop Is Coming Apart

*jj, crabbox, and a field guide to CI caching are three unrelated projects. They are quietly arguing for the same future: the edit-save-run loop is unbundling into networked parts, and the diff is becoming the thing that moves.*

---

For about fifty years the inner dev loop has been one thing happening in one place. You edit a file. You save it. You build and run it. All on the machine under your hands, against a checkout that lives on its disk. Git made the *history* distributed in 2005, but the loop itself never left the laptop.

Three recent projects disagree, each from a different corner, none of them talking to the others. Put them side by side and a shape appears.

## jj: the working copy stops being special

[Jujutsu](https://github.com/jj-vcs/jj) (`jj`) is a version control system that speaks Git's storage format but throws out Git's mental model. The headline feature sounds like a footnote: **there is no staging area, and your working copy is itself a commit.**

In Git, your uncommitted edits are a special, fragile, un-addressable state. They live in the index and the worktree, outside the object graph, and half of Git's complexity exists to shuttle them in and out — `add`, `stash`, `reset --soft`, the difference between `--mixed` and `--hard`. In jj, the moment you save a file, that change *is* a commit. It has an ID. It is part of the graph. `jj` snapshots the working copy on every command.

That sounds like a quality-of-life tweak. It is actually a change in what a "diff" *is*. In Git, the dirty checkout is a verb — a thing in motion that you have to freeze before you can do anything with it. In jj, the dirty checkout is a noun: a first-class, addressable, syncable object that exists whether or not you've decided you're "done."

Hold that thought.

## crabbox: the diff goes somewhere else to run

[Crabbox](https://github.com/openclaw/crabbox) describes itself in seven words: **warm a box, sync the diff, run the suite.** It's a control plane for remote test and command execution — for maintainers, and increasingly for AI agents that need real compute and reviewable evidence instead of ephemeral terminal scrollback.

The workflow is `lease → sync → run → release`:

```bash
crabbox run -- pnpm test          # lease a microVM, sync, run, tear down
crabbox warmup                    # provision a box and keep it warm
crabbox run --id blue-lobster --  # reuse the warm box
```

Under the hood: a Go CLI on your laptop, a Cloudflare Worker broker that owns provider credentials and lease state, and stateless runners that are "leaves: provisioned, used, deleted." It leases ephemeral microVMs from providers like Sprites, E2B, and Firecracker sandboxes.

The line that matters is buried in the sync layer: it is **"local-first and does not require a clean checkout."** It seeds remote Git from your base ref, then overlays your dirty files with rsync, skipping no-op syncs via fingerprints. It guards against suspicious mass deletions. It does *not* ask you to commit first.

Read that next to jj. Crabbox treats your dirty working copy as a portable, addressable thing it can ship to a fresh machine — exactly the noun that jj just made it. One project made the diff a first-class object. The other made it the unit of transport.

## The cache field guide: warm is the whole game

The catch with "run it on a fresh, ephemeral microVM" is that *fresh* and *fast* are enemies. A clean runner has no dependency cache, no compiler cache, no warm Docker layers. It is correct and it is slow.

The [GitHub Actions cache field guide](https://github.com/zozo123/gha-cache-field-guide) is a small, honest answer to that problem, and its thesis is a refusal to oversell: **CI caching is not one cache.** There is no single knob.

- **Native dependency/build caches** win when the same workspace is already warm.
- **Incredibuild Build Cache** wins when runners are fresh, ephemeral, or invalidated by timestamp churn — strongest for C/C++ and Rust.
- **BuildKit cache** is for Docker.
- For Go, Docker, and JavaScript, reach for the tool-specific cache first.

It is deliberately *not* a universal endorsement of anything. The whole point is that the right cache depends on whether your runner is warm or cold — and ephemeral runners are cold by construction.

This is the third corner of the same shape. Once the diff detaches from your laptop and runs on disposable compute, **warmth becomes an explicit, separate, networked concern.** It used to be free — it was just whatever your machine had lying around. Now someone has to engineer it, and the honest engineering answer is "it depends, here's the decision tree."

## The loop, unbundled

Stack the three up and the inner dev loop has come apart into parts that no longer have to live together:

| Concern | Used to be | Becoming |
|---|---|---|
| **Edit** | local file | local file (still) |
| **Version** | freeze the diff first (Git) | the diff is already an object (jj) |
| **Transport** | n/a — it's right here | sync the diff to compute (crabbox) |
| **Compute** | your laptop | leased ephemeral microVM |
| **Warmth** | whatever your disk had | an engineered cache strategy |

None of these projects set out to build that table. jj is fixing Git's ergonomics. Crabbox is giving maintainers cloud compute without the commit dance. The field guide is just trying to make CI honest about caching. But they rhyme, and the rhyme is the future: **the diff is the unit of work, and it's mobile.**

The accelerant here is agents. An AI coding agent produces dirty diffs at machine speed and has no laptop to run them on. It wants exactly this stack — a diff as a first-class object, a way to ship it to real compute, ephemeral isolation so a bad run can't touch anything, and verifiable evidence at the end instead of terminal output that scrolls away. Crabbox's pitch — "agent-ready observability" — is not a coincidence. It's the loop being rebuilt for a participant that was never sitting at the keyboard.

## Where this lands

I work on sandboxed runtimes at [ISLO](https://islo.dev), so I'll say the obvious thing the other three projects only imply: once the loop is unbundled, **the sandbox is the substrate everything else sits on.** The diff is mobile, the compute is disposable, and the only question that matters is whether the thing you ship your code into is fast to start, safe by construction, and warm enough to be useful.

jj made the diff addressable. Crabbox made it portable. The cache field guide made warmth a first-class decision. The remaining job — give every mobile diff a clean, fast, isolated machine to land on — is the one worth getting right.

The loop isn't dying. It's just stopped fitting on one desk.
