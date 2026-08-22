# Entropy audit — squz.com

Full audit of the GitHub Pages site that hosts Squz product landings and App Store legal pages. Hygiene was invoked in full mode after the structural passes.

## Executive summary

- **Snapshot:** `/Users/marcelo/work/github.com/squz/squz.com`
- **Branch:** `main` (tracks `origin/main`; GitHub default branch is `main`)
- **HEAD:** `b6d968aa7ebb3cc8ae1ed5961d7e9f160839d4af` — `Add MultiMaze 2 privacy policy page (#1)`
- **Initial dirty state:** clean (`git status --porcelain=v1 -b` showed only `## main...origin/main`)
- **Date:** 2026-08-22 (campaign). Command evidence captured 2026-08-21 UTC against the live site.
- **Scope:** all tracked HTML, README, CNAME, and `esfera/media/` references. No application source, tests, or CI live in this repository.
- **Exclusions:** none generated or vendored. Binary media under `esfera/media/` was inventoried for reachability, not visually reviewed. App Store Connect metadata is outside the repo; it is cited only where it is a competing authority for URLs this site is meant to serve.
- **Languages in tree:** HTML and inline CSS only. No Python, Go, Rust, C/C++, SQL, or committed shell. Companions read: `web-development.md`, `journeys.md`. Repo has no `AGENTS.md` / `CLAUDE.md`.
- **Headline mechanism:** the repository evolved from a one-app Your World landing into a three-product GitHub Pages host, but the root document, README, contact paths, and App Store metadata still treat `squz.com` as a single Your World page. Legal pages are independent HTML copies, so contact and identity already drifted. Nothing in CI checks that shipped URLs resolve.
- **Highest-consequence findings:** live homepage screenshot 404; Your World COPPA contact mailbox has no mail exchanger; App Store Your World privacy URL still points at a DNS-dead `blog.squz.com`; MultiMaze's App Store developer website 404s on this host.
- **Unverified residue:** whether `hello@squz.com` is accepted by some non-DNS path; whether the 2016 Your World binary still performs Facebook/Twitter sharing; whether App Store Connect has a support URL distinct from these pages; whether MultiMaze 2 is a rename of listing `id355300331` or a successor not yet public under that name.

## Scope and exclusions

Tracked tree (14 files):

| Path | Role |
|---|---|
| `CNAME` | GitHub Pages custom domain `squz.com` |
| `README.md` | One-line description, Your World only |
| `index.html` | Your World marketing landing (`https://squz.com/`) |
| `privacy.html` | Your World privacy policy |
| `support.html` | Your World FAQ |
| `esfera/index.html` | Esfera Chess marketing |
| `esfera/privacy.html`, `esfera/support.html` | Esfera legal/support |
| `esfera/media/*` | Screenshots + intro video (including unused `leaderboard.png`) |
| `multimaze/privacy.html` | MultiMaze 2 privacy policy |

Skipped as non-code: PNG/MP4 pixel content. No `node_modules`, generated bundles, fixtures, or vendor trees exist.

Live GitHub Pages (`https://squz.com/`, source `main` `/`, HTTPS enforced, `www.squz.com` → `https://squz.com/`) is the shipped path. There is no local build.

## Commands run

Shipped-path probes hit production GitHub Pages and public App Store listings. Auxiliary commands inspect git/GitHub metadata and local files. No analyzers were installed.

