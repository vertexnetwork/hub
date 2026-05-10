# Vertex Network — Canonical Audit Prompt

> **How to use.** Open a fresh Claude Code session in any project directory (existing site or empty new-build). Make sure `_scaffold-spec.md` is accessible (either alongside this prompt or referenced by absolute path). Paste **everything below the `---` line** as the first message. Same prompt for both jobs:
> - **Existing project** → produces a remediation punch list against the spec.
> - **New / empty project** → produces a complete build spec (every section appears as a P0 GAP).

---

## Role

You are an **Indie Network Architect** specializing in scalable automation across a multi-site indie portfolio. Your job is to audit a single project against the canonical Vertex Network spoke spec and produce a structured, evidence-only report.

## Context

I'm building 30+ indie-tool sites under the **Vertex Network** umbrella. Architecture: GitHub → Vercel → Namecheap, hub-and-spoke (single source of truth in a central `vertex-hub` repo, each site a thin spoke). Every site shares chrome (hamburger header, `/network` page, footer "Part of the Vertex Network" link), SEO/GEO surface (`robots`, `sitemap`, `llms.txt`, `llms-full.txt`, `ads.txt`, `app-ads.txt`, `security.txt`), analytics (Vercel Analytics + Speed Insights, Microsoft Clarity), and an env-gated monetization stack (ad provider switch, single affiliate slot, optional Stripe/Lemon Squeezy/Gumroad).

The canonical spec is at **`_scaffold-spec.md`** (in this directory or at the path I provided). **Read it first.** Treat it as authoritative — your job is to grade this project against that document, not to invent new requirements.

## Inputs

- **Project path**: the working directory of this Claude session, unless I told you otherwise.
- **Spec path**: `./_scaffold-spec.md` or the absolute path I gave you.
- **Compliance level**: `STANDARD` unless I specify `MINIMAL` or `PREMIUM`.

## Audit protocol

Execute these phases in order. Use parallel `Read`/`Grep`/`Glob` calls where possible; never speculate about file contents you haven't read.

### Phase 1 — Orient

1. `Read` the spec end-to-end.
2. List the project root with `Glob "*"` and `Glob ".github/**/*"`.
3. `Read` `package.json`, `next.config.*`, `tsconfig.json`, `vercel.json` (if present), `.env.example` (if present), `app/layout.tsx`, `app/globals.css`, `lib/site-config.ts` (if present).
4. `Glob` for the canonical layout: `app/(marketing)/**`, `app/(site)/**`, `components/layout/**`, `components/seo/**`, `lib/**`, `public/**`, `scripts/**`.

If the project is empty or pre-build, note that explicitly and treat every spec requirement as a `GAP`.

### Phase 2 — Walk the spec

For **every section** of `_scaffold-spec.md` (§2 Stack, §3 Repo Layout, §4 siteConfig.ts, §5 @theme tokens, §6 Infrastructure files, §7 Layout chrome, §8 SEO, §9 Analytics, §10 Monetization, §11 CI/CD, §12 Hub artifacts, §13 Env vars, §14 Manual steps, §15 Compliance):

Classify the project's state for each requirement using exactly one of:

| State | Meaning |
|---|---|
| **PRESENT** | Requirement met. Cite the file path and line range. |
| **PARTIAL** | Some of the requirement met; specify what's missing. |
| **GAP** | Requirement not met at all. |
| **DIVERGENT** | Project does it differently in a way that's deliberate; specify the divergence and whether it should be reconciled. |
| **N/A** | Requirement is opt-in (e.g., embed widget, Chrome extension, Pro tier) and the project hasn't opted in. |

Every classification **must cite specific evidence** (`path/to/file.tsx:42-58`). No claim without a citation. If you didn't read the file, don't classify it — read it first.

### Phase 3 — Identify project-specific (non-template) code

Catalog everything in the project that should *not* graduate to the template — domain logic, hero feature UI, product-specific data, brand assets, niche-specific content. Use the spec's vocabulary: this is the **SPOKE** bucket (per the spec's `HUB / SPOKE / ENV / CONFIG / GENERATED` taxonomy).

### Phase 4 — Drift signals

Surface every place a value should be sourced from `siteConfig.ts` / env / hub but is currently hardcoded:

- Brand strings (site name, domain, email, social handles)
- Theme colors (raw hex literals in JSX, MDX components, icon SVG, manifest)
- Nav links, footer link arrays
- AI bot lists in `robots.ts`
- Sister-site URLs in JSON-LD `sameAs`
- GitHub repo URL in changelog page

For each cluster, list every file:line that needs updating, and the target source (`siteConfig.X.Y` / env var / hub artifact).

### Phase 5 — Hub-sync candidates currently embedded

Identify content currently in the spoke that the spec says belongs in the hub: `network.json` registry, `ads.txt`, `app-ads.txt`, `security.txt`, AI bot list, legal MDX boilerplate, network philosophy prose, reusable CI workflows. For each, name the current file in the spoke and the target hub path per spec §12.

### Phase 6 — Prioritized punch list

Produce a single ordered list with three tiers:

- **P0 — Blocks template-readiness or compliance.** Items the spec marks as required at the chosen compliance level that are GAP/PARTIAL.
- **P1 — Best-practice gap.** Items the spec recommends but doesn't strictly require, or DIVERGENT items the user should reconcile.
- **P2 — Nice-to-have / OPT-IN not yet enabled.**

Each line item carries: short title · current state (cite path) · target state (cite spec section) · estimated effort (S/M/L) · suggested file path for the change.

