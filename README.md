# The Loop Is Coming Apart

A short essay on the future of the dev cycle: how [jj](https://github.com/jj-vcs/jj),
[crabbox](https://github.com/openclaw/crabbox), and the
[GitHub Actions cache field guide](https://github.com/zozo123/gha-cache-field-guide)
— three unrelated projects — are quietly arguing for the same future. The
edit-save-run loop is unbundling into networked parts, and **the diff is becoming
the thing that moves.**

🔗 **Read it:** [zozo123.github.io/loop-coming-apart](https://zozo123.github.io/loop-coming-apart/)

## The argument in one table

| Concern    | Used to be                  | Becoming                            |
|------------|-----------------------------|-------------------------------------|
| Edit       | local file                  | local file (still)                  |
| Version    | freeze the diff first (Git) | the diff is already an object (jj)  |
| Transport  | n/a — it's right here       | sync the diff to compute (crabbox)  |
| Compute    | your laptop                 | leased ephemeral microVM            |
| Warmth     | whatever your disk had      | an engineered cache strategy        |

## Build

No build step. It's a single self-contained `index.html` published via GitHub
Pages. The prose source lives in [`POST.md`](./POST.md).

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## License

MIT — words and code.