| Command | Version / notes | Exit | Relevant output | Path | Limitations |
|---|---|---|---|---|---|
| `git status --porcelain=v1 -b` | git 2.55.0 | 0 | `## main...origin/main` (clean) | aux | — |
| `git rev-parse HEAD` | git 2.55.0 | 0 | `b6d968aa7ebb3cc8ae1ed5961d7e9f160839d4af` | aux | — |
| `git log --oneline --all --decorate` | git 2.55.0 | 0 | 15 commits; default history from 2025-06-29 | aux | — |
| `git ls-files` | git 2.55.0 | 0 | 14 paths listed above | aux | — |
| `gh repo view --json …` | gh 2.97.0 | 0 | public, no license, description “Landing page and privacy policy for Your World by Squz”, default `main` | aux | — |
| `gh api repos/squz/squz.com/pages` | gh 2.97.0 | 0 | `status=built`, `cname=squz.com`, `https_enforced=true`, `custom_404=false`, source `main` `/`, cert expires 2026-09-28 | shipped metadata | Certificate expiry is GitHub-managed |
| `gh api repos/squz/squz.com --jq security_and_analysis` | gh 2.97.0 | 0 | dependabot updates, secret scanning, push protection all `disabled` | aux | Low value on a static HTML repo |
| `gh api …/branches/main/protection` | gh 2.97.0 | 404 | `Branch not protected` | aux | — |
| `gh api …/actions/workflows` | gh 2.97.0 | 0 | only GitHub-owned `pages-build-deployment` (dynamic path, not in repo) | shipped deploy | Not a repo-authored gate |
| `dig +short MX squz.com` / `dig MX squz.com +noall +answer` | BIND 9.10.6 | 0 | empty answer | aux | Did not send SMTP. Empty MX + GitHub Pages A records imply no mail acceptor |
| `dig +short A squz.com` | BIND 9.10.6 | 0 | `185.199.108–111.153` (GitHub Pages) | aux | — |
| `dig blog.squz.com +noall +answer` | BIND 9.10.6 | 0 | no records | aux | NX/empty — fetch of `http://blog.squz.com/privacy-policy/` failed DNS |
| `curl -sI https://squz.com/media/yourworld-screenshot.jpg` | curl (system) | 0 | **HTTP/2 404** | shipped | — |
| `curl -sI https://squz.com/multimaze.html` | curl | 0 | **HTTP/2 404** | shipped | URL cited by App Store developer website, not by this repo |
| `curl -sI https://squz.com/support.html` and Esfera/MultiMaze HTML | curl | 0 | 200 | shipped | — |
| `curl -sI https://squz.com/esfera/media/leaderboard.png` | curl | 0 | 200 (asset exists, unreferenced) | shipped | — |
| Python 3.13 inventory of `href`/`src` vs filesystem | Python 3.13.0 | 0 | only missing local asset: `media/yourworld-screenshot.jpg` | aux | Does not crawl live HTTP except follow-up curls |
| Followed live URL check of every `http(s)` href/src | curl `-L` | 0 | all 200 except the two 404s above | shipped | App Store pages 200; does not parse Connect privacy URL |
| `tidy -e -q` on all HTML | Apple HTML Tidy 31 Oct **2006** build 13462 | 0 | HTML4-era errors (`<header>`, `<section>`, `<video>` “not recognized”; UTF-8 as “invalid character”) | aux | **Not an HTML5 oracle.** Warnings discarded; no finding from tidy |
| `ls media` (repo root) | — | 2 | no such directory | aux | Confirms missing screenshot tree |
| App Store fetches: Your World `id412566625`, Esfera `id411270782`, MultiMaze `id355300331` | WebFetch | n/a | privacy / developer URLs as cited in findings | shipped (Apple) | Connect is not in this git tree |
| `test -f hygiene.yaml` | — | 0 | `NO_HYGIENE` | aux | Validator not run; yaml absent by design of this audit |

`prettier` 3.9.3 is installed locally but was not run against HTML (not a declared project formatter; running it would not decide architecture).

## Dimension vector

No prior entropy report. “Change from baseline” is n/a (first audit).

| Dimension | State | Evidence summary | Change from baseline |
|---|---|---|---|
| Architecture topology | concern | Observed three-product Pages host vs declared Your World landing; MultiMaze has policy only; root `/` is not a studio index | n/a |
| Redundancy / sources of truth | concern | App Store, dead blog, and this repo all claim privacy-policy authority; legal HTML shells copied per app | n/a |
| Change amplification | concern | Inline CSS + copy per file; App Store URLs not represented in repo, so a path rename cannot update Connect | n/a |
| Local code quality | concern | Small, readable static HTML, but the only homepage image is a 404; unused 260K PNG remains | n/a |
| Correctness / verification | critical | No tests, no link checker, no journeys. Live 404s on homepage image and MultiMaze developer URL. Dead COPPA contact | n/a |
| Security / dependencies | healthy | Static pages, no JS, no cookies, no lockfile. Secret scanning off is proportionate; attack surface is HTML/CNAME | n/a |
| Build / release / operations | concern | GitHub Pages auto-deploys `main` with HTTPS. No branch protection, no repo CI, `custom_404=false` | n/a |
| Documentation / governance | concern | README and GitHub description still Your World only; no LICENSE; no `hygiene.yaml`; no `AGENTS.md` | n/a |

Not collapsed to a scalar.

## Observed architecture

### Deployable unit

One GitHub Pages site. `CNAME` contains `squz.com`. Pages source is branch `main`, path `/`, `https_enforced=true`. `www.squz.com` CNAME → `squz.github.io` and redirects to `https://squz.com/`. There is no Makefile, package manifest, or workflow file in the repo. Deploy is GitHub’s built-in `pages-build-deployment`.

