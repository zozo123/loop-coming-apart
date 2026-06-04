# The Loop Is Coming Apart

A short essay on the future of the dev cycle. [jj](https://github.com/jj-vcs/jj),
[Crabbox](https://crabbox.sh), [Depot](https://depot.dev), and
[Blacksmith](https://www.blacksmith.sh) are all making the same promise — *zero
distance between changing your code and knowing if it works, on compute that
isn't yours and isn't precious.* The edit-save-run loop is unbundling into
networked parts, and **the diff is becoming the thing that moves.**

🔗 **Read it:** [zozo123.github.io/loop-coming-apart](https://zozo123.github.io/loop-coming-apart/)

## The argument in one table

| Concern    | Used to be                  | Becoming                       | Who's building it              |
|------------|-----------------------------|--------------------------------|--------------------------------|
| Edit       | local file                  | local file (still)             | you                            |
| Version    | freeze the diff first       | the diff is already an object  | jj                             |
| Transport  | n/a — it's right here       | sync the diff to compute       | Crabbox                        |
| Compute    | your laptop                 | leased ephemeral runner        | Depot, Blacksmith              |
| Warmth     | whatever your disk had      | engineered cache strategy      | Depot, Blacksmith, field guide |
| Isolation  | none — same OS              | a fresh sandbox per run        | islo.dev, Testbox, E2B         |

## The tell

Two companies — Depot and Blacksmith — independently *reverse-engineered the
GitHub Actions cache protocol* and each wrote a blog post about it. When cache
restore is the long pole of every fresh-runner build, **warmth becomes the
product.** That's the [field guide's](https://github.com/zozo123/gha-cache-field-guide)
whole point: CI caching is not one cache.

## Build

No build step. A single self-contained `index.html` published via GitHub Pages.
Prose source lives in [`POST.md`](./POST.md).

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## License

MIT — words and code.
