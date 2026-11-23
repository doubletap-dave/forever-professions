# GitHub Actions CI/CD & Publishing Guide

This repository uses **two complementary GitHub Actions workflows** following the official [Polestar guide authors documentation](https://polestar.guide/docs/authors).

---

## 1. `.github/workflows/pack.yml` — Lint, Playtest & GitHub Releases

Based on Polestar's official pack template (`tools/templates/pack/.github/workflows/pack.yml`).

- **Triggers:**
  - Every `push` to any branch (`**`) and every `pull_request` → runs `check` job (`lint` + `playtest`).
  - Every `push` of a version tag (`v*`, e.g. `v0.2.0`) → runs `check` (lint, playtest, `forge release pack`, upload zip artifact) followed by `release` (attaches `dist/*.zip` to a GitHub Release).
- **What `check` does:**
  1. Checks out this repo with `contents: read`.
  2. Sets up Node 24 and installs `lua5.1`.
  3. Clones `https://github.com/fitzforge/polestar.git`, runs `npm link`, compiles `forge` (`cargo build --release --locked -p forge`, with Cargo caching), downloads Forever `1.60.1.71037` client tables (`wago`) and `QuestieDB-1.1.5-Forever.zip`, stages the bundled Forever TDB extract, and builds both `data/forever` (`forge data build --offline --questie ...`) and `data/forever/professions.json` (`forge data professions`).
  4. Runs `forge guide lint --format github` (emits inline annotations on PRs).
  5. Runs `polestar playtest` (simulates every guide end-to-end across factions/races/classes).
  6. On `v*` tags: runs `forge release pack` and uploads `dist/*.zip` as an artifact.
- **What `release` does (tags only):**
  - Runs with `contents: write` only after `check` succeeds, creating a GitHub Release with `gh release create "$GITHUB_REF_NAME" dist/*.zip --title "$GITHUB_REF_NAME" --generate-notes`.
- **Secrets needed:** None (uses the automatic `GITHUB_TOKEN`).

---

## 2. `.github/workflows/polestar.yml` — Publish to `polestar.guide`

Based on the official workflow in [Publish from GitHub Actions](https://polestar.guide/docs/authors#publish-from-github-actions).

- **Trigger:** `push` of any `v*` tag (e.g. `v0.2.0`).
- **Permissions:** `contents: read` only, serialized via `concurrency: group: polestar-publish`.
- **What it does:**
  1. Checks out the repository (`actions/checkout` pinned by commit SHA, `persist-credentials: false`).
  2. Extracts the annotated tag message as release notes (up to 5,000 bytes).
  3. Zips **source files only** (`pack.json`, `legs.json`, `retired.json`, `LICENSE*`, `README*`, `*.guide`, `*.tga`, `*.blp`) while excluding `dist/*`, `drafts/*`, `out/*`, `node_modules/*`, and dotfiles.
     - *Important:* polestar.guide rejects any `.lua` file outside `locales/<locale>/`. Always keep compiled addon output in `dist/` (gitignored).
  4. Uploads `guide.zip` via `POST https://polestar.guide/api/packs/forever-professions/versions` with `Authorization: Bearer $POLESTAR_TOKEN`, `version=${GITHUB_REF_NAME#v}`, and `changelog`.
  5. Polls `GET .../versions/<version>` every 10 seconds (up to 10 minutes) until `status` is `ready` (passes → VirusTotal malware scan → live) or `failed` (prints JSON build report and fails the run).

### Required One-Time Setup for `polestar.yml`

1. **Enable 2FA** on your `polestar.guide` account (`Account settings > Sign-in & security`).
2. **Generate a publish token** scoped to this pack:
   - Go to `Your guides > forever-professions > Publishing > Create a token for this guide`.
   - Copy the token (shown once; expires in 90 days by default, up to 1 year max).
3. **Add the repository secret** in GitHub:
   - `Settings > Secrets and variables > Actions > New repository secret`
   - Name: `POLESTAR_TOKEN`
   - Value: *(the token from step 2)*

---

## 3. Release Checklist

1. Verify locally:
   ```powershell
   forge guide lint
   polestar playtest
   forge release pack
   ```
2. Bump `"version"` in `pack.json` (must **exactly** match the tag without `v`, e.g. `"0.2.0"` ↔ `v0.2.0` — mismatches are refused by polestar.guide).
3. Commit and push to `main`.
4. Create and push an **annotated** tag (its message becomes the polestar.guide changelog):
   ```powershell
   git tag -a v0.2.0 -m "Add multi-profession leveling rules, Fishing + Cooking paired route, and CI/CD"
   git push origin v0.2.0
   ```
5. Note on retired/removed routes: polestar.guide refuses automated API uploads that remove existing guide IDs players may be following. Never rename or delete published guide IDs without using `retired.json` or uploading via the studio confirmation UI.
