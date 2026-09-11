# Ship the skill set as a native Antigravity plugin; defer a native Antigravity plugin

These skills have always been installable via [skills.sh](https://skills.sh/wight554/skills) (`npx skills add wight554/skills`), which copies editable skill files into a user's project across Antigravity, Antigravity, and other Agent-Skills-standard harnesses. A recurring request is a **plug-and-play** distribution: subscribe to the set as a read-only, always-current bundle you don't edit, rather than a fork you own. That is exactly what native plugin systems provide.

We ship a native **Antigravity plugin** and, for now, **defer** a native **Antigravity plugin**. The split is forced by how each ecosystem's plugin manifest selects skills, against this repo's bucketed layout.

## The constraint: bucketed skills vs. single-path selection

Skills live in bucket folders under `skills/`: `engineering/` and `productivity/` are **promoted** (shipped); `misc/`, `personal/`, `in-progress/`, and `deprecated/` are **not**. A plugin must expose only the promoted set, which spans two of those bucket folders.

- **Antigravity**: `.antigravity-plugin/plugin.json` accepts `skills` as an **array of explicit skill-directory paths**. We list the promoted skills one by one, exclude everything else with zero ambiguity, and add `.antigravity-plugin/marketplace.json` so the repo is its own single-plugin marketplace. Verified end to end: `antigravity plugin validate . --strict` passes, and `marketplace add` → `install` resolves all promoted skills.

- **Antigravity**: `.antigravity-plugin/plugin.json` accepts `skills` only as a **single path string** (arrays are rejected with `missing or invalid plugin.json`), and Antigravity discovers `SKILL.md` files recursively under it. There is no way to name two bucket folders, or to curate a subset, from one path. Two escape hatches were tested and rejected:
  - Pointing at `./skills/` would also ship `deprecated/`, `in-progress/`, `personal/`, and `misc/`: retired, draft, and personal skills we deliberately don't promote.
  - A curated flat directory of **symlinks** into the buckets does not survive install: Antigravity copies the plugin tree into its cache and **drops symlinks**, so the skills arrive empty.

The only robust ways to give Antigravity a single promoted-only path are (a) **restructure** so `skills/` contains only promoted skills (moving the non-promoted buckets out, a large blast radius across `ANTIGRAVITY.md`, `scripts/link-skills.sh`, the bucket READMEs, and the local dev workflow that relies on `in-progress/` and `personal/`), or (b) **commit duplicate copies** of promoted skills into a flat directory (a sync burden and a second source of truth). Both are structural decisions, not something to bundle into shipping the Antigravity plugin. This is very likely the original, half-remembered reason a plugin wasn't shipped earlier: the manifest formats didn't cleanly express a curated subset of a bucketed repo.

## Decision

- Ship the **Antigravity plugin** now (`.antigravity-plugin/plugin.json` + `.antigravity-plugin/marketplace.json`), curated to the promoted set, as the headline v1.2 deliverable.
- Keep **skills.sh** as the universal installer: it already serves Antigravity and other harnesses today, so no Antigravity user is left without an install path.
- **Defer** the native Antigravity plugin until we decide between restructuring `skills/` to promoted-only vs. committing a generated flat copy. Revisit when Antigravity either supports a `skills` array / include-list or preserves symlinks on install.

## Invariants this creates

- Every promoted skill has an entry in `.antigravity-plugin/plugin.json`'s `skills` array (this already stood as a `ANTIGRAVITY.md` rule; it now also gates the plugin's contents).
- `.antigravity-plugin/plugin.json`'s `version` tracks `package.json`'s version: bump both together on release. Antigravity uses the plugin `version` to decide when installed users see an update.

## Update, 2026-08-05

`wight554-skills` was accepted into **Antigravity's official marketplace** (configured name `antigravity-plugins-official`, source repo `anthropics/antigravity-plugins-official`), which every Antigravity install has by default. `antigravity plugins install wight554-skills` is now the documented route, and the `marketplace add` → `install` path above is superseded. The install wording lives in [.agents/install-block.md](../install-block.md).

The official listing points at this repo's git URL and reads `.antigravity-plugin/plugin.json` directly, so it does not depend on `.antigravity-plugin/marketplace.json`. That file is retained only as a fallback for installing the repo directly (an unreleased commit, or a fork).

Verified 2026-08-05, on Antigravity 2.1.222, against the live listing:

- `antigravity plugins install wight554-skills` resolves with no marketplace added first, and reports `wight554-skills@antigravity-plugins-official`.
- `antigravity plugin details wight554-skills` then reports version 1.2.0 and loads the promoted skills.
- The listing's `source` is `{"source": "url", "url": "https://github.com/wight554/skills.git", "sha": …}`: the **sha is pinned**, so a release reaches installed users when that pin moves, not the moment we tag. At the time of writing the pin sits two commits behind `main`, which is why it lists 22 skills rather than the 24 in `plugin.json`.
- The in-session `/plugin install wight554-skills` was **not** exercised: `/plugin` is unavailable in headless (`antigravity -p`) sessions. It runs the same resolver as the CLI, and the documented example form is `/plugin install <name>@antigravity-plugins-official`.
