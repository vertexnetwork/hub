# SETUP — what you (Josh) actually need to do

**Read this top-to-bottom in order.** Skip nothing. Each step is small.

---

## Step 1 — Create a GitHub Personal Access Token (PAT)

The hub needs permission to fire `repository_dispatch` events at every spoke. That requires a PAT.

1. Go to https://github.com/settings/personal-access-tokens/new
2. **Token name**: `vertex-hub-dispatch`
3. **Expiration**: 1 year (set a calendar reminder to rotate)
4. **Resource owner**: your account (`ThatMovieGuyOriginal`)
5. **Repository access**: **Only select repositories** — pick all 5 spoke repos (`captionsnap`, `etsymargin`, `kdpcover`, `shopifont`, `tokenmath`).
6. **Permissions** → **Repository permissions** →
   - **Contents**: Read and write
   - **Metadata**: Read-only (auto-selected)
   - **Pull requests**: Read and write
   - **Actions**: Read and write
7. Click **Generate token**. **Copy the token** — you won't see it again.

> If a fine-grained PAT feels fiddly, a classic PAT with `repo` and `workflow` scopes works too. Fine-grained is more locked-down and recommended.

---

## Step 2 — Add the PAT as a hub secret

1. Go to https://github.com/ThatMovieGuyOriginal/vertex-network-hub/settings/secrets/actions
2. Click **New repository secret**
3. **Name**: `HUB_DISPATCH_TOKEN`
4. **Secret**: paste the PAT from Step 1
5. **Add secret**

---

## Step 3 — Verify `config/spokes.json` has the right repo names

Open [`config/spokes.json`](../config/spokes.json). Each `repo` should be `<your-github-username>/<actual-repo-name>`. Edit any that are wrong.

The defaults assume:

```json
ThatMovieGuyOriginal/captionsnap
ThatMovieGuyOriginal/etsymargin
ThatMovieGuyOriginal/kdpcover
ThatMovieGuyOriginal/shopifont
ThatMovieGuyOriginal/tokenmath
```

If any of those don't match your actual repo names, fix them. Then commit + push.

---

## Step 4 — `v1` tag (already done)

The `v1` tag is already in place — spokes can reference `@v1` immediately. Verify at https://github.com/ThatMovieGuyOriginal/vertex-network-hub/tags.

Later, when you change `spoke-ci.yml`:
- **Bug fixes / additive changes** → force-move the `v1` tag: `git tag -f v1 && git push -f origin v1`
- **Breaking changes** (input renames, behavior changes) → cut a fresh `v2`, migrate spokes one at a time.

---

## Step 5 — Install `sync-from-hub.yml` into one spoke first (pilot)

Pick **tokenmath** as the pilot (smallest divergence per the synthesis report).

1. Open the tokenmath repo locally.
2. Copy this file:
   - From: `vertex-network-hub/templates/spoke/.github/workflows/sync-from-hub.yml`
   - To:   `tokenmath/.github/workflows/sync-from-hub.yml`
3. Commit + push to tokenmath's `main`.

That's it for the spoke side. The workflow will now listen for hub dispatches.

---

## Step 6 — Test the loop

1. In `vertex-network-hub`, open `config/network.json`.
2. Add a **`"status": "soon"`** test entry near the bottom of `sites[]` (any plausible-looking placeholder is fine).
3. Commit + push to `main`.
4. Go to https://github.com/ThatMovieGuyOriginal/vertex-network-hub/actions — you should see **Propagate to spokes** running.
5. Within a minute, go to https://github.com/ThatMovieGuyOriginal/tokenmath/actions — you should see **Sync from vertex-network-hub** running.
6. When that finishes, a PR titled `chore: sync from vertex-network-hub` should appear at https://github.com/ThatMovieGuyOriginal/tokenmath/pulls
7. The PR's diff should show `public/network.json` changing to include your test entry.
8. **Don't merge it** — go back to the hub, remove the test entry, push again. The PR should auto-close (or close it manually) once a clean sync replaces it.

If all 8 steps work, the loop is live.

---

## Step 7 — Roll out to the other 4 spokes

Repeat **Step 5** for `captionsnap`, `etsymargin`, `kdpcover`, `shopifont`. Just copy `sync-from-hub.yml` into each spoke's `.github/workflows/`.

**Note**: each spoke needs a `public/` directory for the synced files to land in. If a spoke doesn't have one yet, create an empty `public/.gitkeep`.

---

## Step 8 — (Later, when ready) Re-audit each spoke against the new spec

For each spoke:

1. Open it in Claude Code.
2. Paste [`docs/_canonical-audit-prompt.md`](_canonical-audit-prompt.md) verbatim, telling Claude where the spec lives (this hub's `docs/_scaffold-spec.md`, or copy that file into the spoke first).
3. Save the output back to `vertex-network-audits/<spoke>.md`, overwriting the old per-project audit.
4. Compare old vs new — that's your migration backlog.

Don't tackle migration until at least 3 spokes are re-audited. The cross-audit P0 union is what you actually want to fix.

---

## What you do NOT need to do (yet)

- Build a CLI scaffolder (`create-vertex-site`).
- Stand up an `@vertex/*` npm package monorepo.
- Refactor any spoke yet — the hub is the foundation; spokes migrate one at a time.
- Wire up Lighthouse CI on every spoke — it ships in `templates/spoke/lighthouserc.json` for when you want it.

---

## Troubleshooting

**"Propagate to spokes" workflow fails with `Resource not accessible`** → the `HUB_DISPATCH_TOKEN` PAT doesn't have the right repo access. Re-issue with the exact 5 spoke repos selected.

**Spoke's `sync-from-hub.yml` runs but no PR appears** → that's correct if no synced file changed. PRs only open on actual diffs. Force a change by editing `config/network.json` again.

**`uses: ThatMovieGuyOriginal/vertex-network-hub/.github/workflows/spoke-ci.yml@v1` fails** → you skipped Step 4 (tagging `v1`). Run `git tag v1 && git push origin v1`.

**The PR auto-merge label doesn't actually auto-merge** → that's by design (no `auto-merge.yml` ships yet). Add one later, or merge by hand. The label is just for filtering.