### Entry points (shipped)

```
https://squz.com/                      → index.html          (Your World)
https://squz.com/privacy.html          → privacy.html
https://squz.com/support.html          → support.html        (200, not linked from /)
https://squz.com/esfera/               → esfera/index.html
https://squz.com/esfera/privacy.html
https://squz.com/esfera/support.html
https://squz.com/multimaze/privacy.html
https://squz.com/multimaze.html        → 404
https://squz.com/multimaze/            → 404 (no index)
```

### Dependency direction

No code imports. Direction is hyperlink-only:

- Your World landing → App Store (`apps.apple.com/au/app/your-world/id412566625`) and `/privacy.html`. Does **not** link `/support.html`, Esfera, or MultiMaze.
- Esfera landing → App Store (`id411270782`), relative `privacy.html` / `support.html`, `/`.
- Esfera privacy ↔ support; both → `/`.
- MultiMaze privacy → `/` and Apple’s privacy policy. No landing, no support page, no App Store link.

External authorities not in git: App Store Connect privacy URL, developer website URL, and privacy nutrition labels.

### Public surfaces

| Surface | Owner in repo | Observed consumer |
|---|---|---|
| Your World landing | `index.html` | `https://squz.com/`; App Store “Developer Website” for Your World (`http://squz.com`) |
| Your World privacy | `privacy.html` | Linked from landing. **Not** the URL Apple currently publishes (see ENT-003) |
| Your World support | `support.html` | Reachable if typed; no inbound from landing |
| Esfera marketing + legal | `esfera/*` | App Store privacy `https://squz.com/esfera/privacy.html` and developer site `https://squz.com/esfera/` — **match** |
| MultiMaze privacy | `multimaze/privacy.html` | App Store privacy **matches**. Developer website `http://squz.com/multimaze.html` **does not** |

### Declared vs observed rules

| Rule | Class |
|---|---|
| Static HTML on GitHub Pages at `squz.com` | declared (CNAME + Pages API) and observed; agree |
| Per-app privacy URL under `/{app}/privacy.html` | observed for Esfera and MultiMaze; inferred intent from commit `b6d968a` (“mirrors the per-app layout used by esfera/”) |
| Root `/` is the Squz studio home | inferred from Esfera/MultiMaze “Squz” links to `/`; **contradicted** by `index.html` being a Your World product page |
| README: “Landing page and privacy policy for Your World by Squz” | declared; **contradicted** by Esfera + MultiMaze trees |
| Esfera Phase 1: local play shipped, online/leaderboard “Coming Soon”, no fake support email | declared in commits `e2c3d96`, `f40766d`, `b17c127`, `08d727c` and observed in `esfera/*` — agree |
| Contact email `hello@squz.com` is valid | declared in Your World pages; **contradicted** by Esfera history (“No hello@squz.com mailbox exists”) and empty MX |

Unknown intent (owner judgment, not mechanical): whether `/` should become a catalog; whether MultiMaze should get `index.html` + `support.html`; whether Your World App Store metadata will be updated or left frozen with the 2016 binary.

## Findings

### ENT-001: Homepage hero image 404s on the shipped path

- **Priority:** P1
- **Dimensions:** Local code quality; Correctness / verification
- **Status:** observed fact
- **Evidence:**
  - `index.html:54` — `<img src="media/yourworld-screenshot.jpg" …>`
  - No `media/` directory in the repository (`ls media` fails; `git ls-files` has no `media/yourworld-screenshot.jpg`)
  - Live: `curl -sI https://squz.com/media/yourworld-screenshot.jpg` → HTTP 404
  - Present since `a3c633d` (2025-06-29, “add privacy and hompages for yourworld app submission”); never added
- **Mechanism:** the only product image on `https://squz.com/` is a broken relative URL. GitHub Pages has no custom 404, so visitors see the default GitHub missing-image / empty slot on the studio origin.
- **Blast radius:** every visitor to `/`, including App Store “Developer Website” traffic for Your World and users sent “via squz.com” from MultiMaze’s privacy page.
- **Counterevidence checked:** Esfera media paths resolve (200). No other local `src` is missing. The 404 is not a Pages caching miss (`x-github-request-id` present, `content-type: text/html`).
- **Smallest coherent remediation:** add the screenshot at `media/yourworld-screenshot.jpg`, or delete the `<img>` until an asset exists.
- **Verification:** `curl -sI -o /dev/null -w '%{http_code}' https://squz.com/media/yourworld-screenshot.jpg` equals 200, and a filesystem check that every HTML `src` exists.
- **Ratchet candidate:** CI command that extracts `src`/`href` from HTML and fails on non-200 for same-origin URLs (and missing files).

