# vertex-network-hub

Source of truth for the **Vertex Network** — a hub-and-spoke architecture across an indie portfolio of tool sites.

## Two ways spokes use this hub

### 1. As an audit reference (zero spoke install)

Open a spoke project locally → paste [`docs/_canonical-audit-prompt.md`](docs/_canonical-audit-prompt.md) into Claude → Claude fetches this hub over the network, audits the spoke, produces a punch list. **The spoke gets nothing copied in.** That's the primary loop.

### 2. As a live data source (one-file spoke install, optional)

If you want hub edits (like a new entry in `network.json`) to land in a spoke automatically, copy [`templates/spoke/.github/workflows/sync-from-hub.yml`](templates/spoke/.github/workflows/sync-from-hub.yml) into that spoke's `.github/workflows/`. One file. The spoke now opens auto-merge PRs whenever the hub changes.

Most spokes use both: audit first, install the sync workflow only after the audit confirms it's worth it.

## What's in here

| Path | Purpose |
|---|---|
| [`config/network.json`](config/network.json) | Canonical registry of every spoke site. |
| [`config/ai-bots.json`](config/ai-bots.json) | AI crawler allowlist (GPTBot, ClaudeBot, PerplexityBot, etc.). |
| [`config/env.template.json`](config/env.template.json) | Canonical env var contract. |
| [`config/spokes.json`](config/spokes.json) | Repo names for auto-sync fan-out. Edit when you add/rename a spoke. |
| [`crawler/`](crawler/) | `ads.txt`, `app-ads.txt`, `security.txt`, `humans.txt` — files served at well-known paths on every spoke. |
| [`content/legal/`](content/legal/) | `privacy.mdx`, `terms.mdx` with `{{placeholders}}` filled in per spoke. |
| [`content/network-philosophy.mdx`](content/network-philosophy.mdx) | The "what ties them together" prose. |
| [`templates/spoke/`](templates/spoke/) | Two optional workflow files (sync listener, CI stub). Install only when the audit recommends it. |
| [`docs/_scaffold-spec.md`](docs/_scaffold-spec.md) | **The spec.** What every spoke should look like. |
| [`docs/_canonical-audit-prompt.md`](docs/_canonical-audit-prompt.md) | **The prompt.** Paste into Claude in any spoke folder → audit report. |
| [`docs/_synthesis-report.md`](docs/_synthesis-report.md) | Cross-audit Stay-vs-Go report from the founding spokes. |
| [`docs/SETUP.md`](docs/SETUP.md) | Read first — the (small) one-time setup. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How the hub-and-spoke loop works. |
| [`docs/SITE_LAUNCH_CHECKLIST.md`](docs/SITE_LAUNCH_CHECKLIST.md) | Per-spoke launch runbook (manual steps that can't be automated). |

## Get started

1. [`docs/SETUP.md`](docs/SETUP.md) — what (little) you need to configure.
2. Pick a spoke, open [`docs/_canonical-audit-prompt.md`](docs/_canonical-audit-prompt.md), paste into Claude, get an audit.
3. Work the punch list per-spoke at your own pace.
