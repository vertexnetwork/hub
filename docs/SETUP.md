# SETUP

The hub is already built and live. There's almost nothing to configure.

## How you'll actually use this

For each spoke (`tokenmath`, `shopifont`, `kdpcover`, `etsymargin`, `captionsnap`, or any future site):

1. Open that project's folder locally in a fresh Claude Code session.
2. Open [`docs/_canonical-audit-prompt.md`](_canonical-audit-prompt.md) in this hub repo's GitHub UI.
3. Copy everything below the `---` line.
4. Paste it as your first message to Claude in the spoke session.
5. Claude fetches this hub remotely (no files copied into the spoke), reads the spoke locally, produces an audit report and saves it to `vertex-network-audits/<spoke>.md`.

That's the whole loop for the audit half. Spokes get **zero hub files installed** for the audit. Everything Claude needs is read over the network.

---

## Optional: enable auto-sync (only when you want it)

The audit will tell you what each spoke is missing — including whether you want hub-managed files (like `network.json`, `ads.txt`) to land in the spoke automatically.

If yes for a given spoke:

1. Set `HUB_DISPATCH_TOKEN` once on the hub (steps below) — one-time, covers all spokes.
2. Copy [`templates/spoke/.github/workflows/sync-from-hub.yml`](../templates/spoke/.github/workflows/sync-from-hub.yml) into that spoke's `.github/workflows/`. **One file. That's the entire spoke install.**
3. Push the spoke. Done — it's now in the sync loop.

If no: skip everything below. The hub still serves as the spec source for audits.

### Setting `HUB_DISPATCH_TOKEN` (one-time, ~3 minutes)

The hub needs a Personal Access Token to fire `repository_dispatch` events at spokes. Without this, only audits work; auto-sync doesn't.

1. **Create the PAT** → https://github.com/settings/personal-access-tokens/new
   - Name: `vertex-hub-dispatch`
   - Expiration: 1 year
   - Repository access: "Only select repositories" → tick the spoke repos you want syncable
   - Repository permissions (set to **Read and write**): Contents, Pull requests, Actions
   - Generate → copy the token
2. **Save the token as a hub secret** → https://github.com/ThatMovieGuyOriginal/vertex-network-hub/settings/secrets/actions
   - New repository secret
   - Name: `HUB_DISPATCH_TOKEN`
   - Secret: paste the PAT
   - Save
3. **Verify `config/spokes.json`** lists the right repos under `<owner>/<name>` form. Edit if any are wrong.

### Testing the auto-sync loop (after at least one spoke has `sync-from-hub.yml`)

1. Edit `config/network.json` in this hub — add a `"status": "soon"` test entry.
2. Commit + push to `main`.
3. Watch hub Actions — `Propagate to spokes` runs, dispatches events.
4. Watch the spoke's Actions — `Sync from vertex-network-hub` runs, opens a PR.
5. PR diff should show `public/network.json` updated. Don't merge — go back to hub, remove the test entry, push again.

If steps 1–5 work, auto-sync is live.

---

## What you do NOT need to do

- ❌ Copy template files into every spoke. The audit drives per-spoke changes; bulk copy is wrong.
- ❌ Stand up an `@vertex/*` npm monorepo. Future option, not a requirement.
- ❌ Build a `create-vertex-site` CLI yet. The audit prompt + hub URL is the workflow.
- ❌ Migrate spokes before auditing them. Audit first, then prioritize from the punch list.

---

## Troubleshooting

**Audit prompt fails to fetch hub files** → Confirm hub repo is public OR Claude has network access to the raw URLs. If fetching `_scaffold-spec.md` fails, the audit can't proceed. Most likely cause: typo in the hub URL.

**Auto-sync `Propagate to spokes` fails with `Resource not accessible`** → `HUB_DISPATCH_TOKEN` PAT doesn't have access to the target repo. Re-issue with the right repo selection.

**Spoke `sync-from-hub.yml` runs but no PR appears** → Correct if no synced file changed. Force a hub change to test.

**`uses: ThatMovieGuyOriginal/vertex-network-hub/.github/workflows/spoke-ci.yml@v1` fails** → Confirm `v1` tag exists at https://github.com/ThatMovieGuyOriginal/vertex-network-hub/tags (it should — it was tagged at hub init).