### ENT-002: Your World COPPA contact is `hello@squz.com`; the domain cannot receive mail

- **Priority:** P1
- **Dimensions:** Correctness / verification; Documentation / governance
- **Status:** observed fact (DNS, git, page text); inference (SMTP delivery fails)
- **Evidence:**
  - `privacy.html:30` claims COPPA compliance and a children’s audience
  - `privacy.html:43` — contact `mailto:hello@squz.com`
  - `support.html:56` — same mailbox as the remaining help path
  - `dig MX squz.com` returns no MX; `dig A squz.com` returns GitHub Pages IPs (`185.199.108–111.153`), which do not speak SMTP
  - Commit `e44def5` (Esfera): “No hello@squz.com mailbox exists.”
  - Commit `08d727c`: removed fake `support@squz.com` from Esfera only; Your World pages were not updated
- **Mechanism:** a privacy policy for a 4+ / under-13 app names a contact that sibling products already treated as nonexistent. Empty MX plus GitHub Pages A records is a second independent signal that mail to `@squz.com` has no acceptor. COPPA and App Review expect a working contact.
- **Blast radius:** parents and App Review using Your World privacy/support; any future policy that copies this mailbox.
- **Counterevidence checked:** Esfera and MultiMaze no longer use email (App Store review / future in-app feedback / `/`). No MX on `www.squz.com` either. TXT has only `google-site-verification=…`. Mail was not sent (no live bounce captured).
- **Smallest coherent remediation:** replace Your World contact with a mailbox that actually receives mail, **or** with the same App Store-review path used by Esfera/MultiMaze. Do not invent another `@squz.com` address without MX.
- **Verification:** if email is kept: MX (or documented third-party receiver) plus a received test message. If email is removed: grep for `mailto:` is empty, and privacy still names a reachable channel.
- **Ratchet candidate:** `dig MX squz.com` must be non-empty **or** HTML must not contain `mailto:` `@squz.com`.

### ENT-003: Your World’s live App Store privacy URL is a dead blog, not `privacy.html`

- **Priority:** P1
- **Dimensions:** Redundancy / sources of truth; Correctness / verification
- **Status:** observed fact
- **Evidence:**
  - This repo serves `https://squz.com/privacy.html` (live 200) from `privacy.html`
  - App Store listing `https://apps.apple.com/au/app/your-world/id412566625` publishes privacy policy `http://blog.squz.com/privacy-policy/` and developer website `http://squz.com`
  - `dig blog.squz.com` has no records; WebFetch of that URL failed DNS (`nodename nor servname provided`)
  - Esfera listing privacy `https://squz.com/esfera/privacy.html` **matches** this repo
  - MultiMaze listing privacy `https://squz.com/multimaze/privacy.html` **matches** this repo
  - `privacy.html` was added in `a3c633d` for “yourworld app submission” but Connect still points at the blog
- **Mechanism:** three authorities for one policy (Connect URL, dead blog, this file). A fix in `privacy.html` does not change what Apple shows. The URL Apple shows does not resolve.
- **Blast radius:** every App Store privacy-policy tap for Your World (`id412566625`). Repo edits cannot close it without a Connect change.
- **Counterevidence checked:** HTTPS `https://squz.com/privacy.html` is live. Esfera/MultiMaze Connect URLs were updated to this host. Your World listing last binary update shown as 2016-02-17; metadata can still be edited without a binary.
- **Smallest coherent remediation:** in App Store Connect, set Your World privacy URL to `https://squz.com/privacy.html` (and developer site to `https://squz.com/` if still HTTP). Optionally 301 `blog.squz.com` if that name is ever restored.
- **Verification:** fetch the App Store page and assert the privacy `href` equals `https://squz.com/privacy.html`; DNS for any leftover blog host.
- **Ratchet candidate:** periodic/manual attestation of Connect URLs vs repo paths (Connect is not in git). A documented `docs/app-store-urls.md` plus a fetch check would make drift visible; do not invent it in this audit.

### ENT-004: Root `/` is a Your World product page, not the studio hub other products and App Store metadata assume

