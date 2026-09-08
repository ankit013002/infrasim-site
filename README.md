# InfraSim legal / support site

Static HTML, no build step, no external assets. This directory is the source
for the public pages required by App Store / Play Store review: privacy
policy, terms of service, support, and (Play-required) account deletion.

`infrasim.app` does not currently resolve in DNS. Until it does, this site is
published on **GitHub Pages** so the app can ship with working legal URLs.

## Why GitHub Pages needs a workflow instead of a folder setting

GitHub Pages' "deploy from a branch" option only serves the repository root
(`/`) or a `/docs` folder of the chosen branch. This repo already uses `docs/`
for engineering documentation, and the site lives in `/site`, so "deploy from
branch" cannot point at it directly. Instead, `.github/workflows/pages.yml`
uses `actions/upload-pages-artifact` + `actions/deploy-pages` to publish the
`site/` folder on every push to `master` that touches it.

## One-time setup (owner)

1. Push this branch's changes (including `.github/workflows/pages.yml`) to
   `master` on `https://github.com/ankit013002/infrasim`.
2. In the GitHub repo: **Settings → Pages**.
3. Under **Build and deployment → Source**, choose **GitHub Actions** (not
   "Deploy from a branch" — that option is left showing `docs/`, which is
   unrelated to this site).
4. Push to `master` (or manually run the workflow from the **Actions** tab —
   "Deploy legal site to Pages" → **Run workflow**). The first run creates the
   `github-pages` environment automatically.
5. Confirm the deployment URL in the Pages settings page and in the workflow
   run summary. It will be:

   ```
   https://ankit013002.github.io/infrasim/
   ```

## Resulting URLs

```
https://ankit013002.github.io/infrasim/
https://ankit013002.github.io/infrasim/privacy.html
https://ankit013002.github.io/infrasim/terms.html
https://ankit013002.github.io/infrasim/support.html
https://ankit013002.github.io/infrasim/delete-account.html
```

These are the URLs already wired into the app via
`EXPO_PUBLIC_LEGAL_BASE_URL` (see `.env.example` and `src/config/app.ts`) —
no further app change is needed once Pages is live.

**Before submitting to either store**, open each URL from a phone on cellular
data (not wifi, not localhost) to confirm it resolves publicly. A dead legal
URL is an automatic App Store rejection.

## Editing content

Edit the `.html` files directly — each is self-contained (inline `<style>`,
no external JS/CSS). Commit and push to `master`; the workflow redeploys
automatically. There is no template/partial system by design, to keep the
site buildable with zero tooling — if the pages grow enough to need shared
layout, consider a small static site generator at that point, not before.

## Switching to `infrasim.app` once DNS exists

1. Buy/point the domain, then in this repo's **Settings → Pages → Custom
   domain**, enter `infrasim.app` (or a subdomain like `legal.infrasim.app`).
   GitHub commits a `site/CNAME` file automatically when you save this in the
   UI — do not hand-author one, since it must be a single line with exactly
   the domain GitHub verified.
2. At your DNS provider, add the records GitHub's Pages settings page shows
   you (typically an `ALIAS`/`ANAME` or four `A` records for an apex domain,
   or a `CNAME` record for a subdomain) pointing at
   `ankit013002.github.io`.
3. Wait for DNS to propagate and for GitHub to issue the HTTPS certificate
   (shown as "DNS check successful" / "Enforce HTTPS" becoming available in
   Pages settings).
4. Set `EXPO_PUBLIC_LEGAL_BASE_URL=https://infrasim.app` in your EAS
   environment (see `docs/LAUNCH_CHECKLIST.md`) and rebuild — every legal/
   support/delete-account link in the app derives from this one variable.
5. Leave the GitHub Pages deployment running at the `github.io` URL as a
   fallback; it costs nothing and does no harm to keep both working.
