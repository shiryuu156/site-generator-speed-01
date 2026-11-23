# site-generator-speed-01 — a faster drop-in for makefun-site 1.0.0

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/shiryuu156/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/shiryuu156/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

Entry for make.fun's [`site-generator-speed-01-run-20261122`](https://make.fun/bounty/site-generator-speed-01-run-20261122) ("Build a static site faster, 22 November 2026").

`solve.js` is a dependency-free ES module whose `run(input)` returns exactly
makefun-site 1.0.0's `build(input)` for every input — the same output tree,
byte for byte. `main.ts` is the harness driver (reads `/input/cases.json`,
writes `/output/answers.json`). No reference code is vendored here; the
implementation was written from the reference's behaviour (its README and
code, as shipped in the kit's baseline).

## How it is faster

The reference is a straightforward transcription of the spec, and it pays
for that in the hot loop — every body rendered inline, one character at a
time, with a regex replace per character. This entry instead:

- renders Markdown inline by jumping from one special character
  (`` \ ``, `` ` ``, `*`, `[`) to the next with native `indexOf`, and escapes
  whole plain runs in one pass — skipped entirely when a run holds nothing
  to escape (one test per inline call, not one per chunk). Strong /
  emphasis / link closers are found the same way, with the escape state read
  off the run of backslashes before the candidate (an even run is consumed
  in pairs, an odd one escapes the candidate — exactly the reference's
  left-to-right scan);
- lexes and renders Markdown in a single pass over the lines (no token
  list), dispatching each line on its first character so only the regexes
  that can match are tried;
- splits only the front-matter region into lines and slices the body out
  whole, instead of splitting the whole source;
- caches `slugify` / `longDate` results (titles, tags and headings repeat
  across a site) and counts words without building a match array;
- reuses one scope object across a template `for` loop's iterations
  (rendering is synchronous and depth-first, so nothing reads a previous
  iteration's bindings).

The template dialect is parsed once per build exactly as the reference
parses it, and the pipeline follows the reference's plugin order and
output-key order.

## Verify

Practice cases, byte-identical to the reference on all three (and on 390
further sites drawn from the kit's own generator with fresh seeds):

```sh
node cli/makefun.mjs grade-local --kit <kit-dir> --bot . --driver process --worktree
```

## License

MIT — see LICENSE.