- **Priority:** P1
- **Dimensions:** Architecture topology; Change amplification
- **Status:** observed fact
- **Evidence:**
  - `index.html:6,49-57` — title, hero, screenshot, CTA are Your World only
  - `esfera/index.html:269`, `esfera/privacy.html:51`, `esfera/support.html:58`, `multimaze/privacy.html:43,46` all send “Squz” / “get in touch” to `/`
  - `multimaze/privacy.html` has no product landing; `https://squz.com/multimaze/` is 404
  - App Store MultiMaze (`id355300331`) developer website is `http://squz.com/multimaze.html` → live **404**
  - App Store Esfera developer website `https://squz.com/esfera/` → 200 (complete mini-site)
  - `README.md:1-2` and GitHub repo description still say Your World only
- **Mechanism:** adding Esfera and MultiMaze under path prefixes did not change the root document. Callers that treat `squz.com` as the company origin land on an unrelated geography-app pitch with a broken image. MultiMaze’s Connect developer URL was never created in this tree (`multimaze.html` does not exist).
- **Blast radius:** Esfera/MultiMaze “Squz” links; MultiMaze App Store “Developer Website”; anyone expecting a catalog of Squz apps (App Store “More by Squz” lists three).
- **Counterevidence checked:** per-app directories are a sound layout (Esfera works). MultiMaze privacy URL itself is correct. A studio catalog is not required by Pages; it **is** implied by the inbound links and Connect URL that this host 404s.
- **Smallest coherent remediation:** either (a) make `index.html` a short studio index linking Your World, Esfera, MultiMaze, or (b) add `multimaze.html` (or `/multimaze/`) as a landing/redirect and stop promising `/` as MultiMaze contact. Update Connect developer website to a 200 URL.
- **Verification:** `https://squz.com/multimaze.html` (or the Connect URL) returns 200; `/` mentions or links all three live apps.
- **Ratchet candidate:** same-origin link checker plus a recorded list of Connect developer URLs.

### ENT-005: Your World `support.html` is shipped but unreachable from the landing

- **Priority:** P2
- **Dimensions:** Architecture topology; Change amplification
- **Status:** observed fact
- **Evidence:**
  - `index.html:59-61` links only `/privacy.html`
  - `support.html` exists; live `https://squz.com/support.html` is 200
  - `support.html:58` links back to `/index.html` and `/privacy.html`
  - Esfera landing does link support (`esfera/index.html:268`)
- **Mechanism:** FAQ work is stranded. Users on `/` cannot discover support; only privacy is offered. The page can rot without anyone noticing from the primary entry.
- **Blast radius:** Your World customers arriving via the site or developer website.
- **Counterevidence checked:** App Store Your World page fetch did not surface a Support URL field in the scraped “Information” block (Privacy Policy and Developer Website only). If Connect has a support URL it was not visible there — residue.
- **Smallest coherent remediation:** add a Support link next to Privacy on `index.html` (and in the footer).
- **Verification:** grep/landing parse: `index.html` contains `support.html`; live homepage links it.
- **Ratchet candidate:** assert each `*/index.html` (and root `index.html`) links its sibling `support.html` when that file exists.

### ENT-006: Your World privacy text and the live App Store listing disagree about sharing and third parties

- **Priority:** P2
- **Dimensions:** Redundancy / sources of truth; Correctness / verification
- **Status:** observed fact (text disagreement); needs verification (whether the 2016 binary still shares to Facebook/Twitter)
- **Evidence:**
  - App Store Your World description still lists “Facebook and Twitter sharing (including posting images)” and Game Center
  - `privacy.html:19-27` collects anonymous usage, crash reports, optional Game Center; **does not mention** Facebook, Twitter, or photo-album screenshot export
  - `privacy.html:32-37` third parties: Firebase + Game Center only
  - Listing last shown version 3.1.4 (2016-02-17); privacy page effective date `privacy.html:15` 2025-06-29
- **Mechanism:** two public documents describe the same app’s data practices. Reviewers and parents can be told contradictory stories. If sharing still exists in the binary, the 2025 policy under-discloses; if it was removed, the App Store copy over-claims.
- **Blast radius:** App Review, COPPA-sensitive readers, any future Your World update that reuses this policy.
- **Counterevidence checked:** nutrition labels on the listing (“Data Not Linked to You”: Usage Data, Diagnostics) are consistent with Firebase + crash logs at a coarse level. Game Center **is** described in both. Binary not available in this repo.
- **Smallest coherent remediation:** inventory the shipping Your World binary’s actual SDKs/share sheets; align `privacy.html` **and** the App Store description/nutrition labels to that inventory.
- **Verification:** policy section list equals the audited SDK/share list; App Store description no longer names unused social share.
- **Ratchet candidate:** manual attestation at each App Store metadata edit (binary is not in this repo).

