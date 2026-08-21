# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Hugo static site for rosenvold.tech — a personal tech blog. Six posts in `content/posts/`, one `content/about.md`. No build system, no tests, no linter, no CI in the repo.

## Commands

```bash
git submodule update --init          # theme lives in themes/hello-friend-ng; a fresh clone has it empty
hugo server -p 1313                  # dev server, http://localhost:1313
hugo                                 # production build into public/ (gitignored)
```

Requires **extended** Hugo (the theme compiles SCSS). Verified working on 0.165.0.

### Two dev-server traps

- **Never run a bare `hugo` while `hugo server` is running.** The server serves from disk, so a production build overwrites `public/` with `baseURL = https://rosenvold.tech` asset URLs that 404 on localhost. The page then renders completely unstyled, which looks like a CSS bug and isn't one. Restart the server to fix.
- **`pkill -f 'hugo server'` kills your own shell**, because the pattern matches the invoking bash command line. Use `pkill -x hugo`.

## Theme is a submodule, and `layouts/` shadows it

`themes/hello-friend-ng` is a git submodule (rhazdon's hello-friend-ng, MIT). Project files in `layouts/` override same-named theme files. Four of them are **near-verbatim copies of theme files** with one or two lines changed:

| File | Deviation from theme |
|---|---|
| `layouts/_default/baseof.html` | `data-theme="dark"` on `<html>` |
| `layouts/partials/footer.html` | `{{ now.Year }}` instead of the `footer.trademark` param |
| `layouts/partials/header.html` | search icon span + `{{ partial "search.html" . }}` after `</header>` |
| `layouts/index.html` | recent-posts block before `</main>` |

**After any theme submodule bump, diff each of these against its theme counterpart.** A previous bump moved `.content` out of `baseof.html` into each layout and silently broke alignment. Keep the deviations minimal so the diff stays readable.

The submodule was pinned at `8270f8e` for a long time, which **cannot build on Hugo 0.165** (`.Site.Author` and `resources.ToCSS` were both removed). It now points at `1a0a16f`. Conversely, an old host-side Hugo may fail on the current theme (`hugo.IsMultilingual`, `.Page.Store`). If a build breaks, suspect this version pairing first.

`.gitmodules` also has a stale duplicate `hugo-theme-hello-friend-ng` entry with no checkout. Harmless, pre-existing.

## Search

Client-side, no dependencies, no search library.

1. `layouts/index.searchindex.json` emits every post as JSON — title, url, date, tags, plaintext body.
2. It uses a **custom `SearchIndex` output format** (declared in `config.toml`), not Hugo's built-in `json`. That's deliberate: the theme's `head.html` hardcodes a `feed.json` alternate link whenever a `json` output exists, and ours lives at `/index.json`, so the built-in format produces a 404 link tag.
3. `layouts/partials/search.html` renders the header panel, its CSS, and the matcher. Rendered on every page via `header.html`. The index is fetched on **first open**, not page load.
4. Matching is a lowercase substring AND across whitespace-split terms. Fine at six posts; marked with a `ponytail:` comment where it would need replacing.

## Content trap: `type: "post"`

Five of the six posts set `type: "post"` in front matter, which **overrides the section**. So `where .Site.RegularPages "Type" "posts"` matches only `talos-setting-up.md` (the one that omits it).

**Filter posts by `Section`, never `Type`.** Both `layouts/index.html` and `layouts/index.searchindex.json` do. The front matter serves no purpose (there is no `layouts/post/`) and is a candidate for deletion.

## Dark mode only

`data-theme="dark"` is set in `baseof.html` markup, not JavaScript — a JS flip paints light first and flashes. `enableThemeToggle = false` in config makes the theme drop its toggle, and the theme's `main.js` then clears any stale `localStorage` theme preference on its own.

This works because the theme pairs every `prefers-color-scheme: light` rule with a `[data-theme=dark]` counterpart, and the attribute selector outranks a bare class inside a media query. The light SCSS is still in the submodule, just unreachable — don't edit it there, changes won't survive an update.

Search-panel CSS uses translucent black/grey rather than the theme's SCSS hex colors, so it survives theme palette changes.

## Verifying changes

There is no browser in this environment, so **visual claims cannot be verified** — say so rather than implying a change was seen working. What can be checked:

```bash
curl -s localhost:1313/index.json | python3 -c 'import json,sys; print(len(json.load(sys.stdin)))'   # expect 6
curl -s localhost:1313/ | grep -o '<html[^>]*>'                                                      # expect data-theme="dark"
diff themes/hello-friend-ng/layouts/partials/footer.html layouts/partials/footer.html                # expect the 1-line deviation
```

The search matcher is asserted by loading `/index.json` in `node` and re-running the predicate (`talos hetzner` → 1, `cisco` → 2, empty and garbage → 0). If real browser checks are needed, `npx playwright install chromium` has not been set up but would work.

## Git

- Remote is the SSH alias `git@github-personal:...` from `~/.ssh/config`. HTTPS resolves to the user's **work** GitHub account and gets a 403 on this repo.
- `gh`'s active account is usually the work one, which has only READ here — `gh pr create` etc. need `gh auth switch --user lucas2kdk` first. Switch back afterward.
- History is direct-to-`main` up to `f15f7d1`; the user has since asked for **GitHub flow** (feature branch + PR).
- Deploy is **Cloudflare Pages**, configured in its dashboard, not in the repo. Pushing to `main` publishes.
- `resources/_gen/` is gitignored, but two files there were committed before that and remain tracked.

## Deploy: pin the Hugo version

Cloudflare Pages' legacy build image installs **Hugo 0.54.0 (2019)** when no version is pinned. The current theme cannot build on it — the failure surfaces as a misleading i18n error:

```
Error: ".../themes/hello-friend-ng/i18n/zh-cn.toml:1:1": failed to load translations:
unable to parse translation #5 because invalid plural category newerPosts
```

That is not a theme bug or a content bug. It is Hugo 0.54 meeting a 2025 theme.

The fix lives in the Cloudflare dashboard, **not in this repo**: Settings → Builds & deployments → Environment variables → `HUGO_VERSION = 0.165.0`. Bumping Build system version to 2 also helps, since its default Hugo is modern.

`.tool-versions` pins `hugo 0.165.0` for build image v2, which reads it. **The legacy v1 image ignores it**, so it is a safety net for later, not the fix — do not treat its presence as the version being pinned.

## Config notes

`config.toml` declares CC BY-NC 4.0 in `copyright`, but it is displayed nowhere (`footer.copyright = false`, RSS disabled). It reads as a license grant; the user has been told and hasn't decided yet.
