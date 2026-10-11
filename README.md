# Grida docs

The map of Grida's documentation: where docs are written, where they are
published, and how a change gets from one to the other. This repo holds no
docs pages itself.

Grida publishes **one docs site**, [grida.co/docs](https://grida.co/docs), built
from [gridaco/grida](https://github.com/gridaco/grida). Some products keep their
docs in their own repo, next to their code; the site publishes them too. Those
repos are **docs sources**.

## Topology

| Repo                                              | Writes                                     | Published at                                 | Version published                                 |
| ------------------------------------------------- | ------------------------------------------ | -------------------------------------------- | ------------------------------------------------- |
| [gridaco/grida](https://github.com/gridaco/grida) | `docs/**` — Grida's own docs; **the site** | `grida.co/docs/**`                           | `main`, on merge                                  |
| [gridaco/fx](https://github.com/gridaco/fx)       | `docs/guide/`                              | [grida.co/docs/fx](https://grida.co/docs/fx) | the `@grida/fx` release the `grida` CLI ships     |
| [gridaco/fx](https://github.com/gridaco/fx)       | `docs/wg/`                                 | `grida.co/docs/wg/fx`                        | `main`, synced weekly                             |

The authoritative list of sources is the site's
[`apps/docs/sources.json`](https://github.com/gridaco/grida/blob/main/apps/docs/sources.json);
this table follows it. The engine's working-group docs
([gridaco/nothing](https://github.com/gridaco/nothing/tree/main/docs/wg)) are
not a source yet.

Private repos are never sources. A private product's public docs are written in
gridaco/grida's `docs/`.

## How a change is published

```
source repo                     gridaco/grida (the site)                 grida.co/docs
───────────                     ────────────────────────                 ─────────────
edit docs/… ── PR ── source     pin moves in sources.lock.json   build fetches each source
               check (CI)  ──▶  (weekly sync PR, or the CLI  ──▶ at its pinned commit and
                                 release that bumps @grida/fx)   publishes it
```

- **The source owns the content.** Pages and their images live only in the
  source repo and its history.
- **The site owns the pin.** The site repo stores only the commit each source is
  published at, never its pages or assets.
- **The same rules run three times:** in the source's PRs, when a pin moves, and
  in the site build.

The model, with its invariants and the alternatives considered:
[Docs sources (WG)](https://grida.co/docs/wg/platform/docs-sources).

## Rules for a source

- Links are ordinary relative links that work on GitHub.
- Images live inside the published directory, beside their page, referenced
  with markdown syntax (`![alt](./img/shot.png)`), not `<img src>`.
- `png`, `jpg`, `gif`, `webp`, `svg` or `avif`, at most 500 KiB each. No video
  files: host video and link to it.
- Plain markdown, as GitHub renders it.
- Never move a release tag or force-push a tracked branch.

Details and commands:
[the site's contributing guide](https://github.com/gridaco/grida/blob/main/apps/docs/CONTRIBUTING.md#docs-from-other-repos-docs-sources).

## Adding a source

1. In gridaco/grida, declare it in `apps/docs/sources.json`, then run
   `pnpm --filter docs sources:sync <id>` and commit the lock.
2. If it needs a sidebar entry, add it to `apps/docs/sidebars.js`.
3. Add its repo to the sync App token's `repositories` in
   `.github/workflows/docs-sources.yml`.
4. In the source repo, add the check to CI:

   ```yaml
   jobs:
     docs:
       uses: gridaco/docs/.github/workflows/source-check.yml@main
   ```

5. Add it to the table above.