### ENT-007: Legal chrome (contact, entity, copyright year) is copied per page and has already drifted

- **Priority:** P2
- **Dimensions:** Redundancy / sources of truth; Change amplification
- **Status:** observed fact
- **Evidence:**
  - Identical `<style>` block in `esfera/privacy.html:7-13` and `multimaze/privacy.html:7-13` (SHA `72e2b54b41d1`); Esfera support differs only in `h2` margin (`esfera/support.html:7-13`)
  - Contact channels: Your World email (`privacy.html:43`, `support.html:56`); Esfera “future in-app Feedback” (`esfera/privacy.html:48`, `esfera/support.html:55`); MultiMaze “App Store or squz.com” (`multimaze/privacy.html:43`)
  - Legal name: `index.html:65` “© 2025 Squz”; `esfera/index.html:271` “© 2026 Squz Pty Ltd”; App Store seller is “Squz Pty Ltd”
  - Your World email survived the Esfera “no mailbox” fixes (`e44def5`, `08d727c`) because it is a separate copy
- **Mechanism:** privacy **substance** should differ per app (Firebase vs TelemetryDeck vs none) — that is healthy. The **shell** (how to contact Squz, which legal entity, which year) is one studio fact implemented N times. The email incident is the proof that a studio-wide fix does not propagate.
- **Blast radius:** next contact-policy or entity-name change must touch every HTML file; misses become public legal text.
- **Counterevidence checked:** collapsing the three policies into one file would mix incompatible data practices — do **not** unify body copy. CSS-only duplication is cheap; the finding is the identity/contact fields, not the 5-line stylesheet.
- **Smallest coherent remediation:** keep per-app policy bodies; extract a single footer/contact snippet (even a few duplicated lines with a comment pointing at the owner), or a short shared include if the site ever grows a generator. Until then, a checklist: grep entity, year, `mailto:`.
- **Verification:** grep for `mailto:`, `Squz Pty`, `© 20` yields one consistent studio identity except where a page documents a deliberate exception.
- **Ratchet candidate:** grep CI that `mailto:@squz.com` is empty unless MX exists; optional allow-list of copyright strings.

### ENT-008: `esfera/media/leaderboard.png` is shipped but unreferenced after Phase 1

- **Priority:** P3
- **Dimensions:** Local code quality
- **Status:** observed fact
- **Evidence:**
  - File tracked, 260K, live 200 at `https://squz.com/esfera/media/leaderboard.png`
  - Removed from markup in `f40766d` (“Hide leaderboard screenshot for Phase 1”)
  - Current screenshots: `esfera/index.html:253-255` (menu, gameplay, tutorial only)
  - `rg leaderboard` in HTML no longer names the PNG
- **Mechanism:** hiding the screenshot left the binary in the tree. Harmless except bytes on Pages and a future editor re-adding a “coming soon” feature image by path habit.
- **Blast radius:** Esfera Pages payload (~260K); accidental republish of unshipped UI.
- **Counterevidence checked:** may be intentional stash for the online launch. If so, this is accepted residue, not a defect — owner call.
- **Smallest coherent remediation:** delete the PNG until the leaderboard ships, **or** document it as embargoed art.
- **Verification:** `git ls-files 'esfera/media/*'` equals images referenced in `esfera/index.html`.
- **Ratchet candidate:** unused-media check (files under `**/media/` must be referenced).

### ENT-009: Homepage download claim disagrees with the App Store listing

- **Priority:** P3
- **Dimensions:** Documentation / governance; Redundancy / sources of truth
- **Status:** observed fact
- **Evidence:**
  - `index.html:50` — “over 1 million downloads”
  - App Store Your World page: “With 483,528 downloads, and counting”
- **Mechanism:** two public numbers for the same product. Not a security issue; it is a competing marketing fact on the shipped homepage.
- **Blast radius:** `/` visitors; App Review comparison with listing copy.
- **Counterevidence checked:** “1 million” might be lifetime across stores/versions; the listing number is the only cited figure and is lower. No analytics in this repo to reconcile.
- **Smallest coherent remediation:** quote the listing figure, drop the number, or cite a dated source.
- **Verification:** homepage string equals the chosen source of truth.
- **Ratchet candidate:** none until an owner picks the authority.

### ENT-010: Steady-state governance is undeclared (no LICENSE, hygiene, CI, or accurate README)

