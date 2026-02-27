# Implementation Plan: Set Up Hugo Site for AkamonLabs

## Context

The akamonlabs.com repository is initialized with git but contains no Hugo site structure — only a README, CLAUDE.md, and two planning documents. The goal is to create a fully functional Hugo static site that serves as a brand landing page for AkamonLabs apps, with the first app being StageWhisper. The site should deploy to GitHub Pages via GitHub Actions, following the architecture described in `page-reference-architecture.md` (based on sergiocarrilho.com).

**Prerequisites (already met):** Hugo v0.155.1+extended, Go 1.25.6, git remote configured at `git@github-personal:scarrilho/akamonlabs.com.git`.

**Reference documents in repo:**
- `page-requirement.md` — project goals and requirements
- `page-reference-architecture.md` — detailed architecture blueprint from the reference site

---

## Step 1: Initialize Hugo + Go Modules

Run these commands in the repo root:

```bash
hugo new site . --force
hugo mod init github.com/scarrilho/akamonlabs.com
hugo mod get github.com/adityatelange/hugo-PaperMod
```

- `--force` overlays Hugo scaffolding onto the existing repo without disturbing README.md/CLAUDE.md.
- This creates `go.mod`, `go.sum`, and the standard Hugo directory structure (`archetypes/`, `content/`, `data/`, `i18n/`, `layouts/`, `static/`, `themes/`, `assets/`).

---

## Step 2: Write `hugo.toml`

Replace the auto-generated `hugo.toml` with the following:

```toml
baseURL = "https://akamonlabs.com/"
languageCode = "en-us"
title = "AkamonLabs"
enableRobotsTXT = true
theme = ["github.com/adityatelange/hugo-PaperMod"]

[params]
  description = "AkamonLabs — Apps for iOS, Android, and Mac."
  defaultTheme = "light"
  author = "AkamonLabs"
  keywords = ["AkamonLabs", "StageWhisper", "iOS", "Android", "Mac", "Apps"]
  # Disable blog-specific features
  ShowShareButtons = false
  ShowReadingTime = false
  ShowPostNavLinks = false
  ShowBreadCrumbs = false
  ShowCodeCopyButtons = false
  disableSpecial1stPost = true

[params.cover]
  hidden = false
  hiddenInList = false
  hiddenInSingle = false

[params.assets]
  disableFingerprinting = true

# Homepage: hero-style landing with logo and app links
[params.profileMode]
  enabled = true
  title = "AkamonLabs"
  subtitle = "Apps for iOS, Android, and Mac"
  imageUrl = "/img/logo.png"
  imageTitle = "AkamonLabs"
  imageWidth = 200
  imageHeight = 200

  [[params.profileMode.buttons]]
    name = "StageWhisper"
    url = "/stagewhisper/"

# App-specific params (reusable in templates)
[params.apps]
  [params.apps.stagewhisper]
    name = "StageWhisper"
    # Uncomment and fill when available:
    # appStoreUrl = "https://apps.apple.com/app/stagewhisper/idXXXXXX"
    # playStoreUrl = "https://play.google.com/store/apps/details?id=com.akamonlabs.stagewhisper"

# Navigation menu
[menu]
  [[menu.main]]
    identifier = "home"
    name = "Home"
    url = "/"
    weight = 10

  [[menu.main]]
    identifier = "stagewhisper"
    name = "StageWhisper"
    url = "/stagewhisper/"
    weight = 20
```

Key design choices:
- **profileMode** for the homepage (logo + subtitle + app buttons)
- Blog-specific PaperMod features all disabled
- `defaultTheme = "light"` (brand site, not personal blog)
- `[params.apps.stagewhisper]` for app store URLs (commented out until available, accessible in templates via `{{ .Site.Params.apps.stagewhisper.appStoreUrl }}`)
- No social icons or Google Analytics for now

---

## Step 3: Create Content

### 3a. `content/_index.md`

```markdown
---
title: "Home"
---
```

Content is driven by `profileMode` in hugo.toml. This file just tells Hugo the homepage exists.

### 3b. `content/stagewhisper/_index.md`

Uses `_index.md` (branch bundle) so the privacy policy can nest underneath as a child page. The `layout: "single"` forces PaperMod to render it as a page, not a list.

```markdown
---
title: "StageWhisper"
description: "StageWhisper — your app description here."
layout: "single"
draft: false
hidemeta: true
ShowToc: false
disableShare: true
---

## About StageWhisper

StageWhisper is an app by AkamonLabs.

<!-- Add app store badges when available:
[![Download on the App Store](/img/app-store-badge.svg)](https://apps.apple.com/app/stagewhisper/idXXXXXX)
[![Get it on Google Play](/img/google-play-badge.png)](https://play.google.com/store/apps/details?id=com.akamonlabs.stagewhisper)
-->

---

[Privacy Policy](/stagewhisper/privacy_policy.html)
```

### 3c. `content/stagewhisper/privacy-policy.md`

The `url` front matter overrides Hugo's default URL to produce the exact path required for app store submissions.

```markdown
---
title: "Privacy Policy — StageWhisper"
description: "Privacy Policy for StageWhisper by AkamonLabs."
url: /stagewhisper/privacy_policy.html
layout: "single"
draft: false
hidemeta: true
ShowToc: false
disableShare: true
---

# Privacy Policy

**Effective date:** [DATE]

**App:** StageWhisper
**Developer:** AkamonLabs

## Information We Collect

[Describe what data the app collects, if any.]

## How We Use Information

[Describe how collected information is used.]

## Third-Party Services

[List any third-party SDKs or services, e.g., analytics, crash reporting.]

## Data Retention

[Describe how long data is retained.]

## Children's Privacy

[State compliance with COPPA if applicable.]

## Changes to This Policy

[Describe how users will be notified of changes.]

## Contact

If you have questions about this privacy policy, please contact us at: [EMAIL]
```