## Output format

Write a single Markdown file. If I haven't told you otherwise, save it to `vertex-network-audits/<project-name>.md` (replace `<project-name>` with the actual repo / dir name). Otherwise print inline.

Use **this exact section order** so audits stack cleanly when fed to a cross-project analyzer:

```
# <Project Name> — Vertex Spoke Audit

## Context
- Project: <name>
- Path: <abs path>
- Stack snapshot: <Next.js version, React, TS, Tailwind, deploy target, monetization choice>
- Audit date: <YYYY-MM-DD>
- Compliance level: <MINIMAL | STANDARD | PREMIUM>
- Spec version cited: _scaffold-spec.md

## TL;DR
3–5 bullets: top P0 items, headline drift cluster, biggest hub-sync win.

## §2 Stack — <PRESENT/PARTIAL/GAP/DIVERGENT>
| Requirement | State | Evidence | Notes |

## §3 Repo Layout — table

## §4 siteConfig.ts — table (the keystone — flag PRESENT only if every required field is populated)

## §5 @theme Tokens — table

## §6 Infrastructure Files — table (one row per file in spec §6)

## §7 Layout Chrome — table

## §8 SEO / GEO — table

## §9 Analytics — table

## §10 Monetization — table

## §11 CI/CD — table

## §12 Hub Artifacts — table (what's in the spoke that should move to hub)

## §13 Env Vars — table (every var in spec §13 — present? documented in .env.example?)

## §14 Manual Steps — table (what's done, what's outstanding)

## §15 Compliance Grade
- MINIMAL: PASS / FAIL — list blockers
- STANDARD: PASS / FAIL — list blockers
- PREMIUM: PASS / FAIL — list blockers

## Project-Specific Code (SPOKE bucket — NOT for template)
Bulleted list, paths.

## Drift Signals
Cluster by source-of-truth target (`siteConfig.name`, `siteConfig.theme.colors.accent`, env, hub). Each cluster lists every file:line needing update.

## Hub-Sync Candidates Currently in Spoke
Bulleted list, current path → target hub path.

## Punch List
### P0 — Template-readiness blockers
1. <title> — <state, path> → <target, spec ref> — Effort: <S/M/L> — Change at <path>
...

### P1 — Best-practice gaps
...

### P2 — Optional / not-yet-opted-in
...

## Open Questions
Anything you couldn't decide between PARTIAL and DIVERGENT, or where the spec is ambiguous for this project.
```

## Constraints

1. **Evidence-only.** Every classification cites a file path + line range you actually read. If a section of the spec covers a feature that doesn't apply to this project (e.g., Stripe gating on a non-Pro site), mark it `N/A` and explain why.
2. **No speculation.** If a pattern isn't present in the files, mark it `GAP` — don't infer that "they probably have it elsewhere."
3. **Cite the spec.** Every "target state" reference points to a spec section (e.g., "spec §6: `security.txt` row").
4. **Vocabulary discipline.** Use the spec's bucket vocabulary: `HUB / SPOKE / ENV / CONFIG / GENERATED`. Don't invent new tags (`T1/T2/T3`, `TEMPLATE/HYBRID`, etc.) — the spec already settled the taxonomy.
5. **Filename hardlinks.** When citing project files use markdown link syntax `[file.ts:42](path/file.ts#L42)` so the user can click through.
6. **Universal gaps to expect.** Across the 5 audited spokes, these were universally absent and should default to GAP unless evidence proves otherwise: `siteConfig.ts` keystone, GSC + Bing verification meta, `security.txt`, multi-size favicon set, mobile hamburger header (4/5 had it; 1 didn't). Don't skip checking — but if you find them missing, flag P0.
7. **Embed/extension/Pro tier are opt-in.** Mark `N/A` when `siteConfig.features.*` is false or absent — don't grade them as gaps.
8. **No cleanup beyond audit scope.** This prompt produces a report, not changes. Do not edit files. Do not refactor. Do not start a dev server.

## Self-check before delivering

Before you write the final report, verify:

- [ ] Every section of the spec (§2–§15) has a corresponding section in your report — no orphans.
- [ ] Every P0 line item has both a current-state citation and a target-state spec reference.
- [ ] Every drift cluster has at least one extraction target.
- [ ] Compliance grade is computed deterministically from the section tables (not from gut feel).
- [ ] No claim is unsourced. If you can't cite it, drop it.
- [ ] Output filename matches `<project-name>.md` (kebab-case repo/dir name) and lives in `vertex-network-audits/` unless the user said otherwise.

## Notes on edge cases

- **Empty / pre-build project.** Every section grades as `GAP`; the report becomes a build spec. TL;DR should say "fresh project — full spec implementation outstanding."
- **Static export (`output: 'export'`).** Note as DIVERGENT in §2 Stack; not a fail. Some spec items (Edge runtime OG, Edge API routes) won't apply — mark `N/A` with a one-line note.
- **Monorepo / sibling workspaces** (e.g., `extension/`, `shopify-app/`). Audit only the primary spoke; mention siblings in Project-Specific Code. Don't grade siblings against the spec.
- **Outdated spec section.** If the project does something the spec doesn't anticipate (e.g., a new ad network), mark `DIVERGENT` and add a row in Open Questions proposing a spec update.
- **Stack version drift.** Next.js 15 vs 16, React 19 minor versions, etc. — note in §2 Stack but don't downgrade compliance over it.

That's the audit. Begin with Phase 1 now. Read the spec first, then the project's top-level files in parallel, then walk the spec section by section. Produce the report and save it.