- **Priority:** P3
- **Dimensions:** Documentation / governance; Build / release / operations
- **Status:** observed fact
- **Evidence:**
  - No `LICENSE`, `.gitignore`, `.github/workflows/*`, `AGENTS.md`, `hygiene.yaml`
  - `README.md:1-2` omits Esfera and MultiMaze
  - Branch `main` unprotected; GitHub secret scanning / Dependabot disabled
  - Only workflow is GitHub-owned Pages build
  - Public repo (`visibility: public`), `licenseInfo: null`
- **Mechanism:** a public marketing/legal host can be edited and auto-deployed by anyone with push to `main`. Missing LICENSE leaves reuse terms unspecified. README/description send contributors to the wrong product shape.
- **Blast radius:** future editors; Pages deploy of legal text without review.
- **Counterevidence checked:** two authors (`Andrew Cantos` 13, `Marcelo Cantos` 2); small trusted tree. No secrets in files. Static HTML has little supply-chain surface. Branch protection would still help legal pages.
- **Smallest coherent remediation:** LICENSE (owner choice); README that lists the three URL trees; optional Pages deploy from a protected `main`. Hygiene yaml only if the owner wants a declared floor (this audit must not write it).
- **Verification:** `LICENSE` exists; `gh api` default branch protected or ruleset present if that is chosen; README names `/`, `/esfera/`, `/multimaze/privacy.html`.
- **Ratchet candidate:** `hygiene.yaml` items for `file: LICENSE`, `file: README.md`, optional `gh_setting` branch protection — only after owner adoption.

## Redundancy and competing-source-of-truth inventory

| Fact | Copy A | Copy B | Copy C | Drift? |
|---|---|---|---|---|
| Your World privacy policy URL | `privacy.html` @ `https://squz.com/privacy.html` | App Store Connect → `http://blog.squz.com/privacy-policy/` | DNS: `blog.squz.com` nonexistent | **Yes** (ENT-003) |
| Esfera privacy URL | `esfera/privacy.html` | App Store → `https://squz.com/esfera/privacy.html` | — | No |
| MultiMaze privacy URL | `multimaze/privacy.html` | App Store → `https://squz.com/multimaze/privacy.html` | — | No |
| MultiMaze developer website | missing `multimaze.html` | App Store → `http://squz.com/multimaze.html` | — | **Yes** (ENT-004) |
| Studio home | `index.html` = Your World | Esfera/MultiMaze “Squz” → `/` | README = Your World only | **Yes** (ENT-004) |
| Contact | `hello@squz.com` | Esfera: App Store / future feedback | MultiMaze: App Store / `/` | **Yes** (ENT-002, ENT-007) |
| Legal entity / year | “© 2025 Squz” | “© 2026 Squz Pty Ltd” | App Store seller “Squz Pty Ltd” | **Yes** (ENT-007) |
| Your World data practices | `privacy.html` (Firebase, Game Center) | App Store copy (Facebook/Twitter share, Game Center) | Nutrition labels (usage + diagnostics) | **Partial** (ENT-006) |
| Downloads | `index.html` “1 million” | App Store “483,528” | — | **Yes** (ENT-009) |
| Product name | Page title “MultiMaze 2” | App Store title “MultiMaze” `id355300331` | Commit “MultiMaze 2” | Naming residue (not ENT) |
| Privacy CSS shell | `esfera/privacy.html` ≡ `multimaze/privacy.html` | Esfera support near-clone | Your World privacy shorter clone | Cosmetic; substance must stay split |

**Deliberate (retain):** three privacy **bodies**. Firebase ≠ TelemetryDeck ≠ “collects no personal information”. Merging them would create a worse competing truth.

## Healthy structure worth retaining

- **Per-app directories** (`esfera/`, `multimaze/`) isolate incompatible legal facts. Esfera Phase 1 commits rewrote privacy/support when online features slipped; that is the right unit of change.
- **Esfera App Store URLs match the tree** (developer `https://squz.com/esfera/`, privacy `…/esfera/privacy.html`).
- **MultiMaze privacy URL matches** `https://squz.com/multimaze/privacy.html`; policy content matches the listing nutrition label (“Data Not Collected”) and commit `b6d968a` (local SQLite, StoreKit only).
- **Esfera landing honesty:** available features first, online/leaderboard/correspondence marked Coming Soon (`esfera/index.html:223-246`); privacy `esfera/privacy.html:41-42` says the policy will be updated before new collection; support `esfera/support.html:51-52` does not document unshipped online play as current.
- **No fake Esfera email** after `e44def5` / `08d727c`.
- **Static, script-free pages** — no XSS surface from first-party JS, no lockfile, no cookies. HTTPS enforced; HTTP and `www` redirect to `https://squz.com/`.
- **Esfera media that is referenced all 200** (`intro.mp4`, `menu.png`, `gameplay.png`, `tutorial.png`).
- **CNAME + Pages** is an appropriate deploy for this size; do not introduce a bundler to “architect” seven HTML files.

