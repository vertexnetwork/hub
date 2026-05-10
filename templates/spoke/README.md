# `templates/spoke/`

Two **optional** workflow files for spokes that want hub automation. **Most spokes don't need to install anything from this folder** — the audit prompt at [`docs/_canonical-audit-prompt.md`](../../docs/_canonical-audit-prompt.md) reads the hub directly over the network and tells you what (if anything) to add.

| File | When to install it | What it does |
|---|---|---|
| `.github/workflows/sync-from-hub.yml` | Only if you want hub changes (e.g., new sites added to `network.json`) to land in this spoke automatically. | Listens for `repository_dispatch` events from the hub and opens an auto-merge PR with the changed files. |
| `.github/workflows/ci.yml` | Only if you want to consume the hub's reusable CI workflow. | Calls `ThatMovieGuyOriginal/vertex-network-hub/.github/workflows/spoke-ci.yml@v1`. Saves you from maintaining a CI pipeline per spoke. |

If you're not sure: **skip this folder entirely**. Run the audit. Decide later. The hub still works — you just won't get auto-sync of `network.json` etc.; you'd update those by hand or via your own automation.
