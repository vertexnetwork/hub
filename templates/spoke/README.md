# `templates/spoke/`

Files each spoke repo copies in from the hub. Drop them into the spoke's repo root, preserving the directory structure (so `.github/workflows/ci.yml` lands at `<spoke>/.github/workflows/ci.yml`).

| File | Where it lands in the spoke |
|---|---|
| `.github/workflows/ci.yml` | `<spoke>/.github/workflows/ci.yml` (calls hub reusable workflow) |
| `.github/workflows/sync-from-hub.yml` | `<spoke>/.github/workflows/sync-from-hub.yml` (listens for hub dispatches) |
| `.env.example` | `<spoke>/.env.example` |
| `.gitignore` | `<spoke>/.gitignore` |
| `.gitattributes` | `<spoke>/.gitattributes` |
| `.nvmrc` | `<spoke>/.nvmrc` |
| `lighthouserc.json` | `<spoke>/lighthouserc.json` |
| `vercel.json` | `<spoke>/vercel.json` |

After copying, commit and push to the spoke. The first hub propagation event after the spoke's `sync-from-hub.yml` lands will fan in `network.json`, `ai-bots.json`, `ads.txt`, `app-ads.txt`, and `humans.txt`.