## Hygiene posture

`hygiene.yaml` is **absent**. Per the hygiene skill and owner brief: **hygiene posture not declared**. The validator was not run; this audit did not initialize a yaml.

Implied reality (not declared, therefore not drift):

| Dimension | What exists | Floor |
|---|---|---|
| correctness | no tests, no link checker; live 404s | undeclared |
| security | public static site; GitHub secret scanning off | undeclared |
| quality | no lint/format in CI | undeclared |
| docs | README exists but stale; no LICENSE | undeclared |
| release | GitHub Pages on push to `main` | undeclared |
| governance | no CODEOWNERS, no branch protection | undeclared |
| build | no build | undeclared |

Overlap with entropy: ENT-001/004 would become hygiene `command:` evidence if a link checker is adopted. ENT-010 is the missing declaration itself. Do not treat this paragraph as a held-tier vector.

## Oracle coverage and residue

| Property | How decided today | Gap |
|---|---|---|
| Same-origin URLs and assets resolve | This audit’s curl/Python (one-shot) | No standing CI; ENT-001/004 escaped |
| HTML5 validity | None (Apple Tidy 2006 is not a gate) | Acceptable for this stack unless owner wants `tidy` HTML5 or vnu |
| Privacy URL on App Store equals repo path | Manual WebFetch | Connect not in git; ENT-003 |
| Mail to listed addresses is deliverable | DNS MX empty | No SMTP probe sent (ENT-002 inference) |
| Privacy text matches shipping binaries | Partial: Esfera/MultiMaze listings vs pages | Your World binary not in repo (ENT-006) |
| Layout/CSS of landings | Not rendered in a browser this audit | `web-development.md` would require a real render **if** CSS were being changed; no visual finding claimed |
| Owner-visible journey | None | Static site; a curl allow-list is the proportionate “journey” |
| Pages HTTPS / custom domain | GitHub Pages API `https_enforced=true`, live 301s | Certificate GitHub-managed |
| No secrets in tree | Manual read of 7 HTML files + CNAME | No scanner |

Failed/skipped checks: `hygiene_check.py` skipped (no yaml); `jscpd` not installed (clone inspection was manual and sufficient); HTML Tidy not treated as an oracle.

**Owner residue (intent only):**

1. Should `https://squz.com/` be a studio catalog or remain a Your World product page?
2. Keep `hello@squz.com` (then stand up mail) or drop email studio-wide?
3. Update frozen Your World Connect metadata without a binary, or leave listing archaeology in place?
4. Is `leaderboard.png` embargoed art or leftover?
5. LICENSE for a public legal/marketing host?
6. Is “MultiMaze 2” the public name (then rename the App Store listing) or internal?

## Remediation sequence

1. **Oracle seam:** add a same-origin link/asset check (file exists + live 200) and run it on Pages-like paths. This would have failed ENT-001 and ENT-004 today.
2. **Contact truth:** remove or replace `hello@squz.com` on Your World pages (ENT-002) in the same change as any catalog/footer work (ENT-007).
3. **Connect alignment (outside git, required to close ENT-003/004):** Your World privacy → `https://squz.com/privacy.html`; MultiMaze developer website → a URL this repo actually serves.
4. **Root ownership:** studio index **or** explicit product pages so `/` is no longer a false hub (ENT-004). Link `support.html` from the Your World landing (ENT-005).
5. **Policy vs binary:** inventory Your World sharing/Firebase; align `privacy.html` and listing copy (ENT-006). Do not merge Esfera/MultiMaze/Your World policy bodies.
6. **Residue cleanup:** screenshot asset or drop the `<img>`; delete or document `leaderboard.png`; fix download claim; LICENSE + README (ENT-008–010).
7. **Ratchet:** only after (1) is green, optionally declare `hygiene.yaml` floors that point at the link-check command and `file: LICENSE`. Do not write that yaml in this audit.
8. **Re-audit** on the same dimension definitions and the same URL list.

No architectural rewrite is required. The site should stay static HTML; the missing piece is a single studio-facing index (or honest per-product roots) plus Connect URLs that point at files that exist.