---

## Step 4: Layout Overrides

### 4a. `layouts/partials/extend_head.html`

PaperMod's head injection hook — favicons, OpenGraph/Twitter, and JSON-LD structured data.

```html
{{- /* Favicons */ -}}
<link rel="apple-touch-icon" sizes="180x180" href="/img/favicon/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="/img/favicon/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="/img/favicon/favicon-16x16.png">
<link rel="manifest" href="/img/favicon/site.webmanifest">
<link rel="shortcut icon" href="/img/favicon/favicon.ico">

{{- /* OpenGraph & Twitter Cards (Hugo internal templates) */ -}}
{{- template "_internal/opengraph.html" . -}}
{{- template "_internal/twitter_cards.html" . -}}

{{- /* JSON-LD Structured Data: Organization */ -}}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "AkamonLabs",
  "url": "https://akamonlabs.com/",
  "description": "{{ .Site.Params.description }}"
}
</script>

{{- /* JSON-LD for SoftwareApplication pages */ -}}
{{- if and .IsPage (eq .Section "stagewhisper") (not (in .Title "Privacy")) }}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "{{ .Title }}",
  "description": "{{ .Description }}",
  "url": "{{ .Permalink }}",
  "applicationCategory": "Application",
  "operatingSystem": "iOS, Android, macOS",
  "author": {
    "@type": "Organization",
    "name": "AkamonLabs",
    "url": "https://akamonlabs.com/"
  }
}
</script>
{{- end }}
```

### 4b. `layouts/robots.txt`

```
User-agent: *
{{ if eq hugo.Environment "production" }}
Allow: /
{{ else }}
Disallow: /
{{ end }}
Sitemap: {{ "sitemap.xml" | absURL }}
```

---

## Step 5: Static Assets

### 5a. `static/CNAME`

```
akamonlabs.com
```

Single line, no trailing newline. Required for GitHub Pages custom domain.

### 5b. Create empty directories

```bash
mkdir -p static/img/favicon
```

User needs to provide later:
- `static/img/logo.png` — AkamonLabs logo (referenced by profileMode in hugo.toml)
- `static/img/favicon/favicon.ico`
- `static/img/favicon/favicon-16x16.png`
- `static/img/favicon/favicon-32x32.png`
- `static/img/favicon/apple-touch-icon.png`
- `static/img/favicon/site.webmanifest`

---

## Step 6: GitHub Actions Workflow

### `.github/workflows/hugo.yaml`

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.155.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.25'
          check-latest: true

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4

      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"

      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo mod get -u
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Post-deploy manual steps (GitHub):**
1. Repo Settings → Pages → Source → select "GitHub Actions"
2. Configure custom domain `akamonlabs.com`
3. DNS records pointing to GitHub Pages IPs (185.199.108-111.153)

---

## Step 7: `.gitignore`

```gitignore
# Hugo build output
/public
/resources
/.hugo_build.lock

# Dependencies
/node_modules
/_vendor

# OS
.DS_Store

# Planning docs (local only)
page-requirement.md
page-reference-architecture.md
```

Planning docs gitignored (stay local but not tracked). CLAUDE.md should be committed.

---

## Files Summary

| File | Action |
|------|--------|
| `hugo.toml` | Replace auto-generated |
| `go.mod` / `go.sum` | Auto-generated by `hugo mod init` + `hugo mod get` |
| `content/_index.md` | Create |
| `content/stagewhisper/_index.md` | Create |
| `content/stagewhisper/privacy-policy.md` | Create |
| `layouts/partials/extend_head.html` | Create |
| `layouts/robots.txt` | Create |
| `static/CNAME` | Create |
| `static/img/favicon/` | Create directory |
| `.github/workflows/hugo.yaml` | Create |
| `.gitignore` | Create |

---

## Verification

After all files are created, verify:

1. `hugo server -D` starts without errors
2. `localhost:1313/` — homepage shows profileMode (AkamonLabs title, subtitle, StageWhisper button)
3. `localhost:1313/stagewhisper/` — app landing page renders as a single page (not a list)
4. `localhost:1313/stagewhisper/privacy_policy.html` — privacy policy accessible at the exact URL
5. `localhost:1313/robots.txt` — shows `Disallow: /` (non-production environment)
6. `hugo --gc --minify` builds cleanly
7. `public/CNAME` exists and contains `akamonlabs.com`
8. `public/stagewhisper/privacy_policy.html` exists as a file

## Potential Issues

- **Branch bundle `_index.md` rendering as list:** If PaperMod ignores `layout: "single"`, create `layouts/stagewhisper/list.html` that delegates to the single template.
- **`url: /stagewhisper/privacy_policy.html` not working:** If Hugo strips the `.html` extension, place the privacy policy as a raw HTML file at `static/stagewhisper/privacy_policy.html` instead.
- **Missing logo:** Site builds without `static/img/logo.png` but profileMode shows a broken image. Add the logo before first real deployment.
- **Go 1.25 in CI:** If GitHub Actions doesn't have Go 1.25 yet, fall back to `go-version: '1.23'`.
