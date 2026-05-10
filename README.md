# vertex-network-hub

Source of truth for the **Vertex Network** — a hub-and-spoke architecture across an indie portfolio of tool sites.

## What's in here

| Path | Purpose |
|---|---|
| [`config/network.json`](config/network.json) | Canonical registry of every spoke site. Updates fan out to all spokes. |
| [`config/ai-bots.json`](config/ai-bots.json) | AI crawler allowlist (GPTBot, ClaudeBot, PerplexityBot, etc.). |
| [`config/spokes.json`](config/spokes.json) | Repo names for fan-out. **You must edit this** with your actual repo paths. |
| [`crawler/`](crawler/) | `ads.txt`, `app-ads.txt`, `security.txt`, `humans.txt` — files served at well-known paths on every spoke. |
| [`content/legal/`](content/legal/) | Legal boilerplate (`privacy.mdx`, `terms.mdx`) with `{{placeholders}}` filled in per spoke. |
| [`content/network-philosophy.mdx`](content/network-philosophy.mdx) | The "what ties them together" prose that renders on every spoke's `/network` page. |
| [`templates/spoke/`](templates/spoke/) | Files each spoke copies in (CI workflow, `vercel.json`, `lighthouserc.json`, etc.). |
| [`docs/_scaffold-spec.md`](docs/_scaffold-spec.md) | Canonical spec — what a Vertex spoke looks like. |
| [`docs/_canonical-audit-prompt.md`](docs/_canonical-audit-prompt.md) | Re-usable Claude Code prompt that audits any project against the spec. |
| [`docs/_synthesis-report.md`](docs/_synthesis-report.md) | Cross-audit Stay-vs-Go report from the 5 founding spokes. |
| [`docs/SETUP.md`](docs/SETUP.md) | **Read this first** — one-time setup steps for the human. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How the hub-and-spoke loop works, in plain English. |
| [`docs/SITE_LAUNCH_CHECKLIST.md`](docs/SITE_LAUNCH_CHECKLIST.md) | Per-spoke launch runbook (manual steps that can't be automated). |

## How it works (one paragraph)

Push a change to `main` → `.github/workflows/propagate.yml` reads `config/spokes.json` and fires a `repository_dispatch` event at every spoke repo → each spoke's `sync-from-hub.yml` workflow listens for that event, downloads the changed files via raw GitHub URLs, and opens an auto-merge PR → CI gates the merge → Vercel auto-deploys. End-to-end propagation is usually under 5 minutes.

## Get started

1. Read [`docs/SETUP.md`](docs/SETUP.md) — set the `HUB_DISPATCH_TOKEN` secret, edit `config/spokes.json` with real repo paths, tag a `v1` release.
2. Install [`templates/spoke/.github/workflows/sync-from-hub.yml`](templates/spoke/.github/workflows/sync-from-hub.yml) into each spoke repo.
3. Edit `config/network.json` to test propagation; watch the PRs land.
